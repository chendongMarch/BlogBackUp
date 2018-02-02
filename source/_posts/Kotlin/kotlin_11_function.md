---
layout: post
title: Kotlin开发-11-函数
categories:
  - Kotlin
tags:
  - Kotlin
keywords:
  - Kotlin
  - 函数
abbrlink: acb7c552
date: 2017-08-01 18:15:00

---

本文学习 `Kotlin` 函数的相关用法。主要包括 ：

函数的声明和使用，函数的参数和返回值，单表达式函数的使用，使用不定长参数。

局部函数，尾递归函数，高阶函数，内联函数等。

`Lambda` 表达式，匿名函数和闭包。

其他如，`reifeid` 关键字，非局部返回及带有接受者的函数字面值。

函数在 `Kotlin` 函数是一等公民，它的地位和对象一样，所以不要对函数特别看待，就把它当作对象看就行了，这样很多东西会好理解的多。

<!--more-->

## 声明和使用

函数声明的一般形式为

```kotlin
fun functionName(param1:Type,param2:Type...)[:ReturnType]{

}
```
如果是 `top-level` 级别的函数可以直接使用函数名调用，如果是一个类函数，需要拿到类实例，使用点号标记法调用。

```kotlin
fun simpleFunction(param:String):Int{
    return param.length
}

// 调用
simpleFunction("100")
Engineer().test()
```
`Kotlin` 支持中缀标记法调用函数，如果需要支持中缀标记法调用，需要满足条件：

> 是一个类函数或扩展函数，使用 `infix` 关键字声明，只有一个参数

```kotlin
infix fun Int.cross(another: Int): Int {
    return this * another
}

class Engineer(var name: String = "", var age: Int = 0){
    // together 函数
    infix fun together(e: Engineer): Int {
        return this.age + e.age
    }
}
```
它使得函数可以像运算符那样使用

```kotlin
val x = 100
val result = x cross 10
// 相当于
val result2 = x.cross(10)

val e = Engineer()
var newEngineer = e together Engineer(name = "new")
// 相当于
var newEngineer2 = e.together(Engineer(name = "new"))
```

## 参数和返回值

在 `Kotlin` 中，参数使用 `param:Type` 的形式声明，每个参数必须声明数据类型；返回值在函数的最后，函数体之前，使用 `:returnType` 的形式声明，如果没有返回值，可以使用 `:Unit` 或者不写返回值的形式来表示。

参数支持 默认值 和 参数名指定参数 调用，简化函数的重载。主要体现在：

具有默认值的参数我们在调用过程中可以不传该参数，将会使用参数的默认值，另外我们可以使用参数名指定某些参数，也就是只传部分参数，当然没有默认值的参数是必须传的。

使用函数的默认值进行函数重载时，需要注意参数是从第一个参数开始往后面匹配的，因此无默认值的参数要往前面放，如下函数：

另外，由于如果最后一个参数是一个函数，那么可以将这个函数放在 `()` 之外，使用 `Lambda` 表达式传参，因此，如果有这样的函数参数，建议放在最后。

```kotlin
fun testDefParam(p1: String = "", p2: Int = 10, p3: Int = 1, p4: String) {

}

// 我们如此调用是不可以的
testDefParam("", "") // error

//  我们需要将无默认值的参数提到前面，才能自动匹配上
fun testDefParam(p4: String, p1: String = "", p2: Int = 10, p3: Int = 1) {

}
```

使用默认值调用时，当参数过多时，默认匹配的方式也多，而且我们必须将没有默认值的参数提前，会导致代码阅读性变差，不容易维护，因此相对更推荐使用参数名指定参数的方式，代码会更清晰，参数列表也没有任何限制，但是如果没有默认值的参数是必须传值的。

```kotlin
fun testNamedParam(p1: String = "", p2: Int = 10, p3: Int = 1,p4: String) {
}

// p4 参数是必须传的
testNamedParam(p2 = 11,p4 = "test")
```

类函数在继承时，子类不可以声明默认值，不论父类中同函数参数有没有默认值都不可以。

