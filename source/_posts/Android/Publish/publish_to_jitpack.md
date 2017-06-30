---
layout: post
title: 将自己写的库发布到 JitPack
categories:
  - Android
  - Publish
tags:
  - Android
  - Publish
keywords:
  - Android
  - JitPack
abbrlink: 2091453987
date: 2016-03-12 00:00:00
---


## JitPack优缺 
1. 在使用Jcenter发布库代码时，需要先上传到Bintray网站，比较麻烦，不过我之前已经给出了简化的方案,根据这篇文章[发布自己的库到JCenter](http://blog.csdn.net/chendong_/article/details/50867247)可以很快的配置好项目。本文介绍另外一种发布自己的库代码的方式，使用[JitPack](https://jitpack.io)发布代码更简单一些。
2. AndroidStudio创建项目时没有默认生成JitPack的依赖，当使用库时需要添加如下依赖
```
repositories {
        jcenter()
        maven { url "https://jitpack.io" }
    }
```

## 创建工程
新建Library Module,这些就不仔细说了，大家都会，需要注意的一点是，使用JCenter时，库的名字是跟你的Module name有关的，但是JitPack是与你的Project name有关的,所以给你的项目起一个好名字，他将作为库的名称。

## 配置插件
与发布到JCenter一样，需要插件,这里说一下这个插件，[该插件的GitHub地址](https://github.com/dcendents/android-maven-gradle-plugin)，在发布到JCenter时也用到了这个插件，现在的最新版本1.4.1，我试了一下，编译不成功，看了Issues,大概是因为gradle需要提升到2.14.1才可以，关于这个问题可以自己去查一下，像我下面这样的配置是ok的，版本的对应可以去Git上看看。
```
buildscript {
    repositories {
        jcenter()
        mavenCentral()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.1.2'
        classpath 'com.github.dcendents:android-maven-gradle-plugin:1.3'
    }
}
```

## 配置Module的build.gradle
```
//默认就有的
apply plugin: 'com.android.library'
//这个跟Jcenter一样的插件
apply plugin: 'com.github.dcendents.android-maven'
//你的Github用户名替换一下
group='com.github.chendongMarch'
```


## 发布到GitHub
- 接下来就去GitHub上打开你的项目－>点击release－>点击创建新的release版本－>书写版本号和release信息－>就会看见到此就完成了发布。如下

![](http://7xtjec.com1.z0.glb.clouddn.com/2016-08-12%2023.27.46.png)

![](http://7xtjec.com1.z0.glb.clouddn.com/%202016-08-12%2023.28.18.png)

![](http://7xtjec.com1.z0.glb.clouddn.com/2016-08-12%2023.28.05.png)



## JitPack发布

- 最后去[JitPack](https://jitpack.io),会看见如下界面,输入你的项目地址，会看到版本号，点击get it，下面会出现compile地址

![](http://7xtjec.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-08-12%2023.34.17.png)

![](http://7xtjec.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-08-12%2023.34.31.png)

![](http://7xtjec.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-08-12%2023.36.29.png)


## 完成
- AS依赖

```
compile 'com.github.chendongMarch:JitPackLibs:1.0.2'
```