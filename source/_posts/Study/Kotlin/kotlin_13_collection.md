---
layout: post
title: Kotlin开发-13-集合
categories:
  - Kotlin
tags:
  - Kotlin
keywords:
  - Kotlin
  - 集合
  - Collection
abbrlink: 4c59f5f5
date: 2017-08-08 20:14:00

---

在 `Java` 中集合类型也是很重要的数据结构，它可以用来存储一组同类型的数据；

在 `Kotlin` 中将集合明确分为了可变的集合和不可变的结合，`List<out T>` 是一个只读的接口，只能执行的 `get()`，`size()` 等读取方法，如果想要更改集合的数据，需要使用 `MutableList<T>`，从而区分可辨与不可变集合。`Set<out T>` 和 `MutableSet<T>`、`Map<K,out V>` 和 `MutableMap<K,V>` 也是同样的模式。

相比 `Java`，`Kotlin` 中的集合除了包含 `Java` 中的集合的相关方法，还扩展了很多简化集合操作的函数，结合 `Lambda` 表达式，把一些对集合的操作变的相当相当方便。

本文主要是对集合类的各种函数进行尝试和整理。


<!--more-->


## 创建和访问

在 `Kotlin` 中，`List<out T>` 是只读的，因此只能访问它的 `get()` 方法，如果需要改变元素的值，需要使用 `MutableList<out T>`，`Kotlin` 中列表的元素可以像数组那样使用 `list[index]` 的形式访问。

```kotlin
// 创建一个可变列表
val mutableList = mutableListOf(1,3,4)
// 创建一个只读列表
val readOnlyList = listOf(1,2,3)
```

读写元素


```kotlin

val item = list[0]

// mutable
list[0] = "1"
```

在 `Kotlin` 中也针对元素的读写扩展了很多简化的方法


```kotlin
// index 在范围内返回值，否则返回 lambda 表达式的结果
list.getOrElse(1) { "$it" }
// index 在范围内返回值，否则返回 null
list.getOrNull(1)

// 一些类似的函数
list.elementAt(2)
list.elementAtOrNull(2)
list.elementAtOrElse(2) { "$it" }
```

获取下标

```kotlin
// 该元素出现的最后一次的下标
list.lastIndexOf("10")

// 该元素出现的第一次的下标
list.indexOf("10")

// 从开始第一个开始满足条件的下标
list.indexOfFirst { it.length > 10 }

// 从最后一个开始满足条件的下标
list.indexOfLast { it.length > 10 }
```

查找第一个满足条件的元素

```kotlin
// 查找第一个满足条件的，找不到就抛异常
list.first { it.length > 10 }
// 如果列表为空返回 null,否则返回第 0 项
list.firstOrNull()
// 查找第一个满足条件的，找不到返回 null
list.firstOrNull { it.length > 10 }
```

从后向前查找第一个满足条件的元素

```kotlon
// 从后向前查找第一个满足条件的，找不到就抛异常
list.last { it.length > 10 }
// 如果列表为空返回 null,否则返回第 size-1 项
list.lastOrNull()
// 从后向前查找第一个满足条件的，找不到返回 null
list.lastOrNull { it.length > 10 }
```

## 遍历

集合的遍历是个老话题，`Kotlin` 中集合的遍历更优雅，更强大。

forEach 循环

```kotlin
// 只关注 value
list.forEach { Log.e("test", "value is $it") }

// 带有下标的遍历
list.forEachIndexed {
    index, value ->
    Log.e("test", "index is $index value is $value")
}
```

迭代器遍历，相当于使用 `Java` 中的 `Iterate`	进行遍历。

```kotlin
// 迭代器遍历
for (value in list.listIterator()) {
    Log.e("test", "value is $value")
}

// 从第 n 个开始
for (value in list.listIterator(10)) {
    Log.e("test", "value is $value")
}
```

for...in 循环

```kotlin
// 遍历下标
for (i in list.indices) {
    Log.e("test", "value is ${list[i]}")
}

// 遍历值
for (value in list) {
    Log.e("test", "value is $value")
}

// 值和下标同时遍历
for ((index, value) in list.withIndex()) {
    Log.e("test", "index is $index value is $value")
}
```

