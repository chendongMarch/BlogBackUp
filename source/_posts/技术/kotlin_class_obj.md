---
layout: post
title: Kotlin-类与对象篇
date: 2017-06-08
category: 技术
tags: kotlin
keywords: [kotlin,类,对象,构造器]
---

本文主要介绍 `Kotlin` 类与对象。

看了官方文档的相关描述，发现很多名词在 `Kotlin` 中和 `Java` 都有不太一样，在 `Kotlin` 中 `方法` 被称为 `函数`，而像 `open` , `final` 这种关键字被称为注解(`annotation`)，在我之前的认知当中只有 `@Inject` 这种才是注解。因此文章中的描述都尽量使用 `Kotlin` 中的术语。

<!--more-->

## 属性与域
`Kotlin` 中，使用 `val` 声明常量(不可变)，使用 `var` 声明变量(可变)。

```kotlin
public val a = 100 // 常量，不可变，当你再想改变他的值时会报错
private var b = 1000 // 变量，可变
b = -100

val c:Int? = null
```

### 属性的声明

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

注意，对于 `var` 变量来说，初始化器，设值，取值方法都是允许同时存在的，如下 `1`，对于 `val` 变量来说，不允许有设值方法，同时 `getter` 方法和 `初始化器` 不允许同时指定，如下 `2`，因为只读变量只允许初始化一次。

使用 **注解** 和 **可见度修饰符**，如下实例中国年 `3`，如果不想在外界访问属性的 `set／get` 方法，可以使用可见度修饰符来避免外部访问。同样也可以在 `set／get` 方法前使用注解。


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

### 后端域变量和后端属性	
- 后端域变量

`Kotlin` 的类中不允许具有 `backing fields`，即 `后端域变量`。根本原因在于 `Kotlin` 中当你使用属性名直接方法该属性时会调用 `get/set` 方法。如下实例中中的 `1`。因此当你在 `set` 方法中调用该属性为他赋值时又会调用 `set` 方法，导致堆栈溢出，如实例中 `2`，这是我在开始写的时候遇到的问题。

```kotlin
// 1 伪代码
ClsA().x = 100 == ClsA().setX()
val a = ClsA.x == val a = ClsA().getX()

// 2
var xx: Int = 100
    set(value) {
    	// 再次调用 set 方法
        xx = value + 1
    }
```
但是自定义属性的访问器时不可避免的要使用 域变量，因此 `Kotlin` 提供了一种自动的后端域变量，使用关键字 `field` 表示，`field` 标识符只允许在属性的访问器函数内使用，如下：

```kotlin
var xx: Int = 100
    get() = field++
    set(value) {
        field = value - 1
    }
```
- 后端属性	

有时隐含的后端域属性不能解决某些情况的问题，此时可以使用自定义的后端属性。如下实例中，后端属性就是另外定义一个属性 `_yy` 用来存储数据，而外界访问的属性 `yy` 只是提供了一个访问器方法而已，本身已经不具备什么意义啦，将访问器的声明和数据的存储放在两个地方，就不会发生之前的冲突（我的理解😊）

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

### 编译期常数
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

### 属性的延迟初始化

`lateinit` 关键字表示当前 变量 不会在声明时进行初始化操作，初始化操作会在后面进行，像是一种协议机制，告知编译器我会在后面使用该变量之前的恰当时机初始化该变量，不要进行警告⚠️。在一个 `lateinit` 属性被初始化之前访问它， 会抛出一个特别的异常，这个异常将会指明被访问的属性，以及它没有被初始化这一错误。

需要注意的是使用 `lateinit` 关键字有很多限制:

> - 必须是变量，即使用 `var` 关键字进行声明。   
> - 不能修饰可为 `null` 的类型，比如 `lateinit var str:String?` 是编译不通过的。   
> - 不能修饰基本数据类型。例如 `Int`,`Float`等

```kotlin
lateinit var stuLateInit:Student

stuLateInit = Student("name",11)
```


## 类的构造器

`Kotlin` 中的类可以有一个 **主构造器** (`primary constructor`), 以及一个或多个 **次构造器** (`secondary constructor`)。主构造器是类头部的一部分, 位于类名称(以及可选的类型参数)之后。

