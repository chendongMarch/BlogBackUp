---
layout: post
title: 「View」ProgressBar
categories:
  - Android
tags:
  - Android
keywords:
  - Android
  - View
abbrlink: edb40764
photos: https://images.pexels.com/photos/825904/pexels-photo-825904.jpeg?auto=compress&cs=tinysrgb&dpr=2&w=500
location: 青岛海尔
date: 2018-12-26 11:36:00
---

`ProgressBar` 是比较常见的用来显示进度的的控件，支持圆形和水平进度显示，通过扩展它可以实现大多数情况下的需求，不需要自己自定义控件，特此记录一下，本文主要介绍：

1. 圆形、无进度、自定义颜色；
2. 水平、带进度、自定义背景/进度颜色；
3. 圆形、带进度、自定义背景/进度颜色；
<!--more-->

## 知识准备

用到了几个不常用的 `shape` 属性，当我们使用 `ring` 类型时：

```xml
android:innerRadius  尺寸，内环的半径，单位 dp。
android:innerRadiusRatio  大于1浮点型，以环的宽度比率来表示内环的半径。
android:thickness  尺寸，环的厚度
android:thicknessRatio  大于1浮点型，以环的宽度比率来表示环的厚度    
android:useLevel boolean 值，如果当做是LevelListDrawable使用时值为true，否则为false.
```

举个例子，假设我们有个 `ProgressBar` 宽度和高度都是 `100dp`，想给他一个宽度为 `2dp` 的环形进度，那么：

```
先设置环半径

如果使用 innerRadius，因为宽度是 100dp，去掉两边 2dp 的环，还有 96dp，半径为 96/2 = 48dp
	android:innerRadius="48dp"
如果使用 innerRadiusRatio，半径为 48dp 比例就应该是 100/48 = 2.08
	android:innerRadiusRatio="2.08"
	
再设置环的宽度

如果使用 thickness，很简单，他应该是 2dp
	android:thickness="2dp"
如果使用 thicknessRatio，那么就应该是 105/2 = 50
	android:thicknessRatio="50"
	
使用 Ratio 时，填写是一个大于 0 的比例数字。
```

## 圆形、无进度、自定义颜色

圆形 `ProgressBar` 无进度，并自带了旋转效果，那怎么更改圆形进度的颜色，下面使用了一个渐变色作为进度条颜色，同样也可以使用更简单的纯色表示。

> progress_circle.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<rotate xmlns:android="http://schemas.android.com/apk/res/android"
    android:fromDegrees="0"
    android:pivotX="50%"
    android:pivotY="50%"
    android:toDegrees="360">

    <shape
        android:innerRadius="48dp"
        android:shape="ring"
        android:thickness="2dp"
        android:useLevel="false">

        <gradient
            android:centerX="0.5"
            android:centerY="0.5"
            android:endColor="#FF696969"
            android:startColor="#FFFFFFFF"
            android:type="sweep" />
    </shape>

</rotate>

```
在 `xml` 中使用它：

```xml
<ProgressBar
    android:layout_width="100dp"
    android:layout_height="100dp"
    android:indeterminateDrawable="@drawable/progress_circle" />
```

效果展示：

![](http://cdn1.showjoy.com/shop/images/20181226/PT7EO44B8QYTYY1NVJZ81545802833845.png)

## 水平、带进度、自定义背景/进度颜色


水平进度条，可以设置背景、一级进度、二级进度的颜色，需要借助 `layer-list` 来实现；

> progress_horizontal.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <!--背景颜色-->
    <item android:id="@android:id/background">
        <shape>
            <solid android:color="@color/color999" />
            <size android:height="1dp" />
            <corners android:radius="10dp" />
        </shape>
    </item>
    <!--二级进度颜色-->
    <item android:id="@android:id/secondaryProgress">
        <clip>
            <shape>
                <solid android:color="@color/redPrimary" />
                <size android:height="1dp" />
                <corners android:radius="10dp" />
            </shape>
        </clip>
    </item>
    <!--一级进度颜色-->
    <item android:id="@android:id/progress">
        <clip>
            <shape>
                <solid android:color="@color/colorPrimary" />
                <size android:height="1dp" />
                <corners android:radius="10dp" />
            </shape>
        </clip>
    </item>
</layer-list>
```

