---
layout: post
title: 'Glide-平滑加载的图片框架 [源码]'
category: Android
tags:
  - Android
  - SourceCode
keywords:
  - glide
  - 图片加载
  - android
abbrlink: 47636f69
photos: 'https://images.pexels.com/photos/789433/pexels-photo-789433.jpeg'
location: 杭州尚妆
date: 2020-05-21 09:58:00
---

本文按照 `Glide` 常用的如下用法来分析源码：

```java
Glide.with(context).load("url").thumbnail().into(imageView);
```

<!--more-->

根据上面的代码，将内容分为以下几个章节：

- `Glide` - 强大的顶层管理类
- `Glide.with()` - 创建及缓存 `RequestManager`
- `.load(url)` - 创建 `RequestBuilder`
- `.thumbnail()` - 配置 `RequestBuilder`
- `.into(target)` - 发起 `Request`
- 开始加载资源
- 缓存和数据源
- 数据转换
- 总结



## 强大的 Glide

`Glide` 是一个单例对象，在这个框架中功能相对复杂，每个功能都由对应的类实现，但是他们都被 `Glide` 类统一管理，`Glide` 的构造方法很长，主要是初始化一些内部持有的管理类，类比来看的 `Glide` 就类似高级管理阶层，下属还有很多次级管理层。

简单看一下重要的成员变量：

```java
Glide.java

GenericLoaderFactory loaderFactory;   // 工厂管理类，负责 ModelLoader 的注册和缓存实现
TranscoderRegistry transcoderRegistry = new TranscoderRegistry(); // ResourceTranscoder 工厂管理类
DataLoadProviderRegistry dataLoadProviderRegistry; // DataLoadProvider 注册管理

Engine engine; // 负责从内存缓存、文件缓存、数据源中获取资源
BitmapPool bitmapPool; // 用来控制复用 Bitmap 的内存空间
MemoryCache memoryCache; // Lru 内存缓存
```

## Glide.with()

> `Glide.with()` 方法将会返一个 `RequestManager` 对象。

在讨论 `Glide.with()` 之前，先来关注一下两个比较重要的点：

- 借助 `Fragment` 同步与宿主 `Activity` 的生命周期及内存管理。
- `RequestManager` 的创建和缓存

### 借助 Fragment

当加载图片时，我们需要时刻观察宿主 `Activity` 的状态：

- 结合生命周期，对后台线程做一些耗时操作，比如网络请求、IO 流、图片的编解码等，在适当的时机进行暂停、重启或销毁操作；
- 关注内存变化，当回调 `onTrimMemory()` 等内存回收方法时，框架内部能够感知到；

需要做到这些，就需要在 `Activity` 的各种方法中调用框架对应的方法，但是这样造成很强的耦合，使用起来会变的很繁琐，不过 `Glide` 使用了一种比较巧妙的办法那就是在对应的 `Activity` 中添加一个空的 `Fragment`，这个 `Fragment` 不会绘制任何 `UI`，但是由于 `Fragment` 和 `Activity` 的特殊关系，当这个 `Fragment` 被添加到 `Activity` 中时，他的生命周期就与 `Activity` 的生命周期同步了，内部只需要关注 `Fragment` 的生命周期即可，而对外是完全封闭的。

在 `Glide` 内部提供了 `RequestManagerFragment` 和 `SupportRequestManagerFragment` 对 `v4.Fragment` 做了兼容，他们主要关注的是如下方法：

```java
// 生命周期方法
onAttach()
onDetach()
onStart()
onStop()
onDestroy()

// 内存不足回调
onTrimMemory(int level)
onLowMemory()
```

### RequestManager 的缓存

在 `Fragment` 内部持有一个 `RequestManager`，当 `Fragment` 的生命周期+内存回收方法被回调时，`RequestManager` 的对应方法会被调用，从而实现对请求和内存的管理。

我们希望在每个 `Activity` 中仅有一个 `RequestManager` 来统一处理，而不是每次都创建一个新的，因此我们要对 `RequestManager` 做一个缓存，而这个缓存也是借助 `Fragment` 来实现的，每次向 `Activity` 中添加 `Fragment` 时，总会使用唯一的 `tag`，这样需要使用 `RequestManager` 时，会先去拿对应 `tag` 的 `Fragment`，拿到了说明已经创建过，取出 `RequestManager` 直接使用，否则创建新的 `Fragment`(持有 `RequestManager`)用指定的 `tag` 添加到 `Activity` 中，同时拿到 `RequestManager`。

因为 `v4.Fragment` 的问题，所有的方法都会有一个 `support` 的对应方法，以下仅保留关键代码：

```java
// RequestManagerRetriever.java

// 从 Activity 开始
public RequestManager get(Activity activity) {
    return fragmentGet(activity, fm);
}

// 从 Fragment 开始
public RequestManager get(Fragment fragment) {
    FragmentManager fm = fragment.getChildFragmentManager();
    return supportFragmentGet(fragment.getActivity(), fm);
}

// 从 Fragment 中取得 RequestManager
RequestManager fragmentGet(Context context, android.app.FragmentManager fm) {
    RequestManagerFragment current = getRequestManagerFragment(fm);
    RequestManager requestManager = current.getRequestManager();
    if (requestManager == null) {
        requestManager = new RequestManager(context, current.getLifecycle(), current.getRequestManagerTreeNode());
        current.setRequestManager(requestManager);
    }
    return requestManager;
}

// 从 android.app.FragmentManager 获取 Fragment，没有的话会添加新的
RequestManagerFragment getRequestManagerFragment(final android.app.FragmentManager fm) {
    RequestManagerFragment current = (RequestManagerFragment) fm.findFragmentByTag(FRAGMENT_TAG);
    if (current == null) {
        current = pendingRequestManagerFragments.get(fm);
        if (current == null) {
            current = new RequestManagerFragment();
            pendingRequestManagerFragments.put(fm, current);
            fm.beginTransaction().add(current, FRAGMENT_TAG).commitAllowingStateLoss();
            handler.obtainMessage(ID_REMOVE_FRAGMENT_MANAGER, fm).sendToTarget();
        }
    }
    return current;
}
```

