---
layout: post
title: Kotlin开发-10-代理
categories:
  - Kotlin
tags:
  - Kotlin
keywords:
  - Kotlin
  - delegate
  - 代理
  - 委托
abbrlink: cc5968a9
date: 2017-07-29 18:25:00

---

本文主要学习 `Kotlin` 中的代理模式的实现以及属性代理的使用。

代理模式已被实践证明为类继承模式之外的另一种很好的替代方案，`Kotlin` 对代理模式做了很好的支持，我们可以非常简单的实现代理模式。

属性代理使得我们可以将具有共同性质的属性的初始化通过属性代理来实现，简化了数据的读取和存储，同时在  `Kotlin` 中还内置了很多标准代理，可以大大简化开发过程。

最后我们使用属性代理来实现一个完全的封闭的访问 `SharePreference` 进行数据存取的例子。

<!--more-->

## 代理模式

代理 (Delegate) 模式是一种常用的设计模式，`Kotlin` 对此作了很好的支持。

代理模式已被实践证明为类继承模式之外的另一种很好的替代方案，当我们已经抽象了部分功能出来，又想当前的对象具备这些功能，除了继承，我们可以使用代理模式，将当前对象需要的操作代理给别人做，实现复用。

使用官网的一个例子，并且进行了部分扩展，来理解代理模式的意义。

假设 `BaseA` 声明了 A 类逻辑，`BaseAImpl` 实现了这部分逻辑，`BaseB` 声明了 B 类逻辑，`BaseBImpl` 实现了这部分逻辑。此时我们的 `Derived` 需要同时具备 `BaseA` 和 `BaseB` 的功能，我们可以使用 `Derived` 再次实现 `BaseA` 和 `BaseB` 的相关功能但我们将无法复用在 `BaseAImpl` 和 `BaseBImpl` 中已经实现的逻辑代码。

使用代理模式，我们将 `Derived` 需要实现的功能分别代理给 `BaseA` 和 `BaseB` 的实现类，`Derived` 自己不做任何处理，使用代理去接收处理。事实上，我们会在 `Derived` 类中持有代理的对象，然后在对象的接口方法逻辑实现时转发给代理的相关方法，实现代理模式需要设计很多相关的接口代码，好在 `Kotlin` 已经为我们提供了代理模式的支持，我们只需要使用 `by` 关键字将功能的实现代理给指定对象即可。

```kotlin
interface BaseA {
    fun printA()
}

interface BaseB {
    fun printB()
}

class BaseAImpl(var x: Int) : BaseA {
    override fun printA() {
        log("BaseAImpl $x")
    }
}

class BaseBImpl(var x: Int) : BaseB {
    override fun printB() {
        log("BaseBImpl $x")
    }
}

// 将 BaseA 的逻辑代理给 ba, 将 BaseB 的逻辑代理给 bb
class Derived(ba: BaseA, bb: BaseB) : User(), BaseA by ba, BaseB by bb
```


```bash
val derived = Derived(BaseAImpl(10), BaseBImpl(100))
derived.printA()
derived.printB()

# 输出
com.march.ktexample E/BaseAImpl: BaseAImpl 10
com.march.ktexample E/BaseBImpl: BaseBImpl 100
```

## 属性代理

我们可以使用代理属性的功能来初始化那些具有共性的属性，比如从 `SharePerenferce` 或者数据库中读取和存储，我们创建一个属性代理使用它来初始化这些具有共性的属性将会简单很多。

### 一个简单的代理实现

使用一个简单的代理来初始化 `value` 属性

```kotlin
var value: String by MyDelegate()

class MyDelegate {

    var temp = "test"

    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        log(thisRef.toString())
        log(property.toString())
        log("invoke getValue")
        return temp
    }
    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        log("invoke setValue value is $value")
        temp = value
    }
}
```
当我们读或写 `value` 属性时将会触发对应的代理方法

```kotlin
log(value)
value = "123"
log(value)
```

