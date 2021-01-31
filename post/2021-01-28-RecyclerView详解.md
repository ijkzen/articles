---
title: RecyclerView详解
categories: Android原理
---

## 概论

`RecyclerView`是一个在有限窗口显示大量数据的灵活视图；

## 绘制过程

`ChildView`的实际绘制过程由`LayoutManager`所掌控；

### onMeasure

```java
@Override
protected void onMeasure(int widthSpec, int heightSpec)
{
    // 没有自定义LayoutManager，设置默认的宽高，并不测量子视图的宽高；
    if (mLayout == null)
    {
        // 以高度为例，
        // EXACTLY：MeasureSpec中的size
        // AT_MOST：padding之和和minHeight中的较大值，
        //          这个较大值与MeasureSpec中的size的较小值；
        // UNSPECIFIED：padding之和和minHeight中的较大值
        defaultOnMeasure(widthSpec, heightSpec);
        return;
    }
    
    // 这个auto-measure代表是否由RecyclerView控制测量过程，默认情况下为false
    // 如果这个函数被重载为true，则LayoutManager::onMeasure不应该被重载
    if (mLayout.isAutoMeasureEnabled())
    {
        final int widthMode = MeasureSpec.getMode(widthSpec);
        final int heightMode = MeasureSpec.getMode(heightSpec);

        /**
         * 在默认测量流程中，此处应当直接调用RecyclerView::defaultOnMeasure()，但是很多第三方库并未
         * 按照文档编写代码，而重载了LayoutManager::onMeasure()；默认情况下，LayoutManager::onMeasure()
         * 调用了RecyclerView::defaultOnMeasure()；
         */
        mLayout.onMeasure(mRecycler, mState, widthSpec, heightSpec);

        // 如果宽高都指定了尺寸，则不再测量
        final boolean measureSpecModeIsExactly =
            widthMode == MeasureSpec.EXACTLY && heightMode == MeasureSpec.EXACTLY;
        if (measureSpecModeIsExactly || mAdapter == null)
        {
            return;
        }

        // 在onMeasure和onLayout过程中，有三个状态，代表测量子视图的三个阶段
        // 分别是：
        // STEP_START：
        //     对应dispatchLayoutStep1函数，这个函数的功能是：
        //     1. 处理Adapter更新
        //     2. 更新动画信息
        //     3. 记录数据集改变前的子控件信息
        // STEP_LAYOUT:
        //     对应dispatchLayoutStep2函数，这个函数的功能是：
        //     实际执行子视图的onMeasure和onLayout，调用
        //     LayoutManager::onLayoutChildren这个抽象函数；
        // STEP_ANIMATIONS：
        //     对应dispatchLayoutStep3函数，这个函数的功能是：
        //     记录数据集改变后的子控件信息及触发动画，并且清除一些记录信息；
        if (mState.mLayoutStep == State.STEP_START)
        {
            dispatchLayoutStep1();
        }
        // set dimensions in 2nd step. Pre-layout should happen with old dimensions for
        // consistency
        mLayout.setMeasureSpecs(widthSpec, heightSpec);
        mState.mIsMeasuring = true;
        dispatchLayoutStep2();

        // 调用过dispatchLayoutStep2函数后，子视图的宽高都有了，
        // 根据子视图的宽高再次更新父视图宽高
        mLayout.setMeasuredDimensionFromChildren(widthSpec, heightSpec);

        // if RecyclerView has non-exact width and height and if there is at least one child
        // which also has non-exact width & height, we have to re-measure.
        if (mLayout.shouldMeasureTwice())
        {
            mLayout.setMeasureSpecs(
                MeasureSpec.makeMeasureSpec(getMeasuredWidth(), MeasureSpec.EXACTLY),
                MeasureSpec.makeMeasureSpec(getMeasuredHeight(), MeasureSpec.EXACTLY));
            mState.mIsMeasuring = true;
            dispatchLayoutStep2();
            // now we can get the width and height from the children.
            mLayout.setMeasuredDimensionFromChildren(widthSpec, heightSpec);
        }
    }
    else
    {
        // Adapter数据的变化是否会使得RecyclerView的宽高发生变化
        if (mHasFixedSize)
        {
            mLayout.onMeasure(mRecycler, mState, widthSpec, heightSpec);
            return;
        }
        // custom onMeasure
        if (mAdapterUpdateDuringMeasure)
        {
            startInterceptRequestLayout();
            onEnterLayoutOrScroll();
            processAdapterUpdatesAndSetAnimationFlags();
            onExitLayoutOrScroll();

            if (mState.mRunPredictiveAnimations)
            {
                mState.mInPreLayout = true;
            }
            else
            {
                // consume remaining updates to provide a consistent state with the layout pass.
                mAdapterHelper.consumeUpdatesInOnePass();
                mState.mInPreLayout = false;
            }
            mAdapterUpdateDuringMeasure = false;
            stopInterceptRequestLayout(false);
        }
        else if (mState.mRunPredictiveAnimations)
        {
            // If mAdapterUpdateDuringMeasure is false and mRunPredictiveAnimations is true:
            // this means there is already an onMeasure() call performed to handle the pending
            // adapter change, two onMeasure() calls can happen if RV is a child of LinearLayout
            // with layout_width=MATCH_PARENT. RV cannot call LM.onMeasure() second time
            // because getViewForPosition() will crash when LM uses a child to measure.
            setMeasuredDimension(getMeasuredWidth(), getMeasuredHeight());
            return;
        }

        if (mAdapter != null)
        {
            mState.mItemCount = mAdapter.getItemCount();
        }
        else
        {
            mState.mItemCount = 0;
        }
        startInterceptRequestLayout();
        mLayout.onMeasure(mRecycler, mState, widthSpec, heightSpec);
        stopInterceptRequestLayout(false);
        mState.mInPreLayout = false; // clear
    }
}
```



