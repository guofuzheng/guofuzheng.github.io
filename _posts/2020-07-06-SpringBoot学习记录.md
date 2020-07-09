---
layout:     post                    # 使用的布局（不需要改）
title:      SpringBoot学习记录               # 标题 
subtitle:   SpringBoot学习记录 #副标题
date:       2020-07-06             # 时间
author:     GFZ                     # 作者
header-img: img/post-bg-unix-linux.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - SpringBoot
	- Java
---

### 1 页面无法访问404

如果页面都放在Resources中，但是页面就是一直无法访问，有可能是没有引入模板引擎。

### 2 @Controller和@RestController的区别

首先，@Controller和@RestController的共同点是：用来表示Spring某个类是否可以接受HTTP请求。

他俩的区别是：@Controller标识一个类是Spring MVC Controller处理器。@RestController是@Controller和@RestController的结合体，两个标注合并起来的作用。如果只是使用@RestController注解@Controller的内容，那么页面不会跳转，只会返回引号中的内容。

### 3、关于Mapper的无法注入的问题

在编写数据库查询的Mapper时，需要在该类上加上注解@Mapper，告诉这个类是Mapper。