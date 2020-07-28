---
layout:     post                    # 使用的布局（不需要改）
title:      利用Docker安装MariaDB               # 标题 
subtitle:   利用Docker安装MariaDB #副标题
date:       2020-07-08              # 时间
author:     GFZ                     # 作者
header-img: img/post-bg-unix-linux.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Docker
---

# Docker安装MariaDB与启动MariaDB

Docker是现在比较火的一个工具，简单的来说就是把你需要的工具安装在一个虚拟机当中，然后映射到你本机的端口中。

## 安装MariaDB

安装MariaDB其实是很简单的，就是Pull一下镜像就可以了。

```shell
docker search mariadb
docker pull mariadb
```

等待Pull完就可以了。

## 启动MariaDB

查看当前机器上的所有镜像。

```shell
docker images
```

在本机建立一个目录与容器映射。

```shell
mkdir -p ~/docker/mariadb
```

```shell
docker run --name mariadb -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -v /Users/guofuzheng/docker/mariadb:/var/lib/mysql -d mariadb
```

> 初次设置MYSQL_ROOT_PASSWORD是ROOT用户的密码
>
> /Users/guofuzheng/docker/mariadb用来映射容器的的本机目录，必须是绝对路径。

```shell
docker ps -a
```

可以查看容器的运行状态，同时可以查看容器ID。

```shell
docker container update
```

用来更新容器

```shell
docker exec -it 容器Id bash
```

可以进入容器的终端，注意*这里的容器ID可以只用前两位*。

```shell
mysql -uroot -p
```

这可以进入终端的数据库，后面会让输入数据库的连接密码。

常用命令：

```shell
docker start 容器id　
docker stop 容器id　
```

