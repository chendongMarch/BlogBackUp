---
layout: post
title: Kotlin开发-2-基础
categories:
  - Kotlin
tags: Kotlin
keywords:
  - Kotlin
  - 导包
  - 数值
  - 字符
  - 布尔
  - 数组
  - 字符串
  - if
  - when
  - for
  - while
  - return
  - continue
  - break
abbrlink: 811551029
date: 2017-06-04 00:00:00

---

本篇主要介绍 `Kotlin` 的基础语法，控制流，内建数据类型等内容...
 
> `import` 语句和包的相关概念。   

>  可见度修饰符的使用。   

>  大部分内建数据类型的介绍。包括数值类型，字符类型，布尔类型，数组类型，字符串类型。

>  控制流的使用，`if` , `when` , `for` , `while` 等关键字在 `Kotlin` 中的新特性。

>  如何在函数中返回和跳转，`return` , `continue` , `break` 等关键字的使用。

go~

<!--more-->

## 包

```kotlin
// 导包
import com.march.ktexample.model.Student

// 导包时使用别名，然后就可以使用别名调用
import android.util.Log as L
L.e(TAG, str)
```

## 可见度修饰符
类，对象，接口，构造器，函数，属性以及属性访问器设值方法(访问器取值方法总与属性本身可见度相同，因此不需要控制其可见度)，都可以使用 **可见度修饰符** 。`Kotlin` 中的四种可见度修饰符：`private`、`protected`、`internal`、`public`，默认为 `public`。与 `Java` 不同的是少了 `default` 多了 `internal`。

注意：**局部变量， 局部函数，以及局部类，都不能指定可见度修饰符**。

### top-level

`top-level`， 像 `class` 这种可以直接声明在包下。在 `Kotlin` 中，函数, 属性, 类, 对象, 接口都可以声明为 `top-level`。比如扩展函数和扩展属性就是 声明为 `top-level` 的。注意，这里说的是 `top-level` 级别声明，在这些声明内部再声明别的函数或者属性，不在讨论范畴，将在下一部分说明，对于 `top-level` 中的声明来说：

> `public` ：意为该声明在任何位置都可以访问，public 是默认的，可以省略。If you do not specify any visibility modifier, public is used by default, which means that your declarations will be visible everywhere;    

> `private` ：意为该声明只能在同一个源代码文件中访问。If you mark a declaration private, it will only be visible inside the file containing the declaration;   

> `internal` ：意为该声明在同一个module的任意位置是可以访问的。If you mark it internal, it is visible everywhere in the same module;   

> `protected` ：对 `top-level` 的声明是无效的。protected is not available for top-level declarations.

```kotlin
package com.march.ktexample

// filename:VisibilityModifiersTest.kt

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

### 类与接口

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

### 构造器

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



## 内建数据类型
介绍 `Kotlin` 的内建数据类型，包括数值类型，字符类型，布尔类型，数组类型，字符串类型。
 
### 数值类型

`Kotlin` 数值使用内建对象表示，所有数值类型继承自 `Number` 抽象类，`Number` 中定义了数值在各个类型之间转换的方法。

 `Long` 类型需要在数字末尾追加 **大写** 的 `L` 表示，如 `123L`；`Float` 类型需要在数字末尾追加 `f／F` 表示，如 `123.5F`。默认十进制计数，如： `123`；十六进制以 `0x` 开头，如 `0x1FA`；二进制以 `0b` 开头，如 `0b001101`；**不支持** 八进制表示。
 
小数默认是 `Double` 类型，如果需要显式声明为 `Float`，需要加 `f/F` 后缀。
  
`Double` (64 bit) ；`Float` (32 bit) ；`Long` (64 bit) ；`Int` (32 bit) ；`Short` (16 bit) ；`Byte` (8 bit) ；
 
 
> 同一性：对象的类型，内容相同，使用 `===` 运算符比较为 `true`   

> 相等性：内容相等，使用 `==` 运算法比较为 `true`


- 数值装箱

`Kotlin` 中一切皆为对象，因此内建的数值类型也为对象，但是他们具有如同基本数据类型的特性，即 **同一性**，数值内建对象就如同 `Java` 中的基本数据类型，他们的值是不能为 `null` 的，你可以使用像 `Int?` 这样来进行数值装箱，这里可以参考 `Java` 中数值装箱的概念。装箱之后的数值会保持 **相等性** 但无法保持 **同一性**

```kotlin
// 内建对象的同一性
val aInt = 10
val bInt = 10
logError("${aInt === bInt}")// true
logError("${aInt == bInt}")// true