所有图片的加载总是以 `Glide.with()` 方法开始，当调用 `Glide.with()` 方法时，会借助单例 `RequestManagerRetriever` 来获取到一个 `RequestManager`，当然这里不仅仅是单纯的获取、还包括缓存及同步生命周期等，这个在上面已经说过了。

```java
public static RequestManager with(Context context) {
    RequestManagerRetriever retriever = RequestManagerRetriever.get();
    return retriever.get(context);
}
```

`RequestManagerRetriever` 是一个单例，他对 `RequestManagerFragment` 及其内部的 `RequestManager` 进行管理，`Retriever` 是猎犬、寻回的意思，`RequestManagerRetriever` 也就是对 `RequestManager` 进行寻回，找到已经在使用的 `RequestManager`，可以说很形象啦。当需要获取 `RequestManager` 时，借助这条 **猎犬** 即可，他会给你找回合适的 `RequestManager`。

### RequestManager

`RequestManager` 主要用来管理请求的发送和停止，

内部有一个 `ConnectivityMonitor` 它使用 `BroadcastReceiver` 实现，用于检测网络变化。

```java
ConnectivityMonitor connectivityMonitor = factory.build(context,
        new RequestManagerConnectivityListener(requestTracker));
```

使用 `RequestTracker` 来真正的管理请求队列

```java
private final Set<Request> requests = Collections.newSetFromMap(new WeakHashMap<Request, Boolean>());
private final List<Request> pendingRequests = new ArrayList<Request>();
```


## .load(Type param)

> `load(Type param)` 将会返回一个 `DrawableTypeRequest` 对象，他并不是一个真正的 `Request`，更像一个 `RequestBuilder`，其实他真的是 `GenericRequestBuilder` 的子类，是建造者模式的体现，用来进行其他参数的配置。

利用 `Glide.with()` 此时我们已经获取到了合适的 `RequestManager` 对象，可以使用 `.load()` 方法选择加载文件、`Uri`、网络路径等：

```java
RequestManager.java

public DrawableTypeRequest<String> load(String string) {
    return (DrawableTypeRequest<String>) fromString().load(string);
}
public DrawableTypeRequest<String> fromString() {
    return loadGeneric(String.class);
}

public DrawableTypeRequest<File> load(File file){...}
public DrawableTypeRequest<File> fromFile(){...}

public DrawableTypeRequest<Uri> load(Uri uri){...}
public DrawableTypeRequest<Uri> fromUri(){...}

public DrawableTypeRequest<Integer> load(Integer resourceId){...}
public DrawableTypeRequest<Integer> fromResource(){...}

public DrawableTypeRequest<byte[]> load(byte[] model){...}
public DrawableTypeRequest<byte[]> fromBytes(){...}

```

可以看到每种数据类型会对应不同的 `load(Type param)` 方法，同时又会对应一个 `fromType()` 方法来构造 `DrawableTypeRequest` 对象，而他们最终都会使用 `loadGeneric()` 方法来创建一个 `DrawableTypeRequest`

```java
RequestManager.java

private <T> DrawableTypeRequest<T> loadGeneric(Class<T> modelClass) {
    ModelLoader<T, InputStream> streamModelLoader = Glide.buildStreamModelLoader(modelClass, context);
    ModelLoader<T, ParcelFileDescriptor> fileDescriptorModelLoader =
            Glide.buildFileDescriptorModelLoader(modelClass, context);
    return optionsApplier.apply(
            new DrawableTypeRequest<T>(modelClass, streamModelLoader, fileDescriptorModelLoader, context,
                    glide, requestTracker, lifecycle, optionsApplier));
}
```

### ModelLoader

这里可以看到有 `ModelLoader`，他负责完成数据转换的功能，例如想要加载网络图片，会提供一个 `url` 那么就会有一个 `ModelLoader` 完成 `String-> InputStream` 的转化，通过网络请求拿到 `InputStream`。

这里会有更丰富的内容，统一在数据转换一节中讨论，这里需要知道的是他作为 `DrawableTypeRequest` 的成员，在将来会承担数据转换的工作。

### DataLoadProvider

在上面我们创建了一个 `DrawableTypeRequest`，看一下构造方法

```java
DrawableTypeRequest(Class<ModelType> modelClass, ModelLoader<ModelType, InputStream> streamModelLoader,
        ModelLoader<ModelType, ParcelFileDescriptor> fileDescriptorModelLoader, Context context, Glide glide,
        RequestTracker requestTracker, Lifecycle lifecycle, RequestManager.OptionsApplier optionsApplier) {
    super(context, modelClass,
            buildProvider(glide, streamModelLoader, fileDescriptorModelLoader, GifBitmapWrapper.class,
                    GlideDrawable.class, null),glide, requestTracker, lifecycle);
}

DrawableRequestBuilder(Context context, Class<ModelType> modelClass,
        LoadProvider<ModelType, ImageVideoWrapper, GifBitmapWrapper, GlideDrawable> loadProvider, Glide glide,
        RequestTracker requestTracker, Lifecycle lifecycle) {
}
```

