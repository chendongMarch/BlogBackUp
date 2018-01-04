---
layout: post
title: 使用 digitalocean 科学上网
categories:
  - digitalocean
tags:
  - Extensions
  - digitalocean
abbrlink: f546dc07
date: 2017-10-17 16:50:00
---
 
 使用 digitalocean 科学上网
 
 <!--more-->


连接到服务

```bash
ssh root@128.199.148.155
```

连接镜像

```bash
curl -sSL https://agent.digitalocean.com/install.sh | sh
```

```bash
yum install python

首先安装epel扩展源：
yum -y install epel-release

更新完成之后，就可安装pip：
yum -y install python-pip

安装完成之后清除cache：
yum clean all
　　
升级 pip
pip install --upgrade pip
pip install shadowsocks
```

启动／终止服务

```bash
ssserver -c /etc/shadowsocks.json -d start
ssserver -c /etc/shadowsocks.json -d stop
```


shadowsocks.json 配置文件

```bash
{
        "server": "0.0.0.0",
        "server_port": 8388,
        "local_address": "127.0.0.1",
        "local_port": 1080,
        "password": "chendong911091",
        "timeout": 300,
        "method": "aes-256-cfb",
        "fast_open": false
}
```

安装 bbr 加速

```
wget –no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh
chmod +x bbr.sh
./bbr.sh
```


vim 找不到

```
rpm -qa|grep vim
yum -y install vim-enhanced
yum -y install vim-common
```