```kotlin
open class ATest {
    open fun test(p1: Int, p2: String = "") {
    }
}

class BTest : ATest() {
    // p1 p2 参数都不能再声明默认值
    override fun test(p1: Int, p2: String) {
        super.test(p1, p2)
    }
}
```
虽然在子类中我们不能声明参数默认值，但是父类中的参数默认值在子类中同样生效，如上面的代码我们可以如下调用：

```kotlin
ATest().test(1)
BTest().test(1)	
```

函数的参数中的基本数据类型都是 `val` 类型的变量，也就是在函数体内无法改变其值

```kotlin
fun testFuncParamVal(index:Int,user:User){
    index = 100 // error. val can not be reassigned
    user.name = "" // error. val can not be reassigned
}
```

## 单表达式函数(Single-Expression)

单表达式函数 (Single-Expression function)

如果函数可以直接使用一条简单的表达式表示，可以省略函数体，直接在  `=`  之后声明，如果函数的返回类型可以通过类型推断得到，那么返回类型也可以省略。

```kotlin
fun singleExpressionFunc(): Int = 12 * 10
// 省略返回类型
fun singleExpressionFunc() = 12 * 10
```

## 不定长参数(varargs)

类似 `java` 中的 `...` ，`kotlin` 使用 `varargs` 来表示不定长参数，`varargs` 声明的参数的类型实际上是一个 `Array<out T>` 类型的值，因此可以使用下标访问每一个参数

```kotlin
fun  asList(vararg args: String): List<String> {
    val list = ArrayList<String>()
    for (arg in args) {
        list.add(arg)
    }
    return list
}

// 简单调用
 asList("1","@","3")
```
通常情况下 `varargs` 类型的参数应该是最后一个参数，因为由于  `varargs` 类型的参数接受的是一个数组，后面的参数将无法识别，不过好在 `kotlin` 有使用参数名指定参数的传参方法。

```kotlin
fun  asList(vararg args: String,count:Int): List<String> {
    val list = ArrayList<String>()
    list += args
    return list
}

// test
asList("1","@","3",count = 1)
```
当我们已经有一个数组时，希望将这个数组作为参数直接传递给函数，此时是无法传参的，因为它会将整个 `Array` 作为一项，如下面的代码中，会提示类型不对，这种情况可以使用 **展开操作符(\*)**，将数组展开

```kotlin
val strings = arrayOf("1", "2", "3")
// error. Required:String,Found:Array<String>
asList(strings,count = 1) 

// 正确姿势
val strings = arrayOf("1", "2", "3")
asList(*strings,count = 1)
```


## 几种函数类型


**顶级函数**：函数可以声明为 `top-level` 级别，我们不需要单独定义一个类来容纳这个函数，顶级函数可以访问公开的类和属性。

**局部函数**：指的是声明在函数内部的函数，局部函数可以访问外部函数的局部变量。

```kotlin
// MainActivity 中的变量
val valueInMainActivity = 100
// 外部函数
fun localFunctionOutter(){
    // 外部函数的局部变量
    val valueInLocalFunctionOuter = 10
    // 局部函数
    fun localFunctionInner(){
        // 局部函数的局部变量
        val valueInLocalFunctionInner = 1
        // 局部函数可以访问外部函数的局部变量
        log("$valueInMainActivity $valueInLocalFunctionOuter $valueInLocalFunctionInner")
    }
    // 调用局部函数
    localFunctionInner()
}
```

**成员函数**：声明在类和对象中的函数，它作为类的成员可以访问其他成员。

**范型函数**：函数中可以带有范型，关于范型的相关问题在 [kotlin-范型](../9e4433d5) 会有更细致的描述。

```kotlin
fun <T> test2(t:T) where T : Box<T>, T : Cloneable{

}
```

**扩展函数**：对已有类进行扩展，关于扩展函数的相关问题在 [kotlin-扩展](../d0378250) 会有更细致的描述。

```kotlin
fun <E> MutableList<E>.swap(index1: Int, index2: Int) {
    val temp = this[index1]
    this[index1] = this[index2]
    this[index2] = temp
}
```

