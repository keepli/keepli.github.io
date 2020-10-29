---
title: 环信IM的集成
date: 2020-06-10
updated: 2020-06-10
tags: [即时通讯,环信IM]
categories: 即时通讯
---

*第三方即时通讯服务，环信 IM云的集成使用，我这里主要以 web集成为主，如何把单聊模块集成到自己的项目进行的操作讲解，更多详细操作可以参考官网提供的开发文档*

<!-- more -->

---

## <font color=red>环信IM</font>

### <font color=#F39C12>一、准备工作</font>

#### 1.创建IM应用

[环信官网](https://www.easemob.com/)

- 注册账户，选择**个人开发者**（可以免费使用100个用户）

- 创建IM应用，注册模式选择**开放注册**

  <img src="https://img-blog.csdnimg.cn/20201014141523503.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center" style="zoom: 80%;" />

-  进入创建的IM应用，可以看到`appkey`，`orgname`，`client id`，`client secret`等字段 ，**后面需要使用**
 <img src="https://img-blog.csdnimg.cn/20201014142119171.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center" style="zoom: 80%;" />

#### 2.接口测试

- 打开[REST API Doc]()， 这是环信服务器端集成`Swagger`文档，用于**测试各个API接口**

  <img src="https://img-blog.csdnimg.cn/20201014142744915.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center" style="zoom: 80%;" />

- 这里以**获取token为例**

  <img src="https://img-blog.csdnimg.cn/20201014143411528.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center" style="zoom:80%;" />

  <img src="https://img-blog.csdnimg.cn/20201014144009239.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center" style="zoom:80%;" />  

#### 2.参照开发文档

[开发文档地址](http://docs-im.easemob.com/)

**我是web开发，选择的是web客户端集成**：

[web客户端集成文档直通车链接](http://docs-im.easemob.com/im/web/intro/start)

<img src="https://img-blog.csdnimg.cn/20201014144413681.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center" style="zoom:80%;" />

### <font color=#F39C12>二、Web IM 集成</font>

#### 1.下载集成案例 （下载的文件名为webim）

```shell
git clone https://github.com/easemob/webim.git
```

<img src="https://img-blog.csdnimg.cn/20201014144914299.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center" style="zoom: 50%;" />

#### 2. 复制案例中的文件到项目中

- \webim\sdk目录下的所有js文件到项目resources\static\js中 
- \webim\simpleDemo目录下的demo.html和WebIMConfig.js放入项目resources\static\中

<img src="https://img-blog.csdnimg.cn/20201014150543930.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center" style="zoom:80%;" />

> demo.html文件是一个测试文档，依靠这个文档可以做集成

**注意事项：**

​	要修改demo.html文件引入js文件的路径，否则js路径会不对，因为这里是复制过来的

<img src="https://img-blog.csdnimg.cn/20201014154122352.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center" style="zoom:80%;" />

#### 3.启动springboot进行测试

 我的项目地址：`http://localhost:9008/demo.html `

**demo.html文件中提供了两个测试账号：**

- 
  测试帐号：<font color=red>**1c1c**</font>，密码：<font color=red>**111**</font>
  
- 
  测试账号：<font color=red>**1v1v**</font>，密码：<font color=red>**111**</font>
  
  <img src="https://img-blog.csdnimg.cn/20201014153848211.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center" style="zoom: 80%;" />

<img src="https://img-blog.csdnimg.cn/2020101415395247.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center" style="zoom:80%;" />

> 登录之后可以看到连接建立成功代表成功登录了，之后在新打开一个同样页面在登录另一个测试账号

**登录好另一个测试账号就可以发消息进行测试了：**

- **1v1v**给**1c1c**发送消息

  <img src="https://img-blog.csdnimg.cn/20201014154903465.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center" style="zoom: 80%;" />

- **1c1c**收到**1v1v**发送的消息

  <img src="https://img-blog.csdnimg.cn/20201014155121397.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center" style="zoom:80%;" />

>  简单的单聊测试就完成了，后面就是将demo.html文件中需要用到的功能集成到自己的html文件中就OK了

#### 4. 集成demo文件的功能到自己的HTML文件中

- 以我自己的HTML文件chatroom为例，先从demo文件中复制`meta`和`script`标签的信息到chatroom中

  <img src="https://img-blog.csdnimg.cn/20201014155837534.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center" style="zoom: 80%;" />

- 修改WebIMConfig.js文件中`appkey`为自己创建的IM项目中的`appkey`（在自己IM中注册用户等操作必须的）

  <img src="https://img-blog.csdnimg.cn/20201014161711600.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center" style="zoom:80%;" />

- 从demo文件中将以下这段**建立连接**复制到chatroom的`script`标签中（建立连接必须的）

	```js
    /*建立连接*/
    var conn = {};
    console.log(WebIM, window.WebIM);
    WebIM.config = config;
    conn = WebIM.conn = new WebIM.default.connection({
        appKey: WebIM.config.appkey,
        isHttpDNS: WebIM.config.isHttpDNS,
        isMultiLoginSessions: WebIM.config.isMultiLoginSessions,
        host: WebIM.config.Host,
        https: WebIM.config.https,
        url: WebIM.config.xmppURL,
        apiUrl: WebIM.config.apiURL,
        isAutoLogin: false,
        heartBeatWait: WebIM.config.heartBeatWait,
        autoReconnectNumMax: WebIM.config.autoReconnectNumMax,
        autoReconnectInterval: WebIM.config.autoReconnectInterval,
        isStropheLog: WebIM.config.isStropheLog,
        delivery: WebIM.config.delivery
    });
  ```

- 从demo文件中将以下这段**回调方法**复制到chatroom的`script`标签中（监听必须的）

  ```js
      /*回调方法*/
      conn.listen({
          onOpened: function (message) {          //连接成功回调
              var myDate = new Date().toLocaleString();
              console.log("%c [opened] 连接已成功建立", "color: green");
              console.log(myDate);
              // rek();
              // alert(myDate + "登陆成功")
          },
          onClosed: function (message) {
              console.log("onclose:" + message);
              console.log(error);
          },         //连接关闭回调
          onTextMessage: function (message) {
              $("#log-container").append("<div class='bg-success'><label class='text-info'> 收到用户id为："+message.from+";发的消息是：</label><div class='text-info'>"+message.data+"</div></div><br>");
              console.log('onTextMessage: ', message);
          },  //收到文本消息
      });
  ```

**以上都是必须完成的步骤，后面的内容根据实际开放场景进行添加：**

具体操作步骤查阅[Web IM集成开放文档](http://docs-im.easemob.com/im/web/intro/basic)

<img src="https://img-blog.csdnimg.cn/2020101416134221.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center" style="zoom:80%;" />

我的chatroom.html文件（可供参考）

```html
<!DOCTYPE HTML>
<html>
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="edge" />
    <title>聊天室</title>

    <script src="./WebIMConfig.js"></script>
    <script src="../js/webimSDK3.0.4.js"></script>
    <script src="../js/EMedia_x1v1.js"></script>

    <script src="./js/jquery-1.12.3.min.js"></script>
    <link rel="stylesheet" href="//cdn.bootcss.com/bootstrap/3.3.5/css/bootstrap.min.css">
    <script src="//cdn.bootcss.com/bootstrap/3.3.5/js/bootstrap.min.js"></script>
    <style>
        body {
            margin-top: 5px;
        }
    </style>
</head>
<body>
<div class="container">
    <div class="row">
        <div class="col-md-3">
            <div class="panel panel-primary">
                <div class="panel-heading">
                    <h3 class="panel-title">用户注册或登录环信IM</h3>
                </div>
                <div class="panel-body">
                    <input type="text" class="form-control" id="userId" placeholder="用户id"/><br>
                    <button id="reg" type="button" class="btn btn-primary">注册</button>
                    <button id="login" type="button" class="btn btn-primary">登录</button>
                </div>
            </div>
            <div class="panel panel-primary" id="online">
                <div class="panel-heading">
                    <h3 class="panel-title">当前在线的其他用户</h3>
                </div>
                <div class="panel-body">
                    <input type="text" class="form-control" id="toUserId" placeholder="接收消息用户id"/><br>
                </div>
            </div>
            <div class="panel panel-primary">
                <div class="panel-heading">
                    <h3 class="panel-title">群发系统广播</h3>
                </div>
                <div class="panel-body">
                    <input type="text" class="form-control" id="msg"/><br>
                    <button id="broadcast" type="button" class="btn btn-primary">发送</button>
                </div>
            </div>
        </div>
        <div class="col-md-9">
            <div class="panel panel-primary">
                <div class="panel-heading">
                    <h3 class="panel-title" id="talktitle"></h3>
                </div>
                <div class="panel-body">
                    <div class="well" id="log-container" style="height:400px;overflow-y:scroll">

                    </div>
                    <input type="text" id="myinfo" class="form-control col-md-12"/> <br>
                    <button id="send" type="button" class="btn btn-primary">发送</button>
                </div>
            </div>
        </div>
    </div>
</div>
<script>
    /*建立连接*/
    var conn = {};
    console.log(WebIM, window.WebIM);
    WebIM.config = config;
    conn = WebIM.conn = new WebIM.default.connection({
        appKey: WebIM.config.appkey,
        isHttpDNS: WebIM.config.isHttpDNS,
        isMultiLoginSessions: WebIM.config.isMultiLoginSessions,
        host: WebIM.config.Host,
        https: WebIM.config.https,
        url: WebIM.config.xmppURL,
        apiUrl: WebIM.config.apiURL,
        isAutoLogin: false,
        heartBeatWait: WebIM.config.heartBeatWait,
        autoReconnectNumMax: WebIM.config.autoReconnectNumMax,
        autoReconnectInterval: WebIM.config.autoReconnectInterval,
        isStropheLog: WebIM.config.isStropheLog,
        delivery: WebIM.config.delivery
    });

    /*回调方法*/
    conn.listen({
        onOpened: function (message) {          //连接成功回调
            var myDate = new Date().toLocaleString();
            console.log("%c [opened] 连接已成功建立", "color: green");
            console.log(myDate);
            // rek();
            // alert(myDate + "登陆成功")
        },
        onClosed: function (message) {
            console.log("onclose:" + message);
            console.log(error);
        },         //连接关闭回调
        onTextMessage: function (message) {
            $("#log-container").append("<div class='bg-success'><label class='text-info'> 收到用户id为："+message.from+";发的消息是：</label><div class='text-info'>"+message.data+"</div></div><br>");
            console.log('onTextMessage: ', message);
        },  //收到文本消息
    });

    /*注册*/
    var userId; //用户id
    var password; //密码
    var nickname; //昵称
    document.getElementById('reg').onclick = function () {
        var userId = document.getElementById("userId").value;

        $.ajaxSettings.async = false; //关闭ajax异步，改为同步，因为要保证先获取到信息
        $.get("/user/"+userId,function (data){
            password = data.data.password;
            nickname = data.data.nickname;
        });

        var option = {
            //用户id
            username: userId,
            //密码
            password: password,
            //昵称
            nickname: nickname,
            appKey: WebIM.config.appkey,
            success: function () {
                console.log('注册成功');
            },
            error: function () {
                console.log('注册失败');
            },
            apiUrl: WebIM.config.apiURL
        };
        conn.signup(option);
    };

    /*登录*/
    document.getElementById('login').onclick = function () {
        console.log(WebIM, window.WebIM);
        userId = document.getElementById("userId").value;
        $.ajaxSettings.async = false; //关闭ajax异步，改为同步，因为要保证先获取到信息
        $.get("/user/"+userId,function (data){
            password = data.data.password;
        });
        options = {
            apiUrl: WebIM.config.apiURL,
            user: userId,
            pwd: password,
            appKey: WebIM.config.appkey
        };
        conn.open(options);
        console.log(options)
    };

    /*
       * Message
       */
    //文本消息
    var conf = WebIM.config
    //var WebIM = WebIM.default
    WebIM.config = conf
    WebIM.message = WebIM.default.message
    WebIM.utils = WebIM.default.utils
    WebIM.debug = WebIM.default.debug
    WebIM.statusCode = WebIM.default.statusCode

    var myDate = new Date().toLocaleString();
    document.getElementById('send').onclick = function () {
        var toUserId = document.getElementById("toUserId").value;
        var tmsg = document.getElementById("myinfo").value;
        var id = conn.getUniqueId();                 // 生成本地消息id
        var msg = new WebIM.default.message('txt', id);      // 创建文本消息
        msg.set({
            msg: tmsg,                  // 消息内容
            to: toUserId,				// 接收消息对象（用户id）
            ext: {
                'time': myDate
            },                       
            success: function (id, serverMsgId) {
                console.log('send private text Success');
                msgText = msg.body.msg;
            },
            fail: function (e) {
                console.log("Send private text error");
            }
        });
        msg.body.chatType = 'singleChat';
        conn.send(msg.body);
        $("#log-container").append("<div class='bg-success'><label class='text-info'>用户id为："+userId+";发的消息是：</label><div class='text-info'>"+tmsg+"</div></div><br>");
        console.log(msg);

    };
</script>

</body>
</html>
```

#### 5. 测试使用

- 在mysql数据库中建立表，有`userId`，和`password`

  <img src="https://img-blog.csdnimg.cn/202010141631086.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center"  />

- 在controller中编写`findById`方法用来获取`password`和`ickname`

  <img src="https://img-blog.csdnimg.cn/20201014170529230.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center" style="zoom:80%;" />

- 通过我的chatroom文件中可以看到，通过`userId`就可以获取`password`和`nickname` （注册和登录是同样的逻辑）

  <img src="https://img-blog.csdnimg.cn/20201014163405363.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center" style="zoom:80%;" />

- 注册两个账户，userId分别为`1`和`2`用来进行测试

  <img src="https://img-blog.csdnimg.cn/20201014164548896.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center" style="zoom: 80%;" />

  <img src="https://img-blog.csdnimg.cn/2020101416492466.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center" style="zoom:80%;" />

- 将用户1和用户2分别登录后进行单聊测试

  **用户1给用户2发送消息：**

  <img src="https://img-blog.csdnimg.cn/20201014165416735.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center" style="zoom:80%;" />

  **用户2收到了用户1发过来的消息：**

  <img src="https://img-blog.csdnimg.cn/20201014165648430.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpc2h1d2VuNzk4Ng==,size_16,color_FFFFFF,t_70#pic_center" style="zoom:80%;" />

> OK，单聊的集成就算是完成了，如果需要集成更多的内容可以从demo.html文件中复制过来，也可以参照文档说明
