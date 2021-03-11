---
title: maven工程依赖不自动下载或更新解决方法
date: 2020-11-9
updated: 2020-11-9
tags: maven
categories: maven
---

 **大家有没有碰到过，点刷新和手动下载之后，还是一堆红波浪线，并没有自动下载依赖。**

<img src="https://img-blog.csdnimg.cn/20210115165752142.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70" style="zoom:50%;" />

**解决方法：** 

<img src="https://img-blog.csdnimg.cn/20210115165807240.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70" style="zoom:50%;" />

 **点击如图的按钮，执行一个maven命令即可：**

```shell
mvn -U idea:idea
```



**如果是使用 <dependencyManagement> 标签管理的依赖，执行完以上命令后，在pom文件中可能还会存在一些jar包没被下载。如下图所示：**

<img src="https://img-blog.csdnimg.cn/20210115172318538.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70" style="zoom:50%;" />

**这是因为这里面只是声明一个依赖，并不是真实的下载jar，只有在子module中使用到，才会去下载依赖。** 