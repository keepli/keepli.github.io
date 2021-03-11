---
title: Navicat生成指定sql文件版本
date: 2020-10-16
updated: 2020-10-16
tags: mysql
categories: mysql
---

*mysql在数据备份时因为mysql版本不同，导致于高版本数据库生成的sql文件，放到低版本数据库中不能执行，在Navicat中生成sql文件时是可以指定版本的，那么就可以解决这个问题*

### 1.查看要导入数据的mysql版本号

```shell
mysql -V #执行该命令可以在不登录的情况下查看mysql版本，注意V一定要大写
```

### 2.生成对应版本的sql文件

- 选择数据库后找到Navicat的数据传输

<img src="https://img-blog.csdnimg.cn/20210107103536445.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70" style="zoom: 33%;" />

- 指定相应的sql文件版本

<img src="https://img-blog.csdnimg.cn/20210107103921191.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70" style="zoom:33%;" />

> 完成以上步骤就大工告成了，不过在不同的Navicat版本中会有些不同，我的是12.0的版本