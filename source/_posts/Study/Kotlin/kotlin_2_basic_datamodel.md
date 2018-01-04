---
layout: post
title: Kotlin开发-2-基础-数据类型
categories:
  - Kotlin
tags: Kotlin
keywords:
  - Kotlin
  - 数值
  - 字符
  - 布尔
  - 数组
  - 字符串
abbrlink: 6eea2c65
date: 2017-06-04 02:00:00

---

本篇介绍 `Kotlin` 的内建数据类型，包括数值类型，字符类型，布尔类型，数组类型，字符串类型等。

<!--more-->

## 同一性 和 相等性

同一性：对象的地址，数据类型，内容都相同，在 `Kotlin` 中使用 `===` 运算符进行比较，它相当于 `Java` 中的 `==` 运算符，他要求对象是完全相同的对象，即指向同一地址。

相等性：内容相等，在 `Kotlin` 中使用 `==` 运算法进行比较，相当于 `Java` 中的 `equals()` 方法，这里涉及自定义运算符的一点内容， 在 `Kotlin` 中当你对一个对象使用 `==` 运算符，会自动查找 `equals()` 方法进行调用。
  
## 数值类型

`Kotlin` 数值使用内建对象表示，所有数值类型继承自 `Number` 抽象类，`Number` 中定义了数值在各个类型之间转换的方法。

`Long` 类型需要在数字末尾追加 **大写** 的 `L` 表示，如 `123L`；
 
`Float` 类型需要在数字末尾追加 `f／F` 表示，如 `123.5F`

进制： 默认十进制计数，如： `123`；十六进制以 `0x` 开头，如 `0x1FA`；二进制以 `0b` 开头，如 `0b001101`；**不支持** 八进制表示。
 
默认类型： 小数默认是 `Double` 类型，如果需要显式声明为 `Float`，需要加 `f/F` 后缀；整型如果在 `Int` 范围内默认为 `Int` 类型，需要显式声明为 `Long` 需要添加 `L` 后缀。
  
`Double` (64 bit) ；`Float` (32 bit) ；`Long` (64 bit) ；`Int` (32 bit) ；`Short` (16 bit) ；`Byte` (8 bit) ；
 

### 数值装箱

在 `Kotlin` 中是没有像 `Java` 中那样的基本数据类型，但是内建的数值类型具有和基本数据类型相似的特性，如 `Int` 和 `int` 在比较操作上是完全一样的，可以像基本数据类型那么对待他。


```kotlin
val aInt1 = 10
val bInt1 = 10
Log(aInt1 == bInt1) // true
Log(aInt1 === bInt1) // true

val aInt2 = 129
val bInt2 = 129
Log(aInt2 == bInt2) // true
Log(aInt2 === bInt2) // true
```

在  `Java` 中有基本数据类型的包装类型，区别在于包装类型是对象，可以为 `null` ，在 `Kotlin` 中使用 `Type?` 声明可以为空的类型，如 `Int?` ，这种类型在比较操作上和 `Java` 中的包装类型是一样的， 可以类比来看。

和 `Java` 一样存在着数值常量池的概念，当数值在 `[-128,127]` 之间的数值会缓存在常量池中，因为当值为 10 时，比较起来是相等的。

```kotlin
val aInt3: Int? = 10
val bInt3: Int? = 10
Log(aInt3 == bInt3) // true
Log(aInt3 === bInt3) // true

val aInt4: Int? = 129
val bInt4: Int? = 129
Log(aInt4 == bInt4) // true
Log(aInt4 === bInt4) // false 

val num = 129
val aInt5:Int? = num
val bInt5:Int? = num
Log(aInt5 == bInt5) // true
Log(aInt5 === bInt5) // false
```

### 类型转换

在 `Java` 中的类型转换已经相对严格，它允许将低精度类型隐式转换为高精度类型，但是高精度类型转换为低精度类型时需要进行强制转换，但是 `Kotlin` 在类型转换上更加严格，他不允许进行任何形式的隐式类型转换，也就是说任何的类型转换一定是你主动的操作，不过 `Kotlin` 使用扩展方法提供了一种相对优雅的方式。


