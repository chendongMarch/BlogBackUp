---
layout: post
title: Python Django服务器搭建
categories:
  - Python
tags: Python
keywords:
  - python
  - Django
description: Python Django服务器搭建初体验
abbrlink: 1002618664
date: 2016-05-14 00:00:00
---


## 前言
- 安装的部分工具，我是在Mac下使用，其他的平台自己搜索一下如何安装。现在默认你已经安装了Python环境，我的是Python2.7

## 安装pip包管理器

- pip包管理器是python下安装模块的工具

```bash
$ curl -O https://raw.github.com/pypa/pip/master/contrib/get-pip.py
$ [sudo] python get-pip.py

# 安装包
$ pip install django
# 卸载包
$ pip uninstall django
```

## 安装ipython
- ipython是一个增强式的python交互式操作命令工具，有自动提示和补全的功能

```bash
$ pip install ipython
```

## 安装django
```bash
$ pip install django
```

## 简单尝试
```bash
# 创建一个项目
$ django -admin startproject mysite

# 开启服务
$ python manage.py runserver

# 出现如下字样
May 14, 2016 - 00:00:21
Django version 1.9.6, using settings 'mysite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
Not Found: /
[14/May/2016 00:00:32] "GET / HTTP/1.1" 200 1767
Not Found: /favicon.ico
[14/May/2016 00:00:32] "GET /favicon.ico HTTP/1.1" 404 1936

# 打开浏览器输入`http://127.0.0.1:8000/`就可以看到It worked字样，一个简单的web服务就好了

# 你可以使用`python manage.py`查看更多的命令
```

## 各个文件的简单介绍
```bash
$ cd mysite/
$ ls
manage.py	mysite
# manage.py是一个管理工具，使用它可以管理服务器，比如开启服务等
# mysite是你的工程目录
$ cd mysite/
__init__.py	settings.py	urls.py		wsgi.py
__init__.pyc	settings.pyc	urls.pyc	wsgi.pyc
# settings.py 是一些配置信息
# urls.py 是url的映射，他表示不同的url会映射到不同的网页
```

## 建立app

- django是使用app的形式来配置模块，你可以建立新的模块

```bash
$ python manage.py startapp blog
# 可以发现在mysite目录下有了blog和mysite两个文件夹
# 在mysite/mysite/settings.py中配置这个app模块
# Application definition
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'blog'
]
```

## 构建一个简单的网页
```bash
$ cd blog/
$ ls
_init__.py	admin.pyc	models.py	views.py
__init__.pyc	apps.py		models.pyc	views.pyc
admin.py	migrations	tests.py
# blog目录下的文件，我们现在只使用views.py，他是用来构建输出html 页面的一个类



#（1）打开views.py 定义一个简单的html页面
from django.shortcuts import render
from django.http import HttpResponse
# Create your views here.
def hello(request):
    return HttpResponse('<html>hello world</html>')
    
    
# (2) 打开mysite/mysite/urls.py 配置url映射
from django.conf.urls import url
from django.contrib import admin
urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'hello','blog.views.hello')
]

# (3) 启动服务,在浏览器输入http://127.0.0.1:8000/hello,可以获得返回的网页
$ python manage.py runserver
```

