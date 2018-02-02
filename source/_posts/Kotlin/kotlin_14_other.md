---
layout: post
title: Kotlin开发-14-杂七杂八(解构，类型自动转换，Range，异常)
categories:
  - Kotlin
tags:
  - Kotlin
keywords:
  - Kotlin
  - 类型检测
  - 类型自动转换
  - as
  - is
  - Range
  - 解构声明
abbrlink: 578c5201
date: 2017-08-10 10:18:00

---

汇总一些杂七杂八的点，不值得单独开一篇整理了。

主要内容包括

1. 解构声明
2. 生成数列的 `Range`
3. 类型转换
4. 异常处理等

<!--more-->

## 解构声明

我们创建对象时通常会使用构造函数，他是将一些变量，构造成一个对象，解构就是构造的反操作，解构声明(Destructuring Declaration) 指的是可以将一个对象解构成多个变量。

对象的解构主要依赖于类的 `componentN()` 函数，函数需要使用 `operator` 注解标记，这不是一种强制实现，更像是一种约定声明。

之前介绍数据类时说道，数据类中会默认实现 `componentN()` 函数，因此我们对数据类对象可以直接进行如下操作：

```kotlin
data class DataUser(val name: String, val age: Int)

// 数据类默认声明 ComponentN 函数
val (name, age) = DataUser("chendong", 12)
```
借助数据类的这种特性，我们可以从函数中返回多个值，其实本质上，还是借助了数据类可以自动解构的特性。

ps:`Pair` 类也是数据类。

```kotlin
// 从函数中返回多个值
fun returnMultiFunc():DataUser{
    return DataUser("chendong",12)
}
fun returnPairFunc():Pair<String,Int>{
    return Pair("chendong",12)
}

val (name2,age2) = returnMultiFunc()
val (name3,age3) = returnPairFunc()
```

那么对于我们自己创建的类，他不是数据类，如何进行解构，解决方法还是声明 `componentN()` 函数。

在 `componentN()` 函数中返回指定的属性，会自动匹配到解构的接受者中，`name1` 对应的 `component1()` 的返回值，`age1` 对应 `component2()` 返回值。

```kotlin
class MyUser(val name: String, val age: Int) {
    operator fun component1(): String {
        return "my name = $name"
    }
    operator fun component2(): Int {
        return age + 10
    }
}

// 接下来就可以直接进行解构
val myUser = MyUser("chendong", 12)
val (name1, age1) = myUser
```

借助 `Map` 类的遍历，看一个解构声明的具体应用。

```kotlin
for ((key, value) in map) {
   // 使用 key 和 value 执行某种操作
}
```
要想实现遍历操作，首先需要实现 `iterator`

```kotlin
operator fun <K, V> Map<K, V>.iterator(): Iterator<Map.Entry<K, V>> = entrySet().iterator()
```
返回一个 `Map.Entry<K, V>` 的集合，此时需要将 `Map.Entry` 的 `key` 和 `value` 进行解构声明。

```kotlin
operator fun <K, V> Map.Entry<K, V>.component1() = getKey()
operator fun <K, V> Map.Entry<K, V>.component2() = getValue()
```

## Range 数列

可以使用 `..` 运算符生成一个数列，对于 `Char`，`Int`，`Long` 类型数列由  `CharProgression`，`IntProgression`，`LongProgression` 实现，而 `Float` 和 `Double` 类型由 `ClosedFloatingPointRange<T>` 实现，因此在支持性上有所不同。

使用简单的 `..` 运算符生成 `IntRange` 对象.

```kotlin
val range = 1..10
```
遍历数列，构造数列时，后面的范围要比前面的大，所以下面 `4..1` 是不会有任何结果的。

```kotlin
for (i in 1..4) {
    Log.e(TAG, "test 1..4 i = " + i)
}

// 不打印结果
for (i in 4..1) {
    Log.e(TAG, "test 4..1 i = " + i)
}
```
使用 `..` 并不能构造一个递减数列，此时需要使用 `downTo()` 函数，构造递减数列。

```kotlin
for (i in 4 downTo 1) {
    Log.e(TAG, "test downTo i = " + i)
}
```

使用 `step()` 函数指定步长，表示在构造数列时的间隔，默认是 1。

```kotlin
// 步长
for (i in 1..4 step 2) {
    Log.e(TAG, "test 1..4 step 2 i = " + i)
}
```
输出结果为，每隔两个会获取一个，所以最后一个数将会是 3 ，因为下一个数是 5，不在范围内，类似的 `(1..4 step 2).last` 的结果也会是 3。

