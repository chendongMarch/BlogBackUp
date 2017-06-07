---
layout: post
title: Node.js基础1
date: 2016-05-14
category: 技术
tags: [NodeJs]
keywords: NodeJs
description: Node.js基础
---


## 基本工具的配置

### 检测xcode是否安装
```bash
$ xcode-select -p
/Applications/Xcode.app/Contents/Developer
```
### 检测python和ruby版本
```bash
$ python -v
$ ruby -v
```
### 安装homebrew
```bash
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
### 安装node.js
```bash
$ brew install node
```

## 第一个web应用

### 创建连接

```javascript
//新建一个文件，为server.js,内容如下

const http = require('http');

const hostname = '127.0.0.1';
const port = 3000;

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello World\n');
});

server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```

### 启动服务
- 到相应目录下运行命令,打开浏览器输入`http://127.0.0.1:3000/`可以看到服务已经起来了，使用ctrl+c停止服务

```bash
$ node server.js
```