---
layout: post
title: Android WebView
categories:
  - Android
  - View
tags: Android
keywords:
  - Android
  - WebView
abbrlink: 3518734228
date: 2017-06-27 00:00:00
---
 
本文主要记录 `Android` 中 `WebView` 控件的相关使用，不断完善中...

<!--more-->
## 基本配置
汇总一下 `WebView` 的配置方法，当然不是全部都需要配置的，只是一个汇总和记录，具体比较重要的配置，有需要的话后面会展开介绍。

```java
private static void initWebViewCache(WebView mWebView) {
    String cachePath = new File(Environment.getExternalStorageDirectory(), "appCache").getAbsolutePath();
    WebSettings settings = mWebView.getSettings();
    settings.setAppCacheEnabled(true);
    settings.setAppCachePath(cachePath);
    settings.setDatabaseEnabled(true);
    // 过时
    // settings.setDatabasePath(cachePath);
    settings.setDomStorageEnabled(true);// 开启dom缓存
    // LOAD_DEFAULT：默认的缓存使用模式。在进行页面前进或后退的操作时，如果缓存可用并未过期就优先加载缓存，否则从网络上加载数据。这样可以减少页面的网络请求次数。
    // LOAD_CACHE_ELSE_NETWORK：只要缓存可用就加载缓存，哪怕它们已经过期失效。如果缓存不可用就从网络上加载数据。
    // LOAD_NO_CACHE：不加载缓存，只从网络加载数据。
    // LOAD_CACHE_ONLY：不从网络加载数据，只从缓存加载数据。
    settings.setCacheMode(WebSettings.LOAD_NO_CACHE);
    // 缓存最大值，过时
    settings.setAppCacheMaxSize(1000 * 1024);
}

public static void initWebViewSettings(WebView mWebView) {
    //支持获取手势焦点
    mWebView.requestFocusFromTouch();
    // 触觉反馈，暂时没发现用处在哪里
    mWebView.setHapticFeedbackEnabled(false);
    WebSettings settings = mWebView.getSettings();
    // 支持插件
    settings.setPluginState(WebSettings.PluginState.ON);
    // 允许js交互
    settings.setJavaScriptEnabled(true);
    // 设置WebView是否可以由JavaScript自动打开窗口，默认为false，通常与JavaScript的window.open()配合使用。
    settings.setJavaScriptCanOpenWindowsAutomatically(true);
    // 允许中文编码
    settings.setDefaultTextEncodingName("UTF-8");
    // 使用大视图，设置适应屏幕
    settings.setUseWideViewPort(true);
    settings.setLoadWithOverviewMode(true);
    // 支持多窗口
    settings.setSupportMultipleWindows(true);
    // 隐藏自带缩放按钮
    settings.setBuiltInZoomControls(false);
    // 支持缩放
    settings.setSupportZoom(true);
    //设置可访问文件
    settings.setAllowFileAccess(true);
    //当WebView调用requestFocus时为WebView设置节点
    settings.setNeedInitialFocus(true);
    //支持自动加载图片
    if (Build.VERSION.SDK_INT >= 19) {
        settings.setLoadsImagesAutomatically(true);
    } else {
        settings.setLoadsImagesAutomatically(false);
    }
    // 指定WebView的页面布局显示形式，调用该方法会引起页面重绘。
    // NORMAL,SINGLE_COLUMN 过时, NARROW_COLUMNS 过时 ,TEXT_AUTOSIZING
    settings.setLayoutAlgorithm(WebSettings.LayoutAlgorithm.SINGLE_COLUMN);
    // 从Lollipop(5.0)开始WebView默认不允许混合模式，https当中不能加载http资源，需要设置开启
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
        settings.setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);
    }
}
```


## Java 与 JS 交互
> 案例，做一个支付的交互，java 加载 html 网页，调用 js 进行支付，支付成功后，html 返回支付结果。


首先需要定义一个 `JavaScriptInterface`，它是 `js` 调用 `Java` 代码的桥梁，定义 `PayJsBridge` 类，使用 `@JavascriptInterface` 注解声明 `js` 调用的方法。

```java
private class PayJsBridge {
    @JavascriptInterface
    public void payResult(boolean isSuccess) {
        // 支付结果
        if (mOnPayListener != null) {
            mOnPayListener.onPayResult(isSuccess);
        }
    }
}
```

将 `PayJsBridge` 添加到 `WebView` 中，使用 `mWebView.addJavascriptInterface(Object obj,String name)` 方法，两个参数分别为

- obj：带有 `JavascriptInterface` 注解方法的对象实例。
- name：一个标志符，js 将会使用该标志来调用 java 层的方法。

