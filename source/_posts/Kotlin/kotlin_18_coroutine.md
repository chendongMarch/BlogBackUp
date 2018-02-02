---
layout: post
title: Kotlin开发-18-协程
categories:
  - Kotlin
tags:
  - Kotlin
keywords:
  - Kotlin
  - 协程
  - Coroutine
abbrlink: 794b3d5f
date: 2017-09-09 14:07:00

---

协程 `Coroutines`，指各任务协作运行；

线程是操作系统层面的，由操作系统调度执行，我们可以开启一个线程，但无法知道线程什么时候执行，什么时候执行完，因此我们通常使用回调的形式在线程执行完之后接受执行的结果，线程的运行是抢占式的，后起的 B 线程可能抢占先起的 A 线程的资源，A 线程会被阻塞，从而造成资源的浪费。

协程是应用层面的，它由虚拟机进行调度，我们可以随意开启和终止协程的运行，协程是非抢占氏的，如果当前协程在运行，除非当前运行的协程主动 **退让，挂起**，否则其他协程不会抢占运行机会，由于各任务写作运行，就避免了创建大量的线程。

协程本身并不具备线程切换的功能，耗时操作等仍旧需要我们手动切换到子线程执行，但是协程的 API 设计使得我们可以像编写同步代码一样编写异步代码，避免使用回调，逻辑也更清晰。

<!--more-->

本文中的代码没有真正的进行操作，使用输出 `log` 来代替

## 推荐阅读

