---
layout: post
title: 「Android」ConstraintLayout
categories:
  - Android
abbrlink: c0b3765
tags:
  - Android
  - View
keywords:
  - ConstraintLayout
comments: true
date: 2018-09-18 10:02:00
---


本文主要对 `ConstraintLayout` 的布局形式按照不同的约束类型进行划分，对约束属性和辅助组件进行整理。

其中约束属性包括 `constraint`、`center`、`match_constraint`、`ratio`、`percent`、`bias`、`chain`、`weight`、`circle` 等。

辅助组件包括 `GuideLine`、`Barrier`、`Group`、`Placeholder` 等。

<!--more-->

## constraint 

约束布局，是 `ConstraintLayout` 的核心，借助在 **上下左右** 四个方向上与其他控件、父控件进行相互约束，确定自己的位置，类似 `RelativeLayout`，不过要更加灵活。

```xml
<!--其他方向也是一样-->

app:layout_constraintTop_toBottomOf="@+id/test"

app:layout_constraintLeft_toLeftOf="parent"
```

## center

居中布局，并没有居中布局的属性，但是可以通过约束布局指定两个方向上的约束，则控件会默认居中布局。

```xml
<!--与父控件约束，在水平方向居中-->
app:layout_constraintLeft_toLeftOf="parent"
app:layout_constraintRight_toRightOf="parent"

<!--与其他控件约束，在垂直方向居中-->
app:layout_constraintTop_toTopOf="@id/test_tv"
app:layout_constraintBottom_toBottomOf="@id/test_tv"
```


## match_constraint 

匹配约束，不是简单的类似 `match_parent`，他的意思是在约束范围内匹配空间，在 `ConstraintLayout` 将控件尺寸声明为 `0dp` 可以达到 `match_constraint` 的效果。

```xml
<!--高度 match_constraint，控件会撑满 test_tv1 到 test_tv2 之间的空隙-->
<TextView
    app:layout_constraintTop_toTopOf="@+id/test_tv1"
    app:layout_constraintBottom_toBottomOf="@id/test_tv2"
    android:layout_width="wrap_content"
    android:layout_height="0dp"/>
    
<!--高度 10dp，控件会在 test_tv1 到 test_tv2 之间居中，但是高度仍旧是 10dp-->
<TextView
    app:layout_constraintTop_toTopOf="@+id/test_tv1"
    app:layout_constraintBottom_toBottomOf="@id/test_tv2"
    android:layout_width="wrap_content"
    android:layout_height="10dp"/>
```

## ratio

比例约束，用宽高比来约束控件的尺寸，根据 `ratio` 计算的那个尺寸需要声明为 `match_constraint`。

```xml
<!--宽度 100，高度 match_constraint，最终高度为 50dp-->
android:layout_width="100dp"
android:layout_height="0dp"
app:layout_constraintDimensionRatio="2:1"


<!--默认 宽：高，意为 w:h = 2:1 -->
app:layout_constraintDimensionRatio="w,2:1"

<!--强制高为基准 高：宽，意为 h:w = 2:1 -->
app:layout_constraintDimensionRatio="h,2:1"

<!--宽高都为 match_constraint，test_tv 宽度 100dp，则该控件宽度因为其他约束的存在也是可以被确定为 100dp 的，所以也可以这样使用-->
app:layout_constraintLeft_toLeftOf="@id/test_tv"
app:layout_constraintRight_toRightOf="@id/test_tv"
android:layout_width="0dp"
android:layout_height="0dp"
app:layout_constraintDimensionRatio="2:1"
```

## percent

百分比布局，用来指定尺寸占据父布局尺寸的百分比，达到更好的适配效果。根据 `percent` 计算的那个尺寸需要声明为 `match_constraint`。


```xml
app:layout_constraintWidth_percent="0.5"
app:layout_constraintHeight_percent="0.5"
```


## bias

偏差约束，当一个控件因为其他约束的存在被确定了位置，在他的 **上下**、**左右** 会留有距离，`Bias` 的存在是为了指定 **上面和下面**、**左边和右边** 距离的比例分配。

