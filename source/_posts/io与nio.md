---
title: 传统I/O与NIO
date: 2020-07-15 
updated: 2020-07-15
tags: netty
categories: 网络编程
---

 *传统 I / O 与 NIO 的区别，netty框架的简单了解*

<!-- more -->

---

## <font color=red>一、传统IO编程</font>

### <font color=#F39C12>1. 阻塞式I/O的通信模型示意图 </font>

 每个客户端连接过来后，服务端都会启动一个线程去处理该客户端的请求 

<img src="https://img-blog.csdnimg.cn/20201021163808543.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center" style="zoom:80%;" />

###  <font color=#F39C12>2. 业务场景 </font>

客户端每隔两秒发送字符串给服务端，服务端收到之后打印到控制台 

### <font color=#F39C12>3. 代码实现 </font>

 **服务端实现 ：**

```java
public class IOServer {
    public static void main(String[] args) throws Exception {
        //创建服务端socket对象
        ServerSocket serverSocket = new ServerSocket ( 8000 );

        while (true){
            //接收来自客户端的socket请求
            Socket socket = serverSocket.accept ( );

            //使用匿名内部类和匿名对象的方式创建线程
            new Thread ( ){
                @Override
                public void run() {

                    BufferedInputStream b_in = null;

                    try {
                        String name = Thread.currentThread ( ).getName ( );
                        //获取网络输入流
                        b_in = new BufferedInputStream ( socket.getInputStream ( ) );

                        byte[] bytes = new byte[1024];
                        int len = 0;

                        while ((len = b_in.read ( bytes )) != -1){
                            //打印读取到的数据
                            System.out.println ( "线程" + name + "：" + new String ( bytes,0,bytes.length ) );
                        }

                    } catch (Exception e) {
                        e.printStackTrace ( );

                    }finally {
                        //关闭资源
                        if (b_in!=null){
                            try {
                                b_in.close ();
                            } catch (IOException e) {
                                e.printStackTrace ( );
                            }
                        }
                    }
                }
            }.start ();
        }

    }
}
```

**客户端实现：**

```java
public class IOClient {
    public static void main(String[] args) throws Exception {

        for (int i = 0; i < 5; i++) {
            new ClientDemo ().start ();
        }

    }

    static class ClientDemo extends Thread {
        @Override
        public void run() {
            try {
                //创建客户端socket对象
                Socket socket = new Socket ( "127.0.0.1", 8000 );

                //获取网络字节输出流
                OutputStream out = socket.getOutputStream ( );

                while (true){
                    out.write ( "你好，服务端".getBytes ( ) );
                    Thread.sleep ( 2000 );
                }
            } catch (Exception e) {
                e.printStackTrace ( );
            }
        }
    }
}
```

### <font color=#F39C12>4.存在的问题 </font>

从服务端代码中我们可以看到，在传统的IO模型中，每个连接创建成功之后都需要一个线程来维护，每个线程包含一个while死循环 。

如果在用户数量较少的情况下运行是没有问题的，但是对于用户数量比较多的业务来说，服务端可能需要支撑成千上万的连接，IO模型可能就不太合适了 。



**如果有1万个连接就对应1万个线程，继而1万个while死循环**，这种模型存在以下问题： 

- 当客户端越多，就会创建越多的处理线程。线程是操作系统中非常宝贵的资源，同一时刻有大量的线程处于阻塞状态是非常严重的资源浪费。并且如果务器遭遇洪峰流量冲击，例如双十一活动，线程池会瞬间被耗尽，导致服务器瘫痪。
- 因为是阻塞式通信，线程爆炸之后操作系统频繁进行线程切换，应用性能急剧下降。
- IO编程中数据读写是以字节流为单位，效率不高。



### <font color=red>二、NIO编程</font>

### <font color=#F39C12>1. 非阻塞式I/O的通信模型示意图 </font>

NIO，也叫做new-IO或者non-blocking-IO，可理解为非阻塞IO。NIO编程模型中，新来一个连接不再创建一个新的线程，而是可以把这条连接直接绑定到某个固定的线程，然后这条连接所有的读写都由这个线程来负责，我们用一幅图来对比一下IO与NIO 