[深入理解 Kotlin Coroutine 1 - 知乎](https://zhuanlan.zhihu.com/p/25126223)

[深入理解 Kotlin Coroutine 2 - 知乎](https://zhuanlan.zhihu.com/p/25130621)
 
[Kotlin Coroutines](https://blog.dreamtobe.cn/kotlin-coroutines/)


## 使用回调进行异步

使用回调接口获取异步结果，是在 `Java` 中常用的方法，在主线程进行 `UI` 操作，切换到子线程进行耗时操作，操作的结果返回到主线程更新 `UI`。


```kotlin
// 线程池
val pool: ExecutorService by lazy {
    Executors.newCachedThreadPool()
}

// 一个很忙的方法，回调返回值
fun doLotsOfThingsWithCallBack(callback: (String) -> Unit) {
    pool.execute {
        log("开始耗时")
        Thread.sleep(100)
        Handler(Looper.getMainLooper()).post {
            callback.invoke("result")
        }
    }
}

fun testCallback() {
    log("开始先来个UI操作")
    doLotsOfThingsWithCallBack {
        log("回调拿到结果了 -> $it")
        log("再来个UI操作")
    }
}
```
因为我们只有一层回调，加上 `Kotlin` 对 `Lambda` 表达式的支持，看起来代码还不是那么难看，这也是在 `Java` 中常用的方法，相对好理解一些，看一下输出结果：

```bash
E/Coroutine: main  开始先来个UI操作
E/Coroutine: pool-1-thread-1  开始耗时
E/Coroutine: main  回调拿到结果了 -> result
E/Coroutine: main  再来个UI操作
```

## 使用协程实现

协程借助 `suspend` 关键字实现，使用 `suspend` 关键字描述的函数，表示此函数可以被挂起。

介绍几个概念

`CoroutineContext`，协程上下文，很多框架都会有一个上下文对象，支持 `+` 操作符，因此它可以是多个上下文的累加结果，`Kotlin` 中实现了一个空的实现 `EmptyCoroutineContext`，用来占位。


`Continuation`，协程中的任务执行后的操作，它可以返回任务的结果或者异常。


同样使用协程来实现一个和上面类似的功能：

```kotlin
fun testCoroutine() {
    log("开始先来个UI操作")
    callCoroutine {
        // 执行耗时操作，执行完之后会继续往下走
        val result = doLotsOfThingsCanSuspend("test")
        log("拿到结果了 -> $result")
        log("再来个UI操作")
    }
}


// 一个耗时的方法，可能被挂起
suspend fun doLotsOfThingsCanSuspend(p: String): String = suspendCoroutine {
    pool.execute {
        log("开始耗时")
        Thread.sleep(100)
        log("耗时结束")
        if (p.isNotEmpty())
            it.resume("succeed result")
        else
            it.resumeWithException(IllegalStateException("len < 0"))
    }
}


// 开始协程
fun callCoroutine(block: suspend () -> Unit) {
    block.startCoroutine(object : Continuation<Unit> {
        override val context: CoroutineContext
            get() = EmptyCoroutineContext
        override fun resume(value: Unit) {
        }
        override fun resumeWithException(exception: Throwable) {
        }
    })
}
```

输出结果：

```bash
E/Coroutine: main  开始先来个UI操作
E/Coroutine: pool-2-thread-1  开始耗时
E/Coroutine: pool-2-thread-1  耗时结束
E/Coroutine: pool-2-thread-1  拿到结果了 -> succeed result
E/Coroutine: pool-2-thread-1  再来个UI操作
```

我们看到，最后结果的 `UI` 操作，还是在子线程跑的，这个是不允许的，但是从代码上面，最后打印结果的代码明明是在主线程，其实 `var resule = xx` 之后的代码就如同回调中的代码，它的运行环境取决于 `Continuation` 的 `resume()` 方法执行的环境，在这段协程的代码中，最关键的一句是 

``` kotlin
val result = doLotsOfThingsCanSuspend("test")
```

它替代了原先的回调方法，也实现了类似原来回调的功能，当运行到这边时，函数会被挂起，等结果返回之后才会赋值并且进行后面的操作，避免了层层回调嵌套，另外虽然最后拿到结果的代码看起来是写在主线程，其实最后运行在了子线程，因此我们可以如下，修改代码，保证最后的结果被传递回主线程。

```kotlin
// 一个耗时的方法，可能被挂起
suspend fun doLotsOfThingsCanSuspend(p: String): String = suspendCoroutine {
    pool.execute {
        log("开始耗时")
        Thread.sleep(100)
        log("耗时结束")
        Handler(Looper.getMainLooper()).post {
            if (p.isNotEmpty())
                it.resume("succeed result")
            else
                it.resumeWithException(IllegalStateException("len < 0"))
        }
    }
}
```

但是对于这个操作，借助 `Continuation` 和 `CoroutineContext` 可以有更优雅的实现。

## 将结果返回切换到主线程

写一个  `Continuation` 的包装类 `UIContinuation`，这个很好理解，一个典型的包装者模式，他包装其他的 `Continuation` 在调用 `resume()` 方法时将结果返回主线程。

```kotlin
class UIContinuation<in T>(val delegate: Continuation<T>) : Continuation<T> {
    
    val handler by lazy { Handler(Looper.getMainLooper()) }
    
    override val context: CoroutineContext
        get() = delegate.context
    
    override fun resume(value: T) {
        handler.post { delegate.resume(value) }
    }
    
    override fun resumeWithException(exception: Throwable) {
        handler.post { delegate.resumeWithException(exception) }
    }
}
```

重写 `CoroutineContext` 自动完成 `Continuation` 的包装，需要一个接收外界设置的 `CoroutineContext ` 的 `Continuation`，他其实什么也不需要做。

```kotlin
class ContextContinuation(override val context: CoroutineContext = EmptyCoroutineContext) : Continuation<Unit> {
    override fun resume(value: Unit) {
    }
    override fun resumeWithException(exception: Throwable) {
    }
}
```

重写 `CoroutineContext` 使用插值器的方式包装，这里使用了 `ContinuationInterceptor`，他也是 `CoroutineContext` 的子类，类似 `okHttp` 的 `interceptor`，他的作用就是，拦截旧的 `Continuation` 如果需要的话生成新的 `Continuation`.

```koltin
class AsyncCoroutineContext : 
        AbstractCoroutineContextElement(ContinuationInterceptor.Key), 
        ContinuationInterceptor {
    override fun <T> interceptContinuation(continuation: Continuation<T>): Continuation<T> {
        // 这里使用 fold 函数是为了兼容会有其他插值器想更改 Continuation 的操作，因此不能草率的直接返回
        // 如果不好理解，也可以直接理解为 return UIContinuation()
        return UIContinuation(continuation.context.fold(continuation) {
            con, element ->
            if (element != this && element is ContinuationInterceptor) {
                element.interceptContinuation(con)
            } else con
        })
    }
}
```

此时，调用如下方法启动协程，配置可以将结果返回主线程的上下文，当结果返回时会自动返回主线程。

```kotlin
fun callCoroutine(block: suspend () -> Unit) {
    block.startCoroutine(ContextContinuation(AsyncCoroutineContext()))
}
```


## 使用 CoroutineContext 传参

首先，`CoroutineContext` 有一个不错的特性，就是重载了 `+` 操作符，因为我们可以对 `CoroutineContext` 进行加法操作返回新的 `CoroutineContext`

为了方便使用，我们将基础的方法进行提取，他们实现一个通用的功能，即切换后台线程执行耗时操作，然后在主线程接收结果更改UI。

```kotlin
// 开始协程，可以从外界接受上下文变量
fun callCoroutine(context: CoroutineContext = EmptyCoroutineContext, block: suspend () -> Unit) {
    // 自动具备了将结果切换回主线程的功能
    block.startCoroutine(ContextContinuation(AsyncCoroutineContext() + context))
}

// 一个通用的执行耗时操作的方法，可能被挂起
// 接受的参数是 CoroutineContext 的扩展函数，这样的写法好处在于可以在函数内使用 this 访问
// 在参数 block 指向的函数中，是一个任意的耗时操作
suspend fun <T> doLotsOfThingsCanSuspend(block: CoroutineContext.() -> T): T = suspendCoroutine {
    pool.execute {
        try {
            it.resume(block(it.context))
        } catch (e: Exception) {
            e.printStackTrace()
            it.resumeWithException(e)
        }
    }
}
```

借助上面的抽象出来的两个方法可以开启协程，执行任意耗时操作，并将结果返回到主线程，接下来使用 `ConroutineContext` 进行参数的传递，从上面我们发现 `block` 函数是一个无参的函数，而各种耗时操作参数不一，怎么兼容这个问题，就需要使用 `ConroutineContext` ，他是一个上下文，是整个运行环境，借助它我们可以传递参数，定义一个携带参数的上下文，它仍旧是 `AbstractCoroutineContextElement` 的子类，并标记一个 `Key`

```kotlin
// 携带上下文的 context
class ParamContext(val url: String) : AbstractCoroutineContextElement(Key) {
    companion object Key : CoroutineContext.Key<ParamContext>
}
```

从上下文中获取参数，重点在于 `this[ParamContext.Key]`，也可以写作 `this[ParamContext]`，因为那个 `Key` 是一个伴生对象，能用下标访问是因为 `CoroutineContext` 重载了 `[]` 操作符，从而可以取出之前传递的 `ParamContext`

```kotlin
fun testCoroutine() {
    log("开始先来个UI操作")
    callCoroutine(ParamContext("test")) {
        // 执行耗时操作，执行完之后会继续往下走
        val result = doLotsOfThingsCanSuspend {
            log("开始耗时")
            Thread.sleep(100)
            log("耗时结束")
            this[ParamContext.Key]?.url?.length.toString()
        }
        log("拿到结果了 -> $result")
        log("再来个UI操作")
    }
}
```

## Sequence

官方的一个例子

```kotlin
val fibonacci = buildSequence {
    yield(1) // 此处返回 1 并挂起
    var cur = 1
    var next = 1
    while (true) {
        yield(next) // 返回下一个值并挂起
        val temp = cur + next
        cur = next
        next = temp
    }
}


for (i in fibonacci) {
    log("$i")
    if (i > 100) break
}
```
每次对 `Sequence` 迭代遍历下一个时，会执行协程到 `yield` 的位置返回值，并挂起，等待下一次迭代，这样就构造了一个懒序列，只有在需要的时候才会计算生成，而不是一次生成大量数据，除了第一次，他的每次被调起，都是从这次 `yield` 到下次 `yield`


## 总结

使用 `suspendCoroutine` 构造一个可挂起的函数，在内部执行操作，通过 `Continuation` 的 `resume()` 方法将结果或者异常返回，你可以决定返回之前是不是要进行自己的处理，比如发到主线程。

`Continuation` 是一个接口，他是一个结果的接受处理者，这也是 `Continuation` 的含义，他将在任务完成之后被调用。

`CoroutineContext` 是一个上下文，表示一个运行时环境，它可以做很多事，比如传递参数等。

`ContinuationInterceptor` 是 `Continuation` 的插值器，提供一个更改原来的 `Continuation` 的机会，用来创建返回新的 `Continuation`。

从代码层面上面，可以跟回调的方法稍作比较，下面代码中，其实 `var result = xxx` 这里就相当于一个回调，但是并不是回调的形式，在这里执行 `doLotsOfThingsCanSuspend()` 时，函数会被挂起，等待结果返回，这不是一个阻塞的过程，异步线程的切换在挂起函数内进行，因此我们可以像写同步代码一样，书写异步代码，结果返回之后，后面的代码其实相当于回调里面的代码，此时才会继续执行，和回调一样，它的运行环境取决于挂起函数中调用 `resume()` 方法时的环境。

```kotlin
callCoroutine {
    // 执行耗时操作，执行完之后会继续往下走
    val result = doLotsOfThingsCanSuspend("test")
    log("拿到结果了 -> $result")
    log("再来个UI操作")
}
```


## Kotlinx 官方协程扩展框架

添加依赖

```gradle
compile 'org.jetbrains.kotlinx:kotlinx-coroutines-core:0.18'
```

借助 `launch()` 函数来开启一个协程，`CommonPool` 是内置的一个上下文，他将会在子线程执行任务。

```kotlin
fun test1() {
    var result = 0
    // 开启协程
    log(TAG, "开始协程")
    launch(CommonPool) {
        log(TAG, "协程内开始")
        for (i in 1..1000) {
            result += i
        }
        log(TAG, "协程内结束")
    }
    log(TAG, "结束协程")
    log(TAG, result.toString())
}
```

输出结果为

```bash
E/tag: main  开始协程
E/tag: main  结束协程
E/tag: main  0
E/tag: ForkJoinPool.commonPool-worker-2  协程内开始
E/tag: ForkJoinPool.commonPool-worker-2  协程内结束
```

结果就好像我们在子线程执行了一个任务，他并没有干预外面的代码运行，也没有获得预期的值，我们希望后面的任务在子线程的代码计算完成之后在执行，需要使用 `join()` 函数。

---

`launch()` 函数返回一个 `Job` 对象，同时 `join()` 函数需要在一个可挂起的函数内执行，使用 `runBlocking()` 函数使用 `EmptyCoroutineContext` 可以在当前位置开启协程；

这样就得到了预期的结果，同样我们可以调用 `job.cancel()` 来结束这个任务

```kotlin
fun test2() {
    runBlocking {
        var result = 0
        log(TAG, "开始协程")
        val job = launch(CommonPool) {
            log(TAG, "协程内开始")
            for (i in 1..1000) {
                delay(1)
                result += i
            }
            log(TAG, "协程内结束")
        }
        job.join() // 要求当前协程，等待该协程执行完再执行
        log(TAG, "结束协程")
        log(TAG, result.toString())
    }
}
```
输出结果为

```bash
E/tag: main  开始协程
E/tag: ForkJoinPool.commonPool-worker-2  协程内开始
E/tag: ForkJoinPool.commonPool-worker-2  协程内结束
E/tag: main  结束协程
E/tag: main  500500
```

---

获取返回值，上面我们使用了变量的形式来存储计算的值，当我们需要一个任务的返回值时，最好还是让任务能够返回执行完的结果，使用 `async()` 函数可以返回一个 `Deferred` 对象，从里面可以取到执行结果

```kotlin
launch(CommonPool) {
    val deferred1 = async(CommonPool) {
        delay(1000)
        "result"
    }
    val deferred2 = async(CommonPool) {
        delay(2000)
        100
    }
    log(TAG, "${deferred1.await()},${deferred2.await()}")
    log(TAG, "${deferred1.getCompleted()},${deferred2.getCompleted()}")
}
```

输出结果

```bash
E/tag: ForkJoinPool.commonPool-worker-1  result,100
E/tag: ForkJoinPool.commonPool-worker-1  result,100
```

---

在子线程计算，在主线程更新，`UI` 是一个协程上下文，他使用 `Handler` 将结果分发到主线程。

```kotlin
launch(UI) {
    val deferred = async(CommonPool) {
        var index = 1
        for (i in 1..100) {
            delay(10)
            Thread.sleep(10)
            index += i
        }
        index
    }
    mTestTv.text = deferred.await().toString()
}
```