// 数值装箱后保持相等性，但是不保持同一性
val aOrigin:Int = 10000
val aBoxInt: Int? = aOrigin
val bBoxInt: Int? = aOrigin
logError("${aBoxInt === bBoxInt}")// false
logError("${aBoxInt == bBoxInt}")// true

困惑：
当aOrigin = 10时，aBoxInt === bBoxInt 为 true，
当aOrigin = 10000时，aBoxInt === bBoxInt 为 false
```

- 类型转换

`Kotlin` 不会将较小的数据类型隐式地转换为较大的数据类型. 也就是说, 如果不进行显式类型转换, 我们就不能将一个 `Byte` 类型值赋给一个 `Int` 类型的变量。你需要使用 `toInt()` 显式扩大为更大的类型， `Number` 类中定义了向各个数据类型转换的方式。

```kotlin
val bb : Byte = 1
val ll:Long = 1
// 扩大
val ii1:Int = bb.toInt()
// 缩小
val ii2:Int = ll.toInt()
```

- 数值运算符

```kotlin
shl(bits) – 带符号左移 (等于 Java 的<<)
shr(bits) – 带符号右移 (等于 Java 的 >>)
ushr(bits) – 无符号右移 (等于 Java 的 >>>)
and(bits) – 按位与(and)
or(bits) – 按位或(or)
xor(bits) – 按位异或(xor)
inv() – 按位取反
```


### 字符类型
`Char` 类型不是 `Number` 的子类，也就是说 `Char` 类型并不是数值类型。 `Char` 类型使用 单引号 如 `'a'` 表示。特殊字符使用反斜线转义表达. `Kotlin `支持的转义字符包括: `\t, \b, \n, \r, \', \", \\, \$` 。 其他任何字符, 都可以使用 `Unicode` 转义表达方式: `'\uFF00'`。

`Char` 类型也具有一系列的类型转换方法，你可以使用 `'0'.toInt()` 这样的语法将 `Char` 类型转换为数值类型。

可以使用 `Char?` 类型对 `Char` 类型进行装箱操作，与数值类型一样，装箱操作不能保持对象同一性。

### 布尔类型
`Boolean` 类型用来表示布尔值, 有两个可能的值: `true` 和 `false`.

当需要一个可为 `null` 的布尔值引用时, 布尔值也会被装箱(box).

布尔值的计算与 `Java` 相同。


### 数组类型
`Kotlin` 中的数组通过 `Array` 类表达, 这个类拥有 `get` 和 `set` 函数(这些函数通过运算符重载转换为 `[]` 运算符), 此外还有 `size` 属性, 以及其他一些有用的成员函数。

`Kotlin` 中也有专门的类来表达基本数据类型的数组: `ByteArray`, `ShortArray`, `IntArray` 等等, 这些数组可以避免数值对象装箱带来的性能损耗. 这些类与 `Array` 类之间不存在继承关系, 但它们的方法和属性是一致的. 各个基本数据类型的数组类都有对应的工厂函数。

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

### 字符串类型
使用双引号 `"abc"` 表示转移字符串，支持转义字符。使用三个双引号 `"""abc"""` 表示原生字符串，原生字符串不支持转义，可以包含任何转义字符。

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


## 控制流
控制流主要包括判断结构，分支结构，循环结构

### if 判断结构

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

### when 分支结构
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

### for 循环结构
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

### while 循环结构
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
