---
layout: post
title: 防止app闪白屏或闪黑屏
categories:
  - Android
  - Tips
tags:
  - Android
keywords:
  - Android
  - 去掉启动闪屏
abbrlink: 3133739946
date: 2016-04-18 00:00:00
---

App启动时需要加载应用进程，就算你的软件在 `Appliation` 中什么也没做仍旧会有延时，会显示白屏或者黑屏，很难看。本文主要介绍去掉闪屏，白屏或黑屏的几种方案。


## 透明 Theme
- 使用透明Theme解决，原理就是虽然程序启动了，但是没有显示出来，你看到的还是桌面，目前主流的产品都是用的这种方式，比如QQ，微信。缺点就是等待的时间长，造成程序启动慢的感觉。

```xml
<style name="Theme.AppStartUseTransparent" parent="Theme.AppCompat.NoActionBar">
        <item name="android:windowIsTranslucent">true</item>
        <item name="android:windowNoTitle">true</item>
</style>
```


## 图片 Theme
- 使用图片Theme解决，原理就是设置一张背景图，在你的程序没有加载完成的时候会显示这张背景图，你也可以用shape自定义或者使用图片，优点就是启动很快，显示的效果取决于你的图片。

```xml
<style name="Theme.AppStartUseDrawable" parent="Theme.AppCompat.NoActionBar">
        <item name="android:windowBackground">@drawable/shape_maincolor</item>
        <item name="android:windowNoTitle">true</item>
</style>
```


## 在 manifest.xml 使用
- 我是AppCompat的风格，你可以对parent＝“”做适当修改。

```xml
 <activity
            android:name=".activity.LoadingActivity"
            android:theme="@style/Theme.AppStartUseTransparent">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
</activity>
```