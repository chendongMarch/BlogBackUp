---
layout: post
title: Kotlin开发-12-空值安全
categories:
  - Kotlin
tags:
  - Kotlin
keywords:
  - Kotlin
  - 空值安全
  - NonNull
abbrlink: c5b3d729
date: 2017-08-07 18:15:00

---


在进行属性的初始化时能够感受到 `Kotlin` 在编译时对 `null` 值的控制，正是因为 `Kotlin` 的这些特性，使得 `Kotlin` 成为一种 `空安全(Null Safaty)` 的语言，遵循 `Kotlin` 的规范将有效的避免程序中的 `NPE`。

其实遵循 `Kotlin` 的变量的声明和初始化方式基本可以避免 `NPE` 的出现，`Kotlin` 不允许给一个 变量 `null` 值，但是你可以使用 `Type?` 的形式(例如 `String?` )声明这是一个可以为 `null` 的变量，他就可以被初始化为 `null`。

除此之外，`kotlin` 提供了丰富的运算符在运行时进行 空值检测 ，简化代码逻辑。

<!--more-->

## 安全调用 (?.)

安全调用即 `safe calls`，使用`?.`操作符。

当我们访问一个变量的属性或者方法时，为了避免 `NPE` 我们通常要在前期做严密的空值判断确保调用的对象不为空时才能放心调用，`safe calls` 的机制大大简化了这个过程。

`safe calls` 只有当引用不为 `null` 时才进行调用，否则直接返回 `null`，如下情况中 `b` 的类型为 `Int?`，因为 `a` 可能为 `null`，调用 `a` 的 `length` 属性，如果 `a` 不为 `null`，则返回 `Int` 类型的长度，反之返回 `null`，因此 `b` 为 `Int?`，即可能为 `null` 的 `Int` 类型。

```kotlin
val a: String? = null
val b = a?.length
```

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

有时我们在判空之后会执行一系列的操作，此时就可以结合 `let` 关键字，使用 `?.let{}` 来确保变量不为空时执行方法，同时在 `let` 方法体内，可以使用 `it` 关键字访问变量。

```kotlin
val a: String? = null
val b = a?.length
// b不为null时进行打印
b?.let { log("$it") }
```

## Elvis 操作符 (?:)

`Elvis 操作符` 即 `?:` 操作符。当使用 `?.` 操作符进行安全调用时，如果遇到 `null` 则会直接返回 `null`，那如果当为 `null` 时返回其他值时如何操作呢？`Elvis 操作符` 就是用来解决这个问题的。

如果 `?:` 左侧的表达式值不是 `null`, `Elvis 操作符` 就会返回它的值, 否则, 返回右侧表达式的值. 注意, 只有在左侧表达式值为 `null` 时, 才会计算右侧表达式。

```kotlin
// 原始版本，使用 if...else
val l: Int = if (b != null) b.length else -1
// 使用 ?: 操作符
val l = b?.length ?: -1
```

由于在 `Kotlin` 中 `throw` 和 `return` 都是表达式, 因此它们也可以用在 `Elvis 操作符` s的右侧. 这种用法可以带来很大的方便, 比如, 可以用来检查函数参数值是否合法:

```kotlin
fun foo(node: Node): String? {
    val parent = node.getParent() ?: return null
    val name = node.getName() ?: throw IllegalArgumentException("name expected")
    // ...
}
```

## !! 操作符

如果你确实清楚此时可以抛出一个 `NPE`，那么你可以使用 `!!` 操作符。

对于 `b` 不为 `null` 的情况, 这个表达式将会返回这个非 `null` 的值, 如果 `b` 是 `null`, 这个表达式就会抛出一个 `NPE`。

```kotlin
val l = b!!.length
```


## as? 操作符

如果对象不是我们期望的目标类型, 那么通常的类型转换就会导致 ` ClassCastException` 。 使用 `as?` 进行安全的类型转换, 如果转换不成功, 它将会返回 `null`。

```kotlin
val aInt: Int? = a as? Int
```
 


