---
layout: post
title: '「源码」看看 EventBus'
category: Android
tags:
  - Android
  - SourceCode
keywords:
  - Android
  - EventBus
abbrlink: 4e49ab58
photos: 'http://olx4t2q6z.bkt.clouddn.com/18-5-25/35272473.jpg'
location: 杭州尚妆
date: 2018-05-25 14:11:00
---

[`EventBus`](https://github.com/greenrobot/EventBus) 是基于观察者模式的发布/订阅事件总线，它让组件间的通信变得更加简单。类似广播系统，不过 `EventBus`  所有的订阅和发送都是在内存层面的，使用起来远比广播简单，也更容易管理。
 

<!--more-->

![](https://user-gold-cdn.xitu.io/2018/5/27/163a200cd9fd0892?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)


先说明在事件总线中的几个关键词：

- 事件发送者，发出事件的人
- 订阅者，处理事件的人
- 订阅者中处理事件的方法，因为每个订阅者感兴趣的事件有多种，因此会有多个处理事件的方法
- 订阅，是一个名词，指的是一种关系，一个订阅指的是某个订阅者中的处理某个事件的方法，由订阅者和事件类型唯一确定。

## 订阅事件注册

当希望接受到事件时，需要在 `onCreate()` 执行 `register()` 方法，这里的 `subscriber` 通常是我们的 `activity`，在注册方法中会检索当前类的 `Class` 中声明的接受事件的方法，并将他们注册到对应的映射中。

```java
public void register(Object subscriber) {
    Class<?> subscriberClass = subscriber.getClass();
    List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriberClass);
    synchronized (this) {
        for (SubscriberMethod subscriberMethod : subscriberMethods) {
            subscribe(subscriber, subscriberMethod);
        }
    }
}
```

内存中存储的数据结构有如下几个：

```java
// 事件 - List<订阅(Subscription)> 每个订阅由订阅者、事件类型唯一确定
private final Map<Class<?>, CopyOnWriteArrayList<Subscription>> subscriptionsByEventType;
// 订阅者 - List<关注的事件> 每个订阅者可能关注多个事件
private final Map<Object, List<Class<?>>> typesBySubscriber;
// 事件对应下的粘滞事件
private final Map<Class<?>, Object> stickyEvents;
```
### 查找订阅方法列表

当执行 `register()` 方法时，会借助 `SubscriberMethodFinder` 类从注册的对象的 `Class` 中查找。

```java
List<SubscriberMethod> findSubscriberMethods(Class<?> subscriberClass) {
    //  从缓存中找是否已经检索过了，有缓存就直接返回
    List<SubscriberMethod> subscriberMethods = METHOD_CACHE.get(subscriberClass);
    if (subscriberMethods != null) {
        return subscriberMethods;
    }
    // 是否忽略索引功能，忽略的话会直接使用反射的方法搜索，否则会检测有没有相关的索引可以使用
    if (ignoreGeneratedIndex) {
        subscriberMethods = findUsingReflection(subscriberClass);
    } else {
    	// 支持索引的情况，会优先从索引中查找，加快查找的速度
        subscriberMethods = findUsingInfo(subscriberClass);
    }
    if (subscriberMethods.isEmpty()) {
        // 没有找到任何的订阅方法将会抛出异常，所以至少要用注解订阅一个方法
    } else {
    	// 针对这个 class 查找到订阅的方法列表，存缓存，下次更快的返回
        METHOD_CACHE.put(subscriberClass, subscriberMethods);
        return subscriberMethods;
    }
}
```

因为我们不考虑索引的情况，最终查找方法都会走到方法 `findUsingReflectionInSingleClass`，内部的原理相对简单，遍历该类的所有方法，找到共有的、只有一个参数、且带有 `@Subscribe` 注解的方法，存储到列表中。

```java
private static final int MODIFIERS_IGNORE = Modifier.ABSTRACT | Modifier.STATIC | BRIDGE | SYNTHETIC;

private void findUsingReflectionInSingleClass(FindState findState) {
    Method[] methods;
    methods = findState.clazz.getDeclaredMethods();    
    for (Method method : methods) {
        int modifiers = method.getModifiers();
        // 共有的方法 & 不是静态、抽象、不是编译生成的方法
        if ((modifiers & Modifier.PUBLIC) != 0 && (modifiers & MODIFIERS_IGNORE) == 0) {
            Class<?>[] parameterTypes = method.getParameterTypes();
            // 参数长度只能是1
            if (parameterTypes.length == 1) {
                Subscribe subscribeAnnotation = method.getAnnotation(Subscribe.class);
                // 方法上面带有 @Subscribe 注解
                if (subscribeAnnotation != null) {
                    Class<?> eventType = parameterTypes[0];
                    if (findState.checkAdd(method, eventType)) {
                        ThreadMode threadMode = subscribeAnnotation.threadMode();
                        findState.subscriberMethods.add(new SubscriberMethod(method, eventType, threadMode,
                                subscribeAnnotation.priority(), subscribeAnnotation.sticky()));
                    }
                }
            }
        }
    }
}
```

这个过程是一个循环，每次都会向上查找当前类的父类，知道到达 `java` 内置的类中，这就意味着，父类中声明的订阅方法，在子类实例中也会接收到。查找的结果最终会生成一个 `SubscriberMethod` 的列表，这个类中存储了订阅方法的全部信息，数据结构如下：

```java
public class SubscriberMethod {
    final Method method; // 当前的方法，可执行
    final ThreadMode threadMode; // 线程类型
    final Class<?> eventType; // 参数的类型，也就是他订阅的事件的类型
    final int priority; // 优先级
    final boolean sticky; // 是否是粘滞事件
    String methodString; // 方法的字符串
}
```

### 订阅到映射中

```java
// 事件 - List<订阅(Subscription)> 每个订阅由订阅者、事件类型唯一确定
private final Map<Class<?>, CopyOnWriteArrayList<Subscription>> subscriptionsByEventType;
// 订阅者 - List<关注的事件> 
private final Map<Object, List<Class<?>>> typesBySubscriber;
```

订阅的过程就是根据订阅者 `Subscriber` 及该订阅者的某个处理事件的方法 `SubscriberMethod` 来生成 `Subscription` 并且存储到映射当中。

```java
private void subscribe(Object subscriber, SubscriberMethod subscriberMethod) {
    // 存储到 事件 - List<订阅> 映射中
    Class<?> eventType = subscriberMethod.eventType;
    Subscription newSubscription = new Subscription(subscriber, subscriberMethod);
    CopyOnWriteArrayList<Subscription> subscriptions = subscriptionsByEventType.get(eventType); // ... 不存在则创建新的
    int size = subscriptions.size();
    for (int i = 0; i <= size; i++) {
        if (i == size || subscriberMethod.priority > subscriptions.get(i).subscriberMethod.priority) {
            subscriptions.add(i, newSubscription);
            break;
        }
    }
    // 存储到 订阅者 - List<关注的事件> 映射中
    List<Class<?>> subscribedEvents = typesBySubscriber.get(subscriber); // ... 不存在则创建新的
    subscribedEvents.add(eventType);
    // ...
    // 对 Sticky Event 的处理，后面单独说
}
```


### 取消注册

由于事件总线的机制基于内存实现，所有的订阅都会存储在内存中，因此必须在合适的时机取消注册，来释放占用的内存空间。

当取消注册时：

- 借助之前存储的 `订阅者-List<关注事件>` 的映射快速的获取到，当前订阅者感兴趣的事件列表。
- 然后遍历事件列表，从 `事件-List<订阅>` 的映射中，删除所有的订阅。
- 最后将当前订阅者从 `订阅者-List<关注事件>` 删除，完成取消订阅的过程。

获取当前订阅者关注的全部事件，遍历取消注册。

```java
public synchronized void unregister(Object subscriber) {
    List<Class<?>> subscribedTypes = typesBySubscriber.get(subscriber);
    if (subscribedTypes != null) {
        for (Class<?> eventType : subscribedTypes) {
            unsubscribeByEventType(subscriber, eventType);
        }
        typesBySubscriber.remove(subscriber);
    } else {
    }
}

// 从订阅列表中删除对应的订阅
private void unsubscribeByEventType(Object subscriber, Class<?> eventType) {
    List<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
    if (subscriptions != null) {
        int size = subscriptions.size();
        for (int i = 0; i < size; i++) {
            Subscription subscription = subscriptions.get(i);
            if (subscription.subscriber == subscriber) {
                subscription.active = false;
                subscriptions.remove(i);
                i--;
                size--;
            }
        }
    }
}
```

## 发送事件

当需要发送事件使用 `EventBus` 的 `post()` 方法。

借助  `ThreadLocal` 每个线程单独维护一个、且仅一个 `PostingThreadState` 对象，这个对象的数据结构如下, 内部存储了当前发送事件状态的的一些关键信息。

```java
final static class PostingThreadState {
    final List<Object> eventQueue = new ArrayList<Object>(); // 事件队列
    boolean isPosting; // 是否正在发送事件，是的话不需要启动循环读取事件
    boolean isMainThread; // 是否是主线程
    Subscription subscription; // 一个订阅
    Object event; // 当前的事件
    boolean canceled; // 是否被取消
}
```
获取本线程的 PostingThreadState 对象，进行初始化，并开始轮询处理队列中的事件。

```java
public void post(Object event) {
    PostingThreadState postingState = currentPostingThreadState.get();
    List<Object> eventQueue = postingState.eventQueue;
    eventQueue.add(event);
    if (!postingState.isPosting) {
        postingState.isMainThread = Looper.getMainLooper() == Looper.myLooper();
        postingState.isPosting = true;
        try {
            // 从队列中循环读取事件处理
            while (!eventQueue.isEmpty()) {
                postSingleEvent(eventQueue.remove(0), postingState);
            }
        } finally {
            postingState.isPosting = false;
            postingState.isMainThread = false;
        }
    }
}
```
继续往深里面看 `postSingleEvent()` 方法，他每次处理一个从队列中取出来的事件，这里做了一个区分，是否支持继承，这个值默认是 `true`，支持继承时，如果对当前事件的父类、接口对应的事件感兴趣，那么他也可以处理该事件。例如当前要处理 A 事件，A 继承自 B，同时实现 C 接口，能处理 B,C 事件的订阅者将也会参与处理此 A 事件。

```java
private void postSingleEvent(Object event, PostingThreadState postingState) throws Error {
    Class<?> eventClass = event.getClass();
    boolean subscriptionFound = false;
    if (eventInheritance) {
        // 向父类搜索，将父类、接口全部查找到
        List<Class<?>> eventTypes = lookupAllEventTypes(eventClass);
        int countTypes = eventTypes.size();
        for (int h = 0; h < countTypes; h++) {
            Class<?> clazz = eventTypes.get(h);
            subscriptionFound |= postSingleEventForEventType(event, postingState, clazz);
        }
    } else {
        subscriptionFound = postSingleEventForEventType(event, postingState, eventClass);
    }
    if (!subscriptionFound) {
        // 没有找到订阅的方法，处理分支
    }
}
```


### 事件订阅者排队处理

接下来会走 `postSingleEventForEventType()` 方法，这个方法负责找到对这个事件感兴趣的 **订阅 Subscription** 列表， `Subscription` 里面包含了订阅者、处理对应事件的方法等信息。

拿到列表之后便循环将事件给列表中的订阅依次处理，在之前注册时，是有一个优先级别的，优先级高的将会先获得处理事件的权利。

优先级别较高的处理者可以停止事件的传递，只需要抛出一个异常，被 `finally` 块捕捉后，就会中断轮询，从而终止事件的传递。

```java
private boolean postSingleEventForEventType(Object event, PostingThreadState postingState, Class<?>
    CopyOnWriteArrayList<Subscription> subscriptions;
    synchronized (this) {
        subscriptions = subscriptionsByEventType.get(eventClass);
    }
    // 遍历所有的订阅，处理事件
    if (subscriptions != null && !subscriptions.isEmpty()) {
        for (Subscription subscription : subscriptions) {
            postingState.event = event;
            postingState.subscription = subscription;
            boolean aborted = false;
            try {
                // 让 subscription 处理 event
                postToSubscription(subscription, event, postingState.isMainThread);
                aborted = postingState.canceled;
            } finally {
                // 如果优先级别较高的处理者异常，则后续处理者将无法处理该事件
                postingState.event = null;
                postingState.subscription = null;
                postingState.canceled = false;
            }
            // 退出轮询
            if (aborted) {
                break;
            }
        }
        return true;
    }
    return false;
}
```

### 分发线程处理者执行

处理事件的最后一步，是 `postToSubscription()` 他负责将事件的处理分发到不同的线程队列中，在添加订阅注解 `@Subscribe` 时可以指定 `threadMode`，这极大的方便了我们在事件传递后切换不同线程处理事件，例如我们常常要在子线程处理数据，而通知主线程更新 `UI`，使用 `EventBus` 只需要指定 `@Subscribe(threadMode=ThreadMode.Main)` 则在处理事件时所有操作在内部便被切换到了主线程，真正做到了对线程切换的无感知。

分为了如下几种类型：

- `POSTING` 发送线程，或者说是当前线程更贴切一些，在其他类库中通常叫 `Immediate`， 也就是不用切换线程。
- `MAIN` 主线程，不解释。
- `BACKGROUND` 后台线程，如果发送线程是主线程，则开辟新的线程执行，否则将在当前线程执行。
- `ASYNC` 异步线程，无论怎样，总是开启新的子线程去执行。

这里就要看一下几个处理者 `HandlerPoster`/`BackgroundPoster`/`AsyncPoster` 实现原理大致相同，内部维护一个队列，不停的把里面的事件取出来处理。

- `HandlerPoster` 是基于 `Handler` 实现对队列的轮询。
- `BackgroundPoster` 则是用死循环来做的，谁让人家有自己的线程呢。
- `AsyncPoster` 就更富了，根本不轮询，每次都是一个新的线程。

```java
private void postToSubscription(Subscription subscription, Object event, boolean isMainThread) {
    switch (subscription.subscriberMethod.threadMode) {
        case POSTING:
            invokeSubscriber(subscription, event);
            break;
        case MAIN:
            if (isMainThread) {
                invokeSubscriber(subscription, event);
            } else {
                mainThreadPoster.enqueue(subscription, event);
            }
            break;
        case BACKGROUND:
            if (isMainThread) {
                backgroundPoster.enqueue(subscription, event);
            } else {
                invokeSubscriber(subscription, event);
            }
            break;
        case ASYNC:
            asyncPoster.enqueue(subscription, event);
            break;
    }
}
```

最终调用的 `invokeSubscriber()` 很简单就是利用反射调一下对应的 `method`

```java
subscription.subscriberMethod.method.invoke(subscription.subscriber, event);
```

## 粘滞事件的实现

我把  `Sticky Event` 翻译成 **粘滞事件** 不知道对不对，他的出现主要是因为我们需要处理事件是总是要先注册再发送事件，根本原因在于当一个事件发出时，他的生命周期很短，所有对他感兴趣的订阅者处理完了之后他就被抛弃了，后面的订阅者再感兴趣也没用，因为早就被清理啦。

要解决这个问题也很简单，就是延长事件的生命周期，即使大家都不理他了，他也能顽强的活着，万一后面还有人对他感兴趣呢。所以实现的原理也就很明了了，找个列表把它全部存起来，除非你手动给删除，否则就 **粘不拉几** 的附着在你的内存里，等着他的真命天子出现。


```java
// 事件类型 - 事件实例
private final Map<Class<?>, Object> stickyEvents;
// 发送粘滞事件时，先存起来给后面的人用，然后按照常规流发送出去
public void postSticky(Object event) {
    synchronized (stickyEvents) {
        stickyEvents.put(event.getClass(), event);
    }
    post(event);
}
```
还要提供一个渠道，让新加入进来的订阅者能够察觉到这里有粘滞事件的存在，如果感兴趣也可以处理它。这个时机就是注册时，当一个订阅者被添加到注册表中时，此时如果存在粘滞事件，用当前订阅者感兴趣的事件为 `key` 获取存在的粘滞事件，如果有感兴趣的就临幸一下。于是可以完善一下之前未说完的 `register()` 方法：

- 首先要求当前订阅者的处理事件的方法要对粘滞事件感兴趣，这个在注解上可以声明。
- 继承，如果支持继承，当前事件的子类粘滞事件都会被取出来检查是否可以被处理。

```java
private void subscribe(Object subscriber, SubscriberMethod subscriberMethod) {
    // ... 前面这块说过了
    // 这个订阅者的这个订阅方法是对粘滞事件感兴趣的
    if (subscriberMethod.sticky) {
        // 事件是否继承
        if (eventInheritance) {
            Set<Map.Entry<Class<?>, Object>> entries = stickyEvents.entrySet();
            // 当前事件的子类粘滞事件都会被取出来检查是否可以被处理
            for (Map.Entry<Class<?>, Object> entry : entries) {
                Class<?> candidateEventType = entry.getKey();
                if (eventType.isAssignableFrom(candidateEventType)) {
                    Object stickyEvent = entry.getValue();
                    checkPostStickyEventToSubscription(newSubscription, stickyEvent);
                }
            }
        } else {
            Object stickyEvent = stickyEvents.get(eventType);
            checkPostStickyEventToSubscription(newSubscription, stickyEvent);
        }
    }
}
```

接下来的 `checkPostStickyEventToSubscription()` 就会调用前面已经说过的 `postToSubscription()` 方法，开始发送到不同的线程中执行，这部分和普通的事件是一样的啦。

## 理解事件的继承

粘滞事件这里也出现了一个关于事件继承的检索，在上一节也出现了一次，单独拿出来说一下异同之处。

> **可以类比函数入参的限制，如果一个方法声明中参数是父类，那么传参时可以传递子类对象进去，声明了子类的话，是不能传递父类对象的。**

举个例子，设定下场景，我们现在有事件基类  `BaseEvent` 和一个事件子类 `ImplEvent` 是继承关系。

第一种场景，发送普通事件，我发送了一个 `ImplEvent`，因为我发的是个子类事件，也就是说所有声明关注 `BaseEvent` 的订阅者也都可以将当前事件作为入参，所以向上检索对 `ImplEvent` 父类、父接口感兴趣的订阅者去执行。

第二个场景，发送粘滞事件，发送一个 `BaseEvent` 的粘滞事件，因为是在注册时触发执行，那么说明当前订阅者对 `BaseEvent` 感兴趣，既然他的入参是父类事件，那么子类事件也同样可以作为他的处理事件方法的入参，于是检索所有粘滞事件找到所有 `BaseEvent` 的子类事件都交给当前订阅者处理。


## Weex 事件机制

在 `Weex` 中有一个 [`BroadcastChannel`](https://weex.incubator.apache.org/cn/references/broadcast-channel.html) 的 `API` 用来实现页面间的通信，在原生部分使用 `WebSocketModule` 实现，不过经过实验发现，注册和发送没有什么大问题，不过在取消注册这块做的有漏洞，出现多次页面销毁但是无法取消对事件监听的情况（可能是当时尝试的时候版本低一些），主要是因为 `module` 的生命周期没能和 `weex` 页面实例更好的绑定起来，而且它是基于 `W3C` 的标准设计的，也没有实现类似粘滞事件这种功能的支持。

最后决定根据事件总线的机制来尝试实现页面之间的通信，在 `Weex` 中有一个 **页面内** 通信的接口，他是 `native` 和 `weex` 通信的通道，可以用一个 `key` 作为标示符，触发当前 `weex` 页面中对 `key` 事件感兴趣的的方法，关于 `weex` 相关的内容这里不细说。

```java
((WXSDKInstance)instance).fireGlobalEventCallback(key, params)
```
实现原理类似 `EventBus`，不过因为基于 `weex` 就没那么复杂，同样需要维护一个注册表，相对于 `EventBus` 要对订阅者强引用持有，这里使用了每个 `weex` 页面唯一的 `instanceId` 作为标记，存储这个标记而不是存储真正的 `WXSDKInstance` 对象，避免内存泄漏。

```kotlin
private val mEventInstanceIdMap by lazy { mutableMapOf<String, MutableSet<String>>() }
```

注册，当 `weex` 那边发起注册时，拿到对应的 `instanceId ` 存储到映射中。

```java
// 注册接受某事件
// event.registerEvent('myEvent')
// globalEvent.addEventListener('myEvent', (params) => {});
fun registerEvent(key: String?, instantId: String?) {
    // do check...
    val nonNullKey = key ?: return
    val registerInstantIds = mEventInstanceIdMap[nonNullKey] ?: mutableSetOf()
    registerInstantIds.add(instantId)
    mEventInstanceIdMap[nonNullKey] = registerInstantIds
}
```

发送事件时，根据事件的 `key` 拿到对他关注的订阅者的 `instanceId` 列表，循环从 `weex sdk` 中取出真正的 `WXSDKInstance` 对象，再利用页面内通信的 `API` 将事件发送给指定页面，达到页面间通信的目的。

```kotlin
// 发送事件
// event.post('myEvent',{isOk:true});
fun postEvent(key: String, params: Map<String, Any>) {
    // do check...
    val registerInstantIds = mEventInstanceIdMap[key] ?: listOf<String>()
    val allInstants = renderManager.allInstances
    for (instance in allInstants) {
        // 遍历找到订阅的 instanceId 进而拿到 weex 实例发送页面内事件
        if (instance != null
                && !instance.instanceId.isNullOrEmpty()
                && registerInstantIds.contains(instance.instanceId)) {
            instance.fireGlobalEventCallback(key, params)
        }
    }
}
```
当页面销毁时，同时自动取消注册，释放内存和避免不必要的事件触发

```kotlin
override fun onWxInstRelease(weexPage: WeexPage?, instance: WXSDKInstance?) {
    val nonNullId = instance?.instanceId ?: return
    for (mutableEntry in mEventInstanceIdMap) {
        if (mutableEntry.value.isNotEmpty()) {
            mutableEntry.value.remove(nonNullId)
        }
    }
}
```

最后，目前只是一个简单的实现，能够基本实现页面间通信的需求，不过还需要更多地调研和其他端同学的配合，相信会越来越完善。
       