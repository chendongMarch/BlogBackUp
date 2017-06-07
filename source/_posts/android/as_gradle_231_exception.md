---
layout: post
title: AS升级gradle到2.3.1异常
date: 2017-05-26
category: Android
tags: Android
keywords: 
---

AndroidStudio升级gradle插件版本到2.3.1之后不能运行了，提示` MultiDex找不到 `，由此引发了很多问题。

<!--more-->

## 开始升级
插件版本升级到2.3.1，配置`project / build.gradle`

```gradle
classpath "com.android.tools.build:gradle:2.3.1"
```
gradle版本升级到3.3，配置`project / gradle / gradle-wrapper.properties`

```gradle
#Sun Mar 05 00:36:45 CST 2017
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-3.3-all.zip
```

## 分包引起的问题
配置分包
```gradle
defaultConfig {
        multiDexEnabled true
    }
    
依赖添加
compile 'com.android.support:multidex:1.0.1'
```


## 小米的问题
一切都ok之后出现了新的问题，`Failed to establish session`   
然后无法安装应用，在github上面找到了回答，瞻仰一下。  

最后总结就是要把开发者模式的MIUI优化选项给关闭掉，不然安装不了。   
之前还有一个开启允许Usb安装的选项，不打开的话也无法安装应用，总之小米手机特别一点。

```
Works !
for those who suffer from this:

enable developer mode - In your phone, go to Settings, About phone and click on MIUI version 7 times. You’ll see a pop up which says you are a developer now.  
 
Go back to Settings, Additional settings, Developer options and enable USB Debugging.
   
Connect your phone to your PC/Mac and on the phone authorize your computer

go back to Developer options, scroll down to find Turn on MIUI optimization and disable it. Your phone will be rebooted

Try it now :)
```