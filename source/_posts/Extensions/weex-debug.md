---
layout: post
title: 达人店 weex 多页面调试
tags:
  - Extensions
abbrlink: e15934a2
date: 2018-01-03 13:13:00
categories:
---


本文主要介绍如何在达人店客户端中对 weex 项目（shop-weex-m）进行 **全局调试** || **多页面调试**。

<!--more-->

## Log

2018.1.3   

- 支持 **全部页面** 和 **部分页面** 调试
- 支持缺省配置，只需要配置必要字段，其他字段自动匹配。


## How To Use 

调试依赖于 `weex-debug-config.json` 文件，它位于 `shop-weex-m` 项目根目录下，同时需要被 `ignore`，**不能被提交到远端**。

当从 `master` 切出新分支时，会有一个空的配置文件位于根目录下，如下：

- isGlobal  {boolean}  是否开启全局调试
- list  {array}  自定义调试页面配置

```json
{
    "isGlobal": false, 
    "list": []
}
```
### 缺省配置

为了方便调试，内部做了一些自动检索的工作，我们在配置页面时，只需要配置关键字段 


#### page

字段 `page` 与我们后台的 `page` 字段时相同的意思，所以这个 `page` 的值需要与后台配的一样才可以自动检索。如我要调试 **搜索结果页** 时可以如下配置：
 
```json
{
    "isGlobal": false, 
    "list": [
    	{
    		"page": "search-weex-v300"
    	}
    ]
}
```

#### weexPage

字段 `weexPage` 与我们开发时的 **目录文件夹** 名字相同，这样可以不必去后台或者抓包获取 `page` 字段。但是 ⚠️ 目前 `iOS` 还未支持，如我要调试 **搜索结果页** 时可以如下配置： 

```json
{
    "isGlobal": false, 
    "list": [
    	{
    		"weexPage": "search-result-weex"
    	}
    ]
}
```

####  新老页面的区别

自动检索依赖于线上配置，因此仅支持老页面，如果开发的是新页面，需要将全部字段书写完整；新老页面可以混合，新页面使用完整的配置，老页面使用缺省的配置，比如现在需要调试一个新页面称为  `new-page-weex`，可以如下配置：

ps:  版本 v 和 md5 不需要配置，statusColor 和 hideTitleBar 默认为空，不特别设置时也可以省略。

```json
{
    "isGlobal": false, 
    "list": [
    	{
    		"page": "search-weex-v300"
    	},
    	{
    		"statusColor": "#fff",
    		"hideTitleBar": "",
    		"page": "new-page-weex",
    		"h5": "http://shop.m.showjoy.net/new-page-weex",
    		"url": "http://192.1.1.1:3000/new-page-weex/new-page-weex.weex.min.js"
    	}
    ]
}
```

### 全局调试

全局调试指的是，客户端中所有的 weex 页面全部指向本地开启的服务。

开启全局调试的服务

```bash
spon weex dev
```
将配置文件中的 `isGlobal` 设置为 `true`，此时 `list` 里面的配置也会生效，并优先被使用，不过在全局调试模式下，`list` 里面最好全部都是新增的页面，因为老页面都已经全部指向本地服务啦，配置文件如下（伪注释）：

```json
{
    "isGlobal": true, 
    "list": [
    	// 老页面，也没问题，会被优先使用
    	{	
    		"page": "search-weex-v300"
    	},
    	// 新页面
    	{
    		"statusColor": "#fff",
    		"hideTitleBar": "",
    		"page": "new-page-weex",
    		"h5": "http://shop.m.showjoy.net/new-page-weex",
    		"url": "http://192.1.1.1:3000/new-page-weex/new-page-weex.weex.min.js"
    	}
    ]
}
```

### 部分页面调试

由于同时开启所有页面的服务，第一次编译和后续更改后编译速度都很慢，所以需要对部分页面进行调试。

开启部分页面调试的服务，如下，开启 3 个页面，其中有一个新页面

```bash
spon weex dev -n order-trade-result-weex,search-result-weex,new-page-weex
```

关闭 `isGlobal`，配置文件如下（伪注释）：

```json
{   
    "isGlobal": false, 
    "list": [
    	// 老页面，使用 page
    	{	
    		"page": "search-weex-v300"
    	},
    	// 老页面，使用 weexPage，iOS 不支持
    	{	
    		"weexPage": "user-info-weex"
    	},
    	// 新页面
    	{
    		"statusColor": "#fff",
    		"hideTitleBar": "",
    		"page": "new-page-weex",
    		"h5": "http://shop.m.showjoy.net/new-page-weex",
    		"url": "http://192.1.1.1:3000/new-page-weex/new-page-weex.weex.min.js"
    	}
    ]
}
```

## 设计方案

待完善
