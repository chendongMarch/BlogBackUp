---
layout: post
title: ArrayList源码分析(Java&Android)
date: 2015-11-15
category: 学习
tags: [Android,Java,源码]
---


开始看源代码了才知道android和java的源代码有好多出入，我把java和android源码比较了一下，相似的还是多的。我是以android源码为主，因为感觉android比较繁琐，不同的地方会贴java的代码对比。

java真的设计的很好，很多类内部实现差别很大，但是在外部看起来是一样的，比如LinkedList和ArrayList,设计良好的API使我们不需要去关注内部的实现，举个例子说，我们需要一个绳子，他可以是真丝制成的，那么它比较坚固，也可以是牛皮制成的，那么它可以不怕水，但是对使用绳子的人来说，他声明自己需要的绳子需要具有什么样的特性，但是他却不知道真丝的需要什么工艺，牛皮的又需要什么工艺。我们程序员就是使用绳子的人，各种各样的List就是绳子，虽然内部实现大相径庭，但是都对外开放了一条绳子的外表，使得我们可以极为方便的使用它，这体现了面向对象的编程思想，也可以使java程序员更加专注的实现功能而不尽量少的关心逻辑和算法。

<!--more-->

## ArrayList类
- ArrayList内部基于数组实现,继承了AbstractList类，实现了Cloneable接口表示其可被克隆复制，实现了Serializable接口表示其可被序列化，实现了RandomAccess接口，表示其可快速随机访问（但是这个接口是空的，只是表示它具有这样的特性）。
```java
public class ArrayList<E> extends AbstractList<E> implements Cloneable, Serializable, RandomAccess{}
```

## 一些常量和成员
- android中提供了最小容量，java中并没有这个成员，而是使用了10这个数字作为初始容量。


```java
//anrdroid源码
//最小容量
private static final int MIN_CAPACITY_INCREMENT = 12;
//元素数量
int size;
//对象数组，transient表示在序列化时它会被忽略。。啥意思？
transient Object[] array;
```

