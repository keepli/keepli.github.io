---
title: hexo博客访问部分页面404的解决方法
date: 2020-04-10
updated: 2020-04-10
tags: hexo
categories: hexo
---

*本地启动访问正常，部署之后访问部分页面出现404，主要因为部署之后修改了文件名、标签、分类等等信息，因为缓存原因导致部署之后访问出现404*

<!-- more -->

---

## 解决方法

#### 1.把所有的404的文章全部从 `_posts`目录中移出去

#### 2.重新部署

```shell
hexo clean && hexo g && hexo d 
```

#### 3.把之前移出去的文章再次移回来

#### 4.重新部署

```shell
hexo clean && hexo g && hexo d 
```

