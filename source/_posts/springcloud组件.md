---
title: springcloud组件
date: 2020-05-13
updated: 2020-05-13
tags: 微服务架构
categories: 微服务架构
---

 *SpringCloud各组件的作用和使用*

<!-- more -->

 ## <font color=red>一、服务注册中心Eureka</font>

### 1. 简介

- Eureka就好比是滴滴，负责管理、记录服务提供者的信息。服务调用者无需自己寻找服务，而是把自己的需求告诉Eureka，然后Eureka会把符合你需求的服务告诉你

- Eureka的主要功能是进行服务管理，定期检查服务状态，返回服务地址列表

![](https://img-blog.csdnimg.cn/20200929174342163.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center)

- Eureka：就是服务注册中心（可以是一个集群），对外暴露自己的地址
- 提供者：启动后向Eureka注册自己信息（地址，提供什么服务）
- 消费者：向Eureka订阅服务，Eureka会将对应服务的所有提供者地址列表发送给消费者，并且定期更新
- 心跳(续约)：提供者定期通过http方式向Eureka刷新自己的状态

### 2. 搭建eureka-server工程

**目标**：添加eureka对应依赖和编写引导类搭建eureka服务并可访问eureka服务界面

**分析**：

Eureka是服务注册中心，只做服务注册；自身并不提供服务也不消费服务。搭建web工程使用Eureka，可以使用Spring Boot方式搭建。

**搭建步骤**：

1. 创建工程
2. 添加启动器依赖
3. 编写启动引导类（添加Eureka的服务注解）和配置文件
4. 修改配置文件（端口，应用名称...）
5. 启动测试

**小结**：

- 启动器依赖（父工程中已经指定了springcloud的版本号，所以这里不用指定版本号）

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
```

- 在springboot的启动类中添加@EnableEurekaServer注解

```java
@EnableEurekaServer//声明当前应用是Eureka服务
@SpringBootApplication
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}  
```

- 配置文件

```yml
server:
  #port: ${port:10086}
  port: 10086
spring:
  application:
    name: eureka-server

eureka:
  client:
    service-url:
      # eureka 服务地址（虚拟的），如果是集群的话；需要指定其它集群eureka地址,后面加逗号隔开
      #defaultZone: ${defaultZone:http://127.0.0.1:10086/eureka}
      #例如：defaultZone: http://127.0.0.1:10086/eureka,http://0.0.0.0:10086/eureka
      defaultZone: http://127.0.0.1:10086/eureka
      # 不注册自己，默认是true，如果是集群搭建需要注册自己
    register-with-eureka: false
      # 不拉取服务,默认是true，集群要为true
    fetch-registry: false
  server:
    #服务失效剔除时间，默认60秒（以毫秒为单位）
    eviction-interval-timer-in-ms: 3000
    #关闭自我保护模式（默认是打开的）
    enable-self-preservation: false
```

### 3.服务注册与发现

**目标**：将user-service的服务注册到eureka并在consumer-demo中可以根据服务名称调用

**分析**：

- 服务注册：在服务提供工程user-service上添加Eureka客户端依赖；自动将服务注册到EurekaServer服务地址列表。
  - 添加依赖；
  - 改造启动引导类；添加开启Eureka客户端发现的注解；
  - 修改配置文件；设置Eureka 服务地址
- 服务发现：在服务消费工程consumer-demo上添加Eureka客户端依赖；可以使用工具类根据服务名称获取对应的服务地址列表。
  - 添加依赖；
  - 改造启动引导类；添加开启Eureka客户端发现的注解；
  - 修改配置文件；设置Eureka 服务地址；
  - 改造处理器类ConsumerController，可以使用工具类DiscoveryClient根据服务名称获取对应服务地址列表。

**小结**：

- 添加Eureka客户端依赖

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
```

- 添加启动引导类注解`@EnableDiscoveryClient`<font color=red>（消费端与生产端都要添加）</font>

  - **以下以生产端启动引导类示例**

```java
@EnableDiscoveryClient//开启Eureka客户端发现功能
@SpringBootApplication
@MapperScan("cn.itcast.user.mapper")
public class UserApplication {

    public static void main(String[] args) {
        SpringApplication.run(UserApplication.class, args);
    }
}
```
- 修改配置

```yml
spring:
  application:
    name: consumer-demo
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka
```

- 在消费端的controller中获取生产端注册的url进行访问（后期一般配合负载均衡来获取，通过生产端的服务名就可获取到url）

