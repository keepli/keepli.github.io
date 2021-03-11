---
title: 通过docker挂载目录安装nginx和SSL证书的配置
date: 2020-11-20
updated: 2020-11-20
tags: [docker,nginx]
categories: docker
---

## <font color=red>通过docker挂载目录安装nginx和SSL证书的配置</font>

### <font color=#F39C12>一、通过docker挂载目录安装nginx</font>

####  1.利用docker拉取nginx镜像 

```shell
docker pull nginx
```



####  2.创建需要挂载的相应的挂载目录 

```shell
mkdir ~ nginx/conf

mkdir ~ nginx/conf/conf.d

mkdir ~ nginx/html

mkdir ~ nginx/logs
```



####  3.创建nginx.conf配置文件 

- 执行以下命令，将配置文件内容复制进来

```shell
vim /root/nginx/conf/nginx.conf
```

- 配置文件

```properties
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;

}
```



#### 4.创建default.conf配置文件 

- 执行以下命令，将配置文件内容复制进来

```shell
vim /root/nginx/conf/conf.d/default.conf
```

- 配置文件

```properties
server {
    listen       80;
    #listen  [::]:80;
    server_name  localhost;
	
	#将所有HTTP请求通过rewrite指令重定向到HTTPS。
	#rewrite ^(.*)$ https://$host$1; 
	
    #location / {
    #    root   /usr/share/nginx/html;
    #    index  index.html index.htm;
    #}
    
    #将80端口所有请求转发到百度
    location / {
        proxy_pass   https://www.baidu.com;
     }

    #error_page  404              /404.html;

    # 默认服务错误显示页面
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

> 以上配置以反向代理到百度为例

#### 5.创建docker容器

- 先进到nginx目录下

```shell
cd /root/nginx
```

- 创建容器命令

```shell
docker run -id --name nginx -p 80:80 -p 443:443 -v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf -v $PWD/html:/etc/nginx/html -v $PWD/logs:/var/log/nginx -v $PWD/conf/conf.d:/etc/nginx/conf.d  nginx
```

> 说明：以上命令出了绑定80端口还绑定了443端口，443端口是用来使用ssl证书的，没有可以不用开放



### <font color=#F39C12>二、Nginx配置SSL证书</font>

#### 1.下载SSL证书

下载Nginx使用的证书，并把名字统一改为`ssl.pem`和`ssl.key`



#### 2.将证书文件上传到Linux服务器

把`ssl.pem`和`ssl.key`文件上传到`/root/nginx/conf/conf.d`目录下



#### 3.修改default.conf配置文件

```properties
server {
    listen       80;
    #listen  [::]:80;
    server_name  localhost;
	
	#将所有HTTP请求通过rewrite指令重定向到HTTPS。
	rewrite ^(.*)$ https://$host$1; 
	
    #location / {
    #    root   /usr/share/nginx/html;
    #    index  index.html index.htm;
    #}
    
    #将80端口所有请求转发到百度
    location / {
    	#直接访问将80端口的请求重定向到HTTPS协议的域名（也就是下面配置的）
        proxy_pass   https://yourdomain.com;
     }

    #error_page  404              /404.html;

    # 默认服务错误显示页面
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}


#以下属性中，以ssl开头的属性表示与证书配置有关。
server {
    listen 443 ssl;
    #配置HTTPS的默认访问端口为443。
    #如果未在此处配置HTTPS的默认访问端口，可能会造成Nginx无法启动。
    #如果您使用Nginx 1.15.0及以上版本，请使用listen 443 ssl代替listen 443和ssl on。
    server_name yourdomain.com; #需要将yourdomain.com替换成证书绑定的域名。
    root html;
    index index.html index.htm;
    ssl_certificate /etc/nginx/conf.d/ssl.pem;  #需要将ssl.pem替换成已上传的证书文件的名称。
    ssl_certificate_key /etc/nginx/conf.d/ssl.key; #需要将ssl.key替换成已上传的证书密钥文件的名称。
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    #表示使用的加密套件的类型。
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #表示使用的TLS协议的类型。
    ssl_prefer_server_ciphers on;
    location / {
    	# 这里代表的是最终的访问资源（通过80端口访问HTTPS协议的域名来实现所有请求都走HTTPS进行加密）
        proxy_pass   http://yourdomain.com:8090;
    }
}
```



#### 4.重启docker容器

```shell
docker restart nginx
```