### onLayout

`onLayout`实际上就是调用了`dispatchLayout`函数；

```java
void dispatchLayout() {
        if (mAdapter == null) {
            Log.e(TAG, "No adapter attached; skipping layout");
            // leave the state in START
            return;
        }
        if (mLayout == null) {
            Log.e(TAG, "No layout manager attached; skipping layout");
            // leave the state in START
            return;
        }
        mState.mIsMeasuring = false;
        if (mState.mLayoutStep == State.STEP_START) {
            dispatchLayoutStep1();
            mLayout.setExactMeasureSpecsFrom(this);
            dispatchLayoutStep2();
        } else if (mAdapterHelper.hasUpdates() || mLayout.getWidth() != getWidth()
                || mLayout.getHeight() != getHeight()) {
            // First 2 steps are done in onMeasure but looks like we have to run again due to
            // changed size.
            mLayout.setExactMeasureSpecsFrom(this);
            dispatchLayoutStep2();
        } else {
            // always make sure we sync them (to ensure mode is exact)
            mLayout.setExactMeasureSpecsFrom(this);
        }
        dispatchLayoutStep3();
    }
```

主要还是调用`dispatchLayoutStep3`函数，具体功能在上面已经解释过了；

### onDraw

```java
public void onDraw(Canvas c) {
        // 通过上述操作，所有子视图的位置和宽高都已知晓，所以子视图的绘制交给了父类；
        super.onDraw(c);

		// 绘制子视图装饰器
        final int count = mItemDecorations.size();
        for (int i = 0; i < count; i++) {
            mItemDecorations.get(i).onDraw(c, this, mState);
        }
}
```

## 事件分发

关于事件分发，`NestedScrollingChild`，`NestedScrollingParent`之间的关系极为重要；

这两个接口事关嵌套滚动；

### ScrollingView

```java
// 这个接口主要是提供了关于滚动条的一些API
// 这些API的返回值都是同一个单位
// 滚动条分为两个部分：滚动长条和滚动短条
public interface ScrollingView {
    /**
     *  水平滚动条的范围
     *  滚动长条的逻辑长度
     *  默认值是视图的宽度
     */
    int computeHorizontalScrollRange();

    /**
     *  滚动短条的起始位置到滚动长条的起始位置的距离，X轴
     */
    int computeHorizontalScrollOffset();

    /**
     * 滚动短条的逻辑长度
     * 默认值是视图宽度
     */
    int computeHorizontalScrollExtent();

    /**
     *  垂直滚动条的范围
     *  滚动长条的逻辑长度
     *  默认值是视图的高度
     */
    int computeVerticalScrollRange();

    /**
     *  滚动短条的起始位置到滚动长条的起始位置的距离，Y轴
     */
    int computeVerticalScrollOffset();

    /**
     * 滚动短条的逻辑长度
     * 默认值是视图高度
     */
    int computeVerticalScrollExtent();
}

```




## Adapter详解

## 缓存机制

## 其他问题

### ItemDecoration

### ItemAnimator

### 滑动删除和长按拖拽

### 添加Header和Footer

### 下拉刷新和上滑刷新

## 各种优化

## Viewpager2的实现