<img src="https://img-blog.csdnimg.cn/20201021163822155.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center" style="zoom: 50%;" />

如上图所示，IO模型中，一个连接都会创建一个线程，对应一个while死循环，死循环的目的就是不断监测这条连接上是否有数据可以读。但是在大多数情况下，1万个连接里面同一时刻只有少量的连接有数据可读，因此，很多个while死循环都白白浪费掉了，因为没有数据。

而在NIO模型中，可以把这么多的while死循环变成一个死循环，这个死循环由一个线程控制。这就是NIO模型中选择器（Selector）的作用，一条连接来了之后，现在不创建一个while死循环去监听是否有数据可读了，而是直接把这条连接注册到**选择器**上，通过检查这个**选择器**，就可以批量监测出有数据可读的连接，进而读取数据。

### <font color=#F39C12>2. 举例说明传统I/O和NIO的区别 </font>

举个栗子，在一家餐厅里，客人有点菜的需求，一共有100桌客人，有两种方案可以解决客人点菜的问题：

- 方案一：

  每桌客人配一个服务生，每个服务生就在餐桌旁给客人提供服务。如果客人要点菜，服务生就可以立刻提供点菜的服务。那么100桌客人就需要100个服务生提供服务，这就是IO模型，一个连接对应一个线程。

   

- 方案二：

  一个餐厅只有一个服务生（假设服务生可以忙的过来）。这个服务生隔段时间就询问所有的客人是否需要点菜，然后每一时刻处理所有客人的点菜要求。这就是NIO模型，所有客人都注册到同一个服务生，对应的就是所有的连接都注册到一个线程，然后批量轮询。

这就是NIO模型解决线程资源受限的方案，实际开发过程中，我们会开多个线程，每个线程都管理着一批连接，相对于IO模型中一个线程管理一条连接，消耗的线程资源大幅减少。

### <font color=#F39C12>3. NIO的三大核心组件 </font>

**NIO的三大核心组件：通道（Channel）、缓冲（Buffer）、选择器（Selector）**

- 通道（Channel）

  是传统IO中的Stream(流)的升级版。Stream是单向的、读写分离（inputstream和outputstream），Channel是双向的，既可以进行读操作，又可以进行写操作

- 缓冲（Buffer）

  Buffer可以理解为一块内存区域，可以写入数据，并且在之后读取它

- 选择器（Selector）

  选择器（Selector）可以实现一个单独的线程来监控多个注册在她上面的信道（Channel），通过一定的选择机制，实现多路复用的效果

### <font color=#F39C12>4. NIO相对于IO的优势 </font>

- IO是面向流的，每次都是从操作系统底层一个字节一个字节地读取数据，并且数据只能从一端读取到另一端，不能前后移动流中的数据。NIO则是面向缓冲区的，每次可以从这个缓冲区里面读取一块的数据，并且可以在需要时在缓冲区中前后移动
- IO是阻塞的，这意味着，当一个线程读取数据或写数据时，该线程被阻塞，直到有一些数据被读取，或数据完全写入，在此期间该线程不能干其他任何事情。而NIO是非阻塞的，不需要一直等待操作完成才能干其他事情，而是在等待的过程中可以同时去做别的事情，所以能最大限度地使用服务器的资源
- NIO引入了IO多路复用器selector。selector是一个提供channel注册服务的线程，可以同时对接多个Channel，并在线程池中为channel适配、选择合适的线程来处理channel。由于NIO模型中线程数量大大降低，线程切换效率因此也大幅度提高

### <font color=#F39C12>5. 代码实现</font>

 和前面一样的场景，使用NIO实现（复制代码演示效果即可，客户端照样用原来的）： 