发现他需要一个 `DataLoadProvider`，他也是起到了一个数据转换的作用，不过它是提供编解码器来将 `InputStream` 等格式转化成  `Bitmap` 及 `GifDrawable` 等格式。

这里会有更丰富的内容，统一在数据转换一节中讨论，这里需要知道的是他作为 `DrawableTypeRequest` 的成员，在将来会承担数据转换的工作。

## .thumbnail()


> `.thumbnail()` 等配置项都是 `GenericRequestBuilder` 的成员方法，返回值是 `GenericRequestBuilder`，达到链式调用的目的。

这里只是用 `thumbnial` 来代指一系列的配置请求的方法，实际上配置项要更多，大致列举如下：

```java
// 控制图片尺寸
thumbnail(float size) // 0-1,先加载缩略图，内部实现是创建一个新的 thumbnailRequest
sizeMultiplier(float size) // 0-1，控制加载图片的大小，和缩略图有本质区别
override() // 加载指定大小图片

// 编解码器
encoder()
cacheDecoder()
decoder()
sourceEncoder()

// 缓存策略和优先级
diskCacheStrategy() // 磁盘缓存策略
priority() // 请求的优先级别，缩略图的优先级总会比原图高一些
skipMemoryCache() // 跳过内存缓存

// 变换和动画
transform() // 对图片进行变化，比如圆形裁剪
dontTransform() // 去掉变换效果
transcoder()
animate() // 加载动画
dontAnimate() // 去掉加载动画

// 占位图
placeholder() // 加载时占位图
fallback() // 资源为null时的占位图，没有将会使用 error
error() // 加载失败占位图

// 其他
listener() // 加载监听
signature() // 唯一标记

// 加载
preload() // 提前加载准备资源到内存中，下次使用时速度会快
into() // 加载到指定 target
```
以上的一系列方法都是一些配置项，调用之后作为参数存储起来，请求的时候使用

```java
private Key signature = EmptySignature.obtain(); // 对应 signature()
private RequestListener<? super ModelType, TranscodeType> requestListener; // 对应 listener()
private Float thumbSizeMultiplier; // 对应 thumbnail()
private GenericRequestBuilder<?, ?, ?, TranscodeType> thumbnailRequestBuilder; // 对应 thumbnail()
private Float sizeMultiplier = 1f; // 对应 sizeMultiplier()
private Priority priority = null; // 对应 priority()
private boolean isCacheable = true; // 对应 skipMemoryCache()
private GlideAnimationFactory<TranscodeType> animationFactory = NoAnimation.getFactory(); // 对应 animate()
private int overrideHeight = -1; // 对应 overide()
private int overrideWidth = -1; // 对应 overide()
private DiskCacheStrategy diskCacheStrategy = DiskCacheStrategy.RESULT; // 对应 diskCacheStrategy()
private Transformation<ResourceType> transformation = UnitTransformation.get(); // 对应 transform()
// 缩略图
private int placeholderId;
private Drawable placeholderDrawable;
private int errorId;
private Drawable errorPlaceholder;
private Drawable fallbackDrawable;
private int fallbackResource;
```

## .into

> `.into()` 和 `.preload()` 方法会返回 `Target` 对象。

调用 `into()` 方法后，意味着构造已经完成，将会创建请求开始加载图片，本次图片加载的流程也进入了下一个阶段--请求阶段。

当最后调用 `into()` 方法或者 `preload()` 方法时，将会从这个 `target` 中取出之前在发送的请求，然后清除掉，并构建新的 `Request` 并发起请求。

```java
public <Y extends Target<TranscodeType>> Y into(Y target) {
    Util.assertMainThread();
    Request previous = target.getRequest();
    if (previous != null) {
        previous.clear();
        requestTracker.removeRequest(previous);
        previous.recycle();
    }
    Request request = buildRequest(target);
    target.setRequest(request);
    lifecycle.addListener(target);
    requestTracker.runRequest(request);
    return target;
}
```

使用所有配置的参数构建 `Request`，主要还是有缩略图的情况要特殊处理一下，因为缩略图其实和原图是两个独立的 `Request`，两个请求同时创建出来，同时缩略图请求优先级高于原图请求，因此要特殊处理。

```java
private Request buildRequestRecursive(Target<TranscodeType> target, ThumbnailRequestCoordinator parentCoordinator) {
    if (thumbnailRequestBuilder != null) {
        if (thumbnailRequestBuilder.animationFactory.equals(NoAnimation.getFactory())) {
            thumbnailRequestBuilder.animationFactory = animationFactory;
        }
        if (thumbnailRequestBuilder.priority == null) {
            thumbnailRequestBuilder.priority = getThumbnailPriority();
        }
        if (Util.isValidDimensions(overrideWidth, overrideHeight)
                && !Util.isValidDimensions(thumbnailRequestBuilder.overrideWidth,
                        thumbnailRequestBuilder.overrideHeight)) {
          thumbnailRequestBuilder.override(overrideWidth, overrideHeight);
        }
        ThumbnailRequestCoordinator coordinator = new ThumbnailRequestCoordinator(parentCoordinator);
        Request fullRequest = obtainRequest(target, sizeMultiplier, priority, coordinator);
        // Guard against infinite recursion.
        isThumbnailBuilt = true;
        // Recursively generate thumbnail requests.
        Request thumbRequest = thumbnailRequestBuilder.buildRequestRecursive(target, coordinator);
        isThumbnailBuilt = false;
        coordinator.setRequests(fullRequest, thumbRequest);
        return coordinator;
    } else if (thumbSizeMultiplier != null) {
        ThumbnailRequestCoordinator coordinator = new ThumbnailRequestCoordinator(parentCoordinator);
        Request fullRequest = obtainRequest(target, sizeMultiplier, priority, coordinator);
        Request thumbnailRequest = obtainRequest(target, thumbSizeMultiplier, getThumbnailPriority(), coordinator);
        coordinator.setRequests(fullRequest, thumbnailRequest);
        return coordinator;
    } else {
        return obtainRequest(target, sizeMultiplier, priority, parentCoordinator);
    }
}
```

