---
title: MyBatis-Plus应用
date: 2020-01-19 
updated: 2020-01-19 
tags: mybatis-plus
categories: 持久层框架
top: true 
---

 *MyBatis-Plus是MyBatis的增强版，对经常使用到的SQL中CRUD语法进行的封装，提高了开发效率，并且减少了性能的损耗和繁琐的配置*

<!-- more -->

---



## <font color=red>1. 概述</font>

- 是对 Mybatis框架的二次封装和扩展
- 纯正血统：完全继承原生 Mybatis 的所有特性
- 最少依赖：仅仅依赖 Mybatis以及Mybatis-Spring
- 性能损耗小：启动即会自动注入基本CURD，性能无损耗，直接面向对象操作
- 自动热加载： Mapper对应的xml可以热加载，大大减少重启Web服务器时间，提升开发效率
- 全局拦截：提供全表 delete、update操作智能分析阻断
- 避免 Sql注入：内置Sql注入内容剥离器，预防Sql注入攻击
## <font color=red>2. 配置</font>

#### 1. 在pom.xml文件中引入相关依赖

``` xml
 <!-- mybatis-plus begin -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatisplus-spring-boot-starter</artifactId>
            <version>${mybatisplus-spring-boot-starter.version}</version>
        </dependency>
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus</artifactId>
            <version>${mybatisplus.version}</version>
        </dependency>
<!-- mybatis-plus end -->
```
#### 2. 在配置文件application.yml中添加相关配置

``` yml
# Mybatis-Plus 配置
mybatis-plus:
  #  mapper-locations: classpath:/mapper/*Mapper.xml
  #实体扫描，多个package用逗号或者分号分隔
  typeAliasesPackage: com.tensquare.article.pojo
  global-config:
    id-type: 1  #0:数据库ID自增   1:用户输入id
    db-column-underline: false
    refresh-mapper: true
    configuration:
      map-underscore-to-camel-case: true
      cache-enabled: true #配置的缓存的全局开关
      lazyLoadingEnabled: true #延时加载的开关
      multipleResultSetsEnabled: true #开启延时加载，否则按需加载属性
      log-impl: org.apache.ibatis.logging.stdout.StdOutImpl #打印sql语句,调试用
```
#### 3. 启动类中添加增加@MapperScan注解扫描

``` java
@SpringBootApplication
@MapperScan("com.tensquare_article.dao")
public class ArticleApplication {
    public static void main(String[] args) {
        SpringApplication.run(ArticleApplication.class, args);
    }

    @Bean
    public IdWorker createIdWorker(){
        /**
         * 参数：
         *      workerId:机器编号
         *      datacenterId:序列号
         */
        return new IdWorker (1,1 );
    }
}
```
## <font color=red>3. MVC应用规范</font>

#### 1.实体类要遵守的规范

- @TableName("表名")
- 实现Serializable序列化接口
- @TableId(type = IdType.INPUT)指定主键id

``` java
@TableName("tb_article")
@Data
public class Article implements Serializable {

    @TableId(type = IdType.INPUT)
    private String id;//ID

    private String columnid;    //专栏ID
    private String userid;      //用户ID
    private String title;       //标题
    private String content;     //文章正文
    private String image;       //文章封面
    private Date createtime;    //发表日期
    private Date updatetime;    //修改日期
    private String ispublic;    //是否公开
    private String istop;       //是否置顶
    private Integer visits;     //浏览量
    private Integer thumbup;    //点赞数
    private Integer comment;    //评论数
    private String state;       //审核状态
    private String channelid;   //所属频道
    private String url;         //URL
    private String type;        //类型
}
```

#### 2.持久层dao要遵守的规范

- 只要继承`BaseMapper<T>`接口就行了
- dao接口获得继承方法，`mabatis-plus`自动生成dao实现类

```java
public interface ArticleDao extends BaseMapper<Article> {
}
```

#### 3.业务层service要遵守的规范

- 跟以往使用`mybatis`一样，主要是在调用dao的方法时要调用`mybatis-plus`为我们提供的方法
**案例：通过id修改记录**

```java
public void updateById(Article article) {
        //方式一：根据id修改
        //articleDao.updateById ( article );

        //方式二：根据条件修改
        Wrapper<Article> wrapper = new EntityWrapper<> ( );
        wrapper.eq ( "id",article.getId () );
        articleDao.update ( article, wrapper);//相当于动态添加 and id=article.getId ()
    }
```


#### 4.控制层controller要遵守的规范

- 没有任何变化，跟使用`mybatis`一摸一样

## <font color=red>4. 条件查询和分页查询</font>

#### 1.条件查询

- 使用mybatis-plus提供的`EntityWrapper`对象封装where查询条件

```java
EntityWrapper wrapper = new EntityWrapper<Article>();
wrapper.eq("id", article.getId());

//动态sql，例如<if test="null != field"> and field='xxx' </if>
wrapper.eq(null != map.get(field), field, map.get(field));
```

#### 2.分页

- 使用mybatis-plus提供的`Page`对象

```java
@PostMapping("/search/{page}/{size}")
    public String findByPage(@PathVariable Integer page,
                             @PathVariable Integer size,
                             @RequestBody Map<String,Object> map){
        //根据条件分页查询（service方法的返回类型要为Page）
        Page<Article> pageData = articleService.findByPage(page,size,map);
        
        return "分页查询成功";
    }
```
- 向mybatis-plus中注入`PaginationInterceptor`插件

- 新建 config包，创建MybatisPlusConfig对象，添加下面的代码

```java
@Configuration
public class MybatisPlusConfig {
  @Bean
  public PaginationInterceptor paginationInterceptor() {
    return new PaginationInterceptor();
 	}
}
```

**重点：**

- service层分页代码逻辑

```java
 public Page<Article> findByPage(Integer page, Integer size, Map<String, Object> map) {
        Wrapper<Article> wrapper = new EntityWrapper<> (  );
        Set<String> keys = map.keySet ( );
        for (String key : keys) {
            /*if (map.get ( key )!=null){
                wrapper.eq ( key,map.get ( key ));
            }*/

            //第一个参数是否把后面的条件加入到查询条件中
            //和上面的if判断的写法是一样的效果，实现动态sql
            wrapper.eq ( map.get ( key )!=null,key, map.get ( key ));
        }

        //设置分页参数
        Page<Article> pageData = new Page<> ( page,size );

        //执行查询
        //第一个是分页参数，第二个是查询条件
        List<Article> articleList = articleDao.selectPage ( pageData, wrapper );

        //把list设置到分页参数中
        pageData.setRecords ( articleList );

        return pageData;
    }
```

## <font color=red>5. 具体方法使用</font>

**懒癌发作，直接搬运现成的**

链接地址：

- [官网](https://baomidou.com/)
- [ 简书文章 ](https://www.jianshu.com/p/ceb1df475021)
- [csdn文章](https://blog.csdn.net/zdsg45/article/details/105138493?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522160213305819724848338191%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=160213305819724848338191&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_click~default-1-105138493.pc_first_rank_v2_rank_v28_p&utm_term=mybatis-plus&spm=1018.2118.3001.4187)

