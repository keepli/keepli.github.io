---
title: FastDFS分布式文件系统
date: 2020-08-13 
updated: 2020-10-29
tags: FastDFS
categories: 分布式文件系统
---

### <font color=red>一、FastDFS简介</font>

#### <font color=#F39C12>1.FastDFS体系结构</font>

FastDFS是一个开源的轻量级[分布式文件系统](https://baike.baidu.com/item/%E5%88%86%E5%B8%83%E5%BC%8F%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F)，它对文件进行管理，功能包括：文件存储、文件同步、文件访问（文件上传、文件下载）等，解决了大容量存储和负载均衡的问题。特别适合以文件为载体的在线服务，如相册网站、视频网站等等。

<!-- more -->

FastDFS为互联网量身定制，充分考虑了冗余备份、负载均衡、线性扩容等机制，并注重高可用、高性能等指标，使用FastDFS很容易搭建一套高性能的文件服务器集群提供文件上传、下载等服务。



FastDFS  架构包括 Tracker server 和 Storage server。客户端请求 Tracker server 进行文件上传、下载，通过Tracker server 调度最终由 Storage server 完成文件上传和下载。



Tracker server 作用是负载均衡和调度，通过 Tracker server 在文件上传时可以根据一些策略找到Storage server 提供文件上传服务。可以将 tracker 称为追踪服务器或调度服务器。Storage server 作用是文件存储，客户端上传的文件最终存储在 Storage 服务器上，Storageserver 没有实现自己的文件系统而是利用操作系统的文件系统来管理文件。可以将storage称为存储服务器。



**流程架构图：**

![](https://img-blog.csdnimg.cn/20201029235518595.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center)

![](https://img-blog.csdnimg.cn/20201029235527729.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center)

#### <font color=#F39C12>2.上传流程</font>

![](https://img-blog.csdnimg.cn/20201029235534873.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center)

客户端上传文件后存储服务器将文件 ID 返回给客户端，此文件 ID 用于以后访问该文件的索引信息。文件索引信息包括：组名，虚拟磁盘路径，数据两级目录，文件名。

![](https://img-blog.csdnimg.cn/20201029235543233.png#pic_center)

**组名 ：**文件上传后所在的 storage 组名称，在文件上传成功后有storage 服务器返回，需要客户端自行保存。

**虚拟磁盘路径：**storage 配置的虚拟路径，与磁盘选项store_path*对应。如果配置了store_path0 则是 M00，如果配置了 store_path1 则是 M01，以此类推。

**数据两级目录：**storage 服务器在每个虚拟磁盘路径下创建的两级目录，用于存储数据文件。

**文件名：**与文件上传时不同。是由存储服务器根据特定信息生成，文件名包含：源存储服务器 IP 地址、文件创建时间戳、文件大小、随机数和文件拓展名等信息。



### <font color=red>二、FastDFS搭建</font>

#### <font color=#F39C12>1.Docker搭建FastDFS的开发环境</font>

**1.1 拉取镜像**

```shell
docker pull morunchang/fastdfs
```

**1.2 运行tracker**

```shell
docker run -d --name tracker --net=host morunchang/fastdfs sh tracker.sh
```

**1.3 运行 storage**

- 模板

```shell
docker run -d --name storage --net=host -e TRACKER_IP=<your tracker server address>:22122 -e GROUP_NAME=<group name> morunchang/fastdfs sh storage.sh
```

- 例子

```shell
docker run -d --name storage --net=host -e TRACKER_IP=x.x.x.x:22122 -e GROUP_NAME=group1 morunchang/fastdfs sh storage.sh
```

  1.使用的网络模式是–net=host, <your tracker server address> 替换为你机器的Ip即可

  2.<group name> 是组名，即storage的组

  3.如果想要增加新的storage服务器，再次运行该命令，注意更换 新组名

**1.4 修改nginx的配置**

- 进入容器

```shell
docker exec -it storage /bin/bash
```

- 修改nginx.conf

```shell
vi /etc/nginx/conf/nginx.conf
```

- 往配置文件中添加一下内容（新版默认配置已经配置好了，没有就添加，添加之后重启容器）

```shell
        location ~ /M00 {
                    root /data/fast_data/data;
                    ngx_fastdfs_module;
        }
```

- 不需要图片缓存到本地

```shell
        location ~ /M00 {
        			add_header Cache-Control no-store;#不缓存图片到本地
                    root /data/fast_data/data;
                    ngx_fastdfs_module;
        }
```

### <font color=#F39C12>2.Tracker和Storage配置文件</font>

默认配置就行，这里主要是知道怎么修改  [参考链接](https://www.itread01.com/content/1544541603.html)

**2.1 Tracker配置文件**

- 进入容器

```shell
docker exec -it tracker /bin/bash
```

-  修改文件tracker.conf（参考）

```shell
vi /etc/fdfs/tracker.conf
```

```shell
# is this config file disabled
# false for enabled
# true for disabled
disabled=false

# bind an address of this host
# empty for bind all addresses of this host
bind_addr=

# the tracker server port
port=22122

# connect timeout in seconds
# default value is 30s
connect_timeout=30

# network timeout in seconds
# default value is 30s
network_timeout=60

# the base path to store data and log files
base_path=/data/fast_data

# max concurrent connections this server supported
max_connections=256

# accept thread count
# default value is 1
# since V4.07
accept_threads=1

# work thread count, should <= max_connections
# default value is 4
# since V2.00
work_threads=4

# the method of selecting group to upload files
# 0: round robin
# 1: specify group
# 2: load balance, select the max free space group to upload file
store_lookup=2

# which group to upload file
# when store_lookup set to 1, must set store_group to the group name
store_group=group2

# which storage server to upload file
# 0: round robin (default)
# 1: the first server order by ip address
# 2: the first server order by priority (the minimal)
store_server=0

# which path(means disk or mount point) of the storage server to upload file
# 0: round robin
# 2: load balance, select the max free space path to upload file
store_path=0
```

-  修改文件client.conf 

```shell
vi /etc/fdfs/client.conf
```

**2.2 Storage配置文件**

- 进入容器

```shell
docker exec -it storage /bin/bash
```

-  修改文件storage.conf (参考)

```yml
vi /etc/fdfs/storage.conf
```

```shell
# is this config file disabled
# false for enabled
# true for disabled
disabled=false

# the name of the group this storage server belongs to
#
# comment or remove this item for fetching from tracker server,
# in this case, use_storage_id must set to true in tracker.conf,
# and storage_ids.conf must be configed correctly.
group_name=group1 #组名

# bind an address of this host
# empty for bind all addresses of this host
bind_addr=

# if bind an address of this host when connect to other servers 
# (this storage server as a client)
# true for binding the address configed by above parameter: "bind_addr"
# false for binding any address of this host
client_bind=true

# the storage server port
port=23000 #通信端口

# connect timeout in seconds
# default value is 30s
connect_timeout=30 #连接超时时间，秒

# network timeout in seconds
# default value is 30s
network_timeout=60 #网络请求超时时间

# heart beat interval in seconds
heart_beat_interval=30 #心跳

# disk usage report interval in seconds
stat_report_interval=60 

# the base path to store data and log files
base_path=/data/fast_data #数据存储的基础目录

# max concurrent connections the server supported
# default value is 256
# more max_connections means more memory will be used
max_connections=256 #最大支持并发数

# the buff size to recv / send data
# this parameter must more than 8KB
# default value is 64KB
# since V2.00
buff_size = 256KB

# accept thread count
# default value is 1
# since V4.07
```

-  修改文件client.conf 

```shell
vi /etc/fdfs/client.conf
```

> 如果修改了配置文件注意重启容器



### <font color=red>三、FastDFS微服务搭建</font>

#### <font color=#F39C12>1.环境搭建</font>

**1.1 导入FastDFS依赖**

```xml
    <dependencies>
        <!--FastDFS客户端程序包-->
        <dependency>
            <groupId>net.oschina.zcx7878</groupId>
            <artifactId>fastdfs-client-java</artifactId>
            <version>1.27.0.0</version>
        </dependency>
    </dependencies>
```

**1.2 在resources文件夹下创建fasfDFS的配置文件fdfs_client.conf**

```
#连接超时时间，单位为秒
connect_timeout = 60
#通信超时时间，单位为秒。
#发送或接收数据时。假设在超时时间后还不能发送或接收数据，则本次网络通信失败
network_timeout = 60
#字符集
charset = UTF-8
#Tracker的Http请求端口
http.tracker_http_port = 8080
#Tracker的TCP通信端口
tracker_server = x.x.x.x:22122
```

**1.3 在resources文件夹下创建application.yml**

```yml
server:
  port: 18082
spring:
  application:
    name: file
  servlet:
    multipart:
      #上传文件大小限制
      max‐file‐size: 10MB
      #请求文件大小限制
      max‐request‐size: 10MB
eureka:
  client:
    service‐url:
      defaultZone: http://127.0.0.1:7001/eureka
  instance:
    prefer‐ip‐address: true
feign:
  hystrix:
    enabled: true
```

**1.4 启动springboot**

<font color=red>这里在启动时可能会出现以下异常而导致启动失败</font>（因为有mysql包的存在，springboot自动启动了，但是我们并没有进行配置）

```
Description:

Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured.

Reason: Failed to determine a suitable driver class


Action:

Consider the following:
	If you want an embedded database (H2, HSQL or Derby), please put it on the classpath.
	If you have database settings to be loaded from a particular profile you may need to activate it (no profiles are currently active).
```

解决办法<font color=red>（在`@SpringBootApplication`注解中排除自动加载数据源配置）</font>

```java
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)//排除自动加载数据源配置
@EnableDiscoveryClient//eureka发现服务
public class FileApplication {
    public static void main(String[] args) {
        SpringApplication.run(FileApplication.class, args);
    }
}
```

**1.5 阿里云端口开放**

以下三个端口必须开放

![](https://img-blog.csdnimg.cn/20201029235605848.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center)

### <font color=#F39C12>2.使用FastDFS</font>

**2.1 添加依赖**

```xml
        <!--FastDFS客户端程序包-->
        <dependency>
            <groupId>net.oschina.zcx7878</groupId>
            <artifactId>fastdfs-client-java</artifactId>
            <version>1.27.0.0</version>
        </dependency>
```

**2.2 编写封装文件实体类FastDFSFile**

```java
/**
 * 封装文件实体类
 */
@Data
public class FastDFSFile implements Serializable {

    //文件名字
    private String name;
    //文件内容
    private byte[] content;
    //文件扩展名
    private String ext;
    //文件MD5摘要值
    private String md5;
    //文件创建作者
    private String author;

    // getter and setter ...


    public FastDFSFile(String name, byte[] content, String ext) {
        this.name = name;
        this.content = content;
        this.ext = ext;
    }

    public FastDFSFile(String name, byte[] content, String ext, String md5, String author) {
        this.name = name;
        this.content = content;
        this.ext = ext;
        this.md5 = md5;
        this.author = author;
    }
}
```

**2.3 编写FastDFSUtil工具类**

```java
/**
 * 实现FastDFS文件管理
 * 文件上传
 * 文件删除
 * 文件下载
 * 文件信息获取
 * Storage信息获取
 * Tracker信息获取
 */
public class FastDFSUtil {

    /**
     * 加载Tracker连接信息
     */
    static {
        try {
            //ClassPathResource是spring提供的类，可以获取resources目录下的配置文件信息
            String path = new ClassPathResource ( "fdfs_client.conf" ).getURL ( ).getPath ( );
            //加载Tracker连接信息
            ClientGlobal.init ( path );
        } catch (Exception e) {
            e.printStackTrace ( );
        }
    }

    /**
     * 获取TrackerServer
     *
     * @return
     * @throws Exception
     */
    public static TrackerServer getTrackerServer() throws Exception {
        //创建一个Tracker客户端对象TrackerClient
        TrackerClient trackerClient = new TrackerClient ( );

        //通过TrackerClient获取TrackerServer服务连接信息
        TrackerServer trackerServer = trackerClient.getConnection ( );

        return trackerServer;
    }

    /**
     * 文件上传
     *
     * @param fastDFSFile:上传的文件信息封装
     */
    public static String[] uploadFile(FastDFSFile fastDFSFile) throws Exception {
        //调用封装方法获取TrackerServer
        TrackerServer trackerServer = getTrackerServer ( );

        //通过TrackerServer获取Storage的连接信息，并且创建Storage客户端对象StorageClient存储获取的连接信息
        StorageClient storageClient = new StorageClient ( trackerServer, null );

        //通过StorageClient访问Storage实现文件上传，并且获取文件上传后的存储信息
        /**
         * 参数说明：
         *  1.file_buff是文件数组
         *  2.file_ext_name是文件后缀名 例如：jpg
         *  3.meta_list是文件附加信息，不添加填写null 例如：拍摄地点，时间，文件作者等
         *
         * uploads[]:
         *  uploads[0]是文件上传所存储的Storage的组名
         *  uploads[1]是文件上传到Storage的文件名(包含了目录结构，例如：M00/00/00/rBKx1F-ab_SAOzS3ADkUkK8vhDk359.jpg)
         */
       /* NameValuePair[] meta_list = new NameValuePair[1];
        meta_list[0] = new NameValuePair ( "author",fastDFSFile.getAuthor () );//示例给上传文件添加附加信息*/
        String[] uploads = storageClient.upload_file ( fastDFSFile.getContent ( ), fastDFSFile.getExt ( ), null );

        return uploads;
    }

    /**
     * 文件下载
     * @param groupName:组名
     * @param remoteFileName:文件的存储路径名 例如：M00/00/00/rBKx1F-ab_SAOzS3ADkUkK8vhDk359.jpg
     * @return
     * @throws Exception
     */
    public static InputStream downloadFile(String groupName, String remoteFileName) throws Exception {
        //调用封装方法获取TrackerServer
        TrackerServer trackerServer = getTrackerServer ( );

        //通过TrackerServer获取Storage的连接信息，并且创建Storage客户端对象StorageClient存储获取的连接信息
        StorageClient storageClient = new StorageClient ( trackerServer, null );

        //获取文件字节数组
        byte[] bytes = storageClient.download_file ( groupName, remoteFileName );

        //将文件字节数组转换为InputStream
        ByteArrayInputStream in = new ByteArrayInputStream ( bytes );
        return in;
    }

    /**
     * 删除文件
     * @param groupName:组名
     * @param remoteFileName:文件的存储路径名 例如：M00/00/00/rBKx1F-ab_SAOzS3ADkUkK8vhDk359.jpg
     * @throws Exception
     */
    public static void deleteFile(String groupName, String remoteFileName) throws Exception {
        //调用封装方法获取TrackerServer
        TrackerServer trackerServer = getTrackerServer ( );

        //通过TrackerServer获取Storage的连接信息，并且创建Storage客户端对象StorageClient存储获取的连接信息
        StorageClient storageClient = new StorageClient ( trackerServer, null );

        //删除文件
        storageClient.delete_file ( groupName, remoteFileName );
    }

    /**
     * 获取文件信息
     * @param groupName:组名
     * @param remoteFileName:文件的存储路径名 例如：M00/00/00/rBKx1F-ab_SAOzS3ADkUkK8vhDk359.jpg
     * @return
     * @throws Exception
     */
    public static FileInfo getFile(String groupName, String remoteFileName) throws Exception {
        //调用封装方法获取TrackerServer
        TrackerServer trackerServer = getTrackerServer ( );

        //通过TrackerServer获取Storage的连接信息，并且创建Storage客户端对象StorageClient存储获取的连接信息
        StorageClient storageClient = new StorageClient ( trackerServer, null );

        //获取文件信息
        /**
         * fileInfo可以获取的信息：
         *  source_ip_addr是访问的ip地址
         *  file_size是文件的大小
         *  create_timestamp是文件的创建时间
         *  crc32是文件签名信息
         */
        FileInfo fileInfo = storageClient.get_file_info ( groupName, remoteFileName );
        return fileInfo;
    }

    /**
     * 获取指定组的StorageServer信息
     * @param groupName:组名
     * @return
     * @throws Exception
     */
    public static StorageServer getStorageServer(String groupName) throws Exception {
        //创建一个Tracker客户端对象TrackerClient
        TrackerClient trackerClient = new TrackerClient ( );

        //通过TrackerClient获取TrackerServer服务连接信息
        TrackerServer trackerServer = trackerClient.getConnection ( );

        //获取指定组的StorageServer信息
        StorageServer storageServer = trackerClient.getStoreStorage ( trackerServer,groupName);
        //StorageServer[] storageServers = trackerClient.getStoreStorages ( trackerServer,groupName);//获取当前组的所有Storage服务器
        return storageServer;
    }

    /**
     * 获取StorageServer的ip和端口
     * 指定组下的所有StorageServer信息
     * @param groupName:组名
     * @param remoteFileName:文件的存储路径名 例如：M00/00/00/rBKx1F-ab_SAOzS3ADkUkK8vhDk359.jpg
     * @return
     * @throws Exception
     */
    public static ServerInfo[] getStorageServerInfo(String groupName, String remoteFileName) throws Exception {
        //创建一个Tracker客户端对象TrackerClient
        TrackerClient trackerClient = new TrackerClient ( );

        //通过TrackerClient获取TrackerServer服务连接信息
        TrackerServer trackerServer = trackerClient.getConnection ( );

        //获取指定组的StorageServerInfo信息
        ServerInfo[] storageServers = trackerClient.getFetchStorages ( trackerServer, groupName, remoteFileName );
        return storageServers;
    }

    /**
     * 获取TrackerServer的URL
     * TrackerServer的端口和Nginx的端口最好保持一致
     * @param groupName
     * @param remoteFileName
     * @return
     * @throws Exception
     */
    public static String getTrackerServerUrl(String groupName, String remoteFileName) throws Exception {
        //调用封装方法获取TrackerServer
        TrackerServer trackerServer = getTrackerServer ( );

        //获取TrackerServer的IP和请求端口
        int port = ClientGlobal.getG_tracker_http_port ( );
        String ip = trackerServer.getInetSocketAddress ( ).getHostName ( );
        String url = "http://"+ip+":"+port;
        return url;
    }

    public static void main(String[] args) throws Exception {
        //获取文件信息
        /*FileInfo fileInfo = getFile ( "group1", "M00/00/00/rBKx1F-arI2AHjruAAOyk4wRlUU283.jpg" );
        System.out.println ( fileInfo.getSourceIpAddr ( ) );//内网ip
        System.out.println ( fileInfo.getFileSize ( ) );
        System.out.println ( fileInfo.getCreateTimestamp ( ) );
        System.out.println ( fileInfo.getCrc32 ( ) );*/

        //文件下载
       /* long startTime = System.currentTimeMillis ( );
        BufferedInputStream buffer_in = new BufferedInputStream ( downloadFile ( "group1", "M00/00/00/rBKx1F-arI2AHjruAAOyk4wRlUU283.jpg"  ) );
        BufferedOutputStream buffer_out = new BufferedOutputStream ( new FileOutputStream ( "D:\\a.jpg" ) );

        byte[] bytes = new byte[1024];


        int len = 0;
        while ((len = buffer_in.read (bytes))!=-1){
            buffer_out.write ( bytes );
        }

        buffer_out.close ();
        buffer_in.close ();

        long endTime = System.currentTimeMillis ( );
        System.out.println ((endTime-startTime)+"ms" );*/

        //文件删除
        //deleteFile ( "group1", "M00/00/00/rBKx1F-ams-ABG71ADkUkK8vhDk958.jpg" );

        //获取指定组的StorageServer信息
       /* StorageServer storageServer = getStorageServer ( "group1" );
        System.out.println (storageServer.getInetSocketAddress () );//公网ip加端口号
        System.out.println (storageServer.getInetSocketAddress ().getHostName () );//公网ip
        //System.out.println (storageServer.getInetSocketAddress ().getHostString () );//公网ip
        System.out.println (storageServer.getSocket ());*/

        //获取StorageServer的ip和端口
        /*ServerInfo[] storageServerInfos = getStorageServerInfo ( "group1", "M00/00/00/rBKx1F-arI2AHjruAAOyk4wRlUU283.jpg" );
        for (ServerInfo storageServerInfo : storageServerInfos) {
            System.out.println (storageServerInfo.getIpAddr () );
            System.out.println (storageServerInfo.getPort () );
        }*/

        //获取TrackerServer的URL
        System.out.println ( getTrackerServerUrl ( "group1", "M00/00/00/rBKx1F-arI2AHjruAAOyk4wRlUU283.jpg"  ) );
    }
}
```

**2.4 controller控制层**

只举例文件上传

```java
@RestController
@RequestMapping("/file")
@CrossOrigin//开启跨域
public class FileController {

    /**
     * 文件上传
     * @param file
     * @return
     * @throws Exception
     */
    @PostMapping("/upload")
    public Result<String> upload(@RequestParam("file") MultipartFile file) throws Exception {
        //封装文件信息
        FastDFSFile fastDFSFile = new FastDFSFile (
                file.getOriginalFilename ( ),//获取文件名称
                file.getBytes ( ),//获取文件字节数组
                StringUtils.getFilenameExtension ( file.getOriginalFilename () )//获取文件后缀名
        );

        //调用FastDFSUtil工具类将文件传入到FastDFS
        String[] uploads = FastDFSUtil.uploadFile ( fastDFSFile );

        //拼接访问地址url http://x.x.x.x:8080/group1/M00/00/00/rBKx1F-ab_SAOzS3ADkUkK8vhDk359.jpg
        String url = FastDFSUtil.getTrackerServerUrl ( uploads[0],uploads[1])+"/"+uploads[0]+"/"+uploads[1];

        return new Result<> ( true, StatusCode.OK, "文件上传成功",url );
    }

}
```

### <font color=#F39C12>3.Storage的存储目录结构</font>

- 进入storage容器

```shell
docker exec -it storage /bin/bash
```

- 进入到图中标红线的目录就可以查看存储目录结构了

![](https://img-blog.csdnimg.cn/20201029235622615.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center)