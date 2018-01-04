---
layout: post
title: Kotlin开发-19-Anko
categories:
  - Kotlin
tags:
  - Kotlin
keywords:
  - Kotlin
  - Anko
abbrlink: 49416638
date: 2017-09-15 16:57:00
---
 
下面是我抄的 [Anko - GitHub](https://github.com/Kotlin/anko) 上面的描述

> Anko is a Kotlin library which makes Android application development faster and easier. It makes your code clean and easy to read, and lets you forget about rough edges of the Android SDK for Java.

Anko consists of several parts:

```bash
Anko Commons: a lightweight library full of helpers for intents, dialogs, logging and so on;

Anko Layouts: a fast and type-safe way to write dynamic Android layouts;

Anko SQLite: a query DSL and parser collection for Android SQLite;

Anko Coroutines: utilities based on the kotlinx.coroutines library.
```

<!--more-->


## Commons

```gralde
compile "org.jetbrains.anko:anko-commons:$anko_version"
```
`Commons` 库就类似于我们的快速开发框架，在一些基础 `API` 的基础上进一步封装，达到更加简化的目的。

### Intent

对 `Intent` 的扩展，大大简化构造 `Intent` 的过程

```kotlin
 // 打开 Activity
 startActivity(intentFor<HomeActivity>())
 
 intentFor<HomeActivity>(
         "id" to 5,
         "name" to "chendong")
         
 intentFor<HomeActivity>(
         "id" to 5,
         "name" to "chendong").singleTop()
         
 intentFor<HomeActivity>(
         "id" to 5,
         "name" to "chendong")
         .addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)
```
针对一些 `Intent` 的常用操作，也做了封装

```kotlin
// 打电话-需要申请权限
makeCall("13611301718")
// 发邮件
email("helloworld@gmail.com", "subject", "email-body")
// 发短信
sendSMS("13611301718", "msg-body")
// 分享文字
share("title", "msg-body")
// 开启浏览器
browse("http://www.baidu.com", true)
```

### UI Extension

显示 `Toast`

```kotlin
toast("toast msg")
toast(R.string.msg)
longToast("long toast msg")
```

显示 `AlertDialog`

```kotlin
alert(Appcompat, "message", "title") {
    okButton {
        it.dismiss()
    }
    cancelButton {
        it.dismiss()
    }
}.show()


alert {
    customView { editText() }
}.show()
```

显示选择列表的 `Dialog`

```kotlin
selector("selector", listOf("1", "2", "3")) { dialog, index ->
    dialog.dismiss()
    toast("click $index")
}
```

显示 `ProgressDialog`

```kotlin
progressDialog("message", "title") {
}

indeterminateProgressDialog("message", "title") {
}
```


### Log

创建 `log` 打印

```kotlin
// tag 为 TestCommonsActivity
val logger = AnkoLogger<TestCommonsActivity>()
// tag 为 CustomTag
val logger2 = AnkoLogger("CustomTag")
// tag 为 Test
val logger3 = AnkoLogger(Test::class.java)
```
输出日志，支持函数计算，这样可以在输出的再计算相关的值

```kotlin
logger.verbose { "lazy calculate verbose msg" }
logger.verbose("verbose msg")

logger.debug { "lazy calculate debug msg" }
logger.debug("debug msg")

logger.info { "lazy calculate info msg" }
logger.info("info msg")

logger.warn { "lazy calculate warn msg" }
logger.warn("warn msg")

logger.error { "lazy calculate error msg" }
logger.error("error msg")

logger.wtf("error", IllegalStateException("error"))
```

### Misc

将颜色转化为非透明色，`alpha` 部分会被转换为 `ff`

```kotlin
0x00ff0000.opaque
```

灰化，将给的颜色单一色值进行灰化，红绿蓝三部分会变成相同的色值，但是最好使用一个色值的两位，它使用了移位运算，不然结果会有点混乱

```kotlin
// 将会返回 0xff999999
0x99.gray.opaque 
// 将会返回 0xffffff99
0xff99.gray.opaque
```
尺寸转换

```kotlin
// sp 转 px
sp(10)
// dp 转 px
dip(10)
// px 转 dp
px2dip(10)
// px 转 sp
px2sp(10)
```
递归遍历所有 `child view`，这边需要结合一点 `Layouts` 框架的内容，用于最后对 `View` 进行设置

```kotlin
verticalLayout {
    button("log") {}
    button("color") {}
}.applyRecursively {
    when (it) {
        is Button -> it.text = "${it.text} add"
    }
}
```

简化版本的 `findViewById()`

```kotlin
textView(R.string.app_name){
    textResource = R.string.app_name
    backgroundResource = R.mipmap.ic_launcher
}
```

在 `Activity` 和 `Fragment` 中可以直接使用 `ctx` 和 `act` 来访问上下文。

```kotlin
// context 
ctx
// Activity
act
```

## Layouts

`Layouts` 框架最大的亮点就是 `DSL(Domain Specific Language)`，领域专用语言，它提供了一种新的构建 `UI` 的方式，为什么要使用 `DSL`，下文摘自 `GitHub`

```bash
By default, UI in Android is written using XML. That is inconvenient in the following ways:

It is not typesafe;
It is not null-safe;
It forces you to write almost the same code for every layout you make;
XML is parsed on the device wasting CPU time and battery;
Most of all, it allows no code reuse.
```

使用传统的 `xml` 构建方式，使用起来真的很简单，还有实时预览，但是同样有很多缺点，比如类型安全，空安全，不能进行声明数据访问数据，`xml` 的解析浪费了大量性能等，而如果使用代码编写 `UI`，简直痛苦死，但是使用 `DSL` 可以避免这些问题，而且还有插件支持实时预览，还有什么理由拒绝呢？

添加依赖，发现 `com.google.android` 无法下载，因此过滤掉，另外需要注意的是如果依赖了 `kotlin` 协程官方的扩展库，需要删除掉，否则版本不同可能会出问题。

```gradle
// 不需要重复依赖
compile 'org.jetbrains.kotlinx:kotlinx-coroutines-core:0.18'
```
使用 `Layouts` 需要以下依赖

```gradle
// Anko Layouts  // sdk15, sdk19, sdk21, sdk23 are also available
compile("org.jetbrains.anko:anko-sdk25:$anko_version") {
    exclude group: 'com.google.android', module: 'android'
}
compile("org.jetbrains.anko:anko-appcompat-v7:$anko_version") {
    exclude group: 'com.google.android', module: 'android'
}

// Coroutine listeners for Anko Layouts
compile("org.jetbrains.anko:anko-sdk25-coroutines:$anko_version") {
    exclude group: 'com.google.android', module: 'android'
}
compile("org.jetbrains.anko:anko-appcompat-v7-coroutines:$anko_version") {
    exclude group: 'com.google.android', module: 'android'
}
```

### Basics

不再需要 `setContentView()` 

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    
    var view1 = find<TextView>(R.id.mTestTv)
    var view2 = findOptional<TextView>(R.id.mTestTv)
    
    // 直接声明布局
    verticalLayout {
    }
}
```

使用 `DSL` 布局，用起来和 `xml` 相差不大，借助代码的缩进可以清晰的想象出布局的样式，支持 `Android` 所有的控件。

```kotlin
verticalLayout {
    // 垂直的 LinearLayout
    verticalLayout { 
        button {  }
        progressBar {  }
    }
    // 水平的 LinearLayout
    linearLayout {}
    // RelativeLayout
    relativeLayout {}
    // FrameLayout
    frameLayout {}
    // AbsoluteLayout
    absoluteLayout {}
}
```

### 控件属性和 LayoutParams

声明控件的属性，控制控件的绘制显示，在创建控件时会传入一个初始化函数，在该函数中可以进行控件属性的初始化，写过 `xml` 这些属性用起来其实没什么太大差别，学习成本不高，同样每个属性都会对应一个使用 `resource` 的方法，用来兼容使用 `xml` 声明的 `string`,`color` 等，来避免硬编码。

```kotlin
verticalLayout {
    backgroundColor = 0xff.gray.opaque
    textView("textview") {
        text = "i am tv"
        textColor = 0x00.gray.opaque
        textSize = 18f
        backgroundColor = Color.YELLOW
        gravity = Gravity.CENTER
    }
}
```

设置 `LayoutParams`，他用来表示该控件在父布局中绘制显示

```kotlin
textView("textview") {
    // ...
    // 文字在控件内居中
    gravity = Gravity.CENTER
}.lparams {
    // 整个控件在父布局中靠右
    gravity = Gravity.RIGHT
    width = matchParent
    height = dip(200)
    // margin
    margin = dip(10)
    horizontalMargin = dip(10)
    verticalMargin = dip(10)
    leftMargin = dip(10)
    // padding
    padding = dip(10)
    verticalPadding = dip(10)
    horizontalPadding = dip(10)
    leftPadding = dip(10)
}
```

### RelativeLayout

别的布局不用多说，控件之间的依赖相对简单，针对 `RelativeLayout` 要复杂一些，在 `Anko` 中可以类似 `xml` 中一样，使用相似的语法依赖其他控件的 `id`，也可以直接依赖控件引用，当然里面的原理还是根据 `id` 依赖。

关于 `id`，无法像 `xml` 中可以自动生成，所以要么使用 `val ID_VIEW = 100` 这样的形式声明，要么使用 `resources` 的方式，推荐后面一种，如下：

```kotlin
<resources>
    <item name="test_tv" type="id"/>
