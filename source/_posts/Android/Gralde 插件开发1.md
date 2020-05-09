---
layout: post
title: 记一次使用 Gradle 插件重构项目的经历
category: Android
tags:
  - Android
abbrlink: 882f967a
date: '2019-01-02 18:15:00'
location: 杭州拱墅-宝拍
keywords:
---

## 背景

先打个广告，在维护的一个三方登录分享的项目(求 🌟)，以更简单、更轻量、更加面向业务需求为设计目标，提供 **微博**、**微信**、**QQ**、**Tim**、**QQ 轻聊版**、**钉钉** 的登陆分享功能支持；

刚开始的时候，所有代码实现都写在一个 `module` 里面，虽然按照平台在代码结构上进行分离了，但是如果遇到只需要其中几个平台的情况使用起来非常不灵活，所以就萌生了写一个 `gradle` 插件来统一管理几个平台的实现和依赖的想法。

最后开发完成以后发现和 `ShareSdk` 有点像（就接入方式而言），可能这是公认的一个比较好的实现方式，所以类似需要动态的接入不同的依赖、并且配置项比较繁多的业务场景，是非常适合用插件来管理和配置的，本文主要记录使用 `gradle` 插件该项目的重构过程。

项目地址 : [GitHub - SocialSdkLibrary](https://github.com/chendongMarch/SocialSdkLibrary)

博客地址 ：[快速接入微信微博QQ钉钉原生登录分享](http://zfyx.coding.me/article/3067853428/)

<!--more-->

## 基本设计

因为本文主要是针对 `Gradle` 插件开发来整理的，所以在此之前我已经将原来的项目进行了拆分，划分为了核心库、微信平台库、QQ 平台库、微博平台库、钉钉平台库 5 个部分，并将他们发布到了 `Bintray` 获得依赖坐标。

```gradle
implementation "com.zfy:social-sdk-core:0.0.7"
implementation "com.zfy:social-sdk-wx:0.0.7"
implementation "com.zfy:social-sdk-dd:0.0.7"
implementation "com.zfy:social-sdk-qq:0.0.7"
implementation "com.zfy:social-sdk-weibo:0.0.7"
```

整个 `gradle` 插件的开发过程大致分为了如下几步：

1. 创建插件项目 - 搭建 `HelloWorld` 项目，并在本地完成编译依赖；
3. 自定义参数的配置和解析 - 在插件中解析 `gradle` 文件中配置的参数；
4. 动态生成 `java` 配置文件 - 根据配置的参数动态生成 `SocialSdkConifg.java` 文件；==~~==
4. 动态增加依赖 - 根据参数动态为项目添加不同的依赖，取决于需要哪个平台；
5. 动态更改 `Android` 配置- 操作主项目的 `manifestPlaceholders`，增加 `qq_id` 字段。
6. 将插件发布到本地目录，并依赖到项目中；
7. 将插件发布到 `bintray`，并依赖到项目中；


















 




