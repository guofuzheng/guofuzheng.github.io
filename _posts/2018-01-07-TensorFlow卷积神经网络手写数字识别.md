---
layout:     post                    # 使用的布局（不需要改）
title:      利用TensorFlow手写数字识别               # 标题 
subtitle:   利用TensorFlow手写数字识别 #副标题
date:       2018-01-07            # 时间
author:     GFZ                     # 作者
header-img: img/post-bg-unix-linux.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - 机器学习
    - TensorFlow
---
# 前言
老师让我们写神经网络作业，所以先用TensorFlow写一个手写数字的卷积神经网络。
> 一点小建议，数据集最好先自己下载下来，防止网络问题。

# 引入相关包和数据
```python
import tensorflow as tf
from tensorflow.examples.tutorials.mnist import input_data
mnist = input_data.read_data_sets('./data',one_hot=True)
```
# 定义准确率函数
```python
def compute_accuracy(v_xs,v_ys):
    global prediction
    y_pre = sess.run(prediction,feed_dict={xs:v_xs,keep_prob:1})
    correct_prediction = tf.equal(tf.argmax(y_pre,1),tf.argmax(v_ys,1))
    accuracy = tf.reduce_mean(tf.cast(correct_prediction,tf.float32))
    result = sess.run(accuracy,feed_dict={xs:v_xs,ys:v_ys,keep_prob:1})
    return result
```
# 定义权重函数
```python
def weight_variable(shape):
    initial = tf.truncated_normal(shape,stddev=0.1)#类似随机变量生成 stddev样本标准偏差
    return tf.Variable(initial)
```
#定义偏置函数
```python
def bias_variable(shape):
    initial = tf.constant(0.1,shape=shape)
    return tf.Variable(initial)
```
# 定义卷积层
```python
def conv2d(x,w):
    return tf.nn.conv2d(x,w,strides=[1,1,1,1],padding='SAME')#strides 第一个和最后一个一定要等于1，中间两个是x和y的跨度

```
# 定义池化层（最大化池）
```python
def max_pool_2x2(x):
    return tf.nn.max_pool(x,ksize=[1,2,2,1],strides=[1,2,2,1],padding='SAME')#ksize第一个和最后一个都要等于1，中间两个是x和y的跨度

```
# 准备数据
```python
xs = tf.placeholder(tf.float32,[None,784])#28*28
ys = tf.placeholder(tf.float32,[None,10])
keep_prob = tf.placeholder(tf.float32)
x_image = tf.reshape(xs,[-1,28,28,1])#-1不管多少个数据
```
# 第一卷积层加池化
```python
#conv1 layer
w_conv1 = weight_variable([5,5,1,16])#5x5卷积核 1 是输入颜色通道 16是输出颜色通道
b_conv1 = bias_variable([16])
h_conv1 = tf.nn.relu(conv2d(x_image,w_conv1)+b_conv1) #ouput size 28*28*16
h_pool1 = max_pool_2x2(h_conv1)                                    #output size 14*14*16
```
> 因为卷积层的padding为SAME所以卷积后输出的大小是和原图像一样的矩阵大小,池化跨度为2所以大小变成长和宽的一半

# 第二卷积层加池化
```python
#conv2 layer
w_conv2 = weight_variable([5,5,16,32])#5x5卷积核 16 是输入颜色通道 32是输出颜色通道
b_conv2 = bias_variable([32])
h_conv2 = tf.nn.relu(conv2d(h_pool1,w_conv2)+b_conv2) #ouput size 14*14*32
h_pool2 = max_pool_2x2(h_conv2)                       #output size 7*7*32
```
# 第一全连接
```python
#func1 layer
w_fc1 = weight_variable([7*7*32,500])
b_fc1 = bias_variable([500])
h_pool2_flat = tf.reshape(h_pool2,[-1,7*7*32])
h_fc1 = tf.nn.relu(tf.matmul(h_pool2_flat,w_fc1)+b_fc1)
h_fc1_drop = tf.nn.dropout(h_fc1,keep_prob)
```
# 第二全连接
```python
#func2 layer
w_fc2 = weight_variable([500,10])
b_fc2 = bias_variable([10])
prediction = tf.nn.softmax(tf.matmul(h_fc1_drop,w_fc2)+b_fc2)
```
# 计算误差
```python
#计算误差
cross_entropy = -tf.reduce_sum(ys*tf.log(prediction))
train_step = tf.train.AdamOptimizer(1e-4).minimize(cross_entropy)
```
# 初始化参数
```python
sess = tf.Session()
init = tf.global_variables_initializer()
sess.run(init)
```
# 训练与测试
```python
for i in range(1000):
    batch_xs,batch_ys = mnist.train.next_batch(100)
    sess.run(train_step,feed_dict={xs:batch_xs,ys:batch_ys,keep_prob:0.5})
    if i%40==0:
        print(compute_accuracy(mnist.test.images[:1000],mnist.test.labels[:1000]))
```
> keep_prob为dropout的保存节点的概率

