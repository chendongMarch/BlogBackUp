---
layout: post
title: Android新控件CardView
date: 2015-10-14
category: Android
tags: [Android,Android5.+]
keywords: 
description: Android新控件CardView
---

## Gradle
- `compile 'com.android.support:cardview-v7:21.+`

## xml文件
```xml
<android.support.v7.widget.CardView

android:layout_marginTop="20dp" 
android:layout_width="wrap_content"
android:layout_height="wrap_content"
app:cardCornerRadius="10dp"
app:cardElevation="5dp"
app:contentPadding="10dp"      
app:cardPreventCornerOverlap="true"
>
<ImageView           
android:scaleType="centerCrop"
android:src="@mipmap/test"          android:layout_width="match_parent"            android:layout_height="wrap_content"/>
</android.support.v7.widget.CardView>

```
## 几个属性值
```
app:cardElevation="5dp"//阴影
app:cardElevation 阴影的大小
app:cardMaxElevation 阴影最大高度
app:cardBackgroundColor 卡片的背景色
app:cardCornerRadius 卡片的圆角大小
app:contentPadding 卡片内容于边距的间隔
app:contentPaddingBottom
app:contentPaddingTop
app:contentPaddingLeft
app:contentPaddingRight
app:contentPaddingStart
app:contentPaddingEnd
app:cardUseCompatPadding 设置内边距，V21+的版本和之前的版本仍旧具有一样的计算方式
app:cardPreventConrerOverlap 在V20和之前的版本中添加内边距，这个属性为了防止内容和边角的重叠
```