```java
public class NIOServer {
    public static void main(String[] args) throws IOException {
        // 负责轮询是否有新的连接
        Selector serverSelector = Selector.open();
        // 负责轮询处理连接中的数据
        Selector clientSelector = Selector.open();

        new Thread() {
            @Override
            public void run() {
                try {
                    // 对应IO编程中服务端启动
                    ServerSocketChannel listenerChannel = ServerSocketChannel.open();
                    listenerChannel.socket().bind(new InetSocketAddress(8000));
                    listenerChannel.configureBlocking(false);
                    // OP_ACCEPT表示服务器监听到了客户连接，服务器可以接收这个连接了
                    listenerChannel.register(serverSelector, SelectionKey.OP_ACCEPT);

                    while (true) {
                        // 监测是否有新的连接，这里的1指的是阻塞的时间为1ms
                        if (serverSelector.select(1) > 0) {
                            Set<SelectionKey> set = serverSelector.selectedKeys();
                            Iterator<SelectionKey> keyIterator = set.iterator();

                            while (keyIterator.hasNext()) {
                                SelectionKey key = keyIterator.next();

                                if (key.isAcceptable()) {
                                    try {
                                        // (1) 每来一个新连接，不需要创建一个线程，而是直接注册到clientSelector
                                        SocketChannel clientChannel = ((ServerSocketChannel) key.channel()).accept();
                                        clientChannel.configureBlocking(false);
                                        // OP_READ表示通道中已经有了可读的数据，可以执行读操作了（通道目前有数据，可以进行读操作了）
                                        clientChannel.register(clientSelector, SelectionKey.OP_READ);
                                    } finally {
                                        keyIterator.remove();
                                    }
                                }
                            }
                        }
                    }
                } catch (IOException ignored) {
                }
            }
        }.start();


        new Thread() {
            @Override
            public void run() {
                String name = Thread.currentThread().getName();
                try {
                    while (true) {
                        // (2) 批量轮询是否有哪些连接有数据可读，这里的1指的是阻塞的时间为1ms
                        if (clientSelector.select(1) > 0) {
                            Set<SelectionKey> set = clientSelector.selectedKeys();
                            Iterator<SelectionKey> keyIterator = set.iterator();

                            while (keyIterator.hasNext()) {
                                SelectionKey key = keyIterator.next();

                                if (key.isReadable()) {
                                    try {
                                        SocketChannel clientChannel = (SocketChannel) key.channel();
                                        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
                                        // (3) 读取数据以块为单位批量读取
                                        clientChannel.read(byteBuffer);
                                        byteBuffer.flip();
                                        System.out.println("线程" + name + ":" + Charset.defaultCharset().newDecoder().decode(byteBuffer)
                                                .toString());
                                    } finally {
                                        keyIterator.remove();
                                        key.interestOps(SelectionKey.OP_READ);
                                    }
                                }
                            }
                        }
                    }
                } catch (IOException ignored) {
                }
            }
        }.start();
    }
}
```

## <font color=red>三、Netty</font>

### <font color=#F39C12>1. 为什么使用Netty </font>

我们已经有了NIO能够提高程序效率了，为什么还要使用Netty？

简单的说：Netty封装了JDK的NIO，让你用得更爽，你不用再写一大堆复杂的代码了。

官方术语：Netty是一个异步事件驱动的网络应用框架，用于快速开发可维护的高性能服务器和客户端。

 

**下面是使用Netty不使用JDK原生NIO的一些原因：**

- 使用JDK自带的NIO需要了解太多的概念，编程复杂
- Netty底层IO模型随意切换，而这一切只需要做微小的改动，就可以直接从NIO模型变身为IO模型
- Netty自带的拆包解包，异常检测等机制，可以从NIO的繁重细节中脱离出来，只需要关心业务逻辑
- Netty解决了JDK的很多包括空轮询在内的bug
- Netty底层对线程，selector做了很多细小的优化，精心设计的线程模型做到非常高效的并发处理
- 自带各种协议栈让你处理任何一种通用协议都几乎不用亲自动手
- Netty社区活跃，遇到问题随时邮件列表或者issue
- Netty已经历各大rpc框架，消息中间件，分布式通信中间件线上的广泛验证，健壮性无比强大



### <font color=#F39C12>2. 代码实现 </font>

和IO编程一样的案例：

**添加Netty依赖**

```java
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.5.Final</version>
</dependency>
```

 

**服务端：**

