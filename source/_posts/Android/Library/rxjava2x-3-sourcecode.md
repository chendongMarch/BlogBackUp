---
layout: post
title: RxJava2.x开发-3-源码分析
categories:
  - Android
  - Library
tags:
  - Android
  - RxJava2.x
keywords:
  - Android
  - RxJava2.x
  - 源码分析
comments: trues
abbrlink: 7ae7177b
date: 2017-07-05 09:54:28
password:
---

> 知其然(知道轮子是怎么用的)，知其所以然(也要知道轮子是怎么造的)。

本文主要介绍 `RxJava2.x` 是如何通过流式API完成事件的传递和变换的，我们不是要全部把它弄的清清楚楚，那需要大量的时间和不断深入才可以，只是通过简单的例子来理解他的基本工作原理和主要功能。

看了很多文章，文章中会讲代理模式什么的，但我觉得更像是包装者模式，可能我理解有偏差😭，但是我觉得这样更好理解一些，我就先按照我的理解来写，后面不对再修正好了。

文中源码我会去掉错误检查和注解的部分代码，只保留核心代码，看起来更清晰。

<!--more-->

## 简单的订阅

看一个最最简单的例子

```java
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
        e.onNext(1);
    }
}).subscribe(new Observer<Integer>() {
    @Override
    public void onSubscribe(@NonNull Disposable d) {
        log("onSubscribe");
    }
    @Override
    public void onNext(@NonNull Integer integer) {
        log("onNext - > " + integer);
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

这可能是最最基本的用法了，然后看一下源码，被观察者是如何向观察者发送数据的，首先是创建被观察者的过程。

```java
// Observable.java

public static <T> Observable<T> create(ObservableOnSubscribe<T> source) {
    ObjectHelper.requireNonNull(source, "source is null");
    return RxJavaPlugins.onAssembly(new ObservableCreate<T>(source));
}
```

`ObservableOnSubscribe` 是个接口，里面只有一个 `subscribe()` 方法，而我们就是在这个方法中进行事件的发送的。源码中可以看见实际上创建了一个 `ObservableCreate` 返回了，这个 `ObservableCreate` 其实是
`Observable` 的子类，他是一个被观察者的具体实现，因为是内部自己创建的所以叫 `ObservableCreate`，我猜的。整个过程就创建了一个 `ObservableCreate` 并返回了回来，然后怎么就可以继续链式调用了，比如在上面的简单例子中，调用了 `subscirbe()` 方法，这也是最后将调用的方法。

我们就上面的简单例子来继续看 `subscribe()` 发生的一刻发生了什么，我去掉了各种检查错误的代码，只保留核心代码。

```java
// Observable.java

public final void subscribe(Observer<? super T> observer) {
	subscribeActual(observer);
}
```
没错，就剩了一句了，`subscribeActual()` 是一个抽象方法，我们还要去看一下子类的具体实现，上面我们发现创建的 `Observable` 的子类是 `ObservableCreate`，那我们肯定是要去看这个子类了。

```java
// ObservableCreate.java

public final class ObservableCreate<T> extends Observable<T> {
    
    final ObservableOnSubscribe<T> source;

    public ObservableCreate(ObservableOnSubscribe<T> source) {
        this.source = source;
    }

    @Override
    protected void subscribeActual(Observer<? super T> observer) {
        CreateEmitter<T> parent = new CreateEmitter<T>(observer);
        observer.onSubscribe(parent);
        source.subscribe(parent);
    }
}
```

还是只看重点的，`final ObservableOnSubscribe<T> source;` 就是我们创建 `Observable` 时传进去的接口类，他只有一个 `subscribe()` 方法，再来仔细看一下 `subscribeActual()` 的实现，参数自然是要订阅过来的观察者 `Observer`，然后可以看到 `Observer` 被包装成了一个 `CreateEmitter` 发射器，我们知道他是用来发射事件的，那我们再看一眼发射器，不细看，他是一个静态内部类，看一下类声明即可

```java
static final class CreateEmitter<T>
    extends AtomicReference<Disposable>
    implements ObservableEmitter<T>, Disposable{}
