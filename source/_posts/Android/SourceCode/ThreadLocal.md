---
layout: post
title: 「源码」ThreadLocal 存储线程本地变量
category: Android
tags:
  - Android
  - SourceCode
keywords:
  - ThreadLocal
  - ThreadLocalMap
abbrlink: f69b558f
photos: 'https://images.pexels.com/photos/789433/pexels-photo-789433.jpeg'
location: 杭州尚妆
date: 2018-08-13 15:31:00
---


`ThreadLocal` 顾名思义就是 **线程本地数据** 的意思，用于在不同线程之间独立的存取数据，这个数据在每个线程都有一个副本，不同线程存取过程不会相互影响。表现出来的效果就是，我在 `A` 线程存了一个 `a`，则我只能在 `A` 线程再取到、更改这个 `a`，我在 `B` 线程是拿不到这个值的。

## 推荐阅读

> [ThreadLocal的使用及原理分析](https://mp.weixin.qq.com/s/bxIkMaCQ0PriZtSWT8wrXw)

## 数据的存储结构

> 线程独立的数据是如何被存储的呢？

数据其实仍然是被存储在各自线程中，**由各自线程去维护**，这样实现线程间数据独立的同时，也降低了维护数据的成本，大家管好自己的数据就可以了。而这个数据被存储在 `ThreadLocalMap` 中，**他是 `Thread` 的一个成员**，主要用来存储 **本线程** 的数据。

**每个线程单独维护自己的数据，TheadLocal 只是存取的一个中介，他不管理数据，理解这一点至关重要。**

`ThreadLocalMap` 的数据结构大致如下，看的出来他内部维护的是一个 `Entry` 数组，`Entry` 是 `WeakRef` 的子类，他把 `ThreadLocal` 和要存储的 `Value` 成对的存储在了一起。

- 这个数组的下标索引：是通过 `ThreadLocal` 哈希处理后获取到的，也就是说使用 `ThreadLocal` 可以再次定位到这个 `Entry`
- 这个数组的值：是 `Entry` 对象，他是 `ThreadLocal` 和 `Value` 的整合。


```java
class ThreadLocalMap {
	static class Entry extends WeakReference<ThreadLocal<?>> {
    		Object value;
	}
	private Entry[] table;
}
```

在每个线程中都单独维护一个 `ThreadLocalMap`，初始值为 `null`，当使用 `ThreadLocal` 存取数据时通过对 `ThreadLocal` 进行 `hash` 处理获得一个数组下标，`ThreadLocal` 和需要存储的数据对象被打包成一个 `Entry` 放入数组的指定位置。

也就是说 `ThreadLocalMap` 是一个 `Entry` 数组，`table[ThreadLocal hash] = Entry(ThreadLocal, Value)` 

```java
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    table = new Entry[INITIAL_CAPACITY];
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1);
    table[i] = new Entry(firstKey, firstValue);
    size = 1;
    setThreshold(INITIAL_CAPACITY);
}
```

## 线程独立的存取数据

> 借助 `ThreadLocal` 如何保证线程间数据是互相独立的？

我们通常会获得一个 `ThreadLocal` 对象，并使用它在不同线程做存取数据的操作。

```java
// 1.8 之后可以使用一个工厂函数返回初始值
ThreadLocal<Integer> integerThreadLocal = ThreadLocal.withInitial(() -> 100);
// 1.8 之前在子类重写放啊返回初始值
ThreadLocal stringThreadLocal = new ThreadLocal<String>() {
    @Override
    protected String initialValue() {
        return "hahha";
    }
};
```

当我们获取数据时，首先借助 `Thread.currentThread()` 拿到当前方法执行的线程，再从线程中获取本线程维护 `ThreadLocalMap` 对象，他是一个 `Entry(ThreadLocal.hash, Object)` 数组，那么此时我么可以将当前的 `ThreadLocal` 做 `hash` 处理后，定位到相应的数组下标，取出对应的 `Entry` 从而拿到里面的 `value`，如果取不到就返回初始值，简单看一下 `get()` 的代码会更清晰。

```java
public T get() {
    // 取到线程
    Thread t = Thread.currentThread();
    // 拿到线程里面的 ThreadLocalMap
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        // 用 ThreadLocal 做 hash 定位对应的 Entry
        // this 是 ThreadLocal
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            // 取到 Entry 里面的 value
            T result = (T)e.value;
            return result;
        }
    }
    // 返回初始值
    return setInitialValue();
}
```

当往 `ThreadLocal` 中存入数据时，逻辑也是一样的，唯一的不同时，如果线程中的 `ThreadLocalMap` 没有创建时，此时会创建一个新的 `ThreadLocalMap` 并将数据存储进去

```java
public void set(T value) {
    // 获取到线程
    Thread t = Thread.currentThread();
    // 获取线程维护的 ThreadLocalMap
    ThreadLocalMap map = getMap(t);
    // 对 ThreadLocal 做 hash 拿到数组下标，打包 TreadLocal 和 value 存储
    if (map != null)
    	// this 是 ThreadLocal
        map.set(this, value);
    else
        // 创建新的 ThreadLocalMap 并存储 
        createMap(t, value);
}
```

## 哈希散列

> 数据存入表中下索引计算的规则？

上面我们看到的都是 `ThreadLocal` 的方法，获取数据时，他调用了 `ThreadLocalMap`  的 `getEntry(ThreadLocal)` 方法，存入数据时调用了 `ThreadLocalMap` 的 `set(ThreadLocal, Object)` 方法，所以说真正完成数据存取的 `ThreadLocalMap` 类。

前面介绍了 `ThreadLocalMap` 本质上是一个数组 `Entry[]`，数组就会有一个容量大小，我们存取数据使用的是下标，这个下标必然需要在 `[0, len - 1]` 范围内，索引的获取使用的是哈希算法，每个 `ThreadLocal` 都有一个 `threadLocalHashCode` 他以一定的规则自增，而且必然不会重复，可以把它看作唯一的 `id`，当我们使用 `ThreadLocal` 获取数据时，首先对 `threadLocalHashCode` 在容量大小 `length` 上散列，他会生成一个 `[0, len - 1]` 范围内一个不重复索引，这个操作是可逆的，也就是说我每次用这个 `ThreadLocal` 一定会定位到这个索引，也就会定位到这个数据。

## LocalThreadMap#set

向 `ThreadLocal` 设置数据

```java
private void set(ThreadLocal<?> key, Object value) {
    Entry[] tab = table;
    int len = tab.length;
    // 散列得到数组索引
    int i = key.threadLocalHashCode & (len-1);
    // 从当前索引的位置向后查找
    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        ThreadLocal<?> k = e.get();
        // 如果匹配上了，直接覆盖原来的值
        if (k == key) {
            e.value = value;
            return;
        }
        // 发现当前数组中的 ThreadLocal 为空，说明因为一些原因被回收了
        // 则清理陈旧的数据
        if (k == null) {
            replaceStaleEntry(key, value, i);
            return;
        }
    }
    // 没有查找到，创建新的 Entry
    tab[i] = new Entry(key, value);
    int sz = ++size;
    // 扩容
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}
```