## 开始加载资源

其实这里的请求是一个更广义上的概念，虽然我们大多情况下会加载一个网络图片，但是这里的请求意思是对资源的请求，这个资源可能是一个文件、一张网络图片或者是一个 `Uri`，不要先入为主的觉得是一个网络请求。

到目前为止，经历了一系列的配置，请求被构建出来了，接下来是加载请求阶段，在这之前可以对之前的一些关键点做一下总结：

- 所有请求都会通过 `RequestManager` 来管理，他监控宿主的生命周期，并调整请求的开始和结束，这一切都是通过它内部 `RequestTracker` 来完成的，作为 `RequestManager` 的一个成员，他负责了请求队列的管理（发起和暂停等），是真正意义上的请求管理类。
- 使用配置的参数构建出来的对象是 `GenericRequest`，他是一个真正的请求，上面我们看到调用了 `requestTracker.runRequest()` 方法，里面调用了 `request.begin()` 方法，请求从这里开始。

在发起请求时会调用 `onSizeReady` 方法，在方法内部，从 `DataLoadProivder` 中取得 `ModelLoader` 进而获取到 `DataFetcher` 他是获取数据的工具类，然后还有 `ResourceTranscoder` 用来转码操作。

```java
GenericRequest.java

public void onSizeReady(int width, int height) {
    status = Status.RUNNING;
    width = Math.round(sizeMultiplier * width);
    height = Math.round(sizeMultiplier * height);
    ModelLoader<A, T> modelLoader = loadProvider.getModelLoader();
    final DataFetcher<T> dataFetcher = modelLoader.getResourceFetcher(model, width, height);
    ResourceTranscoder<Z, R> transcoder = loadProvider.getTranscoder();
    loadedFromMemoryCache = true;
    loadStatus = engine.load(signature, width, height, dataFetcher, loadProvider, transformation, transcoder,
            priority, isMemoryCacheable, diskCacheStrategy, this);
    loadedFromMemoryCache = resource != null;
}
```

## 缓存和数据源

最关键的地方是使用 `engine.load()` 触发加载，这个 `Engine` 是 `Glide` 的成员，之前被传递到 `Request` 中的，简单看一下 `load()` 方法中的伪代码，表面看起来分成了 3 步：

-  loadFromCache()
-  loadFromActiveResources()
-  EngineJob.run()

前面两步是从内存缓存中获取数据，`EngineJob`  内部则是从文件缓存和数据源中获取，因为涉及耗时操作，这部分会在线程池中执行。

```java
Engine.java

public <T, Z, R> LoadStatus load(...) {
    EngineKey key = ...
    // 内存中获取
    EngineResource<?> cached = loadFromCache(key, isMemoryCacheable);
    if (cached != null) {
        cb.onResourceReady(cached);
        return null;
    }
    // 内存中获取
    EngineResource<?> active = loadFromActiveResources(key, isMemoryCacheable);
    if (active != null) {
        cb.onResourceReady(active);
        return null;
    }
    // 该任务是不是正在执行
    EngineJob current = jobs.get(key);
    if (current != null) {
        current.addCallback(cb);
        return new LoadStatus(cb, current);
    }
    // 从文件缓存和数据源获取、包含资源的编解码等
    EngineJob engineJob = ...
    EngineRunnable runnable = new EngineRunnable(engineJob, decodeJob, priority);
    jobs.put(key, engineJob);
    engineJob.addCallback(cb);
    engineJob.start(runnable);
    return new LoadStatus(cb, engineJob);
}
```

当使用 `engine.load()` 时，就会开始获取资源，资源的获取会按照 `内存缓存 -> 文件缓存 -> 原始数据` 这样的顺序去查找,  `Glide` 采用还是常用的内存、文件两级缓存的方式，针对资源返回的位置，可以分为如下几个来源：

- `LruCache<Key,Resource>` - 基于 Lru 算法的内存缓存，在超出指定内存时，会进行清理
- `Map<Key,WeakRef<Resource>>` - 虚引用做的内存缓存，系统内存不足时回收资源
- `DiskLruCache` - 基于 Lru 算法做的文件缓存
- `Source` - 数据源，可能是文件，可能是网络图，他是最原始的目标文件

其中内存缓存和文件缓存对应的数据结构如下：

```java
Engine.java

private final MemoryCache cache;  // Lru 缓存
private final Map<Key, WeakReference<EngineResource<?>>> activeResources; //  虚引用缓存
private final LazyDiskCacheProvider diskCacheProvider; // 文件 Lru 缓存
```


### 内存缓存

内存缓存方案的选择，考虑如下几个方面：

- `LruCache` 的优点是可以有效的将内存控制在一个范围内，并优先保留最后使用的图片，缺点是当内存到达界限时就会开始回收，长列表时有可能会将正在显示的图片回收掉。
- `WeakRef` 的优点是在发生gc时，会回收只有弱引用的资源，并且通过 `cleanReferenceQueue`  我们可以观察到被回收的动作。
- 当产生大量加载事件时，会不停的开辟和回收资源占用的内存空间，造成显著的内存抖动。

