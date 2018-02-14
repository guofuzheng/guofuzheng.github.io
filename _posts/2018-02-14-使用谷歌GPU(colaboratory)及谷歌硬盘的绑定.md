---
layout:     post                    # 使用的布局（不需要改）
title:      使用谷歌GPU(colaboratory)及谷歌硬盘配置              # 标题 
subtitle:   使用谷歌GPU(colaboratory)及谷歌硬盘配置 #副标题
date:       2018-02-14             # 时间
author:     GFZ                     # 作者
header-img: img/post-bg-unix-linux.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - 机器学习
    - 环境配置
---
# 前言
我想在使用这一系列的工具之前，首先得拥有一个谷歌账号。
> 关于谷歌账号的注册，有一个地方要注意，在填写手机的时候选中国，在填写位置的时候填写VPN的国家。同时，我之前遇到过用自己的邮箱死活发不了验证码，注册谷歌自家的邮箱，就一次成功了。  

# 开始设置
我们首先登陆[谷歌移动硬盘](https://drive.google.com)，添加与colaboratory的关联
![](https://ws1.sinaimg.cn/large/006tNc79gy1fog9pcr6u2j30zu0sc414.jpg)
我这个是已经添加了关联，如果没有添加关联可以在关联更多应用里选择colaboratory进行关联，我们可以直接新建ipynb文件，也可以上传ipynb文件。
# 创建项目文件夹
我们只需要在网页右击选择创建文件夹就可以创建文件夹。
![](https://ws1.sinaimg.cn/large/006tNc79gy1fog9xjxkf0j30hq0fugma.jpg)
# 上传相关文件
我们右击网页就可以看到上传文件，如果使用的是Chrome浏览器还可以上传文件夹。
# 关联谷歌硬盘
在项目文件夹里的代码前加上如下几段代码。
```
!apt-get install -y -qq software-properties-common python-software-properties module-init-tools   
!add-apt-repository -y ppa:alessandro-strada/ppa 2>&1 > /dev/null  
!apt-get update -qq 2>&1 > /dev/null  
!apt-get -y install -qq google-drive-ocamlfuse fuse  
from google.colab import auth  
auth.authenticate_user()  
from oauth2client.client import GoogleCredentials
creds = GoogleCredentials.get_application_default()
import getpass
!google-drive-ocamlfuse -headless -id={creds.client_id} -secret={creds.client_secret} < /dev/null 2>&1 | grep URL
vcode = getpass.getpass()
!echo {vcode} | google-drive-ocamlfuse -headless -id={creds.client_id} -secret={creds.client_secret}
```
网页会跳出一个连接，让你点击登录，并把相关的代码复制粘贴进输入框，重复一次后，就设置成功了。
![](https://ws4.sinaimg.cn/large/006tNc79gy1foga3tkogdj31kw04b40p.jpg)
之后再输入以下代码并运行。  

```
!mkdir -p drive
!google-drive-ocamlfuse drive
```
> 完成后我们的colaboratory 就接管了我们自己的谷歌移动硬盘，根目录名为drive。就是说，如果我们在谷歌移动硬盘中新建了一个名为hello的文件夹，那么路径就应该为drive/hello

如果我们想把路径切换到项目的目录，那么就运行以下代码。
```
import os
os.chdir("drive/您的项目路径")
```  
这个时候我们就可以在谷歌的GPU上运行项目了，如果不关联谷歌硬盘也是可以的，但是不能运行有自己数据的项目。比如手写数字识别就可以直接跑。
> 不推荐上传压缩文件然后在谷歌移动硬盘解压缩，有时候会丢失文件，直接上传文件夹就可以，虽然速度有点慢🤣。