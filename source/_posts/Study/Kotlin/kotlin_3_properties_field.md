---
layout: post
title: Kotlin开发-3-属性与域
categories:
  - Kotlin
tags: Kotlin
keywords:
  - Kotlin
  - 类
  - 对象
  - 构造器
abbrlink: 12ba0671
date: 2017-06-08 00:00:00
---

本文主要介绍 `Kotlin` 属性和域的相关内容。


`Kotlin` 中，使用 `val` 声明常量(不可变)，使用 `var` 声明变量(可变)。

```kotlin
public val a = 100 // 常量，不可变，当你再想改变他的值时会报错
private var b = 1000 // 变量，可变
b = -100

val c:Int? = null
```

<!--more-->



## 属性的声明

声明属性的完整语法如下，其中的初始化器(`initializer`)， 取值方法(`getter`)， 以及设值方法(`setter`)都是可选的。`Kotlin` 具有类型自动推断得特性，如果属性类型可以通过初始化器自动推断得到， 或者可以通过这个属性覆盖的基类成员属性推断得到， 则属性类型的声明也可以省略。

标准的初始化方法，需要清楚其中几个概念

> - `propertyName` ：变量名
> - `PropertyType` ：变量类型，如果可以推断得到，可以省略
> - `property_initializer` ：初始化器，它可以是一个固定值或一个表达式
> - `getter` ：取值方法，访问器的一种
> - `setter` ：设值方法，访问器的一种

```kotlin
var <propertyName>: <PropertyType> [= <property_initializer>]
    [<getter>]
    [<setter>]
   
```

注意，如下注释 `1` 处，对于 `var` 变量来说，初始化器，设值，取值方法都是允许同时存在的；如下注释 `2` 处，对于 `val` 变量来说，不允许有设值方法，同时 `getter` 方法和 `初始化器` 不允许同时指定，因为只读变量只允许初始化一次。

使用 **注解** 和 **可见度修饰符**，如下注释 `3` 处，，如果不想在外界访问属性的 `set／get` 方法，可以使用可见度修饰符来避免外部访问。同样也可以在 `set／get` 方法前使用注解。


```kotlin
open class ClsA {
    // 1
    var x: Int = 100
        get() = 100
        set(value) {
            field = value
        }
        
    // 2. 不允许有设值方法，同时get方法和初始化器不允许同时指定，因为只读变量只允许初始化一次
    val y: Int
        get() = 100
    // 这是不允许的，因为val只读的
    // set(value) {
    //    field = value
    // }
    
    // 3. 在set/get方法前使用可见度修饰符和注解
    var z: Int = 100
    @Inject get
    private set
}

eg:
val param:ClsA = ClsA()
// 编译错误，因为不允许访问z的设值方法
param.z = 100
```

## Backing Fields  

`Kotlin` 我们可以直接使用属性名称对类的属性进行访问，但是实际上我们并没有直接访问该属性的引用，而是在编译时被转换成了 `getter/setter` 方法来进行访问，如注释中 `1` 处代码，这样我们既可以简化代码，又保持了类的封闭性。

`Kotlin` 的类中不允许拥有域(`Field`)，也就是说你在类内部也无法直接使用属性的名称对属性进行访问，而是都会转换为 `getter/getter` 方法。因此当你在 `set` 方法中调用该属性为他赋值时又会调用 `set` 方法，导致 **堆栈溢出**，如实例中 `2`，这是我在开始写的时候遇到的问题。

```kotlin
// 1 伪代码
ClsA().x = 100 相当于 ClsA().setX(100)
val a = ClsA.x 相当于 val a = ClsA().getX()

// 2
var xx: Int = 100
    set(value) {
    	// 再次调用 set 方法
        xx = value + 1
    }
```
但是自定义属性的访问器时不可避免的要使用 域变量，因此 `Kotlin` 提供了 `Backing Fields` 的特性，使用关键字 `field` 表示，`field` 标识符只允许在属性的访问器函数内使用，如下代码中，使用 `field` 即可真实的访问(不经过`getter/setter`) 变量 `xx`，完成赋值操作。

```kotlin
var xx: Int = 100
    get() = field++
    set(value) {
        field = value - 1
    }
```

## Backing Property

有时隐含的后端域属性不足以解决某些情况的问题，此时可以使用自定义的 `Backing Property`。如下实例中，`Backing Property` 就是另外定义一个属性 `_yy` 用来存储数据，而外界访问的属性 `yy` 只是提供了一个访问器方法而已，本身已经不具备什么意义啦，将访问器的声明和数据的存储放在两个地方，就不会发生之前的冲突（我的理解😊）

```kotlin
private var _yy: String? = null

var yy: String
    get() {
        if (_yy == null) {
            _yy = "init str"
        }
        return _yy ?: "be changed"
    }
    set(value) {
        _yy = "this is $value"
    }
    
// 访问yy
val param: ClsA = ClsA()
param.yy
```

## 编译期常数

如果属性值在编译期间就能确定, 则可以使用 `const` 修饰符, 将属性标记为 **编译期常数值(`compile time constants`)**。 这类属性必须满足以下所有条件:

> - 必须是顶级属性， 或者是一个 `object` 的成员。
> - 值被初始化为 `String` 类型, 或基本类型(`primitive type`)。
> - 不存在自定义的取值方法。

```kotlin
// 顶级属性
const val TEST_CONST = "TEST_CONST"

// 声明在 object 中
object Const{
    const val TEST_CONST_IN_OBJ = "TEST_CONST_IN_OBJ"
}
```

## 属性初始化 - 延迟(lateinit)

`lateinit` 关键字表示当前 变量 不会在声明时进行初始化操作，初始化操作会在后面进行，像是一种协议机制，告知编译器我会在后面使用该变量之前的恰当时机初始化该变量，不要进行警告⚠️。在一个 `lateinit` 属性被初始化之前访问它， 会抛出一个特别的异常，这个异常将会指明被访问的属性，以及它没有被初始化这一错误。

需要注意的是使用 `lateinit` 关键字有很多限制:

> - 必须是变量，即使用 `var` 关键字进行声明。   
> - 不能修饰可为 `null` 的类型，比如 `lateinit var str:String?` 是编译不通过的。   
> - 不能修饰基本数据类型。例如 `Int`,`Float`等

```kotlin
lateinit var stuLateInit:Student

stuLateInit = Student("name",11)
```


## 属性延迟初始化 - 代理(by lazy)

`by lazy` 是属性代理的基本运用，是经过简化后的属性代理，他为属性提供初始化方法。这里不扩展属性代理的相关问题。

因为他只提供取值方法所以仅可以用在常量的初始化中，使用 `by lazy` 当属性被第一次访问时，就会触发初始化流程。

> - 只有常量，也就是使用 `val`才能使用 `by lazy` 延迟初始化

```kotlin
val stuByLazy:Student by lazy {
    Student("name",11)
}

val mMyMsgTv:TextView by lazy {
    findViewById(R.id.mTestTv) as TextView
}

// 当常量被使用时才进行初始化
logError(stuByLazy.myCls)
mMyMsgTv.text="文本"
```