---
layout: post
title: Rxjava2.x开发-4-操作符-创建
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
  - 创建操作符
comments: true
abbrlink: 6cad82c9
date: 2017-07-10 15:51:09
password:
---

本文以 `Observable` 为例，本文主要总结 `RxJava2.x` 关于 **创建** 操作相关操作符的用法。

<!--more-->

## Create 

`Create` 是最基本的创建操作符，他用来创建一个标准的被观察者，然后恰当的调用观察者的 `onNext`，`onError` 和 `onCompleted` 方法。 一个形式正确的有限 `Observable` 必须尝试调用观察者的 `onCompleted` 正好一次或者它的 `onError` 正好一次，而且此后不能再调用观察者的任何其它方法。

好的做法是在数据发射之前判断观察者的状态，在没有观察者时不进行事件发送和计算操作。`Create` 操作符不在任何线程调度器上执行。

```java
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
        if(!e.isDisposed()) {
            e.onNext(1);
            e.onComplete();
        }
    }
});
```



## Just

`Just` 类似于 `From`，但是 `From` 会将数组或 `Iterable` 的数据取出然后逐个发射，而 `Just` 只是简单的原样发射，将数组或 `Iterable` 当做单个数据，如下情况，将会直接发送一个 `List` 出去，而不是里面的数字 `0`;

```java
List<Integer> integers = new ArrayList<>();
integers.add(0);
Observable.just(integers);
```

`Just` 它最多接受 10 个参数，返回一个按参数列表顺序发射这些数据的 `Observable`。从 `RxJava2.x` 开始，使用 `just` 不允许传递 `null`，否则会出现异常(NPE)


```java
Observable.just(0);
Observable.just(0, 1);
Observable.just(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
        .subscribe(new Consumer<Integer>() {
            @Override
            public void accept(@NonNull Integer o) throws Exception {
            }
        });
```

## From

在 `RxJava1.x` 时，只有一个 `From` 操作符，接受不同类型的参数，但是在 `RxJava2.x` 对这个操作符进行了细分。



### fromIterable 和 fromArray

`from` 操作符可以转换 **数组** 和 **Iterable**，产生的 `Observable` 会发射 **数组** 和 **Iterable** 的每一项数据。

```java
List<Integer> integers = new ArrayList<>();
integers.add(0);
Observable.fromIterable(integers);
```

```java
int[] array = new int[]{1,2,3};
Observable.fromArray(array);
Observable.fromArray(1,2,3);
```

### fromCallable 和 fromFuture

todo... `Callable` 和 `Future` 都是 `java.util.concurrent` 包里面的类，具体使用方法暂时不清楚，后面补充。

`fromCallable()` 返回的是 `onNext` 传递的数据，`fromCallable()` 获取要发送的数据的代码只会在有 `Observer` 订阅之后执行，且获取数据的代码可以在子线程中执行。

```java
Observable.fromCallable(new Callable<Integer>() {
    @Override
    public Integer call() throws Exception {
        return 100 + 2;
    }
});
```

```java
public static <T> Observable<T> fromFuture(Future<? extends T> future)
public static <T> Observable<T> fromFuture(Future<? extends T> future, long timeout, TimeUnit unit)
public static <T> Observable<T> fromFuture(Future<? extends T> future, long timeout, TimeUnit unit)
public static <T> Observable<T> fromFuture(Future<? extends T> future, long timeout, TimeUnit unit, Scheduler scheduler)
```


### fromPublisher

todo...暂时不是很清楚它的用法， 但是 `Flowable` 实现了 `Publisher` 接口，可以使用该方法将 `Flowable` 转换为 `Observable`

```java
Flowable<Integer> integerFlowable = Flowable.create(new FlowableOnSubscribe<Integer>() {
    @Override
    public void subscribe(@NonNull FlowableEmitter<Integer> e) throws Exception {
    }
}, BackpressureStrategy.BUFFER);
Observable.fromPublisher(integerFlowable);
```

## Defer

直到有观察者订阅时才创建 `Observable`，并且为每个观察者创建一个新的`Observable`。

`Defer` 操作符会一直等待直到有观察者订阅它，然后它使用 `Observable` 工厂方法生成一个 `Observable`。它对每个观察者都这样做，因此尽管每个订阅者都以为自己订阅的是同一个 `Observable`，事实上每个订阅者获取的是它们自己的单独的数据序列。

```java
Observable<Object> defer = Observable.defer(new Callable<ObservableSource<?>>() {
    @Override
    public ObservableSource<?> call() throws Exception {
        return Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exceptio
                RxHelper.log("发射数据 --> " + hashCode());
                e.onNext(1);
                e.onComplete();
            }
        });
    }
});
defer.subscribe(new MyObserver<>());
defer.subscribe(new MyObserver<>());
```


## Empty/Never/Throw

`empty` 创建一个不发射任何数据但是正常终止的 `Observable`;

`never` 创建一个不发射数据也不终止的 `Observable`;

`throw` 创建一个不发射数据以一个错误终止的 `Observable`;

