---
layout: post
title: Js 开发-1(基础)
categories:
  - JavaScript
tags:
  - JavaScript
keywords:
  - JavaScript
comments: true
abbrlink: 22324dc8
date: 2017-07-05 22:30:11
password:
---

 

<!--more-->
`Js` 是弱类型动态语言

## 内建数据类型

object (Func,Array,Date...)
number
string
boolean
null
undefined

`+` : 只要有字符串就是字符串拼接
`-` : 如果有字符串，转换为数字计算


`==` : 等于，类型相同，同 ===，类型不同，进行转换，null == undefined, number == string(转换为数字比较)，boolean == number(boolean转换为number),object == number/string(尝试将obj转为基本类型)
`===` : 严格等于，类型不同，返回 false，类型相同 null === null,undefined === undefind,NaN != NaN,new Object != new Object

