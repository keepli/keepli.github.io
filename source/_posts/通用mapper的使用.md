---
title: 通用mapper+pagehelper分页插件的使用
date: 2020-07-25 
updated: 2020-07-28
tags: [通用mapper,pagehelper分页插件]
categories: 通用mapper
---

### <font color=#F39C12>一、环境搭建</font>

#### 1.导入maven依赖

```xml
        <!--通用mapper起步依赖-->
        <dependency>
            <groupId>tk.mybatis</groupId>
            <artifactId>mapper-spring-boot-starter</artifactId>
            <version>2.0.4</version>
        </dependency>
```

<!-- more -->

#### 2.dao接口编写

```java
//继承通用mapper提供的接口
public interface BrandMapper extends Mapper<Brand> {
}
```

#### 3.Brand实体类编写

```java
@Table(name="tb_brand")
public class Brand implements Serializable{
	@Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")
	private Integer id;//品牌id

    @Column(name = "name")
	private String name;//品牌名称

    @Column(name = "image")
	private String image;//品牌图片地址
	
    @Column(name = "letter")
	private String letter;//品牌的首字母
	
    @Column(name = "seq")
	private Integer seq;//排序
```

#### 4.配置@MapperScan("dao包路径")

**注意：这个注解是通用mapper提供的**



### <font color=#F39C12>二、查询方法</font>

#### 1.查询所有

```java
selectAll ()//返回List
```

#### 2.通过id查询

```java
selectByPrimaryKey ( id )//返回实体类
```

```java
selectOne ( brand )//返回实体类，参数是实体类
```



### <font color=#F39C12>三、新增方法</font>

#### 1.新增并且算上null值

```java
insert ( brand )
```

#### 2.新增不算上null值

- 带`Selective`是排除`null`值的

```java
insertSelective ( brand )//推荐使用
```

<img src="https://img-blog.csdnimg.cn/20201028115605247.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center"  />



### <font color=#F39C12>四、更新方法</font>

#### 1.更新并且算上null值

```java
updateByPrimaryKey ( brand )
```

#### 2.更新不算上null值

```java
updateByPrimaryKeySelective ( brand )//推荐使用
```



### <font color=#F39C12>五、删除方法</font>

#### 1.通过id删除

```java
deleteByPrimaryKey ( id )
```



### <font color=#F39C12>六、按条件查询</font>

- controller层

```java
    /**
     * 多条件查询
     * @param brand
     * @return
     */
    @PostMapping("/search")
    public Result<List<Brand>> search(@RequestBody Brand brand){
        List<Brand> brandList = brandService.findList ( brand );
        return new Result<> ( true,StatusCode.OK,"根据条件查询品牌成功",brandList);
    }
```

- service层

```java
    public List<Brand> findList(Brand brand) {
        //自定义条件搜索对象 Example
        Example example = new Example ( Brand.class );
        Example.Criteria criteria = example.createCriteria ( );//条件构造器

        if (brand!=null){

            //brand.name!=null 并且不为""
            if (!StringUtils.isEmpty ( brand.getName () )){
                //根据名字模糊查询 where name like '%华为%'
                /*
                    第一个参数是String类型（对应数据表的列名）
                    第二个参数是Object类型（对应的value值）
                */
                criteria.andLike ( "name","%"+brand.getName ()+"%" );
            }

            //brand.letter!=null 并且不为""
            if (!StringUtils.isEmpty ( brand.getLetter () )){
                //根据首字母查询 and letter='H' (因为首字母就一个字符，不用like)
                criteria.andEqualTo ( "letter",brand.getLetter () );
            }
        }
        return brandMapper.selectByExample ( example );
    }
```

> 在使用上建议把条件查询逻辑抽调一个方法，可以方便复用

**更多`criteria`的方法可以查看源码：**

- 以下是截取的一部分源码的方法

```java
        public Example.Criteria andIsNull(String property) {
            this.addCriterion(this.column(property) + " is null");
            return (Example.Criteria)this;
        }

        public Example.Criteria andIsNotNull(String property) {
            this.addCriterion(this.column(property) + " is not null");
            return (Example.Criteria)this;
        }

        public Example.Criteria andEqualTo(String property, Object value) {
            this.addCriterion(this.column(property) + " =", value, this.property(property));
            return (Example.Criteria)this;
        }

        public Example.Criteria andNotEqualTo(String property, Object value) {
            this.addCriterion(this.column(property) + " <>", value, this.property(property));
            return (Example.Criteria)this;
        }

        public Example.Criteria andGreaterThan(String property, Object value) {
            this.addCriterion(this.column(property) + " >", value, this.property(property));
            return (Example.Criteria)this;
        }
```



### <font color=#F39C12>七、分页查询</font>

#### 1.`pagehelper`分页插件

- maven依赖

```xml
        <!--mybatis分页插件-->
        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper-spring-boot-starter</artifactId>
            <version>1.2.3</version>
        </dependency>
```

#### 2.代码实现

- controller层

```java
    /**
     * 分页查询
     * @param page
     * @param size
     * @return
     */
    @GetMapping("/search/{page}/{size}")
    public Result<PageInfo<Brand>> findByPage(@PathVariable Integer page,
                                              @PathVariable Integer size){
        PageInfo<Brand> brandPageInfo = brandService.findByPage ( page, size );
        return new Result<> ( true,StatusCode.OK,"分页查询品牌成功",brandPageInfo);
    }
```

- service层

```java
    public PageInfo<Brand> findByPage(Integer page, Integer size) {
        //开启分页查询
        PageHelper.startPage ( page,size );
        //封装分页bean
        PageInfo<Brand> brandPageInfo = new PageInfo<> ( brandMapper.selectAll ( ) );
        return brandPageInfo;
    }
```



### <font color=#F39C12>八、分页条件查询</font>

```java
    public PageInfo<Brand> findByPageAndList(Integer page, Integer size, Brand brand) {
        //调用抽取出来的条件查询方法获得查询条件
        Example example = createExample ( brand );
        //开启分页查询
        PageHelper.startPage ( page,size );
        //封装分页bean
        PageInfo<Brand> brandPageInfo = new PageInfo<> ( brandMapper.selectByExample ( example ) );
        return brandPageInfo;
    }
```

