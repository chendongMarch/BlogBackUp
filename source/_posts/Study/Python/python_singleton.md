---
layout: post
title: Python实现单例
categories:
  - Study
  - Python
tags: Python
keywords:
  - python
  - 单例模式
  - singleton
abbrlink: 3789624758
date: 2016-08-11 00:00:00
---

## 单例装饰器
- 在网上查到很多方式，选择一种比较pythonic的方式

```python
def singleton(cls, *args, **kwargs):
    instances = {}

    def _singleton():
        if cls not in instances:
            instances[cls] = cls(*args, **kwargs)
        return instances[cls]

    return _singleton
```

## 使用

```python
@singleton
class HttpHelper:
    def __init__(self):
        pass

    name = 'http helper'
    ......
    ....
```