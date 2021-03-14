---
title: Mybatis批量插入数据
date: 2020-12-11
updated: 2020-12-11
tags: mybatis
categories: mybatis
---

本文主要讲Mybatis批量插入数据的三种方式效率对比和往数据库批量插入100万条数据实现的思路分析，数据库为MySql。

## <font color=red>一、三种方式批量插入</font>

### <font color=#F39C12>1.for循环insert</font>

``` java
        long start = System.currentTimeMillis();
        for(int i = 0 ;i < 100000; i++) {
            User user = new User();
            user.setId("id" + i);
            user.setName("name" + i);
            user.setPassword("password" + i);
            userMapper.insert(user);
        }
        long end = System.currentTimeMillis();
        System.out.println("---------------" + (start - end) + "---------------");
```
``` xml
    <insert id="insert">
      INSERT INTO t_user (id, name, password)
          VALUES(#{id}, #{name}, #{password})
    </insert>
```

时间为380826ms

### <font color=#F39C12>2.Mybatis batch模式</font>

```java
    SqlSession sqlSession = sqlSessionTemplate.getSqlSessionFactory().openSession(ExecutorType.BATCH, false);//跟上述sql区别
        UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

        long start = System.currentTimeMillis();
        for (int i = 0; i < 100000; i++) {
            User user = new User();
            user.setId("id" + i);
            user.setName("name" + i);
            user.setPassword("password" + i);
            userMapper.insert(user);
        }
        sqlSession.commit();
        long end = System.currentTimeMillis();
        System.out.println("---------------" + (start - end) + "---------------");
```
```xml
    <insert id="insert">
      INSERT INTO t_user (id, name, password)
          VALUES(#{id}, #{name}, #{password})
    </insert>
```
时间为203660ms

### <font color=#F39C12>3.批量foreach插入</font>

```java
      long start = System.currentTimeMillis();
        List<User> userList = new ArrayList<>();
        for (int i = 0; i < 100000; i++) {
            User user = new User();
            user.setId("id" + i);
            user.setName("name" + i);
            user.setPassword("password" + i);
            userMapper.insert(user);
        }
        userMapper.insertBatch(userList);
        long end = System.currentTimeMillis();
        System.out.println("---------------" + (start - end) + "---------------");
```
```xml
    <insert id="insertBatch">
        INSERT INTO t_user
        (id, name, password)
        VALUES
        <foreach collection ="userList" item="user" separator =",">
            (#{id}, #{name}, #{password})
        </foreach >
    </insert>
```

时间为8706ms

> 结论:foreach批量插入 > mybatis batch模式插入 > for循环insert

## <font color=red>二、一次性往数据库插入100万条数据思路分析</font>

### <font color=#F39C12>1.创建数据文件</font>

在数据文件的选择中，我们可以使用表格文件来存储数据，而且表格文件可以跟数据库表做对应，并且方便查看与修改。

### <font color=#F39C12>2.读取数据文件完成批量插入</font>

读取表格文件需要使用**POI**等框架，推荐使用**EasyExcel**来进行读取，使用这些框架读取很方便的是可以直接将读取到的数据给我们封装为实体类，而该实体类又可以和数据库表所对应 ，如此一来我们可以将实体类存到一个List集合中，当List集合长度达到了20或者30的时候则通过上面的方法批量进行插入，插入完毕将List集合里的元素清空，之后就进行for循环操作就可以完成批量插入了。