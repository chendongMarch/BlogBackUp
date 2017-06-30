---
layout: post
title: NestedScrollingParent分析
category: Android
tags:
  - Android
  - Android5.+
description: NestedScrollingParent分析
abbrlink: 1655050953
date: 2016-08-22 00:00:00
keywords:
---


## 前言
- 对NestedScrollingParent接口进行分析，主要是文档的翻译和一些理解，以及每个方法触发的时机。

## NestedScrollingParent接口

- This interface should be implemented by {@link android.view.ViewGroup ViewGroup} subclasses
that wish to support scrolling operations delegated by a nested child view.

- 翻译：这个接口被ViewGroup的子类实现来支持处理nest子视图的滑动操作。

- Classes implementing this interface should create a final instance of a {@link NestedScrollingParentHelper} as a field and delegate any View or ViewGroup methods
to the <code>NestedScrollingParentHelper</code> methods of the same signature.

- 翻译：实现这个接口的类应该创建一个final类型的NestedScrollingParentHelper实例作为一个属性，来代理NestedScrollingParent中与NestedScrollingParentHelper具有相同方法名字的操作。

- Views invoking nested scrolling functionality should always do so from the relevant {@link ViewCompat}, {@link ViewGroupCompat} or {@link ViewParentCompat} compatibility shim static methods. This ensures interoperability with nested scrolling views on Android 5.0 Lollipop and newer.

- 翻译：View调用嵌套滑动的相关功能时应该从相关的ViewCompat，ViewGroupCompat，ViewParentCompat调用兼容的静态方法，这保证了在5.0或者更新版本中与嵌套滑动的互用性。



## onStartNestedScroll方法
```java
public boolean onStartNestedScroll(View child, View target, int nestedScrollAxes);
```

- 参数和返回值

```java
//ViewParent包涵的直接的子View
@param child Direct child of this ViewParent containing target 
//启动嵌套滑动操作的View
@param target View that initiated the nested scroll 
//flag垂直滑动还是水平滑动
@param nestedScrollAxes Flags consisting of {@link ViewCompat#SCROLL_AXIS_HORIZONTAL},                       {@link ViewCompat#SCROLL_AXIS_VERTICAL} or both 
//如果返回true代表父View接受消耗这个滑动操作。
@return true if this ViewParent accepts the nested scroll operation
```

- React to a descendant view initiating a nestable scroll operation, claiming the nested scroll operation if appropriate.

- 翻译：对一个派生View的嵌套滑动操作作出反应，如果需要的话将会拦截（消耗）这个滑动操作。

- This method will be called in response to a descendant view invoking ink ViewCompat#startNestedScroll(View, int)}. Each parent up the view hierarchy will be given an opportunity to respond and claim the nested scrolling operation by returning <code>true</code>.

- 翻译：这个方法将会被作为派生View调用ViewCompat.startNestedScroll(View, int)时的回复响应，每个上层的父视图如果这个方法返回true,都将有机会响应和拦截（消耗）这个嵌套滑动操作.


     
## onNestedScrollAccepted方法
```java
public void onNestedScrollAccepted(View child, View target, int nestedScrollAxes);
```

-  React to the successful claiming of a nested scroll operation.

- 翻译：对嵌套滑动操作的成功声明作出响应，