## 尾递归函数(tailrec)

我们在 `Java` 中使用递归函数时，如果递归的次数太多，具体能够承载的递归次数与函数的复杂程度有关，会导致堆栈溢出，`kotlin` 对递归函数做了优化，使用 `tailrec`  关键字声明尾递归函数，并且满足要求的形式，在编译时，会消除函数的递归调用，产生一段基于循环的代码，避免嵌套过深导致堆栈溢出。

要符合 `tailrec` 修饰符的要求，函数必须在它执行的所有操作的最后一步，递归调用它自身。如果在这个递归调用之后还存在其他代码，那么你不能使用尾递归，而且你不能将尾递归用在 `try/catch/finally` 结构内。

尾递归函数和我们平常的递归函数没有很大差别，使用 `tailrec` 关键字声明，参照官网上面的例子。

```kotlin
tailrec fun findFixPoint(x: Double = 1.0): Double
        = if (Math.cos(x) == x) x else findFixPoint(Math.cos(x))
```
在编译时，他将会生成类似如下的结构

```kotlin
private fun findFixPoint(): Double {
    var x = 1.0
    while (true) {
        val y = Math.cos(x)
        if (x == y) return y
        x = y
    }
}
```

## 高阶函数

在 `kotlin` 中函数和属性一样可以作为另一个函数的参数或返回值，这类函数被称为高阶函数，它使得函数可以如同对象一样进行传递。

如同字符串类型是 `String`，整型的类型是 `Int`，函数具有自己的类型，函数的类型使用 `(p1:Type1,p2:Type2...) -> returnType`，其中参数的名称 `p1,p2` 是可以省略的，但是参数列表的 `()` 不能省略，需要注意的是即使函数返回值是 `Unit` 也无法省略，因为编译器将无法识别 `(Type)` 是一个什么类型。 

```kotlin
// 声明函数的类型
val fun1: (String, Int) -> Int = {
    p1, p2 ->
    p1.length + p2
}

// 带有参数名
val fun2: (p1: Int) -> Int = {
    p1 ->
    p1
}

// 返回值为 Unit
val fun3: (String) -> Unit = {}

// error. 不能省略 Unit
val fun4: (String) = {}
```

### 函数作为返回值

函数可以作为返回值，作为返回值时，需要使用 `Lambda` 表达式的方式书写。

```kotlin
// 函数作为返回值
fun testReturnFunction(): (String, Int) -> Int {
    // 返回一个 参数为 (String,Int) 返回值为 Int 的函数
    return {
        p1: String, p2: Int ->
        p1.length + p2
    }
}

// 我们可以使用该函数获取内部返回的函数
val returnFunction = testReturnFunction()
var value = returnFunction("2", 1)
```

### 函数作为参数

函数可以作为函数参数，声明函数时可以像普通对象那样对待函数类型的参数

```kotlin
// 函数作为参数
fun testParamFunction(inFun: (String, Int) -> Int): Int {
    return inFun("1", 1)
}
```

我们可以使用一个 `Lambda` 表达式创建一个函数作为参数，当函数 A 的最后一个参数也是函数时，可以在函数 A 后面直接用 `{}` 声明函数体。

```kotlin
// 普通用法
testParamFunction(inFun = {
    s, i ->
    s.length + i
})

// 简单用法
testParamFunction { s, i -> s.length + i }
```
如果该函数还有其他参数，那就必须使用 `()` 来放置这些参数，如下：

```kotlin
// 函数作为参数
fun testParamFunction(count:Int,inFun: (String, Int) -> Int): Int {
    return inFun("1", 1)
}

// 普通用法
testParamFunction(1,inFun = {
    s, i ->
    s.length + i
})
// 简单用法
testParamFunction(1) { s, i -> s.length + i }
```
使用  `::` 操作符，可以将一个定义好的参数作为参数传递，这边还涉及了一点反射的内容，暂时不去深究。

