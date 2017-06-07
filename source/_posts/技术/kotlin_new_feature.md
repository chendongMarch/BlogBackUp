---
layout: post
title: Kotlin-新特性篇
date: 2017-05-23
category: 技术
tags: kotlin
keywords: 
---

Google 2017 IO大会宣布Kotlin作为Android一级开发语言，一时间各大技术博客网站，微信订阅号，朋友圈都被kotlin刷屏了，全都是对kotlin的简要介绍，现在过了大约一周感觉大家慢慢平静下来了，发布的文章的质量也更高，更有针对性，相信不久的将来再广大开发者的推动下，kotlin会发展的越来越好。

就像Google发布Android专用的开发工具AndroidStudio一样，仅仅一年多的时间，Eclipse和Idea在Android开发领域就被完全取代了，Google的决心是看得见的。kotlin能够与Java无缝兼容，性能更好，语法更优雅，又有类型推断，函数式编程等新特性支持，相信在Android开发领域，取代Java只是时间问题，毕竟是官网支持的。

本文是对Kotlin的第一次接触，重点放在项目搭建和新特性的体验上，更深的细节将会放在后面的文章中研究，go ~


<!--more-->

## 推荐文章

`Kotlin` 官网和对应中文版，这个中文版文档的风格和官网风格一样，对比看起来舒服一点，推荐。

