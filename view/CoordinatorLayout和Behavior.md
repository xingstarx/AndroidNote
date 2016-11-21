##CoordinatorLayout
CoordinatorLayout是一个更加强大的FrameLayout布局，它的作用主要是处理子View之间的相互交互

##onMeasure

```
@Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        //对子view按照依赖关系排序
        prepareChildren();
        //确保设置了preDrawListener监听器
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

##prepareChildren
```
 private void prepareChildren() {
        final int childCount = getChildCount();
        boolean resortRequired = mDependencySortedChildren.size() != childCount;
        for (int i = 0; i < childCount; i++) {
            final View child = getChildAt(i);
            //为LayoutParams设置Behavior
            final LayoutParams lp = getResolvedLayoutParams(child);
            if (!resortRequired && lp.isDirty(this, child)) {
                resortRequired = true;
            }
            //寻找设置的mAnchorView
            lp.findAnchorView(this, child);
        }
        if (resortRequired) {
            mDependencySortedChildren.clear();
            for (int i = 0; i < childCount; i++) {
                mDependencySortedChildren.add(getChildAt(i));
            }
            //按照依赖关系排序
            Collections.sort(mDependencySortedChildren, mLayoutDependencyComparator);
        }
    }
```

##getResolvedLayoutParams


##findAnchorView
```
View findAnchorView(CoordinatorLayout parent, View forChild) {
            if (mAnchorId == View.NO_ID) {
                mAnchorView = mAnchorDirectChild = null;
                return null;
            }
            if (mAnchorView == null || !verifyAnchorView(forChild, parent)) {
                resolveAnchorView(forChild, parent);
            }
            return mAnchorView;
        }

```
寻找到mAnchorView 

##resolveAnchorView
```
private void resolveAnchorView(View forChild, CoordinatorLayout parent) {
            mAnchorView = parent.findViewById(mAnchorId);
            if (mAnchorView != null) {
                View directChild = mAnchorView;
                for (ViewParent p = mAnchorView.getParent();
                        p != parent && p != null;
                        p = p.getParent()) {
                    if (p == forChild) {
                        if (parent.isInEditMode()) {
                            mAnchorView = mAnchorDirectChild = null;
                            return;
                        }
                        throw new IllegalStateException(
                                "Anchor must not be a descendant of the anchored view");
                    }
                    if (p instanceof View) {
                        directChild = (View) p;
                    }
                }
                mAnchorDirectChild = directChild;
            } else {
                if (parent.isInEditMode()) {
                    mAnchorView = mAnchorDirectChild = null;
                    return;
                }
                throw new IllegalStateException("Could not find CoordinatorLayout descendant view"
                        + " with id " + parent.getResources().getResourceName(mAnchorId)
                        + " to anchor view " + forChild);
            }
        }
```
循环遍历，进一步找到mAnchorView


##onMeasureChild

##onLayout

##onLayoutChild