</resources>
```

> ps：就算是依赖控件的引用，也必须为控件声明 `id`，因为内部原理还是使用 `id` 依赖，没有 `id` 会异常。

使用 `RelativeLayout` 布局

```kotlin
relativeLayout {
    val tv = textView {
        id = R.id.test_tv
        textColor = 0x00.gray.opaque
        text = "i am textview in relative layout"
    }
    button {
        text = "i am button below textview"
    }.lparams {
        // 支持 id
        rightOf(R.id.test_tv)
        // 支持 控件
        rightOf(tv)
        
        rightOf(R.id.test_tv)
        leftOf(R.id.test_tv)
        above(R.id.test_tv)
        below(R.id.test_tv)
        
        centerInParent()
        centerVertically()
        centerHorizontally()
        
        alignParentTop()
        alignParentBottom()
        alignParentRight()
        alignParentLeft()
        
        sameLeft(R.id.test_tv)
        sameRight(R.id.test_tv)
        sameTop(R.id.test_tv)
        sameBottom(R.id.test_tv)
        
        alignStart(R.id.test_tv)
        alignEnd(R.id.test_tv)
        baselineOf(R.id.test_tv)
    }
}
```
### Theme

对每个控件都有一个 `themedXXXX` 的实现，支持使用  `R.style.xxx` 来初始化

```xml
<style name="MyTv">
    <item name="android:text">来自style的text</item>
    <item name="android:textColor">#000000</item>