遍历一个列表时，如果该列表是基于数组结构的，那么会具有数组的特点，访问容易但是插入相对困难，如果列表是基于链表结构的，那么会具有链表的特点，访问困难但是插入相对容易；遍历数组时可以根据此特点进行优化。


```kotlin
if (list is RandomAccess) {
    for (index in list.indices)
        Log.e("test", "value is ${list[index]}")
} else {
    for (value in list.listIterator())
        Log.e("test", "value is $value")
}
```

## 检测元素

这部分函数主要用来简化我们需要遍历集合得到一个 `boolean` 结果的情况，在 `Java` 中我们通常需要写一个循环然后挨个判断每个元素达到结果。在 `Kotlin` 中借助 `Lambda` 表达式，我们可以像操作一个整形变量一样操作集合。

```kotlin
// 如果全都不满足条件返回 true
if (list.none { it.length > 10 }) {
}

// 如果全部满足条件返回 true,否则 false
if (list.all { it.length > 10 }) {
}

// 如果有一个满足条件就返回 true
if (list.any { it.length > 10 }) {
}
```

## filter 操作

列表的 `filter` 操作是指按照一定的条件对列表元素进行过滤，返回包含过滤后元素的新列表。

在进行这部分函数设计时，总会有 `xxxx()` 和 `xxxxTo(list:MutableCollection<T>)` 这样的函数组合，前者将会返回一个 `List<out T>` 类型的集合，也就是只读的，而后者将会返回参数中传递的 `MutableCollection<T>`，他是可变的，下面将不再赘述。

```kotlin
val destList = mutableListOf<String>()

// 根据 lambda 表达式返回结果过滤元素，返回 List<out T>
list.filter { it.length in 2..3 }
// 根据 lambda 表达式返回结果过滤元素，返回可变集合
list.filterTo(destList) {
    it.length in 2..3
}
```

带有下标的的过滤

```
// 与 filter 函数相似，但是函数中会有 index 参数，返回 List<out T>
list.filterIndexed {
    index, value ->
    value.length in 2..3 && index in 3..5
}
// 与 filter 函数相似，但是函数中会有 index 参数，返回可变集合
list.filterIndexedTo(destList) {
    index, value ->
    value.length in 2..3 && index in 3..5
}
```

过滤属于某个类型的元素

```kotlin
// 使用 reified 关键字，因此不需要使用 class
list.filterIsInstance<String>()
list.filterIsInstanceTo(destList)

// 使用 class
list.filterIsInstance(String::class.java)
list.filterIsInstanceTo(destList, String::class.java)
```

filterNot

```kotlin
// 过滤不满足的条件的元素，与 filter 函数的功能相反，存储到新的列表中
list.filterNot { it.length > 10 }
list.filterNotTo(destList) { it.length > 10 }

// 是 filterNot 的一种扩展，过滤 null 数据
list.filterNotNull()
list.filterNotNullTo(destList)
```


## 搜索

从列表中按照条件搜索元素

```kotlin
// 查找第一个满足条件的值，返回 T？类型，查找不到将会为 null
list.find { it.length > 10 }

// 查找最后一个满足条件的值，返回 T？类型，查找不到将会为 null
list.findLast { it.length > 10 }
```

二分搜索，使用 `binarySearch` 之前，列表应被排为升序，否则结果是不确定的
 
```kotlin
// 根据传入的 lambda 表达式参数判断查找的值是否合适，返回负数表示，查找到的值过小，正数表示查找到的值过大，0 表示查找到
// 下面的写法可能相对复杂，只是为了体现如果告知当前查找结果的正确性。
list.binarySearch(fromIndex = 2, toIndex = 10) {
    if (it.length > 10) 1
    else if (it.length < 10) -1
    else 0
}


// 另一个重载方法，使用 Comparator 来进行数据的比较，参数都是可选的。
list.binarySearch(
        element = "10", 
        fromIndex = 0, 
        toIndex = 2,
        comparator = Comparator<String> { o1, o2 -> o1.length - o2.length })
list.binarySearch(element = "10", fromIndex = 0, toIndex = 2)
list.binarySearch(element = "10")
```

