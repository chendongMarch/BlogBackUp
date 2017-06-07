---
layout: post
title: NestedScrolling交互
date: 2016-08-25
category: Android
tags: [Android,Android5.+]
keywords: 
description: NestedScrolling交互
---


## 前言



## 交互

|触发时机|NestedScrollChild|NestedScrollParent|
|:---:|:---|:----|
|Down|startNestScroll|onStartNestScroll|
|||onNestScrollAccept|
|Move|dispatchNestPreScroll|onNestPreScroll|
||dispatchNestScroll|onNestScroll|
|Up|stopNest||