```

接着上面的说，然后将发射器作为参数执行了 `ObservableOnSubscribe` 的 `subscribe()` 方法，然后会怎么样？自然是执行我们自己实现的 `ObservableOnSubscribe` 的 `subscribe()` 方法，开始使用发射器发送事件。其实我们是调用发射器，发射器调用他包装的 `Observer` 中对应的方法而已。

插一句，我们注意到当订阅发生时，首先执行了 `observer.onSubscribe(parent);` 方法，这也就是为啥观察者中的方法为什么会首先被调用，而 `CreateEmitter` 是实现了 `Disposable` 接口的，它可用来切断事件流。

> **总结，当观察者被订阅到被观察者时，被观察者被包装成一个发射器，调用 `subscribe()` 方法，使用发射器发射事件，被观察者收到事件。**


## 操作符

在实际应用过程中我们可能经历无数次操作符的变化，但是为了简化分析的过程，我们只看使用了一个操作符的例子，然后类推一下，其实多个变化也是一样的。

看一个使用操作符的场景

```java
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
        e.onNext(1);
    }
}).map(new Function<Integer, String>() {
    @Override
    public String apply(@NonNull Integer integer) throws Exception {
        return String.valueOf(integer);
    }
}).subscribe(new Consumer<String>() {
    @Override
    public void accept(@NonNull String s) throws Exception {
        log("accept -> " + s);
    }
});
```
在上个问题的基础上，我们以 `map` 操作符为例看一下，使用操作符的场景是如何建立连接，发送事件的，直接来看 `map` 方法，在 `map` 方法中创建了一个 `ObservableMap` 返回。

```java
// Observable.java

public final <R> Observable<R> map(Function<? super T, ? extends R> mapper) {
    ObjectHelper.requireNonNull(mapper, "mapper is null");
    return RxJavaPlugins.onAssembly(new ObservableMap<T, R>(this, mapper));
}
```
这个 `ObservableMap` 也是 `Observer` 的子类，也就是说它也是一个被观察者，参数是当前的被观察者和一个 `Function`，我们再来看一下 `ObservableMap`。

```java
public final class ObservableMap<T, U> extends AbstractObservableWithUpstream<T, U> {
    final Function<? super T, ? extends U> function;

    public ObservableMap(ObservableSource<T> source, Function<? super T, ? extends U> function) {
        super(source);
        this.function = function;
    }

    @Override
    public void subscribeActual(Observer<? super U> t) {
        source.subscribe(new MapObserver<T, U>(t, function));
    }


    static final class MapObserver<T, U> extends BasicFuseableObserver<T, U> {
        final Function<? super T, ? extends U> mapper;

        MapObserver(Observer<? super U> actual, Function<? super T, ? extends U> mapper) {
            super(actual);
            this.mapper = mapper;
        }

        @Override
        public void onNext(T t) {
            U v = mapper.apply(t)
            actual.onNext(v);
        }
    }
}
```

`ObservableMap` 包装了原来的 `SourceObservable`，也接受了我们进行 `map` 操作的 `Function`。当订阅发生时，仍旧会调用 `subscribeActual()` 方法，在这个方法中，我们把最终的观察者包装成了一个 `MapObserver`，然后把这个 `MapObserver` 订阅到了 `SourceObservable`。结合上面简单订阅时的分析，此时，`SourceObservable` 便会将 `MapObserver` 包装成一个发射器，开始发射事件了。再关注一下 `onNext()` 方法，这里会先调用 `Function` 进行 `map` 操作。


> **总结：我们称最开始创建的被观察者为 `SourceObservable`，如果中间增加一个 `map` 操作符，此时创建并且返回了一个 `ObservableMap` 包装 `SourceObservable` 作为新的被观察者，此时链式调用的对外开放的就是 `ObservableMap` 了， 当最终的观察者，我们叫他 `finalObserver`，被订阅到 `ObservableMap` 时，会将`finalObserver` 包装成一个 `MapObserver` _(这个 `MapObserver` 在调用 `onNext()` 时会继续调用他包装的 `finalObserver` 的 `onNext()` 方法和对应的 `Function` 方法)_ 订阅到 `SourceObservable`。然后开始第一节中的流程，也就是这个 `MapOserver` 将会被包装成一个发射器，开始发射事件，相比之前，此时调用 `OnNext()` 发送事件时，会首先调用 `MapObserver` 的 `onNext()`,然后继续调用它包装的 `finalObserver` 的 `onNext()`,不过在这之间会使用 `Function` 指定的操作对数据进行变换**
>
>**再增加一个操作符会怎么样，其实就是是一样的流程了，只不过此时对外开放的 `ObservalMap` 扮演了 `SourceObservable` 的角色。**

----

我在 [RxJava2.x开发-2 (Schedulers)](../8de84f35) 这篇文章中介绍了如何在 `RxJava2.x` 中如何使用线程调度，和如何判断当前方法是运行在哪个线程，强烈建议去看一下才能更明白他是怎么样使用的，之前总结了怎么做，现在看一下为什么。

## subscribeOn()

在 [RxJava2.x开发-2 (Schedulers)](../8de84f35) 总结了 `subscribeOn()` 用来切换上游线程，而且只有第一次有效，后面的调用只对 `doOnSubscribe()/onSubscribe()` 方法有效，但是为什么会是这样呢？

看一个简单的例子

```java
Observable.create(new ObservableOnSubscribe<Integer>() {
    @Override
    public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
        e.onNext(1);
    }
}).subscribeOn(Schedulers.io())
        .subscribe(new Consumer<Integer>() {
            @Override
            public void accept(@NonNull Integer s) throws Exception {
                log("accept -> " + s);
            }
        });