## 排序

对于 `List<out T>` 类型来说，排序之后仍然只能返回只读列表，注意，返回的列表将是一个新的列表，与原来的列表没有关系，为啥呢？因为原来的列表是不可变的啊。

```kotlin
// 范型 T 对应的类型要是 Comparable 的子类
list.sorted()

// 不必是 Comparable 的子类 但是 lambda 表达式返回值需要是 Comparable 的子类，即将一个不可比较的类型映射到一个可比较的类型
list.sortedBy { it.length }

// 使用指定比较器排序
list.sortedWith(comparator = Comparator { o1, o2 -> o1.length - o2.length })

// 降序排序
list.sortedDescending()
list.sortedByDescending { it.length }
```

对于 `MutableCollection<T>` 类型来说，内部使用的是 `Collection.sort()`   方法来排序，这是一个在 `java` 中就使用的方法，函数的返回值是 `Unit`，这意味着函数不会返回一个新的列表，排序后的列表与原来的列表是同一个，当然他也可以使用上面的函数排序返回一个不可变列表，但是一个不可变列表是无法使用下面的函数排序的。

函数声明与上面的函数类似，但是没有 `ed` 后缀。

```kotlin
list.sort()
list.sortBy { it.length }
list.sortWith(comparator = Comparator { o1, o2 -> o1.length - o2.length })
list.sortDescending()
list.sortByDescending { it.length }
```

## 删除

一般来说，`List<out T>` 类型是不可变的，但是可以使用 `drop()` 函数从列表中删除元素，但是此时删除后返回的列表是一个全新的列表。

```kotlin
// 去除前 n 个返回列表
list.drop(10)
// 去掉后面10个，返回列表
list.dropLast(10)
// 删除满足条件的元素
list.dropWhile { it.length > 10 }
// 从后向前删除满足条件的元素
list.dropLastWhile { it.length > 10 }
```

对于可变列表，可以使用 `remove()` 函数删除元素

```kotlin
// 删除元素，返回是否删除成功
list.remove("")
// 删除指定下标的元素，返回删除的这个元素
list.removeAt(0)
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
    // 按照条件删除，内部使用迭代器实现
    list.removeIf { it.length > 10 }
}
// 按条件删除
list.removeAll { it.length > 10 }
```

## 转换

类似 `RxJava` 中的操作符，使用操作符可以将列表转换为其他类型。主要有 `map`，`flapMap`，`associate` 等。


### map

使用 `Lambda` 表达式将每一项 `map` 到新的数据

```kotlin
// 返回新的 List<out Int>
list.map { it.length }
// 返回传入的 MutableList<T>
list.mapTo(mutableListOf<Int>()) { it.length }

// 与上面类似，只是 lambda 表达式参数有 index
list.mapIndexed { index, value -> index + value.length }
list.mapIndexedTo(mutableListOf<Int>()) { index, value -> index + value.length }
```

使用 `mapNotNull` 将集合中的非空数据进行 `map`，与上面的函数是类似的。

```kotlin
list.mapNotNull { it.length }
list.mapNotNullTo(mutableListOf<Int>()) { it.length }

list.mapIndexedNotNull { index, s -> index + s.length }
list.mapIndexedNotNullTo(mutableListOf()) { index, s -> index + s.length }
```

### flatmap

使用 `flatmap` 对集合中每一项都转换成一个集合，最后在一个存储在一个集合中。

```kotlin
// 对每一项映射返回一个集合
list.flatMap { mutableListOf(it.length) }
list.flatMapTo(mutableListOf()) { mutableListOf(it.length) }
```

### associate

这个操作主要用来将列表转换成对应的 `Map`。

以 `Pair` 的形式返回键值对，将 `List<T>` 类型转换为 `Map<K,V>`，返回的 `Map` 类型取决于返回 `Pair` 类型。

```kotlin
list.associate { Pair(it, it.length) }

list.associateTo(mutableMapOf()) { Pair(it, it.length) }
```

