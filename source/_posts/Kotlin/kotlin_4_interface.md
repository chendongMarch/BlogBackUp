---
layout: post
title: Kotlin开发-4-接口
categories:
  - Kotlin
tags: Kotlin
keywords:
  - Kotlin
  - 接口
abbrlink: 6a146f82
date: 2017-06-08 00:00:00

---

本文主要介绍 `Kotlin` 接口。


<!--more-->

## 声明

接口使用 `interface` 关键字声明；

- 接口中可以包含抽象方法的声明，也可以包含方法的实现。
- 接口与抽象类的区别在于， 接口不能存储状态数据。 
- 接口可以有属性， 但这些属性必须是抽象的，不能使用初始化器来初始化，或者必须提供访问器的自定义实现。
- 接口不支持 `Backing Field`，因此 `var` 变量无法定义访问器，因此 `var` 类型的属性必定是抽象的。

```kotlin
interface FirstInterface {

    // 可以具有属性，但是属性必须是抽象的
    // 或者必须提供访问器的自定义实现
    val testVal1: Int
    var testVar2: Int
    val testVal3: Int
        get() = 100

    // 编译错误，接口不支持 backing field
    // 因此 var 类型的属性不能自定义访问器
    var testVar4: Int
        get() = 100

    // 编译错误，属性必须是抽象的
    // 或者必须提供访问器的自定义实现
    val testVal10: Int = 1
    var testVar10: Int = 1

    // 可以包含抽象方法的声明
    fun testFun1()

    // 也可以包含方法的实现
    fun testFun2(param: Int): String {
        return "test fun 2"
    }
}
```

## 实现接口

```kotlin

class SecondInterface:FirstInterface{

    // 使用初始化器覆盖接口抽象属性
    override val testVal1: Int = 10

    // 使用自定义访问器覆盖接口抽象属性
    override var testVar2: Int
        get() = 100
        set(value) {}

    // 覆盖接口非抽象属性
    override val testVal3: Int
        get() = super.testVal3

    // 覆盖接口抽象方法
    override fun testFun1() {

    }

    // 覆盖接口非抽象方法
    override fun testFun2(param: Int): String {
        return super.testFun2(param)
    }
}
```

## 解决接口覆盖冲突

由于接口是可以多继承的，如果实现多个接口，同时接口中有相同方法的声明，就会出现覆盖冲突，使用官网的一个例子来说明一下覆盖冲突的解决。单继承时自然要实现接口中所有抽象方法，当实现多个接口时，如果实现的接口中具有同名的抽象方法，即使在接口中对该方法都已经有了实现，那么在子类中也必须实现该方法，并使用`super<接口名称>.方法名`，如下面的 `super<A>.bar()` 来在子类中显式的声明到底是继承哪一个实现。	

```kotlin
interface A {
    fun foo() { print("A") }
    fun bar()
}

interface B {
    fun foo() { print("B") }
    fun bar() { print("bar") }
}

class C : A {
    override fun bar() { print("bar") }
}

class D : A, B {
    // 重名函数必须实现，即使继承的接口中已经有了具体实现
    override fun foo() {
        super<A>.foo()
        super<B>.foo()
    }

    override fun bar() {
        super<B>.bar()
        // 由于 A 中对 bar 没有实现，可以如下简写
        // super.bar()
    }
}
```
 