最终结果为95.6%
# 完整代码
```python
import tensorflow as tf
from tensorflow.examples.tutorials.mnist import input_data
mnist = input_data.read_data_sets('./data',one_hot=True)
#定义准确率函数
def compute_accuracy(v_xs,v_ys):
    global prediction
    y_pre = sess.run(prediction,feed_dict={xs:v_xs,keep_prob:1})
    correct_prediction = tf.equal(tf.argmax(y_pre,1),tf.argmax(v_ys,1))
    accuracy = tf.reduce_mean(tf.cast(correct_prediction,tf.float32))
    result = sess.run(accuracy,feed_dict={xs:v_xs,ys:v_ys,keep_prob:1})
    return result
def weight_variable(shape):
    initial = tf.truncated_normal(shape,stddev=0.1)#类似随机变量生成 stddev样本标准偏差
    return tf.Variable(initial)
def bias_variable(shape):
    initial = tf.constant(0.1,shape=shape)
    return tf.Variable(initial)
def conv2d(x,w):
    return tf.nn.conv2d(x,w,strides=[1,1,1,1],padding='SAME')#strides 第一个和最后一个一定要等于1，中间两个是x和y的跨度
def max_pool_2x2(x):
    return tf.nn.max_pool(x,ksize=[1,2,2,1],strides=[1,2,2,1],padding='SAME')#ksize第一个和最后一个都要等于1，中间两个是x和y的跨度

xs = tf.placeholder(tf.float32,[None,784])#28*28
ys = tf.placeholder(tf.float32,[None,10])
keep_prob = tf.placeholder(tf.float32)
x_image = tf.reshape(xs,[-1,28,28,1])#-1不管多少个数据
#conv1 layer
w_conv1 = weight_variable([5,5,1,16])#5x5卷积核 1 是输入颜色通道 16是输出颜色通道
b_conv1 = bias_variable([16])
h_conv1 = tf.nn.relu(conv2d(x_image,w_conv1)+b_conv1) #ouput size 28*28*16
h_pool1 = max_pool_2x2(h_conv1)                       #output size 14*14*16
#conv2 layer
w_conv2 = weight_variable([5,5,16,32])#5x5卷积核 16 是输入颜色通道 32是输出颜色通道
b_conv2 = bias_variable([32])
h_conv2 = tf.nn.relu(conv2d(h_pool1,w_conv2)+b_conv2) #ouput size 14*14*32
h_pool2 = max_pool_2x2(h_conv2)                       #output size 7*7*32
#func1 layer
w_fc1 = weight_variable([7*7*32,500])
b_fc1 = bias_variable([500])
h_pool2_flat = tf.reshape(h_pool2,[-1,7*7*32])
h_fc1 = tf.nn.relu(tf.matmul(h_pool2_flat,w_fc1)+b_fc1)
h_fc1_drop = tf.nn.dropout(h_fc1,keep_prob)
#func2 layer
w_fc2 = weight_variable([500,10])
b_fc2 = bias_variable([10])
prediction = tf.nn.softmax(tf.matmul(h_fc1_drop,w_fc2)+b_fc2)
#计算误差
cross_entropy = -tf.reduce_sum(ys*tf.log(prediction))
train_step = tf.train.AdamOptimizer(1e-4).minimize(cross_entropy)
sess = tf.Session()
init = tf.global_variables_initializer()
sess.run(init)
for i in range(1000):
    batch_xs,batch_ys = mnist.train.next_batch(100)
    sess.run(train_step,feed_dict={xs:batch_xs,ys:batch_ys,keep_prob:0.5})
    if i%40==0:
        print(compute_accuracy(mnist.test.images[:1000],mnist.test.labels[:1000]))

```
