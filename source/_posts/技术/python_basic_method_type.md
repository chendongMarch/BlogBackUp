---
layout: post
title: Python普通方法、静态方法、类方法
date: 2016-08-11
category: 技术
tags: Python
keywords: 
description: Python普通方法、静态方法、类方法
---


## 开始

```python
# -*-coding:utf-8-*-
# 普通方法,类方法,静态方法的区别

__metaclass__ = type


class Tst:
    name = 'tst'

    data = 'this is data'

    # 普通方法
    def normalMethod(self, name):
        print self.data, name

    # 类方法,可以访问类属性
    @classmethod
    def classMethod(cls, name):
        print cls.data, name

    # 静态方法,不可以访问类属性
    @staticmethod
    def staticMethod(name):
        print name
```

## 测试

- 三种方法都可以通过实例来调用，但是静态方法和类方法无法访问实例属性，所以更改了tst.data仅对普通方法起了作用


```python
tst = Tst()
tst.data = 'this is new'
tst.normalMethod('name')
tst.staticMethod('name')
tst.classMethod('name')

#结果
this is new name
name
this is data name
```
## 区别

- 普通方法不能通过类名调用，但是静态方法和类方法是可以的

```
# error普通方法必须通过实例调用
# Tst.normalMethod('name')
Tst.classMethod('name')
Tst.staticMethod('name')

#结果
this is data name
name
```

## 总结
- 普通方法,可以通过self访问实例属性


```python
def normalMethod(self,data)
```

- 类方法,可以通过cls访问类属性


```python
@classmethod
def classMethod(cls,data)
```

- 静态方法,不可以访问,通过传值的方式


```python
@staticmethod
def staticMethod(data)
```