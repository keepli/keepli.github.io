Mybatis是一个ORM框架，ORM是（ **Object Relational Mapping**  ）的简写，翻译过来就是 **对象-关系映射** ，简单来说就是把数据库和实体类及实体类的属性对应起来，让我们可以操作实体类就能实现操作数据库。

在分析底层原理之前，先看单独使用Mybatis时的执行流程：

```java
public class MybatisTest {
    /**
     * 入门案例
     * @param
     */
    @Test
    public void test1() throws IOException {
        //1.读取主配置文件
        InputStream is = Resources.getResourceAsStream ( "mybatis.xml" );
        //2.创建SqlSessionFactory工厂
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder ( );
        SqlSessionFactory factory = builder.build ( is );
        //3.使用工厂生产一个SqlSession对象
        SqlSession sqlSession = factory.openSession ( );
        //4.使用SqlSession创建Dao接口的代理对象
        UserDao userDao = sqlSession.getMapper ( UserDao.class );
        //5.使用代理对象执行方法
        List<User> users = userDao.findAll ( );

        /*String sqlId = "com.itheima.dao.UserDao"+"."+"findAll";
        List<User> users = sqlSession.selectList ( sqlId );*/
        for (User user : users) {
            System.out.println (user );
        }
        //6.释放资源
        sqlSession.close ();
        is.close ();
    }
}
```

以下是我通过debug调试得到的结果：

![](https://img-blog.csdnimg.cn/2021031116072717.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center)

抛开我们使用mybatis时这几步执行流程来说，在mybatis底层最重要的两个对象就是`configuration`和`transaction`。

**configuration是mybatis配置对象，它是用来存储从xml配置文件中解析得到的所有数据，它里面有非常重要的三个属性：**

- environment
- mapperRegistry
- mappedStatements

<img src="https://img-blog.csdnimg.cn/20210311164146769.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70" style="zoom: 50%;" />

**transaction是mybatis的事务管理对象，如果在配置文件中配置了事务管理，那么由同一个sqlSession创建的代理对象，在执行CRUD操作时每次获取到的connection将会是同一个，属性如下：**

- connection
- dataSource
- level
- autoCommit

![](https://img-blog.csdnimg.cn/20210311180809643.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70)

**这两个对象创建的时机：**

`configuration`对象在创建`sqlSessionFactoryBuild`时进行创建，`transaction`对象在创建`sqlSession`时创建，并且`transaction`中包含有`configuration`对象，不同的`sqlSession`它的`transaction`也不相同，但是`configuration`对象是同一个。