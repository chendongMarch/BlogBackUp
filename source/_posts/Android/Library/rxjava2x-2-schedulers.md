---
layout: post
title: RxJava2.x开发-2-Schedulers
categories:
  - Android
  - Library
tags:
  - Android
  - RxJava2.x
keywords:
  - Android
  - RxJava
  - 线程调度
  - Schedulers
comments: trues
abbrlink: 8de84f35
date: 2017-07-03 17:54:28
password:
---

本文主要介绍 `RxJava2.x` 强大的线程调度。

在 `Android` 开发中因为不允许阻塞主线程，所以所有的耗时请求都必须全部放在子线程来做，然后再去主线程更新UI，关于主线程和子线程的通信其实异常复杂，好在`Android` 给我们提供了 `AsyncTask`，`Handler` 等方式来简化这一过程。使用 `RxJava` 会让切换线程变得更简单。

文中部分描述可能有些混乱，为了更好的看出在哪个线程调用，我会在子线程中执行我的代码，我就称它为 `MyThread`，也就是调用代码所在的线程。io线程，计算线程，newThread线程就是我对 `RxJava` 几种内置线程的简称。上游线程就是被观察者所在的线程，下游线程是观察者所在线程，调用线程就是我调用代码的线程 `MyThread`。这里简单理一下，虽然还是有点乱。

<!--more-->


## Schedulers 种类

`RxJava` 根据不同应用场景内置了多种线程调度器，可以大多数场景的后台操作需求。

Schedulers|Desc|
---|---|
Schedulers.computation()|	用于计算任务，如事件循环或和回调处理，不要用于IO操作，默认线程数等于处理器的数量
Schedulers.io()|	用于IO密集型任务，如异步阻塞IO操作，这个调度器的线程池会根据需要增长；对于普通的计算任务，请使用Schedulers.computation() 
Schedulers.newThread()|	为每个任务创建一个新线程
Schedulers.trampoline()|当其它排队的任务完成后，在当前线程排队开始执行
Schedulers.from(executor)	|使用指定的 Executor 作为调度器
AndroidSchedulers.mainThread()| Android 主线程
AndroidSchedulers.from(looper) | 从 Looper 创建

## 线程调度

最为关键的就是两个方法 `subscribeOn()` 和 `observeOn()`，从代码的链式调用可以简单的总结为：

> 上游 `Observable` 总是默认运行在被调用的线程当中，即你在哪个线程调用就会运行在哪个线程。

> 下游 `Onserver` 总是默认运行在上游所在线程中_(当然如果你没有切换上游的线程，那么下游也会运行在调用的线程中)_，除非使用 `observeOn()` 进行线程的切换。

> `subscribeOn()` 用来声明上游事件发送时的所在线程，当调用多次 `subscribeOn()` 时，上游会运行在最早的一次调用声明的线程中。当然也不是说多次的调用是完全没效果的，后面会细说。

> `observeOn()` 用来声明下游观察者所在线程，每次调用 `observeOn()` 都会发生线程切换，此次调用直到下次切换线程中间的过程中的操作运行在此次调用指定的线程中。


```java
log("start");
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exceptio
        log("Observable");
        e.onNext(10);
    }
})
        // 第1次调用subscribeOn， Observable 将运行在io线程
        .subscribeOn(Schedulers.io())
        // io 线程，下游总是会默认运行在上游所在线程中
        .filter(new Predicate<Integer>() {
            @Override
            public boolean test(@NonNull Integer integer) throws Exception {
                log("filter1");
                return true;
            }
        })
        // 第2次调用subscribeOn，不会生效
        .subscribeOn(Schedulers.newThread())
        // 第1次调用observeOn，切换线程，filter操作运行在主线程
        .observeOn(AndroidSchedulers.mainThread())
        .filter(new Predicate<Integer>() {
            @Override
            public boolean test(@NonNull Integer integer) throws Exception {
                log("filter2");
                return true;
            }
        })
        // 第2次调用observeOn，切换线程到子线程
        .observeOn(Schedulers.newThread())
        .subscribe(new Consumer<Integer>() {
            @Override
            public void accept(@NonNull Integer integer) throws Exception {
                log("subscribe = " + integer);
            }
        });
```

输出结果,我整个代码运行在一个我自己的子线程中，我就叫他 MyThread 好了，方便描述

