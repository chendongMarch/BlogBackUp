---
layout: post
title: Python开发-1-基础
categories:
  - Python
tags: Python
keywords:
  - Python
  - basic
abbrlink: 1811789999
date: 2016-01-21 00:00:00
---


## 前言
- 我使用的Pycharm编译器，刚开始就遇到了一个问题，不能输入中文，解决办法就是在文件头部添加代码`#-*- coding:utf8 -*-`


## Python交互
```python
# 使用python交互模式

# 在命令行界面输入｀python｀进入交互模式

# ｀help()｀进入帮助模式
# 尝试｀keywords｀获取关键字帮助
#｀help("modules")｀查看modules
# ｀quit｀ 退出帮助模式

#dir(object) 它返回传递给它的任何对象的属性名称经过排序的列表
#dir()当前倒入的模块


#下面以内建函数id为例
#获取文档介绍,｀id.__doc__｀
#获取名称,并不是所有的都有`id.__name__`属性
#`hasattr(dir,'__doc__')`,`getattr(dir,'__doc__')`获取一个对象的属性
#callable可调用性`callable("a")` false,`callable(dir)`  true
#`isinstance("python", str)`,`issubclass(child.father)`
```
## 数据类型
```python
# 列表(同类型数据构成的数组)
list ＝ ［1，2，3，4］
# 元组（数据类型可以不同，简单方便的数据聚合，类似结构体）
tuple = (1,"second",1.2)
# set（数据不重复，使用list构建，无序，不能使用索引访问）
set = set([1,2,3,4])
# 字典（键值对存储数据）
dict ＝ ｛"a":"b","c":"d"｝
```
## 基础语法
```python
# 打印
# 自动追加换行符
print "hello world"
# 将不会打印换行符号
print "hello world",
＃ 使用匹配符
num = 100
num2 = 200
print "num is %d"%num
print "num is %d,num2 is %d"%(num,num2)


# 内建函数id()和type()
a = 3
# 获取内存地址
print "id(a) is ", id(a)
# 获取变量类型
print "type(a) is ", type(a)


# 运算符(两个比较特殊的)
# 级数乘2的3次方
print "use ** = ", 2 ** 3
# 除后取整
print "use // = ", 9 // 4



# 逻辑运算 and | or | not
if a == 0 and b == 0:
    a = 2



# for in  循环
for i in [1, 2, 3, 4]:
    print i,



# 模块的导入的几种方式
import math
from math import pow
from math import pow,abs
from math import *
print pow(10, 3)
from math import pow as pingfang
print pingfang(10, 3)



# 赋值
# 多个赋值可以直接顺序写，会挨个自动赋值
x, y, z = 1, "abc", [1, 2, 3]
print x, y, z
# 自动创建元组，将后面的数据,装入元组
xx = 1, 2, 3, 4, "qqq"
print xx



# 交换数据的方式
yy, zz = 3, 4
print yy, zz
yy, zz = zz, yy
print yy, zz



# 链式赋值,两个变量指向统一空间,id()一样的
m = n = 123
print m, n
print m is n


# 判断结构
xxx = 1
if xxx == 1:
    print "this is test ", 1
elif xxx == 2:
    print "this is test ", 2
else:
    print "this is test ", 100



#获取控制台输入
inputNum = raw_input()



# 三元运算
# A = X if B else Y
# 如果B为真,执行A = X,否则执行A = Y
mm, nn = 1, 2
rstStr = "abc" if mm > nn else "def"
print rstStr



# for循环和内建函数range(start,stop,step),range(start,stop,1),range(0,stop,1)返回的是数字元素的列表start开始,stop结束,每次增加step个
strx = "abcdefg"
for i in range(len(strx)):
    print strx[i],
range(9)
range(3, 9)
range(3, 9, 4)



＃ 一个可迭代的对象可以直接转换为序列对象
print list(strx)
print set(strx)
print tuple(strx)



# 遍历字典
dictStr = {"a": "b", "c": "d"}
# 不建议使用,效率相对低
for key in dictStr:
    print "1key  value is ", key, dictStr[key]
# 遍历key
for key in dictStr.keys():
    print "2key  value is ", key, dictStr[key]
    
for key, value in dictStr.items():
    print "3key  value is ", key, value
# 更优
for key, value in dictStr.iteritems():
    print "4key  value is ", key, value   
# 单独取value使用下面的方法效率更高
for value in dictStr.values():
    print "5value is ", value



# 内建函数zip(),将两个序列数据类型,每一项取出来,合并为一个元组,构成元组的list
str1 = "abcdef"
str2 = "12345"
print zip(str1, str2, str2)
# [('a', '1', '1'), ('b', '2', '2'), ('c', '3', '3'), ('d', '4', '4'), ('e', '5', '5')]



# 交换key,value,每个item是两个元素的tuple的list可以直接转换为dict
dict1 = {"a": "1", "b": "2", "c": "3"}
print dict(zip(dict1.values(), dict1.keys()))



# 内建函数enumerate()生成的是（item为下标和元素构成的两元元组）的可迭代对象，可以直接转换为列表
mList = ["a", "b", "c", "d", "e"]
for (i, j) in enumerate(mList):
    print i, " -- ", j
print list(enumerate(mList, start=1))
#[(1, 'a'), (2, 'b'), (3, 'c'), (4, 'd'), (5, 'e')]


# 替换字符串中的字符,字符串正则分割
originStr = "Do u like Canglaoshi? Canglaoshi is a good teather!"
splitStrs = originStr.split(" ")
for i, s in enumerate(splitStrs):
    if "Canglaoshi" in s:
        splitStrs[i] = "Python"



# list解析,去掉元素前后空格,list解析可以简单快捷的修改每一个元素，生成新的list
originList = [" abc", " vbf ", "rtg "]
afterList = [one.strip() for one in originList]
print afterList

# list切片[start = 0,stop = len(list),step = 1],一个简单从list提取部分数据的方式
list2cut = [1,2,3,4,5,6,7]
listRst = list2cut[0:5:2]
print "list2Rst is " ,listRst


# 一个猜数字的小程序，简单介绍循环结构和判断结构的简单实用
import random
number = random.randint(1,100)
while(True):
    print "请输入一个数字,猜测约定的数字是多少?"
         input = raw_input()
     if (int)(input) == number:
         print "正确!"
         break
     elif (int)(input) < number:
         print "太小了!"
     else:
         print "太大了!"


# for... else 和 while... else
# 跳出循环结构之后执行else语句
for i in range(1,100):
    print i,
else:
    print "end"


# 迭代
list2Iter = [1,2,3,4,5,6]
iterIt = iter(list2Iter)
# 这里会提示一个警告
while True:
	print iterIt.next()



# 文件迭代
#f其实是一个可以迭代的对象,可以使用列表操作
list2File = [line for line in open(fileName)]
print "list2File is ",list2File
# 可以更简单
list2File = list(open(fileName))
print "list2File2  is ",list2File
```

