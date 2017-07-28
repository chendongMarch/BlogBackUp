---
layout: post
title: Kotlin开发-8-范型
categories:
  - Kotlin
tags:
  - Kotlin
keywords:
  - Kotlin
  - generic
  - 范型
abbrlink: 9e4433d5
date: 2017-07-27 10:14:00
---
 
本文主要介绍 `Kotlin` 范型的相关用法。

泛型的本质是参数化类型，操作的数据类型被指定为一个参数。使用范型约束：

1. 增加代码的复用性，有时我们使用一些公用的数据结构，方法，类等，只是操作的对象类型不一样，但是代码逻辑一样，此时可以使用范型复用代码，比如 `List<T>`，就可以用来存储任何一种对象。

2. 保证代码中类型转换的安全性，使用范型进行约束，能够保证在编译期对类型的匹配进行检测和转换，避免运行期出现类型转换异常，而且这些转换都是自动和隐式的。

[推荐阅读 - Kotlin 泛型详解 - 10条](http://www.10tiao.com/html/330/201707/2653579163/2.html)
<!--more-->

## 简单实现

下面在 `Java` 和 `Kotlin` 中实现了最简单的范型的用法，用法相似，但是 `Kotlin` 就更加简单。当类型可以通过推断得到时，不必显式声明类型。

```java
// java
class Box<T> {
    private T t;
    public Box(T t) {
        this.t = t;
    }
    public T get() {
        return t;
    }
    public void set(T t) {
        this.t = t;
    }
}

Box box = new Box<>(1);
```


```kotlin
// kotlin
class Box<T>(var t: T)

val box = Box(1)
```

## Java 中的范型

范型类型是不可变的，因此 `List<Child>` 并不是  `List<Parent>` 的子类型

```java
// List<String> 不是 List<Object> 的子类型

List<Object> objects = new ArrayList<>();
List<String> strings = new ArrayList<>();

objects = strings; // error.
```

为什么是不可变的呢？我们看一段伪代码，假如类型是可变的，那么 `List<String>` 就是 `List<Object>` 的子类。

```java
List<Object> objects = new ArrayList<>();
List<String> strings = new ArrayList<>();
objects = strings; 
objects.add(100);
String s = strings.get(0); // ClassCastException
```
由于范型类型的不可变，尽管 `String` 是 `CharSequence` 的子类型，但是我们甚至不能简单的做到如下操作：

```java
class Box<T> {
    private T t;
    public void resetBox(Box<T> box) {
        this.t = box.t;
    }
}

Box<CharSequence> charSequenceBox = new Box<>();
Box<String> stringBox = new Box<>();
charSequenceBox.resetBox(stringBox); // error,在编译期禁止此操作
```
正因为如此，才有了 **通配符类型参数** ，我们应该如下声明 `resetBox()` 的方法

```java
class Box<T> {
    private T t;
    public void resetBox(Box<? extends T> box) {
        this.t = box.t;
    }
}
```

## 通配符类型参数

我们参照 `List` 类，来看一个简单的例子

```java
List<? extends String> extendsTStringList = new ArrayList<>();
// error  不能写入
extendsTStringList.add("");
// 可以读取
String s = extendsTStringList.get(0);
```
这里的 通配符类型参数 `? extends T` 表示，集合元素的类型是 `T` 的某种子类型, 而不限于 `T` 本身，这就意味着，我们可以安全地从集合元素中 读取 `T` (因为集合的元素是 `T` 的某个子类型的实例)，但 不能写入 到集合中去，因为我们不知道什么样的对象实例才能与这个 T 的未知子类型匹配。指定了 `extends` 边界 (上边界)的通配符类型, 使得我们的类型成为一种 `协变(covariant)` 类型。

同理，我们可以指定类型的下边界:

```java
 List<? super String> superTStringList = new ArrayList<>();
 // 可以写入
 superTStringList.add("");
 // 不能读取或只能读取为 object 类型
 Object o = superTStringList.get(0);
```
通配符类型参数 `? super T` 表示，集合元素的类型是 `T` 的父类型，而不限于 `T ` 本身，我们可以向集合中添加元素，因为集合中的元素一定是 `T` 的父类型，但我们不能从集合中读取元素，或者说只能读取为 `object` 类型的元素，因为我们不知道什么样的对象才能与 `T` 的未知父类型匹配。指定了 `super` 边界 (下边界)的通配符类型, 使得我们的类型成为一种 `逆变(contravariance)`。

> 协变；子类取代父类的位置是被允许的，也就是需要父类型时可以使用子类型代替
> 逆变；父类取代子类的位置是被允许的，也就是需要子类型时可以使用父类型代替
> 不变；不允许改变类型。
        
对于只能读取的对象，称为 **生产者**，因为它可以产出对象，对于只能写入的对象称为 **消费者**，因为它可以消费对象，生产者对应 `extends`，消费者对应 `super`。

## 声明处类型变异

我们有如下 `Box` 类，它只有读取方法，也就是只能生产 `T` 类型对象，不存在消费者方法的调用，因此我们完全可以使用 `Box<Parent>` 存储 `Box<Child>` 的数据，这是安全的，但是在 `java` 中不能理解这一点。

```java
class Box<T> {
    public T find(){
        return null;
    }
}

public void test(Box<String> stringBox) {
    Box<CharSequence> objectBox = stringBox; // error
}
```

为了解决上述问题，我们必须进行如下声明才能消除错误，但是使用了更加复杂的声明与上面所能调用的方法是一样的，但是编译器并不能理解这一点。

```java
public static void test(Box<String> stringBox) {
    Box<? extends CharSequence> objectBox = stringBox;
}
```

在 `Kotlin` 中我们使用在范型声明处使用注解标注的方式将这种情况告诉编译器，由于这种注解出现在声明处，因此称为 **声明处类型变异(declaration-site variance)** ，这种方案与 `Java` 中的 **使用处类型变异(use-site variance)** 刚好相反, 在 `Java` 中, 是类型使用处的通配符产生了类型的协变。有两种注解修饰符：
 
> `out` 被称为 **协变注解(variance annotation)**，他表示此类型只能被生产，不能被消费，也就是只能出现在返回类型中，此时 `Box<Parent>` 可以安全的用作 `Box<Child>` 的父类型。

> `in` 被称为 **逆变注解(contravariant annotation)**，他表示此类型只能被消费，不能被生产，也就是只能出现在参数类型中，此时可以用 `Box<Child>` 安全的接受 `Box<Parent>`

```kotlin
interface Box1<out T>{
    // error，不能作为参数类型
    fun testIn(t:T)
    // yes
    fun testOut(): T
}

interface Box2<in T>{
    // yes
    fun testIn(t:T)
    // error，不能作为返回类型
    fun testOut(): T
}
```
那么此时我们就可以解决上面的复杂声明的问题

```kotlin 
fun test(box:Box1<String>){
    val box1:Box1<CharSequence> = box
}

fun test(box:Box2<CharSequence>){
    val box2:Box2<String> = box
}
```
同时看一下官网 `Comparable` 使用逆变注解的例子

```kotlin
abstract class Comparable<in T> {
    abstract fun compareTo(other: T): Int
}

fun demo(x: Comparable<Number>) {
    x.compareTo(1.0) // 1.0 类型为 Double, 是 Number 的子类型
    // 因此, 我们可以将 x 赋值给 Comparable<Double> 类型的变量
    val y: Comparable<Double> = x // OK!
}
```

## 类型投射

我们有时并不能保证范型类型只作为返回值出现，或只作为参数出现，我们通常会有更复杂的需求:

```kotlin
class Box3<T> {
    fun get(): T?{
        return null
    }
    fun set(t: T){

    }
}
```

如果我们写一个 `copy` 函数

```kotlin
fun copyBox(to: Box3<Any>, from: Box3<Any>) {
    to.set(from)
}
```

由于范型参数类型的不变性，就是我们最初在 `Java` 中遇到的问题，下面的调用将无法通过编译，因为 `Box<String>` 并不是 `Box<Any>` 的子类，这种操作是为了避免向 `from` 中进行写入操作，比如 `Int` 类型，导致 `ClassCastException`。

```kotlin
val from = Box3<String>()
val to = Box3<Any>()
copyBox(to,from) // error,required:Box3<Any> found:Box3<String>
```
我们需要使用 `out` 注解，说明我们在 `copy` 函数中对 `from` 不会进行写入操作。

```kotlin
fun copyBox(to: Box3<Any>, from: Box3<out Any>) {
    to.set(from)
}

val from = Box3<String>()
val to = Box3<Any>()
copyBox(to,from) // ok
```
同理我们可以使用 `in` 注解来声明我们不会对对象进行读取操作，如下

```kotlin
fun fill(dest: Box3<String>, value: String) {
    // ...
}

val dest = Box3<CharSequence>()
fill(dest,"") // error


fun fill(dest: Box3<in String>, value: String) {
    // ...
}

val dest = Box3<CharSequence>()
fill(dest,"") // ok
```

这种声明在 `Kotlin` 中称为 **类型投射(type projection)**，他用来声明该类型不是一个简单的类型，而是一个被限制的类型，我们只能对该类型进行读取或写入操作，从而保证数据的安全性，这是 `Kotlin`  **使用处类型变异**  的实现方式，与 `Java` 中的通配符相似，但是更简单。

## 星号投射

星号投射（Star-projection）

有时可能想表示你并不知道类型参数的任何信息，但是仍然希望能够安全地使用它。这里所谓”安全地使用”是指，对泛型类型定义一个类型投射，要求这个泛型类型的所有的实体实例，都是这个投射的子类型。

星号投射与 `Java` 的原生类型(`raw type`)非常类似，但可以安全使用。

下面是官方的说法：

```
对于 Foo<out T>，其中 T 是一个具有上界 TUpper 的协变类型参数，Foo<*> 等价于 Foo<out TUpper>。 这意味着当 T 未知时，你可以安全地从 Foo<*> 读取 TUpper 的值。

对于 Foo<in T>，其中 T 是一个逆变类型参数，Foo<*> 等价于 Foo<in Nothing>。 这意味着当 T 未知时，没有什么可以以安全的方式写入 Foo<*>。

对于 Foo<T>，其中 T 是一个具有上界 TUpper 的不型变类型参数，Foo<*> 对于读取值时等价于 Foo<out TUpper> 而对于写值时等价于 Foo<in Nothing>。
```

在什么时候可以使用 星号投射，什么时候不可以

```kotlin
// error 不允许作为变量的的范型参数
val list = ArrayList<*>()

// error 不允许作为函数的范型参数
fun <T> hello(args: Array<T>){} 
hello<*>(args)

// error 不允许作为父类的范型参数
interface Foo<T> 
class Bar : Foo<*>

// yes 尽管 * 不能直接作为类的泛型参数，Foo<*> 却可以，按照前面官方给出的说法，它在读时等价于Foo<out Any> 写时等价于 Foo<in Nothing>
interface Foo<T> 
class Bar : Foo<Foo<*>> 

// yes 这表示接受的参数的类型在读写时分别等价于Array<out Any> 和 Array<in Nothing> 
fun hello(args: Array<*>){ 
     ... 
}
```

`Raw` 类型就是对于定义时有泛型参数要求，但在使用时指定泛型参数的情况，这个只在 `Java` 中有，显然也是为了前向兼容。在 `Java` 中我们可以如下声明

```java
List list = new ArrayList(); 
```
这样的声明在 `Kotlin` 中是不允许的，但是可以使用

```kotlin
val list = ArrayList<Any?>() 
```
在 `Java` 中可以有这样的写法，会引发异常

```java
List<Integer> integers = new ArrayList<>(); 
List list = new ArrayList(); 
list = integers;
```
在 `Kotlin` 中这样的写法是错误的

```kotlin
var list = ArrayList<Any?>() 
val integers = ArrayList<Int>() 
list = integers // ERROR! 
```

## 范型函数

函数中也可以使用范型参数类型

```kotlin
fun <T> test(t:T):List<T>{
    return mutableListOf(t)
}
fun <T> Context.test(t:T):List<T>{
    return mutableListOf(t)
}

var testGenericFun = testGenericFun(1)
var testGenericFunInContext = testGenericFunInContext(1)
```


## 范型约束

约束范型的上界

如下范型参数 `T` 必须是 `Box<T>` 的子类

如果有多个上限，则需要使用 `where` 字句

如果上限的对象带有范型参数的声明处类型编译注解，则当前声明也需要同步

```kotlin
class Box4<T : Box<T>> {

}

class Box5<T> where T : Box<T>, T : Cloneable {

}

class Box6<out T> where T : Box1<T>, T : Cloneable {

}
```
在函数中约束范型上界

```kotlin
fun <T : Box<T>>test1(t:T){

}

fun <T> test2(t:T) where T : Box<T>, T : Cloneable{

}
```