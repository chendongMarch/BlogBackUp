---
layout: post
title: 分析SwipeRefreshLayout实现自定义刷新
category: Android
tags:
  - Android
  - SourceCode
abbrlink: 308486771
date: 2016-08-25 00:00:00
keywords:
---


## 前言
-  分析SwipeRefreshLayout的源代码来深入的理解一下关于刷新的事件处理和NestedScrolling机制的使用。Android5.0之后官方推出了SwipeRefreshLayout来实现下拉刷新，当然还有很多其他控件，因为像下拉刷新，抽屉效果，tabLayout效果几乎是每个App都需要的，在之前的版本中Android没有定义这些控件，因此GitHub上也涌现了大量的自定控件的库，大家可以很方便的引用这些库实现效果，比如很火的`pull-to-refresh`以及后来的`ultra-pull-to-refresh`。Android在推出SwipeRefreshLayout之后并没有在国内得到很好的使用，大家还是在用自己的刷新和开源库来实现。可能是因为：

1. SwipeRefreshLayout的效果跟以前的刷新效果有些出入，大家接受了以前的效果对新的效果不太满意。
2. SwipeRefreshLayout自定义度不强，每个App都想有自己的特色，自然不愿意采用大众的做法。

-   这篇文章的目的就是，理解SwipeRefreshLayout 的实现，基于SwipeRefreshLayout实现类似传统的刷新效果并支持自定义。


