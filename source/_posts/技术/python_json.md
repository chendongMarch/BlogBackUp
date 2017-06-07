---
layout: post
title: Python Json相关
date: 2016-08-11
category: 技术
tags: Python
keywords: 
description: Python Json 相关
---


## 开始
```python
# -*-coding:utf-8-*-
import json

from httptst.Singleton import singleton


@singleton
class JsonHelper(object):
    name = 'json helper'

    def convert_to_builtin_type(obj):
        print 'default(', repr(obj), ')'
        dict = {}
        dict.update(obj.__dict__)
        return dict

    # obj 转 json
    def getJson(self, obj):
        data = json.dumps(obj, sort_keys=True, default=self.convert_to_builtin_type)
        return data

    # json str 转dict
    def parse(self, jsonStr):
        jsonDict = json.loads(jsonStr)
        return jsonDict

```