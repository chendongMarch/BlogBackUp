---
layout: post
title: Kotlin开发-15-操作符重载
categories:
  - Kotlin
tags:
  - Kotlin
keywords:
  - Kotlin
  - 操作符重载
abbrlink: 75caba4c
date: 2017-08-26 8:51:00

---


在 `Kotlin` 中允许对操作符进行重载，并提供了简单友好的方式来支持这个特性，这样的好处就是我们可以对一些原本不支持操作符的类型使用操作符，简化代码，举个例子，`Kotlin` 中的集合类型是支持操作符的，而 `Java` 中不支持，对比一下使用和不使用操作符对集合增减元素的操作。

```kotlin
// 使用操作符
val a = list + "1"
val b = list + destList
val c = list - "2"
val d = list - destList

// 不使用操作符
val e = list.add("1")
val f = list.addAll(destList)
val g = list.remove("2")
val h = list.removeAll(destList)
```

<!--more-->

操作符重载的基本原理就是每种操作符都有其对应的函数名，这是一一对应的，当对某个类型的对象使用操作符时，编译器便会查找该类型下，是否存在该名称的函数，并使用 `operator` 关键字标记，**可以是成员函数也可以是扩展函数**，如果有的话，匹配相应的参数，调用该函数。

## 操作符函数一览表

操作符 | 函数 | 描述
:--|:--|:--
+a|unaryPlus()|
-a|unaryMinus()|
!a|not()|
a++/++a|inc()|
a- -/- -a|dec()|
a+b|plus(b)|
a-b|minus(b)|
a*b|times(b)|
a/b|div(b)|
a%b|rem(b)|
a+=b|plusAssign(b)|不能有返回值
a-=b|minusAssign(b)|不能有返回值
a\*=b|timesAssign(b)|不能有返回值
a/=b|divAssign(b)|不能有返回值
a%=b|remAssign(b)|不能有返回值
a[i][j]|get(i,j)|
a[i][j=b]|set(i,j,b)|至少有一个参数为b
a(p)|invoke(p)|
a..b|rangeTo(b)|
b in a|contains(b)|
a==b |equals(b)|
a>b|compareTo(b)|


## 一元操作符

主要包括 `+a,-a,!a,a++,a--,++a,--a` 等一元操作符运算。

```kotlin
class MyData(var myName: String, var myAge: Int):Comparable<MyData> {
    
    var desc = ""

    override fun toString(): String {
        return "name $myName,age $myAge $desc"
    }

    // +a
    operator fun unaryPlus(): MyData {
        desc = "执行了 +a "
        return this
    }

    // -a
    operator fun unaryMinus(): MyData {
        desc = "执行了 -a "
        return this
    }

    // !a
    operator fun not(): MyData {
        desc = "执行了 !a "
        return this
    }
    
    // a++/++a
    operator fun inc(): MyData {
        desc = "执行了 a++"
        return this
    }

    // a--/--a
    operator fun dec(): MyData {
        desc = "执行了 a--"
        return this
    }
}
```

调用

```kotlin
var data = MyData("chendong", 11)
log((+data).toString())
log((-data).toString())
log((!data).toString())
log((data++).toString())
log((data--).toString())
log((++data).toString())
log((--data).toString())
```

## 四则运算与模运算

这类运算符使得任何类型的对象都可以进行 `+ - * / %` 的运算，当然这些运算的具体实现细节是由我们自己实现的。

