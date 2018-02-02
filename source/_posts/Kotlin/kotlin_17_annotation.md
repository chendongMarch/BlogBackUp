---
layout: post
title: Kotlin开发-17-注解
categories:
  - Kotlin
tags:
  - Kotlin
keywords:
  - Kotlin
  - 注解
abbrlink: bc7a3f35
date: 2017-09-06 14:24:00

---

本文主要学习 `Kotlin` 注解的相关内容。



<!--more-->

## 源注解

**@Target** : 指定这个注解可被用于哪些元素(类, 函数, 属性, 表达式...)

```kotlin
@Target(
        AnnotationTarget.ANNOTATION_CLASS, // 注解类
        AnnotationTarget.CLASS, // 对象类，接口，也包括注解类
        AnnotationTarget.CONSTRUCTOR, // 构造器，主构造器或次级构造器
        AnnotationTarget.EXPRESSION, // 表达式
        AnnotationTarget.FIELD, // 属性，包括 backing-field
        AnnotationTarget.FILE, // 文件
        AnnotationTarget.FUNCTION, // 函数，也包括构造器
        AnnotationTarget.LOCAL_VARIABLE, // 局部变量
        AnnotationTarget.PROPERTY, // 属性
        AnnotationTarget.PROPERTY_GETTER, // 属性 get 方法
        AnnotationTarget.PROPERTY_SETTER, // 属性 set 方法
        AnnotationTarget.TYPEALIAS, // 类型别名
        AnnotationTarget.TYPE, // 类型
        AnnotationTarget.TYPE_PARAMETER, // 范型参数类型
        AnnotationTarget.VALUE_PARAMETER //  值类型参数
)
```
**@Retention** : 注解信息在 class 文件中的保存和访问限制

```kotlin
@Retention(
        AnnotationRetention.SOURCE // 注解信息不保存 class 文件，不能反射访问
        // AnnotationRetention.BINARY // 注解信息保存 class 文件，不能反射访问
        // AnnotationRetention.RUNTIME // 注解信息保存 class 文件，能反射访问
)
```
**@Repeatable** : 允许在一个元素上多次使用同一注解

**@MustBeDocumented** : 表示这个注解是公开 `API` 的一部分，在自动产生的 `API` 文档的类或者函数签名中，应该包含这个注解的信息。

##  注解和使用

注解的声明使用 `annotation` 关键字，类似一个类的声明，在上面添加源注解对定义的注解进行描述。

```kotlin
@Target(AnnotationTarget.ANNOTATION_CLASS)
@Retention(AnnotationRetention.SOURCE)
@Repeatable
@MustBeDocumented
annotation class AnnoSimple
```
使用注解，注解可以添加在很多地方，取决于声明 `@Target` 中支持的类型

```kotlin
// 类注解
@AnnoSimple class AnnoTestA
// 构造器注解
@AnnoSimple constructor(name: String) {

    // 函数注解
    @AnnoSimple fun test() {
        // lambda 表达式注解
        val f = @AnnoSimple { log("test") }
    }

    // 属性注解
    @AnnoSimple var property: Int = 100
        // get 方法注解
        @AnnoSimple get() = 10
        // set 方法注解
        @AnnoSimple set(value) {
            field = value
        }
}
```

## 注解参数

注解可以添加参数，参数支持 基本数据类型，字符串，KClass，枚举，其他注解，由以上数据类型构成的数组。

```kotlin
@Target(AnnotationTarget.CLASS,AnnotationTarget.FIELD)
@Retention(AnnotationRetention.SOURCE)
annotation class AnnoWithParam(val msg: String, val cls: KClass<*>)
```

注解同样可以作为注解的参数，如果一个注解被用作另一个注解的参数, 那么在它的名字之前不使用 `@` 前缀

```kotlin
annotation class AnnoWithAnotherAnno(val msg: String,
                                     val anno: AnnoWithParam = AnnoWithParam("test", Int::class))
```

使用注解

```kotlin
@AnnoWithParam("test", Int::class) class AnnoTestB
```

## 注解的使用目标


当你对一个属性或一个主构造器的参数添加注解时，从一个 `Kotlin` 元素会产生出多个 `Java` 元素，因此在编译产生的 `Java` 字节码中，你的注解存在多个可能的适用目标。 

支持的注解目标

```bash
file
property (使用这个目标的注解, 在 Java 中无法访问)
field (域变量)
get (属性的 get 方法)
set (属性的 set 方法)
receiver (扩展函数或扩展属性的接受者参数)
param (构造器的参数)
setparam (属性 set 方法的参数)
delegate (保存代理属性的代理对象实例的域变量)
```

为了明确指定注解应该使用在哪个元素上，可以使用以下语法：

```kotlin
class AnnoTestC(
        @field:AnnoSimple val name: String, // 对域变量注解
        @param:AnnoSimple val age: Int, // 构造器参数注解
        @get:AnnoSimple val cls: KClass<*> // get方法注解
)
```

当对同一注解目标使用多个注解时，可以使用 `@set[anno1 anno2]` 的语法

```kotlin
class AnnoTestD(
        @field:[AnnoSimple AnnoWithParam("test", Int::class)] val name: String
)
```

对扩展函数使用注解

```kotlin
fun @receiver:Fancy String.myExtension() { }
```