更优化的内存缓存方案应该满足 3 个条件:

- 保证正在使用的图片不会被回收，如果图片不被使用了要及时缓存到别的缓存中
- 对没有使用的资源有一个上限管控，既能保证缓存的优势，又不会对内存空间造成压力
- 减少内存空间的开辟操作，能够尽量复用已经开辟好的内存空间

`Glide` 选择的缓存方案是将内存缓存分成两级，同时用一个 `BitmapPool` 来保留已经开辟的内存空间:

- **activeResouces** `Map<Key, WeakRef<Resource>>` 用来存储正在使用的资源
- **lruCache** `LruCache<Key, Resource>` 用来存储没有引用计数的资源
- **BitmapPool** `Lru<Bitmap>` 每次创建 `Bitmap`  都会优先从 `BitmapPool` 中查找可以复用的内存空间

**资源的存储**：当从文件缓存或 `Source` 中获取到目标资源时，存储到 `activeResouces` 中，因为该资源处于被获取使用的状态，当然除了 `activeResouces` 的 `WeakRef` 它还被展示它的 `View` 持有，具有一个强引用，这保证了即使发生 `gc` 也只会回收掉 `activeResouces` 中那些不被 `View` 引用的资源。

**资源的获取**：先从 `lruCache` 中获取，拿到资源后从 `lruCache` 删除，加入 `activeResouces` 中，保证该资源不会因为达到 `lruCache` 的内存上限而被回收。

**资源的回收**：当资源被触发回收时，计算引用数，没有引用就从 `activeResouces` 移除，存储到 `lruCache` 中，方便下次可以从内存中读取到。

如何判断资源没有引用了？

启动一个后台线程不停的遍历 `ReferenceQueue` 一旦发现被系统回收的资源，立刻重新构建（引用数变为0）并且通知资源被 release 了，加入到 LRU 缓存中

### 文件缓存

文件缓存选择的是 `DiskLruCache` ，是一个基于 `Lru` 算法的文件缓存方案，在 Android 上有比较广泛的应用，原理这里不展开说了。

文件缓存的查找在 `EngineRunnable` 的 `run()` 方法中：

```java
EngineRunnable.java

public void run() {
    resource = decode();
}

private Resource<?> decode() throws Exception {
    if (isDecodingFromCache()) {
        return decodeFromCache();
    } else {
        return decodeFromSource();
    }
}
```

从文件缓存读取和从数据源读取都在 `run()` 方法中实现，这里加载的策略取决于一个 `Stage` 的枚举类型，他有两个值：

- **CACHE**：表示从文件缓存中读取资源
- **SOURCE**：表示从数据源中读取资源，数据源是指网络资源、原始文件等

初始值永远是 `Stage.CACHE` 意味着总是尝试先从文件缓存中读取，注意这个枚举决定了是从 **缓存文件** 加载还是从 **数据源** 加载，不要和下面的弄混了。

当从文件缓存中读取时，又需要关注一个 `DiskCacheStrategy` 的枚举，在构建 `Glide` 的请求时我们可能会配置这个加载策略，它由两个内部字段构成

- **cacheSource**：意为缓存和加载原始缓存数据
- **cacheResult**：意为缓存和加载转换后的缓存数据，这个转换包括根据宽高压缩、transform 转换等

对于文件缓存加载来说，如果不 **强制指定** 总是优先 `cacheResult` 如果获取不到再 `cacheSource` 从原始缓存数据中加载并作转换操作，注意这个枚举决定了是加载 **原始文件** 还是加载 **转换后** 的文件。下面分为 `result` 和 `source` 两个流程比较一下差别:

- **result**：`Key` -> `Disk 获取` -> `cacheDecoder` 解码 -> `transcode()` 转码 -> 返回数据
- **source**：`Key.getOriginKey()` -> `Disk` 获取 -> `cacheDecoder` 解码 -> 使用 `Key` 写入文件缓存 -> `transcode()` 转码 -> 返回数据

对比上面的过程，主要取决于 `Key`， 这个 `Key` 是使用 `id/signature/width/height` 等组成的联合键，而 `Key.getOriginKey()` 则只比对 `id/signature` 从而达到缓存原始数据 （`Source`）和 结果数据 (`Result`)


### 数据源

当使用 `State.CACHE` 策略从文件缓存中加载失败时，将会转换成 `Stage.SOURCE`，并重新运行当前 `EngineRunnable`，从而开始从数据源加载数据

```java
private void onLoadFailed(Exception e) {
    if (isDecodingFromCache()) {
        stage = Stage.SOURCE;
        manager.submitForSource(this);
    } else {
        manager.onException(e);
    }
}
```

数据源数据的加载由 `DataFetcher` 来完成，`DataFetcher` 是在 `Glide` 初始化时注册的 `ModelLoaderFactory` 生成的，这个在后面会再细说。

借助 `fetcher.loadData()` 可以获取到数据源的原始数据，此时根据 `DiskCacheStrategy` 的不同，也分为两个不同的缓存渠道：

- **result**：`Key` -> `SourceDecoder` 解码原始数据 -> `transform()` 转换 -> 使用 `Key` 写入文件缓存 -> `transcode()` 转码
- **source**：`Key.getOriginalKey()` -> `SourceEncoder` 编码写入 `Disk` -> `Disk` 获取 -> `cacheDecoder` 解码 -> `transform()` 转换 -> 使用 `Key` 写入文件缓存 -> `transcode()` 转码


