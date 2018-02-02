---
layout: post
title: 从 Retrofit 看动态代理 [设计模式]
categories:
  - 设计模式
tags:
  - 设计模式
abbrlink: e73f8f32
keywords:
  - Retrofit
  - 代理模式
  - 动态代理
  - 静态代理
date: 2018-01-29 10:58:00
photos: http://olx4t2q6z.bkt.clouddn.com/18-2-1/44700615.jpg
location: 杭州尚妆
---


本文主要学习代理模式在 `Java` 下的实现，以及 **动态代理** 在 `Retrofit` 中的应用。

代理模式 ：给某一个对象提供一个代理，并由代理对象控制对原对象的引用，它可以在屏蔽对目标对象访问的同时，进行自定义的扩展。

<!--more-->
## 推荐阅读

[Graphic Design Patterns - 代理模式](http://design-patterns.readthedocs.io/zh_CN/latest/structural_patterns/proxy.html)

## 优缺

优点

- 代理模式能够协调调用者和被调用者，在一定程度上降低了系 统的耦合度。

- 远程代理使得客户端可以访问在远程机器上的对象，远程机器 可能具有更好的计算性能与处理速度，可以快速响应并处理客户端请求。

- 虚拟代理通过使用一个小对象来代表一个大对象，可以减少系 统资源的消耗，对系统进行优化并提高运行速度。

- 保护代理可以控制对真实对象的使用权限。

缺点

- 由于在客户端和真实主题之间增加了代理对象，因此 有些类型的代理模式可能会造成请求的处理速度变慢。
- 实现代理模式需要额外的工作，有些代理模式的实现 非常复杂。

## 静态代理

我们模拟一个场景：我们要邀请一个歌手来参加一场演唱会，那么我们是不能直接和歌手联系的，我们需要先联系歌手的经纪人，在这里经纪人扮演了代理的角色，他屏蔽了我们对歌手的访问，我们要联系歌手，必须通过经纪人代理实现。

首先定义接口实现

```java
// 歌手接口
interface SingingInterface {
    // 开始表演
    void performance();
}
```

同时根据接口实现一个真正的歌手类，比如 周董

```java
class JayChou implements SingingInterface {
    @Override
    public void performance() {
        log("`快快使用双截棍，哼哼哈嘿--`");
    }
}
```

定义经纪人类，它可以与歌手联系，并且屏蔽外界对歌手的访问，而且他还要做一些额外的工作，比如安排歌手档期及收取报酬等。

```java
// 经纪人
class StarBroker implements SingingInterface{
    private SingingInterface mSingingInterface;
    public StarBroker() {
        mSingingInterface = new JayChou();
    }
    @Override
    public void performance() {
        log("安排歌手档期。");
        mSingingInterface.performance();
        log("收取报酬。");
    }
}
```

最后我们可以预约歌手来参加我们的演唱会啦

```java
StarBroker starBroker = new StarBroker();
starBroker.performance();
```



## 动态代理

在静态代理中我们使用代理类来控制对目标对象的访问，实现起来相对麻烦，我们要定义代理类并且让它持有目标对象的引用，而且当需要代理的方法越来越多时，就需要在每个方法中再去调用目标对象的对应方法。

动态代理是一种在运行期间动态生成代理类的方法，他不需要定义代理类的类结构，而是借助 `InvocationHandler` 接口，对目标对象所有方法进行代理。

关于 `InvocationHandler` 接口，他只有一个方法：

- `Object proxy`  当前的代理类
- `Method method`  代理类被调用的方法
- `Object[] args`  调用方法时传入的参数

```java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable
```

我们动态生成代理类在执行方法时，会通过这个方法，然后我们就可以在这个方法里面去调用目标对象的对应方法，并做一些自己的扩展，达到代理目标对象访问的目的。


我们定义 `InvocationHandler` 的实现类，

```java
class SingingInvocationHandler implements InvocationHandler {
    // 目标对象
    private Object singer;
    public SingingInvocationHandler(Object singer) {
        this.singer = singer;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 下面实现了类似经纪人的功能
        log("安排歌手档期。");
        // 调用目标对象的对应方法
        Object invoke = method.invoke(singer, args);
        log("收取报酬。");
        return invoke;
    }
}
```

借助 `SingingInvocationHandler` 实现运行期的动态代理，这里说一下几个参数的意义

- `SingingInterface.class.getClassLoader()` => 是代理类生成需要的的 loader
- `new Class[]{SingingInterface.class}` => 代理类需要实现的接口列表，这里可以是多个
- `new SingingInvocationHandler(new JayChou())` => InvocationHandler + 目标对象

```java
SingingInterface proxy = (SingingInterface) Proxy.newProxyInstance(
        SingingInterface.class.getClassLoader(),
        new Class[]{SingingInterface.class},
        new SingingInvocationHandler(new JayChou()));
```
当我们使用生成的代理类对象 `proxy` 调用方法时，就会经过 `InvocationHandler` 里面的实现啦。

```java
// 执行 SingingInterface 的方法
proxy.performance();
```

```bash
安排歌手档期。
`快快使用双截棍，哼哼哈嘿--`
收取报酬。
```

为了体现动态代理和静态代理的相似之处，上面的写法相对简单，其实也是存在很多问题的，我们生成的代理对象 `proxy` 可以实现多个接口，执行的所有方法甚至是从 `Object` 中继承的方法都会经过 `InvocationHandler`，因此在进行代理是我们必须是选择性的代理部分方法。

声明 `TestInterface` 用来测试代理类实现多个接口的情况

```java
interface TestInterface {
    void testProxyMethod();
}
```

所以我们生成代理类时就变成了如下，此时的 `proxy` 既是 `SingingInterface` 的实现类也实现了 `TestInterface`

```java
SingingInterface proxy = (SingingInterface) Proxy.newProxyInstance(
        SingingInterface.class.getClassLoader(),
        new Class[]{SingingInterface.class, TestInterface.class},
        new SingingInvocationHandler(new JayChou()));
```

对 `InvocationHandler` 也进行一下完善，让他可以区分不同的方法的来源，然后选择性的进行代理

```java
class SingingInvocationHandler implements InvocationHandler {
    // 目标对象
    private Object singer;
    public SingingInvocationHandler(Object singer) {
        this.singer = singer;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 全部的方法都会被代理到这边，如果是 Object 的方法，不尽兴代理
        // 我们可以使用该方法区分，方法来自哪个父类或者接口，选择性的进行代理
        if (method.getDeclaringClass() == Object.class) {
            return method.invoke(this, args);
        }
        if (method.getDeclaringClass() == TestInterface.class) {
            return method.invoke(new TestInterface() {
                @Override
                public void testProxyMethod() {
                    log("执行了 TestInterface 的方法哦");
                }
            }, args);
        }
        // 下面实现了类似经纪人的功能
        if (method.getDeclaringClass() == SingingInterface.class) {
            log("安排歌手档期。");
            Object invoke = method.invoke(singer, args);
            log("收取报酬。");
            return invoke;
        }
        return null;
    }
}
```
我们分别执行 `Object` 、`SingingInterface` 和 `TestInterface` 的方法

```java
// 执行 Object 的方法
proxy.hashCode();
// 执行 SingingInterface 的方法
((SingingInterface) proxy).performance();
// 执行 TestInterface 的方法
((TestInterface) proxy).testProxyMethod();
```

```bash
执行了 Object 的方法哦
安排歌手档期。
`快快使用双截棍，哼哼哈嘿--`
收取报酬。
执行了 TestInterface 的方法哦
```

## Retrofit 与 动态代理

这里不是对 `Retrofit` 源码进行分析，那需要更大的篇幅，这里只是针对动态代理在 `Retrofit` 的应用方式进行学习。

作为现在网络请求框架中最受欢迎的 `Retrofit`，框架设计上最大的特点就是使用了动态代理将网络请求变成一个使用注解配置的过程。

在原来的模式中，我们需要借助 `OkHttp` 做创建请求，配置数据，发起请求，返回数据，解析数据等一系列操作，而借助动态代理，可以把这些操作都对使用者屏蔽掉，使用者只需要做两件事，那就是配置请求和处理解析好的数据，这样更大程度上的专注于业务逻辑。

整个过程是这样的：

- 我们声明包含一系列请求方法的接口类 `ApiService`，并在方法上使用注解对 `Path`、`Header`、`Query` 等请求必须的参数进行配置。
- 借助 `Retrofit` 的静态方法 `create()` 动态生成 `ApiService` 类型的代理对象返回给调用者。
- 当调用者调用代理类的方法发起请求时，就会进入 `InvocationHandler` 中
- 从注解中获取到全部的配置，初始化请求，发起请求，等待返回，解析数据
- 将解析好的数据返回，调用者就会拿到期望的数据


声明请求的 `ApiService`，它只是一个接口，普普通通的接口，每个方法上面的注解用来承载一些请求的配置

```java
public  interface ApiService{
   
    @GET("/page/list")
    @Headers("domain:baidu")
    Observable<List<String>> getList(@Query("type") int type);
}
```

使用 `Retrofit` 的静态方法创建动态代理对象 

```java
ApiService apiService = mRetrofit.create(ApiService.class);

apiService.getList(2);
```

看一下 `create()` 方法中的实现

```java
public <T> T create(final Class<T> service) {
  Utils.validateServiceInterface(service);
  if (validateEagerly) {
    eagerlyValidateMethods(service);
  }
  return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
      new InvocationHandler() {
        private final Platform platform = Platform.get();
        @Override public Object invoke(Object proxy, Method method, @Nullable Object[] args
            throws Throwable {
          // If the method is a method from Object then defer to normal invocation.
          if (method.getDeclaringClass() == Object.class) {
            return method.invoke(this, args);
          }
          if (platform.isDefaultMethod(method)) {
            return platform.invokeDefaultMethod(method, service, proxy, args);
          }
          // 创建请求、发起请求、解析数据
          ServiceMethod<Object, Object> serviceMethod =
              (ServiceMethod<Object, Object>) loadServiceMethod(method);
          OkHttpCall<Object> okHttpCall = new OkHttpCall<>(serviceMethod, args);
          return serviceMethod.callAdapter.adapt(okHttpCall);
        }
      });
}
```

我们不去关注解析注解配置和请求发起的具体实现，看起来和我们之前介绍的动态代理实现是一样的，所以 `Retrofit` 选择了一种非常合适的设计模式，让原本很复杂的操作变得很简单很优雅。


 