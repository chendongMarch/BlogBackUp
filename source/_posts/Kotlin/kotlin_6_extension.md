---
layout: post
title: Kotlin开发-6-扩展
categories:
  - Kotlin
tags:
  - Kotlin
keywords:
  - Kotlin
  - 类
  - 对象
  - 扩展函数
  - 扩展属性
  - Extension
abbrlink: d0378250
date: 2017-07-20 08:57:00

---

本文主要介绍 `Kotlin` 扩展 ( `Extension` )的相关内容。

以下翻译自官方文档：

> 与 `C#` 和 `Gosu` 类似， `Kotlin` 提供了向一个类扩展新功能的能力， 而且不必从这个类继承，也不必使用任何设计模式， 比如 `Decorator` 模式之类。 这种功能是通过一种特殊的声明来实现的， `Kotlin` 中称为 扩展(`extension`)。`Kotlin` 支持 扩展函数(`extension function`) 和 扩展属性(`extension property`)。

**扩展** 是很多高级语言都具备的特性，使用扩展可以在不侵入原来类的基础上扩展新的功能，比如我们常用的 `Utils` 方法，就可以完全使用扩展方法来替代。
<!--more-->
 
## 扩展函数
扩展函数需要声明在`top-level` 级别下，如果声明在一个类里面，那么只能在这个类内使用，也就失去了扩展的意义。我开始写的时候因为写 `java` 的惯性，就声明在类里面，结果别的地方根本用不了，一度十分怀疑扩展存在的意义。。。

声明的方式与普通函数稍有不同，在函数名前面要使用扩展的接收者，扩展的接收者就是我们要扩展的对象，如下：

```kotlin
fun 扩展接收者.函数名:返回类型(参数列表){
	函数体
}
```

先来声明两个扩展函数看看效果

```kotlin
// ExtensionTest.kt

fun Context.extensionLog(msg: String) {
    Log.e(this.javaClass.simpleName, msg)
}

fun <E>MutableList<E>.swap(index1:Int,index2:Int){
    val temp = this[index1]
    this[index1] = this[index2]
    this[index2] = temp
}
```

在 `MainActivity.kt` 中使用它们

```kotlin
extensionLog("测试扩展函数")

val list = mutableListOf("str1","str2")
extensionLog(list.toString())
list.swap(0,1)
extensionLog(list.toString())
```
输出结果：

```bash
com.march.ktexample E/MainActivity: 测试扩展函数
com.march.ktexample E/MainActivity: [str1, str2]
com.march.ktexample E/MainActivity: [str2, str1]
```

总结一下：

1. 扩展函数的声明对子类是有效的， 比如我们给 `Context` 类扩展，那么 `Context` 所有子类都可以使用扩展函数，扩展虽然不是继承，但是却具有和继承类似的效果。

2. 在扩展函数中可以使用 `this` 关键字，访问当前扩展接收者对象。

## 扩展函数的静态解析

扩展函数并不是真的给扩展接收者的类增加了新的方法，他只是创建了一个新的函数，并且可以让扩展接收者的类使用点号的形式去调用而已，对原来的类本身没有任何影响。

扩展函数的调用派发过程是静态的，这就意味着，调用扩展函数时，具体被调用的函数是哪一个，是通过调用函数的对象表达式的类型来决定的，而不是在运行时刻表达式动态计算的最终结果类型决定的。也就是说如果有多个扩展调用，那么调用哪一个取决于你使用哪种类型去调用。

```kotlin
open class A

class B : A()

fun A.extension(): String {
    return "a的扩展"
}
fun B.extension(): String {
    return "b的扩展"
}

// 一个普通的函数，扩展函数的实际调用由调用他的类决定
// 因此这里不管传什么类型的值，都会是调用 A 的扩展函数
fun printExtension(a:A){
    extensionLog(a.extension())
}

// 调用 A 的扩展
extensionLog(A().extension())
// 调用 B 的扩展
extensionLog(B().extension())
// 调用 A 的扩展
printExtension(A())
// 调用 A 的扩展
printExtension(B())
```
输出结果

```bash
com.march.ktexample E/MainActivity: a的扩展
com.march.ktexample E/MainActivity: b的扩展
com.march.ktexample E/MainActivity: a的扩展
com.march.ktexample E/MainActivity: a的扩展
```

## 成员函数优先

当扩展函数和成员函数都存在时，相同的函数是说函数名与参数列表都相同，优先使用成员函数。

```kotlin
// B 有成员函数
class B {
    fun extension(): String {
        return "b的成员"
    }
}

// B 有扩展函数
fun B.extension(): String {
    return "b的扩展"
}

// 将优先使用成员函数
fun printExtension(b:B){
    extensionLog(b.extension())
}

printExtension(B())

结果：
com.march.ktexample E/MainActivity: b的成员
```

## 可为空的接收者
使用可为空的类型作为扩展接收者时，即使调用的对象为空仍然可以调用，不会空指针，可以在扩展方法内检测接受者是不是为空。

```kotlin
// 可为空的接收者
fun String?.testNullableReceiver():String{
    if(this == null) return "I am null"
    return "I am not null ${toString()}"
}
```
再看一下，可空类型的 `toString()` 方法

```kotlin
fun Any?.toString(): String {
    if (this == null) return "null"
    // 进行过 null 检查后, 'this' 会被自动转换为非 null 类型, 因此下面的 toString() 方法
    // 会被解析为 Any 类的成员函数
    return toString()
}
```

测试

