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

## <font color=red>一、MongoDB简介</font>

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

## <font color=red>二、MongoDB常用命令</font>

### 1.选择和创建数据库

> 选择和创建数据库的语法格式：

```shell
use 数据库名称
```

- 如果数据库存在则选择该数据库，如果数据库不存在则自动创建。以下语句创建commentdb数据库：

```
use commentdb
```

> 查看数据库：

```shell
show dbs
```

> 查看集合:

- 需要先选择数据库之后，才能查看该数据库的集合

```shell
show collections
```
### 2.插入与查询文档

> 选择数据库后，使用集合来对文档进行操作，插入文档语法格式：

```shell
db.集合名称.insert(数据);
```

- 插入以下测试数据：

```shell
db.comment.insert({content:"十次方课程",userid:"1011"})
```

> 查询集合的语法格式：

```shell
db.集合名称.find()
```

- 查询comment集合的所有文档，输入以下命令：

```shell
db.comment.find()
```

`发现文档会有一个叫_id的字段，这个相当于我们原来关系数据库中表的主键，当你在插入文档记录时没有指定该字段，MongoDB会自动创建，其类型是ObjectID类型。如果我们在插入文档记录时指定该字段也可以，其类型可以是ObjectID类型，也可以是MongoDB支持的任意类型` 

---

**输入以下测试语句:**

```shell
db.comment.insert({_id:"1",content:"到底为啥出错",userid:"1012",thumbup:2020});
db.comment.insert({_id:"2",content:"加班到半夜",userid:"1013",thumbup:1023});
db.comment.insert({_id:"3",content:"手机流量超了咋办",userid:"1013",thumbup:111});
db.comment.insert({_id:"4",content:"坚持就是胜利",userid:"1014",thumbup:1223});
```

 

按一定条件来查询，比如查询userid为1013的记录，只要在find()中添加参数即可，参数也是json格式，如下：

```shell
db.comment.find({userid:'1013'})
```

只需要返回符合条件的第一条数据，我们可以使用findOne命令来实现：

```shell
db.comment.findOne({userid:'1013'})
```

返回指定条数的记录，可以在find方法后调用limit来返回结果，例如：

```shell
db.comment.find().limit(2)
```

### 3.修改与删除文档

> 修改文档的语法结构：

```shell
db.集合名称.update(条件,修改后的数据)
```

- 修改_id为1的记录，点赞数为1000，输入以下语句：

```shell
db.comment.update({_id:"1"},{thumbup:1000})
```

**执行后发现，这条文档除了thumbup字段其它字段都不见了**

为了解决这个问题，我们需要使用<font color=red>修改器$set</font>来实现，命令如下：

```shell
db.comment.update({_id:"2"},{$set:{thumbup:2000}})
```

> 删除文档的语法结构：

```shell
db.集合名称.remove(条件)
```

- <font color=red>以下语句可以将数据全部删除，慎用~</font>

```shell
db.comment.remove({})
```

- 删除条件可以放到大括号中，例如删除thumbup为1000的数据，输入以下语句：

```bash
db.comment.remove({thumbup:1000})
```

###  4.统计条数

> 统计记录条件使用count()方法。以下语句统计comment集合的记录数：

```shell
db.comment.count()
```

- 按条件统计 ，例如统计userid为1013的记录条数：

```shell
db.comment.count({userid:"1013"})
```

 ### 5.模糊查询

> MongoDB的模糊查询是通过正则表达式的方式实现的：

```
/模糊查询字符串/
```

- 查询评论内容包含“流量”的所有文档，代码如下：

```shell
db.comment.find({content:/流量/})
```

- 查询评论内容中以“加班”开头的，代码如下：

```shell
db.comment.find({content:/^加班/})
```

 ### 6.大于 小于 不等于

> <, <=, >, >= 这个操作符也是很常用的，格式如下:

```shell
db.集合名称.find({ "field" : { $gt: value }}) // 大于: field > value
db.集合名称.find({ "field" : { $lt: value }}) // 小于: field < value
db.集合名称.find({ "field" : { $gte: value }}) // 大于等于: field >= value
db.集合名称.find({ "field" : { $lte: value }}) // 小于等于: field <= value
db.集合名称.find({ "field" : { $ne: value }}) // 不等于: field != value
```

- 查询评论点赞数大于1000的记录：

```shell
db.comment.find({thumbup:{$gt:1000}})
```

 ### 7.包含与不包含

> 包含使用$in操作符

- 查询评论集合中userid字段包含1013和1014的文档：

```shell
db.comment.find({userid:{$in:["1013","1014"]}})
```

> 不包含使用$nin操作符

- 查询评论集合中userid字段不包含1013和1014的文档：

```shell
db.comment.find({userid:{$nin:["1013","1014"]}})
```

 ### 8.条件连接

> 查询同时满足两个以上条件，需要使用$and操作符将条件进行关联（相当于SQL的and）。格式为：

```shell
$and:[ {条件},{条件},{条件} ]
```

- 查询评论集合中thumbup大于等于1000 并且小于2000的文档：

```
db.comment.find({$and:[ {thumbup:{$gte:1000}} ,{thumbup:{$lt:2000} }]})
```

> 如果两个以上条件之间是或者的关系，使用操作符进行关联，与前面and的使用方式相同，格式为：

```shell
$or:[ {条件},{条件},{条件} ]
```

- 查询评论集合中userid为1013，或者点赞数小于2000的文档记录：

```shell
db.comment.find({$or:[ {userid:"1013"} ,{thumbup:{$lt:2000} }]})
```

### 9.列值增长

> 对某列值在原有值的基础上进行增加或减少，可以使用$inc运算符：

- 正数增加

```shell
db.comment.update({_id:"2"},{$inc:{thumbup:1}})
```

- 负数减少

```shell
db.comment.update({_id:"2"},{$inc:{thumbup:-2}})
```

