---
layout: post
title: Rxjava2.x开发-n-Backpressure-todo
categories:
  - Android
  - Library
tags:
  - Android
  - Library
  - RxJava2.x
keywords:
  - Android
  - RxJava
  - Flowable
  - backpressure
  - 背压
comments: true
abbrlink: 2a4b463b
date: 2018-07-05 16:38:00
password:
hide: true
---

本文主要记录总结背压的相关知识。

当上游发送的事件的速度很快，下游处理事件的速度很慢，就会出现背压，这很好理解，就好比一条水管，上游水流量太大，下游水流量太小，上游送过来的水下游不能及时输出，就会产生压力，这就是背压。

**背压是指在异步场景中，被观察者发送事件速度远快于观察者的处理速度的情况下，一种告诉上游的被观察者降低发送速度的策略**

<!--more-->

本来想写一段程序触发一下 `MissingBackpressureException` ，结果没成功。

在 `RxJava2.x` 中 `Observable` 是默认不支持背压的，而 `Flowable` 是支持背压的。




