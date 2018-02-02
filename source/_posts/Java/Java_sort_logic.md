---
layout: post
title: 常见排序及搜索算法总结
categories:
  - Java
tags: Java
keywords:
  - Java
  - sort
  - 排序
abbrlink: 395850883
date: 2015-01-01 00:00:00
---


## Arrays.sort()

Arrays.sort()使用了两种排序方法，快速排序和优化的合并排序。

快速排序主要是对哪些基本类型数据（int,short,long等）排序。

而合并排序用于对对象类型进行排序。

原因：使用不同类型的排序算法主要是由于快速排序是不稳定的，而合并排序是稳定的。这里的稳定是指比较相等的数据在排序之后仍然按照排序之前的前后顺序排列。对于基本数据类型，稳定性没有意义，而对于对象类型，稳定性是比较重要的，因为对象相等的判断可能只是判断关键属性，最好保持相等对象的非关键属性的顺序与排序前一致。

- 另外一个原因是由于合并排序相对而言 **比较次数** 比快速排序少， **移动(对象引用的移动)次数**  比快速排序多，而对于对象来说，移动是简单的，只是引用的转换，但是比较相对更加耗时。

- 合并排序的时间复杂度是n\*logn, 快速排序的平均时间复杂度也是n*logn，但是合并排序的需要额外的n个引用的空间。

## 排序算法实现

### 冒泡排序
```java
//冒泡:0 ~ n-1  0 ~ n-1-i
//冒泡排序:两两比较,比较n-1趟,每一趟比较n-i-1次
public static void BubbleSort(int a[]){
    int n = a.length;
    for(int i = 0; i < n - 1; i++) //比较n-1趟
    for(int j = 0; j < n - 1 - i; j++){//比较n-i-1次!
    if(a[j+1] < a[j]){
    a[j+1] = a[j+1] ^ a[j];
    a[j] = a[j] ^ a[j+1];
    a[j+1] = a[j] ^ a[j+1];
	}
    }
}
```
### 选择排序
```java
//选择排序:遍历前n-1个元素,与 i+1 直到 n 个元素比较,记录小数下标
public static void SelectSort(int a[])
{
	int n = a.length;
	for(int i = 0; i < n - 1; i++){
			int min = i;
			for(int j = i + 1; j < n; j++){
					if(a[min] > a[j]){
						min = j;
					}
				}
				if(min != i){
					a[i] = a[i] ^ a[min];
					a[min] = a[min] ^ a[i];
					a[i] = a[min] ^ a[i];
				}
		}
}
```
### 插入排序
```java
//插入排序:左边为有序区域,遍历第一个直到第n个,与前i个比较
		public static void InsertSort(int a[]){
			int n = a.length;
			for(int i = 1;i < n; i++)
				for(int j = i; j > 0; j--){
				if(a[j] < a[j-1]){
					a[j] = a[j] ^ a[j-1];
					a[j-1] = a[j-1] ^ a[j];
					a[j] = a[j-1] ^ a[j];
				}
				else
					break;
			}
		}
```
### 快速排序
```java
public static void quicksort(int n[], int left, int right) {
	        int dp;
	        if (left < right) {
	            dp = partition(n, left, right);
	            quicksort(n, left, dp - 1);
	            quicksort(n, dp + 1, right);
	        }
	    }
public static int partition(int n[], int left, int right) {
	        int pivot = n[left];
	        while (left < right) {
	            while (left < right && n[right] >= pivot)
	                right--;
	            if (left < right)
	                n[left++] = n[right];
	            while (left < right && n[left] <= pivot)
	                left++;
	            if (left < right)
	                n[right--] = n[left];
	        }
	        n[left] = pivot;
	        return left;
}
```


## 搜索算法实现
### 二分法搜索
```java
public static int HalfSearch(int a[],int target){
			int bottom = 0;
			int top = a.length - 1;
			int middle = (top + bottom) / 2;
			while(bottom <= top){
				if(a[middle] > target){
					top = middle - 1;
				}
				else if(a[middle] < target){
					bottom = middle + 1;
				}
				else {
					return middle;
				}
				middle = (top + bottom) / 2;
			}
			return -1;
		}
```