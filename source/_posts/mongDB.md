---
title: MongDB非关系型数据库
date: 2020-04-15 
updated: 2020-04-15 
tags: [mongDB,nosql]
categories: 非关系型数据库
---

 *MongDB做为一款由 C++编写的非关系型数据库（NoSql）,却有着最接近 SQL型数据库的功能，使用成本比 SQL型数据库低，做为评论系统来使用非常不错*

<!-- more -->

---

## <font color=red>MongoDB简介</font>

### 1.文章评论数据分析

-  数据量大
- 写入操作频繁
- 价值较低（评论数据丢失不会有什么太大影响）

对于这样的数据，我们更适合使用MongoDB来实现数据的存储，<font color=red>如果使用mysql来存储成本太高</font>

### 2.什么是MongoDB

- MongoDB是一个基于分布式文件存储的数据库。由C++语言编写。旨在为WEB应
  用提供可扩展的高性能数据存储解决方案
- MongoDB是一个介于关系数据库和非关系数据库之间的产品<font color=red>（自身是非关系型数据库NoSql）</font>，是非关系数据库当中功能最丰富，最像关系数据库的
- 它支持的数据结构非常松散，是类似json的bson格式，因此可以存储比较复杂的数据类型

### 3.MongoDB特点

Mongo最大的特点是它支持的查询语言非常强大，其语法有点类似于面向对象的
查询语言，几乎可以实现类似关系数据库单表查询的绝大部分功能，而且还支持对
数据建立索引

它的特点是**高性能、易部署、易使用，存储数据非常方便**，主要功能特性有：

- 面向集合存储，易存储对象类型的数据
- 模式自由
- 支持动态查询
- 支持完全索引，包含内部对象
- 支持查询
- 支持复制和故障恢复
- 使用高效的二进制数据存储，包括大型对象（如视频等）
- 自动处理碎片，以支持云计算层次的扩展性
- 支持RUBY，PYTHON，JAVA，C++，PHP，C#等多种语言
- 文件存储格式为BSON（一种JSON的扩展）

### 4.MongoDB体系结构

**文档(document)、集合(collection)、数据库(database)三部分组成**

- MongoDB 的文档（document），相当于关系数据库中的一行记录
- 多个文档组成一个集合（collection），相当于关系数据库的表
- 多个集合（collection），逻辑上组织在一起，就是数据库（database）
  
>一个 MongoDB 实例支持多个数据库（database）

**对照表：**

| MongoDB体系结构   | MySql体系结构     |
| ----------------- | ----------------- |
| 数据库(databases) | 数据库(databases) |
| 集合(collections) | 表(table)         |
| 文档(document)    | 行(row)           |

### 5.MongoDB数据类型

<table><thead><tr><th><span>数据类型</span></th><th><span>描述</span></th></tr></thead><tbody><tr><td><span>String</span></td><td><span>字符串。存储数据常用的数据类型。在  MongoDB 中，UTF-8 编码的字符串才是合法的。</span></td></tr><tr><td><span>Integer</span></td><td><span>整型数值。用于存储数值。根据你所采用的服务器，可分为  32 位或 64 位。</span></td></tr><tr><td><span>Boolean</span></td><td><span>布尔值。用于存储布尔值（真/假）。</span></td></tr><tr><td><span>Double</span></td><td><span>双精度浮点值。用于存储浮点值。</span></td></tr><tr><td><span>Array</span></td><td><span>用于将数组或列表或多个值存储为一个键。</span></td></tr><tr><td><span>Timestamp</span></td><td><span>时间戳。记录文档修改或添加的具体时间。</span></td></tr><tr><td><span>Object</span></td><td><span>用于内嵌文档。</span></td></tr><tr><td><span>Null</span></td><td><span>用于创建空值。</span></td></tr><tr><td><span>Date</span></td><td><span>日期时间。用  UNIX 时间格式来存储当前日期或时间。你可以指定自己的日期时间：创建 Date 对象，传入年月日信息。</span></td></tr><tr><td><span>Object  ID</span></td><td><span>对象  ID。用于创建文档的 ID。</span></td></tr><tr><td><span>Binary  Data</span></td><td><span>二进制数据。用于存储二进制数据。</span></td></tr><tr><td><span>Code</span></td><td><span>代码类型。用于在文档中存储  JavaScript 代码。</span></td></tr><tr><td><span>Regular  expression</span></td><td><span>正则表达式类型。用于存储正则表达式。</span></td></tr></tbody></table>
特殊说明：

1. **ObjectId**

   ObjectId 类似唯一主键<font color=red>（不指定，会自动生成）</font>，可以很快的去生成和排序，包含 12 bytes，含义是：

   - 前 4 个字节表示创建 unix 时间戳，格林尼治时间 UTC 时间，比北京时间晚了 8 个小时
   - 接下来的 3 个字节是机器标识码
   - 紧接的两个字节由进程 id 组成 PID
   - 最后三个字节是随机数
   
   ![](https://img-blog.csdnimg.cn/20201009015303923.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center)
   	
   MongoDB 中存储的文档必须有一个 _id 键。这个键的值可以是任何类型的，默认是个 ObjectId 对象 
   
2. **时间戳** 

   BSON 有一个特殊的时间戳类型，与普通的日期类型不相关。时间戳值是一个 64 位的值。其中： 

   - 前32位是一个 time_t 值【与Unix新纪元（1970年1月1日）相差的秒数】
   - 后32位是在某秒中操作的一个递增的序数

   在单个 mongod 实例中，时间戳值通常是唯一的 

3. **日期** 

    表示当前距离 Unix新纪元（1970年1月1日）的毫秒数。日期类型是有符号的, 负数表示 1970 年之前的日期 