```kotlin
class FunParamTest() {

    fun isYes(str: String): Int {
        return str.length
    }

    fun test1(param:String) {

        // 直接定义一个函数
        val p1: (String) -> Int = { 100 }
        test2(p1)

        // 一个局部函数
        fun isOdd(str: String): Int {
            return str.length
        }

        // 用一个局部变量接受这个函数，并调用他
        val p2 = ::isOdd
        p2("")

        // 直接传递该函数作为参数
        test2(::isOdd)

        // error，暂时不知道为什么，只有局部函数可以使用 :: 操作符传递
        test2(::isYes)
    }

    fun test2(f: (String) -> Int) {

    }
}
```

融会贯通，让函数如同对象一样自由的传递，我们以上面的两种函数为例

```kotlin
// 声明一个函数
val fun1: (String, Int) -> Int = {
    p1, p2 ->
    p1.length + p2
}
testParamFunction(1, inFun = fun1)


// 从另一个函数中返回一个函数
val fun5: (String, Int) -> Int = testReturnFunction()
testParamFunction(1, inFun = fun5)


// 使用 lambda 表达式创建一个函数
testParamFunction(1, inFun = {
            p1, p2 ->
            p1.length + p2
        })


// 使用声明好的函数 
fun fun6(p1:String,p2:Int):Int{
    return p1.length + p2
}
testParamFunction(1, inFun = ::fun6)
```

## Lambda 表达式

在 `Kotlin` 中使用 `Lambda` 表达式，`Lambda` 表达式使用如下方式。


```kotlin
{
	param1,param2 ->
	...函数体...
}
```


当某个参数我们不用时，可以使用 `_` 代替，如函数

```kotlin
fun testLambda(inFun: (String, Int) -> Int): Int {
    return inFun("1", 1)
}

testLambda{
    p1,p2 ->
    p1.length + p2
}

// 不使用 p2 参数
testLambda{
    p1, _ ->
    p1.length
}
```
当只有一个参数时，如果没有用到这个参数可以直接不用声明，也可以使用单一参数的隐含名称 `it` 代替。

```kotlin
fun testLambda(inFun: (String) -> Int): Int {
    return inFun("1")
}

// 当不使用参数时，可以不用声明
testLambda {
    0
}

// 隐含参数 it
testLambda {
    it.length
}
```

默认函数体的最后一条语句将会作为函数的返回值，但是不需要显式使用 `return` 关键字。如果想在其他位置返回值，需要使用带有后缀的 `return` 语句。

```kotlin
// 最后一句将作为返回值
testLambda {
    it.length
}

// 使用带后缀的 return 语句返回指定值
testLambda {
    if (it.length > 100)
        return@testLambda it.length
    else
        return@testLambda 100
}
```

## 匿名函数

匿名函数与普通函数类似，但是不需要指定函数名，支持单单表达式函数，也支持多行函数。

匿名函数必须使用  `()` 传递，对 `Lambda` 表达式支持的一些使用方式，对匿名函数并不支持。

`Lambda` 表达式与匿名函数之间的另一个区别是，它们的 **非局部返回(non-local return)** 的行为不同。不使用标签的 `return` 语句总是从 `fun` 关键字定义的函数中返回。也就是说，`Lambda` 表达式内的 `return` 将会从包含这个 `Lambda ` 表达式的函数中返回，而匿名函数内的 `return` 只会从匿名函数本身返回。 

```kotlin
testParamFunction(1, inFun = (fun(p1, p2) = p1.length + p2))

testParamFunction(1, inFun = (fun(p1, p2): Int {
    return p1.length + p2
}))
```

## 函数的闭包

`Lambda` 表达式和匿名函数可以访问外部变量，因此会形成闭包。

函数 和 函数能访问到的变量(环境) 的总和 就形成了一个闭包。也就是说，当一个内部函数访问了外部函数的变量，此时会形成一个闭包。

使用闭包的目的是隐藏一些变量，这些变量只在外部函数中声明，但是不需要声明为全局属性。

如下函数声明，在 `MainActivity` 中和外部函数中分别都有变量，内部函数访问这些变量。

