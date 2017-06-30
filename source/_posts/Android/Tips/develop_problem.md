---
layout: post
title: Android开发问题常见问题
categories:
  - Android
  - Tips
tags:
  - Android
keywords:
  - Android
abbrlink: 1817351074
date: 2015-01-18 00:00:00
---

## 方法数量超出限制
- 使用分包策略

- 出现exit value 3之类的错误通常是方法数目超出限制了，建议使用分包机制，但这不是解决问题的方法,最重要的还是准确选择类库，精简方法数。


```java
//引入分包
compile 'com.android.support:multidex:1.0.1'

／／添加允许多方法
defaultConfig {
        multiDexEnabled true
    }

／／在Application添加分包初始化
@Override
protected void attachBaseContext(Context base) {
        super.attachBaseContext(base);
        MultiDex.install(this);
    ｝
```

## 方法数统计
- 推荐一个插件，可以计算每个包下面的方法数目

- 在app.gradle文件头添加以下插件，会在app/build/outputs/dexcount/debug.txt文件里生成每个包下面的方法数，如下：
apply plugin: 'com.getkeepsafe.dexcount' 

```xml
methods  fields   package/class name
8        0        <unnamed>
21186    8027     android
6        0        android.accessibilityservice
34       0        android.animation
556      33       android.app
3        0        android.app.usage
2        0        android.appwidget
488      50       android.content
42       43       android.content.pm
106      6        android.content.res
79       0        android.database
```


## 签名问题
```java
//正式签名相关信息
signingConfigs {
   myConfig {
        storeFile file("相对或者绝对路径")
        storePassword "sangular123"
        keyAlias "sangular"
        keyPassword "sangular123"
   }
}
buildTypes {
   //release版本的正式签名
    release {
        //minifyEnabled true
        打包发布时会使用该代码
        //proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        signingConfig signingConfigs.myConfig
     }
     //debug版本也用正式签名调试，这样可以在debug版本下使用第三方登陆等功能
     debug {
            signingConfig signingConfigs.myConfig
            ｝
}
```
