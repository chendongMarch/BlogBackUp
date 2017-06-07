---
layout: post
title: Webview和原生应用交互
date: 2016-04-26
category: Android
tags: Android
keywords: Android
description: webview和原生应用JS交互
---


## 定义关联的对象
```java
public class Bridge2Js {
   private Activity activity;
	public Bridge2Js(Activity activity){
		this.activity = activity;
	}
	//4.2之后需要加注解否则报错
  @JavascriptInterface
   public void startPage(int pageSign, String json){
        activity....
   }
   
   @JavaInterface
   public void clickOnAndroid(){
   
   }
}
```
## 设置关联类
```java
WebSettings mWebSettings = mProgressWebView.getSettings();
//允许js交互
mProgressWebView.addJavascriptInterface(new Bridge2Js(this), "bridge");
```

## 在JS代码中调用Android代码
```html
//button
<input type="button"  value="click me"  onclick="window.bridge.startPage(0,'asd')"/>

//标准html
<!DOCTYPE html>   
<html>
    <script language="javascript">
        /在Android代码中可以调用下面js函数/
        function wave() {
            alert("1");
            document.getElementById("droid").src="android_waving.png";
            alert("2");
        }
    </script>
    <body>
        <!-- 在这里调用Andriod函数 -->
        <a onClick="window.bridge.clickOnAndroid()">Click me!</a>
    </body>
</html> 
```

## 在Android中调用JS代码
```java 
webview.loadUrl("javascript:wave()");
```