> [kotlin官网文档 ／ http://kotlinlang.org ](http://kotlinlang.org/docs/reference/classes.html)

> [Kotlin官方文档中文翻译版本 ／ http://www.liying-cn.net/kotlin/](http://www.liying-cn.net/kotlin/docs/reference/) 

`Kotlin` 的两篇推荐文章，很早就在推荐使用了

> [微信文章-你为什么需要kotlin](https://mp.weixin.qq.com/s?__biz=MzA3NTYzODYzMg==&mid=2653578489&idx=1&sn=17d6e657d81c0daa345489271d305b36)

> [Segmentfault上腾讯bugly的一篇文章，介绍了kotlin的基本特性](https://segmentfault.com/a/1190000004494727)

另外备注几份参考文档，各有优势，作补充用

> [kotlin官网文档中文版 ／ https://hltj.gitbooks.io](https://hltj.gitbooks.io/kotlin-reference-chinese/)

> [< kotlin for andriod developer> 中文翻译版 ／ https://wangjiegulu.gitbooks.io](https://wangjiegulu.gitbooks.io/kotlin-for-android-developers-zh/content/)

> [https://huanglizhuo.gitbooks.io](https://huanglizhuo.gitbooks.io/kotlin-in-chinese/content/)




## 基本配置
AS 3.0 已经自带了Kotlin的配置，本文在 AS 2.3 基础上进行配置。

打开Setting -> plugin下载插件，重启AS，创建项目 -> new File -> Kotlin Activity，会提示配置相关信息，点击配置即可，插件会在工程中作下面的相关配置，以上可以使用IDE很快完成，整体项目没有很大变化。   

` project/build.gradle `文件

```gradle
buildscript {
    ext.kotlin_version = '1.0.6'
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}

```

`app/build.gradle` 文件

```gradle

apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
// 基于Android的扩展增强
apply plugin: 'kotlin-android-extensions'

android {
    compileSdkVersion 25
    buildToolsVersion "25.0.1"
    defaultConfig {
       ...
    }
    buildTypes {
       ...
    }
    // 资源路径
    sourceSets {
        main.java.srcDirs += 'src/main/kotlin'
    }
}

dependencies {
    // kotlin
    compile "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
}
// 需要加入这个
repositories {
    mavenCentral()
}
```


## 汇总一些细节
1. `Kotlin` 不强制在每句代码后面加 `;`。

2. `Kotlin` 不支持三目运算符了，可以使用 `if(condition) 表达式1 else 表达式2` 的方式代替，比如 `val a = if(b>0) 100 else 0`

3. 不再使用 `new` 关键字创建对象，而是更简单的 `val user = User()` 

4. 与 `Java` 相反，`Kotlin` 中所有类和方法都默认是不允许继承的，也就是 `final` 的，如果想要继承类或重写方法可以使用 `open` 关键字。

5. `Kotlin` 中类名不必与文件同名，函数也不是必须声明在类中才可以，可以直接声明在文件中。

6. `Kotlin` 中注释允许嵌套。

7. 使用 `is` 代替 `instanceof` ，如 `if(str is String){...}`

## 引用xml中的控件
再也不用 `findViewById()` 也不用 `ButterKnife` 注解啦.

这种扩展在性能上和 `findViewById()` 是一样的，另外需要注意的是，使用这种扩展必须要在 `layout` 被设置好了才能使用, 在 `fragment` 里面, 要等到 `onViewCreated()` 之后才能使用.

```xml
<!--在xml定义，id为 mTestTv -->
<TextView
    android:id="@+id/mTestTv"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#fff"
    android:gravity="center"
    android:text="test"
    android:textColor="#000"
    android:textSize="18sp"/>

// app/build.gradle中扩展增强插件
apply plugin: 'kotlin-android-extensions'  
      
import kotlinx.android.synthetic.main.main_activity.*
mTestTv.text = "test it"
```

## 空值安全(Null Safety)
`Kotlin` 是一种空值安全的语言，他基本能帮助你告别 `NPE`，并且提供了大量的操作符简化判空操作，更多内容将在下一篇文章<常量变量篇>中详细介绍。

一个使用 `safe calls` 简化调用的实例

```kotlin
val myParam1:Student? = null

// java 中，在使用之前，我们通常会做一系列判空操作
if(myParam1!=null && myParam1.myCls!=null){
    Log.e(TAG,"${myParam1.myCls.length}")
}

// Kotlin safe calls
Log.e(TAG,"${myParam1?.myCls?.length}")
```


## 字符串模版
再也不用 `+` 或者 `StringBuilder` 来回拼接和计算字符串，kotlin提供了更加优雅的实现方式。

使用 `${表达式}` 的方式可以在字符串中间插入数据。只有一个简单值时可以省略 `{}` ,同时也支持复杂表达式的计算。

```kotlin
val user = User()
val param = 100
var str = "test str template like $param"
str = "test str template like ${user.name} "
str = "test str template like ${if (user.age <= 0) 100 else user.age}"
```

    
## 扩展函数和属性

`Kotlin` 提供了向一个类扩展新功能的能力，而且不必从这个类继承, 也不必使用任何设计模式, 比如 `Decorator` 模式之类。 这种功能是通过一种特殊的声明来实现的, `Kotlin` 中称为 `扩展(extension)`。 `Kotlin` 支持 `扩展函数(extension function)` 和 `扩展属性(extension property)`。

另外我刚开始看这个地方的时候，是将扩展函数声明在使用它的类中，在另一个类中要使用就无法引用了，难道每次都要重新写？那不是很鸡肋。其实由于 `java` 的惯性，我觉得所有的方法必须声明在类中，`java` 中确实是这样的， 但是 `Kotlin` 不必这样做，你的类文件的名字可以不和文件名相同，函数也可以直接声明在文件中，而不是某个类中。

> 注意：不需要声明 `class`，扩展方法和属性是静态加载的， 直接写在 `.kt` 文件中即可。

> 注意：在扩展属性的 `get()` 方法中不要访问该属性，因为你一旦访问该属性，就会调用 `get()` 方法，从而造成堆栈溢出。

新建 `ContextExtensions.kt` 文件，在里面声明该扩展函数和扩展属性

```kotlin
// 扩展函数
fun Context.logError(msg: String) {
    Log.e(this.javaClass.simpleName, msg)
}

// 扩展属性
var Context.extensionProperty: String
    get() {
        return javaClass.simpleName
    }
    set(value) {
        extensionProperty = value
    }
```

接下来就可以在任何 `Context` 的实现类，比如 `Activity` 中使用 `logError()` 函数和 `extensionProperty` 属性

```kotlin
// 扩展函数
logError("方法扩展")
// 扩展属性
logError(extensionProperty)
```


## 自动类型推断
Kotlin支持自动的类型推断，比如我们可以如下声明一个常量，那么常量 `myStr` 的类型被推断为 `String` 类型。可以使用 `String` 的API。

```kotlin
val myStr = ""
Log.e(tag,"str length = ${myStr.length}")
```
当进行显示类型转换时，自动进行类型推断，下面声明了两个类，`Student` 继承自 `User` ,在 `stuUser is Student` 的判断之后，`stuUser` 的类型被推断为 `Student` ，因此可以直接访问 `myClass` 属性。

```kotlin
open class User(val name: String, val age: Int) {
    // 二级构造函数
    constructor() : this("", 0)
}
class Student(name: String, age: Int,val myCls: String) : User(name, age)

// 自动类型推断
val stuUser = User()
if (stuUser is Student) {
	// 这里stuUser已经是Student类型，不需要进行强制转换。
    Log.e(tag, stuUser.myCls)
}
```

## 支持 lambda 表达式
使用 lambda 表达式可以使代码更加清晰，原来一层一层的 `{}` 都被省略掉了，不可否认的是可读性要比原来差一点，不过习惯了就好多了。

另外 Kotlin 也为 lambda 表达式提供了很多扩展支持，来强化 lambda 表达式的使用，在kotlin中,如果函数的最后一个参数是函数，那么这个参数可以直接写在圆括号外面（要用花括号），如果只有一个函数参数，可以直接省略圆括号！而且当参数只有一个时可以省略参数而使用 `it` 来代替。

```kotlin
mTestTv.setOnClickListener {
    v ->
    run {
        logError("test" + v.id)
    }
}

mTestTv.setOnClickListener {
    v ->
    if (v.id == 0) {
        logError("test${v.id})")
    } else {
        logError( "test${v.id})")
    }
}

mTestTv.setOnClickListener {
    logError("test${it.id})")
}
```

## 函数是一等公民
函数是一等公民指的是函数和对象，基本数据类型一样，可以作为变量，可以作为参数传递，可以作为返回值。 函数不依赖于类，可以独立存在。

Kotlin支持函数式编程。函数声明的方式为 `(参数名:参数类型...)-> 返回值类型` ，比如 `(param:String)->Int` ，指的是一个参数为 `String` 类型，返回值是 `Int` 类型的函数，如果没有返回值则使用 `Unit` 关键字。

刚开始尝试像使用变量一样使用函数，有些不太适应，当你需要将函数进行传递赋值时，先写一个 `{}`，将函数声明在 `{}` 中即可；

尝试简单声明函数

```kotlin
fun myFun(param: String): Int {
    logError("函数")
    return param.hashCode()
}

fun myFunReturnVoid(param: String): Unit {
    logError("声明函数返回空")
}
```
函数作为变量

```kotlin
// 函数作为变量
val funVal: (param: String) -> Int = {
    // 参数param
    param: String ->
    // 返回hashcode
    param.hashCode()
}
funVal("测试函数作为变量调用")
```
函数作为参数，下面的函数接受一个参数为 `String` 返回值为 `Int` 的函数作为参数，并调用他。`lis` 是形参，`(String) -> Int)`是函数类型。下面的两个重载函数的例子，是为了展示只有一个函数参数时的简化写法，多个参数时使用常规写法。结合上面 `lambda` 表达式的相关用法。

```kotlin
// 只有一个函数做参数，并且只有一个参数
fun testFunOneFunParam(lis: (String) -> Int): Int {
    return lis("测试函数作为参数传递,只有一个函数作为参数")
}
// 有多个参数，并且函数不是最后一个
fun testFunMultiParam(lis: (String) -> Int, param: String): Int {
    return lis("测试函数作为参数传递,第一个参数是函数")
}

// 调用
testFunOneFunParam {
    param ->
    param.hashCode()
}

testFunMultiParam({
    param ->
    param.hashCode()
}, "测试函数不是作为第一个参数的情况")
```
函数作为返回值

```kotlin
// 函数作为返回值
fun returnFun(param: String): ((intParam: Int) -> String) {
    return {
        intParam ->
        "int param is $intParam param is $param"
    }
}

// 调用
returnFun("执行完返回一个函数")(100)
```
单表达式函数，当函数的返回值只有一个简单表达式时，可以使用单表达式函数，简化代码

```kotlin
// 单表达式函数
fun testFun2() = 100
// 单表达式函数结合when
fun testFun3(param: Int) = when (param) {
    0 -> 0
    in 1..5 -> 1
    5, 6, 7 -> 2
    else -> 100
}
```

## 数据类
我们经常会创建一些数据类, 什么功能也没有, 而仅仅用来保存数据. 在这些类中, 某些常见的功能经常可以由类中保存的数据内容即可自动推断得到. 在 `Kotlin` 中, 我们将这样的类称为 数据类, 通过 `data` 关键字标记；
 
定义一个数据类，只需要一行代码即可，`{}` 可以省略掉，数据类会默认生成 `equals()` ，`toString()` ， `copy()` 等方法，这里特别提一下 `copy() ` ，可以生成一个新的数据，这个特性是相当不错的。

当然数据类也有很多限制
>主构造器至少要有一个参数;   
>主构造器的所有参数必须标记为 val 或 var;   
>数据类不能是abstract类, open 类, 封闭(sealed)类, 或内部(inner)类;    
>数据类不能继承自任何其他类(但可以实现接口).

```kotlin
data class DataModel(val param1: String, val param2: Int)
// 使用
val dataModel = DataModel("p1",100)
dataModel.copy("p2",99)
```
数据类添加成员方法

```kotlin
data class DataModel(val param1: String, val param2: Int) {
    fun speak(msg: String) {
        Log.e("DataModel", "speak$msg")
    }
}
dataModel.speak("haha")
```