### 主构造器

主构造器紧跟在类名后面进行声明，声明的一般形式如下，可见度修饰符默认为 `public`，(如果主构造器是 `public` 的，同时没有注解那么可以省略 `constructor` 关键字)。当然如果类内部没有具体的实现，`{}` 也是可以省略的。

```kotlin
// 完整的写法
class Engineer private @Inject constructor(name: String, age: Int, language: String)

// 简化写法，由于是public的，同时没有注解，省略了constructor关键字
class Engineer (name: String, age: Int, language: String)

// 默认生成一个public的无参构造器
class Engineer
```

由于主构造器中无法使用初始化代码，可以在类内进行属性的初始化，使用 `init{}` 关键字表示。

```kotlin
class Engineer (name: String, age: Int, language: String) {
    var name: String = ""
    var age:Int = 0
    var language:String = ""

    init {
        this.name = name
        this.age = age
        this.language = language
        log("name = $name,age = $age, lan = $language")
    }
}
```
在主构造器内可以直接使用 `val` 和 `var` 声明和初始化属性，他们都是类的成员属性。

```kotlin
class Engineer(val name: String = "", age: Int = 0, language: String = "") {

}
```
> 注意: 在 JVM 中, 如果主构造器的所有参数都指定了默认值, 编译器将会产生一个额外的无参数构造器, 这个无参数构造器会使用默认参数值来调用既有的构造器. 有些库(比如 Jackson 或 JPA) 会使用无参数构造器来创建对象实例, 这个特性将使得 Kotlin 比较容易与这种库协同工作.


### 次级构造器
如果类有主构造器, 那么每个次级构造器都必须委托给主构造器, 要么直接委托, 要么通过其他次级构造器间接委托. 委托到同一个类的另一个构造器时, 使用 `this` 关键字实现。

```kotlin
class Engineer(val name: String = "", age: Int = 0, language: String = "") {
    constructor(name: String, age: Int) : this(name, age, "en"){
        log("次级构造器")
    }
}
```

### 创建对象
构造器中的属性指定初始值后在创建对象时可以省略，使用如下方法，避免重载构造方法，主构造器中具有默认值的属性不是必须赋值的。

```kotlin
val en = Engineer("",12)
val en1 = Engineer(name = "test")
val en2 = Engineer(name = "test",age = 12,language = "")
```

	
## 继承
`kotlin` 中所有的类默认继承自超类 `Any`,`Any` 不是 `java.lang.Object`; 尤其要注意, 除 `equals()` , `hashCode()` 和 `toString()` 之外, 它没有任何成员。  `Kotlin` 中所有的类都是 `fnial` 的，也就是不允许继承，这点和 `Java` 正好相反，如果你想继承一个类必须使用 `open` 关键字声明。 

`Kotlin` 使用 `:` 表示继承，子类和父类构造器调用分为以下两种情况

- 如果类有主构造器, 那么必须在主构造器中使用主构造器的参数来初始化基类。

```kotlin
open class Engineer(var name: String = "", var age: Int = 0, var language: String = "") {
}

class BigEngineer(var level: Int) : Engineer("name",12) {
}
```

- 如果类没有主构造器, 那么所有的次级构造器都必须使用 `super` 关键字来初始化基类, 或者委托到另一个构造器, 由被委托的构造器来初始化基类。 注意, 这种情况下, 不同的次级构造器可以调用基类中不同的构造器。

```kotlin
open class Engineer(var name: String = "", var age: Int = 0, var language: String = "") {
    
}

class BigEngineer : Engineer {
    var level = 0
    constructor(name:String,age:Int,level:Int):super(name,age,"en"){
        this.level = level
    }
}
```
### 方法的覆盖
和类一样，类中的所有方法都默认是 `final` 的，即不允许在子类中进行重写，如果想要在子类中更改，需要使用 `open` 注解声明，如实例中 `3`。

如果子类想要覆盖父类的方法，必须添加 `override` 注解。 如果遗漏了这个注解，编译器将会报告错误。 如果一个函数没有标注 `open` 注解， 那么在子类中声明一个同名同参的方法将是非法的, 无论是否添加 `override` 注解， 都不可以，如实例中 `4`。 

