---
layout: post
title: Android 超轻量级路由方案 [开源]
category: Android
tags:
  - Android
  - 开源
abbrlink: 3272e02
date: 2020-05-1 00:00:00
keywords:
---

本文主要介绍 **Android 轻量级路由方案** 的实现过程；

什么是路由协议？路由协议就是约定一套路径解析的规则，然后大家都遵循这个规则来进行页面跳转，从而达到动态和解耦的目的。

路由的存在有什么意义？

- 多模块，组件化，插件化开发时，使用路由进行解耦，组件之间遵循约定好的路由协议进行跳转，不再需要互相依赖。

- 混合开发时，`Web` 端使用约定好的路由路径，可以方便跳转 `app` 内各个页面并进行参数传递。

<!--more-->

在 `GitHub` 上已经有很多相当成熟的路由框架，他们支持编译时注解路径，支持隐式跳转，支持跳转到 `Fragment`，支持自定义解析等等... 可以说非常强大了。。

那我为啥还要自己写？主要还是因为，我们要知其然(知道轮子怎么用)，知其所以然(知道轮子怎么造)。另外虽然我写的功能不如那些框架强大，但是也更轻量，完成路由的生成、解析和跳转，大约只有 300 行代码，用来完成简单的路由需求也是不错的选择。

我并没有给这个功能单独创建一个工程，因为他太轻量了，我把他集成在了我常用开发库中，本文相关的源代码可以在 **[GitHub-DevKitSample](https://github.com/chendongMarch/DevKitSample/tree/master/devKit/src/main/java/com/march/dev/router)** 中查看。

总的来说这个路由方案的核心的原理就是对路径进行解析，然后生成 `intent`，设置参数，进行页面跳转，同时又要支持使用参数，配置生成跳转的路由路径，感谢强大的 `Uri`，简化了我很多解析的过程。


## 路由协议

一个路由路径应该有 `Scheme`、`Authority`、`Path`、`QueryParam` 几部分组成，他们以下面的形式组装。

```
Scheme://Authority/Path?QueryParam
```

比如下面的 `https` 协议的路径，`Scheme` 就是 `https`，`Authority` 就是 `www.baidu.com`，`Path` 就是 `test/list`，`QueryParam` 以 `key=value` 的形式表现，如 `id=100`。

```
https://www.baidu.com/test/list?id=100&key=value
```

因此我们可以定制自己的协议，作为应用内跳转的解析方式，比如定义下面的协议：

```
chendong://test.march.com/page/1?id=100
```

## 定制协议

我们在 `Android` 中进行数据传递时需要知道参数的类型，而这些仅仅靠字符串的路径是无法表达的，因此我们需要对 `QueryParam` 的声明形式进行如下约定，在 `QueryParam` 的 `key` 中需要表达数据的类型。

int | long | boolean | float | double | String | Object
:--|:--|:--|:--|:--|:--|:--
i-key|l-key|b-key|f-key|d-key|s-key|o-key

那么我们就会有如下的一个路径，参数仍然是 `key-value` 的形式，只不过 `key` 的写法我们是有约定要求的。

```java
chendong://testJs.march.com/page/1?i-iKey=10&f-fKey=1.25&l-lKey=10000000000&d-dKey=1.23456789&b-bKey=true&s-sKey=test
```

当路径为 `/page/pageId` 这种路径时表示是一个页面跳转，其中 `pageId` 为页面的唯一标示，至于 `pageId` 和页面的唯一映射，需要提前配置起来，我们会存储以下两个映射 `map`。

```java
// 存储 pageId - Activity.class 的映射
private Map<Integer,Class>  pageMap  = new HashMap<>();
// 存储 Activity.class - pageId 的映射
private Map<Class, Integer> classMap = new HashMap<>();
```

## 解析 pageId
首先将路径转换为 `Uri`

```java
Uri uri = Uri.parse(url);
```

接下来我们需要进行 `pageId` 的解析，我们只有获取到 `pageId` 才能确认跳转的是哪个页面，使用正则来进行解析。

```java
// 解析 pageId
private static int parsePageId(Uri uri) {
    int pageId = -1;
    String path = uri.getPath();
    if (path.startsWith(PATH_PAGE)) {
        Pattern compile = Pattern.compile("/page/([0-9]*$)");
        Matcher matcher = compile.matcher(path);
        if (matcher.find()) {
            pageId = Integer.parseInt(matcher.group(1));
        }
    }
    return pageId;
}

// 根据映射拿到跳转界面的 class
Class pageCls = sRouterConfig.pageMap.get(parsePageId(uri));
```


## 解析传递的参数

声明以下几种数据类型的标示

```java
private static final String TYPE_I = "i";
private static final String TYPE_L = "l";
private static final String TYPE_B = "b";
private static final String TYPE_F = "f";
private static final String TYPE_D = "d";
private static final String TYPE_S = "s";
```

定义一个数据类型，除了存储 `key-value`，我们还需要将数据的类型分离出来存储

```java
private static class Param {
    String type; // 类型，i,l,b,f,d,s
    String key; 
    String value;
    Param(String type, String key, String value) {
        this.type = type;
        this.key = key;
        this.value = value;
    }
}
```
解析参数，使用 `Uri` 的几个 `api` 和简单的正则，将 `type`，`key`，`value` 三部分分离出来存储起来，so easy ~

```java
// 解析参数
private static List<Param> parseParams(Uri uri) {
    Set<String> keySets = uri.getQueryParameterNames();
    List<Param> params = new ArrayList<>();
    Param param;
    Pattern compile = Pattern.compile("(\\b[ilbfds])-(.*)");
    for (String queryKey : keySets) {
        String queryValue = uri.getQueryParameter(queryKey);
        Matcher matcher = compile.matcher(queryKey);
        if (matcher.find()) {
            String type = matcher.group(1);
            String key = matcher.group(2);
            if (!TextUtils.isEmpty(type) && !TextUtils.isEmpty(key)) {
                param = new Param(type, key, queryValue);
                params.add(param);
            }
        }
    }
    return params;
}
```

## 构建跳转的 intent

我们已经拿到了想要跳转的界面，直接创建跳转的 `intent`

```java
// find class
Class pageCls = sRouterConfig.pageMap.get(parsePageId(uri));
Intent intent = new Intent(context, pageCls);
```
向 `intent` 中添加传递的参数

```java
// parse params
List<Param> params = parseParams(uri);
for (Param param : params) {
    putExtra(intent, param);
}

// 向 intent 根据类型输入参数
private static void putExtra(Intent intent, Param param) {
    switch (param.type) {
        case TYPE_I:
            intent.putExtra(param.key, Integer.parseInt(param.value));
            break;
        case TYPE_L:
            intent.putExtra(param.key, Long.parseLong(param.value));
            break;
        case TYPE_B:
            intent.putExtra(param.key, Boolean.parseBoolean(param.value));
            break;
        case TYPE_F:
            intent.putExtra(param.key, Float.parseFloat(param.value));
            break;
        case TYPE_D:
            intent.putExtra(param.key, Double.parseDouble(param.value));
            break;
        case TYPE_S:
            intent.putExtra(param.key, param.value);
            break;
    }
}
```
进行页面跳转，兼容了一下 `startActivityForResult()` 的情况。

```java
// 启动 activity
private static void startActivity(Context context, int reqCode, Class pageCls, List<Param> params) 
    Intent intent = new Intent(context, pageCls);
    for (Param param : params) {
        putExtra(intent, param);
    }
    if (reqCode == -1) {
        context.startActivity(intent);
    } else if (context instanceof Activity) {
        ((Activity) context).startActivityForResult(intent, reqCode);
    } else {
        context.startActivity(intent);
    }
}
```

展示一下完整的流程

```java
public static boolean goFrom(Context context, String url, int reqCode) {
    try {
        Uri uri = Uri.parse(url);
        if (checkSchemeAuthority(uri))
            return false;
        // find class
        Class pageCls = sRouterConfig.pageMap.get(parsePageId(uri));
        if (pageCls == null) {
            return false;
        }
        // parse params
        List<Param> params = parseParams(uri);
        startActivity(context, reqCode, pageCls, params);
    } catch (Exception e) {
        e.printStackTrace();
        return false;
    }
    return true;
}
```


## 生成跳转的路由路径

我们支持使用路由路径跳转的同时，也要支持配置参数生成跳转的路由路径。同样借助 `Uri` 的相关 `api`，这个操作并不困难。

```java
// url 构建器
public static class RouterUrlBuilder {
    
    private final Uri.Builder uriBuilder;
    
    RouterUrlBuilder(String scheme, String authority, int pageId) {
        uriBuilder = new Uri.Builder()
                .scheme(scheme)
                .authority(authority)
                .path(PATH_PAGE + pageId);
    }
    
    public RouterUrlBuilder put(String key, int value) {
        uriBuilder.appendQueryParameter(TYPE_I + SEPARATOR + key, String.valueOf(value));
        return this;
    }
    
    public RouterUrlBuilder put(String key, long value) {
        uriBuilder.appendQueryParameter(TYPE_L + SEPARATOR + key, String.valueOf(value));
        return this;
    }
    
    public RouterUrlBuilder put(String key, boolean value) {
        uriBuilder.appendQueryParameter(TYPE_B + SEPARATOR + key, String.valueOf(value));
        return this;
    }
    
    public RouterUrlBuilder put(String key, float value) {
        uriBuilder.appendQueryParameter(TYPE_F + SEPARATOR + key, String.valueOf(value));
        return this;
    }
    
    public RouterUrlBuilder put(String key, double value) {
        uriBuilder.appendQueryParameter(TYPE_D + SEPARATOR + key, String.valueOf(value));
        return this;
    }
    
    public RouterUrlBuilder put(String key, String value) {
        uriBuilder.appendQueryParameter(TYPE_S + SEPARATOR + key, value);
        return this;
    }
    
    public String build() {
        return uriBuilder.build().toString();
    }
}
```

我们需要使用 `Router` 中配置的 `Scheme`、`Authority` 和页面映射。

```java
public static RouterUrlBuilder newRouterUrlBuilder(Class targetActivity) {
    if (sRouterConfig == null || !sRouterConfig.isSchemeAndAuthoritySet()) {
        throw new IllegalArgumentException("set scheme and authority first");
    }
    return new RouterUrlBuilder(sRouterConfig.scheme, sRouterConfig.authority,
            sRouterConfig.classMap.get(targetActivity));
}
```
然后我们可以这样配置传递的参数

```java
String url = Router.newRouterUrlBuilder(HomeActivity.class)
        .put("iKey", 10)
        .put("sKey1", "ss=ss")
        .put("sKey2", "ss=s&s")
        .put("sKey3", "ss&ss")
        .build();
```

## Usage

配置 `Scheme`、`Authority` 和页面映射

```java
new Router.RouterConfig("chendong", "testJs.march.com")
        .add(1, JBTestActivity.class)
        .add(2, HomeActivity.class)
        .apply();
```

根据路径跳转

```java
String url = "chendong://testJs.march.com/page/1?i-iKey=10&f-fKey=1.25&l-lKey=10000000000&d-dKey=1.23456789&b-bKey=true&s-sKey=test";
Router.goFrom(mActivity, url);
```

生成路由路径

```java
String jumpUrl = Router.newRouterUrlBuilder(HomeActivity.class)s
        .put("iKey", 10)
        .put("sKey1", "ss=ss")
        .put("sKey2", "ss=s&s")
        .put("sKey3", "ss&ss")
        .build();
```
我们可以在 `html` 页面中使用路由路径进行 `app` 内页面跳转

```html
<a href="chendong://testJs.march.com/page/2?i-id=10&s-name"> 打开首页  </a>
```
截断路径进行跳转

```java
mWebView.setWebViewClient(new WebViewClient() {
    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        if (!Router.goFrom(mContext, url)) {
            mWebView.loadUrl(url);
        }
        return true;
    }
});
```

## todo

现在只是简化的版本，支持基本的功能，需要完善和验证的地方还有很多。


----

我把他集成在了我常用开发库中，本文相关的源代码可以在 **[GitHub-DevKitSample](https://github.com/chendongMarch/DevKitSample/tree/master/devKit/src/main/java/com/march/dev/router)** 中查看。