### 4. 高可用的Eureka Server（集群搭建）

- 多个Eureka Server之间也会互相注册为服务，当服务提供者注册到Eureka Server集群中的某个节点时，该节点会把服务的信息同步给集群中的每个节点，从而实现数据同步。因此，无论客户端访问到Eureka Server集群中的任意一个节点，都可以获取到完整的服务列表信息

- 如果有三个Eureka，则每一个EurekaServer都需要注册到其它几个Eureka服务中，例如：有三个分别为10086、10087、10088，则：
  - 10086要注册到10087和10088上
  - 10087要注册到10086和10088上
  - 10088要注册到10086和10087上

### 5. Eureka客户端与服务端配置

**目标**：配置eureka客户端user-service的注册、续约等配置项，配置eureka客户端consumer-demo的获取服务间隔时间，了解失效剔除和自我保护

**分析**：

- [Eureka客户端工程](https://img-blog.csdnimg.cn/2020092923374064.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center)
  - user-service 服务提供
    - 服务地址使用ip方式
    - 服务续约（心跳）
  - consumer-demo 服务消费
    - 获取服务地址的频率
- [Eureka服务端工程 eureka-server](https://img-blog.csdnimg.cn/20200929233755362.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center)
  - 失效剔除
  - 自我保护

**小结**：

- user-service 

```yml
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka
  instance:
    # 更倾向使用ip地址，而不是host名
    prefer-ip-address: true
    # ip地址
    ip-address: 127.0.0.1
    # 续约间隔，默认30秒
    lease-renewal-interval-in-seconds: 5
    # 服务失效时间，默认90秒
    lease-expiration-duration-in-seconds: 5
```

- consumer-demo 

```yml
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka
    # 获取服务地址列表间隔时间，默认30秒
    registry-fetch-interval-seconds: 10
```

- eureka-server

```yml
eureka:
  server:
    # 服务失效剔除时间间隔，默认60秒
    eviction-interval-timer-in-ms: 60000
    # 关闭自我保护模式（默认是打开的）
    enable-self-preservation: false
```

## <font color=red>二、负载均衡Ribbon</font>

### 1. 简介

- 负载均衡是一个算法，可以通过该算法实现从地址列表中获取一个地址进行服务调用，在Spring Cloud中提供了负载均衡器：Ribbon

- **Eureka中已经集成了负载均衡组件：Ribbon**，简单修改代码即可使用，不需要单独引入jar包

**小结**：

Ribbon提供了轮询、随机两种负载均衡算法（默认是轮询）可以实现从地址列表中使用负载均衡算法获取地址进行服务调用

### 2. 应用

**目标**：配置启动两个用户服务，在consumer-demo中使用服务名实现根据用户id获取用户

**分析**：

需求：可以使用RestTemplate访问http://user-service/user/8获取服务数据<font color=red>（不需要拼字符串来获取了）</font>

可以使用Ribbon负载均衡：在执行RestTemplate发送服务地址请求的时候，使用负载均衡拦截器拦截，根据服务名获取服务地址列表，使用Ribbon负载均衡算法从服务地址列表中选择一个服务地址，访问该地址获取服务数据。

实现步骤：

1. 启动多个user-service实例（9091,9092）；
2. 修改RestTemplate实例化方法，添加负载均衡注解；
3. 修改ConsumerController；
4. 测试

**小结**：

- 在<font color=red>实例化RestTemplate</font>的时候使用`@LoadBalanced`，服务地址直接可以使用服务名，并且按算法来分配

```java
	@Bean
    @LoadBalanced//开启负载均衡
    public RestTemplate restTemplate(){
        return new RestTemplate (  );
    }
```

- 不需要修改配置文件了，因为默认自动配置了`轮询方式`，如果要随机可以通过配置（了解）

## <font color=red>三、熔断器Hystrix</font>

### 1. 简介

- 它在微服务系统中是一款提供保护机制的组件，和eureka一样也是由netflix公司开发

- Hystrix是一个延迟和容错库，用于隔离访问远程服务，防止出现级联失败

### 2. 线程隔离&服务降级

[Hystrix解决雪崩效应](https://www.showdoc.com.cn/439784909434024?page_id=2827350358722234)：

- 线程隔离：用户请求不直接访问服务，而是使用线程池中空闲的线程访问服务，加速失败判断时间
- 服务降级：及时返回服务调用失败的结果，让线程不因为等待服务而阻塞

**小结**：

- 添加依赖

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
```

- 开启熔断

```java
    /*@SpringBootApplication
    @EnableDiscoveryClient //开启Eureka客户端发现功能
    @EnableCircuitBreaker //开启熔断*/
    @SpringCloudApplication //这个注解包含了上面三个注解
    @EnableFeignClients //开启Feign功能
    public class ConsumerApplication {
        public static void main(String[] args) {
            SpringApplication.run(ConsumerApplication.class, args);
        }

        @Bean
        @LoadBalanced
        public RestTemplate restTemplate(){
            return new RestTemplate (  );
        }
    }
```

- 降级逻辑示例<font color=red>（要注意回调方法的返回类型要跟获取请求的方法的返回类型保持一致）</font>

```java
@RestController
@RequestMapping("/consumer")
@Slf4j
@DefaultProperties(defaultFallback = "defaultFallback")
public class ConsumerController {

    @Autowired
    private RestTemplate restTemplate;

    @Autowired
    private DiscoveryClient discoveryClient;

    @GetMapping("/{id}")
    //@HystrixCommand(fallbackMethod = "queryByIdFallback")
    @HystrixCommand
    public String queryById(@PathVariable Long id){
        String url = "http://user-service/user/" + id;
        return restTemplate.getForObject(url, String.class);
    }

    public String queryByIdFallback(Long id){
        log.error("查询用户信息失败。id：{}", id);
        return "对不起，网络太拥挤了！";
    }

    public String defaultFallback(){
        return "默认提示：对不起，网络太拥挤了！";
    }
}
```

- 修改超时配置

```yml
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            #配置Hystrix的超时时长为2秒，默认为1秒
            timeoutInMilliseconds: 1000
```

### 3. 服务熔断演示

- 熔断器工作原理

![](https://img-blog.csdnimg.cn/20200930220131513.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center)

**状态机有3个状态：**

- Closed：关闭状态（断路器关闭），所有请求都正常访问
- Open：打开状态（断路器打开），所有请求都会被降级。Hystrix会对请求情况计数，当一定时间内失败请求百分比达到阈值，则触发熔断，断路器会完全打开。默认失败比例的阈值是50%，请求次数最少不低于20次
- Half Open：半开状态，不是永久的，断路器打开后会进入休眠时间（默认是5S）。随后断路器会自动进入半开状态，此时会释放部分请求通过，若这些请求都是健康的，则会关闭断路器，否则继续保持打开，再次进行休眠计时

**可以通过配置服务熔断参数修改默认：**<font color=red>（在开启了hystrix方配置）</font>

```yml
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            #配置Hystrix的超时时长为2秒，默认为1秒
            timeoutInMilliseconds: 1000
      circuitBreaker:
        errorThresholdPercentage: 50 # 触发熔断错误比例阈值，默认值50%
        sleepWindowInMilliseconds: 10000 # 熔断后休眠时长，默认值5秒
        requestVolumeThreshold: 10 # 熔断触发最小请求次数，默认值是20
```

## <font color=red>四、Feign应用</font>

### 1. 简介

- Feign也叫伪装
- Feign可以把Rest的请求进行隐藏，伪装成类似SpringMVC的Controller一样。你不用再自己拼接url，拼接参数等等操作，一切都交给Feign去做
- 我们只要写接口并且使用`@FeignClient`注解声明就行了，实现类自动为我们创建

**目标**：Feign的作用，使用Feign实现consumer-demo代码中调用服务

**分析**：

1. 导入启动器依赖
2. 开启Feign功能`@EnableFeignClients `（在启动类中添加）
3. 编写Feign客户端`@FeignClient`
4. 编写一个处理器ConsumerFeignController，注入Feign客户端并使用
5. 测试

**小结**：

Feign主要作用：自动根据参数拼接http请求地址

- 启动器依赖

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```

- 启动类中开启Feign功能

```java
/*@SpringBootApplication
@EnableDiscoveryClient //开启Eureka客户端发现功能
@EnableCircuitBreaker //开启熔断*/
@SpringCloudApplication //这个注解包含了上面三个注解
@EnableFeignClients //开启Feign功能
public class ConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(){
        return new RestTemplate (  );
    }
}
```
- Feign客户端<font color=red>（自动会我们创建一个实体类）</font>

```java
//声明当前类是一个Feign客户端，指定服务名为user-service
@FeignClient("user-service")
public interface UserClient {

    //http://user-service/user/123
    @GetMapping("/user/{id}")
    User queryById(@PathVariable Long id);
}
```

- 调用feign<font color=red>（feign中已经自动拼接了url，并且使用了负载均衡方式获取服务）</font>

```java
/**
 * Feign的处理器
 */
@RestController
@RequestMapping("/cf")
public class ControllerFeignConsumer {

    @Autowired
    private UserClient userClient;

    @GetMapping("/{id}")
    public User findById(@PathVariable Integer id){
        return userClient.findById ( id );
    }
}
```

访问http://localhost:8080/cf/8跟访问http://localhost:8080/user-service/user/8一样

### 2.Feign负载均衡及熔断

- Feign中已经自动集成了Ribbon负载均衡，因此不需要自己定义RestTemplate进行负载均衡的配置
- Fegin还集成了Hystrix熔断器

**目标**：可以配置Feign内置ribbon配置项和Hystrix熔断的Fallback配置

**分析**：

- 负载均衡
- 服务熔断
- 请求压缩
- 日志级别

都可以通过配置项在Feign中开启使用

**小结**：

在服务消费工程consumer-demo中的配置文件：

```yml
ribbon:
  ConnectTimeout: 1000 # 连接超时时长
  ReadTimeout: 2000 # 数据通信超时时长
  MaxAutoRetries: 0 # 当前服务器的重试次数
  MaxAutoRetriesNextServer: 0 # 重试多少次服务
  OkToRetryOnAllOperations: false # 是否对所有的请求方式都重试
feign:
  hystrix:
    enabled: true # 开启Feign的熔断功能
  compression:
    request:
      enabled: true # 开启请求压缩
      mime-types: text/html,application/xml,application/json # 设置压缩的数据类型
      min-request-size: 2048 # 设置触发压缩的大小下限
    response:
      enabled: true
logging:
  level:
    com.itheima: debug
```

## <font color=red>五、服务网关Gateway</font>

### 1.简介

- 核心就是一系列的过滤器，可以将客户端的请求转发到不同的微服务

- 本身也是一个微服务，需要注册到Eureka服务注册中心

- 主要作用：**过滤、断言、路由**

![](https://img-blog.csdnimg.cn/20201001004059122.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center)

### 2.入门使用

**目标**：搭建网关服务工程测试网关服务作用

**分析**：

需求：通过网关系统heima-gateway将包含有 /user 的请求 路由到 http://127.0.0.1:9091/user/用户id 

实现步骤：

1. 创建工程；
2. 添加启动器依赖；
3. 编写启动引导类和配置文件；
4. 修改配置文件，设置路由信息；
5. 启动测试

http://127.0.0.1:10010/user/8 --> http://127.0.0.1:9091/user/8

**小结**：

- 启动器依赖

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
    </dependencies>

```

- 配置文件

```yml
server:
  port: 10010
spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      routes:
        # 路由id，可以任意
        - id: user-service-route
          # 代理的服务地址
          #uri: http://127.0.0.1:9091
          uri: lb://user-service #lb是LoadBalance（负载均衡）的简写,负载均衡获取
          # 路由断言： 可以匹配映射路径
          predicates:
            - Path=/user/**

eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka
  instance:
    prefer-ip-address: true
```

### 3. 面向服务的路由

如果将路由服务地址写死明显是不合理的；在Spring Cloud Gateway中可以通过配置动态路由解决。

**小结**：

面向服务的路由；只需要在配置文件中指定路由路径类似： `lb://user-service`

> lb 之后编写的服务名必须要在eureka中注册才能使用

### 4. 路由前缀处理

**目标**：可以对请求到网关服务的地址添加或去除前缀

**分析**：

提供服务的地址：http://127.0.0.1:9091/user/8

- 添加前缀：对请求地址添加前缀路径之后再作为代理的服务地址；

http://127.0.0.1:10010/8 --> http://127.0.0.1:9091/user/8 添加前缀路径/user

- 去除前缀：将请求地址中路径去除一些前缀路径之后再作为代理的服务地址；

http://127.0.0.1:10010/api/user/8 --> http://127.0.0.1:9091/user/8 去除前缀路径/api

**小结**：

客户端的请求地址与微服务的服务地址如果不一致的时候，可以通过配置路径过滤器实现路径前缀的添加和去除

### 5.过滤器

#### 1.类型和使用场景

- 类型：局部、全局

- 使用场景：请求鉴权、异常处理、记录调用时长等

#### 2.默认过滤器的用法

- 用法：在配置文件中指定要使用的过滤器名称


Gateway自带过滤器有几十个，常见自带过滤器有：

| 过滤器名称           | 说明                         |
| -------------------- | ---------------------------- |
| AddRequestHeader     | 对匹配上的请求加上Header     |
| AddRequestParameters | 对匹配上的请求路由添加参数   |
| AddResponseHeader    | 对从网关返回的响应添加Header |
| StripPrefix          | 对匹配上的请求路径去除前缀   |

更多过滤器和说明：[官网](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.1.1.RELEASE/single/spring-cloud-gateway.html#_gatewayfilter_factories)

#### 3.自定义局部过滤器

**目标**：按照默认过滤器编写并配置一个自定义局部过滤器，该过滤器可以通过配置文件中的参数名称获取请求的参数值

**分析**：

需求：在过滤器（MyParamGatewayFilterFactory）中将http://localhost:10010/api/user/8?name=itcast中的参数name的值获取到并输出到控制台；并且参数名是可变的，也就是不一定每次都是name；需要可以通过配置过滤器的时候做到配置参数名。

实现步骤：

1. 配置过滤器；
2. 编写过滤器；
3. 测试

**小结**：

- 配置；与其他过滤器的配置一致

```yml
spring:
  application:
    name: spring-gateway
  #配置网关
  cloud:
    gateway:
      routes:
        #id可以任意
        - id: user-service-route
        #代理的服务地址
          #uri: http://127.0.0.1:9091
          uri: lb://user-service #lb是LoadBalance（负载均衡）的简写
          
          filters:
            - MyParam=name #自定义的局部过滤器
```

- 实现过滤器

```java
@Component //需要交给spring来管理，否则过滤器无效
public class MyParamGatewayFilterFactory extends AbstractGatewayFilterFactory<MyParamGatewayFilterFactory.NameValueConfig> {

    public MyParamGatewayFilterFactory() {
        super( MyParamGatewayFilterFactory.NameValueConfig.class);
    }

    public static class NameValueConfig{
        //对应在配置过滤器的时候指定的参数名
        private String param;

        public String getParam() {
            return param;
        }

        public void setParam(String param) {
            this.param = param;
        }
    }

    public List<String> shortcutFieldOrder() {
        //param要跟静态内部类的成员变量名一样
        return Arrays.asList("param");
    }

    @Override
    public GatewayFilter apply(NameValueConfig config) {
        return new GatewayFilter ( ) {
            @Override
            public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
                // http://localhost:10010/api/user/8?name=itcast   config.param ==> name
                //获取请求参数中param对应的参数名的值
                ServerHttpRequest request = exchange.getRequest ( );
                if (request.getQueryParams ().containsKey ( config.param )){
                    List<String> strings = request.getQueryParams ( ).get ( config.param );
                    for (String string : strings) {
                        System.out.printf ("------------局部过滤器--------%s = %s------", config.param, string );
                    }
                }
                return chain.filter(exchange);
            }
        };
    }

}
```

#### 4.自定义全局过滤器<font color=red>（不需要在配置文件中配置参数）</font>

**目标**：定义一个全局过滤器检查请求中是否携带有token参数

**分析**：

需求：编写全局过滤器，在过滤器中检查请求地址是否携带token参数。如果token参数的值存在则放行；如果token的参数值为空或者不存在则设置返回的状态码为：未授权也不再执行下去。

实现步骤：

1. 编写全局过滤器
2. 测试

**小结**：

```java
@Component
public class MyGlobalFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        System.out.println("--------------全局过滤器MyGlobalFilter------------------");
        String token = exchange.getRequest().getQueryParams().getFirst("token");
        if(StringUtils.isBlank(token)){
            //设置响应状态码为未授权
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        //值越小越先执行
        return 1;
    }
}
```

### 6.Gateway其它配置说明

**目标**：Gateway网关的负载均衡和熔断参数配置

**小结**：

网关服务配置文件：

```yml
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 6000
ribbon:
  ConnectTimeout: 1000
  ReadTimeout: 2000
  MaxAutoRetries: 0
  MaxAutoRetriesNextServer: 0
```

### 7.Gateway跨域配置<font color=red>（只发生在前端，一般是ajax）</font>

一般网关都是所有微服务的统一入口，必然在被调用的时候会出现跨域问题

**跨域：**在js请求访问中，如果访问的地址与当前服务器的域名、ip或者端口号不一致则称为跨域请求。若不解决不
能获取到对应地址的返回结果

如：从在http://localhost:9090中的js访问 http://localhost:9000的数据，因为端口不同，所以也是跨域请求

```yml
spring:
  application:
    name: spring-gateway
  #配置网关
  cloud:
    gateway:
      #跨域请求处理
      globalcors:
        corsConfigurations:
          '[/**]':
            #allowedOrigins: * # 这种写法或者下面的都可以，*表示全部
            allowedOrigins:
              - "http://docs.spring.io"
            allowedMethods:
              - GET
```

>上述配置表示：可以允许来自 http://docs.spring.io 的get请求方式获取服务数据。
>allowedOrigins 指定允许访问的服务器地址，如：http://localhost:10000 也是可以的。
>'[/**]' 表示对所有访问到网关服务器的请求地址
>官网具体说明：[点击跳转](https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.1.1.RELEASE/multi/multi__cors_configuration.html)

### 8.总配置

```yml
server:
  port: 10010

spring:
  application:
    name: spring-gateway
  #配置网关
  cloud:
    gateway:
      routes:
        #id可以任意
        - id: user-service-route
        #代理的服务地址
          #uri: http://127.0.0.1:9091
          uri: lb://user-service #lb是LoadBalance（负载均衡）的简写
        #路由断言：可以匹配映射地址
          predicates:
            #- Path=/user/**
            #- Path=/**
            - Path=/api/user/**
          filters:
            # 添加前期路径前缀（和Path结合）
            #- PrefixPath=/user #http://localhost:10010/2 -> http://localhost:9091/user/2

            # 去除路径前缀，1表示去一个，2表示去两个，以此类推（和Path结合）
            - StripPrefix=1 #http://localhost:10010/api/user/2 -> http://localhost:9091/user/2
            - MyParam=name #自定义的局部过滤器
      #默认过滤器，会对所有路由都生效
      default-filters:
        - AddResponseHeader=X-Response-Foo,Bar #头名称和值可以自定义
        - AddResponseHeader=MyKey,Hello
      #跨域请求处理
      globalcors:
        corsConfigurations:
          '[/**]':
            #allowedOrigins: * # 这种写法或者下面的都可以，*表示全部
            allowedOrigins:
              - "http://docs.spring.io"
            allowedMethods:
              - GET

eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka
  instance:
    prefer-ip-address: true

hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 6000
ribbon:
  ConnectTimeout: 1000
  ReadTimeout: 2000
  MaxAutoRetries: 0
  MaxAutoRetriesNextServer: 0
```

### 9.Gateway的高可用（了解）

启动多个Gateway服务，自动注册到Eureka，形成集群。如果是服务内部访问，访问Gateway，自动负载均衡，没问题。但是，Gateway更多是外部访问，PC端、移动端等。它们无法通过Eureka进行负载均衡，那么该怎么办？此时，可以使用其它的服务网关，来对Gateway进行代理。比如：`Nginx`

### 10.Gateway与Feign的区别

Gateway网关一般直接给终端请求使用；Feign一般用在微服务之间调用。

- Gateway 作为整个应用的流量入口，接收所有的请求，如PC、移动端等，并且将不同的请求转发至不同的处理微服务模块，其作用可视为nginx；大部分情况下用作权限鉴定、服务端流量控制
- Feign 则是将当前微服务的部分服务接口暴露出来，并且主要用于各个微服务之间的服务调用

## <font color=red>六、分布式配置中心Config</font>

### 1.简介

在分布式系统中，由于服务数量非常多，配置文件分散在不同的微服务项目中，管理不方便。为了方便配置文件集中管理，需要分布式配置中心组件。在Spring Cloud中，提供了Spring Cloud Config，它支持配置文件放在配置服务的本地，也支持放在远程Git仓库（GitHub、码云）

- 使用Spring Cloud Config配置中心后的架构如下图：

![](https://img-blog.csdnimg.cn/20201003003403139.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center)

> spring cloud config作用：可以通过修改在git仓库中的配置文件实现其它所有微服务的配置文件的修改

### 2.搭建配置中心微服务

#### 1.创建配置文件

在新建的仓库中创建需要被统一配置管理的配置文件

**配置文件的命名方式：**`{application}-{profile}.yml `或 `{application}-{profile}.properties`

- application为应用名称
- profile用于区分开发环境，测试环境、生产环境等
- 如user-dev.yml，表示用户微服务开发环境下使用的配置文件

#### 2.添加配置中心依赖

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
    </dependencies>
```

#### 3. 给启动类添加`@EnableConfigServer`

```java
@SpringBootApplication
@EnableConfigServer //开启配置服务
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

#### 4.配置文件

 http://localhost:12000/user-dev.yml 可以获取到

```yml
server:
  port: 12000
spring:
  application:
    name: config-server
  cloud:
    config:
      server:
      	#配置Git远程仓库地址
        git:
          uri: https://gitee.com/keepli/my-config.git
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka
```

#### 5.获取配置中心配置

**目标**：改造用户微服务user-service，配置文件信息不再由微服务项目提供，而是从配置中心获取

**分析**：

需求：将服务提供工程user-service的application.yml配置文件删除，修改为从配置中心config-server中获取

实现步骤：

1. 添加启动器依赖
2. 修改配置文件
3. 启动测试

**小结**：

将原来的application.yml删除；然后添加bootstrap.yml配置文件，该文件也是spring boot的默认配置文件，其内容经常配置一些项目中固定的配置项。如果是项目经常变动的应该配置到application.yml中，现在使用了配置中心则应该配置到git仓库中对于的配置文件

- 依赖

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
            <version>2.2.2.RELEASE</version>
        </dependency>
```

- 配置文件`bootstrap.yml`<font color=red>（此文件名也是springboot支持的）</font>

```yml
spring:
  cloud:
    config:
      # 要与仓库中的配置文件的application保持一致
      name: user
      # 要与仓库中的配置文件的profile保持一致
      profile: dev
      # 要与仓库中的配置文件所属的版本（分支）一样
      label: master
      discovery:
        # 使用配置中心
        enabled: true
        # 配置中心服务名
        service-id: config-server

eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka
```

## <font color=red>七、Spring Cloud Bus</font>

### 1.简介

Spring Cloud Bus是用轻量的消息代理将分布式的节点连接起来，可以用于广播配置文件的更改或者服务的监控管
理。也就是消息总线可以为微服务做监控，也可以实现应用程序之间相互通信。 Spring Cloud Bus可选的消息代理有RabbitMQ和Kafka

使用了Bus之后：

![](https://img-blog.csdnimg.cn/20201003123913334.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center)

> Spring Cloud Bus作用：将git仓库的配置文件更新，在不重启系统的情况下实现及时同步到各个微服务

### 2.应用

**目标**：启动RabbitMQ通过修改码云中的配置文件后发送Post请求实现及时更新用户微服务中的配置项

**分析**：

需求：在码云的git仓库中修改user-dev.yml配置文件，实现不重启user-service的情况下可以及时更新配置文件。

实现步骤：

1. 启动RabbitMQ
2. 修改配置中心config-server
3. 修改服务提供工程user-service
4. 测试

**小结**：

- config-server的依赖添加内容

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-bus</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-stream-binder-rabbit</artifactId>
        </dependency>

```

- config-server的配置文件添加内容

```yml
server:
  port: 12000

spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        #配置Git远程仓库地址
        git:
          uri: https://gitee.com/keepli/my-config.git
  #配置RabbitMQ信息
  rabbitmq:
    host: 121.196.161.193
    port: 5672
    username: guest
    password: guest
    virtual-host: /

eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:10086/eureka

management:
  endpoints:
    web:
      exposure:
        #暴露触发消息总线的地址
        include: bus-refresh
```

- UserController的修改

```java
@RestController
@RequestMapping("/user")
@RefreshScope //刷新配置（bus）
public class UserController {

    @Value( "${test.name}" )
    private String name;

    @Autowired
    private UserService userService;

    @GetMapping("/{id}")
    public User findById(@PathVariable Integer id){
        System.out.println ("配置文件test.name为："+name );
        return userService.findById ( id );
    }
}
```

**测试：**

修改了配置内容后使用Postman或者RESTClient工具发送<font color=red>POST方式</font>请求访问地址http://127.0.0.1:12000/actuator/bus-refresh进行刷新

 ## <font color=red>八、Spring Cloud 体系技术综合应用说明</font>

Spring Cloud中的Eureka、GateWay、Config、Bus、Feign等技术的综合应用

![](https://img-blog.csdnimg.cn/20201004152823392.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center)