```java 
[ThreadName:pool-13-thread-1] start // 因为在 MyThread 子线程调用
[ThreadName:RxCachedThreadScheduler-1] Observable // io
[ThreadName:RxCachedThreadScheduler-1] filter1 // io.由于没有切换线程，默认运行在上游线程中。
[ThreadName:main] filter2 // 切换到主线程
[ThreadName:RxNewThreadScheduler-2] subscribe = 10 // 切换到 newThread 线程
```

## doOnSubscribe()

当调用多次 `subscribeOn()` 方法时，上游将运行在最早调用指定的线程中，这个没什么问题，对于下游的 `doOnNext()`，`doOnComplete()` 来说，遵循上面说的规则，除非你使用 `observerOn()` 切换线程，不然运行在上游线程中。

但是 `doOnSubscribe()` 有点不一样，除非他后面有调用 `subscribeOn()` 切换线程，否则他默认运行在执行 `Observable.subcribe()` 语句的线程中。

其实这里的 `doXXX()` 方法和上一篇文章中观察者中的几个方法是一一对应的，在观察者的中 `onSubscribe()` 方法也有同样的属性，他在订阅发生的一瞬间首先执行，并且它运行在订阅发生的线程。其他几个方法也是一样，除非你使用 `observerOn()` 切换线程，不然运行在上游线程中。

```java
log("start");
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
        log("Observable");
        e.onNext(10);
        e.onComplete();
    }
})
        // 计算线程，因此 Observable 将会运行在计算线程
        .subscribeOn(Schedulers.computation())
        .observeOn(Schedulers.newThread())
        .doOnNext(new Consumer<Integer>() {
            @Override
            public void accept(@NonNull Integer integer) throws Exception {
                log("doOnNext");
            }
        })
        .doOnSubscribe(new Consumer<Disposable>() {
            @Override
            public void accept(@NonNull Disposable disposable) throws Exception {
                // 后面没有调用 subscribeOn() 因此将运行在 Observable.subscribe() 执行的线程
                log("doOnSubscribe");
            }
        })
        .doOnComplete(new Action() {
            @Override
            public void run() throws Exception {
                log("doOnComplete");
            }
        })
        .observeOn(Schedulers.newThread())
        // doOnSubscribe() 运行的线程取决于在哪里执行订阅
        // 除非后面有调用 subscribeOn() 进行线程的切换
        .subscribe(new Consumer<Integer>() {
            @Override
            public void accept(@NonNull Integer integer) throws Exception {
                log("subscribe = " + integer);
            }
        });
```

输出结果，我整个代码运行在一个我自己的子线程中，我就叫他 MyThread 好了，方便描述
 
```java
// MyThread 子线程运行
[ThreadName:pool-16-thread-1] start 
// 因为 subscribe() 方法也同样运行在 MyThread 子线程，所以 doSunscribe() 页运行在 MyThread 子线程
[ThreadName:pool-16-thread-1] doOnSubscribe 
// 由于调用 subscribeOn(Schedulers.computation()) 所以上游运行在计算线程
[ThreadName:RxComputationThreadPool-1] Observable 
// 遵循上面说的规则，运行在 observerOn() 指定的 newThread 中。
[ThreadName:RxNewThreadScheduler-1] doOnNext
// 同上 
[ThreadName:RxNewThreadScheduler-1] doOnComplete    
[ThreadName:RxNewThreadScheduler-2] subscribe = 10 
```

### 切换 doOnSubscribe() 所在线程

再换一个更改 `doOnSubscribe()` 运行线程的例子，跟上面不同的是在 `doOnSubscribe()` 之后我们使用 `subscribeOn(Schedulers.io())` 切换了线程，因此 `doOnSubscribe()` 将运行在 `io` 线程。


```java
log("start");
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
        log("Observable");
        e.onNext(10);
        e.onComplete();
    }
})
        // 计算线程，因此 Observable 将会运行在计算线程
        .subscribeOn(Schedulers.computation())
        .observeOn(Schedulers.newThread())
        .doOnSubscribe(new Consumer<Disposable>() {
            @Override
            public void accept(@NonNull Disposable disposable) throws Exception {
                // doOnSubscribe 之后第一次调用 subscribeOn(Schedulers.io()) 切换到了io线程
                // 因此 doOnSubscribe 运行在 io 线程。
                log("doOnSubscribe");
            }
        })
        .subscribeOn(Schedulers.io())
        .observeOn(Schedulers.newThread())
        // doOnSubscribe() 运行的线程取决于在哪里执行订阅
        // 除非后面有调用 subscribeOn() 进行线程的切换
        .subscribe(new Consumer<Integer>() {
            @Override
            public void accept(@NonNull Integer integer) throws Exception {
                log("subscribe = " + integer);
            }
        });
```

