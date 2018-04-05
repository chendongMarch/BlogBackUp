---
layout: post
title: 借助 Vultr 科学上网
categories:
  - 科学上网
tags:
  - Extensions
abbrlink: f546dc07
date: 2017-10-17 16:50:00
---
 
作为一个开发人员，用惯了 `Google` 再回来用百度，特别难受，而且像  `Android` 的一些官方文档只能出去了才能看，为了能够流畅的访问 `Google`，一路走来尝试了很多不同的方式，也发现每个平台都各有利弊，最终选择了 [Vultr.com](https://www.vultr.com/?ref=7379956)，记录一下。

## 方案选择

方案有很多，大致分为两种：

- 省事儿的，用人家做好的服务，只要交钱就好了，缺点是受制于人（动不动就涨价、还偶尔抽风不能用）。
- 费事儿的，自己买服务器搭建 ss，适合开发人员，这个服务器还可以用来挂一些自己的小服务，也可以练习部署服务的一些基本操作。


### 第三方服务

- `Lattern` 是最开始使用的方法，有 `500M` 流量可以免费用，但是连接不稳定，网速又慢，好的时候只能浏览个网页。
- `vpn.so` 这是一个三方的 `vpn` 网站，每年 `200RMB` 就可以获得一个节点，速度的话一般吧，好的时候可以看看视频，前段时间开会给封了，不知道最近好了没有。

### 自建 ss 服务

通过服务器网站购买国外服务器，自建梯子，转发流量，这种方法相对复杂一些，不过购买后你获得的是一台服务器，你还可以利用这台服务器做很多事情。

国内有阿里云和腾讯云等服务商，但是服务器的价格实在太贵了，所以我们会选择一些小众的服务器代理商。

- `digitalocean` 是我最先使用的一个平台，它可以允许你透支大约20美元的样子，不过缺点是操作界面太卡，而且仅支持 `paypal` 支付。
- [Vultr.com](https://www.vultr.com/?ref=7379956) 是我最近刚开始使用的，之前 `digitalocean` 透支之后就没有再继续使用了。


### vultr 的优势

- 操作界面相对清新
- 可以支持支付宝支付，不必绑定 paypal，对国内用户更友好
- 服务器选择更多一些
- 支持支付宝支付，对国内用户友好很多
- 按小时计费，你不用了可以暂停掉，不用担心一直计费
 

## 充值

Vultr.com 按小时计费，但是首先需要充值，最少 10 美元，支持支付宝。

![充值](http://olx4t2q6z.bkt.clouddn.com/18-4-1/71773190.jpg)

## 选择合适的服务

配置服务器共有 7 个选项，按照顺序配置好即可。

![](http://olx4t2q6z.bkt.clouddn.com/18-4-1/69553185.jpg)

- Server Location => 建议选择 Chicago 和 Los Angeles，速度会好一些

- Server Type => CentOS 7 * 64

- Server Size => $5 / mo

- Startup Script => 删除原来的脚本，添加以下脚本进行工具的安装

```bash
yum install python
yum -y install epel-release
yum -y install python-pip
yum clean all
pip install --upgrade pip
pip install shadowsocks
```

- SSH Keys => 配置 SSH

```bash
ssh-keygen -t rsa -b 4096 -C 'your email'

# 然后获取  ~/.ssh/id_rsa.pub (mac) 里面的内容配置到相应位置
cat ~/.ssh/id_rsa.pub
```

- Server Hostname & Label => 随便选一个你喜欢的

点击 `Deploy Now` 即可创建一个服务。



## 管理服务器

![](http://olx4t2q6z.bkt.clouddn.com/18-4-1/23342516.jpg)

按照图中所示，点击查看 `IP` 地址和初始密码（随机字符串）

在本地打开 terminal 执行 ssh 命令连接到远程服务，此时会要求你输入 `root` 用户的密码，按照上面查看到的输入即可

```bash
sudo ssh root@128.199.148.155
```
连接之后使用下面的命令更改 `root` 密码，随机密码用起来太麻烦了

```bash
passwd
```

## 配置 SS

目前我们已经连接到了远程服务器上，由于开始我们配置服务器时使用了启动脚本，那么目前所有的工具应该都是已经安装好的，检查一下对应的命令，如果没有就重新安装一下。

主要需要以下工具

- python => 一般自带，使用 python -V 检查
- pip => python 包管理工具
- ssserver => shadowsocks 服务
- vi、vim => 用于编辑配置文件

以上命令找不到的话，使用以下命令安装

```bash
# 安装 python
yum install python

# 安装 pip
# 首先安装epel扩展源：
yum -y install epel-release
# 更新完成之后，就可安装pip：
yum -y install python-pip
# 安装完成之后清除cache：
yum clean all
# 升级 pip
pip install --upgrade pip

# 安装 ssserver
pip install shadowsocks

# 安装 vim
rpm -qa|grep vim
yum -y install vim-enhanced
yum -y install vim-common
```

环境和工具安装好之后，新建并编辑 shadowsocks 配置文件

```bash
vi /etc/shdowsocks.json

# 输入 i, 进入 insert 模式，粘贴下面的内容

{
	"server": "0.0.0.0",
	"server_port": 8388,
	"local_address": "127.0.0.1",
	"local_port": 1080,
	"password": "你的密码",
	"timeout": 300,
	"method": "aes-256-cfb",
	"fast_open": false
}
```

开启 `8388` 端口

```bash
firewall-cmd --zone=public --add-port=8388/tcp --permanent
firewall-cmd --reload
```

启动／终止服务 ss 服务

```bash
ssserver -c /etc/shadowsocks.json -d start
ssserver -c /etc/shadowsocks.json -d stop
```

## ShadowSocks

小飞机，代理软件，在 `Mac` 和 `Android` 都有对应的客户端，这里提供一个[下载链接](https://pan.baidu.com/s/1eNB2f7xRkgK4F_id44oYqw)，下载安装后，配置服务器

![](http://olx4t2q6z.bkt.clouddn.com/18-4-1/9332692.jpg)

配置好服务后，访问一下 [www.google.com](www.google.com)，尝试一下是否可以正常访问。



## 安装 bbr 加速

安装 `bbr` 加速，速度可以提升接近一倍左右。

```
wget –no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh
chmod +x bbr.sh
./bbr.sh
```

## 综上

就目前来看，自建 ss 的方案各个服务商的价格基本一致，大约都在 $5 / mo，不过 [Vultr.com](https://www.vultr.com/?ref=7379956) 可以按小时计费要相对合理一些。

快去试试吧 => [Vultr.com](https://www.vultr.com/?ref=7379956)	

 