---
layout: post
title: RxJava2.x开发-1-基础
categories:
  - Android
  - Library
tags:
  - Android
  - RxJava2.x
keywords:
  - Android
  - RxJava
  - Observable
  - Flowable
comments: true
abbrlink: 3251baff
date: 2017-07-01 23:00:28
password:
---

开始总结记录 `RxJava` 的相关内容，本文中所有涉及 `RxJava` 的地方均指 `Rxjava2.0`。

本文主要以 `Observable` 和 `Flowable` 为例介绍如何创建被观察者和观察者，并连接他们。

> **[RxJava https://github.com/ReactiveX/RxJava](https://github.com/ReactiveX/RxJava)**

> **[RxJava https://github.com/ReactiveX/RxAndroid](https://github.com/ReactiveX/RxAndroid)**

<!--more-->

## 推荐文章

记录一下看过的几篇还不错的关于 `RxJava` 的文章，感谢他们的总结和分享。

[RxJava1 - 《 给 Android 开发者的 RxJava 详解 》- 扔物线)](https://gank.io/post/560e15be2dca930e00da1083)，这篇文章是对 `RxJava1` 的讲解，很不错，图文并茂，看了一遍之后仍旧有些疑惑，建议多看几遍，很多关于原理的介绍可以加强对 `Rx` 的理解。

[RxJava2 - 《 给初学者的RxJava2.0教程(一) 》- 掘金](https://juejin.im/post/5848d96761ff4b0058c9d3dc) 以及 [RxJava2 - 《 给初学者的RxJava2.0教程(二) 》- 掘金](https://juejin.im/post/5848dd11b123db0066030123)，特点就是作者使用水管上游下游的描述方式，简化了对观察者模式和事件发送的理解，浅显易懂，也很全面，适合入门。

[《 RxJava2.0 你不知道的事》- 简书](http://www.jianshu.com/p/785d9dfb0a5b) 是对 `RxJava1.x` 和 `RxJava2.x` 的对比，文章中对 `2.x`  的部分 `api`，进行了列举，对比起来看就清晰多了，另外很好的解释了背压的问题，受益匪浅。

[《关于RxJava最友好的文章》- 知乎](https://zhuanlan.zhihu.com/p/23584382)

[RxJava2.x APi 文档](http://reactivex.io/RxJava/2.x/javadoc/)

## 我的理解

上面推荐的文章中对 `RxJava` 的相关原理都做了部分说明，我就不做过多描述，说一下我的一些理解吧。

`RxJava` 最关键的两个点就是 **观察者模式** 和 **异步**。在 `RxJava` 中被观察者作为事件的产生方，是 _主动_ 的，是整个事件流程的起点。观察者作为事件的处理方，是 _被动_ 的，是整个事件流程的终点。在起点和终点之间，即事件传递的过程中是可以被加工，过滤，转换，合并等等方式处理的。

> `Observable`，被观察者，被订阅者，可被观察的，他是数据和事件发射的源，他在 `RxJava` 中是有多种实现方式，这里不是说的哪个类，而是一种泛指。

> `Observer／Subscriber`，观察者，订阅者，他是事件接受者。

整体来看，可以理解为一条事件流，被观察者在上游发送事件，观察者在下游接受事件，中游会有很多针对事件的处理和变换，这样理解更简单一些，也更有助于理解背压(`Backpressure`)的存在。为了更好的理解，避免叙述的混乱，文章中我会用 **上游** 和 **下游** 这样的描述来代替 **被观察者** 和 **观察者**。


## 使用 Observable

创建一个最简单的 `Observable`，发送 `Integer` 类型的数据，`RxJava` 有很多创建 `Observable` 的简单方法，我们暂时就使用最原始的那种，方便理解。

`ObservableEmitter` 继承 `Emitter`，是一个数据发射器，用来向观察者发送数据。

```java
// 创建 Observable
Observable<Integer> observable = Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
        // 发射数据
        e.onNext(1);
        e.onNext(2);
        // 结束发送
        e.onComplete();
        e.onNext(3);
    }
});
```

创建一个 `Observer`，接受数据并打印。

```java
Observer<Integer> observer = new Observer<Integer>() {
    @Override
    public void onSubscribe(@NonNull Disposable d) {
        log("onSubscribe");
    }
    @Override
    public void onNext(@NonNull Integer integer) {
        log("onNext == > " + integer);
    }
    @Override
    public void onError(@NonNull Throwable e) {
        log("onError");
    }
    @Override
    public void onComplete() {
        log("onComplete");
    }
};
```

实现订阅，查看结果。

```java
observable.subscribe(observer);

// 结果如下，3 不会被打印
---------------
[ThreadName:pool-13-thread-1] onSubscribe
[ThreadName:pool-13-thread-1] onNext == > 1
[ThreadName:pool-13-thread-1] onNext == > 2
[ThreadName:pool-13-thread-1] onComplete
---------------
```
这样我们就实现了一个简单的订阅流程，完成了数据的传递，总结以下要点：

`ObservableEmitter` 继承 `Emitter`，是一个发射器，用来向下游发送事件，它可以发送如下3种事件，也对应下游订阅者的相关方法。

```java
public interface Emitter<T> {
    void onNext(@NonNull T value);
    void onError(@NonNull Throwable error);
    void onComplete();
}
```
> 上游和下游的所有方法都默认运行在当前所在线程内，如上运行结果，我在子线程运行则所有方法会在子线程调用。

> 订阅发生在 `observable.subscribe(observer);` 时，此时上游才开始发送事件，并且 `onSubscribe()` 方法会在开始订阅时首先执行。

> 上游可以发送无数个 `onNext(T t)` 事件，下游都可以接受到。

> 当上游发送 `onComplete()` 事件之后，上游的事件会继续发送，但是下游在接受到 `onCompelete()` 事件之后就会切换事件流，不会在接受后续的事件，因此发送多个  `onComplete()` 虽然不会导致程序 crash，但是是无意义的。

> 当上游发送 `onError()` 事件之后，上游的事件会继续发送，但是下游在接受到 `onError()` 事件之后就会切换事件流，不会在接受后续的事件，当你发送第二个 `onError()` 事件时会导致程序 crash。

> 发送 `onComplete()` 和 `onError()` 事件不是必须的。

> `onComplete()` 和 `onError()` 是唯一且互斥的，你不能发送多个 `onComplete()` 或多个 `onError()`，也不能不能发送一个 `onCompelete()` 事件再发送 `onError()`，反过来也是。


## 中断事件流

`Disposable` 对象可用用来切断事件流，在 `onSubscribe()` 被调用时会返回 `Disposable` 对象，我们在获取到数字 `4` 时切断事件流。将上面的代码稍微简化一下

```java
// 创建 Observable
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
        // 发射数据
        e.onNext(1);
        e.onNext(2);
        e.onNext(3);
        e.onNext(4);
        e.onNext(5);
        // 结束发送
        e.onComplete();
        e.onNext(6);
    }
}).subscribe(new Observer<Integer>() {
    Disposable mDisposable;
    @Override
    public void onSubscribe(@NonNull Disposable d) {
        mDisposable = d;
        log("onSubscribe");
    }
    @Override
    public void onNext(@NonNull Integer integer) {
        log("onNext == > " + integer);
        if (integer == 4) {
            mDisposable.dispose();
        }
    }
    @Override
    public void onError(@NonNull Throwable e) {
        log("onError");
    }
    @Override
    public void onComplete() {
        log("onComplete");
    }
});
```
结果为

```bash
[ThreadName:pool-13-thread-1] onSubscribe
[ThreadName:pool-13-thread-1] onNext == > 1
[ThreadName:pool-13-thread-1] onNext == > 2
[ThreadName:pool-13-thread-1] onNext == > 3
[ThreadName:pool-13-thread-1] onNext == > 4
```


## 其他订阅方法

在实际应用过程中我们可能并不关注下游所有的接受事件的方法，因此 `RxJava` 提供了多种订阅方式来简化订阅过程。

这里说一下 `Action` 和 `Consumer`，与 `RxJava1.x` 不同，没有使用 `ActionN` 这种命名方式。

`Action` 是无参无返回值的接口，它可以用来替代类似 `onComplete()` 这种无参无返回值值的方法。

```
public interface Action {
    void run() throws Exception;
}
```

`Consumer` 是单个参数无返回值的接口，它可以用来代替类似 `onSubscribe(@NonNull Disposable d)`，`onNext(@NonNull Integer integer)`，`onError(@NonNull Throwable e)` 这类单个参数无返回值的方法。

```java
public interface Consumer<T> {
    void accept(@NonNull T t) throws Exception;
}
```
重载的订阅方法，除了以 `Observer` 的方式订阅之外，其他方法都返回 `Disposable` 对象用来中断事件流。

```java
// 下游不关注上游的任何事件
public final Disposable subscribe()

// 观察者
public final void subscribe(Observer<? super T> observer)

// 只关注 onNext() 事件
public final Disposable subscribe(Consumer<? super T> onNext) 

// 只关注 onNext() onError() 
public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError)

// 只关注 onNext() onError() onComplete()
public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError,Action onComplete)

// 只关注 onNext() onError() onComplete() onSubscribe()
public final Disposable subscribe(Consumer<? super T> onNext, Consumer<? super Throwable> onError,	Action onComplete, Consumer<? super Disposable> onSubscribe)
```

使用只关注 `onNext()` 事件的订阅方法实现订阅。

```java
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
        // 发射数据
        e.onNext(1);
        e.onNext(2);
        // 结束发送
        e.onComplete();
    }
}).subscribe(new Consumer<Integer>() {
    @Override
    public void accept(@NonNull Integer integer) throws Exception {
        log("use Consumer onNext = > " + integer);
    }
});
```

------

> 对 `Observable` 的介绍相对详细，下面的介绍会简单一些，因为很多相似的地方，就不再赘述了。

## 使用 Flowable

`Flowable` 与 `Observable` 的区别就是实现了 **背压(`Backpressure`)** 的管理，讲真，我对背压这个概念也理解的不是很全面，概括的讲背压就是上游发送的事件太多，下游处理事件的速度太慢，导致上游事件堆积，此时如何处理堆积的事件，就是背压处理的策略。

背压处理策略，在 `RxJava2.x` 中 `Observable` 不再支持背压，需要支持背压时需要使用 `Flowable` 创建被观察者，并要求明确指定背压处理策略。

关于 `Flowable` 和 `Backpressure` 的内容后面作单独研究，这里不展开。

```java
public enum BackpressureStrategy {
    ERROR,
    BUFFER,
    DROP,
    LATEST
}
```
实现一个订阅，支持背压时，需要在下游调用 `request(long n)` 来向上游请求，自己要多少数据，请求多少数据上游就会发多少数据过来，如下实例中，只会获取到一次事件。

```java
Flowable.create(new FlowableOnSubscribe<Integer>() {
    @Override
    public void subscribe(@NonNull FlowableEmitter<Integer> e) throws Exception {
        for (int i = 0; i < 10; i++) {
            e.onNext(i);
        }
        e.onComplete();
    }
}, BackpressureStrategy.BUFFER).subscribe(new FlowableSubscriber<Integer>() {
    @Override
    public void onSubscribe(Subscription s) {
        log("onSubscribe");
        s.request(1);
    }
    @Override
    public void onNext(Integer integer) {
        log("onNext == > " + integer);
    }
    @Override
    public void onError(Throwable t) {
        log("onError");
    }
    @Override
    public void onComplete() {
        log("onComplete");
    }
});
```

### 中断事件流

`Subscription` 类似于 `Observable` 中的 `Disposable`，可以用来中断事件流，不同的是需要使用 `cancel()` 方法，另外有另一个 `request(long n)` 方法用来向上游请求数据。

```java
public interface Subscription {
    public void request(long n);
    public void cancel();
}
```
如下，使用 `Subscription` 中断事件。

```java
Flowable.create(new FlowableOnSubscribe<Integer>() {
    @Override
    public void subscribe(@NonNull FlowableEmitter<Integer> e) throws Exception {
        for (int i = 0; i < 10; i++) {
            e.onNext(i);
        }
        e.onComplete();
    }
}, BackpressureStrategy.BUFFER).subscribe(new FlowableSubscriber<Integer>() {
    
    Subscription mSubscription;
    
    @Override
    public void onSubscribe(Subscription s) {
        log("onSubscribe");
        mSubscription = s;
        s.request(1);
    }
    @Override
    public void onNext(Integer integer) {
        log("onNext == > " + integer);
        mSubscription.request(5);
        if (integer == 5) {
            mSubscription.cancel();
        }
    }
    // ... 
});
```

### 关于 request

下游请求多少就会收到多少事件，但是不会阻塞上游事件发送的过程，上游的事件会一直发，但是下游没请求的话接受不到事件。

`request(long n)` 中的数量是会累加的，累加的数量就是请求的总量，如果请求的总量超过了发送的总量，则上游事件会被全部接受到，但是不会多出来。

如下实例中，总共请求了 `1 + 2 + 2 = 5` 次事件，因此只收到了 5 次事件，但是上游的事件发送并没有停止。 

```java
Flowable.create(new FlowableOnSubscribe<Integer>() {
    @Override
    public void subscribe(@NonNull FlowableEmitter<Integer> e) throws Exception {
        for (int i = 0; i < 10; i++) {
            log("send next " + i);
            e.onNext(i);
        }
        e.onComplete();
    }
}, BackpressureStrategy.BUFFER).subscribe(new FlowableSubscriber<Integer>() {
    Subscription mSubscription;
    @Override
    public void onSubscribe(Subscription s) {
        log("onSubscribe");
        mSubscription = s;
        s.request(1);
    }
    @Override
    public void onNext(Integer integer) {
        log("onNext == > " + integer);
        if(integer == 0 || integer == 1) {
            mSubscription.request(2);
        }
    }
    // ...
});
```

输出结果

```bash
[ThreadName:pool-13-thread-1] onSubscribe
[ThreadName:pool-13-thread-1] send next 0
[ThreadName:pool-13-thread-1] onNext == > 0
[ThreadName:pool-13-thread-1] send next 1
[ThreadName:pool-13-thread-1] onNext == > 1
[ThreadName:pool-13-thread-1] send next 2
[ThreadName:pool-13-thread-1] onNext == > 2
[ThreadName:pool-13-thread-1] send next 3
[ThreadName:pool-13-thread-1] onNext == > 3
[ThreadName:pool-13-thread-1] send next 4
[ThreadName:pool-13-thread-1] onNext == > 4
[ThreadName:pool-13-thread-1] send next 5
[ThreadName:pool-13-thread-1] send next 6
[ThreadName:pool-13-thread-1] send next 7
[ThreadName:pool-13-thread-1] send next 8
[ThreadName:pool-13-thread-1] send next 9
```

### 其他订阅方法

`Flowable` 跟 `Observable` 一样对订阅操作也有很多重载方法，可以参照[Obervable#其他订阅方法](#其他订阅方法)。


## 更多被观察者实现

我现在还不太清楚它们之间的关系，就先列举一下 `API`，后面有机会再仔细看看

`Maybe`

```java
Maybe.create(new MaybeOnSubscribe<Integer>() {
    @Override
    public void subscribe(@NonNull MaybeEmitter<Integer> e) throws Exception {
    }
}).subscribe(new MaybeObserver<Integer>() {
    @Override
    public void onSubscribe(@NonNull Disposable d) {
    }
    @Override
    public void onSuccess(@NonNull Integer integer) {
    }
    @Override
    public void onError(@NonNull Throwable e) {
    }
    @Override
    public void onComplete() {
    }
});
```

`Completable`

```java
Completable.create(new CompletableOnSubscribe() {
    @Override
    public void subscribe(@NonNull CompletableEmitter e) throws Exception {
    }
}).subscribe(new CompletableObserver() {
    @Override
    public void onSubscribe(@NonNull Disposable d) {
    }
    @Override
    public void onComplete() {
    }
    @Override
    public void onError(@NonNull Throwable e) {
    }
});
```

`Single`

```java
Single.create(new SingleOnSubscribe<Object>() {
    @Override
    public void subscribe(@NonNull SingleEmitter<Object> e) throws Exception {
    }
}).subscribe(new SingleObserver<Object>() {
    @Override
    public void onSubscribe(@NonNull Disposable d) {
    }
    @Override
    public void onSuccess(@NonNull Object o) {
    }
    @Override
    public void onError(@NonNull Throwable e) {
    }
});
```