```kotlin
val bb : Byte = 1
val ll:Long = 1
// 低精度转高精度
val ii1:Int = bb.toInt()
// 高精度转低精度
val ii2:Int = ll.toInt()
```

### 数值运算符

```kotlin
shl(bits) – 带符号左移 (等于 Java 的<<)
shr(bits) – 带符号右移 (等于 Java 的 >>)
ushr(bits) – 无符号右移 (等于 Java 的 >>>)
and(bits) – 按位与(and)
or(bits) – 按位或(or)
xor(bits) – 按位异或(xor)
inv() – 按位取反
```


## 字符类型

`Char` 类型不是 `Number` 的子类，也就是说 `Char` 类型并不是数值类型。 

`Char` 类型使用 单引号 如 `'a'` 表示。特殊字符使用反斜线转义表达。`Kotlin `支持的转义字符包括: `\t, \b, \n, \r, \', \", \\, \$` 。 其他任何字符, 都可以使用 `Unicode` 转义表达方式: `'\uFF00'`。

`Char` 类型也具有一系列的类型转换方法，你可以使用 `'0'.toInt()` 这样的语法将 `Char` 类型转换为数值类型。

可以使用 `Char?` 类型对 `Char` 类型进行装箱操作，与数值类型一样，装箱操作不能保持对象同一性。


## 布尔类型

`Boolean` 类型用来表示布尔值, 有两个可能的值: `true` 和 `false`.

当需要一个可为 `null` 的布尔值引用时, 布尔值也会被装箱(box).

布尔值的计算与 `Java` 相同。


## 数组类型

`Kotlin` 中的数组通过 `Array` 类表达, 这个类拥有 `get` 和 `set` 函数(这些函数通过运算符重载转换为 `[]` 运算符), 此外还有 `size` 属性, 以及其他一些有用的成员函数。

`Kotlin` 中也有专门的类来表达基本数据类型的数组: `ByteArray`, `ShortArray`, `IntArray` 等等, 这些数组可以避免数值对象装箱带来的性能损耗。这些类与 `Array` 类之间不存在继承关系, 但它们的方法和属性是一致的. 各个基本数据类型的数组类都有对应的工厂函数。

```kotlin
// 初始化一个 Array
val array1 = arrayOf(1, 2, 4)
// 初始化一个全为 null 的 Array
val array2 = arrayOfNulls<Int>(10)

// IntArray类型
val array3 = intArrayOf(1, 2, 3)
// ShortArray类型
val array4 = shortArrayOf(1, 2, 3)

// 使用工厂函数初始化
val array5 = Array(10, {
    i ->
    i * i
})

// 推荐下标访问
logError(array5[0])
logError(array5.get(0))
logError(array5.size)
```

## 字符串类型

使用双引号 `"abc"` 表示转移字符串，支持转义字符。

使用三个双引号 `"""abc"""` 表示原生字符串，原生字符串不支持转义，可以包含任何转义字符。

下面的 `trimMargin()` 函数会将 `|` 替换掉，这样保持在编码时期的整洁，但是在运行期间不会输出。

```kotlin
// 默认使用 | 分割
val text = """
    |Tell me and I forget\n.
    |Teach me and I remember\t.
    |Involve me and I learn.
    |(Benjamin Franklin)
    """.trimMargin()
logError(text)

// 输出
Tell me and I forget\n.
Teach me and I remember\t.
Involve me and I learn.
(Benjamin Franklin)
```

字符串模板，转义字符串和原生字符串都支持字符串模板，使用 `${表达式}` 的方式可以在字符串中间插入数据。只有一个简单值时可以省略 `{}` ,同时也支持复杂表达式的计算。转义字符串中你可以使用 `\$` 来写入 `$` 字符，原生字符串中由于不支持转义字符，你可以使用 `${"$"}` 的方式写入 `$` 字符。

```kotlin
val user = User()
val param = 100
var str = "test str template like $param"
str = "test str template like ${user.name} "
str = "test str template like ${if (user.age <= 0) 100 else user.age}"
```
 
