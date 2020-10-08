---
title: springboot整合druid数据源
date: 2020-01-22 
updated: 2020-01-22 
tags: [springboot,druid]
categories: jdbc
---

 *springboot2.X版整合druid数据源配置*

<!-- more -->

---

- maven依赖

```xml
<!--druid数据源-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.17</version>
        </dependency>
```

- springboot的`yml`配置文件

```yml
spring:
  datasource:
    #使用的是druid数据源
    druid:
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://ip地址/数据库名?characterEncoding=utf-8
      username: root
      password: root
```

