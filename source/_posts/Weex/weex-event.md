---
layout: post
title: '「Weex」Better Weex - 多页面事件交互'
categories:
  - weex
tags: weex
keywords:
  - weex
abbrlink: 93c948f2
date: 2018-07-24 10:46:00
---

`Weex` 本身的设计初衷是单页面应用，本身不具有多页面之间通信的能力，但因为客户端应用的特殊性，多页面的通信需求十分常见，比如我在设置页面更新了用户数据，同时需要刷新首页等场景，当遭遇这些场景时，多页面的事件交互就会变得非常吃力。

为了实现多页面通信的需求，参考 `Android` 中一个比较著名、比较成熟的基于事件总线的通信类库 `EventBus` 的设计原理，对 `Weex` 事件机制进行扩展。

<!--more-->


## BroadcastChannel

在 `Weex` 中有一个 [`BroadcastChannel`](https://weex.incubator.apache.org/cn/references/broadcast-channel.html) 的 `API` 用来实现页面间的通信，在原生部分使用 `WebSocketModule` 实现，不过经过实验发现，注册和发送没有什么大问题，不过在取消注册这块做的有漏洞，出现多次页面销毁但是无法取消对事件监听的情况（可能是当时尝试的时候版本低一些），主要是因为 `module` 的生命周期没能和 `weex` 页面实例更好的绑定起来，而且它是基于 `W3C` 的标准设计的，也没有实现类似粘滞事件这种功能的支持。

因此我们暂时不考虑使用 `BroadcastChannel` 实现多页面通信的需求，希望以后他会变得更完善。

## globalEvent

```js
const globalEvent = weex.requireModule('globalEvent');
```

他是 `Weex` 中 **页面内** 通信的接口，是 `native` 和 `weex` 通信的通道，可以用一个 `key` 作为标示符，触发当前 `weex` 页面中对 `key` 事件感兴趣的的方法，关于 `weex` 相关的内容这里不细说。

```java
((WXSDKInstance)instance).fireGlobalEventCallback(key, params)
```

前面说过了，他是负责 `native` 和 `weex` 通信的，我们姑且叫他 **页面内** 通信，因为 `weex` 和 `native` 是完全分离的，当我们想在 `native` 部分向 `weex` 发送事件和数据，就需要依赖该接口，比如，在页面 `resume` 时，也通知 `weex` 一个 `resume` 事件，让 `weex` 也可以察觉这一时机。


## 实现多页面通信


实现原理类似 `EventBus`，关于 `EventBus` 的源码解析，可以参考[这篇博客](../4e49ab58)，因为我们是基于 `Weex` 进行设计，可以借助 `Weex` 已经存在的数据和 `API`，所以实现起来要简单一些。

- 关于注册表的维护，在 `EventBus` 中是用列表存储了订阅者的强引用，通过注册和反注册的方法从列表中增加和删除，这样一来因为我们需要存储每个对象的引用，占据内存空间大，而且容易内存泄漏，好在 `Weex` 每个 `Weex` 页面渲染后都会唯一生成一个 `instanceId`，所以我们只要维护一个 `instanceId` 的列表，然后在事件发送时动态检索对应的 `Weex` 实例，而 `Weex` 实例的维护还是交给 `WeexSDK` 去管理，就简单和稳定很多。

- 关于事件的发送，在 `EventBus` 中需要遍历注册表，找到对应接受者然后执行他的方法，而在 `Weex` 中，我们拿到 `Weex` 实例后借助 `globalEvent` 发送事件的方法将事件发送到 `Weex` 即可。

因此，首先我们会维护一个 `event` - `instanceId` 的注册表，它的意义是 **事件 - 对该事件感兴趣的Weex实例列表** 的映射，每个 `event` 后面跟着一个 `Set<instanceId>` 里面存储了对该 `Event` 页面感兴趣的 `Weex` 实例，比如 事件 `updateUser`，对该事件感兴趣的页面有 `A(instanceId = 132)` 页面，`B(instanceId = 133)` 页面，`C(instanceId = 135)` 页面。

```kotlin
private val mEventInstanceIdMap by lazy { mutableMapOf<String, MutableSet<String>>() }
```

### 注册事件

我们需要在 `Module` 中增加注册方法，供 `Weex` 调用，调用方法大致如下：

```js
const event = weex.requireModule('event');
event.registerEvent('updateUser',(data) => {
	// get user data
});
```

当 `Weex` 那边发起注册时，根据 `event` 拿到对应的 `instanceId` 列表，并将当前页面的 `instanceId` 存入列表中，完成 `事件-Weex实例` 的映射。

```java
fun registerEvent(event: String?, instantId: String?) {
    // check...
    val nonNullEvent = event ?: return
    val registerInstantIds = mEventInstanceIdMap[nonNullEvent] ?: mutableSetOf()
    registerInstantIds.add(instantId)
    mEventInstanceIdMap[nonNullEvent] = registerInstantIds
}
```


### 发送事件

同样需要在 `Module` 中提供接口方法，大致如下：

```js
const event = weex.requireModule('event');
event.postEvent('updateUser', { data: user });
```

发送事件时，根据 `event` 拿到关注该事件的 `instanceId` 列表，循环从 `WeexSDK` 中取出真正的 `WXSDKInstance` 对象，再利用 `globalEvent` 将事件发送给 `Weex`，达到页面间通信的目的。

```kotlin
fun postEvent(event: String, params: Map<String, Any>) {
    if (WXSDKManager.getInstance() == null) {
        log("post event WXSDKManager.getInstance() == null")
        return
    }
    val renderManager = WXSDKManager.getInstance().wxRenderManager
    if (renderManager == null) {
        log("post event WXSDKManager.getInstance().wxRenderManager == null")
        return
    }
    val registerInstantIds = mEventInstanceIdMap[event] ?: listOf<String>()
    val allInstants = renderManager.allInstances
    for (instance in allInstants) {
        // 该事件被该 instant 注册过
        if (instance != null
                && !instance.instanceId.isNullOrEmpty()
                && registerInstantIds.contains(instance.instanceId)) {
            instance.fireGlobalEventCallback(event, params)
        }
    }
}
```

### 取消注册

我们需要在 `Module` 中增加接口方法，供 `Weex` 调用，调用方法大致如下：

```js
const event = weex.requireModule('event');
event.unRegisterEvent('updateUser');
```

当页面销毁时，或者使用者人为触发反注册事件时，会将该页面的 `instantceId` 移除掉，那么该页面将不会再接受到事件通知。

这里多做了一次校验，当不传入 `event` 时，将会删除当前页面注册的所有事件，而当 `event` 指定时，会从关注该 `event` 的 `instanceId` 列表中将本页面 `instanceId` 删除。

```kotlin
fun unRegisterEvent(event: String?, instanceId: String?) {
    val nonNullId = instanceId ?: return
    if (event.isNullOrBlank()) {
        // 删除本页面所有的事件
        for (mutableEntry in mEventInstanceIdMap) {
            if (mutableEntry.value.isNotEmpty()) {
                mutableEntry.value.remove(nonNullId)
            }
        }
    } else {
        // 删除本页面的指定事件
        val mutableSet = mEventInstanceIdMap[event]
        mutableSet?.remove(nonNullId)
    }
```

### 粘滞事件

考虑到在 `Weex` 使用场景不多，暂时没有实现