只接受一个 `keySelector` 参数用来映射生成最终 `Map` 的 `key`，将 `List<T>` 类型转换为 `Map<K,T>` 

```kotlin
list.associateBy { it }

list.associateByTo(mutableMapOf()) { it }
```

接受一个 `keySelector` 和 `valueTransform` 作为参数，映射生成 `key-value`，将 `List<T>` 类型转换为 `Map<K,V>`；

```kotlin
list.associateBy(keySelector = { it }, valueTransform = { it.length })

list.associateByTo(mutableMapOf(), keySelector = { it }, valueTransform = { it.length })
```

## 最大值最小值

用于查找整个列表中的最大值最小值

```kotlin
// 范型对应的的类 需要是 Comparable 的子类
list.min()
// 使用 lambda 表达式将列表元素转换为可比较类型
list.minBy { it.length }
// 自定义比较器
list.minWith(Comparator { o1, o2 -> o1.length - o2.length })


// max 同理
list.max()
list.maxBy { it.length }
list.maxWith(Comparator { o1, o2 -> o1.length - o2.length })
```


## 运算符

列表支持直接使用运算符操作；

从列表中减去一个元素，或者加上一个元素，我们可以使用如下函数

```kotlin
// 函数，减法操作
list.minusElement("1")

// 函数，加法操作
list.plusElement("1")
```

也有相同功能的 `operator` 函数，返回值为一个新的 `List<out T>`

```kotlin
// operator，减法，接受一个元素
list.minus("1")
// operator，减法，接受一个列表
list.minus(destList)

// operator，加法，接受一个元素
list.plus("1")
// operator，加法，接受一个列表
list.plus(destList)
```

由于上面的函数使用了 `operator` 注解标记，我们可以直接使用 `+` 和 `-` 代替使用函数。


```kotlin
val a = list + "1"
val b = list + destList
val c = list - "2"
val d = list - destList
```

同样，可变列表也支持 `+=` 和 `-=` 运算符，但是不可变列表无法使用这些函数，操作符对应的函数如下，返回值为 `Unit`，新的更改将作用于当前的列表。


```kotlin
// operator,-= 操作，接受一个元素
list.minusAssign("1")
// operator,-= 操作，接受一个集合
list.minusAssign(destList)
// operator,+= 操作，接受一个元素
list.plusAssign("1")
// operator,+= 操作，接受一个集合
list.plusAssign(destList)
```

也可以直接使用 `+=` 和 `-=` 运算符

```kotlin
list += "1"
list += destList
list -= "1"
list -= destList
```


## 累积

累积是我自己起的名字，是说遍历整个列表做一个累积操作，相关的函数有 `sum`，`reduce`，`fold` 等。

### sum

函数 `sum()` 是对列表每一项使用 `Lambda` 表达式转换为数字类型数据然后求和的操作；

```kotlin
// 对列表求和,返回 int
list.sumBy { it.length }

// 对列表求和,返回 double 类型
list.sumByDouble { it.length.toDouble() }
```

### reduce

函数 `reduce()` 将列表的第一项作为初始值，然后对列表每一项使用 `Lambda` 表达式转换，然后累积，由于使用了列表的第一项作为初始值，所以 `reduce()` 的返回值与列表范型类型相同。

下面函数中 `index` 表示元素下标，`acc` 表示累积的结果，`next` 表示下一个元素。

```kotlin
list.reduce {
    acc, next ->
    acc + next
}
// 从右边开始累积
list.reduceRight {
    acc, next ->
    acc + next
}
// 带下标
list.reduceIndexed {
    index, acc, next ->
    acc + index + next
}
list.reduceRightIndexed {
    index, next, acc ->
    acc + index + next
}
```

### fold

函数 `fold()` 与 `reduce()` 有些类似，但是 `fold()` 函数不是以列表的第一项作为初始值进行累积，而是可以指定一个初始值，也就是说，`fold()` 函数可以返回任意类型的累积结果。

```kotlin
list.fold(10) {
    acc, next ->
    acc + next.length
}
// 从右到左
list.foldRight(10) {
    next, acc ->
    acc + next.length
}
// 带有下标的从左到右
list.foldIndexed(10) {
    index, acc, next ->
    acc + index + next.length
}
// 带有下标的从右到左
list.foldRightIndexed(10) {
    index, next, acc ->
    acc + index + next.length
}
```

