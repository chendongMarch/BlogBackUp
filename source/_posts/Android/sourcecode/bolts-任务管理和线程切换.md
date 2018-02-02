---
layout: post
title: Bolts 更简单任务管理和线程切换 [源码]
category: Android
tags:
  - Android
  - SourceCode
keywords:
  - Android
  - Blots
  - Task
abbrlink: 1dbbedd
date: 2017-11-04 23:31:00
photos: http://olx4t2q6z.bkt.clouddn.com/18-2-1/70364567.jpg
location: 杭州尚妆
---

尤塞恩·圣利奥·博尔特 `Usain St Leo Bolt`，牙买加短跑运动员，男子100米、男子200米以及男子400米接力赛的世界纪录保持人，同时是以上三项赛事的连续三届奥运金牌得主。

使用 `Bolts` 可以将一个完整的操作拆分成多个子任务，这些子任务可以自由的拆分、组合和替换，每个任务作为整个任务链的一环可以运行在指定线程中，同时既能从上行任务中获取任务结果，又可以向下行任务发布当前任务的结果，而不必考虑线程之间的交互。

> [Bolts-Android Bolts 在 Android 下的实现](https://github.com/BoltsFramework/Bolts-Android)   
> [Bolts-ObjC Bolts 在 OC 下的实现](https://github.com/BoltsFramework/Bolts-ObjC)   
> [Bolts-Swift Bolts 在 Swift 下的实现](https://github.com/BoltsFramework/Bolts-Swift)

<!--more-->

## 前言

一个关于线程调度的简单需求，在子线程从网络下载图片，并返回下载的图片，在主线程使用该图片更新到 UI，同时返回当前 UI 的状态 json，在子线程将 json 数据保存到本地文件，完成后在主线程弹出提示，这中间涉及到了 4 次线程切换，同时后面的任务需要前面任务完成后的返回值作为参数。

使用 `Thread + Handler` 实现，线程调度很不灵活，代码可读性差，不美观，扩展性差，错误处理异常麻烦。

```java
String url = "http://www.baidu.com";
Handler handler = new Handler(Looper.getMainLooper());
new Thread(() -> {
    // 下载
    Bitmap bitmap = downloadBitmap(url);
    handler.post(() -> {
        // 更新 UI
        String json = updateUI(bitmap);
        new Thread(() -> {
            // 向存储写入UI状态
            saveUIState(json);
            // 保存成功后，提示
            handler.post(() -> toastMsg("save finish."));
        }).start();
    });
}).start();
```

使用 `RxJava` 实现，线程调度非常灵活，链式调用，代码清晰，扩展性好，有统一的异常处理机制，不过 `Rx` 是一个很强大的库，如果只用来做线程调度的话，`Rx` 就显得有点太重了。

```java
Observable.just(URL)
        // 下载
        .map(this::downloadBitmap)
        .subscribeOn(Schedulers.newThread())
        // 更新UI
        .observeOn(AndroidSchedulers.mainThread())
        .map(this::updateUI)
        // 存储 UI 状态
        .observeOn(Schedulers.io())
        .map(this::saveUIState)
        // 显示提示
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(rst -> toastMsg("save to " + rst),
                // handle error
                Throwable::printStackTrace);
```

使用 `bolts` 实现，线程调度灵活，链式调用，代码清晰，具有良好的扩展性，具有统一的异常处理机制，虽然没有 `Rx` 那么丰富的操作符，但是胜在类库非常非常小，只有 38 KB。

```java
Task
        .forResult(URL)
        // 下载
        .onSuccess(task -> downloadBitmap(task.getResult()), Task.BACKGROUND_EXECUTOR)
        // 更新UI
        .onSuccess(task -> updateUI(task.getResult()), Task.UI_THREAD_EXECUTOR)
        // 存储UI状态
        .onSuccess(task -> saveUIState(task.getResult()), Task.BACKGROUND_EXECUTOR)
        // 提示
        .onSuccess(task -> toastMsg("save to " + task.getResult()), Task.UI_THREAD_EXECUT
        // handle error
        .continueWith(task -> {
            if (task.isFaulted()) {
                task.getError().printStackTrace();
                return false;
            }
            return true;
        });
```

## 线程调度器

共有 4 种类型执行线程，将任务分发到指定线程执行，分别是

1. `backgroud` - 后台线程池，可以并发执行任务。
2. `scheduled` - 单线程池，只有一个线程，主要用来执行 `delay` 操作。
3. `immediate` - 即时线程，如果线程调用栈小于 15，则在当前线程执行，否则代理给 `background`。
4. `uiThread` - 针对 `Android` 设计，使用 `Handler` 发送到主线程执行。


### backgroud

主要用来在后台并发执行多任务

```java
public static final ExecutorService BACKGROUND_EXECUTOR = BoltsExecutors.background();
```
在 `Android` 平台下根据 `CPU` 核数创建线程池，其他情况下，创建缓存线程池。

```java
background = !isAndroidRuntime()
    ? java.util.concurrent.Executors.newCachedThreadPool()
    : AndroidExecutors.newCachedThreadPool();
```


### scheduled

主要用于任务之间做 `delay` 操作，并不实际执行任务。

```java
scheduled = Executors.newSingleThreadScheduledExecutor();
```

### immediate

主要用来简化那些不指定运行线程的方法，默认在当前线程去执行任务，使用 `ThreadLocal` 保存每个线程调用栈的深度，如果深度不超过 15，则在当前线程执行，否则代理给 `backgroud` 执行。

```java
private static final Executor IMMEDIATE_EXECUTOR = BoltsExecutors.immediate();

// 关键方法
@Override
public void execute(Runnable command) {
  int depth = incrementDepth();
  try {
    if (depth <= MAX_DEPTH) {
      command.run();
    } else {
      BoltsExecutors.background().execute(command)
    }
  } finally {
    decrementDepth();
  }
}
```

### uiThread

为 `Android` 专门设计，在主线程执行任务。

```java
public static final Executor UI_THREAD_EXECUTOR = AndroidExecutors.uiThread();
```

```java
private static class UIThreadExecutor implements Executor {
  @Override
  public void execute(Runnable command) {
    new Handler(Looper.getMainLooper()).post(command);
  }
}
```


## 核心类

`Task`，最核心的类，每个子任务都是一个 `Task`，它们负责自己需要执行的任务。每个 `Task` 具有 3 种状态 `Result`、`Error` 和 `Cancel`，分别代表成功、异常和取消。

`Continuation`，是一个接口，它就像链接子任务每一环的锁扣，把一个个独立的任务链接在一起。

通过 `Task` - `Continuation` - `Task` - `Continuation` ... 的形式组成完整的任务链，顺序在各自线程执行。


## 创建 Task

根据 `Task` 的 3 种状态，创建简单的 `Task`，会复用已有的任务对象

```java
public static <TResult> Task<TResult> forResult(TResult value)

public static <TResult> Task<TResult> forError(Exception error)

public static <TResult> Task<TResult> cancelled()
```

使用 `delay` 方法，延时执行并创建 `Task`

```java
public static Task<Void> delay(long delay)

public static Task<Void> delay(long delay, CancellationToken cancellationToken)
```

使用 `whenAny` 方法，执行多个任务，当任意任务返回结果时，保存这个结果

```java
public static <TResult> Task<Task<TResult>> whenAnyResult(Collection<? extends Task<TResult>> tasks)

public static Task<Task<?>> whenAny(Collection<? extends Task<?>> tasks)
```

使用 `whenAll` 方法，执行多个任务，当全部任务执行完后，返回结果

```java
public static Task<Void> whenAll(Collection<? extends Task<?>> tasks) 

public static <TResult> Task<List<TResult>> whenAllResult(final Collection<? extends Task<TResult>> tasks)
```

使用 `call` 方法，执行一个任务，同时创建 `Task` 

```java
public static <TResult> Task<TResult> call(final Callable<TResult> callable, Executor executor,
      final CancellationToken ct)
```

## 链接子任务

使用 `continueWith` 方法，链接一个子任务，如果前行任务已经执行完成，则立即执行当前任务，否则加入队列中，等待。

```java
public <TContinuationResult> Task<TContinuationResult> continueWith(
      final Continuation<TResult, TContinuationResult> continuation, final Executor executor,
      final CancellationToken ct)
```


使用 `continueWithTask` 方法，在当前任务之后链接另一个任务链，这种做法是为了满足那种将部分任务组合在一起分离出去，作为公共任务的场景，他接受将另外一个完全独立的任务链，追加在当前执行的任务后面。

```java
public <TContinuationResult> Task<TContinuationResult> continueWithTask(
      final Continuation<TResult, Task<TContinuationResult>> continuation, final Executor executor,
      final CancellationToken ct)
```

使用 `continueWhile` 方法链接子任务，与 `continueWith` 区别在于，他有一个 `predicate` 表达式，只有当表达式成立时，才会追加子任务，这样做是在执行任务前可以做一个拦截操作，也是为了不破环链式调用的整体风格。

```java
public Task<Void> continueWhile(final Callable<Boolean> predicate,
      final Continuation<Void, Task<Void>> continuation, final Executor executor,
      final CancellationToken ct)
```

使用 `onSuccess` 和 `onSuccessTask` 链接单个任务个任务链，区别于 `continueWith` 在于，`onSuccess` 方法，前行任务如果失败了，后行的任务也会直接失败，不会再执行，但是 `continueWith` 的各个子任务之间没有关联，就算前行任务失败，后行任务也会执行。

```java
public <TContinuationResult> Task<TContinuationResult> onSuccess(
      final Continuation<TResult, TContinuationResult> continuation, Executor executor,
      final CancellationToken ct)
```

## 取消任务

`Task` 没有 `cancel` 方法，而是使用了 `CancellationToken` 作为标记，任务执行之前会检查这个标记，如果标记为退出，则会直接退出任务。

```java
CancellationTokenSource cancellationTokenSource = new CancellationTokenSource();
CancellationToken token =   cancellationTokenSource.getToken();
Task.call((Callable<String>) () -> null,
        Task.BACKGROUND_EXECUTOR,
        token);
// 取消任务
cancellationTokenSource.cancel();
```

## 异常的处理

关于异常的处理，整个机制下来，每个任务作为一个独立的单位，异常会被统一捕捉，因此不必针对任务中的方法进行单独的处理。

如果使用了 `continueWith` 链接任务，那么当前任务的的异常信息，将会保存在当前 `Task` 中在下行任务中进行处理，下行任务也可以不处理这个异常，直接执行任务，那么这个异常就到这里停止了，不会再向下传递，也就是说，只有下行任务才知道当前任务的结果，不管是成功还是异常。

当然了，如果任务之间有关联，由于上行任务的异常极大可能造成当前任务的异常，那么当前任务异常的信息，又会向下传递，但是上行任务的异常就到这里为止了。

如果使用 `onSuccess` 之类的方法，如果上行任务异常了，那么下行任务根本不会执行，而是直接将异常往下面传递，直到被处理掉。

## 任务的分离和组合

我们可以将一个完整的操作细分成多个任务，每个任务都遵循单一职责的原则而尽量简单，这样可以在任务之间再穿插新的任务，或者将部分任务分离出来组合到一起等。

### 扩展性

我们可以在两个细分的任务之间添加一个新的操作，而不影响上行和下行任务，如我们给文章开头的需求中更新 `UI` 之前，将 `Bitmap` 先保存到本地。

```java
Task
        .forResult(URL)
        // 下载
        .onSuccess(task -> downloadBitmap(task.getResult()), Task.BACKGROUND_EXECUTOR)
        // 保存在本地
        .onSuccess(task -> saveBitmapToFile(task.getResult()),Task.BACKGROUND_EXECUTOR)
        // 更新UI
        .onSuccess(task -> updateUI(task.getResult()), Task.UI_THREAD_EXECUTOR)
        ...
```

### 复用性

对一些公共的操作，可以单独分离成新的任务，当需要做类似操作时，即可复用这部份功能，如可以将**下载图片并更新 `UI`**、**保存状态并弹出提示** 两块功能分离出来，作为公共的任务。


```java
// 下载图片->更新UI
public Continuation<String, Task<String>> downloadImageAndUpdateUI() {
    return task ->
            Task.call(() -> downloadBitmap(task.getResult()), Task.BACKGROUND_EXECUTOR)
                    .continueWith(taskWithBitmap -> updateUI(taskWithBitmap.getResult()), Task.UI_THREAD_EXECUTOR);
}

// 保存状态->提示信息
public Continuation<String, Task<Boolean>> saveStateAndToast() {
    return task ->
            Task.call(() -> saveUIState(task.getResult()), Task.BACKGROUND_EXECUTOR)
                    .continueWith(taskWithPath -> toastMsg("save to " + taskWithPath.getResult()));
}
```
使用分离的任务

```java
Task
        .forResult(URL)
        .continueWithTask(downloadImageAndUpdateUI())
        .continueWithTask(saveStateAndToast())
        ...
```

## 总结

在 `Task` 中有一个 `continuations` 是当前任务后面追加的任务列表，当当前任务成功、异常或者取消时，会去执行列表中的后续任务。

通常情况下，我们使用链式调用构建任务链，结果就是一条没有分支的任务链。

**添加任务时** ：每次添加一个 `Continuation`，就会生成一个 `Task`，加到上行任务的 `continuations` 列表中，等待执行，同时返回当前的 `Task`，以便后面的任务可以链接到当前任务后面。

**执行任务时** ：当前任务执行完之后，结果可能有 3 种，都会被保存到当前的 `Task` 中，然后检查 `continuations` 列表中的后续任务，而当前的 `Task` 就会作为参数，传递到后续链接的任务中，来让后面的任务得知上行任务的结果。