```java
PayJsBridge  mPayJsBridge = new PayJsBridge();
mWebView.addJavascriptInterface(mPayJsBridge, "native");
```

### Java 调用 Js
使用 `java` 调用 `js` 方法相对简单。

在 `js` 中应该声明如下 `function`，

```js
<script>
function startPay(orderId,channelId){
  document.getElementById("display").innerHTML= 'orderId = '+orderId +',channelId = '+ channelId;
  window.native.payResult(true);
}
</script>
```
在 `java` 层调用时只需  `loadUrl("Javascript:js方法名())` 即可。

```java
mWebView.loadUrl("javascript:startPay(100,1)");
```

### Js 调用 Java
在添加调用接口时，我们添加了一个标记，如上面代码中添加的是 `native`，它相当于调用接口对象的一个别名，因此在 `js` 中调用 `java` 方法时，只需要使用 `window` 对象，如下

```js
window.native.payResult(true);
```

### 附测试 html 和 java 源码

index.html

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">

<script>
function startPay(orderId,channelId){
  document.getElementById("display").innerHTML= 'orderId = '+orderId +',channelId = '+ channelId;
  window.native.payResult(true);
}
</script>
</head>
<body>
 
<p id="display">将要显示</p>
<button type="button" onclick="startPay(1,3)">支付</button>

</body>
</html>
```

test.java

```java
// 交互接口
private class PayJsBridge {
    @JavascriptInterface
    public void payResult(boolean isSuccess) {
        if (mOnPayListener != null) {
            mOnPayListener.onPayResult(isSuccess);
        }
    }
}

// 添加调用接口
mWebView.getSettings().setJavaScriptEnabled(true);
PayJsBridge mPayJsBridge = new PayJsBridge();
mWebView.addJavascriptInterface(mPayJsBridge, "native");

// 调用 js
public void actionPay(long orderId) {
    StringBuilder jsCmd = new StringBuilder("javascript:startPay(")
            .append(orderId)
            .append(",")
            .append(mCurrentPaymentId)
            .append(")");
    mWebView.loadUrl(jsCmd.toString());
    ToastUtil.show("模拟支付  jsCmd = " + jsCmd);
}
```


## 拦截 URL 打开应用(支付宝)

前端使用支付宝进行支付时，需要打开手机支付宝客户端，断点看到支付宝会加载一个 `alipays://` 开头的url，链接中配置了支付的相关信息，客户端要做的就是拦截该Url，使用 `intent` 打开支付宝。

当 `html` 加载网页之前会先走 `shouldOverrideUrlLoading()` 方法，此时截断不使用 `webView` 加载，而是使用 `intent` 打开。此方法不仅适用支付宝，也适用打开其他应用，被打开的应用需要暴露一个 `schema`...

```java
mWebView.setWebViewClient(new WebViewClient() {
    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        // 如果不使用intent覆盖加载这个链接，就走原来
        if (!shouldOverrideIntentUrl(getContext(), url)) {
            view.loadUrl(url, Api.makeHttpHeaders(getContext()));
        }
        return true;
    }
});

// 是不是使用 intent 打开
private boolean shouldOverrideIntentUrl(Context context, String url) {
    if (url.startsWith("alipays://") || url.startsWith("intent://")) {
        try {
            Intent intent = Intent.parseUri(url, Intent.URI_INTENT_SCHEME);
            intent.addCategory("android.intent.category.BROWSABLE");
            intent.setComponent(null);
            intent.setSelector(null);
            context.startActivity(intent);
            return true;
        } catch (URISyntaxException e) {
            e.printStackTrace();
            return false;
        }
    }
    return false;
}
```


## 支持下载链接
当网页中加载一个下载的链接，如 `http://xxx.apk` 这种链接，会走 `shouldOverrideUrlLoading()` 方法，但是，它是无法加载一个网页的，此时需要设置 `DownloadListener`，需要下载的链接会进入监听，你可以在监听中自己进行网络下载保存到文件，下面我使用直接打开浏览器的方式，更简单一些。

```java
mWebView1.setDownloadListener(new DownloadListener() {
    @Override
    public void onDownloadStart(String url, String userAgent, String contentDisposition, String mimetype, long contentLength) {
        // 去浏览器打开
        Intent intent = new Intent(Intent.ACTION_VIEW);
        String downLoadUrl = url;
        if (!downLoadUrl.contains("http://")) {
            downLoadUrl = "http://" + downLoadUrl;
        }
        intent.setData(Uri.parse(downLoadUrl));
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        BaoBaoApplication.getInstance().startActivity(intent);
    }
});
```


## WebChromeClient

`WebView` 默认是无法弹出 `alert()` 的，需要设置 `WebChromeClient `

```java
mWebView.setWebChromeClient(new WebChromeClient());
```