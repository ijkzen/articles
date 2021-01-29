---
title: RecyclerView详解
categories: Android原理
---

## 概论

`RecyclerView`是当前使用最广泛的列表视图组件；

`RecyclerView`直接继承了`ViewGroup`，同时实现了`ScrollingView`，`NestScrollingChild2`，`NestScrollingChild3`接口；

既然继承了`ViewGroup`，那就一定有三大流程，详细的在[这里](##绘制过程)；

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



`NestScrolling`接口与事件分发有关，请到[这里]((##事件分发))；

## 绘制过程

`ChildView`的实际绘制过程由`LayoutManager`所掌控；

### onMeasure

```java
@Override
protected void onMeasure(int widthSpec, int heightSpec)
{
    // 没有自定义LayoutManager，设置默认的宽高，并不测量CHild View的宽高；
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
    
    if (mLayout.isAutoMeasureEnabled())
    {
        final int widthMode = MeasureSpec.getMode(widthSpec);
        final int heightMode = MeasureSpec.getMode(heightSpec);

        /**
         * This specific call should be considered deprecated and replaced with
         * {@link #defaultOnMeasure(int, int)}. It can't actually be replaced as it could
         * break existing third party code but all documentation directs developers to not
         * override {@link LayoutManager#onMeasure(int, int)} when
         * {@link LayoutManager#isAutoMeasureEnabled()} returns true.
         */
        mLayout.onMeasure(mRecycler, mState, widthSpec, heightSpec);

        final boolean measureSpecModeIsExactly =
            widthMode == MeasureSpec.EXACTLY && heightMode == MeasureSpec.EXACTLY;
        if (measureSpecModeIsExactly || mAdapter == null)
        {
            return;
        }

        if (mState.mLayoutStep == State.STEP_START)
        {
            dispatchLayoutStep1();
        }
        // set dimensions in 2nd step. Pre-layout should happen with old dimensions for
        // consistency
        mLayout.setMeasureSpecs(widthSpec, heightSpec);
        mState.mIsMeasuring = true;
        dispatchLayoutStep2();

        // now we can get the width and height from the children.
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

### onDraw

## 事件分发

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

