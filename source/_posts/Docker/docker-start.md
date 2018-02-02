---
layout: post
title: 基于 Docker 环境搭建 Jenkins
categories:
  - Docker
  - Jenkins
hide: true
keywords:
  - Docker
  - Jenkins
tags:
  - Docker
  - Jenkins
abbrlink: fa67563
date: 2017-10-19 14:15:31
---

[一篇相当全面的文档](https://yeasy.gitbooks.io)

[docker.com](https://www.docker.com/)

## docker 加速

官网下载 `docker` 速度很慢，可以前往 [daocloud.io](https://get.daocloud.io/) 下载 docker，我下载的是 `docker for mac`，速度很快，但是版本比较老，安装后再 `check for updates` 升级到新版本，速度会快一些。

安装好之后，下载镜像速度也很慢，可以使用打开 `Preference->Daemon->Registry Mirrors` 添加如下镜像，然后 `apply&restart` 就可以生效；

可以使用 `docker info` 命令查看是否生效

```bash
http://b5886426.m.daocloud.io/
https://docker.mirrors.ustc.edu.cn/
```


## docker command

docker pull jenkins:latest


以普通用户运行 bash

docker exec -it mycontainer bash

以 root 用户运行 bash
docker exec -u 0 -it mycontainer bash

运行 jenkins

```bash
sudo docker run -d --name jenkins-node -p 49002:8080 -v /Users/march/docker/jenkins-node:/var/jenkins_home jenkins-android
```

解释一下这条命令

```bash
-d => debug 模式，后台运行，不会占用终端，终端关闭也不会停止服务

--name jenkins_node => 别名，否则会是一个随机别名，别名可以用来管理该容器

-p 49002:8080 => 端口映射，将 8080 映射到 49002，后面使用 localhost:49002 访问

-v /Users/march/docker/jenkins_node:/var/jenkins_home => 文件挂载，将 /var/jenkins_home 挂载到指定目录，如果不挂载，则jenkins所有log、用户配置文件都会在docker容器内，如果容器销毁，则jenkins得重新配置一遍。挂载出来方便jenkins迁移以及管理，/Users/march/docker/jenkins_node 是我的一个本地目录，在该目录下可以看到 jenkins 的相关文件，这个目录随意，注意权限问题。

jenkins:latest => 镜像名称
```


## linux

安装 sudo

```
apt-get install sudo
```

安装 nodejs、npm

```
curl -sL https://deb.nodesource.com/setup_9.x | sudo -E bash -
sudo apt-get install -y nodejs
sudo apt-get install -y npm
```


## ssh

ssh-keygen -t rsa -b 4096 -C 'email'

本机存放 id_rsa，远端存放 id_rsa.pub 公钥

ssh-agent bash

ssh-add id_rsa

