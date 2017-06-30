---
layout: post
title: Mac 安装 Mongdb
categories:
  - Study
  - Db
tags:
  - Database
  - MongoDb
keywords:
  - Database
  - MongoDb
abbrlink: 3372882306
date: 2016-05-18 00:00:00
---


MongoDB 是由C++语言编写的，是一个基于分布式文件存储的开源数据库系统。

在高负载的情况下，添加更多的节点，可以保证服务器性能。MongoDB 旨在为WEB应用提供可扩展的高性能数据存储解决方案。

MongoDB 将数据存储为一个文档，数据结构由键值(key=>value)对组成。MongoDB 文档类似于 JSON 对象。字段值可以包含其他文档，数组及文档数组。

特点:高性能、易部署、易使用，存储数据非常方便。
<!--more-->



## 安装 MongoDb
安装`homebrew`

```bash
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

更新`homebrew`

```bash
$ brew update
```

安装`mongodb`

```bash
$ brew install mongodb
```


## 配置环境变量

```bash
$ open ~/.bash_profile 
# 写入以下路径
# mongodb
export PATH=${PATH}:/usr/local/Cellar/mongodb/3.2.6/bin
```
 

## 根据配置启动mongodb
使用该命令将会使用mongod.conf文件的相关配置来启动mongodb

文件的路径在 `/usr/local/etc/mongod.conf`（Mac）

从配置文件可以看到数据库的路径以及log的路径等。

```bash
$ mongod --config /usr/local/etc/mongod.conf

# mongod.conf 文件内容
systemLog:
  destination: file
  path: /usr/local/var/log/mongodb/mongo.log
  logAppend: true
storage:
  dbPath: /usr/local/var/mongodb
net:
  bindIp: 127.0.0.1
```


## 直接启动

直接启动快捷简单，数据将会存储在/data/db下,其中/data路径与/usr路径是同级的。

```bash
数据会存放在/data/db 目录下，需要首先创建目录，并更改目录权限
$ mkdir -p /data/db  
$ sudo chown -R march /data

启动mongodb服务
$ mongod

输入 mongo ，连接到mongo服务，即可进入命令行操作
$ mongo
```