运行结果

```java
// MyThread
[ThreadName:pool-16-thread-1] start
// 由于后面调用了 subscribeOn(io) 因此运行在 io 线程
[ThreadName:RxCachedThreadScheduler-1] doOnSubscribe
// 运行在上游线程中
[ThreadName:RxComputationThreadPool-1] Observable
// 使用 observeOn(Schedulers.newThread()) 切换到了 newThread
[ThreadName:RxNewThreadScheduler-2] subscribe = 10
```

### 举个🌰

有啥用呢？模拟一个场景，1⃣️上游发送网络请求，要求在子线程执行，2⃣️ 但是请求刚开始的时候我们要显示 `dialog` 提示用户等待，需要在主线程执行，3⃣️ 完成之后又要在子线程处理数据，4⃣️ 然后去主线程更新UI。

```java
log("开始操作，为了搞复杂点，我在我自己创建子线程操作");
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
        log("子线程发起网络请求");
        e.onNext(10);
        e.onComplete();
    }
})
        // 在子线程发起请求
        .subscribeOn(Schedulers.newThread())
        .doOnSubscribe(new Consumer<Disposable>() {
            @Override
            public void accept(@NonNull Disposable disposable) throws Exception {
                log("请求之前，主线程弹起dialog");
            }
        })
        // 切换到主线程弹 dialog
        .subscribeOn(AndroidSchedulers.mainThread())
        // 切换到计算线程 处理数据
        .observeOn(Schedulers.computation())
        .filter(new Predicate<Integer>() {
            @Override
            public boolean test(@NonNull Integer integer) throws Exception {
                log("计算线程处理数据");
                return true;
            }
        })
        // 切换到主线程更新UI
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(new Consumer<Integer>() {
            @Override
            public void accept(@NonNull Integer integer) throws Exception {
                log("主线程更新UI");
            }
        });
```

输出结果

```java
[ThreadName:pool-13-thread-1] 开始操作，为了搞复杂点，我在我自己创建子线程操作
[ThreadName:main] 请求之前，主线程弹起dialog
[ThreadName:RxNewThreadScheduler-1] 子线程发起网络请求
[ThreadName:RxComputationThreadPool-1] 计算线程处理数据
[ThreadName:main] 主线程更新UI
```

## 总之，如何判断到底在什么线程运行

综上所述，总结几条公式，其实就是把上面说的再说一遍：

几个概念 ：

> **上游**，上游指的是 数据发射源，`Observable` 中的代码。

> **下游**，下游指的是 操作符方法，`Observable` 调用方法，观察者中的方法 

> **调用线程**，我在哪个线程运行该代码，这个线程就是调用线程。

1⃣️、上游总是默认运行在被 `调用线程` 当中，即你在哪个线程调用就会运行在哪个线程。

2⃣️、 下游总是默认运行在上游所在线程中 _(当然如果你没有切换上游的线程，那么下游也会运行在 `调用线程` 中)_ ，除非使用 `observeOn()` 进行线程的切换。

3⃣️、 `subscribeOn()` 用来声明上游事件发送时的所在线程，当调用多次 `subscribeOn()` 时，上游会运行在 **最早** 的一次调用声明的线程中。

4⃣️、 `observeOn()` 用来声明下游观察者所在线程，每次调用 `observeOn()` 都会发生线程切换，此次切换 到 下次切换 之间运行在 此次切换 的线程中。 

5⃣️、 对于 `doOnNext()/onNext()`，`doOnComplete()/onComplete()`，`doOnError()/onError()` 几个方法 _(前者是 **被观察者** 调用的方法，后者是 **观察者** 接口里面的对应方法)_ ，他们都和操作符一样，遵循 2⃣️ 中的规则。

6⃣️、对于 `doOnSubscribe()/onSubscribe()` 方法 _(前者是 **被观察者** 调用的方法，后者是 **观察者** 接口里面的对应方法)_ 来说，如果他后面有调用 `subscribeOn()` 切换线程，那么它运行在切换的线程，否则他默认运行在执行 `Observable.subcribe()` 语句的线程中。


## 为什么是这样

上面只是总结一些规则，一个方法是运行在什么线程，使用上面的规则可以更简单判断出来，我们其实是在通过一些表象来总结如何判断，那为什么会是这样的规则呢？我现在也还没研究明白，，，，，，所以，待办 ～

ok了～关于 `RxJava2.x` 的一些源码分析见[RxJava2.x开发 (源码解析)](../7ae7177b)