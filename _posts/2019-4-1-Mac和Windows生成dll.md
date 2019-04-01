---
layout:     post                    # 使用的布局（不需要改）
title:      Mac和Windows生成Dll            # 标题 
subtitle:  Mac和Windows生成Dll #副标题
date:       2019-4-1           # 时间
author:     GFZ                     # 作者
header-img: img/post-bg-coffee.jpeg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - C++
---
# 前言
最近在做一个项目的时候，老师让我把核心代码写到DLL中就是动态链接库里面。然后用Python来调用，这样就不会泄露核心代码了。
# Mac操作
## 创建项目
用Mac来做这些操作的话就相对简单一点。我们用的是Clion，就是jetbrain的C++编辑器。
首先创建一个项目，如下图：
![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1mxah0cw7j31720qkmzv.jpg)
## 创建CPP
在main.cpp 中写入如下代码：  

```
#include <iostream>
#include "main.h"
using namespace std;
void hello(){
    std::cout << "hello,world" << endl;

};
int sum(int a,int b){
    return (a+b);
};
```
## 创建h
在项目中新建一个main.h文件，然后在其中写入：  

```
#ifdef __cplusplus
extern "C"{
#endif


#ifndef MYLIBTEST_MAIN_H
#define MYLIBTEST_MAIN_H
void hello();
int sum(int a,int b);
#endif //MYLIBTEST_MAIN_H
#ifdef __cplusplus
};
#endif
```
## CmakeLists
最后就是CmakeLists文件，写入如下：  

```
cmake_minimum_required(VERSION 3.9)
project(MyLibTest)
set(CMAKE_CXX_STANDARD 11)
set(MYLIBLIB main.h main.cpp)
add_library(MyLibTest SHARED ${MYLIBLIB})
```
如果有cmake的版本不对，比指定的低，就把第一行的版本改成你自己的版本就好了。
## 编译操作
切到对应目录
输入

```
cmake .
make
```
如下图:
![](https://ws1.sinaimg.cn/large/006tKfTcgy1g1mxk0wzldj31bx0u0tid.jpg)
![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1mxk8dg90j31da07a0v7.jpg)
这样之后就会在对应的项目目录有一个dylib的文件，用Python调用这个文件就可以了。
## Py调用
在动态库生成成功之后，就可以使用了，但是这个dylib只能在Mac使用，Windows是dll，Linux是so。
对应的Py代码如下：  

```
import ctypes
dll = ctypes.CDLL("./libmylib.dylib")
print(dll.add(1,2))
dll.hello()
```
运行的结果如下：
![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1mxny2ygtj31d203smxq.jpg)
# Win

Windows下就是使用万能的VS就可以了，我用的是VS2015.
## 创建项目
首先使用VS来创建C++项目。
![](https://ws2.sinaimg.cn/large/006tKfTcgy1g1n22phjwxj30qj0icgm1.jpg)
进行命名之后就是详细的设置。
![](https://ws4.sinaimg.cn/large/006tKfTcgy1g1n23x9g0hj30j10g70sp.jpg)
这里要设置成空项目，不然就会有乱七八糟的各种文件。
点击完成后，就在项目中添加**dll.cpp**,**dll.h**。
## 编辑代码
dll.cpp:   
   
```
#include <windows.h>#include "dll.h"#include <iostream>using namespace std;EXPORT int sum(int a, int b){	return a + b;};EXPORT void hello() {	printf("%s\n", "hello,world");}
```
dll.h:  

```
#pragma once#ifndef DLLTEST_H#define DLLTEST_H#ifdef __cplusplus#define EXPORT extern "C" __declspec (dllexport)#else#define EXPORT __declspec (dllexport)#endifEXPORT int sum(int a, int b);EXPORT void hello();#endif
```
这里完成之后，就可以生成解决方案了。
但是，这里有一个一定要注意的点。
**x86一定要换成x64，不然就会生成32位的，Python调用会出错，不是有效的win32程序**
![](https://ws3.sinaimg.cn/large/006tKfTcgy1g1n28llsh8j30oq0303ye.jpg)
这样在项目的文件夹下面就会有dll文件了。