```kotlin
val strNull1:String? = null
val strNull2:String? = "strNull2 "
val strNoNull:String = "strNoNull "
extensionLog(strNull1.testNullableReceiver())
extensionLog(strNull2.testNullableReceiver())
extensionLog(strNoNull.testNullableReceiver())

// 结果
com.march.ktexample E/MainActivity: I am null
com.march.ktexample E/MainActivity: I am not null strNull2 
com.march.ktexample E/MainActivity: I am not null strNoNull 
```

## 扩展属性

扩展属性并不真的是类的成员，他不能更改类的内容，因此扩展属性不支持 `backing-field`，因此没办法使用 `set()` 为属性赋值。

扩展属性不能使用 **初始化器** 为属性赋值

```kotlin
var Context.extensionProperty: String
    get() {
        return "${javaClass.simpleName} properties = "
    }
    // ⚠️ 这样是错误的，没有backing-field支持，这里将会StackOverflow
    set(value) {
        extensionProperty = value
    }

// 错误，不允许初始化器
val Context.testValue1:Int = 100

// 一个正确的示范
val <T> List<T>.lastIndex: Int
    get() = size - 1
```

## 扩展同伴对象

```kotlin
class MainActivity : Activity() {

    companion object MyCompanion {

    }
}

// 扩展函数
fun MainActivity.MyCompanion.testCompanionFunc(): String {
    return "testCompanionFunc"
}

// 扩展属性
val MainActivity.MyCompanion.testCompanionProperty
    get() = "testCompanionProperty"
    
    
extensionLog(MainActivity.MyCompanion.testCompanionFunc())
extensionLog(MainActivity.MyCompanion.testCompanionProperty)
```


## 声明扩展为成员

以下翻译自官方文档：在类的内部，你可以为另一个类定义扩展。在这类扩展中，存在多个 **隐含接受者(`implicit receiver`)** - 这些隐含接收者的成员可以不使用限定符直接访问。扩展方法的定义所在的类的实例，称为 `派发接受者(dispatch receiver)`， 扩展方法的目标类型的实例，称为 `扩展接受者(extension receiver)`。

也就是说我们可以在一个类 BB 内部声明另一个类 AA 的扩展，AA 是扩展接受者，BB 是派发接受者，此时存在多个接受者，在 AA 的扩展方法中，可以自由的不使用限定符的访问 AA 和 BB 的成员。

```kotlin
class AA {
    fun funcInAA(): String {
        return "func in AA"
    }
}

class BB {
    fun funcInBB(): String {
        return "func in BB"
    }
    fun AA.extendAAFuncInBB(): String {
        return "extendAAFuncInBB" + funcInBB() + funcInAA()
    }
    fun testFunc(aa: AA): String {
        return aa.extendAAFuncInBB()
    }
}
```
当派发接受者与扩展接受者的成员名称发生冲突时，扩展接受者的成员将会被优先使用。 如果想要使用派发接受者的成员，需要使用 `this` 关键字

```kotlin
class BB {
 
    fun AA.testToString(){
        // 默认使用 AA.toString()
        toString()
        // 使用 BB.toString()
        this@BB.toString()
    }
}
```

扩展函数可以在子类中被覆盖，这类扩展函数在派发过程中针对派发接受者是虚拟的，针对扩展接受者是静态的。

前面说过，扩展的解析是静态的，使用哪个扩展取决于调用他的是哪个类型的类对象，但是派发接受者是虚拟的是说，他将使用子类中更具体的实现。

看一个官网的例子，下面的 `caller(d:D)` 方法接收的是 D 类型的对象，此时调用扩展函数时，不管你传递什么类型的值(D 或 D 的子类对象)，都会调用 D 的扩展函数，因为扩展接受者的解析是静态的， 所以所有的打印结果都是 `D.foo` ；但是派发接受者的解析是虚拟的，所以 `C1().caller()` 将使用子类中更具体的实现，C1 对扩展函数进行了重载，因此会使用 C1 重载后的 `caller()` 打印出 `in C1`。

```kotlin
open class D {
}

class D1 : D() {
}

open class C {
    open fun D.foo() {
        println("D.foo in C")
    }

    open fun D1.foo() {
        println("D1.foo in C")
    }

    fun caller(d: D) {
        d.foo()   // 调用扩展函数
    }
}

class C1 : C() {
    override fun D.foo() {
        println("D.foo in C1")
    }

    override fun D1.foo() {
        println("D1.foo in C1")
    }
}

C().caller(D())   // 打印结果为 "D.foo in C"
C1().caller(D())  // 打印结果为 "D.foo in C1" - 派发接受者的解析过程是虚拟的
C().caller(D1())  // 打印结果为 "D.foo in C" - 扩展接受者的解析过程是静态的
```

## 总结

扩展声明在 `top-level` 才能在所有地方都可以使用。

扩展函数的声明对子类是有效的，我们扩展父类，子类也会有这些扩展。

在扩展函数中可以使用 `this` 关键字，访问当前扩展接收者对象。

扩展的的接受者可以为空，不会 `NPE`。

成员函数和扩展函数冲突时，成员函数优先。

扩展接受者的解析是静态的，将会使用哪个扩展取决于调用的类的类型，也就是说类型确定了，使用的方法就确定了，并不在乎实际上是什么类型，比如我参数是 `d:D`，就决定了是 `D` 类型的扩展方法，传它的子类 `D1:D` 类型也不会改变。

派发接受者的解析是虚拟的，将会使用哪个扩展取决于调用的类的实际的类型，将会使用当前实际类型重载的方法，比如我参数是  `d:D`，但我传它的子类 `D1:D` 类型则会使用 `D1` 中对扩展的重载。