---
layout: post
title: Apt 开发语法
category: Android
tags:
  - Android
abbrlink: af68580c
keywords:
  - APT
  - 编译时注解
location: 青岛
photos: >-
  https://images.pexels.com/photos/1684915/pexels-photo-1684915.jpeg?auto=compress&cs=tinysrgb&dpr=2&w=500
date: 2019-01-17 09:14:00
---



APT 语法


public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv)


Element 表示元素

TypeElement 类

VariableElement 成员



- 从成员变量中获取类型, 转变为参数类型

```java

TypeName typeName = TypeName.get(variableElement.asType());
builder.addParameter(typeName, "name");
```

- 获取包名，用于在同包下生成代码

```java
processingEnv.getElementUtils().getPackageOf(typeElement).getQualifiedName().toString();
```