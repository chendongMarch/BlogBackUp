---
layout: post
title: Kotlin开发-4-类与对象
categories:
  - Kotlin
tags: Kotlin
keywords:
  - Kotlin
  - 类
  - 对象
  - 构造器
abbrlink: 2717980190
date: 2017-06-08 00:00:00
---

本文主要介绍 `Kotlin` 类与对象。

看了官方文档的相关描述，发现很多名词在 `Kotlin` 中和 `Java` 都有不太一样，在 `Kotlin` 中 `方法` 被称为 `函数`，而像 `open` , `final` 这种关键字被称为注解(`annotation`)，在我之前的认知当中只有 `@Inject` 这种才是注解。因此文章中的描述都尽量使用 `Kotlin` 中的术语。

<!--more-->

## 接口

接口使用 `interface` 关键字声明；
>- 接口中可以包含抽象方法的声明，也可以包含方法的实现。
>- 接口与抽象类的区别在于， 接口不能存储状态数据。 
>- 接口可以有属性， 但这些属性必须是抽象的， 或者必须提供访问器的自定义实现。
>- 接口不支持 `Backing Field`，因此 `var` 变量无法定义访问器

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

### 实现接口

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

### 解决接口覆盖冲突

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
