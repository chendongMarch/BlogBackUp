---
layout: post
title: android_libjpeg_cmake
categories:
  - null
tags:
  - null
keywords:
  - null
comments: true
abbrlink: fc67caea
date: 2017-07-19 19:56:37
password:
---

本文主要介绍使用  `CMAKE` 编译 `libjpeg-turbo` 类库。

`libjpeg-turbo` 是一个图片编辑的库，附[ GitHub 地址](https://github.com/libjpeg-turbo/libjpeg-turbo)，`libjpeg-turbo` 是个运用极其广泛的库。可以说，基本上电脑上手机上能见到的 JPEG 压缩的地方用的一般都是 `libjpeg-turbo`。本文只使用了图片压缩的功能。

<!--more-->

使用 `Android` 保存图片时，我们通常使用的是 `Bitmap.compress()` 方法，但是使用该方法时，就算 `quality` 设置为 100，图片质量还是会越来越模糊，颜色也会越来越绿～，至于为什么会这样，请看 [知乎回答](https://www.zhihu.com/question/29355920)。

这个问题在贴吧上体现尤为明显，贴吧里面经常很多绿绿的图片就是因为大家保存下来上传上去，保存下来上传上去 ... 导致质量越拉越低。