## 文件简单操作
```python
文件操作
fileName = "/Users/march/123.txt"
# 以一种模式打开文件获取可迭代索引
file = open(fileName, "rw")
for line in file:
    print line,
file.close()

#打开文件的模式
r	以读方式打开文件，可读取文件信息。
w	以写方式打开文件，可向文件写入信息。如文件存在，则清空该文件，再写入新内容
a	以追加模式打开文件（即一打开文件，文件指针自动移到文件末尾），如果文件不存在则创建
r+	以读写方式打开文件，可对文件进行读和写操作。
w+	消除文件内容，然后以读写方式打开文件。
a+	以读写方式打开文件，并把文件指针移到文件尾。
b	以二进制模式打开文件，而不是以文本模式。该模式只对 Windows 或 					   Dos 有效，类 Unix 的文件是用二进制模式进行操作的。


# 不需要关闭的安全方法
with open("/Users/march/123.txt", "a") as ff:
     ff.write("this is new line")

# 获取文件状态
import os
fileStatus = os.stat(fileName)
print fileStatus
import time
print time.localtime(fileStatus.st_ctime)


# 指定size时读取指定size的数据,否则读取全文，下面的所有方法都遵循这一规则
file = open(fileName)



# 读取字节
file.read()
file.read(100)
# 读取一行
file.readline()
file.readline(100)
# 读取所有数据存储在列表中
file.readlines()
file.readlines(100)
# 移动指针
file.seek(10)
# 当前指针的位置
file.tell()



# 另外一种简化的读取文件的方式
import fileinput
for line in fileinput.input(fileName):
    print "use fileinput print is " , line
```
## 列表操作
```python
charList = ['1A','2','3','4','5','6','7','8','9','10','11','11']
strList = ["1","2","3","4","5","6","7","8","9","10","11","11"]
charList.append("new_a")
#在指定位置添加元素,指定位置之后的元素向后移动一个,长度+1
charList.insert(1,12)
#当前元素在列表中的位置
print("pos of 3 ",charList.index("3"))
#当前元素在列表中出现的次数
print("total num of 11 ",charList.count('11'))
#将当前字符串\列表(可迭代类型的数据)拆分成单个字符添加在末尾，返回值是none
print(charList.extend(strList))
#从末尾弹出一个元素,或从指定位置弹出,返回值 是该元素
print(charList.pop(2))
#移除指定元素,不能使用下标做参数
charList.remove('1A')
#列表反向
charList.reverse()
#针对第一个字母或数字排序,更多参数比较复杂
charList.sort()
```
