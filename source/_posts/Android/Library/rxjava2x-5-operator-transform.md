---
layout: post
title: Rxjava2.x开发-5-操作符-变换
categories:
  - Android
  - Library
tags:
  - Android
  - RxJava2.x
keywords:
  - Android
  - RxJava2.x
  - 操作符
  - 变换操作符
comments: true
abbrlink: cc39cbd0
date: 2017-07-11 
password:
---

本文以 `Observable` 为例，主要总结 `RxJava2.x` 关于 **变换** 操作相关操作符的用法。

<!--more-->

## Buffer

收集 `Observable` 的数据放进一个数据包裹，然后发射这些数据包裹，而不是一次发射一个值。`Buffer` 操作符将一个 `Observable` 变换为另一个，原来的`Observable` 正常发射数据，变换产生的 `Observable` 发射这些数据的缓存集合。 

注意：如果原来的 `Observable` 发射了一个 `onError` 通知，`Buffer` 会立即传递这个通知，而不是首先发射缓存的数据，即使在这之前缓存中包含了原始`Observable` 发射的数据。

我们创建一个 `Observable` 用来发送 0 ～ 49 的50个整数值，如下：

```java
List<Integer> integers = new ArrayList<>();
for (int i = 0; i < 50; i++) {
    integers.add(i);
}
Observable<Integer> observable = Observable.fromIterable(integers);
```

### buffer(count[,skip][,bufferSupplier])

```java
observable
        .buffer(10)
        .subscribe(new MyObserver<List<Integer>>("buffer1"));
```