```kotlin
testRange: test 1..4 step 2 i = 1
testRange: test 1..4 step 2 i = 3
```
使用 `until` 函数构造一个前闭后开的区间，比如下面的代码将会创建一个 `[1,4)`  区间的数列。

```kotlin
for (i in 1 until 4) {
    Log.e(TAG, "test 1 until 4 i = " + i)
}
```
反转数列 

```kotlin
1.rangeTo(10).reversed()
(1..10).reversed()
```
	
所有的基本类型在生成数列的实现方式不同，但是都支持 `..` 运算符生成数列，另外 `Byte`，`Short`，`Int`，`Long` 在生成数列时都是可以自动转换的。

```kotlin
// byte
(0b1..0b11)
// char
('a'..'z')
// int
(1..10)
// long
(1L..10L)
// float
(1f..2.2f)
// double
(1.1..2.2)
```

运算符 `..` 对应的 `rangeTo()` 函数，其中 `Float` 和 `Double` 的 `rangeTo()` 函数使用扩展函数，其他为成员函数。

```kotlin
// byte
0b1.rangeTo(0b111)
// char
'a'.rangeTo('z')
// int
1.rangeTo(10)
// long
1L.rangeTo(10L)
// double
1.1.rangeTo(2.2)
// float
1.1f.rangeTo(2.2f)
```
函数 `downTo()`，`Float` 和 `Double` 不支持 `downTo()`

```kotlin
// byte
0b11.downTo(0b1)
// int
1.downTo(10)
// char
'a'.downTo('z')
// long
1L.downTo(10L)
// double error
// 1.1.downTo(2.2)
// float error
// 1.1f.downTo(2.2f)
```
函数 `step()`，`Float` 和 `Double` 不支持 `step()`

```kotlin
// byte
0b1.rangeTo(11L).step(2)
// int
1.rangeTo(10).step(2)
// char
'a'.rangeTo('z').step(2)
// long
1L.rangeTo(10L).step(2)
// double error
// 1.1.rangeTo(2.2).step(2)
// float error
// 1.1f.rangeTo(2.2f).step(2)
// int
1.downTo(10).step(2)
// char
'a'.downTo('z').step(2)
// long
1L.downTo(10L).step(2)
```

## 类型转换

使用 `is` 关键字进行类型的检测，使用 `as` 关键字进行类型转换。

```kotlin
val str = "100"
val isString = str is String
val notString = str !is String

// Int
val num = str as Int
// Int?
val num2:Int? = str as? Int
```

### 类型自动转换

当使用 `is` 关键字进行了类型判断，那么一定作用域内，类型将被自动转换。

比如在判断结构当中

```kotlin
val myNum: Any = 100
if (myNum is String) {
    // 自动转换为 String
    myNum.length
}

// error
// myNum.length

if (myNum !is String)
    return
// 在此之后将被自动转换为 String
 myNum.length
```

在 `||` 和  `&&` 因为遵循短路判断，因此类型也将会被自动转换。

```kotlin
if (myNum is String && myNum.length > 10) {
}

if (myNum !is String || myNum.length > 10) {
}
```
在 `when` 结构中进行了类型判断

```kotlin
when (myNum) {
    is Int -> myNum.rangeTo(10)
    is String -> myNum.length
    is IntArray -> myNum.size
}
```
并不是所有的属性都可以进行自动的类型转换，需要满足以下条件：

> 局部的 `val` 变量 - 永远有效 。

> `val` 属性 - 如果属性是 `private` 或 `internal` 的，或者类型检查处理与属性定义出现在同一个模块内，那么智能类型转换是有效的。对于 `open` 属性， 或存在自定义 `get` 方法的属性， 智能类型转换是无效的。

> 局部的 `var` 变量 - 如果在类型检查语句与变量使用语句之间，变量没有被改变, 而且它没有被 `Lambda` 表达式捕获并在 `Lambda` 表达式内修改它，那么智能类型转换是有效的。

> `var `属性 - 永远无效(因为其他代码随时可能改变变量值)。



## 异常处理

在 `Kotlin` 中与 `Java` 大致相同，有几个特别的地方。

`try{}` 可以作为一个表达式，并返回值。

```kotlin
val result = try {
    Integer.parseInt("10")
} catch (e: Exception) {
    0
}finally {
    
}
```

不支持函数声明时抛出异常，这项改进主要是避免代码中出现大量的异常捕捉代码块。

在 `Java` 中可以在方法声明时抛出异常，来告知外界使用该函数可能会有异常

```java
public static void test() throws IllegalStateException{

}
```
在 `Kotlin` 中这种语法是不支持的。

```kotlin
// error
fun test() throws IllegalStateException{
    
}
```

	
	