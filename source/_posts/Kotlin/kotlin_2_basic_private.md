---
layout: post
title: Kotlin开发-2-基础-可见度修饰符
categories:
  - Kotlin
tags: Kotlin
keywords:
  - Kotlin
  - 可见度修饰符
  - public
  - private
  - protected
  - internal
abbrlink: '70650835'
date: 2017-06-04 01:00:00

---

可见度修饰符用来表达类、接口、属性等被开放的程度，即在什么位置可以访问，在什么位置被限制访问。

本篇主要介绍 `Kotlin` 中的四种可见度修饰符，分别为 `public`，`protected`、`internal` 和 `private`。

<!--more-->


类，对象，接口，构造器，函数，属性以及属性访问器设值方法(访问器取值方法总与属性本身可见度相同，因此不需要控制其可见度)，都可以使用 **可见度修饰符** 。

四种可见度修饰符：`private`、`protected`、`internal`、`public`，默认为 `public`。与 `Java` 不同的是少了 `default` 多了 `internal`。

注意：**局部变量，局部函数，以及局部类，都不能指定可见度修饰符**。

## top-level 

`top-level`， 像 `class` 这种可以直接声明在包下。在 `Kotlin` 中，函数, 属性, 类, 对象, 接口都可以声明为 `top-level`。比如扩展函数和扩展属性就是 声明为 `top-level` 的。注意，这里说的是 `top-level` 级别声明，在这些声明内部再声明别的函数或者属性，不在讨论范畴，将在下一部分说明，对于 `top-level` 中的声明来说：

> `public` ：意为该声明在任何位置都可以访问，public 是默认的，可以省略。If you do not specify any visibility modifier, public is used by default, which means that your declarations will be visible everywhere;    

> `private` ：意为该声明只能在同一个源代码文件中访问。If you mark a declaration private, it will only be visible inside the file containing the declaration;   

> `internal` ：意为该声明在同一个module的任意位置是可以访问的。If you mark it internal, it is visible everywhere in the same module;   

> `protected` ：对 `top-level` 的声明是无效的。protected is not available for top-level declarations.

```kotlin
// 类声明
class MyCls 

// 接口声明，同 module 访问
internal interface MyInter 

// 函数声明，只允许在文件内访问
private fun MyFun(){} 

// 属性声明，所有地方都可以访问，默认 public 可以省略
public var MyVal = 10
    // 设置方法只有文件内可以访问
    private set
```

## 类与接口

在类和接口内部使用可见度修饰符

> `private` ：在类内(以及它的所有成员之间)可以访问。private means visible inside this class only (including all its members);
 
> `protected` ：和 `private` 相同，而且在子类中也可以访问。protected — same as private + visible in subclasses too;

> `internal` ：在同一个 `module` 内，能访问该类的地方，也能访问该类的 `internal` 成员。any client inside this module who sees the declaring class sees its internal members;

> `public` ：在任何位置凡是能访问该类，则也能访问该类的 `public` 成员。any client who sees the declaring class sees its public members.

注意： 在 Kotlin 中，外部类不能访问其内部类的 `private` 成员。如果你覆盖一个 `protected` 成员，并且没有明确指定可见度，那么覆盖后成员的可见度也将是 `protected`。

```kotlin
open class OuterCls {
    // 类内可访问
    private var a = 1
    // 类内可访问，子类可访问
    protected open val b = 2
    // 模块内可以访问该类的地方，都可以访问该属性
    internal val c = 3
    // 默认为 public,任何位置，可以访问该类的地方都可以访问该属性
    var d = 4

    fun test() {
        a = 1
        // 外部类访问嵌套类pubic成员
        NestedCls().publicVar
        // 编译错误，外部类不能访问嵌套类 private 成员
        NestedCls().privateVar
        // 编译错误，外部类不能访问嵌套类 protected 成员
        NestedCls().protectedVar
        // 外部类访问内部吧类public成员
        InnerCls().publicVar
        // 编译错误，外部类不能访问内部类 protected 成员
        InnerCls().privateVar
        // 编译错误，外部类不能访问内类 protected 成员
        InnerCls().protectedVar
    }

    // 嵌套类
    private class NestedCls {
        public var publicVar: Int = 5
        private var privateVar: Int = 5
        protected var protectedVar: Int = 5

        // 嵌套类，不能访问外部类成员，不论 private public
        fun test() {
            // 编译错误
            a = 1
            // 编译错误
            d = 10
        }
    }

    inner class InnerCls {
        public var publicVar: Int = 5
        private var privateVar: Int = 5
        protected var protectedVar: Int = 5

        // 内部类，可以访问外部类成员，不论 private public
        fun test() {
            a = 1
            d = 10
        }
    }
}

class Subclass : OuterCls() {
    // a 不可访问
    // b, c 和 d 可以访问
    // Nested 和 e 可以访问

    // 覆盖了父类中的 b，b可见度仍然为 protected，可以被子类覆盖
    override val b = 5

    fun testSub(){
        // 编译错误，子类不可以访问 NestedCls 因为他是 private 的
        NestedCls().publicVar
        // 可以访问 InnerCls，他是 public 的
        InnerCls().publicVar
    }
}

class Unrelated(o: OuterCls) {
    // o.a, o.b 不可访问
    // o.c 和 o.d 可以访问(属于同一模块)
    // Outer.Nested 不可访问, Nested::e 也不可访问
}
```

## 构造器

注意，指定类构造器可见度，你需要明确添加一个 `constructor` 关键字。

>`private` ：表示构造器只在类内可以访问。

>`protected` ：类构造器可见度不支持 `protected`。

>`internal` ：表示同模块内可以访问该构造器。

>`public` ：在任何位置都可访问，构造器默认 `public`。

```kotlin
class MyCls1 private constructor(){
    fun test(){
        // 类内可以访问
        MyCls1()
    }
}
class MyCls2 public constructor()
class MyCls3 internal constructor()
class MyCls4 internal constructor(){
    fun test(){
        // 编译错误，private 的构造器同文件内不能访问
        MyCls1()
    }
}
```

