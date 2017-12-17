---
layout:     post                    # 使用的布局（不需要改）
title:      利用TensorFlow写逻辑回归               # 标题 
subtitle:   利用TensorFlow写逻辑回归 #副标题
date:       2017-12-17              # 时间
author:     GFZ                     # 作者
header-img: img/post-bg-unix-linux.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - 机器学习  - TensorFlow
---
# 前言
TensorFlow现在贼火，所以我就尝试着用TensorFlow去写一个拟合y = 0.3*x+0.1的代码。
# 引入相关包
```python
import tensorflow as tf
import numpy as np
```
# 定义数据
```Python
x_data = np.random.rand(100,1).astype(np.float32)
y_data = 0.3*x_data+0.1
```
# 定义权重和偏置
```Python
weight = tf.Variable(tf.random_uniform([1],-1.0,1.0))
bias = tf.Variable(tf.zeros([1]))
```
# 定义误差函数
```Python
y = weight*x_data+bias
loss = tf.reduce_mean(tf.square(y-y_data))
optimizer = tf.train.GradientDescentOptimizer(0.5)
train = optimizer.minimize(loss)
```
# 初始化所有参数
```Python
init = tf.initialize_all_variables()
sess = tf.Session()
sess.run(init)
```
# 开始训练并打印误差和参数
```Python
for i in range(200):
    sess.run(train)
    if i%20==0:
        print(i,sess.run(loss))
print(sess.run(weight),sess.run(bias))
```
> TensorFlow中所有的变量都需要通过sess.run()来输出
