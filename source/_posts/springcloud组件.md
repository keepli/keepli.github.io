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

## <font color=red>五、Spring Cloud Gateway网关简介</font>

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
          uri: lb://user-service #lb是LoadBalance（负载均衡）的简写
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

