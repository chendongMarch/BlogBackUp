---
layout: post
title: SSH 命令
categories:
  - bash
tags:
  - ssh
abbrlink: f225b262
date: 2017-02-06 16:34:00
---


命令行工具 `SSH` 命令记录

<!--more-->


- 本地生成 `ssh` 密钥对，使用时本机存放 `id_rsa`，远端存放 `id_rsa.pub` 公钥

```bash
ssh-keygen -t rsa -b 4096 -C 'email'
```


- 本地命令行信任 ssh，不用每次都输入密码

```bash
ssh-agent bash

ssh-add id_rsa
```

- 连接远程服务

```bash
ssh root@ip
```