```

被观察者将会在 `io` 线程运行，为什么？看源码，仍旧是跟之前一样的模式，创建新的被观察者包装原来的观察者。

```java
// Observable.java

public final Observable<T> subscribeOn(Scheduler scheduler) {
    ObjectHelper.requireNonNull(scheduler, "scheduler is null");
    return RxJavaPlugins.onAssembly(new ObservableSubscribeOn<T>(this, scheduler));
}
```

我们还是将最开始的 `Observable` 称作 `SourceObservable`，`subscribeOn()` 创建并返回了 `ObservableSubscribeOn` 对象，没错它也是 `Observable` 的子类，是一个被观察者。我们再来看一下 `ObservableSubscribeOn` 类，创建时使用了原来的 `SourceObservable` 和一个线程调度器，先贴一下源代码，后面解释。


```java
// ObservableSubscribeOn.java

public final class ObservableSubscribeOn<T> extends AbstractObservableWithUpstream<T, T> {
    
    final Scheduler scheduler;

    public ObservableSubscribeOn(ObservableSource<T> source, Scheduler scheduler) {
        super(source);
        this.scheduler = scheduler;
    }

    @Override
    public void subscribeActual(final Observer<? super T> s) {
        final SubscribeOnObserver<T> parent = new SubscribeOnObserver<T>(s);
        s.onSubscribe(parent);
        parent.setDisposable(scheduler.scheduleDirect(new SubscribeTask(parent)));
    }

    static final class SubscribeOnObserver<T> extends AtomicReference<Disposable> implements Observer<T>, Disposable {

        final Observer<? super T> actual;
 
        SubscribeOnObserver(Observer<? super T> actual) {
            this.actual = actual;
        }

        @Override
        public void onNext(T t) {
            actual.onNext(t);
        }

    }

    final class SubscribeTask implements Runnable {
        private final SubscribeOnObserver<T> parent;

        SubscribeTask(SubscribeOnObserver<T> parent) {
            this.parent = parent;
        }

        @Override
        public void run() {
            source.subscribe(parent);
        }
    }
}
```

重点就在 `subscribeActual()` 方法，我们知道当订阅发生时会调用 `subscribeActual()` 方法，此时，创建了一个 `SubscribeOnObserver` 包装了真正订阅的观察者，然后调用观察者 `onSubscribe()` 方法，这跟之前的介绍是一样的，不同的是将新创建的 `SubscribeOnObserver` 订阅到 `SourceObservable` 这个过程，做成了一个 `Task`，并使用了线程调度器去执行，此时会发生什么？还是之前说的 `SubscribeOnObserver` 会被包装成一个发射器开始发射事件，而此时因为使用了线程调度器执行，将会运行在指定的线程。

到现在大致清楚了 `RxJava` 如何使用 `subscribeOn()` 方法切换了被观察者所运行的线程，那为什么只有第一次有效果，后面不生效呢？为什么对于 `doOnSubscribe()/onSubscibe()` 又是生效的呢？

虽然我们写代码时是从被观察者一路链式编程写下去的，但是其实真正开始执行的时机是，订阅开始的时候，即 `subscribe()` 调用的时候。而且按照我们之前的分析，`subscribe()` 方法会往上一层一层的调用上去，一直到最开始创建的被观察者，然后就开始发送事件。那我们调用两次 `subscribeOn()` 时，每次都会在里面生成新的被观察者，然后在指定线程调用 `subscribe()` 方法，所以开始的被观察者被调用的线程只取决了离他最近的那个 `subscribeOn()` ，因为在这里面会将发送最开始的被观察者的 `subscribe()` 方法 到指定线程运行，就好像假如 ObserableA 在 ThreadA 线程完成订阅，接着会继续调用里面包装的 ObserableB 在 ThreadB 线程完成订阅，其实最终的 ObserableB 还是在 ThreadB 线程完成订阅发送事件的。另外可以发现 `doOnSubscribe()/onSubscibe()` 是在发送到指定线程执行之前就执行的，所以他仍旧受到后面指定线程的影响，`doOnSubscribe()` 返回的也是一个 `Observable`，机制大致相同。

## observarOn()

调用 `observarOn()` 可以切换下游所在线程，每次调用都会切换线程。

类似上面的原理，仍旧生成了一个新的被观察者 `ObservableObserveOn`

```java
// Observable.java

