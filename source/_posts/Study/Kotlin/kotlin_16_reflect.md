---
layout: post
title: Kotlin开发-16-反射
categories:
  - Kotlin
tags:
  - Kotlin
keywords:
  - Kotlin
  - 反射
abbrlink: 2650e1c8
date: 2017-08-26 09:59:00

---

本文介绍 `Kotlin` 反射的用法。

在 `Java` 中也有反射的相关用法，不过因为运行时反射效率差，而且 `java` 中的反射使用起来相对麻烦，在实际开发中反射的使用还是很少的。不过在 `Kotlin` 中的反射使用起来十分简单。

反射，我们平常访问对象的属性和函数，是针对对象来说的，总是访问他们的 "值"，比如一个属性的具体值，或者执行某个函数，都是使用对象来调用，这都是访问 "值" 的操作，而反射，是针对类来说的，也就是不必有类实体，而是访问类的属性和函数本身，不再在乎值到底是什么，而是真正的把这个属性或函数拿出来，当我们想要这个属性的值或函数的执行结果，我们就需要用拿到的属性和函数本身去调用一个对象，这个过程就反过来了，不是对象调用属性和函数，而是属性和函数本身调用对象。

<!--more-->

在 `Kotlin` 中使用反射需要借助 `::` 操作符。

## Class Reference

获取某个类的引用

```java
// 获取 kotlin class
var kClass = ReflectTest::class
// 获取 java class
var clazz = ReflectTest::class.java
```
从 `KClass` 类型的变量中可以获取类的的属性

```java
kClass.constructors
kClass.isAbstract
kClass.isData
kClass.isFinal
kClass.isCompanion
kClass.isInner
kClass.isOpen
...
```


## Function Reference

反射获取函数引用，非类成员函数时可以直接使用 `::funcName` 的形式获取函数，

```java
// top level 级别函数
fun funcInTopLevel(str: String) = str.length

class ReflectFunTest() {

    // 成员函数
    fun funcInClass(str: String) = str.length

    fun test() {

        // 局部函数
        fun funcInLocal(str: String) = str.length

        // ok
        val f1: (String) -> Int = ::funcInTopLevel
        // error
        val f2: (String) -> Int = ::funcInClass
        // ok，局部函数
        val f3: (String) -> Int = ::funcInLocal
    }
}
```

对于类成员函数反射返回的结果其实是 `类名.(参数)->返回值` 类型的函数，因此上面是编译错误的。

```java
val f4: ReflectFunTest.(String) -> Int = ReflectFunTest::funcInClass
```	
也可以获取 `String` 类的成员函数

```java
val f3: String.() -> CharArray = String::toCharArray
val f4: String.(Int, Int) -> CharSequence = String::subSequence
```

执行函数，对于非类成员函数可以直接执行，而对于类成员函数则需要依赖类的实例来执行。

```java
val f1: (String) -> Int = ::funcInTopLevel
val f4: ReflectFunTest.(String) -> Int = ReflectFunTest::funcInClass

f1.invoke("test")
f4.invoke(ReflectFunTest(), "test")
```

## Property Reference

反射获取属性引用，支持类属性和扩展属性。

```java
// 扩展属性
val ReflectPropertyTest.extensionProperty: Int
    get() = 200

class ReflectPropertyTest {

    // 成员属性 
    var myProperty = 10

    fun test() {

        val model = ReflectPropertyTest()

        // 反射属性
        val kPropertyForMyProperty = ReflectPropertyTest::myProperty
        // 从 model 中取出该属性的值
        var myPropertyValue = kPropertyForMyProperty.get(model)
        // 向 model 中设置该属性的值
        kPropertyForMyProperty.set(model, 100)


        // 反射扩展属性
        val kPropertyForExtensionProperty = ReflectPropertyTest::extensionProperty
        // 从 model 中取出该属性的值
        var extensionPropertyValue = kPropertyForExtensionProperty.get(model)


        // 反射其他类属性
        val lengthProperty = String::length
        lengthProperty.get("test")
    }
}
```


## Constructor Reference

反射获取构造器引用

```java
class ReflectConstructorTest() {
    
    constructor(age: Int) : this()

    constructor(age: Int, name: String) : this(age)

    fun test() {
        testConstructor1(::ReflectConstructorTest)
        testConstructor2(::ReflectConstructorTest)
        testConstructor3(::ReflectConstructorTest)
    }

    fun testConstructor1(factory: () -> ReflectConstructorTest): ReflectConstructorTest {
        return factory.invoke()
    }

    fun testConstructor2(factory: (Int) -> ReflectConstructorTest): ReflectConstructorTest {
        return factory.invoke(12)
    }

    fun testConstructor3(factory: (Int, String) -> ReflectConstructorTest): ReflectConstructorTest {
        return factory.invoke(12, "chen")
    }
}
```