## 主要代码
网上资料太多，将这块的内容也不少，无论是知乎，简书，还是github上的，这里只针对我所处理的项目

粘贴一下我觉得关键的代码，处理TRANSLUCENT的，使之成为透明主题
```java
@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_match_actionbar);

		if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
			setTranslucentStatus(true);
		}

		SystemBarTintManager tintManager = new SystemBarTintManager(this);
		tintManager.setStatusBarTintEnabled(true);
		tintManager.setStatusBarTintResource(R.color.statusbar_bg);

	}

	@TargetApi(19) 
	private void setTranslucentStatus(boolean on) {
		Window win = getWindow();
		WindowManager.LayoutParams winParams = win.getAttributes();
		final int bits = WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS;
		if (on) {
			winParams.flags |= bits;
		} else {
			winParams.flags &= ~bits;
		}
		win.setAttributes(winParams);
	}
```
设置fitsSystemWindows，让xml布局内容位于状态栏之下，当然了这块的代码可以做一下进一步的封装
```java
 @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        supportRequestWindowFeature(Window.FEATURE_NO_TITLE);
        setContentView(getLayoutResId());//把设置布局文件的操作交给继承的子类
        ViewGroup contentFrameLayout = (ViewGroup) findViewById(Window.ID_ANDROID_CONTENT);
        View parentView = contentFrameLayout.getChildAt(0);
        if (parentView != null && Build.VERSION.SDK_INT >= 14) {
            parentView.setFitsSystemWindows(true);
        }
    }
```
这段代码来自[AndroidSystemUiTraining](https://github.com/D-clock/AndroidSystemUiTraining/blob/master/note/00_AndroidSystemUI%EF%BC%9ATranslucentBar%E7%89%B9%E6%80%A7%E7%9A%84%E4%BD%BF%E7%94%A8.md) 做了一定的封装，这样就不需要我们在每个xml的root节点里面加上`android:fitsSystemWindows = 'true'`

对于透明主题，我们需要做的就是，修改window的主题，然后修改状态栏颜色值，android4.4的版本稍微特殊，可以通过其他的方式实现修改状态栏的颜色，4.4以下无法修改颜色，5.0以上的可以通过系统api，修改statusBarColor。
修改为了透明主题后，默认情况下，我们的布局是占满全屏区域的，那么我们的xml布局是会遮挡住statsbar的。

有两种解决办法，一种是如上述的代码中，设置`findViewById(Window.ID_ANDROID_CONTENT).getChildAt(0).setFitsSystemWindows(true)`，这样强制的让布局内容从状态栏下面开始，状态栏以上的显示的是`SystemBarTintManager.setStatusBarTintResource(R.color.statusbar_bg)`设置的颜色

另外一种是不设置`setFitsSystemWindows(true)`,xml布局占满全屏，通过一个高度为25dp的view，占满状态栏的位置,用paddingTop也可以实现，可以针对具体情况处理,具体可以看知乎上的这篇https://www.zhihu.com/question/31468556，第一个答案

## 优化后
本文优化后的代码主要是这两种方式混用的，针对不同的页面采用不同的方案。



## 相关可以参考的资料
[AndroidSystemUiTraining](https://github.com/D-clock/AndroidSystemUiTraining/blob/master/note/00_AndroidSystemUI%EF%BC%9ATranslucentBar%E7%89%B9%E6%80%A7%E7%9A%84%E4%BD%BF%E7%94%A8.md)
[SystemBarTint](https://github.com/jgilfelt/SystemBarTint)