## take

使用 `take()` 函数可以从列表中取出指定的部分值组成新的列表

```kotlin
// 返回此时的列表的前 n 项，新的列表
list.take(10)
// 从后面返回 n 项，顺序还是原来的顺序
list.takeLast(10)

// 从前面取，直到满足条件，停止
list.takeWhile { it.length > 10 }
// 从后面取，直到满足条件，终止
list.takeLastWhile { it.length > 10 }

// 如果此列表满足条件，返回列表，否则返回 null
list.takeIf { it.size > 10 }
// 如果此列表不满足条件，返回列表，否则返回 null
list.takeUnless { it.size > 10 }
```

## 分组和切片

对列表进行分组和切片操作。

使用 `group()` 函数对列表分组，返回结果是 `Map<K,List<T>>` 类型，`key` 的类型由 `Lambda` 表达式的返回值决定。

```kotlin
list.groupBy {
    it.length
}

list.groupByTo(mutableMapOf()) { it.length }

// 返回 grouping 对象， since kt 1.1
list.groupingBy { it.length }
```
使用 `partition()` 函数，根据 `Lambda` 表达式返回 `true/false` 将列表分成两个列表，返回值为 `Pair<List<T>,List<T>>`

```kotlin
list.partition { it.length > 10 }
```

对列表进行切片，返回切片后的新列表

```kotlin
list.subList(0, 2)

list.slice(IntRange(1, 3))
```


## 其他

另外一些无法归类的函数

使用 `contains()` 函数判断列表是否包含某个或某些元素

```kotlin
list.contains("")
list.containsAll(mutableListOf("1", "2"))
```

判空

```kotlin
// 如果 list 为空，返回一个新列表
list.orEmpty()
list.isEmpty()
list.isNotEmpty()
```

使用 `reverse()` 函数反转列表，不可变列表可以使用 `asReversed()` 函数，这个函数将返回一个反转后的新的列表；

可变列表使用 `reverse()` 函数，返回值为 `Unit`，列表的反转将作用在自己身上，内部使用 `Java` 的 `Collection. reverse()` 实现，不可变列表不能调用该函数。

```kotlin
list.reverse()

list.asReversed()
```

使用 `distinct()` 对列表进行去重

```kotlin
list.distinct()

// lambda 表达式返回比较的 key
list.distinctBy { it.length }
```

使用 `count()` 函数，获取列表长度

```kotlin
// 元素个数
list.count()
// 满足条件的元素个数
list.count { it.length > 10 }
```

使用 `replaceAll()` 函数替换全部数据

```kotlin
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
    list.replaceAll { "${it.length}" }
}
```

使用 `join()` 函数，对每一项进行拼接，接受一个 `Appendable` 对象作为参数，可以设置拼接的前缀，后缀，分隔符等

```kotlin
list.joinTo(StringBuilder(), "-", "#", "*", 4) {
    "${it.length}"
}

list.joinToString("-", "#", "*", 4) { "${it.length}" }
```

使用 `single()` 函数，当列表只有一个元素时获取这个元素

```kotlin
// 当只有一个元素时返回该元素，否则异常
list.single()
// 当只有一个元素时返回该元素，否则返回 null
list.singleOrNull()
// 查找符合条件的一个，否则返回 null
list.singleOrNull { it.length > 10 }
```
使用 `union()` 函数，返回两个列表的非重复集合

```kotlin
list.union(destList)
```

使用  `zip()` 函数压合两个列表，返回值为 `List<Pair<T,R>>`，列表长度等于较短的一个

```kotlin
val list2 = listOf(4, 5, 6)

var zip = list.zip(list2)
```
使用 `subtract()` 函数从列表中剔除部分元素

```kotlin
list.subtract(destList)
```


使用 `retainAll()` 函数对两个列表取交集。

```kotlin
list.retainAll(destList)

// 保留符合条件的元素
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
    list.retainAll { it.length > 10 }
}
```