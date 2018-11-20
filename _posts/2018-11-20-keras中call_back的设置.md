---
layout:     post                    # 使用的布局（不需要改）
title:      Keras 中 call_back的设置              # 标题 
subtitle:   Keras 中 call_back的设置   #副标题
date:       2018-11-20           # 时间
author:     GFZ                     # 作者
header-img: img/post-bg-coffee.jpeg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Keras
    - 深度学习
---
# 前言
这篇文章不是我写的，是我转载的，原文章是[csdn](https://blog.csdn.net/weiyongle1996/article/details/79342875)。我怕以后还得找，就放上来了。
# 保存模型结构、训练出来的权重、及优化器状态
keras的callback参数可以帮助我们实现在训练过程中的适当时机被调用。实现实时保存训练模型以及训练参数。  

```
keras.callbacks.ModelCheckpoint(    filepath,     monitor='val_loss',     verbose=0,     save_best_only=False,     save_weights_only=False,     mode='auto',     period=1)1. filename：字符串，保存模型的路径2. monitor：需要监视的值3. verbose：信息展示模式，0或14. save_best_only：当设置为True时，将只保存在验证集上性能最好的模型5. mode：‘auto’，‘min’，‘max’之一，在save_best_only=True时决定性能最佳模型的评判准则，例如，当监测值为val_acc时，模式应为max，当检测值为val_loss时，模式应为min。在auto模式下，评价准则由被监测值的名字自动推断。6. save_weights_only：若设置为True，则只保存模型权重，否则将保存整个模型（包括模型结构，配置信息等）7. period：CheckPoint之间的间隔的epoch数
```  
# 当验证损失不再继续降低时，如何中断训练？当监测值不再改善时中止训练
用EarlyStopping回调函数    

```  
from keras.callbacksimport EarlyStopping keras.callbacks.EarlyStopping(    monitor='val_loss',     patience=0,     verbose=0,     mode='auto')model.fit(X, y, validation_split=0.2, callbacks=[early_stopping])1. monitor：需要监视的量2. patience：当early stop被激活（如发现loss相比上一个epoch训练没有下降），则经过patience个epoch后停止训练。3. verbose：信息展示模式4. mode：‘auto’，‘min’，‘max’之一，在min模式下，如果检测值停止下降则中止训练。在max模式下，当检测值不再上升则停止训练。
```
# 学习率动态调整
keras.callbacks.LearningRateScheduler(schedule) schedule：函数，该函数以epoch号为参数（从0算起的整数），返回一个新学习率（浮点数） 也可以让keras自动调整学习率    


```  
keras.callbacks.ReduceLROnPlateau(    monitor='val_loss',     factor=0.1,     patience=10,     verbose=0,     mode='auto',     epsilon=0.0001,     cooldown=0,     min_lr=0)1. monitor：被监测的量2. factor：每次减少学习率的因子，学习率将以lr = lr*factor的形式被减少3. patience：当patience个epoch过去而模型性能不提升时，学习率减少的动作会被触发4. mode：‘auto’，‘min’，‘max’之一，在min模式下，如果检测值触发学习率减少。在max模式下，当检测值不再上升则触发学习率减少。5. epsilon：阈值，用来确定是否进入检测值的“平原区”6. cooldown：学习率减少后，会经过cooldown个epoch才重新进行正常操作7. min_lr：学习率的下限  
```
# 示例，多个回调函数用逗号隔开
```
# checkpointcheckpointer = ModelCheckpoint(filepath="./checkpoint.hdf5", verbose=1)# learning rate adjust dynamiclrate = ReduceLROnPlateau(min_lr=0.00001)answer.compile(optimizer='rmsprop', loss='categorical_crossentropy',               metrics=['accuracy'])# Note: you could use a Graph model to avoid repeat the input twiceanswer.fit(    [inputs_train, queries_train, inputs_train], answers_train,    batch_size=32,    nb_epoch=5000,    validation_data=([inputs_test, queries_test, inputs_test], answers_test),    callbacks=[checkpointer, lrate])
```