```java
Observable.empty();

Observable.never();

Observable.error(new Callable<Throwable>() {
    @Override
    public Throwable call() throws Exception {
        return new RuntimeException("test");
    }
});
Observable.error(new RuntimeException("test"));
```

## Interval 和 Range

默认在 `computation` 调度器上执行，你也可以传递一个可选的`Scheduler` 参数来指定调度器。

### interval

`interval` 返回一个以固定时间间隔发送无限递增的 `Long` 型数列的 `Observable`。

```java
public static Observable<Long> interval(long period, TimeUnit unit)
public static Observable<Long> interval(long initialDelay, long period, TimeUnit unit)
public static Observable<Long> interval(long period, TimeUnit unit, Scheduler scheduler)
public static Observable<Long> interval(long initialDelay, long period, TimeUnit unit, Scheduler scheduler)

// 延迟 1s，间隔 100ms，发送无限增长的 Long 型数列
Observable.interval(1, 100, TimeUnit.MILLISECONDS)
        .subscribe(new MyObserver<Long>("interval"));
```

### intervalRange

`intervalRange` 类似于 `interval`，但是它可以指定起始数值，而且不再是一个无限数列，需要注意的是，假设起始值为 start = a，count = b，即为从 start 开始，发送 count 个数据，那么发送的区间是 [a,a+b)，左闭右开，例如 start = 100，count = 120，区间是 [100,220)，即 100 ～ 219，另外 `count` 不能为负数，否则会异常。

```java
public static Observable<Long> intervalRange(long start, long count, long initialDelay, long period, TimeUnit unit)
public static Observable<Long> intervalRange(long start, long count, long initialDelay, long period, TimeUnit unit, Scheduler scheduler)

// 延迟 1s，间隔 100ms，发送从100 开始的递增数列，发送的区间是 [100,100+120)
Observable.intervalRange(100, 120, 1, 100, TimeUnit.MILLISECONDS)
        .subscribe(new MyObserver<Long>("intervalRange"));
```

### range 和 rangeLong

从 start 开始发送 count 个数据，区间为 [start,start+count)

```java
Observable.range(100,50).subscribe(new MyObserver<Integer>("range"));

Observable.rangeLong(100,50).subscribe(new MyObserver<Long>("rangeLong"));
```


## Repeat

`repeat` 默认在 `trampoline` 调度器执行。


### repeat

重复发送 `Observable` 的数据，如果不指定数目，将会无限发送。

```java
Observable.just(1).repeat(10)
        .subscribe(new MyObserver<Integer>("repeat1"));
        
Observable.just(1).repeat()
        .subscribe(new MyObserver<Integer>("repeat2"));
```

### repeatWhen

todo...对这个运算符不是很清楚它的作用

它不是缓存和重放原始 `Observable` 的数据序列，而是有条件的重新订阅和发射原来的 `Observable`。

将原始 `Observable` 的终止通知（完成或错误）当做一个 `void` 数据传递给一个通知处理器，它以此来决定是否要重新订阅和发射原来的 `Observable`。这个通知处理器就像一个 `Observable` 操作符，接受一个发射 `void` 通知的 `Observable` 为输入，返回一个发射 `void` 数据（意思是，重新订阅和发射原始 `Observable`）或者直接终止（意思是，使用 `repeatWhen` 终止发射数据）的`Observable`。

```java
Observable.just(1,2,4).repeatWhen(new Function<Observable<Object>, ObservableSource<?>>() {
    @Override
    public ObservableSource<?> apply(@NonNull Observable<Object> objectObservable) throws Exception {
        RxHelper.log("apply");
        return objectObservable;
    }
})
        .subscribeOn(Schedulers.computation())
        .subscribe(new MyObserver<Integer>("repeatWhen"));
```


## Start

发送事件之前先发送某些事件。

```java
// 接收单个值
Observable.just(1, 2, 4).startWith(100)
        .subscribe(new MyObserver<Integer>("startWith1"));

// 接收 iterable
List<Integer> ints = new ArrayList<>();
ints.add(100);
ints.add(200);
Observable.just(1, 2, 4).startWith(ints)
        .subscribe(new MyObserver<Integer>("startWith2"));

// 接收 array
Observable.just(1, 2, 4).startWithArray(100, 200)
        .subscribe(new MyObserver<Integer>("startWith3"));
// 接收 array
Integer[] intArray = new Integer[]{100, 200};
Observable.just(1, 2, 4).startWithArray(intArray)
        .subscribe(new MyObserver<Integer>("startWith4"));

// 接收 observable
Observable.just(1, 2, 4).startWith(Observable.just(100, 200))
        .subscribe(new MyObserver<Integer>("startWith5"));
```


## Timer

在一定时间之后发送一个特殊值 0，`timer` 操作符默认在 `computation` 线程执行。

```java
public static Observable<Long> timer(long delay, TimeUnit unit)
public static Observable<Long> timer(long delay, TimeUnit unit, Scheduler scheduler)

Observable.timer(10, TimeUnit.MILLISECONDS).subscribe(new MyObserver<Long>("timer"));
```