---
layout: post
title: NestedScrolling交互
categories:
  - Android
  - todo
tags:
  - Android
keywords:
  - NestedScrolling
abbrlink: 2940188893
date: 2016-08-25 00:00:00
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