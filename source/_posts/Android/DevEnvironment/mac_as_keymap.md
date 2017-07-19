---
layout: post
title: Mac + AndroidStudio 快捷操作
categories:
  - Android
tags:
  - Android
  - DevEnvironment
keywords:
  - Android
  - AndroidStudio
  - 快捷键
abbrlink: 8d16bcb4
date: 2015-08-05 00:00:00
---

## 快捷键

### 常用

```bash
- 快速提示
alt + Enter

- 跳转到某一行
command + L

- 整理代码
alt + command + L

- 重命名变量
shift + F6


- 代码大小写转换
shift + command + u

- 列选中
alt + 鼠标
```
  

### 提取

```bash
- 提取代码块为方法
alt + command + M (method)

- 提取局部变量为全局变量
alt + command + F (field)
```

 
### 搜索／替换

```bash
- 搜索 java 文件
command + O
- 搜索所有文件
shift + command + O 

- 文件内查找文本
command + F
- 项目内查找文本
shift + command + F
- 文件内文本替换
command + R
- 项目内文本替换
shift + command + R

- search everywhere 
双击 shift
```
 

### project 操作

```bash
- 打开 ProjectStructure
command + ;(分号)
- 打开 settings
command + ,(逗号)

- 运行项目
control + R

- 运行 freeline
shift + command + F10
```


## 代码快捷用法

### 通用

```java
User user = new User();

// 判断不为空; user.nn
if (user != null) {
}

// 判断为空; user.null
if (user == null) {
}

// 返回; user.return
return user;

// instanceof 判断; user.inst
List temp = user instanceof List ? ((List) list) : null;

// 生成条件表达式; 条件表达式.if 如输入 100>10.if
if (100 > 10) {
            
}

// 生成while循环; 条件表达式.while 如输入 100>10.while
while (100 > 10) {
            
}

```
### 快速遍历

```java
List<String> list = new ArrayList<>();

// list.for
for (String s : list) {
    
}

// list.fori
for (int i = 0; i < list.size(); i++) {
    
}

// list.forr
for (int i = list.size() - 1; i >= 0; i--) {
    
}
```