```kotlin
class MyData(var myName: String, var myAge: Int):Comparable<MyData> {

    var desc = ""

    override fun toString(): String {
        return "name $myName,age $myAge $desc"
    }

    
    // 二元操作符


    // a + b
    operator fun plus(p: MyData): MyData {
        desc = "执行了 a+b"
        myAge += p.myAge
        return this
    }

    // a - b
    operator fun minus(p: MyData): MyData {
        desc = "执行了 a-b"
        myAge -= p.myAge
        return this
    }

    // a * b
    operator fun times(p: MyData): MyData {
        desc = "执行了 a*b"
        myAge *= p.myAge
        return this
    }

    // a / b
    operator fun div(p: MyData): MyData {
        desc = "执行了 a/b"
        myAge /= p.myAge
        return this
    }

    // a % b
    operator fun rem(p: MyData): MyData {
        desc = "执行了 a%b"
        myAge %= p.myAge
        return this
    }
    // a += b
    operator fun plusAssign(p: String) {
        desc = "执行了 a+=b"
    }

    // a -= b
    operator fun minusAssign(p: String) {
        desc = "执行了 a-=b"
    }

    // a *= b
    operator fun timesAssign(p: String) {
        desc = "执行了 a*b"
    }

    // a /= b
    operator fun divAssign(p: String) {
        desc = "执行了 a/=b"
    }

    // a %= b
    operator fun remAssign(p: String) {
        desc = "执行了 a%b"
    }
}
```

调用

```kotlin
var data = MyData("chendong", 11)
var data2 = MyData("new", 10)
log((data + data2).toString())
log((data - data2).toString())
log((data * data2).toString())
log((data / data2).toString())
log((data % data2).toString())
data += "test"
log(data.toString())
data -= "test"
log(data.toString())
data *= "test"
log(data.toString())
data /= "test"
log(data.toString())
data %= "test"
log(data.toString())
```


## 下标访问

这类匀速符使对象可以如同数组一样使用 `a[i][j]..` 的形式访问。

```kotlin
class MyData(var myName: String, var myAge: Int):Comparable<MyData> {

    var desc = ""

    override fun toString(): String {
        return "name $myName,age $myAge $desc"
    }
    
    // a[i]
    operator fun get(i: Int): MyData {
        desc = "执行了 a[$i]"
        return this
    }

    // a[i][j]
    operator fun get(i: Int, j: Int): MyData {
        desc = "执行了 a[$i][$j]"
        return this
    }

    // a[i] = "test"
    operator fun set(i: Int, value: String): MyData {
        desc = "执行了 a[$i]= $value"
        return this
    }

    // a[i][j] = "test"
    operator fun set(i: Int, j: Int, value: String): MyData {
        desc = "执行了 a[$i][$j]= $value"
        return this
    }
}
```

调用

```kotlin
var data = MyData("chendong", 11)
log(data[0])
log(data[1][2])
data[0] = "test"
log(data.toString())
data[1][2] = "test"
log(data.toString())
```

## 函数调用

这个操作符使得我们可以像函数一样直接使用对象后面添加括号，如 `a()` 来调用这个对象的 `invike()` 函数。

```kotlin
class MyData(var myName: String, var myAge: Int):Comparable<MyData> {

    var desc = ""

    override fun toString(): String {
        return "name $myName,age $myAge $desc"
    }


    // a()
    operator fun invoke(): MyData {
        desc = "执行了 a()"
        return this
    }

    // a("test")
    operator fun invoke(p: String): MyData {
        desc = "执行了 a(\'$p\')"
        return this
    }
}
```

调用

```kotlin
var data = MyData("chendong", 11)
var data2 = MyData("new", 10)

log(data().toString())
log(data("test").toString())
```

## 其他操作符

使用 `..` 操作符

```kotlin
// a .. b
operator fun rangeTo(p: MyData): MyData {
    desc = "执行了 a..b"
    return this
}
```

使用 `in` 操作符

```kotlin
// b in a
operator fun contains(p: MyData): Boolean {
    desc = "执行了 b in a"
    return true
}
```

使用 `==` 操作符

```kotlin
override fun equals(other: Any?): Boolean {
    desc = "执行了 =="
    return super.equals(other)
}
```

使用 `<,>,<=,>=` 操作符

```kotlin
override fun compareTo(other: MyData): Int {
    desc = "执行了 比较操作    "
    return 0
}
```

调用

```kotlin
var data = MyData("chendong", 11)
var data2 = MyData("new", 10)
            
data == data2
log(data.toString())

data > data2
log(data.toString())

log((data..data2).toString())

if (data.contains(data2)) {
    log(data.toString())
}
```
	