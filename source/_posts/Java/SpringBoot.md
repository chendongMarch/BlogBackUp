---
layout: post
title: SpringBoot学习
categories:
  - Java
tags: Java
keywords:
  - Java
  - SpringBoot
abbrlink: b2a6184b
date: 2015-08-15 00:00:00
---

## 依赖说明

```gradle
// JSP
implementation 'javax.servlet:jstl'
implementation 'org.apache.tomcat.embed:tomcat-embed-jasper'

// 数据库
implementation 'org.springframework.boot:spring-boot-starter-jdbc'
implementation 'org.mybatis.spring.boot:mybatis-spring-boot-starter:2.1.1'
runtimeOnly 'mysql:mysql-connector-java'

// web
implementation 'org.springframework.boot:spring-boot-starter-web'

// 热部署
developmentOnly 'org.springframework.boot:spring-boot-devtools'
```

## 热部署

- 依赖 `developmentOnly 'org.springframework.boot:spring-boot-devtools'`
- `Setting` -> `Compiler` -> `build auto` 勾选
- `Ctrl + Shfit + A` 搜索 `Registry` 勾选 `compiler.automake.allow.when.app.running`

## 配置

application.yaml

```yaml
server:
  port: 8089
  servlet:
    context-path: /zfy

spring:
  mvc:
    view:
      suffix: .jsp
      prefix: /WEB-INF/views/
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/db_blog?characterEncoding=UTF-8
    username: root
    password: chendong911091
    driver-class-name: com.mysql.cj.jdbc.Driver

author:
  name: chendong
  description: Welcome To My Server, My Name Is ${author.name}

mybatis:
  configuration:
    # 下划线命名转驼峰
    map-underscore-to-camel-case: true
```


## Swagger

```
@Api
@ApiOperation
@ApiModel
@ApiModelProperties
@ApiParam
```


## RabbitMQ


```
brew install rabbitmq

erl -version

路径 
/usr/local/Cellar/rabbitmq/3.8.0/sbin

启动服务
sudo ./rabbitmq-server start

停止服务
sudo ./rabbitmq-server stop


安装WebUI
sudo ./rabbitmq-plugins enable rabbitmq_management

// 添加用户
sudo ./rabbitmqctl add_user chen chen
// 为用户设置 tag
sudo ./rabbitmqctl set_user_tags chen administrator
// 设置权限
sudo ./rabbitmqctl ser_permissions chen -p "/" chen ".*" ".*" ".*"
// 查看用户
sudo ./rabbitmqctl list_users
```

同步变异步
解耦
流量削峰


发布订阅模式 direct
主题订阅模式 topic

## docker

物理机 -》 虚拟机 ——〉

```
// 安装 ng
docker run -p 80:80 -d nginx


// 查看本地镜像
docker images
// 查找镜像
docker search mysql
// 获取镜像
docker pull mysql:5.7
// 删除镜像
docker rmi  [IMAGE ID]
// 构建
docker build -t registy-chen/appimg:1.0.0 .

docker run -d -p 8089:8089 --name=chenapp3 registy-chen/appimg:1.0.0
// 登录
docker login
// 命名
docker tag registy-chen/appimg:1.0.0 chendongmarch/zfy-repo:1.0.0
// 推送镜像
docker push chendongmarch/zfy-repo:1.0.0
// 拉取镜像
docker pull chendongmarch/zfy-repo:1.0.0

// 启动容器
sudo docker run -d -p 5000:5000 --restart=always --privileged=true --name=registry-zfy -v /Users/march/Downloads/test_code/docker-build/registry:/var/lib/registry registry

curl -X GET http://127.0.0.1:5000/v2/_catalog
curl -X GET http://127.0.0.1:5000/v2/zfy-repo-local/tags/list

```