### 关于缓存的总结

上面涉及了一些编解码、转换转码等操作，我们不需要格外关注，只需要知道他是对数据的一种变换，我们在理解整个流程时，不需要太关注那些编解码、写文件等操作，只要能够理清整体缓存流程，理解其中的设计原理。

为了更好的理解，下面对一些步骤进行简单的解释：

- `Key` 是由 `id/signature/width/height` 组成的联合键，作为缓存文件的键值，他是非常精确的，而 `Key.getOriginKey()` 只关心 `id/signature`，是更通用的键值。
- `fetcher` 类似数据加载器，提前配置好的，由 `GenericLoaderFactory` 管理，用来加载数据，可以简单理解为通过网络 `Url` 转换成 `InputStream` 流的过程
- `SourceDecoder 解码原始数据` 是说将 `fetcher` 拿回来的数据进行解码，可以简单理解为将 `InputStream` 转换为 `Bitmap` 的过程
- `SourceEncoder 编码写入 Disk` 将 `fetcher` 返回的数据进行编码操作，输出到 `OutputStream`，可以简单理解为网络请求返回的 `InputStream` 转换成 `outputStream` 的过程。
- `cacheDecoder` 解码，`cacheEncoder` 是之前就配置好的一个转码器，由 `DataLoadProvider` 提供，主要负责将文件转换成目标数据，可以简单理解为将文件转为 `Bitmap` 类似的操作
- `transcode()` 转码，这是一个转码操作，也是之前就配置好的，由 `DataLoadProviderRegistry` 管理，可以简单理解为将 `Bitmap` 转换为 `GlideDrawable` 这种数据的过程
- `transform() 转换` 在构建 `Glide` 请求时配置的，是自定义，比如转换成圆角、圆形图之类的操作

针对 **内存 -> 文件 -> 数据源** 的整个流程，整理一个流程图，展示一下整个数据加载的过程。

