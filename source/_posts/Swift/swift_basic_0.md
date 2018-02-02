---
layout: post
title: Swift基础语法1
categories:
  - Swift
tags: Swift
keywords:
  - Swift
  - basic
abbrlink: 1996349588
date: 2016-03-15 00:00:00
---


1. 从今天开始学习Swift－2016-3-16；

2. 先学习简单语法，不涉及详细API

3. Swift 3.0已经移除i++方法，请使用i+=1

<!--more-->

## 快捷键
```swift
//格式化代码 
ctrl+i
//删除一行 
command+delete
//注释一行
command+/
```

## 一些琐碎
```swift
//swift是一门安全的语言
//不支持隐式类型转换
//不支持空值，nil是一种单独的类型


//数据类型后面带有数字，表示使用几位来表示，Int8最大为127
var int8:Int8
print(Int8.max)

//二进制，八进制，16进制的表示
var two = 0b111
var eight = 0o111
var sixTeen = 0x111
print("\(two)   \(eight)  \(sixTeen)")

//下划线表示忽略,更清晰的数据定义方式
var bigNum = 1_000_000
var _ = 100

//轻量级的数据聚合，元组，元组可以存储任意类型的数据
//指定类型
var yuanzu0:(Int,String,Int,String) = (100,"0909",30,"222")
print("\(yuanzu0.1)")
//不指定类型，使用下标访问
var yuanzu1 = (100,"0909",30,"222")
print("\(yuanzu1.1)")
//标志位，访问
var yuanzu2 = (x:100,y:"aaa")
print("\(yuanzu2.x)")
//解包访问，不关心的数据可以使用_代替
var (m,_) = yuanzu2
print("\(m) ")
```
## 常量和变量
```swift
print("hello world");
//常量(let)和变量(var)，未赋值的变量常量显示声明类型,赋值的变量常量会自动推断数据类型
let contast = 1;
print(contast)
var a:Int//10进制
a = 1000
var str = "this is a str"
        
//使用\(变量常量)可以直接打印变量常量出来
var rst = "this is a rst = \(a)"
print(rst)
```

##数组和字典
```swift
//定义数组，使用［］访问元素
var list = [1,2,3,4,5];
print(list);
print(list[0]);

//定义字典，使用key访问
var map = ["a":1,"b":2,"c":3];
print(map);
print(map["a"])
print(map["a"]! + 100)
      
//如果列表和字典的类型可以被推断出来，可以不使用类型，也不用带（），在下面，shoplist首先被声明为String类型的数组，后又重新赋值，此时可以推断shoplist 是［String］
var shoplist = [String]()
shoplist = []

//跟常量和变量的定义一样，如果类型可以被推断时，可以不使用类型声明，上面的list和map类型已经可以推断，当将它重新指向空的数组和字典
list = []
map = [:]
        
//定义空的数组和字典
var emptyList = [String]()
var emptyMap = [String:Int]()
print(emptyList.count)
print(emptyMap.count)
```

## 控制流
### for
```swift
let testList = [1,2,3,4,5,6,7,8];
//简单for循环
for var i=0;i<8;i+=1 {
    print("for this is \(testList[i])")
}
//for in 循环
//注意这里的x是常量，不能修改它的值，所以下面（1）是错误的
for x in testList{
    print("for in this is \(x)")
    //(1)
    x = 1
}

//遍历字典中的数组
let map2scan = [
    "a":[1,2,3,4,5],
    "b":["a","b","c"]
]

       
for (key,value) in map2scan{
     print("key is \(key)")
     for num in value{
     	print("num is \(num)")
  }
}

```

### if关键字
```swift
//涉及可选型的概念，后面单独介绍
var optionalStr:String? = "hello"
optionalStr = "world"
print(optionalStr == nil)
        
var greet = "hey"

if let name = optionalStr{
    greet = "hello \(name)";
}
print(greet)
        
if(optionalStr != nil){
    print("optional str is not nil")
}

if score >= 10{
}
```

### switch关键字
```swift
//swift支持任意数据类型的switch比较，不仅限于Int和enum
//break语句,不需要显式添加，默认语句后面都会break;
//default语句,是不可以省略的,除非所有的值都被穷举出来了；
//fallthrough语句，当满足某个case之后仍旧想使它匹配下一个case，使用 fallthrough，则不会被截断
let vegetable = "red peper"
switch vegetable{
//因为每个case之后都会有break，所以每个case之后必须至少有一行可执行语句，当需要多个匹配时，可以像下面这样
case "b","B"
	  print("this is b/B")
case "a":
     print("this is a")
     fallthrough
case let x where x.hasSuffix("peper"):
     print("has suffix " + x)
default:
     print("default")
}

//高级用法
//区间
let num = 100
switch num{
	case 0 ..< 100:
		print("小于100")
}

//元组
let point = (1,1)
switch point{
	case (_,0):
		print("x aliaxs");
		fallthrough
	case (0,_):
		print("y aliaxs")
		fallthrough
	case (0,0):
		print("origin point")
		fallthrough
	case (-2 ... 2,-2 ... 2):
	print("near by origin point")
}
```


