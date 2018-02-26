---
layout: post
title: RecyclerView Light Adapter [开源]
categories:
  - Android
tags:
  - Android
  - 开源
keywords:
  - Android
  - RecyclerView
  - Adapter
abbrlink: 1632666977
date: 2017-06-19 00:00:00
photos: http://olx4t2q6z.bkt.clouddn.com/18-2-6/27323356.jpg
location: 杭州
---

`LightAdapter` 的设计初衷是能够快速、简单的完成 `RecyclerView` 的数据适配工作，同时也对使用过程中的一些常用功能进行了扩展和封装。

随着功能的慢慢丰富，使用起来也变得越来越复杂，最后决定使用注解的方式对适配器进行配置。
 
> - 基于注解实现基本的数据适配功能。
> - 预加载，支持顶部、底部预加载更多数据。
> - `Header & Footer`，为列表添加 头部 和 尾部。
> - 单击、双击、长按事件支持。
> - 自动 `UI` 线程更新数据，避免数据更新问题。
> - 选择器功能扩展，主要针对点击选中这种场景。

<!--more-->
 
##
