---
layout: post
title: WebView
categories:
  - Android
  - View
tags: 
  - Android
  - View
keywords:
  - Android
  - WebView
abbrlink: 3518734228
date: 2017-06-27 00:00:00
---
 
本文记录 `Android` 中 `WebView` 控件的相关使用，不断完善中...

主要包括：

- 基本属性的配置
- `WebView` 缓存相关内容
- `Java` 与 `Js` 的交互
- 打开本地应用(支付宝等)
- 加载远程网页，本地网页，`assert` 目录下的网页
- 支持自定义下载
- 自定义 `WebChromeClient` 本地化弹窗、文件选择

更详细的扩展实现都可以在 [GitHub-WebKit](https://github.com/chendongMarch/DevKit/tree/85cb7c8c2426a40f929d1f152775f30acef6b678/devKit/src/main/java/com/march/dev/webkit) 中找到。
<!--more-->

## 基本配置汇总

汇总的记录一下 `WebView` 的配置方法，重要的属性后面会展开说明。

```java
@SuppressLint("SetJavaScriptEnabled")
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
    // 设置WebView是否可以由 JavaScript 自动打开窗口，默认为 false
    // 通常与 JavaScript 的 window.open() 配合使用。
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
    // 设置可访问文件
    settings.setAllowFileAccess(true);
    // 当WebView调用requestFocus时为WebView设置节点
    settings.setNeedInitialFocus(true);
    //支持自动加载图片
    settings.setLoadsImagesAutomatically(true);
    // 指定WebView的页面布局显示形式，调用该方法会引起页面重绘。
    // NORMAL,SINGLE_COLUMN 过时, NARROW_COLUMNS 过时 ,TEXT_AUTOSIZING
    settings.setLayoutAlgorithm(WebSettings.LayoutAlgorithm.SINGLE_COLUMN);
    // 从Lollipop(5.0)开始WebView默认不允许混合模式，https当中不能加载http资源，需要设置开启
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
        settings.setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);
    }
}
```

## WebView 缓存

清除缓存，`true` 表示清除磁盘缓存，这个方法是全局的，也就是说会清除掉整个应用所有的 网页缓存。

```java
mWebView.clearCache(true);
```
清除历史记录

```java
mWebView.clearHistory();
```

网上有介绍说，设置缓存的目录，清除缓存时删除掉缓存文件，但是使用下面的方法设置缓存路径后，发现文件并没有缓存到指定的目录，不知道是怎么回事？求指教

缓存模式|描述
:--|:--
LOAD\_DEFAULT|默认的缓存使用模式。在进行页面前进或后退的操作时，如果缓存可用并未过期就优先加载缓存，否则从网络上加载数据。这样可以减少页面的网络请求次数。
LOAD\_CACHE\_ELSE\_NETWORK|只要缓存可用就加载缓存，哪怕它们已经过期失效。如果缓存不可用就从网络上加载数据。
LOAD\_NO\_CACHE|不加载缓存，只从网络加载数据。
LOAD\_CACHE\_ONLY|不从网络加载数据，只从缓存加载数据。


```java
private static void initWebViewCache(WebView mWebView) {
    
    String cachePath = new File(Environment.getExternalStorageDirectory()
            , "webCache").getAbsolutePath();
    
    WebSettings settings = mWebView.getSettings();
    settings.setAppCacheEnabled(true);
    settings.setAppCachePath(cachePath);
    settings.setDatabaseEnabled(true);
    // 过时
    settings.setDatabasePath(cachePath);
    // 开启dom缓存
    settings.setDomStorageEnabled(true);
    // 加载模式
    settings.setCacheMode(WebSettings.LOAD_NO_CACHE);
    // 缓存最大值，过时
    settings.setAppCacheMaxSize(1000 * 1024);
}
```

## 加载 Url

加载网络路径，`headers` 不是必选的

```java
Map<String,String> headers = new HashMap<>();
headers.put("auth","110");
mWebView.loadUrl("https://www.baidu.com/",headers);
```

加载 `assert` 路径

```java
// WebViewUtil.java
public static String getAssertUrl(String fileUrl) {
    return "file:///android_asset/" + fileUrl;
}

mWebView.loadUrl(WebViewUtil.getAssertUrl("index.html"));
```
 
加载 `sd` 卡路径

```java
// WebViewUtil.java
public static String getSdUrl(String fileUrl) {
    return "file://" + Environment.getExternalStorageDirectory() + "/" + fileUrl;
}

mWebView.loadUrl(WebViewUtil.getSdUrl("index.html"));    
```

## Js 调用 Java

`Java` 与 `Js` 交互需要定义一个带有 `@JavascriptInterface` 注解方法的对象，如下：

```java
public class JsBridge {
 
    @JavascriptInterface
    public void toast(String msg) {
        ToastUtils.show(msg);
    }

    @JavascriptInterface
    public void log(String msg) {
        Log.e(TAG, msg);
    }
}
```

使用 `JsBridge` 连接 `Java` 和 `Js`，使用 `mWebView.addJavascriptInterface(obj, name);` 方法。

- obj：带有 `JavascriptInterface` 注解方法的对象实例。
- name：一个标志符，js 将会使用该标志来调用 java 层的方法。

```java
mWebView.getSettings().setJavaScriptEnabled(true);
mWebView.addJavascriptInterface(new JsBridge(), "android");
```

在添加调用接口时，我们添加了一个标记，如上面代码中添加的是 `android`，它相当于调用接口对象的一个别名，因此在 `js` 中调用 `java` 方法时，只需要使用 `window` 对象，如下:

```js
window.android.log('test js invoke java method');
```


## Java 调用 Js

使用 `java` 调用 `js` 方法相对简单，假如在 `js` 中应该声明如下 `function`，

```js
<script>
// 有参无返回值
    function funcParam(param){
        alert(param+"");
    }

    // 有参有返回值
    function newFunc(param1,param2,param3){
        alert((param1 +  param2) + " " + param3);
        return param3;
    }
</script>
```
在 `java` 层调用时只需  `loadUrl("Javascript:js方法名())` 即可，但是 Android 4.4 之后在调用 `js` 方法时可以获取返回值，写一个方法兼容一下

```java
public static final String JS_FUNC_PREFIX = "javascript:";

public void invokeJs(String jsFunc, final ValueCallback<String> callback) {
    if (!jsFunc.startsWith(JS_FUNC_PREFIX)) {
        jsFunc = JS_FUNC_PREFIX + jsFunc;
    }
    // api 19
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
        mWebView.evaluateJavascript(jsFunc, callback);
    } else {
        mWebView.loadUrl(jsFunc);
    }
}
```

在与 `js` 进行参数传递时，基本数据类型可以直接传递，字符串类型要使用引号包含，而且直接拼接字符串不太好，因此写一个简化构建 `js` 方法，自动按顺序进行参数的拼接。

```java
// 创建一个 js 方法
public static String generateJsFunc(String funcName, Object... params) {
    StringBuilder sb = new StringBuilder(funcName).append("(");
    for (int i = 0; i < params.length; i++) {
        if (params[i] != null) {
            if (params[i] instanceof String) {
                sb.append("'").append(params[i]).append("'");
            } else {
                sb.append(params[i]);
            }
            if (i != params.length - 1) {
                sb.append(",");
            }
        }
    }
    sb.append(")");
    return sb.toString();
}
```

简单调用

```java
// 调用有参有返回值的方法
String jsFunc = JsBridge.generateJsFunc("newFunc", "str", 100, 100);
mJsBridge.invokeJs(jsFunc, new ValueCallback<String>() {
    @Override
    public void onReceiveValue(String value) {
        Log.e(TAG, "value = " + value);
    }
});

// 调用有参无返回值方法
String funcParam = JsBridge.generateJsFunc("funcParam", 100);
mJsBridge.invokeJs(funcParam, null);
```


## 拦截 Url 打开应用(支付宝)

主要原理是由于某些应用会暴露一些页面出来供别的 app 唤起，因此我们只需要拦截这种 `Url`，然后使用 `intent` 打开即可，下面以支付宝为例说明：

前端使用支付宝进行支付时，需要打开手机支付宝客户端，断点看到支付宝会加载一个 `scheme` 为 `alipays` 的url，链接中配置了支付的相关信息，客户端要做的就是拦截该 `Url`，使用 `intent` 打开支付宝。

当 `html` 加载网页之前会先走 `shouldOverrideUrlLoading()` 方法，此时截断不使用 `webView` 加载，而是使用 `intent` 打开，此方法不仅适用支付宝，也适用打开其他应用。

```java
mWebView.setWebViewClient(new WebViewClient() {
    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        // 如果不使用 intent 覆盖加载这个链接，就走原来
        if (!shouldOverrideIntentUrl(getContext(), url)) {
            view.loadUrl(url, Api.makeHttpHeaders(getContext()));
        }
        return true;
    }
});

public static final String ALIPAY_SCHEME = "alipays";
public static boolean shouldOverrideIntentUrl(Context context, String url) {
    Uri uri = Uri.parse(url);
    if (uri != null
            && uri.getScheme() != null
            && uri.getScheme().equals(ALIPAY_SCHEME)) {
        try {
            Intent intent = Intent.parseUri(url, Intent.URI_INTENT_SCHEME);
            intent.addCategory(Intent.CATEGORY_BROWSABLE);
            intent.setComponent(null);
            intent.setSelector(null);
            context.startActivity(intent);
        } catch (Exception e) {
            e.printStackTrace();
            if (e instanceof ActivityNotFoundException) {
                ToastUtil.show("未检测到支付宝客户端");
            }
            return true;
        }
        return true;
    }
    return false;
}
```


## 支持下载链接

当网页中加载一个下载的链接，如 `http://xxx.apk` 这种链接，会走 `shouldOverrideUrlLoading()` 方法，但是，它是无法加载一个网页的，结果就是没有任何反应，此时需要设置 `DownloadListener`，需要下载的链接会进入监听，你可以在监听中自己进行网络下载保存到文件，下面我使用直接打开浏览器的方式，更简单一些。

```java
public static void setDefDownloadListener(final WebView webView) {
    webView.setDownloadListener(new DownloadListener() {
        @Override
        public void onDownloadStart(String url, String userAgent,
                                    String contentDisposition,
                                    String mimetype,
                                    long contentLength) {
            //去下载
            Intent intent = new Intent(Intent.ACTION_VIEW);
            String downLoadUrl = url;
            if (!downLoadUrl.contains("http://")) {
                downLoadUrl = "http://" + downLoadUrl;
            }
            intent.setData(Uri.parse(downLoadUrl));
            intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            webView.getContext().startActivity(intent);
        }
    });
}
```


## WebChromeClient


### 本地化弹窗

`WebView` 默认是无法弹出 `alert()` 的，需要设置 `WebChromeClient`

```java
mWebView.setWebChromeClient(new WebChromeClient());
```

从网页上弹起的弹窗与设备整体风格不是很搭，需要重载 `onJsAlert()`、`onJsConfirm()` 和 `onJsPrompt()` 来将弹窗本地化。`JavaScript` 与本地方法对应如下：

```javascript
// 提示 返回值为 undefind - onJsAlert()
var result = alert("test");
// 确认和取消 返回值为 true | false
var result = confirm("test"); - onJsConfirm()
// 用户输入 返回值为用户输入的内容 - onJsPrompt()
var result = prompt("message","default");
```
以 `onJsConfirm()` 为例实现响应用户输入的方法，使用 `JsResult` 向 `JavaScript` 返回值，否则网页端会一直阻塞，不能操作。

```java
@Override
public boolean onJsConfirm(WebView view, String url, String message, final JsResult result) {
    new AlertDialog.Builder(view.getContext())
            .setCancelable(false)
            .setMessage(message)
            .setPositiveButton("确定", new DialogInterface.OnClickListener() {
                @Override
                public void onClick(DialogInterface dialog, int which) {
                    result.confirm();
                    dialog.dismiss();
                }
            })
            .setNegativeButton("取消", new DialogInterface.OnClickListener() {
                @Override
                public void onClick(DialogInterface dialog, int which) {
                    result.cancel();
                    dialog.dismiss();
                }
            })
            .create()
            .show();
    return true;
}
```

### 文件选择

需要重载 `onShowFileChooser()` 方法，使用 `Intent` 打开文件选择，然后返回文件，在 `api21` 以上提供了一些特有的方法，在这个版本以下要自己适配。

```java
@Override
public boolean onShowFileChooser(WebView webView,
        ValueCallback<Uri[]> filePathCallback,
        FileChooserParams fileChooserParams) {
    if (mActivity == null)
        return false;
    /*
    How to use:
    1. Build an intent using {@link #createIntent}
    2. Fire the intent using {@link android.app.Activity#startActivityForResult}.
    3. Check for ActivityNotFoundException and take a user friendly action if thrown.
    4. Listen the result using {@link android.app.Activity#onActivityResult}
    5. Parse the result using {@link #parseResult} only if media capture was not requested.
    6. Send the result using mFilePathCallback of {@link WebChromeClient#onShowFileChooser}
    * */
    Intent intent;
    if (AppUtils.isOver(Build.VERSION_CODES.LOLLIPOP)) {
        intent = fileChooserParams.createIntent();
    } else {
        intent = new Intent(Intent.ACTION_GET_CONTENT);
        intent.addCategory(Intent.CATEGORY_OPENABLE);
        intent.setType("*/*");
    }
    mActivity.startActivityForResult(intent, MyWebView.WEB_REQ_CODE);
    mMyWebView.mFilePathCallback = filePathCallback;
    return true;
}
```

在 `onActivityResult()` 中接受返回的数据.

```java
public void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode != WEB_REQ_CODE)
        return;
    Uri[] uris;
    if (AppUtils.isOver(Build.VERSION_CODES.LOLLIPOP)) {
        uris = WebChromeClient.FileChooserParams.parseResult(resultCode, data);
    } else {
        uris = new Uri[]{data.getData()};
    }
    mFilePathCallback.onReceiveValue(uris);
    mFilePathCallback = null;
}
```

## todo

更多内容补充中 ...