当一个子类成员标记了 `override` 注解来覆盖父类成员时， 覆盖后的子类成员本身也将是 `open` 的, 也就是说， 子类成员可以被自己的子类再次覆盖，如实例中 `1`。 如果你希望禁止这种再次覆盖，可以使用 `final` 注解，如实例中 `2`。

实例：

```kotlin
open class Engineer(var name: String = "", var age: Int = 0, var language: String = "") {
    open fun test() {}
    open fun test2() {}
    // 此函数不允许在子类覆盖
    fun test3() {}
}

open class BigEngineer(name: String = "", age: Int = 0, var level: Int = 0) 
    : Engineer(name, age, "en") {
    
    // 1. 此函数覆盖了超类中的函数，他将默认是open的，他的子类可以覆盖该方法。
    override fun test() {}

    // 2. 此函数在子类中将不会被覆盖，因为显式的添加了final注解
    final override fun test2() {}

    // 3. 这句是编译错误的，因为test3()不是open的
    // override fun test3() {} 
    
    //  4. 这句是编译错误的，超类中test3()是final的，在子类中声明同名同参的函数是非法的
    // fun test3(){}
}
```

在一个 `final` 类(比如， 一个没有添加 `open` 注解的类)中， 声明 `open` 成员是没有意义的。

```kotlin
class Test{
    // 警告⚠️open has no effect in a final class。
    // 在一个final类中声明open成员是无意义的，因为类无法被继承，成员函数和变量也无法被重写。
    open fun test(){}
    open val a = 0
}
```

### 属性的覆盖
前提，属性的覆盖方式与方法覆盖类似，同样必须在父类中声明为 `open`，当超类中声明的属性在后代类中再次声明时，必须使用 `override` 关键字来标记，而且覆盖后的属性数据类型必须与超类中的属性数据类型兼容。 

可以使用带初始化器的属性来覆盖超类属性， 也可以使用带取值方法(`getter`)的属性来覆盖。

可以在主构造器的属性声明中使用 `override` 关键字，覆盖主构造器中的属性。

可以使用一个 `var` 属性覆盖一个 `val` 属性，但不可以反过来使用一个 `val` 属性覆盖一个 `var` 属性。 允许这种覆盖的原因是， `val` 属性本质上只是定义了一个 `get` 方法，使用 `var` 属性来覆盖它， 只是向后代类中添加了一个 `set` 方法。


```kotlin
// name属性是open的，它可以在子类中被覆盖
open class Engineer(open var name: String = "", var age: Int = 0, var language: String = "") {
    
    open val a: Int
        get() = 100
        
    open val b: Int = 100

    open var c:Int = 100
}

// 覆盖了超类中的name属性
open class BigEngineer(override var name: String = "", age: Int = 0, var level: Int = 0)
    : Engineer(name, age, "en") {
    
    // 可以使用初始化器或getter方法对属性进行覆盖，都是允许的。
    override val a: Int = 100
    
    override var b = 0
        get() = 1
        
    // 这句是编译错误的，因为val变量不能覆盖var变量
    // override val c:Int = 1
}
```

### 覆盖的原则

在 `Kotlin` 中， 类继承中的方法实现问题, 遵守以下规则: 如果一个类从它的直接超类中继承了同一个成员的多个实现，那么这个子类必须覆盖这个成员，并提供一个自己的实现，这样在子类中消除歧，为了表示使用的方法是从哪个超类继承得到的， 我们使用 `super` 关键字, 将超类名称放在尖括号类, 比如 `super<父类>`。

```kotlin

open class ClsA {
    open val a = 100
    open fun test() {}
}

interface ClsB {
    val a: Int
    fun test() {}
}

class ClsC : ClsA(), ClsB {
    // 属性的覆盖不存在这样的问题
    override val a = 1000
    
    // 方法的覆盖
    // 此时编译器强制必须实现test()方法,因为可以从超类中继承多个实现
    override fun test() {
        // 明确的调用父类中的方法
        super<ClsA>.test()
        super<ClsB>.test()
    }
}
```
