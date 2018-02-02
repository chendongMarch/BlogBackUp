---
layout: post
title: Kotlin开发-2-基础-控制流
categories:
  - Kotlin
tags: Kotlin
keywords:
  - Kotlin
  - if
  - when
  - for
  - while
  - return
  - continue
  - break
abbrlink: 57e86c00
date: 2017-06-04 03:00:00

---

本篇主要介绍 `Kotlin` 中

控制流的使用，`if` , `when` , `for` , `while` 等关键字在 `Kotlin` 中的新特性。

如何在函数中返回和跳转，`return` , `continue` , `break` 等关键字的使用。

<!--more-->


## if 判断结构

```kotlin
var a = 0
if (a == 0) a = 100
if (a == 0) a = 100 else a = 1000

// 作为表达式时必须有else分支
val rst1 = if (a == 0) 100 else 1000

// 具有多行语句必须使用{}
val rst2 = if (a == 0) {
    logError(100)
    100
} else {
    logError(1000)
    1000
}
```

## when 分支结构

`Kotlin` 使用 `when` 关键字实现分支结构。

当 `when` 作为表达式出现时，必须具有 `else` 分支，除非你的分支已经包含了所有的情况，这是因为作为表达式时他必须有一个返回值，而 `else` 分支将会在所有条件都不满足时执行。

`when` 作为表达式

```kotlin
fun testFun3(param: Int): Int = when (param) {
    // 一般用法
    0 -> 0
    // 使用 in 操作符匹配范围
    in 1..5 -> 1
    !in 100..105 -> -1
    // 使用多个匹配
    5, 6, 7 -> 2
    // 类型判断
    is Int -> 3
    // 作为表达式必须有else分支
    else -> 100
}
```
在函数中使用 `when`

```kotlin
// 作为函数时不需要必须有 else 分支
fun testFun5() {
    val param: Any? = 100
    when (param) {
        // 自动的类型推断，此时param 是 String 类型
        is String -> param.startsWith("")
    }
}
```

## for 循环结构

任何值, 只要能够产生一个迭代器( `iterator`), 就可以使用 `for` 循环进行遍历。

能够产生一个迭代器是指：

> 存在一个成员函数- 或扩展函数 iterator(), 它的返回类型应该 存在一个成员函数- 或扩展函数 next(), 并且 存在一个成员函数- 或扩展函数 hasNext(), 它的返回类型为 Boolean 类型。

遍历数组和 `List` 

```kotlin
// 遍历临时数组
for (item in (1..4)) {
    logError(item)
}

// 遍历list
val lists = mutableListOf(1, 2, 4)
// 遍历item
for (item in lists) {
    logError(item)
}
// 使用下标遍历
for (i in lists.indices) {
    logError(lists.get(i))
}
// 获取下标和值
for((i,item) in lists.withIndex()){
    logError("$i = $item")
}

// 遍历数组也是一样的
val arrays = arrayOf(1,2,3)
for (item in arrays){
    logError(item)
}
for (i in arrays.indices){
    logError(arrays[i])
}
```

遍历 `map`

```kotlin
// 遍历map
val maps = mapOf(Pair("a", "b"), Pair("a1", "b1"))
for ((key, value) in maps) {
    logError("map($key->$value)")
}
```
遍历字符串

```kotlin
// 遍历字符串
val str = "test"
for (s in str){
    logError("$s in str")
}
```

## while 循环结构

用法与其他语言是一致的。

```kotlin
var a = 0
while (a > 100) {
    logError("a is $a")
    a++
}

do {
    logError("a is $a")
    a++
} while (a > 100)
```


## 返回与跳转

`Kotlin` 中有三种标签可以跳出程序流程
> return. 默认行为是, 从最内层的函数或 匿名函数 中返回。   
> break. 结束最内层的循环。   
> continue. 在最内层的循环中, 跳转到下一次循环。   

`Kotlin` 中的任何表达式都可以用 `label` 标签来标记。标签的形式与标识符相同, 后面附加一个 `@` 符号,如 `loopOut@`,使用标签标记位置，就可以使用程序跳出操作符跳出指定位置，如 `break@loopOut`,`continue@loopOut`,`return@loopOut`。需要注意的是中间不需要有空格，他们是一体的。如果有标签的同时又有返回值，使用 `return@loopOut 100` 这样的形式，意为跳出到 `loopOut` 标签位置，返回值是 `100` 。

```kotlin
fun testFun6() {
    loopOut@ for (i in 1..10) {
        loopIn@ for (j in 10..100) {
            if (i * j == 100)
                break@loopOut
            if (i * j < 10)
                continue@loopIn
            if (i * j == 101)
                break@loopIn
            logError("result = ${i * j}")
        }
    }
}
```
> 在 `Kotlin` 中, 通过使用字面值函数(`function literal`), 局部函数(`local function`), 以及对象表达式(`object expression`), 允许实现函数的嵌套。通过标签限定的 `return` 语句, 可以从一个外层函数中返回. 最重要的使用场景是从 `Lambda` 表达式中返回。

如下面的例子中，默认会从函数 `testFun61()` 中返回，返回值为9，而且你写 `return` 语句时，编译器会提示你必须返回一个 `Int` 类型。这种非局部的返回(`non-local return`), 仅对传递给 内联函数(`inline function`) 的 `Lambda` 表达式有效。

```kotlin
//  函数会返回 9
fun testFun61(): Int {
    val arrays = Array(10, { i -> i * i })
    arrays.forEach {
        if (it == 9)
            return it
    }
    return 0
}
```
如果需要从内部 `Lambda` 表达式跳出而不是从函数中返回，可以使用标签指定跳出目标。使用隐含标签会更方便一些, 隐含标签的名称与 `Lambda` 表达式被传递去的函数名称相同。如下面的隐含标签为 `forEach@`。

```kotlin
// 此时返回 0
fun testFun62():Int{
    val arrays = Array(10, { i -> i * i })
    arrays.forEach signal@{
        if (it == 9)
            return@signal
    }
    return 0
}

// 使用隐含标签
fun testFun63():Int{
    val arrays = Array(10, { i -> i * i })
    arrays.forEach {
        if (it == 9)
            return@forEach
    }
    return 0
}
```
