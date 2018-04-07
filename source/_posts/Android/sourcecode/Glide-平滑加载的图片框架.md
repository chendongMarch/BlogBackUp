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
photos: 'http://olx4t2q6z.bkt.clouddn.com/18-3-21/70223294.jpg'
location: 杭州尚妆
date: 2018-03-21 09:58:00
---

本文按照 `Glide` 常用的如下用法来分析源码：

```java
Glide.with(context).load("url").thumbnail().into(imageView);
```

<!--more-->
s
根据上面的代码，将内容分为以下几个章节：

- `Glide` - 强大的顶层管理类
- `Glide.with()` - 创建及缓存 `RequestManager`
- `.load(url)` - 创建 `RequestBuilder`
- `.thumbnail()` - 配置 `RequestBuilder`
- `.into(target)` - 发起 `Request`



## 强大的 Glide

`Glide` 是一个单例对象，在这个框架中功能相对复杂，每个功能都由对应的类实现，但是他们都被 `Glide` 类统一管理，`Glide` 的构造方法很长，主要是初始化一些内部持有的管理类，类比来看的 `Glide` 就类似高级管理阶层，下属还有很多次级管理层。

简单看一下重要的成员变量：

```java
Glide.java

GenericLoaderFactory loaderFactory;   // 工厂管理类，负责 ModelLoader 的注册和缓存实现
TranscoderRegistry transcoderRegistry = new TranscoderRegistry(); // 工厂管理类，负责 DataLoadProvider 的管理
DataLoadProviderRegistry dataLoadProviderRegistry; // DataLoadProvider 注册管理

Engine engine;
BitmapPool bitmapPool;
MemoryCache memoryCache;
DecodeFormat decodeFormat;
ImageViewTargetFactory imageViewTargetFactory = new ImageViewTargetFactory()


CenterCrop bitmapCenterCrop;
GifBitmapWrapperTransformation drawableCenterCrop;
FitCenter bitmapFitCenter;
GifBitmapWrapperTransformation drawableFitCenter;
Handler mainHandler;
BitmapPreFiller bitmapPreFiller;
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


## 开始加载请求

到目前为止，经历了一系列的配置，请求被构建出来了，接下来是加载请求阶段，在这之前可以对之前的一些关键点做一下总结：

- 所有请求都会通过 `RequestManager` 来管理，他监控宿主的生命周期，并调整请求的开始和结束，这一切都是通过它内部 `RequestTracker` 来完成的，作为 `RequestManager` 的一个成员，他负责了请求队列的管理（发起和暂停等），是真正意义上的请求管理类。
- 使用配置的参数构建出来的对象是 `GenericRequest`，他是一个真正的请求，上面我们看到调用了 `requestTracker.runRequest()` 方法，里面调用了 `request.begin()` 方法，请求从这里开始。

在发起请求时会调用 `onSizeReady` 方法，在方法内部，从 `DataLoadProivder` 中取得 `ModelLoader` 进而获取到 `DataFetcher` 他是获取数据的工具类，然后还有 `ResourceTranscoder` 用来转码操作。

```java
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

最关键的地方是使用 `engine.load()` 触发加载，这个 `Engine` 是 `Glide` 的成员，之前被传递到 `Request` 中的。

## 缓存

总共分了多级缓存

- LruCache<Key,Resource> 缓存，在超出指定内存时，会进行清理
- Map<Key,WeakRef<Resource>> 虚引用做的缓存，正在使用的照片会被加入该缓存，除非系统内存不够了，尽量避免被回收
- DiskLruCache 做的文件缓存
- Source 源，可能是文件，可能是网络图，他是最原始的目标文件


DecodeCahceResult 从文件缓存中获取

- load(key) 
- transcode

DecodeCahceSource

- laod(key.origin)
- transform
- saveToResultCache
- transcode

DecodeSource

- fetcher.loadData()
- decodeSource
	- source -> saveData -> put(key.origin)
	- result -> sourceDecoder.decode()
- transform
- saveToResultCache
- transcode



## 数据转换

前面提到了几种进行数据转换的类，这里汇总整理介绍，更直观的归纳一下这几种数据转化的作用和使用的时机。

- `ModelLoader` 负责将 `String`、`Uri`、`int`、`URL` 等等类型转化成 `InputStream` 和 `ParcelFileDescriptor`，是整个转化流程的第一步，他的原始数据类型都是我们调用 `load()` 方法里面支持的数据类型。
- `DataLoadProvider` 负责编解码的转换，它提供 `Encoder` 和 `Decoder`，可以将  `InputStream` 和 `ParcelFileDescriptor` 转换成 `Bitmap`、`GifDrawable` 等，很明显，是接着上一步进行的一个编解码操作，他的原始数据类型，就是 `ModelLoader` 的目标数据类型。
- `ResourceTranscoder` 负责结果数据类型的转换，可以将 `Bitmap` 和 `GifDrawable` 转换成 `GlideDrawable` 等，原始数据类型就是上一步的目标数据类型。


### ModelLoader

`ModelLoader` 可以理解为数据对象加载器，他负责将某种数据类型转换为另一种数据类型，例如当调用 `load("url")` -> `fromString()` -> `loadGeneric(String.class)` 这样一个调用链时，结合上面的代码我们其实获得的是如下两个 `Loader`，分别是将 `String->InputStream` 和 `String-> ParcelFileDescriptor`，他们作为 `DrawableTypeRequest` 的构造器参数，在后面会被使用到。

```java
ModelLoader<String, InputStream> streamModelLoader = Glide.buildStreamModelLoader(modelClass, context);
ModelLoader<String, ParcelFileDescriptor> fileDescriptorModelLoader = 	Glide.buildFileDescriptorModelLoader(modelClass, context);
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
Glide.java
---	
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

### DataLoadProvider

`DataLoadProvider` 负责编解码操作，主要是提供编解码器，将 `InputStream` 和 `File` 转换成真正意义上的图片，也就是 `Bitmap` 和 `GifDrawable` 等。
 
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

将 `Bitmap` 和 `GifDrawable` 转换成 `GlideDrawable`。

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


















