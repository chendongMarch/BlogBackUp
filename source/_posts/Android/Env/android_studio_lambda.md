---
layout: post
title: 在 AS 中使用 Lambda 表达式
categories:
  - Android
tags:
  - Android
  - DevEnvironment
keywords:
  - Android
  - AndroidStudio
  - Lambda
abbrlink: 4229621729
date: 2016-07-16 00:00:00
---
 

配置 AndroidStudio 支持 Lambda 表达式

2017.9.30 升级到 3.7.0 并与 `freeline` 编译使用

<!--more-->


## 根目录下gradle文件配置 

在 `project / build.gradle` 中配置

```
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.3.3'
        classpath 'me.tatarka:gradle-retrolambda:3.7.0'
    }
}


```

## app下gradle文件配置

在 `project / app / build.gradle` 顶部配置插件

```
apply plugin: 'me.tatarka.retrolambda'
```



## 在android{...}中配置
```
android {

    retrolambda {
        javaVersion JavaVersion.VERSION_1_7
        jvmArgs '-noverify'
        defaultMethods false
        incremental true
    }

    // 使用Java1.8
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}
```