---
layout:     post                    # 使用的布局（不需要改）
title:      Keras中Adam优化器学习率衰减               # 标题 
subtitle:   Keras中Adam优化器学习率衰减 #副标题
date:       2018-10-29              # 时间
author:     GFZ                     # 作者
header-img: img/post-bg-unix-linux.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - 机器学习
    - keras
---
# 前言
之前看小伙伴们用pytorch都可以设置学习率衰减，我的Keras代码的学习率就老是0.0001，就很不爽，今天扒了下Keras里 *optimizers.py* 的代码。
# 更新方式
代码里lr的更新方式是这么写的
```
if self.initial_decay > 0:
            lr *= (1. / (1. + self.decay * K.cast(self.iterations,
                                                  K.dtype(self.decay))))
```
就是更新方式是 lr=1/(1+decay x iteration) x lr
所以，如果跑100个epoch，我觉得最好设置大于0.01，不然我感觉没什么效果。