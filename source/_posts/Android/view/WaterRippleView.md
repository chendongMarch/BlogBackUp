---
layout: post
title: 「Android」自定义控件 - 水波纹效果
category: Android
tags:
  - Android
  - 自定义 View
keywords:
  - Android
  - 自定义 View
abbrlink: 5c2aef5c
photos: 'https://images.pexels.com/photos/140966/pexels-photo-140966.jpeg?auto=compress&cs=tinysrgb&dpr=2&h=350'
location: 青岛海尔
date: 2018-10-25 10:42:00
---

自定义水波纹效果控件，支持以下特性。

1. 进度控制
2. 波纹效果控制（宽度，层次，颜色，波峰高度）
3. 形状形状，理论上支持所有形状，配合 `drawable` 实现
4. 资源控制，资源的回收和复用，避免占用内存
5. 列表复用

 <!--more-->

## 效果展示

![](http://cdn1.showjoy.com/shop/images/20181025/2TSSQE8Y1HH2RAEMQBLA1540435182348.gif)


