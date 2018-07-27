---
layout: post
title: Gradle 日常
category: Android
tags:
  - Android
keywords:
  - Android
  - gradle
abbrlink: 3c4d21f6
photos: 'http://olx4t2q6z.bkt.clouddn.com/18-7-25/39117870.jpg'
location: 杭州尚妆
date: 2018-07-25 16:12:00
---

本文主要记录学习到的一些 `gradle` 使用技巧，并不全面，但是是在开发中会用到的一些点，可以借助这些技巧少做很多额外的工作。

<!--more-->

## 代码分包

主要是为了应对在不同 `buildType` 和 `flavor` 下需要使用不同的代码包这种场景，比如我们在 `fortest` 时初始化调试工具，但是在 `release` 时不会初始化，同时我们不希望通过 `if(){}else{}` 判断来实现，因为这种方式会将所有 `buildType` 的代码揉在一起，不利于扩展和维护，此时希望最好能够将不同 `buildType` 的代码分离，不同的构建方式时，打包不同的代码，因此代码包应该这样组织：

代码除了原来的 `main` 包之外，还分为了 `fortest`，`debug`，`release` 三个包，当使用不同的 `buildType` 构建时，会打包各个文件夹下的代码，这里不仅限于 `java` 代码，也包括资源文件等，可以看作 `debug` 和 `main` 是同级的，两个包加在一起，打包成最后的 `class` 文件。

![](http://cdn1.showjoy.com/shop/images/20180725/53ZD1ADF83QTFPOIGZ5F1532508940652.png.652x488.png)

题外话：那么我们什么时候使用 **分包** 策略，什么时候使用 `if(BuildConfig.DEBUG)` 判断的方法呢，相比起啦，使用 `BuildConfig` 判断写起来更简单，不需要在每个包里面去创建类，分包更适合那种不同 `buildType` 依赖都不同的场景，比如，我在 `debug` 的时候依赖了调试的 `library` 而在 `release` 时没有这个依赖，那么如果用判断的方法的话，在构建 `release` 时，就会找不到对应的类，因此这种场景下，需要把代码分包，在 `debug` 包里面才去引用 `debugCompile` 的内容，而在 `release` 时是完全没有感知的。另外使用分包的方法有个很大的好处就是代码的完全分离，不会因为其他 `buildType` 的代码污染你的 `release` 构建产品。


## resValue

使用该属性可以根据 `buildType` 和 `flavor` 的类型来分别设置不同的 `res` 资源的 `value`，比如在不同的 `buildType` 使用不同的 `app_name`:


```gradle
buildTypes {
    release {
        signingConfig signingConfigs.card
        resValue "string", "app_name", "达卡"
    }
    debug {
        signingConfig signingConfigs.card
        resValue "string", "app_name", "达卡(debug)"
    }
    fortest {
        signingConfig signingConfigs.card
        resValue "string", "app_name", "达卡(fortest)"
    }
}
```

## buildConfigField

使用该属性可以根据 `buildType` 和 `flavor` 的类型来更改最后编译生成的 `BuildConfig` 类，比如我们有一个调试开关 `AssistantEntry` 需要动态改变，则

```gradle
buildTypes {
    release {
        signingConfig signingConfigs.card
        buildConfigField "boolean", "AssistantEntry", "false" // 是否开启测试入口
        buildConfigField "String", "TypeName", '"release"' // 构建类型
    }
    debug {
        signingConfig signingConfigs.card
        buildConfigField "boolean", "AssistantEntry", "true" // 是否开启测试入口
        buildConfigField "String", "TypeName", '"debug"' // 构建类型
    }
    fortest {
        signingConfig signingConfigs.card
        buildConfigField "boolean", "AssistantEntry", "true" // 是否开启测试入口
        buildConfigField "String", "TypeName", '"fortest"' // 构建类型
    }
}
```

编译后我们可以生成新的 `BuildConfig` 类，在原来基础上增加了新的字段 `AssistantEntry`

```java
public final class BuildConfig {
  public static final boolean DEBUG = Boolean.parseBoolean("true");
  public static final String APPLICATION_ID = "com.showjoy.card";
  public static final String BUILD_TYPE = "debug";
  public static final String FLAVOR = "";
  public static final int VERSION_CODE = 98;
  public static final String VERSION_NAME = "1.0.0";
  // Fields from build type: debug
  public static final boolean AssistantEntry = true;
}
```


## matchingFallbacks

当定义了一个新的 `buildType`，但是依赖的 `library` 中并没有该 `buildType` 会造成编译失败，`matchingFallbacks` 可以使新定义的 `buildType` 匹配到 `library` 中已经存在的 `buildType`。

比如我新定义了 `fortest`，我可以使 `fortest` 匹配 `library` 中的 `release` 的 `buildType`，这样都是用 `fortest` 类型构建时，使用的就是 `library` 中 `release` 类型的代码，他接受的是个数组，按照优先级匹配。

```gradle
buildTypes {
    release {
        signingConfig signingConfigs.card
    }
    debug {
        signingConfig signingConfigs.card
    }
    fortest {
        signingConfig signingConfigs.card
        matchingFallbacks = ['release', 'debug']
    }
}
```




