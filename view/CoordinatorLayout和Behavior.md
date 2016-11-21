##CoordinatorLayout
CoordinatorLayout是一个更加强大的FrameLayout布局，它的作用主要是处理子View之间的相互交互

##onMeasure

```
@Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        prepareChildren();
        ensurePreDrawListener();
        final int childCount = mDependencySortedChildren.size();
        for (int i = 0; i < childCount; i++) {
            final View child = mDependencySortedChildren.get(i);
            final LayoutParams lp = (LayoutParams) child.getLayoutParams();
            int keylineWidthUsed = 0;
            final Behavior b = lp.getBehavior();
            if (b == null || !b.onMeasureChild(this, child, widthMeasureSpec, keylineWidthUsed,
                    heightMeasureSpec, 0)) {
                onMeasureChild(child, widthMeasureSpec, keylineWidthUsed,
                        heightMeasureSpec, 0);
            }
        }
        final int width = ViewCompat.resolveSizeAndState(widthUsed, widthMeasureSpec,
                childState & ViewCompat.MEASURED_STATE_MASK);
        final int height = ViewCompat.resolveSizeAndState(heightUsed, heightMeasureSpec,
                childState << ViewCompat.MEASURED_HEIGHT_STATE_SHIFT);
        setMeasuredDimension(width, height);
    }
```
在`onMeasure`方法中，执行的是对每一个childView进行测量，在测量前，先排好children的顺序(依据是否依赖来确定)，再依次测量childView，测量规则如下，先使用Behavior.onMeasureChild()计算，如果为false的话，再使用CoordinatorLayout的onMeasureChild方法进行测量





##onMeasureChild

##onLayout

##onLayoutChild