比如一个控件因为左右各有约束居中显示了，指定 `Horizontal_bias` 为 `0.1`，则表示他左边的距离占据水平方向空余距离的 `1/10`。

```xml
<!--左右约束使他居中，bias 使他靠近左边显示，左边空白占据全部空白距离的1/10-->
<TextView
    app:layout_constraintLeft_toLeftOf="parent"
    app:layout_constraintRight_toRightOf="parent"
    app:layout_constraintHorizontal_bias="0.1"
    android:layout_width="100dp"
    android:layout_height="10dp"/>
    
<!--也可以指定垂直方向的偏差-->   
app:layout_constraintHorizontal_bias="0.5"
app:layout_constraintVertical_bias="0.5"
```



## chain

链约束，多个控件在一个方向上面互相约束，环环相扣，好像一条锁链一样，形成链约束，`chainStyle` 决定了在一条链当中，空白间距和控件之间是如何摆放的。

```xml
<!--控件受拉力均匀分布，两边和控件之间平分间距--> 
app:layout_constraintVertical_chainStyle="spread"
app:layout_constraintHorizontal_chainStyle="spread"
<!--控件受拉力均匀分布，两边没有空白间距，控件之间平分间距--> 
app:layout_constraintVertical_chainStyle="spread_inside"
app:layout_constraintHorizontal_chainStyle="spread_inside"
<!--控件全部挤压在中间，空白间距在两边--> 
app:layout_constraintVertical_chainStyle="packed"
app:layout_constraintHorizontal_chainStyle="packed"

<!--以下三个控件在一个方向上互相约束，达到一种链的效果--> 
<View
    android:id="@+id/v1"
    app:layout_constraintLeft_toLeftOf="parent"
    app:layout_constraintRight_toLeftOf="@+id/v2"/>
<View
    android:id="@+id/v2"
    app:layout_constraintLeft_toRightOf="@+id/v1"
    app:layout_constraintRight_toLeftOf="@+id/v3"/>
<View
    android:id="@+id/v3"
    app:layout_constraintLeft_toRightOf="@+id/v2"
    app:layout_constraintRight_toRightOf="parent"/>
```

## weight

权重，类似 `LinearLayout` 的 `weight` 属性，根据 `weight` 计算的那个尺寸需要声明为 `match_constraint`。

使用 `weight` 布局的控件需要在一个方向上面有一个完整的约束，也就是他们就好像在一个 `LiearLayout` 里面的那种感觉，这几个控件互相也是需要有约束的，`weight` 常和 `Chain` 布局合作，可以看下面的例子。

```xml
app:layout_constraintHorizontal_weight="1"
app:layout_constraintVertical_weight="1"

<!--以下三个控件是链约束，宽度是 match_constraint，中间的权重是其他的 2 倍--> 
<View
    android:id="@+id/v1"
    android:layout_width="0dp"
    android:layout_height="50dp"
    android:background="#f00"
    app:layout_constraintHorizontal_weight="1"
    app:layout_constraintLeft_toLeftOf="parent"
    app:layout_constraintRight_toLeftOf="@+id/v2"/>
<View
    android:id="@+id/v2"
    android:layout_width="0dp"
    android:layout_height="50dp"
    android:background="#0f0"
    app:layout_constraintHorizontal_weight="2"
    app:layout_constraintLeft_toRightOf="@+id/v1"
    app:layout_constraintRight_toLeftOf="@+id/v3"/>
<View
    android:id="@+id/v3"
    android:layout_width="0dp"
    android:layout_height="50dp"
    android:background="#00f"
    app:layout_constraintHorizontal_weight="1"
    app:layout_constraintLeft_toRightOf="@+id/v2"
    app:layout_constraintRight_toRightOf="parent"/>
```

## circle

圆形约束，通过指定圆心、半径、角度达到圆形布局的效果，`0度` 的位置是控件的垂直正上方，顺时针旋转增大。

```xml
<View
    android:id="@+id/v4"
    android:layout_width="50dp"
    android:layout_height="50dp"
    android:background="#00f"
    app:layout_constraintCircle="@+id/v2"
    app:layout_constraintCircleAngle="90"
    app:layout_constraintCircleRadius="100dp"/>
```

