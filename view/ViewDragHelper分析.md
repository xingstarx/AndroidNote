#ViewDragHelper

[鸿洋的博客](http://blog.csdn.net/lmj623565791/article/details/46858663)

[ViewDragHelper的源码分析](https://github.com/LittleFriendsGroup/AndroidSdkSourceAnalysis/blob/master/article/ViewDragHelper%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.md)

ViewDragHelper is a utility class for writing custom ViewGroups. It offers a number 
of useful operations and state tracking for allowing a user to drag and reposition 
views within their parent ViewGroup.

ViewDragHelper帮我们处理了各种拖拽事件，我们只需要实现固定的方法，便可以做出酷炫的效果。

```
mDragger = ViewDragHelper.create(this, 1.0f, new ViewDragHelper.Callback() {
            /**
             * 捕获该View
             * @param child
             * @param pointerId
             * @return
             */
            @Override
            public boolean tryCaptureView(View child, int pointerId) {
                return true;
            }

            /**
             *用来限制child在移动过程中左右的边界
             * @param child
             * @param left
             * @param dx
             * @return
             */
            @Override
            public int clampViewPositionHorizontal(View child, int left, int dx) {
                return left;
            }

            /**
             * 限制child在移动过程中上下的边界
             * @param child
             * @param top
             * @param dy
             * @return
             */
            @Override
            public int clampViewPositionVertical(View child, int top, int dy) {
                return top;
            }

            /**
             *手指释放的时候回调
             * @param releasedChild
             * @param xvel
             * @param yvel
             */
            @Override
            public void onViewReleased(View releasedChild, float xvel, float yvel) {
            }

            /**
             *在边界拖动时回调
             * @param edgeFlags
             * @param pointerId
             */
            @Override
            public void onEdgeDragStarted(int edgeFlags, int pointerId) {
            }
        });
```