![](http://cdn1.showjoy.com/shop/images/20181124/BX9JCXJAYZTRMOMCCI2O1543028939641.png)


## 数据转换

上面页陆陆续续提到了一些编解码前面提到了几种进行数据转换的类，这里汇总整理介绍，更直观的归纳一下这几种数据转化的作用和使用的时机。

- `ModelLoader` 负责将 `String`、`Uri`、`int`、`URL` 等等类型转化成 `InputStream` 和 `ParcelFileDescriptor`，是整个转化流程的第一步，他的原始数据类型都是我们调用 `load()` 方法里面支持的数据类型。
- `DataLoadProvider` 负责编解码的转换，它提供 `Encoder` 和 `Decoder`，可以将  `InputStream` 和 `ParcelFileDescriptor` 转换成 `Bitmap`、`GifDrawable` 等，很明显，是接着上一步进行的一个编解码操作，他的原始数据类型，就是 `ModelLoader` 的目标数据类型。
- `ResourceTranscoder` 负责结果数据类型的转换，可以将 `Bitmap` 和 `GifDrawable` 转换成 `GlideDrawable` 等，原始数据类型就是上一步的目标数据类型。


### ModelLoader

`ModelLoader` 用来针对某种数据源和返回数据创建一个 `DataFetcher` 用来请求数据，这里的请求是对数据源的请求，不仅仅限于网络请求。

```java
public interface ModelLoader<T, Y> {
    DataFetcher<Y> getResourceFetcher(T model, int width, int height);
}
```

`ModelLoader` 的类型很多，他们统一被 `GenericLoaderFactory` 管理，使用的工厂模式，在使用时使用对应的工厂类创建，同时加入缓存，获取时优先从缓存中获取，他是 `Glide` 的成员，在 `Glide` 初始化时被创建，并注册常用的 `ModelLoaderFactory`，这个类主要做两件事情：

- 管理 `ModelLoader` 工厂的注册表
- 管理 `ModelLoader` 的缓存

注册表和缓存使用两层 `Map` 结构实现，理解起来比较麻烦，例如注册表表的定义如下：

```java
Map<Class/*T*/, Map<Class/*Y*/, ModelLoaderFactory/*T, Y*/>>
```

所以 `String->InputStream` 和 `String->ParcelFileDescriptor`的  `ModelLoaderFactory` 注册进去是：

```java
Map<String,Map<InputStream, ModelLoaderFactory<String,InputStream>>>

Map<String,Map<ParcelFileDescriptor, ModelLoaderFactory<String, ParcelFileDescriptor>>>
```
这样一个结构的注册表，当你拿着 `String->InputStream` 去查找时，就能找到对应的 `Factory`。

```java
GenericLoaderFactory.java

private <T, Y> ModelLoaderFactory<T, Y> getFactory(Class<T> modelClass, Class<Y> resourceClass) {
    Map<Class/*Y*/, ModelLoaderFactory/*T, Y*/> resourceToFactories = modelClassToResourceFactories.get(modelClas
    ModelLoaderFactory/*T, Y*/ result = null;
    if (resourceToFactories != null) {
        result = resourceToFactories.get(resourceClass);
    }
    if (result == null) {
        for (Class<? super T> registeredModelClass : modelClassToResourceFactories.keySet()) {
            if (registeredModelClass.isAssignableFrom(modelClass)) {
                Map<Class/*Y*/, ModelLoaderFactory/*T, Y*/> currentResourceToFactories =
                        modelClassToResourceFactories.get(registeredModelClass);
                if (currentResourceToFactories != null) {
                    result = currentResourceToFactories.get(resourceClass);
                    if (result != null) {
                        break;
                    }
                }
            }
        }
    }
    return result;
}
```

缓存就不必说了，每次使用工厂创建耗时耗力，在内存中缓存一份，等查找时优先去内存中查找已经创建过的 `ModelLoader` 直接复用就可以了。

```java
GenericLoaderFactory.java

public class GenericLoaderFactory {
    // 注册表
    private final Map<Class/*T*/, Map<Class/*Y*/, ModelLoaderFactory/*T, Y*/>> modelClassToResourceFactories =
            new HashMap<Class, Map<Class, ModelLoaderFactory>>();
    // 缓存实现
    private final Map<Class/*T*/, Map<Class/*Y*/, ModelLoader/*T, Y*/>> cachedModelLoaders =
            new HashMap<Class, Map<Class, ModelLoader>>();
}
```

最后，这个注册表是可扩展的，你可以给他注册自己的 `ModelLoaderFactory` 来解析合适的数据类型，当然针对常用的类型，`Glide` 类中已经注册了默认的许多工厂。

```java
register(File.class, ParcelFileDescriptor.class, new FileDescriptorFileLoader.Factory());
register(File.class, InputStream.class, new StreamFileLoader.Factory());
register(int.class, ParcelFileDescriptor.class, new FileDescriptorResourceLoader.Factory());
register(int.class, InputStream.class, new StreamResourceLoader.Factory());
register(Integer.class, ParcelFileDescriptor.class, new FileDescriptorResourceLoader.Factory()
register(Integer.class, InputStream.class, new StreamResourceLoader.Factory());
register(String.class, ParcelFileDescriptor.class, new FileDescriptorStringLoader.Factory());
register(String.class, InputStream.class, new StreamStringLoader.Factory());
register(Uri.class, ParcelFileDescriptor.class, new FileDescriptorUriLoader.Factory());
register(Uri.class, InputStream.class, new StreamUriLoader.Factory());
register(URL.class, InputStream.class, new StreamUrlLoader.Factory());
register(GlideUrl.class, InputStream.class, new HttpUrlGlideUrlLoader.Factory());
register(byte[].class, InputStream.class, new StreamByteArrayLoader.Factory());
```

这样理解起来稍微有些抽象，我们来看一个网络相关的注册实现，`GlideUrl` 是一个包装，所有和网络相关的都会最终转成 `GlideUrl` ：

```java
register(GlideUrl.class, InputStream.class, new HttpUrlGlideUrlLoader.Factory());
```

看一下 `HttpUrlGlideUrlLoader` 的实现，实现一个方法 `getResourceFetcher()`，返回 `HttpUrlFetcher` 这个类主要用来使用 `HttpUrlConnection` 发起网络请求，请求图片返回 `InputStream` 流。

```java
public class HttpUrlGlideUrlLoader implements ModelLoader<GlideUrl, InputStream> {

    private final ModelCache<GlideUrl, GlideUrl> modelCache;

    public static class Factory implements ModelLoaderFactory<GlideUrl, InputStream> {
        private final ModelCache<GlideUrl, GlideUrl> modelCache = new ModelCache<GlideUrl, GlideUrl>(500);
        @Override
        public ModelLoader<GlideUrl, InputStream> build(Context context, GenericLoaderFactory factories) {
            return new HttpUrlGlideUrlLoader(modelCache);
        }
    }

    @Override
    public DataFetcher<InputStream> getResourceFetcher(GlideUrl model, int width, int height) {
        return new HttpUrlFetcher(url);
    }
}
```

总结：可以发现 `ModelLoader` 主要就是用来对源数据进行一个请求操作。

### DataLoadProvider

`DataLoadProvider` 负责编解码操作，主要是提供编解码器，将 `InputStream` 和 `File` 转换成真正意义上的图片，也就是 `Bitmap` 和 `GifDrawable` 等。

```java
public interface DataLoadProvider<T, Z> {
    ResourceDecoder<File, Z> getCacheDecoder(); // 缓存解码器，从文件缓存中解码出目标资源
    ResourceDecoder<T, Z> getSourceDecoder(); // 数据源解码器，从数据源中解码出目标资源
    Encoder<T> getSourceEncoder(); // 数据源编码器，将数据源编码到 OutputStream
    ResourceEncoder<Z> getEncoder(); // 资源解码器，将资源编码到 OutputStream
}
```

上面的接口使用了很多范型，我们借助 `StreamBitmapDataLoadProvider` 来理解他，这个类是一个实现，它是针对 `InputStream` 和 `Bitmap` 编解码的实现

```java
public class StreamBitmapDataLoadProvider implements DataLoadProvider<InputStream, Bitmap> {
    private final StreamBitmapDecoder decoder;
    private final BitmapEncoder encoder;
    private final StreamEncoder sourceEncoder;
    private final FileToStreamDecoder<Bitmap> cacheDecoder;

    public StreamBitmapDataLoadProvider(BitmapPool bitmapPool, DecodeFormat decodeFormat) {
        sourceEncoder = new StreamEncoder();
        decoder = new StreamBitmapDecoder(bitmapPool, decodeFormat);
        encoder = new BitmapEncoder();
        cacheDecoder = new FileToStreamDecoder<Bitmap>(decoder);
    }

    @Override
    public ResourceDecoder<File, Bitmap> getCacheDecoder() {
        // 缓存解码器，从文件缓存中解码出目标资源
        // 这里就是将 File 解码成 Bitmap
        return cacheDecoder;
    }

    @Override
    public ResourceDecoder<InputStream, Bitmap> getSourceDecoder() {
        // 数据源解码器，从数据源中解码出目标资源
        // 这里是将请求回来的 InputStream 解码成 Bitmap
        return decoder;
    }

    @Override
    public Encoder<InputStream> getSourceEncoder() {
        // 数据源编码器，将数据源编码到 OutputStream
        // 这里是将请求回来的 InputStream 转换成 OutputStream 用来存文件什么的
        return sourceEncoder;
    }

    @Override
    public ResourceEncoder<Bitmap> getEncoder() {
        // 资源解码器，将资源编码到 OutputStream
        // 将已经转换好的 Bitmap 转成 OutputStream 用来存文件什么的
        return encoder;
    }
}
```

也是采用注册表的方式，但是没有使用工厂，直接存储在内存中，使用 `DataLoadProviderRegistry` 来管理，内部同样是一个 `Map` 结构，存放了需要使用 `DataLoadProvider`，而且他也是 `Glide` 的成员对象。

```java
public class DataLoadProviderRegistry {
    private static final MultiClassKey GET_KEY = new MultiClassKey();
    private final Map<MultiClassKey, DataLoadProvider<?, ?>> providers =
            new HashMap<MultiClassKey, DataLoadProvider<?, ?>>();
}
```

在 `Glide` 创建的时候初始化

```java
Glide.java

dataLoadProviderRegistry = new DataLoadProviderRegistry();
StreamBitmapDataLoadProvider streamBitmapLoadProvider =
        new StreamBitmapDataLoadProvider(bitmapPool, decodeFormat);
dataLoadProviderRegistry.register(InputStream.class, Bitmap.class, streamBitmapLoadProvider);
FileDescriptorBitmapDataLoadProvider fileDescriptorLoadProvider =
        new FileDescriptorBitmapDataLoadProvider(bitmapPool, decodeFormat);
dataLoadProviderRegistry.register(ParcelFileDescriptor.class, Bitmap.class, fileDescriptorLoadProvider);
ImageVideoDataLoadProvider imageVideoDataLoadProvider =
        new ImageVideoDataLoadProvider(streamBitmapLoadProvider, fileDescriptorLoadProvider);
dataLoadProviderRegistry.register(ImageVideoWrapper.class, Bitmap.class, imageVideoDataLoadProvider);
GifDrawableLoadProvider gifDrawableLoadProvider =
        new GifDrawableLoadProvider(context, bitmapPool);
dataLoadProviderRegistry.register(InputStream.class, GifDrawable.class, gifDrawableLoadProvider);
dataLoadProviderRegistry.register(ImageVideoWrapper.class, GifBitmapWrapper.class,
        new ImageVideoGifDrawableLoadProvider(imageVideoDataLoadProvider, gifDrawableLoadProvider, bitmapPool));
dataLoadProviderRegistry.register(InputStream.class, File.class, new StreamFileDataLoadProvider());
```

### Transcoder

`Transcoder` 做的是一个转码操作，负责将一种资源转换成另一种资源。

```java
public interface ResourceTranscoder<Z, R> {
    Resource<R> transcode(Resource<Z> toTranscode);
}
```

使用 `TranscoderRegistry` 管理，注册表管理。
```java
public class TranscoderRegistry {
    private static final MultiClassKey GET_KEY = new MultiClassKey();

    private final Map<MultiClassKey, ResourceTranscoder<?, ?>> factories =
            new HashMap<MultiClassKey, ResourceTranscoder<?, ?>>();
}
```

同样是 `Glide` 的成员，在 `Glide` 创建的时候注册

```java
Glide.java

transcoderRegistry.register(Bitmap.class, GlideBitmapDrawable.class,
        new GlideBitmapDrawableTranscoder(context.getResources(), bitmapPool));
transcoderRegistry.register(GifBitmapWrapper.class, GlideDrawable.class,
        new GifBitmapWrapperDrawableTranscoder(
                new GlideBitmapDrawableTranscoder(context.getResources(), bitmapPool)));
```

### 对比

|类|管理类|原始数据类型|目标数据类型|
|:--|:--|:--|:--|:--|
|ModelLoader|GenericLoaderFactory|String(网络，文件等)</br>int(资源)</br>Uri(资源定位)</br>File(文件)</br>URL(网络)|InputStream</br>ParcelFileDescriptor|
|DataLoadProvider|DataLoadProviderRegistry|ParcelFileDescriptor</br>InputStream|Bitmap</br>GifDrawable|
|ResourceTranscoder|TranscoderRegistry|Bitmap</br>GifBitmapWrapper|GlideDrawable</br>GlideBitmapDrawable|



## 总结

我们就以加载文件里的图片为例完整的理解一下第一次的基本加载过程：

```bash
数据源
ModelLoader(DataFetcher)
转化为 InputStream
DataLoadProvider(SourceDecoder) -> 存文件缓存
解码为 Bitmap
ResourceTranscoder(transcode)
转码为 GlideDrawable
```

对于上述加载过程构造的 `RequestBuilder` 为

```java
public class GenericRequestBuilder<ModelType, DataType, ResourceType, TranscodeType> {}
public class GenericRequestBuilder<File, InputStream, Bitmap, GlideDrawable> {}
```

可以看出

```
File -> InputStream -> Bitmap -> GlideDrawable
ModelType 数据源类型
DataType 数据类型，从数据源 fetch 得到
ResourceType 资源类型，由数据类型解码得到
TranscodeType 转码类型，由资源类型转码得到
```
理解了这几种转换就能更清晰的理解 `Glide` 的内部原理