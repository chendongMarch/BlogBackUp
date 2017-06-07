---
layout: post
title: 在Android平台下合成Gif
date: 2016-03-09
category: Android
tags: Android
keywords: Android
description: 在Android平台下合成Gif
---


## 前言
- 最近在做合成gif的工作，在网上搜到了很多资料，总结一下，总共找到了两种方法，分别是使用jni和java算法实现。测试表明java算法比so文件更快，大约快两倍，不知道为啥。




## java算法合成
- [Java算法源代码](https://github.com/chendongMarch/cdlibs/tree/master/src/main/java/com/march/libs/gif)

- 网上流传很广的就是下面在java中的算法，修改了一些bug
1. AnimatedGifEncoder.java
2. LZWEncoder.java
3. NeuQuant.java

## C算法
- [C的源代码](http://download.csdn.net/detail/chendong_/9457106)

- 网上有很多关于gifflen合成gif的文章 可以很轻松的get到C源代码，但是需要ndk进行编译 当然也可以用现成的so文件，但是需要将utils类建立在指定包下


## 一份合成好的so文件
- [so文件](http://download.csdn.net/detail/chendong_/9457110)

- 加载so文件时需要将jni接口写在指定的包下，目前com.xingye.gif



## 合成好的jar文件
- [jar文件下载](http://download.csdn.net/detail/chendong_/9457110)

- 结合so文件可以直接使用，不需要自己建包


## GitHub
- [详见该类](https://github.com/chendongMarch/cdlibs/blob/master/src/main/java/com/march/libs/utils/GifUtils.java)



## 总结
- java的库很简单，有那几个合成的类，定制一个工具类就好了，C的库比较繁琐，虽然效率还没有java快，总结一下就是
1. 自己用C代码合成so  
2. 用我合成的so,但是你需要按照编译好的con.xingye.gif包下的GifUtil 来执行代码  
3. 更简单，jar文件已经打包好，集成so文件和jar文件，编译可以使用,就是自定义不方便了。
