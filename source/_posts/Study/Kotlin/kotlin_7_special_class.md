---
layout: post
title: Kotlin开发-7-特殊类
categories:
  - Kotlin
tags:
  - Kotlin
keywords:
  - Kotlin
  - 类
  - 数据类
  - 嵌套类
  - 内部类
  - 匿名内部类
  - 枚举类
abbrlink: aa2c40d4
date: 2017-07-23 08:57:00
---
 
本文主要关于 `Kotlin` 中的特殊类，包括 **数据类**，**嵌套类**，**内部类**，**匿名内部类**，**枚举类**

<!--more-->
## 数据类

数据类是一种特别的类，顾名思义，就是用来结构化的存放数据的类，在 `Kotlin` 中可以使用关键字 `data` 来很简单的声明一个数据类。

```kotlin
data class Person(val name: String = "123", val age: Int = 123)

data class User(val name: String, val age: Int)
```

如果在数据类中对以下函数没有明确的定义，也没有从父类中国年继承的多，那么数据类会自动生成这些函数，如 `hashCode()` 和 `equals()` 函数对、`Person(name=c, age=3)` 形式的 `toString()` 方法、用来访问属性址的 `componentN()` 函数群、用来复制对象 `copy()` 方法。

```kotlin
// 创建数据类对象
val p: Person = Person("a", 1)
val p1 = Person("b", 2)
// equals() & hasoCode()
log(p.equals(p1))
// toString()
log(p.toString())
// componentN
log(p.component1() + " " + p.component2())
// copy()，copy 的对象将会是一个全新的对象
val p2 = p.copy("c", 3)
log( "${p2.hashCode()} ${p.hashCode()} $p2")
```
输出

```bash
MainActivity: false
MainActivity: Person(name=a, age=1)
MainActivity: a 1
MainActivity: 3072 3008 Person(name=c, age=3)
```
数据类应该满足以下要求：

1. 主构造器至少有一个参数，如果需要无参构造方法，可以使用指定默认值的方式实现。

2. 主构造器的所有参数都必须标记为 `var` 或 `val`，像 `data class Use2(name: String, age: Int)` 这样是不允许的。

3. 数据类不能是 `抽象类`，`open 类` ，`封闭(sealed)类` 或 `内部类`

4. 数据类不能继承自其他类，但是可以实现接口。

### copy 函数

自动生成的 `copy()` 函数用来实现拷贝一个对象，同时修改部分属性，但是保留其他属性的值的功能。	

自动生成的 `copy()` 函数将会是这样的

```kotlin
fun copy(name: String = this.name, age: Int = this.age) = User(name, age)
```

### 解构

数据类默认生成了 `componentN()` 组件函数，我们可以用如下方法快速获取数据类中属性的值

```kotlin
val p: Person = Person("a", 1)
val (myName,myAge) = p
log("name = $myName,age = $myAge")
```

## 嵌套类

类可以嵌套在另一个类中，称为 嵌套类（Nested Class），其实相当于 `Java` 中的静态内部类，不过在 `Java` 中这样什么关键字都不加的情况默认声明的是内部类，而 `Kotlin` 默认是嵌套类，也就是静态内部类，可以直接使用 `外部类类名.嵌套类名` 来访问创建嵌套类对象。

```kotlin
class OuterClass {
    class NestedClass(val name: String)
}

// 可以直接创建，不依赖外部类对象
val nested = OuterClass.NestedClass("nest")
```

##  内部类
`Kotlin` 中的内部类和  `Java` 中的内部类一样，但是 `kotlin` 中要用 `inner` 关键字声明是内部类，否则默认是嵌套类。

使用 `inner` 关键字声明的内部类，可以自由访问外部类成员属性，内部类会持有一个外部类的引用，指向外部类对象实例。 

当属性冲突时，将优先使用内部类成员，如果想访问外部类成员可以使用带限定符的 `this` 表达式。

内部类是不能直接创建的，需要使用外部类对象才能创建。

```kotlin
class OuterClass {
    class NestedClass(val name: String)
    val myVal = 100
    val myVal1 = 100
    inner class InnerClass(){
        val myVal = 10
        fun test():String{
            // 当属性冲突时，将优先使用内部类成员
            // 如果想访问外部类成员可以使用带限定符的this表达式
            return "$myVal $myVal1 ${this@OuterClass.myVal}"
        }
    }


// 必须依赖外部类对象才能创建
val inner = OuterClass().InnerClass()
``` 


## 匿名内部类

匿名内部类也是一个常见的用法，适用于只需要创建一次使用一次的情况，比如为控件设置监听。

匿名内部类使用 `object` 关键字的对象表达式声明，其实下面中监听的设置不必这么复杂，这是最原始的版本。

```kotlin
mMyMsgTv.setOnClickListener(object : View.OnClickListener{
    override fun onClick(v: View?) {
    	log("click")
    }
})
```
如果匿名内部类对象是只有一个抽象方法的接口，可以使用带有接口名称前缀的 `lambda` 表达式简化创建这个对象，简化版本：

```kotlin
mMyMsgTv.setOnClickListener {
    log("click")
}
```


## 枚举类

创建一个简单的枚举类

```kotlin
enum class EnumClass{
    ONE,TOW,THREE
}
```

每个枚举值都是枚举类的一个实例，因此可以被初始化，需要注意的是 `name` 和 `ordinal` 是父类中已经存在的不允许被重写的属性，因此自定义的属性不能使用这些。


```kotlin
enum class EnumClass2(val desc:String){
    ONE("one"),TOW("two"),THREE("three")
}
```

枚举可以具有自己的属性和函数，也可以重写父类中的方法

```kotlin
enum class EnumClass3(val desc: String) {
    ONE("one") {
        override fun log(): String {
            return "one"
        }
    },
    TOW("two") {
        override fun log(): String {
            return "two"
        }
    },
    THREE("three") {
        override fun log(): String {
            return "three"
        }
    };
    // 属性
    val myVal = 100
    // 函数
    fun test() {}
    abstract fun log(): String
}
```

每个枚举都具有 `name` 和 `ordinal` 属性，并且实现了 `Compaable` 接口

```kotlin
EnumClass1.ONE.name
EnumClass1.ONE.ordinal

// 转化为一个枚举类对象
var enumClass1:EnumClass1 = EnumClass1.valueOf("one")
// 枚举类对象列表
var values:Array<EnumClass1> = EnumClass1.values()
```

