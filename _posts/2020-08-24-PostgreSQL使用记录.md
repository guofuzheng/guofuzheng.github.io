---
layout:     post                    # 使用的布局（不需要改）
title:      PostgreSQL学习记录               # 标题 
subtitle:   PostgreSQL学习记录 #副标题
date:       2020-08-20              # 时间
author:     GFZ                     # 作者
header-img: img/post-bg-miui-ux.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - PostgreSQL   
    
---   
### 1、创建主键自增
pg中好像没有可以自增的选项，需要自己写：
```sql
CREATE SEQUENCE test_id_auto_increase
    START WITH 1
    INCREMENT BY 1
    NO MINVALUE
    NO MAXVALUE
    CACHE 1;
    
alter table test alter column test_id set default nextval('test_id_auto_increase');
```
这是在创建了数据表后对字段进行修改，`test_id_auto_increase`是这序列生成器的id。
这是在创建表的时候的写法：
```sql
create table test_b
(
  id serial PRIMARY KEY,
  name character varying(128)
); 

NOTICE:  CREATE TABLE will create implicit sequence "test_b_id_seq" for serial column "test_b.id"
NOTICE:  CREATE TABLE / PRIMARY KEY will create implicit index "test_b_pkey" for table "test_b"
CREATE TABLE
```
