---
layout: post
title: Kotlin开发-9-object关键字
categories:
  - Kotlin
tags:
  - Kotlin
keywords:
  - Kotlin
  - object
  - 对象表达式
  - 对象声明
abbrlink: 40486d0b
date: 2017-07-29 16:47:00

---

本文学习 `Kotlin` 中 `object` 关键字的使用。

使用 `object` 关键字的对象表达式可以创建匿名对象，适合那些只需要使用一次的类实现，使用匿名对象我们不需要给这些只用一次的类对象声明 `class` 而是在运行时直接创建即可。

使用 `object` 关键字进行对象声明，借助这种方法可以简单的实现单例和同伴对象。

<!--more-->


## 对象表达式

对象表达式（Object expression）

对象表达式则会在使用处 **立即** 执行(并且初始化)

使用 `object` 可以创建匿名内部类，以点击事件为例

```kotlin
mMyMsgTv.setOnClickListener(object: View.OnClickListener{
    override fun onClick(v: View?) {
        // click
    }
})

// 使用 Lambda 表达式简化。
mMyMsgTv.setOnClickListener {
    log("click")
}
```

自己定义两个接口

```kotlin
interface OneFunInterface{
    fun test(param:String)
}

interface TwoFunInterface{
    fun test1(param:String)
    fun test2(param:String)
}
```
我们可以使用匿名内部类创建对象

```kotlin
val o1 = object : OneFunInterface {
    override fun test(param: String) {
    }
}
val o2 = object : TwoFunInterface {
    override fun test1(param: String) {
    }
    override fun test2(param: String) {
    }
}
```
实现匿名内部类时可以继承多个基类，如果基类有构造器，那么必须传递合适的参数

```kotlin
val o3 = object:User("zhang",22),OneFunInterface{
    override fun test(param: String) {
        
    }
}
```
也可以不继承任何基类直接创建对象，👍

```kotlin
val o4 = object{
    val x = 10
    val y = 20
}
log(o4.x)
```

对象表达式的内部可以访问创建对象表达式的域内的变量，这点和 `Java` 的匿名内部类是一样的， 但是 `Kotlin` 中不需要将变量强制声明为 `final` ，👍

```kotlin
fun test(){
	var t = 100 // 不必声明为 final
	val o1 = object : OneFunInterface {
   	 	override fun test(param: String) {
   	   	  t = 10
   	 	}
	}
}
```

## 对象声明

对象声明（Object declaration）不可以是局部的(也就是说，不可以直接嵌套在函数之内)， 但可以嵌套在另一个对象声明之内，或者嵌套在另一个非内部类之内。

对象声明是 **延迟(lazily)** 初始化的， 只会在首次访问时才会初始化

使用 `object` 关键字创建单例，不能具有构造器，可以指定基类，可以直接使用类名引用。

```kotlin
object Singleton : User(name = "test", age = 12) {
    fun testFun() {
        Log.e(javaClass.simpleName, "name = $name")
    }
    val testVal = age
}

// usage
Singleton.testFun()
Singleton.testVal
```

同伴对象，一个类内部对象声明可以使用 `companion` 关键字标记为同伴对象(`Companion Object`)。`Companion Object` 可以具有自己的类名，也可以直接使用默认类名 `Companion`。`Companion Object` 可以继承基类和实现接口。

同伴对象会在对应的类被装载(解析)时初始化

> 虽然同伴对象的成员看起来很像其他语言中的类的静态成员，但在运行时期，这些成员仍然是真实对象的实例的成员，它们与静态成员是不同的，但是，如果使用`@JvmStatic` 注解， 你可以让同伴对象的成员在 `JVM` 上被编译为真正的静态方法(static method)和静态域(static field)。

```kotlin
// 默认类名 Companion
class MyClass {
    companion object {
        fun testFun() {}
        val testVal = 10
    }
}

// 自己的类名
class MyClass1 {
    companion object MyCompanion {
        fun testFun() {}
        val testVal = 10
    }
}

// 继承基类 实现接口
class MyClass2 {
    companion object : User(), OneFunInterface {
        override fun test(param: String) {
            
        }
        fun testFun() {}
        val testVal = 10
    }
}
```
我们可以直接使用外部类的类名调用其同伴对象的方法和属性

```kotlin
 MyClass.Companion
 MyClass1.MyCompanion
 MyClass.testVal
 MyClass.testFun()
```
 
