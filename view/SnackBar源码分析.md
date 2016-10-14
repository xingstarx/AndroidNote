#Snackbar源码分析

##相关类
`Snackbar, Snackbar.Callback, SnackbarManager, SnackbarManager.Callback`


##常用API
`Snackbar.make(view, text, duration)`

`setAction(text, listener)`

`setCallback(callback)`

`show()`

`dismiss()`

##作用
Snackbar是android support包中，用来在屏幕底部弹出消息的控件，和Toast类似，用法也比较接近。


##用法

一般采用如下的代码，当然下面是一个完整的例子，包括了添加点击事件，以及显示

```
Snackbar snackbar =  Snackbar.
make(mRootView, getString(R.string.show_message),Snackbar.LENGTH_INDEFINITE)
.setAction(getString(R.string.open), mSnackBarClickListener);
snackbar.show();

```

##原理分析
本篇主要是想要对源码进行分析，用法之类的可以参考其他相关的博客，或者到github上找相关的代码

准备从两个方法讲起，一块是`show()`， 一个是`dismiss()`

###show()
在我们调用`snackbar.show()`的时候,我们能够马上看到屏幕底部显示出了一个黑色的弹出框。但是这一过程是如何完成的，不看源码还是不太清楚的，


想要把`snackbar`显示到屏幕上，需要调用`show()`，那么`show`方法究竟做了什么工作呢。

翻开源码我们看到，show方法内部只有一行代码
`SnackbarManager.getInstance().show(mDuration, mManagerCallback);`，一开始我还在想，就这一行代码能够把`snackbar`这个控件显示到屏幕上，真让人惊讶，同时也让人一脸懵逼，完全不知这是个什么情况，只好跟代码，到`SnackbarManager`里面，看看`SnackbarManager.show(mDuration, mManagerCallback)`方法是干嘛的。对于`SnackbarManager.getInstance()`先看做单例吧，目前只分析关键代码。


```
public void show(int duration, Callback callback) {
        synchronized (mLock) {
            if (isCurrentSnackbarLocked(callback)) {
                // Means that the callback is already in the queue. We'll just update the duration
                mCurrentSnackbar.duration = duration;

                // If this is the Snackbar currently being shown, call re-schedule it's
                // timeout
                mHandler.removeCallbacksAndMessages(mCurrentSnackbar);
                scheduleTimeoutLocked(mCurrentSnackbar);
                return;
            } else if (isNextSnackbarLocked(callback)) {
                // We'll just update the duration
                mNextSnackbar.duration = duration;
            } else {
                // Else, we need to create a new record and queue it
                mNextSnackbar = new SnackbarRecord(duration, callback);
            }

            if (mCurrentSnackbar != null && cancelSnackbarLocked(mCurrentSnackbar,
                    Snackbar.Callback.DISMISS_EVENT_CONSECUTIVE)) {
                // If we currently have a Snackbar, try and cancel it and wait in line
                return;
            } else {
                // Clear out the current snackbar
                mCurrentSnackbar = null;
                // Otherwise, just show it now
                showNextSnackbarLocked();
            }
        }
    }
```
在`SnackbarManager.show()`方法内部，通过加锁机制保证同一时间只显示一个`snackbar`，假设当前是第一次显示`snackbar`，那么初始状态的`mCurrentSnackbar`，`mNextSnackbar`均为`null`，代码第一次会执行`showNextSnackbarLocked()`方法。

```
private void showNextSnackbarLocked() {
        if (mNextSnackbar != null) {
			...
            final Callback callback = mCurrentSnackbar.callback.get();
            if (callback != null) {
                callback.show();
            } 
            ...
        }
    }
```

在`showNextSnackbarLocked()`方法中，是会执行`callback.show()`的，这个`callback`是来自于`mCurrentSnackbar`的属性`callback`，我们知道`mCurrentSnackbar`是`SnackbarRecord`类型的对象.只需要找找代码中什么时候给`SnackbarRecord`被实例化对象了。搜索代码发现是在我们的`show`方法中，也就是这一行代码`mNextSnackbar = new SnackbarRecord(duration, callback);` `mCurrentSnackbar`的`callback`就是这个时候传递进去的，而这个`callback`是调用`SnackbarManager.show()`方法的时候传递进去的参数，通过最开始的调用，我们是在Snackbar.show()方法内部，调用了SnackbarManager.show()，同时也是这个时候传递的参数mManagerCallback,这个mManagerCallback是Snackbar里面的一个成员变量，也是一开始就初始化了。


回到showNextSnackbarLocked()方法中，我们调用的callback.show()，其实就是mManagerCallback.show().具体执行的就是下面的

```
private final SnackbarManager.Callback mManagerCallback = new SnackbarManager.Callback() {
        @Override
        public void show() {
            sHandler.sendMessage(sHandler.obtainMessage(MSG_SHOW, Snackbar.this));
        }

        @Override
        public void dismiss(int event) {
            sHandler.sendMessage(sHandler.obtainMessage(MSG_DISMISS, event, 0, Snackbar.this));
        }
    };
```

`sHandler.sendMessage(sHandler.obtainMessage(MSG_SHOW, event, 0, Snackbar.this));`这段代码就是做的要显示的操作了。绕了一大圈又回到了这个地方，原来还是通过`hanlder`发消息，切换到主线程刷新页面啊，本质的东西还是没有变。

`sHandler`是通过静态代码块初始化的，传递的也是主线程的`looper`。这样就肯定会切换到主线程刷新页面的。

接着看看`handler`发消息后怎么走的，这个时候消息进入队列中等待调度。主线程调度到这个`MSG_SHOW`的消息时,会执行`((Snackbar) message.obj).showView();`  `message.obj`就是当前的`Snackbar`对象，调用的也是当前的`showView`方法，历经千辛万苦终于到了目的地了，太不容易了。

showView()代码还是挺多的，显示的逻辑执行的还是`animateViewIn()`方法，里面是通过view动画，或者是属性动画来实现动画效果

###dismiss()

