```bash
// thisRef 的值
MyDelegate: com.march.ktexample.MainActivity@445adf9
// property 的值，操作的是 value 属性
MyDelegate: property value (Kotlin reflection is not available)

MyDelegate: invoke getValue
MainActivity: test
MyDelegate: invoke setValue value is 123
MyDelegate: invoke getValue
MainActivity: 123
```

代理属性的语法如下：

`by` 关键字后面的表达式就是代理，他需要具有 `getValue()` 方法， 如果属性是  `var` 类型，那么还必须有 `setValue()` 方法。

```kotlin
val/var <propertyName>:<propertyType> by <expression>
```

### getValue() 和 setValue()

代理中的 `getValue()` 和 `setValue()` 方法没有类似接口的那种强制模板的实现，可能没有办法像接口那样声明统一的接口方法。

`getValue()` 和 `setValue()` 函数需要使用 `operator` 关键字标记，它们可以是委托类的成员函数，也可以是它的扩展函数。如果你需要将属性委托给一个对象，而这个对象本来没有提供这些函数，这时使用扩展函数会更便利一些。

参数列表和返回类型：

参数 `thisRef:Any?` 的类型必须与 **属性所属的类** 相同或者是它的基类(对于扩展属性 — 这个参数的类型必须与被扩展的类型相同或者是它的基类)。

参数 `property: KProperty<*>` 是被代理的属性，可以使用 `property.name` 查看属性名，这个参数的类型必须是 `KProperty<*>` 或者是它的基类。

对于 `getValue()` 函数来说返回值必须与被代理的属性的类型相同。

对于 `setValue()` 函数来说参数 `value: Type` 是属性的新值，他必须与属性的类型相同。另外需要将值进行存储，在 `getValue()` 函数中返回，否则将无法改变属性的值。

```kotlin
operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
      return temp
}
operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
    temp = value
}
```

## Standard Delegates

`Kotlin` 内置了一些标准代理。

### Lazy

`by lazy` 是属性代理在 `kotlin` 中的一个简化实现，使用 `by lazy` 可以延迟属性的加载，他接受一个 `Lambda` 表达式作为参数，返回一个 `Lazy<T>` 类型的实例。

> 只有 `val` 属性才能使用 `by lazy` 延迟初始化，因为 `by lazy` 只提供 `getValue()` 方法。

```kotlin
@kotlin.jvm.JvmVersion
public fun <T> lazy(initializer: () -> T): Lazy<T> 
= SynchronizedLazyImpl(initializer)
```
使用 `by lazy` 只有在第一次获取值的时候回去计算 `Lambda` 表达式的值，后面再次访问，只会返回之前的值，不会再做计算。

```kotlin
val lazyValue: Int by lazy {
    log("invoke by lazy")
    100
}

// 测试
log(lazyValue)
log(lazyValue)

// 结果
com.march.ktexample E/MainActivity: invoke by lazy
com.march.ktexample E/MainActivity: 100
com.march.ktexample E/MainActivity: 100
```
默认情况下，`by lazy` 的计算是 **同步的(synchronized)**: 属性值只会在唯一一个线程内计算，然后所有线程都将得到同样的属性值。如果委托的初始化计算不需要同步，多个线程可以同时执行初始化计算，那么可以使用`LazyThreadSafetyMode.PUBLICATION` 参数。相反，如果你确信初期化计算只可能发生在一个线程内，那么可以使用 `LazyThreadSafetyMode.NONE` 模式，这种模式不会保持线程同步，因此不会带来这方面的性能损失。


### Observable & Vetoable

可观察属性，`Delegates.observable()` 函数接受一个初始化值和一个属性值变化的响应器作为参数，可以用来观察属性的变化，当属性的值发生变化时会触发响应器。

```kotlin
var observableValue: Int by Delegates.observable(1) {
    property, oldValue, newValue ->
    log("Delegates.observable - ${property.name} $oldValue $newValue")
}

// 测试
observableValue = 100
observableValue = 200

// 结果
MainActivity: Delegates.observable - observableValue 1 100
MainActivity: Delegates.observable - observableValue 100 200
```
如果想要在检测到属性变化时，截断赋值操作，需要使用 `Delegates.vetoable()`，如下，值等于 200 时，不做赋值操作。