</style>
```

使用 `theme` 创建

```kotlin
verticalLayout {
   themedTextView(R.style.MyTv)
}
```

### Listeners

为控件设置监听，所有监听默认支持协程，这意味着你可以在监听中直接执行异步任务，还有一个优势就是实现一个监听时不再要实现所有方法。

```kotlin
// 在监听之中直接执行耗时操作
button("test") {
    onClick {
        // 耗时
        val result = bg {
            Thread.sleep(1000)
            System.currentTimeMillis().toString()
        }
        textView.text = result.await()
    }
}

// 实现需要的监听
editText {
    textChangedListener {
        afterTextChanged {
        }
    }
}
```

### Extends

在 `DSL` 中支持扩展使用自定义控件

```kotlin
class MyView(context: Context?) : TextView(context)

inline fun ViewManager.myView(init: MyView.() -> Unit): MyView {
    return ankoView({ MyView(it) }, theme = 0, init = init)
}
```

这样在 `DSL` 中就可以使用

```kotlin
verticalLayout {
    myView {
        text = "自定义控件"
    }
}
```

### Include

包含 `layout.xml` 文件，使用 `include`，范型决定了在函数中你可以使用的类型

```kotlin
verticalLayout{
	include<TextView>(R.layout.main_activity){
    	text = "text"
	}
}
```

### Convert

转换 `xml` -> `DSL`

```bash
Code -> Covert to Anko Layouts DSL
```

### Component

使用 `AnkoComponent` 可以让我们将 `UI` 部分单独的独立出来，而不是挂在 `Activity` 生命周期方法中国，可以让代码更清晰

```kotlin
class SplashActivityUI : AnkoComponent<SplashActivity> {
    override fun createView(ui: AnkoContext<SplashActivity>): View = with(ui)
        frameLayout {
            backgroundResource = R.drawable.splash_bg_bitmap
            textView("Security Files") {
                gravity = Gravity.CENTER
                backgroundColor = white
                textColor = black
                verticalPadding = dip(15)
                textSize = sp(16f).toFloat()
            }.lparams(width = matchParent, height = wrapContent) {
                gravity = Gravity.CENTER
            }
        }
    }
}
```
然后在 `Activity` 的 `onCreate()` 方法中添加布局

```kotlin
SplashActivityUI().setContentView(this)
```
使用 `AnkoComponent` 不止可以将 `Activity` 中的布局分离出来，他其实是一种对布局代码进行解耦复用的方法，我们也可以用它来加载 `RecyclerView` 的 `item`


## Coroutines

`Coroutines` 框架是基于 `kotlinx-coroutine` 框架的扩展，在此基础上针对 `Android` 做了简化。

```gradle
compile "org.jetbrains.anko:anko-coroutines:$anko_version"
```

### asReference

这个主要用来避免如果你在一个不能取消的协程之中引用了上下文，由于协程可能被挂起，这一操作有可能导致内存泄漏，因此 `anko` 提供了 `asReference` 方法来获取当前上下文的弱引用，避免内存泄漏。

```kotlin
private val ref: Ref<TestAnkoCoroutineActivity> = asReference()

private val tv: TextView by lazy {
    find<TextView>(R.id.test_tv)
}

fun test() {
    launch(UI) {
        ref().tv.hide()
    }
}
```

###  bg

用来快速的创建一个需要耗时的异步任务，并返回结果

```kotlin
tv.setOnClickListener {
    launch(UI) {
        pb.show()
        val rst = bg {
            Thread.sleep(1000)
            val index = time + (1..1000).sum()
            index
        }
        tv.text = rst.await().toString()
        pb.hide()
    }
}
```

## SQLite

暂不研究