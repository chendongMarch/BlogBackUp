---
layout: post
title: Java 双重检查加锁创建单例
categories:
  - Java
tags:
  - 设计模式
keywords:
  - Singleton
  - 单例模式
  - 双重检查加锁
  - 设计模式
comments: true
abbrlink: 65563eb2
date: 2017-07-06 09:58:05
password:
---

本文介绍 `Java` 单例模式，主要包括：


> 饿汉式单例实现

> 懒汉式单例实现

> 线程安全加锁单例实现

> 双重检查加锁单例实现。

> 使用枚举实现单例

<!--more-->

## 饿汉式

顾名思义，就是很饿了，必须立刻创建。

饿汉式的特点就是空间换时间，在开始时就创建，而且只创建一次，节约了运行时间，保证了线程安全，缺点就是当你不需要用的时候也同样占用内存空间。

```java
public class TokenProvider {
    private static TokenProvider sInst = new TokenProvider();
}
```

## 懒汉式

体现了一种懒加载的思想，不需要时不创建，需要时才创建。

懒汉式的特点就是时间换空间，不需要时不创建，需要时才创建，节约了内存空间，缺点就是每次获取都需要判断，增加了运行时间，而且不加锁的懒汉式无法保证线程安全。

为什么呢？因为多线程访问时，假设 A 线程正在创建实体，此时 B 线程已经开始进行 `sInst == null` 的判空操作，此时 B 线程便会创建一个新的实体。

如何解决？请看下面线程安全加锁

```java
public class TokenProvider {
    
    private static TokenProvider sInst;
    
    public static TokenProvider getInst() {
        if (sInst == null) {
            sInst = new TokenProvider();
        }
        return sInst;
    }
}
```


## 线程安全加锁

当使用懒汉式时，如果多线程同时创建单例，不加锁的话就会创建多个实体，无法保证线程安全，此时应在判空操作之前加锁，让其他线程在外面等待已经进入的线程完成操作，创建完成实体，这时第二个线程进入时，`sInst` 已经不为空可以直接返回，从而保证线程安全。

缺点也很明显，需要使用该单例时，所有线程都会首先进入同步代码块，在线程同步时会浪费很多时间，我们需要避免这种情况，请看下节双重检查加锁


```java
public class TokenProvider {

    private static TokenProvider sInst;

    public static TokenProvider getInst() {
        synchronized (TokenProvider.class) {
            if (sInst == null) {
                sInst = new TokenProvider();
            }
        }
        return sInst;
    }
}
```

## 双重检查加锁

为了解决上面的问题，我们需要避免每次都进行同步加锁，在最外层先进行判空操作，当实体已经创建时，后面的线程则不需要进入同步代码块等待，节约了时间。

> volatile: 被 volatile 修饰的变量的值，将不会被本地线程缓存，所有对该变量的读写都是直接操作共享内存,从而确保多个线程能正确的处理该变量。


```java
public class TokenProvider {

    private volatile static TokenProvider sInst;

    public static TokenProvider getInst() {
        if (sInst == null) {
            synchronized (TokenProvider.class) {
                if (sInst == null) {
                    sInst = new TokenProvider();
                }
            }
        }
        return sInst;
    }
}
```

## 使用枚举实现单例

简单说一下枚举，枚举类似类，一个枚举可以拥有成员变量，成员方法，构造方法。创建`enum` 时，编译器会自动为我们生成一个继承自 `Java.lang.Enum` 的类，构建实例的过程不是我们做的，一个 `enum` 的构造方法限制是 `private` 的，也就是不允许我们调用。单例中的每一个都是 `static final` 类型，下面代码的对比应该更清楚一些

```java
enum Type{
    A,B,C,D;
}
```
等同于

```java
class Type extends Enum{
    public static final Type A;
    public static final Type B;
    public static final Type C;
    public static final Type D;
}
```

在枚举中我们明确了构造方法限制为私有，在我们访问枚举实例时会执行构造方法，同时每个枚举实例都是 `static final` 类型的，也就表明只能被实例化一次。在调用构造方法时，我们的单例被实例化。 
也就是说，因为 `enum` 中的实例被保证只会被实例化一次，所以我们的 `INSTANCE` 也被保证实例化一次。 


```java
public enum TokenProviderSingleton {
    
    INSTANCE;
    
    private TokenProvider mInst;
    
    TokenProviderSingleton() {
        mInst = new TokenProvider();
    }
    public TokenProvider getInst() {
        return mInst;
    }
}
```