public final Observable<T> observeOn(Scheduler scheduler, boolean delayError, int bufferSize) {
        ObjectHelper.requireNonNull(scheduler, "scheduler is null");
        ObjectHelper.verifyPositive(bufferSize, "bufferSize");
        return RxJavaPlugins.onAssembly(new ObservableObserveOn<T>(this, scheduler, delayError, bufferSize));
}
```
 
订阅时，如果使用了 `Schedulers.trampoline()` 那么在当前线程，不需要在创建新的包装观察者，否则创建 `ObserveOnObserver` 包装原来的观察者。

```java
// ObserveOnObserver.java

@Override
protected void subscribeActual(Observer<? super T> observer) {
    if (scheduler instanceof TrampolineScheduler) {
        source.subscribe(observer);
    } else {
        Scheduler.Worker w = scheduler.createWorker();
        source.subscribe(new ObserveOnObserver<T>(observer, w, delayError, bufferSize));
    }
}
```
看一下 `ObserveOnObserver` 它实现了 `Runnable` 接口，在 `run` 方法中根据当前状态，分别调用它包装的观察者的对应方法。然后在相应的方法的最后都会调用 `schedule();` 发送到指定线程操作，达到切换线程的目的，对应方法指的是 `onNext/onComplete/onError`

```java
static final class ObserveOnObserver<T> extends BasicIntQueueDisposable<T>
implements Observer<T>, Runnable {

    @Override
    public void onNext(T t) {
        schedule();
    }
    @Override
    public void onError(Throwable t) {
        if (done) {
            RxJavaPlugins.onError(t);
            return;
        }
        error = t;
        done = true;
        schedule();
    }
    @Override
    public void onComplete() {
        if (done) {
            return;
        }
        done = true;
        schedule();
    }
    // 去指定线程执行
    void schedule() {
        if (getAndIncrement() == 0) {
            worker.schedule(this);
        }
    }
    @Override
    public void run() {
        // 会根据当前的状态队形执行被包装的观察者的相关方法。
        if (outputFused) {
            drainFused();
        } else {
            drainNormal();
        }
    }
}
```

## 总结

说了这么多，我自己都感觉有点乱，心里明白但是写不出来的感觉真难受，还是文笔不行，想画个图表示一下，但画完感觉更复杂了。

在我看来，每次增加一个功能，比如操作符，或者线程切换，都会返回一个新的被观察者包装原来的被观察者，同时创建一个新的观察者，原来的观察者也会被包装进这个新的观察者，操作符会形成进行数据变换的被观察者和观察者，线程调度会形成线程切换的被观察者何观察者，形成一个一层一层包装的关系。

真正触发代码执行的是 `subscribe()` 方法，此时就会一层一层的调用更里面包装的被观察者的 `subscribe()` 方法，当最后一个被观察者，也就是我们最开始创建的那个的 `subscribe()` 方法被触发时，事件便开始发送，事件会一层一层往观察者里面传递，观察者又会调用它里面包装的观察者去处理传递这些事件，这过程中包含了事件的处理，变换，线程调度等。

感觉自己埋了个坑，说了这么多，说的也不是很明白，源码还是要自己看一下才更清楚一些。