```java
public class NettyServer {
    public static void main(String[] args) {
        ServerBootstrap serverBootstrap = new ServerBootstrap();

        NioEventLoopGroup boos = new NioEventLoopGroup();
        NioEventLoopGroup worker = new NioEventLoopGroup();
        serverBootstrap
                .group(boos, worker)
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<NioSocketChannel>() {
                    protected void initChannel(NioSocketChannel ch) {
                        ch.pipeline().addLast(new StringDecoder());
                        ch.pipeline().addLast(new SimpleChannelInboundHandler<String>() {
                            @Override
                            protected void channelRead0(ChannelHandlerContext ctx, String msg) {
                                System.out.println(msg);
                            }
                        });
                    }
                })
                .bind(8000);
    }
}
```

 

**客户端：**

```java
public class NettyClient {
    public static void main(String[] args) throws InterruptedException {
        Bootstrap bootstrap = new Bootstrap();
        NioEventLoopGroup group = new NioEventLoopGroup();

        bootstrap.group(group)
                .channel(NioSocketChannel.class)
                .handler(new ChannelInitializer<Channel>() {
                    @Override
                    protected void initChannel(Channel ch) {
                        ch.pipeline().addLast(new StringEncoder());
                    }
                });

        Channel channel = bootstrap.connect("127.0.0.1", 8000).channel();

        while (true) {
            channel.writeAndFlush("测试数据");
            Thread.sleep(2000);
        }
    }
}
```

### <font color=#F39C12>3. Netty的事件驱动 </font>

例如很多系统都会提供 onClick() 事件，这个事件就代表鼠标按下事件。事件驱动模型的大体思路如下：

1. 有一个事件队列
2. 鼠标按下时，往事件队列中增加一个点击事件
3. 有个事件泵，不断循环从队列取出事件，根据不同的事件，调用不同的函数
4. 事件一般都各自保存各自的处理方法的引用。这样，每个事件都能找到对应的处理方法

<img src="https://img-blog.csdnimg.cn/20201021163832267.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center" style="zoom:80%;" />

**为什么使用事件驱动？**

- 程序中的任务可以并行执行
- 任务之间高度独立，彼此之间不需要互相等待
- 在等待的事件到来之前，任务不会阻塞



**Netty使用事件驱动的方式作为底层架构**，包括：

- 事件队列（event queue）：接收事件的入口
- 分发器（event mediator）：将不同的事件分发到不同的业务逻辑单元
- 事件通道（event channel）：分发器与处理器之间的联系渠道
- 事件处理器（event processor）：实现业务逻辑，处理完成后会发出事件，触发下一步操作

<img src="https://img-blog.csdnimg.cn/20201021163841964.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center" style="zoom:80%;" />

### <font color=#F39C12>4. 核心组件</font>

 **Netty 的功能特性图：** 

<img src="https://img-blog.csdnimg.cn/20201021163851511.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center" style="zoom:80%;" />

**Netty 功能特性：**

- 传输服务，支持 BIO 和 NIO。
- 容器集成：支持 OSGI、JBossMC、Spring、Guice 容器。
- 协议支持：HTTP、Protobuf、二进制、文本、WebSocket 等，支持自定义协议。

 

 **BIO和NIO的区别：** 

| **场景**       | BIO                | NIO                                      |
| -------------- | ------------------ | ---------------------------------------- |
| 有新连接请求时 | 开一个新的线程处理 | 使用多路复用原理，一个线程处理           |
| 适用场景       | 连接数小且固定     | 连接数特别多，连接比较短（轻操作）的场景 |



**Netty框架包含如下的组件：**

- ServerBootstrap ：用于接受客户端的连接以及为已接受的连接创建子通道，一般用于服务端。
- Bootstrap：不接受新的连接，并且是在父通道类完成一些操作，一般用于客户端的。

 

- Channel：对网络套接字的I/O操作，例如读、写、连接、绑定等操作进行适配和封装的组件。（双向通道，可以读又可以写）
- EventLoop：处理所有注册其上的channel的I/O操作。通常情况一个EventLoop可为多个channel提供服务。
- EventLoopGroup：包含有多个EventLoop的实例，用来管理 event Loop 的组件，可以理解为一个线程池，内部维护了一组线程。（一般会创建两个对象，一个用来接收，一个用来处理，如果只有一个压力会比较大）

 

