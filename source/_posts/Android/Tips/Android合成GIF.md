---
layout: post
title: 在Android平台下合成Gif
categories: 
	- Android
tags: 
  - Android
  - AndroidTips
keywords: 
	- Android
	- GIF合成
abbrlink: 3121499733
date: 2016-03-09 00:00:00
---


本文介绍在 `Android` 平台下合成 `GIF` 的方法，查阅资料的过程中发现大致有两种方案。

> 1. 使用 `giflen` (一个 `C` 的合成 `gif` 的库) 进行 `gif` 合成。

> 2. 使用 `java` 层的 `GifEncoder`、`LZWEncoder`、`NeuQuant` 来进行 `gif` 合成。

当然二者都是基于 `LZW` 算法，简单测试的结果是，速度上差不多，由于对 `C` 不是很擅长，因此我选择了 `java` 层进行合成的方法，但是两种方法速度上都是 **很慢很慢** 😭。

因此本文还将会使用多线程独立编码的方法来进行优化，每帧图片并行编码，加快合成速度，现在的成果是 `600 * 450` 的图片 `20` 张的话，时间大约在 `12s` 左右。


> **本文相关源码在 [GitHub - GifMaker](https://github.com/chendongMarch/GifMaker)**


##  使用 giflen 合成

使用 `jni` 合成，感兴趣的同学可以 `google` 一下 `giflen` 这个库，这里有一个编译好的 `so` 和 `jar` 文件，以及相关 `C` 源码，备份在 [GitHub](https://github.com/chendongMarch/GifMaker/tree/master/backup/giflen) 上，有需要的同学可以直接下载使用。经过我的简单尝试发现，使用 `so` 合成的速度和 `java` 合成的速度差不多，都是慢的要死...


## 优化后的 java 合成

原始的合成方法，真的慢...特别慢，简直不能忍。

原始的合成方法是将每一个 `Bitmap` 使用 `LZWEncoder` 进行编码，由于是一个串行的逻辑，后面的图片需要等待前面的编码完成才可以继续下一张编码，优化后的逻辑是启动一个线程池，每张图片独立编码，最后在所有图片编码完成之后输出流合并，输出到文件中，就完成了 `gif` 的合成。

缺点：大量的 `Bitmap` 持有在内存中并行编码，可能会 `OOM`，不过我测试 20 张 450 * 600 的图片，暂时没有出现问题。合成的图片要求宽高应该是一样的，当然使用宽高不一致的图片也不会有问题，但是会优先使用第一张的图片的宽高作为 `gif` 的宽高，出来的图片就有些尴尬，因此图片的转换和处理需要在外面完成。


## 简单演示

```java
private void composeGif(List<Bitmap> bitmaps) {
    String absolutePath = new File(Environment.getExternalStorageDirectory()
            , System.currentTimeMillis() + ".gif").getAbsolutePath();
    new GifMaker(100, mExecutorService)
            .makeGifInThread(bitmaps, absolutePath, new GifMaker.OnGifMakerListener() {
                @Override
                public void onMakeGifSucceed(String outPath) {
                    if (!isFinishing()) {
                        GlideUtils.with(mActivity, outPath).into(mImageView);
                    }
                }
            });
}
```

看一下输出的结果，20 张大约维持在 12s 左右

```java
I/GifMaker: 完成第10帧,耗时:9.594 s - bitmap [600,450]
I/GifMaker: 完成第11帧,耗时:9.774 s - bitmap [600,450]
I/GifMaker: 完成第18帧,耗时:9.880 s - bitmap [450,600]
I/GifMaker: 完成第9帧,耗时:10.41 s - bitmap [600,450]
I/GifMaker: 完成第17帧,耗时:10.145 s - bitmap [450,600]
I/GifMaker: 完成第16帧,耗时:10.331 s - bitmap [450,600]
I/GifMaker: 完成第6帧,耗时:10.586 s - bitmap [450,600]
I/GifMaker: 完成第5帧,耗时:10.557 s - bitmap [450,600]
I/GifMaker: 完成第8帧,耗时:10.701 s - bitmap [450,600]
I/GifMaker: 完成第15帧,耗时:10.715 s - bitmap [450,600]
I/GifMaker: 完成第4帧,耗时:10.736 s - bitmap [450,600]
I/GifMaker: 完成第1帧,耗时:10.835 s - bitmap [450,600]
I/GifMaker: 完成第19帧,耗时:10.842 s - bitmap [450,600]
I/GifMaker: 完成第13帧,耗时:10.940 s - bitmap [450,600]
I/GifMaker: 完成第0帧,耗时:10.944 s - bitmap [450,600]
I/GifMaker: 完成第14帧,耗时:10.967 s - bitmap [450,600]
I/GifMaker: 完成第3帧,耗时:10.989 s - bitmap [450,600]
I/GifMaker: 完成第12帧,耗时:10.994 s - bitmap [450,600]
I/GifMaker: 完成第2帧,耗时:10.994 s - bitmap [450,600]
I/GifMaker: 完成第7帧,耗时:11.148 s - bitmap [450,600]
I/GifMaker: 合成完成,耗时:11.167 s
```


