---
layout: post
title: 仿QQ相册RecyclerView滑动选中
date: 2016-09-06
category: Android
tags: [Android]
keywords: 
---


## 1. 前言
> 显示相册在app中是一个比较常见的操作，大致的操作就是通过ContentProvider获取多媒体资源进行展示，我综合了一下QQ 的和微信的显示效果，实现了一下,[仿微信QQ显示手机相册](http://chendongmarch.github.io/2016/11/02/Android%E5%BC%80%E5%8F%91/%E9%AB%98%E4%BB%BFQQ%E5%BE%AE%E4%BF%A1%E7%9B%B8%E5%86%8C/)，在QQ的相册选择时是支持滑动选中的，即手指碰到哪个就选中哪张照片，正好公司的项目中用到了这个功能，在网上找了找没有很好的解决方案，所以通过自定义控件处理事件，这篇文章主要介绍这个功能的实现。 

> 自定义控件SlidingSelectLayout的源代码[点击这里获取](https://github.com/chendongMarch/CommonLib/blob/master/lib-dev/src/main/java/com/march/dev/widget/SlidingSelectLayout.java)

<!--more-->

## 2 效果演示
[普通模式演示视频](http://7xtjec.com1.z0.glb.clouddn.com/gallery.mp4)

[九宫格模式演示视频](http://7xtjec.com1.z0.glb.clouddn.com/item_header_sliding_select.mp4)

<img src="http://7xtjec.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-09-06%2023.27.04.png"
width=50% > 



## 3. 使用 
### 3.1 xml中使用

```xml
<com.march.slidingselect.SlidingSelectLayout
        android:id="@+id/scl"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
        <android.support.v7.widget.RecyclerView
            android:id="@+id/recyclerview"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />
            
</com.march.slidingselect.SlidingSelectLayout>
```

### 3.2 java代码中配置
在Adapter的onBindViewHolder方法中将pos和data与view进行绑定，之所以这样做是为了可以在手指划过Item时将对应的数据和位置通过监听发送回来。

```java
private SlidingSelectLayout mScl;
mScl = getView(R.id.scl);

class MyAdapter extends RecyclerView.Adapter<RecyclerView.ViewHolder>{
    @Override
    public void onBindViewHolder(RecyclerView.ViewHolder holder, int position) {
    	mScl.markView(holder.itemView,position,demos.get(position));
    }
}
```

### 3.3 监听回调
使用范型获得手指触摸到的位置和当前位置对应的数据进行更新adapter

```java
mScl.setOnSlidingSelectListener(new SlidingSelectLayout.OnSlidingSelectListener<Demo>() {
            @Override
            public void onSlidingSelect(int pos, View parentView, Demo data) {
                demos.get(pos).isChanged = !demos.get(pos).isChanged;
                adapter.notifyItemChanged(pos);
            }
        });
```
 


## 4. 大体思路
自定义控件将会作为`RecyclerView`的父控件，这样使他可以优先于`RecyclerView`捕捉事件  

当手指竖向滑动时，父控件不作处理，`RecyclerView`处理事件，进行滑动。当手指横向滑动达到阈值时自定义父控件会截断事件自己进行进行处理。

根据手指的滑动`MotionEvent`获取x,y坐标，使用`RecycelrView`的`findViewUnder(float x,float y)` 的方法，可以直接获取制定位置的View，再使用tag从view中拿到之前使用`mScl.markView()`方法绑定的pos和data数据

使用该方法就不会因为动态计算距离而局限于RecyclerView的布局，九宫格模式下仍然可以很好的支持。



## 5. 内部处理对外封闭
在处理事件获取手指滑动的位置时，需要使用`RecyclerView`的`LayoutManager`等，为了尽量对外简化使用的流程，在控件内部使用遍历子控件的方式来获取`RecyclerView`和GridLayoutManager的列数等参数，初始化一些值。
 
### 5.1 获取RecyclerView

```java
/**
 * 获取RecyclerView
 */
private void ensureTarget() {
    if (mTargetRv != null)
        return;
    View findView = searchInViewGroup(this);
    if (findView == null) {
        Logger.e(TAG, "can not find RecyclerView");
    } else {
        mTargetRv = (RecyclerView) findView;
    }
}

private View searchInViewGroup(ViewGroup viewGroup) {
    View rstView = null;
    for (int i = 0; i < viewGroup.getChildCount(); i++) {
        View childAt = viewGroup.getChildAt(i);
        if (childAt instanceof RecyclerView) {
            rstView = childAt;
        } else if (childAt instanceof ViewGroup) {
            rstView = searchInViewGroup((ViewGroup) childAt);
        }
    }
    return rstView;
}
```

### 5.2 初始化参数
处理LayoutManager，初始化xTouchSlop，这个值是滑动多大距离触发水平滑动，根据GridLayoutManager的列数来动态设置。

```java
private void ensureLayoutManager() {
    if (mTargetRv == null || itemSpanCount != INVALID_PARAM)
        return;
    RecyclerView.LayoutManager lm = mTargetRv.getLayoutManager();
    if (lm == null)
        return;
    if (lm instanceof GridLayoutManager) {
        GridLayoutManager glm = (GridLayoutManager) lm;
        itemSpanCount = glm.getSpanCount();
    } else {
        Logger.e(TAG,"暂时不支持其他布局类型，请使用GridLayoutManager");
        itemSpanCount = 4;
    }
    int size = (int) (getResources().getDisplayMetrics().widthPixels / (itemSpanCount * 1.0f));
    xTouchSlop = yTouchSlop = size * TOUCH_SLOP_RATE;
}
```

## 6. 拦截事件

```java
// 如果RecyclerView没有获取到，不进行事件的拦截
private boolean isReadyToIntercept() {
    return mTargetRv != null 
    && mTargetRv.getAdapter() != null 
    && itemSpanCount != INVALID_PARAM;
}



@Override
public boolean onInterceptTouchEvent(MotionEvent ev) {
    if (!isEnabled())
        return super.onInterceptTouchEvent(ev);
    // 不支持多点触摸
    int pointerCount = ev.getPointerCount();
    if (pointerCount > 1) {
        return super.onInterceptTouchEvent(ev);
    }
    // 获取RecyclerView
    ensureTarget();
    // 初始化参数
    ensureLayoutManager();
    if (!isReadyToIntercept())
        return super.onInterceptTouchEvent(ev);
    int action = MotionEventCompat.getActionMasked(ev);
    switch (action) {
        case MotionEvent.ACTION_DOWN:
            // init
            mInitialDownX = ev.getX();
            mInitialDownY = ev.getY();
            break;
        case MotionEvent.ACTION_UP:
            // stop
            isBeingSlide = false;
            break;
        case MotionEvent.ACTION_MOVE:
            // handle
            float xDiff = Math.abs(ev.getX() - mInitialDownX);
            float yDiff = Math.abs(ev.getY() - mInitialDownY);
            if (yDiff < yTouchSlop && xDiff > xTouchSlop) {
                isBeingSlide = true;
            }
            break;
    }
    return isBeingSlide;
}
```

## 7. 触摸事件
### 7.1 处理Touch事件
```java
@Override
public boolean onTouchEvent(MotionEvent ev) {
    int action = MotionEventCompat.getActionMasked(ev);
    switch (action) {
        case MotionEvent.ACTION_DOWN:
            break;
        case MotionEvent.ACTION_UP:
            // re init 手指抬起时重置
            isBeingSlide = false;
            preViewPos = INVALID_PARAM;
            break;
        case MotionEvent.ACTION_MOVE:
            // 滑动过程中，触发监听事件
            publishSlidingCheck(ev);
            break;
    }
    return isBeingSlide;
}
```

### 7.2 获取数据

```java
// 从View中使用getTag(int key)获取tag，也就是之前在onBindViewHolder中设置的数据
// 从Tag中获取pos
private int getPos(View parentView) {
        int pos = INVALID_PARAM;
        Object tag = parentView.getTag(tagPosKey);
        if (tag != null)
            pos = (int) tag;
        return pos;
    }
    
    
// 从tag中获取data
private Object getData(View parentView) {
        return parentView.getTag(tagDataKey);
}
```
 
 
### 7.3 触发监听
使用监听向外发布事件,将获取的pos和data通过监听发布

```java
/**
 * 发布结果
 *
 * @param event 事件
 */
private void publishSlidingCheck(MotionEvent event) {
    float x = generateX(event.getX());
    float y = generateY(event.getY());
    View childViewUnder = mTargetRv.findChildViewUnder(x, y);
    // fast stop
    if (onSlidingSelectListener == null || childViewUnder == null)
        return;
    int pos = getPos(childViewUnder);
    Object data = getData(childViewUnder);
    // fast stop 当前触摸的点与上一次触摸的点相同 || 没有pos || 没有数据
    if (preViewPos == pos || pos == INVALID_PARAM || data == null)
        return;
    try {
        // 这里使用范型强制转换
        onSlidingSelectListener.onSlidingSelect(pos, childViewUnder, data);
        preViewPos = pos;
    } catch (ClassCastException e) {
        Log.e("SlidingSelect", "ClassCastException:填写的范型有误，无法转换");
    }
}
```


## 8 源代码
```java
package com.march.dev.widget;

import android.content.Context;
import android.support.v4.view.MotionEventCompat;
import android.support.v7.widget.GridLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.util.AttributeSet;
import android.util.Log;
import android.view.MotionEvent;
import android.view.View;
import android.view.ViewGroup;
import android.widget.FrameLayout;

import com.march.dev.R;
import com.march.dev.helper.Logger;

/**
 * Project  : CommonLib
 * Package  : com.march.lib.view
 * CreateAt : 2016/9/12
 * Describe : 滑动选中
 *
 * @author chendong
 */
public class SlidingSelectLayout extends FrameLayout {

    public static final  String TAG             = SlidingSelectLayout.class.getSimpleName();
    private static final float  TOUCH_SLOP_RATE = 0.15f;// 初始化值
    private static final int    INVALID_PARAM   = -1;

    public SlidingSelectLayout(Context context) {
        this(context, null);
    }

    public SlidingSelectLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
        setTagKey(R.id.sliding_pos, R.id.sliding_data);
        itemSpanCount = INVALID_PARAM;
        preViewPos = INVALID_PARAM;
    }

    private RecyclerView mTargetRv;// 内部的rv
    private int          offsetTop;
    private float        xTouchSlop;// 横轴滑动阈值，超过阈值表示触发横轴滑动
    private float        yTouchSlop;// 纵轴滑动阈值，超过阈值表示触发纵轴滑动
    private int          itemSpanCount;// 横向的item数量
    private float        mInitialDownX;// down 事件初始值
    private float        mInitialDownY;// down 事件初始值
    private boolean      isBeingSlide;// 是否正在滑动
    private int          tagPosKey;
    private int          tagDataKey;
    private int          preViewPos;
    
    private OnSlidingSelectListener onSlidingSelectListener;// 滑动选中监听


    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        if (!isEnabled())
            return super.onInterceptTouchEvent(ev);
        // 不支持多点触摸
        int pointerCount = ev.getPointerCount();
        if (pointerCount > 1) {
            return super.onInterceptTouchEvent(ev);
        }
        // 获取RecyclerView
        ensureTarget();
        // 初始化参数
        ensureLayoutManager();
        if (!isReadyToIntercept())
            return super.onInterceptTouchEvent(ev);
        int action = MotionEventCompat.getActionMasked(ev);
        switch (action) {
            case MotionEvent.ACTION_DOWN:
                // init
                mInitialDownX = ev.getX();
                mInitialDownY = ev.getY();
                break;
            case MotionEvent.ACTION_UP:
                // stop
                isBeingSlide = false;
                break;
            case MotionEvent.ACTION_MOVE:
                // handle
                float xDiff = Math.abs(ev.getX() - mInitialDownX);
                float yDiff = Math.abs(ev.getY() - mInitialDownY);
                if (yDiff < yTouchSlop && xDiff > xTouchSlop) {
                    isBeingSlide = true;
                }
                break;
        }
        return isBeingSlide;
    }
    
    private float generateX(float x) {
        return x;
    }

    private float generateY(float y) {
        return y - offsetTop;
    }

    private void setTargetRv(RecyclerView mTargetRv) {
        this.mTargetRv = mTargetRv;
    }

    @Override
    public boolean onTouchEvent(MotionEvent ev) {
        int action = MotionEventCompat.getActionMasked(ev);
        switch (action) {
            case MotionEvent.ACTION_DOWN:
                break;
            case MotionEvent.ACTION_UP:
                // re init 手指抬起时重置
                isBeingSlide = false;
                preViewPos = INVALID_PARAM;
                break;
            case MotionEvent.ACTION_MOVE:
                // 滑动过程中，触发监听事件
                publishSlidingCheck(ev);
                break;
        }
        return isBeingSlide;
    }

    /**
     * 初始化参数
     */
    private void ensureLayoutManager() {
        if (mTargetRv == null || itemSpanCount != INVALID_PARAM)
            return;
        RecyclerView.LayoutManager lm = mTargetRv.getLayoutManager();
        if (lm == null)
            return;
        if (lm instanceof GridLayoutManager) {
            GridLayoutManager glm = (GridLayoutManager) lm;
            itemSpanCount = glm.getSpanCount();
        } else {
            Logger.e(TAG, "暂时不支持其他布局类型，请使用GridLayoutManager");
            itemSpanCount = 4;
        }
        int size = (int) (getResources().getDisplayMetrics().widthPixels / (itemSpanCount * 1.0f));
        xTouchSlop = yTouchSlop = size * TOUCH_SLOP_RATE;
    }


    /**
     * 发布结果
     *
     * @param event 事件
     */
    private void publishSlidingCheck(MotionEvent event) {
        float x = generateX(event.getX());
        float y = generateY(event.getY());
        View childViewUnder = mTargetRv.findChildViewUnder(x, y);
        // fast stop
        if (onSlidingSelectListener == null || childViewUnder == null)
            return;
        int pos = getPos(childViewUnder);
        Object data = getData(childViewUnder);
        // fast stop 当前触摸的点与上一次触摸的点相同 || 没有pos || 没有数据
        if (preViewPos == pos || pos == INVALID_PARAM || data == null)
            return;
        try {
            // 这里使用范型强制转换
            onSlidingSelectListener.onSlidingSelect(pos, childViewUnder, data);
            preViewPos = pos;
        } catch (ClassCastException e) {
            Log.e("SlidingSelect", "ClassCastException:填写的范型有误，无法转换");
        }
    }

    private void setTagKey(int tagPosKey, int tagDataKey) {
        this.tagPosKey = tagPosKey;
        this.tagDataKey = tagDataKey;
    }

    /**
     * 设置pos和data作为View的tag
     * @param parentView
     * @param pos
     * @param data
     */
    public void markView(View parentView, int pos, Object data) {
        parentView.setTag(tagPosKey, pos);
        parentView.setTag(tagDataKey, data);
    }

    private int getPos(View parentView) {
        int pos = INVALID_PARAM;
        Object tag = parentView.getTag(tagPosKey);
        if (tag != null)
            pos = (int) tag;
        return pos;
    }

    private Object getData(View parentView) {
        return parentView.getTag(tagDataKey);
    }

    /**
     * 是否可以开始拦截处理事件，当recyclerView数据完全ok之后开始
     *
     * @return 是否可以开始拦截处理事件
     */

    private boolean isReadyToIntercept() {
        return mTargetRv != null
                && mTargetRv.getAdapter() != null
                && itemSpanCount != INVALID_PARAM;
    }

    /**
     * 获取RecyclerView
     */
    private void ensureTarget() {
        if (mTargetRv != null)
            return;
        View findView = searchInViewGroup(this);
        if (findView == null) {
            Logger.e(TAG, "can not find RecyclerView");
        } else {
            mTargetRv = (RecyclerView) findView;
        }
    }

    private View searchInViewGroup(ViewGroup viewGroup) {
        View rstView = null;
        for (int i = 0; i < viewGroup.getChildCount(); i++) {
            View childAt = viewGroup.getChildAt(i);
            if (childAt instanceof RecyclerView) {
                rstView = childAt;
            } else if (childAt instanceof ViewGroup) {
                rstView = searchInViewGroup((ViewGroup) childAt);
            }
        }
        return rstView;
    }

    public void setOffsetTop(int offsetTop) {
        this.offsetTop = offsetTop;
    }

    public <D> void setOnSlidingSelectListener(OnSlidingSelectListener<D> onSlidingCheckListener) {
        this.onSlidingSelectListener = onSlidingCheckListener;
    }

    public interface OnSlidingSelectListener<D> {
        void onSlidingSelect(int pos, View parentView, D data);
    }
}
```