- ChannelHandler和ChannelPipeline：例如一个流水线车间，当组件从流水线头部进入，穿越流水线，流水线上的工人按顺序对组件进行加工，到达流水线尾部时商品组装完成。流水线相当于`ChannelPipeline`，流水线工人相当于`ChannelHandler`，源头的组件当做event。
- ChannelInitializer：用于对刚创建的channel进行初始化，将ChannelHandler添加到channel的ChannelPipeline处理链路中。

 

- ChannelFuture：与jdk中线程的Future接口类似，即实现并行处理的效果。可以在操作执行成功或失败时自动触发监听器中的事件处理方法。

 

**服务端：**

```java
public class NettyServer {
    public static void main(String[] args) {
        // 用于接受客户端的连接以及为已接受的连接创建子通道，一般用于服务端。
        ServerBootstrap serverBootstrap = new ServerBootstrap();

        // EventLoopGroup包含有多个EventLoop的实例，用来管理event Loop的组件
        // 接受新连接线程
        NioEventLoopGroup boos = new NioEventLoopGroup();
        // 读取数据的线程
        NioEventLoopGroup worker = new NioEventLoopGroup();

        //服务端执行
        serverBootstrap
                .group(boos, worker)
                // Channel对网络套接字的I/O操作，
                // 例如读、写、连接、绑定等操作进行适配和封装的组件。
                .channel(NioServerSocketChannel.class)
                // ChannelInitializer用于对刚创建的channel进行初始化
                // 将ChannelHandler添加到channel的ChannelPipeline处理链路中。
                .childHandler(new ChannelInitializer<NioSocketChannel>() {
                    protected void initChannel(NioSocketChannel ch) {
                        // 组件从流水线头部进入，流水线上的工人按顺序对组件进行加工
                        // 流水线相当于ChannelPipeline
                        // 流水线工人相当于ChannelHandler
                        ch.pipeline().addLast(new StringDecoder());
                        ch.pipeline().addLast(new SimpleChannelInboundHandler<String>() {
                            //这个工人有点麻烦，需要我们告诉他干啥事
                            @Override
                            protected void channelRead0(ChannelHandlerContext ctx, String msg) {
                                System.out.println(msg);
                            }
                        });
                    }
                })
                .bind(8000);
    }
}
```

**客户端：**

```java
public class NettyClient {
    public static void main(String[] args) throws InterruptedException {
        // 不接受新的连接，并且是在父通道类完成一些操作，一般用于客户端的。
        Bootstrap bootstrap = new Bootstrap();

        // EventLoopGroup包含有多个EventLoop的实例，用来管理event Loop的组件
        NioEventLoopGroup group = new NioEventLoopGroup();

        //客户端执行
        bootstrap.group(group)
                // Channel对网络套接字的I/O操作，
                // 例如读、写、连接、绑定等操作进行适配和封装的组件。
                .channel(NioSocketChannel.class)
                // 用于对刚创建的channel进行初始化，
                // 将ChannelHandler添加到channel的ChannelPipeline处理链路中。
                .handler(new ChannelInitializer<Channel>() {
                    @Override
                    protected void initChannel(Channel ch) {
                        // 组件从流水线头部进入，流水线上的工人按顺序对组件进行加工
                        // 流水线相当于ChannelPipeline
                        // 流水线工人相当于ChannelHandler
                        ch.pipeline().addLast(new StringEncoder());
                    }
                });

        //客户端连接服务端
        Channel channel = bootstrap.connect("127.0.0.1", 8000).channel();

        while (true) {
            // 客户端使用writeAndFlush方法向服务端发送数据，返回的是ChannelFuture
            // 与jdk中线程的Future接口类似，即实现并行处理的效果
            // 可以在操作执行成功或失败时自动触发监听器中的事件处理方法。
            ChannelFuture future = channel.writeAndFlush("测试数据");
            Thread.sleep(2000);
        }
    }
}
```

 