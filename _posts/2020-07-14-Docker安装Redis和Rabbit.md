---
layout:     post                    # 使用的布局（不需要改）
title:      Docker安装Redis和Rabbit               # 标题 
subtitle:   Docker安装Redis和Rabbit #副标题
date:       2020-07-08              # 时间
author:     GFZ                     # 作者
header-img: img/post-bg-unix-linux.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Docker
---

# 1、安装Redis

## 1、搜索镜像

```shell
docker search redis
```

##  2、Pull镜像

```shell
docker pull redis
```

## 3、运行镜像

```shell
docker run -itd --name myredis -p 6379:6379 redis --requirepass "你的密码"
```

## 4、常用操作

与之前的MariaDB相同的有`docker start container-id`和`docker stop container-id`的操作。

## 5、关于连接

连接Redis可以用RDM来连接，管理比较方便。

# 2、安装Rabbit

## 1、Pull镜像

```shell
docker pull rabbitmq:management
```



## 2、运行镜像

```shell
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:management
```

## 3、常用操作

与之前的MariaDB相同的有`docker start container-id`和`docker stop container-id`的操作。

## 4、关于连接

正常的Rabbit消息管理，可以使用自带的后台管理，浏览器访问`0.0.0.0:15672`。地址不固定，看映射的地址。