---
layout: post
title: Android事件交互
date: 2016-08-25
category: Android
tags: Android
keywords: 
description: Android事件交互
---


## 前言
- Android事件交互,当用户手指触碰屏幕时会触发`onTouchEvent(MotionEven event)`方法。


## 推荐阅读
[简书－MotionEvent详解](http://www.jianshu.com/p/0c863bbde8eb)

## 常用事件
```java
MotionEvent.ACTION_DOWN:  0
MotionEvent.ACTION_UP:  1
MotionEvent.ACTION_MOVE:  2
MotionEvent.ACTION_CANCEL:  3
MotionEvent.ACTION_OUTSIDE:  4
MotionEvent.ACTION_POINTER_DOWN:  5
MotionEvent.ACTION_POINTER_UP:  6
```

## 获取事件
- `event.getAction()`和`event.getActionMasked()`和`MotionEventCompat.getActionMasked(event)`都可以获取事件类型。带有Mask标志的方法是针对多点触控的情况拿到的掩码。多点触控时，获取Action会返回每个触摸点的事件，其中使用pointerIndex区分每个触摸点。

- 为了能表示每个pointerIndex对应的事件，以及更方便转化，每个获取的action实际上是pointerIndex和event拼接成的16进制表示，比如第二个点的Move事件，返回的值是`0x0100`,01表示pointerIndex,00表示Down事件，所以返回的action的值为256.

- 单点触摸时只有一个pointerIndex = 0,所以获取的到的action实际上是0x0000(Down事件)，0x0001(UP事件)。。。所以结果和事件的值是一样的。

- 当多点触摸时，第一个点的Down事件0x0000,第二个点的Down事件0x0100,此时简单的`getAction()`方法并不能获取正确的事件值。

- `getActionMasked()`方法是将`getAction()`的值与`ACTION_MASK`做‘与’运算，例如当有一个事件是0x0102,获取掩码时可以过滤掉pointerIndex的值获取event的值，`event.getActionMasked()`原理大致相同。

|描述|值|
|:---|:---|
|getAction()十六进制表示|0x0101|
|ACTION_MASK十六进制表示|0x00ff|
|getAction()二进制表示|000001 000001|
|ACTION_MASK二进制表示|000000 011111|
|getActionMasked()二进制表示|000000 000001|
|getActionMasked()十六进制表示|0x0001(Move事件)|

```java
public static final int ACTION_MASK = 0xff;
public static int getActionMasked(MotionEvent event) {
        return event.getAction() & ACTION_MASK;
}
```

- `MotionEventCompat.getActionMasked()`是对版本兼容的方法。推荐使用这个类进行相关操作达到兼容版本的目的。

## Pointer

- pointer , pointerIndex , event , action , poiterId

- pointer代表一个触摸点
- pointerIndex是pointer在MotionEvent中的索引
- event代表事件，比如0(Down),1(UP)...
- action是pointerIndex和event拼接的十六进制形式
- pointerId是一个pointer的唯一标示，他在整个事件流中是不会改变的，但是pointerIndex的值会改变，比如，先后放置三个指头在屏幕上会接收到`0x0000(第一个指头Down事件)`,`0x0105(第二个指头Pointer_Down事件)`,`0x0205(第三个指头Pointer_Down事件)`,抬起第二个指头时会触发`0x0106(Pointer_Up)`,再抬起第三个指头也会触发`0x0106(Pointer_Up)`,也就是说抬起第二个指头之后第三个指头的index由02变成了01,所以index只是MotionEvnet中每个事件的下标，与事件不能形成标志性的关系。


## MotionEventCompat

- MotionEventCompat是一个针对事件处理提供的辅助类，内部做了版本兼容。

- `getActionIndex(MotionEvent event)`获取该事件的索引，有点类似于`MotionEventCompat.getActionMasked()`,都使用‘与’运算，只不过一个取的是高位的pointerIndex,后者取得是低位的event事件。

```java
public static int getActionIndex(MotionEvent event) {
        return (event.getAction() & ACTION_POINTER_INDEX_MASK)
                >> ACTION_POINTER_INDEX_SHIFT;
    }
```
- `getPointerId(MotionEvent event, int pointerIndex)`在一个MotionEvent中根据触摸点pointerIndex的值获取pointerId

```java
public static int getPointerId(MotionEvent event, int pointerIndex) {
        return IMPL.getPointerId(event, pointerIndex);
}
```

- `findPointerIndex(MotionEvent event, int pointerId)`在一个MotionEvent中根据pointerId获取pointerIndex

```java
public static int findPointerIndex(MotionEvent event, int pointerId) {
        return IMPL.findPointerIndex(event, pointerId);
}
```

- 从一个MotionEvent中根据pointerIndex获取对应触摸点的XY坐标

```java
public static float getX(MotionEvent event, int pointerIndex) {
        return IMPL.getX(event, pointerIndex);
}
public static float getY(MotionEvent event, int pointerIndex) {
        return IMPL.getY(event, pointerIndex);
}
```
