---
layout: post
title: Java 范型
categories:
  - Study
  - Java
tags: Java
keywords:
  - Java
  - 范型
abbrlink: 2053934834
date: 2016-07-20 00:00:00
---

关于如何继承带有范型的基类
<!--more-->
## 继承父类的范型

```java
public class GenericityTst {
	
	public class Temp {
	
	}
	
	//具有泛型的ClassA
	public class ClassA<T>{
		T t;
	}
	
	//继承泛型
	public class ClassB<T> extends ClassA<T>{
		T t;
	}
	
	//缩小泛型的范围，是准许的，但是不允许扩大泛型的范围
	public class ClassC<T extends Temp> extends ClassA<T>{
		T t;
	}
	
	//具有泛型的类ClassD,泛型具有父类约束
	public class ClassD<T extends Temp> {
		T t;
	}
	
	//继承ClassD,声明泛型时至少具有父类同样的约束
	public class ClassE<T extends Temp> extends ClassD<T>{
		T t;
	}
}
```