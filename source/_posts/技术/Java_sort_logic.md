---
layout: post
title: 常见排序及搜索算法总结
category: 技术
date: 2015-01-01
tags: [Java]
keywords: 
description: 常见排序及搜索算法总结
---


## 排序

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
## 搜索
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