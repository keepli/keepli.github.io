---
title: HEXO中如何支持mermaid图表
date: 2020-03-15 12:30:36
updated: 2020-03-16 11:40:30
tags: hexo
categories: hexo
---

 *使用mermaid图表的人都说好，但是hexo不支持，以下是我亲自试用开启hexo对mermaid图表支持，大家可以放心、安全、无毒使用！！！*

<!-- more -->

---



#### 一、首先想要HEXO支持 mermaid 图表需要安装一个插件：

- 执行npm命令

```shell
npm install hexo-filter-mermaid-diagrams
```
#### 二、 编辑*_config.yml* 配置文件

-  在 *_config.yml*配置文件中<font color=red>（根目录下的 ）</font>的最后加上以下内容 

```yml
# mermaid chart
mermaid: ## mermaid url https://github.com/knsv/mermaid
  enable: true  # default true
  version: "7.1.2" # default v7.1.2
  options:  # find more api options from https://github.com/knsv/mermaid/blob/master/src/mermaidAPI.js
    #startOnload: true  // default true
```

- 需要注意的是检查<font color=red>主题中</font>的*_config.yml*配置文件是否有mermaid的配置<font color=red>（next主题）</font>
- 如果有的话默认是一下配置，需要把enable改为`true`，否则插件不会被启用

```yml
# Mermaid tag
mermaid:
  enable: false 
  # Available themes: default | dark | forest | neutral
  theme: forest
```

#### 三、 引入相关的js文件

-  找到主题里面的页脚文件， `themes/next/layout/_partials/footer.swig` ，在文件最末尾加入以下内容 

```
{% if (theme.mermaid.enable)  %}
  <script src='https://unpkg.com/mermaid@{{ theme.mermaid.version }}/dist/mermaid.min.js'></script>
  <script>
    if (window.mermaid) {
      mermaid.initialize({theme: 'forest'});
    }
  </script>
{% endif %}
```

#### 四、配置完成之后执行三部曲：

```shell
hexo clean # 清理
hexo g	   # 构建
hexo s     # 运行
```



#### 五、最后献上几个链接供参考：

`官网：`

>  [https://github.com/webappdevelp/hexo-filter-mermaid-diagrams ](https://github.com/webappdevelp/hexo-filter-mermaid-diagrams )

`参考文献:`

>[https://tyloafer.github.io/posts/7790/  ](https://tyloafer.github.io/posts/7790/    )
>
>[https://rogersnowing.cn/post/38b5106c.html ](https://rogersnowing.cn/post/38b5106c.html )