```kotlin
// MainActivity 中的变量
val valueInMainActivity = 100
// 外部函数
fun localFunctionOuter(): () -> Unit {
    // 外部函数的局部变量
    val valueInLocalFunctionOuter = 10
    // 局部函数
    fun localFunctionInner() {
        // 局部函数的局部变量
        val valueInLocalFunctionInner = 1
        // 局部函数可以访问外部函数的局部变量
        log("valueInMainActivity = $valueInMainActivity " +
                ",valueInLocalFunctionOuter = $valueInLocalFunctionOuter " +
                ",valueInLocalFunctionInner = $valueInLocalFunctionInner")
    }
    // 调用局部函数
    return ::localFunctionInner
}
```

在下面的调用中，虽然 `localFunctionOuter()` 函数已经结束了， 但是仍然可以使用返回的函数访问 `localFunctionOuter()` 内部的变量，我们对外部隐藏了 `valueInLocalFunctionOuter` 这个变量。

```kotlin
val f = localFunctionOuter()
f()

// 输出
MainActivity: valueInMainActivity = 100 ,valueInLocalFunctionOuter = 10 ,valueInLocalFunctionInner = 1
```

## 带有接受者的函数字面值

带有接受者的函数字面值（Function Literals with Receiver）

这是一个类似于扩展函数一样的特性，我们借助官网的几个例子理解一下他的用法。

```kotlin
val sum1 = fun Int.(other: Int): Int = this + other
1.sum1(2)
```

这种声明方式和普通的函数有相似之处，他只是在参数前面追加了 `Int` 类型，来表示这个函数是 `Int` 类型可以直接调用的。

用一个类似的例子看一下与普通函数的对比，我加了些空格来对比，普通函数不能访问 `this` 变量，也不能直接使用 `Int` 类型调用函数。

```kotlin
val sum1 = fun Int.(other: Int): Int = this + other
val sum2 = fun     (other: Int): Int = other
```

对比一下，这种函数和扩展函数的差别，在使用和定义都没啥太大的差别，只是一个又函数名一个没有。

```kotlin
val sum1 = fun Int.(other: Int): Int = this + other

fun Int.sum3(other: Int): Int {
    return this + other
}

1.sum1(2)

1.sum3(2)
```

再来看一下官网的另一个例子，第一眼看上去是有些懵逼的，分析一下

```kotlin
// 一个普通的类
class HTML {
    // 一个普通的函数
    fun body() {
    }
}

// 还是一个普通的函数，返回值为 HTML 对象
// init 参数是一个无参无返回值的函数，
// 由于使用了 (HTML.) 说明这个函数是一个可以被 HTML 对象直接调用的函数
fun html(init: HTML.() -> Unit): HTML {
    val html = HTML()  
    html.init() // 使用创建的 HTML 对象调用该函数
    return html
}
```

调用，接受的参数是 `HTML` 类的一个无参无返回值的函数。

```kotlin
html {
    body()   
}

相当于

html({
    body()
})
```

## 内联函数(inline)

使用高阶函数，每个函数都是一个对象，而且它还要捕获一个闭包，这些都会造成内存的损耗和运行效率的下降。使用内联函数可以将函数内联在函数调用处，而不用生成多余的对象再去调用他。

使用内联函数需要使用 `inline` 关键字声明，`inline` 关键字会影响函数本身，也会影响传递给他的表达式，这两者都会被内联到调用处。

```kotlin
inline fun testInline(str:String,f: (String) -> Int): Int {
    return f(str)
}
```

如果函数有多个函数参数，可以使用 `noinline` 关键字声明那些不需要内联的函数

```kotlin
inline fun testInline(str: String, f1: (String) -> Int, noinline f2: () -> Unit): Int {
    f2()
    return f1(str)
}
```

内联函数不能作为递归参数使用，对于内联函数中没有声明为 `noinline` 函数参数(它们都是内联的)，具有一些限制，这些参数 不能保存在域中，不能作为函数返回值返回，只能在内联函数内部调用，只能作为可以内联的函数参数传递给其他函数，我们通过一个例子说明：