```kotlin
var vetoableValue: Int by Delegates.vetoable(1) {
    property, oldValue, newValue ->
    log("Delegates.vetoable - ${property.name} $oldValue $newValue")
    newValue != 200
}

// 测试
vetoableValue = 100
log("(after set 100) $vetoableValue")
vetoableValue = 200
log("(after set 200) $vetoableValue")

// 结果
MainActivity: Delegates.vetoable - vetoableValue 1 100
MainActivity: (after set 100) value is 100
MainActivity: Delegates.vetoable - vetoableValue 100 200
MainActivity: (after set 200) value is 100
```

### NotNull

使用非空代理 `Delegates.notNull()`，属性在声明时必须进行初始化，避免空指针的出现，`Delegates.notNull()` 类似于 `lateinit` 关键字，使你可以在后面初始化属性的值，在没有初始化之前如果使用了该属性，则会抛出异常 `java.lang.IllegalStateException: Property notNullValue should be initialized before get.`

```kotlin
var notNullValue: String by Delegates.notNull<String>()
```

### Map

使用 `Map` 存储属性的值，就可以从 `Map` 中获取对应属性的名称为 `key` 的值。

```kotlin
class MapUser(propertyMap: Map<String, Any?>) {
    val name by propertyMap
    val age by propertyMap
}
```
使用  `Map` 初始化一个对象

```kotlin
val mapUser = MapUser(mapOf(
        "name" to "myName",
        "age" to 25
))
log("name = ${mapUser.name} ,age = ${mapUser.age}")

// 结果为
MainActivity: name = myName ,age = 25
```


## SharedPreferences 读写代理

一个代理属性的例子。

以属性名为 `key` 从 `SharedPreferences` 读取数据，用来初始化属性的值，存储数据时同时写入 `SharedPreferences`；

```kotlin
// 自定义 SharedPreferences 属性读写
class SpDelegate<T>(val context: Context, val readName: String?, val defValue: T) 
    : ReadWriteProperty<Any?, T> {
    
    constructor(context: Context, defValue: T) : this(context, null, defValue)
    
    val KEY = "SharePreference"
    
    val mSharePreference: SharedPreferences by lazy {
        context.getSharedPreferences(KEY, Context.MODE_PRIVATE)
    }
    
    @Suppress("UNCHECKED_CAST")
    override fun getValue(thisRef: Any?, property: KProperty<*>): T {
        val name = readName ?: property.name
        val result: Any = when (defValue) {
            is Boolean -> mSharePreference.getBoolean(name, defValue)
            is Int -> mSharePreference.getInt(name, defValue)
            is Long -> mSharePreference.getLong(name, defValue)
            is Float -> mSharePreference.getFloat(name, defValue)
            is String -> mSharePreference.getString(name, defValue)
            else -> throw IllegalArgumentException("un support")
        }
        log("getValue : result = " + result)
        return result as T
    }
    
    override fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
        val editor = mSharePreference.edit()
        val name = readName ?: property.name
        when (value) {
            is Boolean -> editor.putBoolean(name, value)
            is Int -> editor.putInt(name, value)
            is Long -> editor.putLong(name, value)
            is Float -> editor.putFloat(name, value)
            is String -> editor.putString(name, value)
            else -> throw IllegalArgumentException("un support")
        }
        log("setValue : newValue = " + value)
        editor.apply()
    }
}
```
测试属性的读取和写入

```kotlin
var shareIntVar1: Int by SpDelegate(this, 1)
var shareStringVar1: String by SpDelegate(this, "def String value")

log(shareIntVar1)
shareIntVar1 = 1000
log(shareIntVar1)
log(shareStringVar1)
shareStringVar1 = "new String value"
log(shareStringVar1)
```

结果

```bash
SpDelegate: getValue : result = 1
MainActivity: 1
SpDelegate: setValue : newValue = 1000
SpDelegate: getValue : result = 1000
MainActivity: 1000
SpDelegate: getValue : result = def String value
MainActivity: def String value
SpDelegate: setValue : newValue = new String value
SpDelegate: getValue : result = new String value
MainActivity: new String value
```



