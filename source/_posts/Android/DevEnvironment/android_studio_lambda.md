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

<!--more-->


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