## 初始化
### 指定初始容量构造
- 首先容量大于0，如果容量==0会使用EmptyArray.OBJECT来初始化，在android的源码里面链接不到EmptyArray这个类，去了[这个网站](http://www.boyunjian.com/javasrc/org.robovm/robovm-rt/0.0.2/_/libcore/util/EmptyArray.java)看到了源码，EmptyArray.OBJECT是一个容量是0的数组。所以也就是说，如果指定容量==0则创建一个容量是0的对象数组，反之创建一个指定容量大小的数组。(java中没有做这个判断，本来这个判断就什么用)

```java
//android 源码
public ArrayList(int capacity) {
        if (capacity < 0) {
            throw new IllegalArgumentException("capacity < 0: " + capacity);
        }
        array = (capacity == 0 ? EmptyArray.OBJECT : new Object[capacity]);
    }
```
### 无参构造函数
- 同上，初始容量是0。但是java源码中初始容量是10。


```java
	//android 源码
    public ArrayList() {
        array = EmptyArray.OBJECT;
    }
    //java源码
    public ArrayList() {
	this(10);
    }
```
### 初始化时拷贝集合
- 主要的操作是判断是不是一个对象数组，如果是直接内部数组直接指向，如果不是进行一次拷贝，java中所有类都继承自Object。这是为了防止基本数据类型吗？（<-这是我不懂的地方）用的方法不一样，java代码更少一点。


1. System.arraycopy()方法是native修饰的，使用C实现的底层方法。
2. Arrays.copyOf()函数内部也使用了System.arraycopy()方法。
3. 就是我不懂的那个地方了,感觉使用这个方法可以解决不是对象数组的问题，但是不懂这个判断是为了什么，System.arraycopy又是如何内部实现的。


```
//android 源码
if (a.getClass() != Object[].class) {
System.arraycopy(a, 0, newArray, 0, a.length);
}
```


```java
//android 源码
public ArrayList(Collection<? extends E> collection) {
        if (collection == null) {
            throw new NullPointerException("collection == null");
        }
        Object[] a = collection.toArray();
        if (a.getClass() != Object[].class) {
            Object[] newArray = new Object[a.length];
            System.arraycopy(a, 0, newArray, 0, a.length);
            a = newArray;
        }
        array = a;
        size = a.length;
}
```

```java
//java源码
public ArrayList(Collection<? extends E> c) {
	elementData = c.toArray();
	size = elementData.length;
	// c.toArray might (incorrectly) not return Object[] (see 6260652)
	if (elementData.getClass() != Object[].class)
	    elementData = Arrays.copyOf(elementData, size, Object[].class);
}
```

## Add方法

### Android中的实现

```java
@Override 
public boolean add(E object) {
        Object[] a = array;
        int s = size;
        if (s == a.length) {
            Object[] newArray = new Object[s +
                    (s < (MIN_CAPACITY_INCREMENT / 2) ?
                     MIN_CAPACITY_INCREMENT : s >> 1)];
            System.arraycopy(a, 0, newArray, 0, s);
            array = a = newArray;
        }
        a[s] = object;
        size = s + 1;
        modCount++;
        return true;
    }
    @Override 
    public void add(int index, E object) {
        Object[] a = array;
        int s = size;
        if (index > s || index < 0) {
            throwIndexOutOfBoundsException(index, s);
        }
        if (s < a.length) {
            System.arraycopy(a, index, a, index + 1, s - index);
        } else {
            // assert s == a.length;
            Object[] newArray = new Object[newCapacity(s)];
            System.arraycopy(a, 0, newArray, 0, index);
            System.arraycopy(a, index, newArray, index + 1, s - index);
            array = a = newArray;
        }
        a[index] = object;
        size = s + 1;
        modCount++;
    }
```

### Java中的实现
- 跟android基本一样的，之所以代码简单了，是因为java将判断扩容的操作放在了方法实现，但是androd只是将计算新容量的方法提取出来了。

```java
public boolean add(E e) {
	ensureCapacity(size + 1);  // Increments modCount!!
	elementData[size++] = e;
	return true;
    }
public void add(int index, E element) {
	if (index > size || index < 0)
	    throw new IndexOutOfBoundsException(
		"Index: "+index+", Size: "+size);
	ensureCapacity(size+1);  // Increments modCount!!
	System.arraycopy(elementData, index, elementData, index + 1,
			 size - index);
	elementData[index] = element;
	size++;
    }
```

### 核心代码分析
- 解释一下下面这段代码，在java中也是如此实现的只是提取到了方法中，会在下面的扩容机制中说明。
- 	首先最后都进行了一个赋值操作`a[index] = object;`前面的代码的工作就是将这个位置空出来。代码5
- 如果当前的size没有超过length,则将index位置之后的位置向后移动一位，将index位置空出来（代码1），如果超过长度了，则进行一次扩容（代码2），将数据拷贝到新数组（代码3），再将数组后移一位（代码4）
- 它是怎么移动的呢？`System.arraycopy(a, index, newArray, index + 1, s - index)`了解参数的含义（src,源数组中开始复制的位置，dest,目标数组开始粘贴的位置，复制的长度），这样就明白了，包括代码1是一样的意思，a数组和newArray数组是相同的，将a中index开始的数据复制到newArray中index+1开始的位置，然后复制的长度是s-index,刚好index位置被空出来了，举个例子，数据a={21，22，23，25，26}，newArray={21，22，23，25，26},需要在3的位置插入一个24，则将3开始的size-index(5-3=2)长度的数组也就是25，26，复制到newArray中index+1(3+1=4)开始的位置，得到新数组21，22，23，空，25，26，然后array[3]=24;

```java
Object[] a = array;
int s = size;
if (s < a.length) {
System.arraycopy(a, index, a, index + 1, s - index);//---1
} else { 
Object[] newArray = new Object[newCapacity(s)];//---2
System.arraycopy(a, 0, newArray, 0, index);//---3
System.arraycopy(a, index, newArray, index + 1, s - index);//---4
array = a = newArray;
}
a[index] = object;//5
size = s + 1;
```


## 扩容机制

- 单独拿出来是因为android和java扩容机制稍有不同


### Android中的实现

- 扩容时，如果当前容量小于6则让其等于12，否则扩大为原来的两倍。

```java
private static int newCapacity(int currentCapacity) {
int increment = (currentCapacity < (MIN_CAPACITY_INCREMENT / 2) ?
MIN_CAPACITY_INCREMENT : currentCapacity >> 1);
return currentCapacity + increment;
}
//这只是计算了新的容量，真正的扩容如此实现
Object[] newArray = new Object[newCapacity(s)];
System.arraycopy(a, 0, newArray, 0, index);
```

### java中的实现,

- java中数组拷贝都是用Arrays.copyOf()方法的，基本原理相同。
- 可以发现java扩容将容量扩大到了原来容量的1.5倍。
- 这里有一行代码,看注释的意思是这个方法更加优化了，调皮的程序员a.大意是minCapacity 通常更加逼近size.

```java
if (newCapacity < minCapacity) newCapacity = minCapacity;
```

```java
public void ensureCapacity(int minCapacity) {
	modCount++;
	int oldCapacity = elementData.length;
	if (minCapacity > oldCapacity) {
	    Object oldData[] = elementData;
	    int newCapacity = (oldCapacity * 3)/2 + 1;
    	if (newCapacity < minCapacity)
		newCapacity = minCapacity;
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
	}
}
```


## 综上
- 就是核心方法了吧，已经可以了解到它基本的原理，当然还有indexOf(),contains(),remove(),writeObject(),readObject(),内部迭代器的实现，toArray()的实现，set(),addAll().....其中的原理与上面提到的很多都是相同的，感兴趣的可以自行了解，这里不介绍了，后面有时间再贴一下吧。