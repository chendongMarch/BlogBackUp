---
layout: post
title: Swift基础2
categories:
  - Swift
tags: Swift
keywords:
  - Swift
  - basic
abbrlink: 1243741076
date: 2016-05-11 00:00:00
---

## 可选型的概念
- 数据类型＋？将会构成一种新的类型，可选型，String?为字符串可选型，，在swift中，空的概念略有不同，java中，如果一个对象变量，没有指向，为null；一个int类型的变量如果没有初始化，值为0，null和0 意为没有，在swift中，空是一种完全独立的数据类型，nil的位置和Int，String的地位是一样的，他不是对象或者数据的一种特殊状态，而是一种新的对象。这也就意味着代码中(1)的部分是编译错误的，会提示，nil类型是不能和String类型进行的比较的，该设计的目的是为了使swift更具安全性，因为任何对象都不能设置为nil,就如同String类型的数据不能设置为int类型的数据，他们是完全不同的对象，可以避免潜在空指针的发生。


```swift
//(1)
var str = "abc"
if(str == nil){

}


//String? 为String可选型，他意味着这种类型，可以为nil,还是那句话，可选型和nil不是一个类型的特殊状态，他是一种新的类型。

//2
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
```


## where与模式匹配
```swift
//switch中使用where语句
let point = (3,3)
switch point {
	case let(x,y) where x == y:
		print("x == y")
	case let(x,y) where x == -y:
		print("x == -y")
 	default:
 		print("rst is \(point.0) , \(point.1)")
}

//switch中使用运算符    
let age = 19
switch age{
	case 10 ... 19:
		print("teenager")
	default:
		print("not teenager")
}
     
//if中使用模式   
if case 10...19 = age where age > 18{
	print("teenager and in colldge")
}
 
//if中使用模式和where语句
let vector = (4,0)
if case (let x,0) = vector where x > 2 && x < 5{
	print("it is vector")
}
 
//if中使用模式＋where+运算符       
for case let i in 1 ... 100 where i % 3==0{
	print("i is \(i)")
}
```

## guard关键字
```swift
//guard else 防止数据错误，也可以认为是需要满足的一种先决条件
//例如下面money>price,capacity > volume是必须满足的条件，简化语法，防止错误
func buy(money:Int,price:Int,capacity:Int,volume:Int) -> Bool{
	guard money > price else {
		print("not money")
		return false
	}
	guard capacity > volume else{
		print("not volume")
		return false
	}
	return true
}   
```

## String类型
```swift
//字符串
var originStr = "this is str"
originStr = "this is new str"
//定义一个空字符串
let emptyStr = ""
//判空操作
print(emptyStr.isEmpty)
//合并字符串
var rstStr = originStr + emptyStr
//使用+=,此时rstStr必须是变量
rstStr += originStr
//字符串插值
rstStr = "this is new str and insert \(100)"
//转义字符 \

//遍历字符串
for c in originStr.characters{
	print(c)
}
        
        
//Character
let cc:Character = "!"
originStr.append(cc)
        
//字符串长度，基于unicode码，也就是说不管是3个汉字或者3个字母，它的count都是3
let englishLetter = "abc"
let chinseLetter = "慕课网"
let emojiLetter = "😢😢😢"
let unicodeLetter = "\u{1f60e}\u{0301}"
print(englishLetter.characters.count)
print(chinseLetter.characters.count)
print(emojiLetter.characters.count)
print("enlish is \(englishLetter) chinse is \(chinseLetter)  emoji is \(emojiLetter) unicode is \(unicodeLetter)")
        
        
//索引访问字符串，需要使用String.Index类来访问
//[startIndex,endIndex)
let startIndex = originStr.startIndex
let endIndex = originStr.endIndex
print("index is \(startIndex) and content is \(originStr[startIndex])")
//向后n个
startIndex.advancedBy(6)
//前面一个位置
endIndex.predecessor()
//后面一个位置
startIndex.successor()

        
//String的一些API
//Range<Index>类型
let range = startIndex.advancedBy(3) ..< endIndex
originStr.replaceRange(range, with: "!!!")
originStr.appendContentsOf("xxx")
originStr.insert("z", atIndex: originStr.endIndex)     originStr.removeAtIndex(originStr.endIndex.predecessor())      originStr.removeRange(originStr.endIndex.advancedBy(-2)..<originStr.endIndex)
        
print("upper \(originStr.uppercaseString) low \(originStr.lowercaseString) First up \(originStr.capitalizedString)")
originStr.containsString("")
originStr.hasSuffix("")
originStr.hasPrefix("")
       
//NSString 
let s2 = NSString(format: "one third is %.2f",1.0/3.0)
print("转换 \(s2 as String)")
        
let nsStr:NSString = "one third is 0.33"
nsStr.substringFromIndex(4)
nsStr.substringFromIndex(3)
nsStr.substringWithRange(NSMakeRange(4, 5))
        
let s6 = "   --- Hello ---   " as NSString
//去掉空格和－
print(s6.stringByTrimmingCharactersInSet(NSCharacterSet(charactersInString:" -"))
```