- This method will be called after {@link #onStartNestedScroll(View, View, int) onStartNestedScroll} returns true. It offers an opportunity for the view and its superclasses to perform initial configuration for the nested scroll. Implementations of this method should always call their superclass's implementation of this method if one is present.

- 翻译：这个方法将会在 `onStartNestedScroll `方法返回`true`时被调用。这个方法提供给View和他的父类一个对嵌套滑动操作进行初始化配置的机会。如果父类对该方法有操作，实现此方法应该调用父类的实现。


## onStopNestedScroll方法
```java
public void onStopNestedScroll(View target);
```

- 参数和返回值

```java
//触发嵌套滑动的View
@param target View that initiated the nested scroll
```

- React to a nested scroll operation ending.

- 翻译：对嵌套滑动结束做出响应

- Perform cleanup after a nested scrolling operation.This method will be called when a nested scroll stops, for example when a nested touch scroll ends with a {@link MotionEvent#ACTION_UP} or {@link MotionEvent#ACTION_CANCEL} event. Implementations of this method should always call their superclass's implementation of this method if one is present.

- 翻译：确认在嵌套滑动操作结束之后的清理操作，比如一些参数的重新初始化，这个方法将会在嵌套滑动操作结束后调用，比如嵌套滑动操作因为ACTION_UP，ACTION_CANCEL事件被终止。如果父类对该方法有操作，实现此方法应该调用父类的实现。



## onNestedScroll方法
```java
public void onNestedScroll(View target, int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed);
```


- 参数和返回值

```java
// 控制嵌套滑动的子View
@param target The descendent view controlling the nested scroll
// 子View水平滑动消耗的距离
@param dxConsumed Horizontal scroll distance in pixels already consumed by target
//  子View垂直滑动消耗的距离
@param dyConsumed Vertical scroll distance in pixels already consumed by target
// 子View水平滑动没有消耗的距离
@param dxUnconsumed Horizontal scroll distance in pixels not consumed by target
// 子View水平滑动没有消耗的距离
@param dyUnconsumed Vertical scroll distance in pixels not consumed by target
```


-  React to a nested scroll in progress.

- 翻译：对滑动过程作出响应

- This method will be called when the ViewParent's current nested scrolling child view dispatches a nested scroll event. To receive calls to this method the ViewParent must have previously returned <code>true</code> for a call to {@link #onStartNestedScroll(View, View, int)}.

- 翻译：这个方法将会在ViewParent中当前正在嵌套滑动的子View分发嵌套滑动事件时被调用。为了能够接受到调用这个方法，ViewParent必须事先在`onStartNestedScroll`返回true。

- Both the consumed and unconsumed portions of the scroll distance are reported to the ViewParent. An implementation may choose to use the consumed portion to match or chase scroll position of multiple child elements, for example. The unconsumed portion may be used to allow continuous dragging of multiple scrolling or draggable elements, such as scrolling a list within a vertical drawer where the drawer begins dragging once the edge of inner scrolling content is reached.

- 翻译：消耗部分和没有消耗部分的嵌套滑动的距离都被上报给ViewParent。一个实现可以选择使用消耗的部分来匹配或追逐多个子元素的滚动位置，例如。
未使用的部分可以用来允许连续滚动或拖动拖动多元素，如在一个垂直的抽屉内滚动一个列表，抽屉开始拖动滚动达到内容的边缘。


## onNestedPreScroll方法
```java
public void onNestedPreScroll(View target, int dx, int dy, int[] consumed);
```

-  参数和返回值

```java
// 发起嵌套滑动的子View
@param target View that initiated the nested scroll
// 水平滑动的距离
@param dx Horizontal scroll distance in pixels
// 垂直滑动的距离
@param dy Vertical scroll distance in pixels
// 输出,水平滑动和垂直滑动被父View消耗的距离
@param consumed Output. The horizontal and vertical scroll distance consumed by this parent
```

- React to a nested scroll in progress before the target view consumes a portion of the scroll.

- 滑动过程中在发起嵌套滑动的子View消耗滑动距离之前作出反应

- When working with nested scrolling often the parent view may want an opportunity to consume the scroll before the nested scrolling child does. An example of this is a drawer that contains a scrollable list. The user will want to be able to scroll the list fully into view before the list itself begins scrolling.

- 在进行嵌套滑动时，通常父View希望获得机会在子View之前消耗滑动距离。例如，抽屉里面包含一个可以滑动的list,用户将希望能够在列表本身开始滚动之前将列表完全滚动到视图中。

- <code>onNestedPreScroll</code> is called when a nested scrolling child invokes {@link View#dispatchNestedPreScroll(int, int, int[], int[])}. The implementation should report how any pixels of the scroll reported by dx, dy were consumed in the <code>consumed</code> array. Index 0 corresponds to dx and index 1 corresponds to dy. This parameter will never be null. Initial values for consumed[0] and consumed[1] will always be 0.

- `onNestedPreScroll` 方法在子View调用`dispatchNestedPreScroll `方法时被调用。这个方法的实现应该报告对dx和dy进行了多少消耗。consumed[0]代表消耗的dx,consumed[1]代表消耗的dy,这个参数不会为空，consumed[0]和consumed[1]初始值为0。

## onNestedFling方法
```java
public boolean onNestedFling(View target, float velocityX, float velocityY, boolean consumed);
```

- 参数和返回值

```java
// 发起嵌套滑动的子View
@param target View that initiated the nested scroll
// 横向滑动的速度
@param velocityX Horizontal velocity in pixels per second
// 竖向滑动的速度
@param velocityY Vertical velocity in pixels per second
// 如果是true表示这个子View消耗了这个滑动
@param consumed true if the child consumed the fling, false otherwise
// 返回true表示，父View消耗滑动或其他情况
@return true if this parent consumed or otherwise reacted to the fling
```

- Request a fling from a nested scroll.

- 需要嵌套滑动发生fling事件

- This method signifies that a nested scrolling child has detected suitable conditions for a fling. Generally this means that a touch scroll has ended with a  {@link VelocityTracker velocity} in the direction of scrolling that meets or exceeds  the {@link ViewConfiguration#getScaledMinimumFlingVelocity() minimum fling velocity} along a scrollable axis.

- 这个方法意味着嵌套滑动的子View监测到一个适合fling的情况。通常意味着触摸事件在一个可滑动的轴上在一个方向上以一定速度结束，这个速度超过了` ViewConfiguration#getScaledMinimumFlingVelocity()`的值。

- If a nested scrolling child view would normally fling but it is at the edge of its own content, it can use this method to delegate the fling to its nested scrolling parent instead. The parent may optionally consume the fling or observe a child fling.

- 如果一个子View在嵌套滑动但是到达了他的内容边缘，它可以使用这个方法将处理fling事件代理给父View,父View可以选择性的消耗fling事件或者观察子View的fling事件。


## onNestedPreFling方法
```java
public boolean onNestedPreFling(View target, float velocityX, float velocityY);
```

- 参数和返回值
```java
// 触发嵌套滑动的子View
@param target View that initiated the nested scroll
// 水平方向滑动速度
@param velocityX Horizontal velocity in pixels per second
// 垂直方向滑动速度
@param velocityY Vertical velocity in pixels per second
// 如果父View要处理这个fling事件返回true
@return true if this parent consumed the fling ahead of the target view
```

- React to a nested fling before the target view consumes it.

- 在子View对嵌套fling事件消耗之前作出响应

- This method siginfies that a nested scrolling child has detected a fling with the given velocity along each axis. Generally this means that a touch scroll has ended with a {@link VelocityTracker velocity} in the direction of scrolling that meets or exceeds  the {@link ViewConfiguration#getScaledMinimumFlingVelocity() minimum fling velocity} along a scrollable axis.

- 这个方法意味着嵌套滑动的子View在每个轴上给定的速度上发生了fling事件，通常意味着触摸事件在一个可滑动的轴上在一个方向上以一定速度结束，这个速度超过了` ViewConfiguration#getScaledMinimumFlingVelocity()`的值。

- If a nested scrolling parent is consuming motion as part of a {@link #onNestedPreScroll(View, int, int, int[]) pre-scroll}, it may be appropriate for  it to also consume the pre-fling to complete that same motion. By returning  <code>true</code> from this method, the parent indicates that the child should not fling its own internal content as well.

- 如果父View在`onNestedPreScroll `方法中消耗了嵌套滑动，那么他可能同样也想在fling事件中进行消耗操作，来完成处理嵌套滑动的请求。这个方法返回`true`表明子View应该只在它自己的区域内滑动。

##  getNestedScrollAxes()方法
```
public int getNestedScrollAxes();
```
-  参数和返回值

```java
// 表明滑动轴方向的flag
@return Flags indicating the current axes of nested scrolling
@see ViewCompat#SCROLL_AXIS_HORIZONTAL
@see ViewCompat#SCROLL_AXIS_VERTICAL
@see ViewCompat#SCROLL_AXIS_NONE
```


- A NestedScrollingParent returning something other than {@link ViewCompat#SCROLL_AXIS_NONE}］is currently acting as a nested scrolling parent for one or more descendant views in the hierarchy.

- 如果没有返回`ViewCompat#SCROLL_AXIS_NONE`，表明有子View在进行嵌套滑动。