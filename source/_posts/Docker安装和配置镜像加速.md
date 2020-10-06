---
title: Docker安装和配置镜像加速
date: 2020-01-24 
updated: 2020-01-24 
tags: docker
categories: docker

---

*Docker 是一个开源的应用容器引擎，诞生于 2013 年初，基于 Go 语言实现， dotCloud 公司出品（后改名为Docker Inc） Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的 Linux 机器上*

<!-- more -->

### 一、安装Docker

```shell
# 1、yum 包更新到最新 
yum update
# 2、安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的 
yum install -y yum-utils device-mapper-persistent-data lvm2
# 3、 设置yum源
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# 4、 安装docker，出现输入的界面都按 y 
yum install -y docker-ce
# 5、 查看docker版本，验证是否验证成功
docker -v

```

### 二、配置镜像加速

因为docker下载镜像时默认都是通过外网下载，速度慢的要死，所以自己还是要配置国内镜像地址

**我使用的是阿里云镜像加速，在阿里云控制台找到容器镜像服务中的镜像加速器，可以看到自己的加速地址与操作步骤**

![](https://img-blog.csdnimg.cn/20201006150308592.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center)