## GuideLine

参考线，`GuideLine` 是一个空的 `View` 他的存在是为了给其他 `View` 布局提供一个参考的标准

```xml
<!--使用 GuideLine 构建两条居中、互相垂直的参考线，将布局分为 4 个象限--> 
<android.support.constraint.Guideline
    android:layout_width="0dp"
    android:layout_height="0dp"
    android:orientation="vertical"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintGuide_percent="0.5"
    app:layout_constraintTop_toTopOf="parent"/>
<android.support.constraint.Guideline
    android:layout_width="0dp"
    android:layout_height="0dp"
    android:orientation="horizontal"
    app:layout_constraintGuide_percent="0.5"
    app:layout_constraintLeft_toLeftOf="parent"
    app:layout_constraintRight_toRightOf="parent"/>
    
<!--可以使用如下属性更好的确定 GuideLine 的位置，其他属性使用起来和 View 一样的-->
android:orientation="horizontal"
app:layout_constraintGuide_percent="0.5"
app:layout_constraintWidth_percent="0.5"
app:layout_constraintHeight_percent="0.5"
app:layout_constraintGuide_begin="10dp"
app:layout_constraintGuide_end="10dp"
```

## Group

分组，因为使用 `ConstraintLayout` 大幅度的降低了布局的层级的同时，很多原本不同逻辑的 `View` 组合现在被布局在相同的层级下面，与此同时也丧失了我们隐藏父控件就可以隐藏一整个逻辑组合控件的便利。`Group` 是一个空的 `View`，他的作用是用来关联其他控件，也被称为分组，这样一来隐藏 `Group` 就可以便捷的同时隐藏他关联的所有控件。

```xml
<android.support.constraint.Group
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:constraint_referenced_ids="test_id1,test_id2,test_id3"/>
```

## Barrier

屏障，因为控件之间需要相互约束控制自己的位置，并且每个方向上只能存在一个约束控件，但是在一个方向上可能出现需要和多个控件同时约束的情况，`Barrier` 是一个空的 `View`，他的出现就是为了解决某个方向上需要动态的和多个控件约束的情况，关联这些不确定的控件创建一个 `Barrier`，`Barrier` 是可以实现在某个方向和多个控件约束的，然后我们控件再和 `Barrier` 进行约束控制即可。

比如当前的控件需要在两个控件中比较宽的那个右边，但是这两个控件在运行期间内容会不断变化，我们没办法确定哪个是更宽的，使用 `Barrier` 可以在这两个控件右边设置一个屏障，关联到这两个控件，则随着两个控件的变化，屏障也总是在两个控件中较宽的控件右边，我们的控件与 `Barrier` 进行约束控制即可。

- `barrierDirection` 屏障的方向。
- `barrierAllowsGoneWidgets` 是否将 `GONE` 掉的控件考虑在内，默认为 `true`，即当 `Barrier` 引用的控件被 `GONE` 掉时，则 `Barrier` 默认的创建行为是在已 `GONE` 掉控件的已解析位置上进行创建。如果设置为 `false`，则不会将 `GONE` 掉的控件考虑在内。


```xml
<android.support.constraint.Barrier
    app:barrierDirection="bottom"
    app:barrierAllowsGoneWidgets="true"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:constraint_referenced_ids="test_tv1,test_tv2"/>
```

## Placeholder

占位，`Placeholder` 是一个空的 `View`，主要用来占位。

- 通过 `setContentId()` 设置内容，被设置的内容会在原来的位置消失而到 `PlaceHolder` 的位置上。
- 通过 `setEmptyVisibility()` 可以设置当没有内容时，`PlaceHolder` 的可见性，默认是 `INVISIBLE`。

```xml
<android.support.constraint.Placeholder
    android:layout_width="100dp"
    android:layout_height="100dp" />
```

## 综上

`ConstraintLayout` 目前的版本是 `1.1.x`，新的特性不断的加入进来，目前的开发中已经基本全部使用 `ConstraintLayout` 来进行布局，它完全可以代其他所有的布局形式，了，相信在不久以后就可以一统江湖了。