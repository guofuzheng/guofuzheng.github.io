---
layout:     post                    # 使用的布局（不需要改）
title:      使用Keras实现Tensor的相乘和相加              # 标题 
subtitle:   使用Keras实现Tensor的相乘和相加 #副标题
date:       2018-05-23             # 时间
author:     GFZ                     # 作者
header-img: img/post-bg-debug.png    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Keras
    - 机器学习
---
# 前言
最近在写行为识别的代码，涉及到两个网络的融合，这个融合是有加权的网络结果的融合，所以需要对网络的结果进行加权（相乘）和融合（相加）。
# 最初的想法
最初的想法是用Keras.layers.Add和Keras.layers.Multiplt来做，后来发现这样会报错。
```
rate_rgb = k.variable(np.ones((1024,),dtype='float32')*0.8)
rate_esti = k.variable(np.ones((1024,),dtype='float32')*0.2)
weight_gru1 = Multiply()([rate_rgb,gru1])
weight_gru2 = Multiply()([rate_esti,gru2])
last = Add()([weight_gru1,weight_gru2])
```
这么写会报错，如下  
```
AttributeError: 'Variable' object has no attribute '_keras_history'
```
#  正确做法
后来在网上参考大神的博客，改为如下
```
weight_1 = Lambda(lambda x:x*0.8)
weight_2 = Lambda(lambda x:x*0.2)
weight_gru1 = weight_1(gru1)
weight_gru2 = weight_2(gru2)
fuse = Lambda(lambda x:add(inputs=[x[0],x[1]]))
last = fuse([weight_gru1,weight_gru2])
 ```
 这样就没问题了。