在 `xml` 文件中使用它：

```xml
<ProgressBar
    style="?android:attr/progressBarStyleHorizontal"
    android:layout_width="100dp"
    android:layout_height="5dp"
    android:progressDrawable="@drawable/progress_horizontal"
    android:max="100"
    android:progress="30"
    android:secondaryProgress="60" />
```

效果展示：

![](http://cdn1.showjoy.com/shop/images/20181226/6OCHAI9JFMFI6P2MEZM31545802761284.png)


## 圆形、带进度、自定义背景/进度颜色


首先定义进度颜色的的 `xml` 文件

> progress_circle.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<rotate xmlns:android="http://schemas.android.com/apk/res/android"
    android:fromDegrees="270"
    android:toDegrees="270">

    <shape
        android:innerRadius="48dp"
        android:thickness="2dp"
        android:shape="ring"
        android:useLevel="true">
        <solid android:color="@color/colorPrimary" />
        <corners android:radius="10dp"/>
    </shape>
</rotate>
```

仅仅这样没有背景效果，我们单独做一个背景

> progress_bg.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:innerRadius="48dp"
    android:shape="oval"
    android:thickness="2dp"
    android:useLevel="true">

    <stroke
        android:width="2dp"
        android:color="@color/color999" />
</shape>
```

在 `xml` 中使用它：

```xml
<ProgressBar
    style="?android:attr/progressBarStyleHorizontal"
    android:layout_width="100dp"
    android:layout_height="100dp"
    android:background="@drawable/progress_bg"
    android:max="100"
    android:progress="60"
    android:progressDrawable="@drawable/progress_horizontal" />
```

效果展示：

![](http://cdn1.showjoy.com/shop/images/20181226/WUUSHLOZE78FMC5FRIMQ1545802687935.png)



## 圆形、带进度、自定义背景/进度颜色2



使用上面的方法比较好理解，但是写起来麻烦，需要写两个 `xml` 文件，而且不支持二级进度，我们理解了他的工作原理，可以使用 `layer-list` 来组合改写它；

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <!--背景颜色,一个简单的 shape-->
    <item android:id="@android:id/background">
        <shape xmlns:android="http://schemas.android.com/apk/res/android"
            android:innerRadius="48dp"
            android:shape="oval"
            android:thickness="2dp"
            android:useLevel="true">

            <stroke
                android:width="2dp"
                android:color="@color/color999" />
        </shape>
    </item>
    <!--二级进度颜色-->
    <item android:id="@android:id/secondaryProgress">
        <rotate xmlns:android="http://schemas.android.com/apk/res/android"
            android:fromDegrees="270"
            android:toDegrees="270">

            <shape
                android:innerRadius="48dp"
                android:thickness="2dp"
                android:shape="ring"
                android:useLevel="true">
                <solid android:color="@color/redPrimary" />
                <corners android:radius="10dp"/>
            </shape>
        </rotate>
    </item>
    <!--一级进度颜色-->
    <item android:id="@android:id/progress">
        <rotate xmlns:android="http://schemas.android.com/apk/res/android"
            android:fromDegrees="270"
            android:toDegrees="270">

            <shape
                android:innerRadius="48dp"
                android:thickness="2dp"
                android:shape="ring"
                android:useLevel="true">
                <solid android:color="@color/colorPrimary" />
                <corners android:radius="10dp"/>
            </shape>
        </rotate>
    </item>
</layer-list>
```

在 `xml` 中使用它：

```xml
<ProgressBar
    style="?android:attr/progressBarStyleHorizontal"
    android:layout_width="100dp"
    android:layout_height="100dp"
    android:max="100"
    android:progress="30"
    android:progressDrawable="@drawable/progress_horizontal"
    android:secondaryProgress="60"/>
```

效果展示：


![](http://cdn1.showjoy.com/shop/images/20181226/IJEG5CMQT5SEBM13XROM1545803221149.png)