### while/repeat...while
```swift
//while/repeat...while循环, 当while语句成立时，语句体会执行。
var num = 0;
while num < 100{
    num+=1
}
print("while num is \(num)")
        
repeat{
    num-=1;
}while num > 0
print("repeat num is \(num)")
```


## 操作符
### 可选值操作符(??)
```swift
//有点类似三目运算符,??表示默认值,当前面的值为空时将会使用后面的
let nickName?String = "a"
let fullName = "b"
let name = "hello \(nickName ?? fullName)"
print(name)
```

### 范围操作符(..>／...)
```swift
//提供一种更简单实现循环的方式,...包含上界

//0 1 2 3
for x in 0..<4{
    print("x is \(x)")
}

//0 1 2 3 4
for x in 0...4{
    print("x is is \(x)")
}
```

### 强制解包操作符(!)
```swift
//强制解包 unwrap 解包的概念会在可变型中介绍
var num = 100
var name = "this is " + num!
```
## 函数和闭包

### 简述
```swift
函数是一个可以被抽取调用的封闭代码块，是可以被传递的数据类型 
```
### 一个简单函数
```swift
//函数使用func关键字命名
//使用  [参数名:参数类型]  ...的方式定义参数
//调用函数时，要用  [参数名:值]  的方式传递
//func 函数名（参数列表（参数名:参数类型））-> 返回类型  {// 函数体｝
func firstFun (name:String,pwd:String,newParams:String)->Bool {
     print("name is \(name) and pwd is \(pwd)")
     return true
}
firstFun("chendong",pwd:"1234567",newParams:"new params");
```

### 返回多个值
```swift
//可以使用元组返回多个值，实际上是以元组作为值传递的方式
//使用元组实现多个返回值
//元组可以使用键访问，rst.max,也可以使用下标访问，rst.2
func getMultiBackFun(scores:[Int])->(max:Int,min:Int,sum:Int){
     var sum = 0
     var min = scores[0]
     var max = scores[0]
     for score in scores{
     	sum += score;
   	    if(score > max){
    	     max = score;
    	 }
     	if(score < min){
       	  min = score
     	}
     }
     return (max,min,sum)
}
let rst = getMultiBackFun([4,8,1,7,4,0,6,3,7])
print("max is \(rst.max)  min is \(rst.min)  sum is \(rst.sum)  rst.2 is \(rst.2)")
```

### 可变长度参数函数
```swift
//跟所有的语言一样，支持可变长度参数，本质是数组的传递
//一个求均值的函数
func changeParamFun(params :Int...)->Void{
     var sum = 0;
     for x in params{
         sum+=x
     }
     print("averge is \(sum/(params.count))")
}
        
changeParamFun(1,2,3,4)
changeParamFun(0,9,8,7)
```

###函数嵌套定义
```swift
//函数可以嵌套定义,被嵌套的函数作用域有限，只能在嵌套它的｛｝中被访问，比如下面（1）中的代码是编译错误的，被嵌套的函数可以访问外面函数的变量或者全局变量，
func aOutFun() -> Void{
            var num = 100
            func aInFun()->String{
                return "chendong \(num)"
            }
            print(aInFun())
}
aOutFun()
//1 编译错误
aInFun()
```

### 函数作为返回值
```swift
//函数作为返回值,下面的函数表示一个无参函数，返回值是一个（参数为Int,返回值是String的）函数类型，在函数returnFuncFun中定义内嵌函数并返回，在外部可以使用该函数
func returnFuncFun()->(Int->String){
     func INTSTRING(num:Int)->String{
          return "this num is \(num)"
     }
     return INTSTRING
}
let funINTSTR = returnFuncFun();
print("func return is \(funINTSTR(1000))")
```


### 函数作为参数传递
```swift
//函数作为参数，下面的函数表示一个参数为Int类型，一个参数为（Int->String）的函数类型，返回值String
func funParamFun(num:Int,param : Int -> String)->String{
     return "funparamfun + \(param(num))"
}

func paramFunc(num:Int)->String{
     return "paramfunc + \(num)"
}

print("rst is \(funParamFun(100, param: paramFunc))")

//结果是  rst is funparamfun + paramfunc + 100
```

### 匿名函数闭包
```swift
//匿名闭包（in 前面是参数类型，in后面的是函数体）
let numbers = [1,2,3,4,5];
let get = numbers.map({
	（in 前面是函数参数和返回值，in后面的是函数体）
    (number:Int)->Int in
    let rst = number*2;
    return 3 + rst;
})
print("rst is \(get)")

//如果一个闭包的类型已知，比如作为一个回调函数，你可以忽略参数的类型和返回值。单个语句闭包会把它语句的值当做结果返回.
numbers.map({
    number in number * 3
})

//使用参数位置代指参数       
numbers.sort({$0>$1})
let get2 =  numbers.map({$0*2})
print("numbers is \(get2)")
```