```kotlin
// 一个内联函数
inline fun testInline2(f1: (String) -> Int){
}

// 一个非内联函数
fun testInline3(f1: (String) -> Int){
}

inline fun testInline(str: String, f1: (String) -> Int, noinline f2: () -> Unit):  (String) -> Int {
    
    // error. inline 不可以保存在域内
    // val localFun = f1
    
    // error. 不能是递归的
    // testInline(str,f1,f2)
    
    // yes. 内联的 lambda 表达式参数，可以作为其他内联函数的参数
    testInline2(f1)
    
    // error. 内联的 lambda 表达式参数，不能作为非内联函数的参数
    // testInline(f1)
    
    // error. inline 不可以作为返回值
    // return f1
}
```

## 非局部返回(Non-local return)

非局部返回（Non-local return）

在 `Kotlin` 中，使用不带标签的 `return` 语句只能返回使用 `fun` 关键字声明的函数，即普通的函数和匿名函数，在 `Lambda` 表达式是不允许使用不带标签的 `return` 语句的。

```kotlin
fun testNonReturn(f: (String) -> Int) {
    log(f("test"))
    log("testNonReturn")
}

testNonReturn {
    // 不允许 return
    it.length
}
```
我们可以使用带有指定标签的 `return` 语句，用来返回不同条件下的结果，因为 `Lambda` 表达式总是将最后一句的结果作为返回值，带有标签的 `return` 语句只会从对应 `Lambda` 表达式返回，后面代码仍会执行。

```kotlin
testNonReturn {
    if (it.length > 2)
        return@testNonReturn 100
    else return@testNonReturn 1
}

MainActivity: 100
MainActivity: testNonReturn
```

对于内联函数，`Lambda` 表达式会被内联在调用处，那么可以使用 `return` 语句，此时 `return` 的作用是结束 `Lambda` 表达式作为参数的函数所在的那个函数，有点绕，借助下面的例子，`testNonLocalReturn()` 函数接受的 `Lambda` 参数将会结束 `testNonLocalReturn()` 函数所在的 `foo()` 函数，这种返回方式被称为 非局部返回（Non-local return）；

```kotlin
inline fun testNonReturn(f: (String) -> Int){
    f("test")
}

fun foo():Boolean {
    log("before testNonLocalReturn")
    testNonLocalReturn {
        // 返回值为 boolean，将直接退出 foo 函数
        log("invoke testNonLocalReturn")
        return true
    }
    log("after testNonLocalReturn")
    return false
}
```
输出

```bash
MainActivity: before testNonLocalReturn
MainActivity: invoke testNonLocalReturn
```
有些时候 `Lambda` 表达式参数并不一定在函数内部执行，他可能通过一个局部对象或一个嵌套函数传递给其他执行环境调用，此时非局部返回也是被禁止的，为了明确这一点，我们使用 `crossinline` 关键字声明。

```kotlin
// 因为将会在另一个环境执行，使用 crossinline 标记
inline fun testNonLocalReturnV2(crossinline f: () -> Unit): Runnable {
    val r = Runnable { f() }
    return r
}

fun fooV2() {
    testNonLocalReturnV2{
        // error 使用 crossinline 标记，不能非局部返回
        // return
    }
    log("invoke fooV2")
}
```

## 实体化的类型参数(reified)

实体化的类型参数(Reified type parameter)

主要用来解决范型参数无法像普通的类那样访问的问题。写一个函数，判断某个对象是不是属于某种类型。

```kotlin
fun <T> testReifiedType(p: Any, cls: Class<T>): Boolean {
    return cls.isInstance(p)
}

// 我们可以这样使用它
testReifiedType("test", String::class.java)
```
这让我想起了 `Gson` 的使用方法，除了约定一个范型，我们还需要传递对应的 `class` 进去才能解析，`kotlin` 中内联函数支持 `Reified type parameter`，使用  `reifeid` 关键字注解范型，就可以像一个普通的类那样来访问范型类型了。

```kotlin
inline fun <reified T> testReifiedType(p: Any): Boolean {
    return p is T
}

// 调用
testReifiedType<String>("")
```
需要注意的是，函数必须是 `inline` 类型才可以使用 `reifeid` 关键字注解。

