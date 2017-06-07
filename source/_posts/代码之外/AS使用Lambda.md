---
layout: post
title: 在AS中使用lambda表达式
date: 2016-07-16
category: 代码之外
tags: [Android,未必知道,AS]
keywords: 
description: 在AS中使用lambda表达式
---

## 前言
- Lambda表达式从Java8开始支持，简化了书写，同时理解上难度也加大了，不过熟悉了就会好很多啦。但是AS默认不支持Lambda的，需要配置一下


## 根目录下gradle文件配置 
- 在project / build.gradle中配置

```
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.1.3'
        classpath 'me.tatarka:gradle-retrolambda:3.2.5'
    }
}


```

## app下gradle文件配置
- 在project / app / build.gradle顶部配置插件

```
apply plugin: 'me.tatarka.retrolambda'
...
```



## 在android{...}中配置
```
android {
    ......    
    // 注释冲突
	packagingOptions {
  	 	 exclude 'META-INF/services/javax.annotation.processing.Processor'
	}
	// 使用Java1.8
	compileOptions {
 	   sourceCompatibility JavaVersion.VERSION_1_8
 	   targetCompatibility JavaVersion.VERSION_1_8
	}
	......
}
```