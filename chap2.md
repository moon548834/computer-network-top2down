## 第二章

NOTE:从这章开始，使用第七版

### 2.1 应用层原则

client: 开始通信的进程
socket: 一个接口，可以看成应用和网络的接口API

为了表明接收信息的进程，我们需要规定两种信息，1.host端的地址，2.一个标识符用来表征哪一个接收端的进程

host地址就是一个独特的IP地址，另外的信息是通过端口来提供的，一些应用分配给了特殊的端口，比如网页服务是80，邮件服务是25。

网络提供不止一种的传输层的协议，我们可以通过以下四种维度界定这些协议:可信传输，吞吐量，时延，安全性。

如果这个协议可以确保数据准确传输，那么就是可信的。一些音视频应用程序(如Skype)就不需要这个保证。

对于吞吐量而言，有些应用需要确保速率不变，这称之为带宽敏感。

TCP/IP网络提供了两种传输层的协议：TCP,UDP。

TCP：提供可信传输，在传输信息前进行握手，并且拥有处理阻塞的机制，当出现阻塞时，就减少(遏制)那个传输的进程。TCP没有提供加密的机制，不过一种叫SSL(Secure Sockets Layer)的东西可以实现这个功能，SSL并不是一个和TCP，UDP具有同等地位的传输协议，他只是TCP的加强版。如果一个应用想使用SSL的服务，那么在客户端和服务器端都需要假如相应的代码。

UDP：轻量级的传输协议，没有握手，不提供阻塞抑制极致化，也没有加密措施。

对于时延和吞吐量，其实这两种协议都没有办法保证,(好吧，个人感觉和底层的layer有关)

应用层的协议一般定义了如下内容:

- 报文的种类
- 报文的语法，划定的位(?没太懂，反正这几部分差不多就是构成报文的格式)
- 报文的语义，哪些位代表哪些意思
- 进程发送和响应报文的规则

### 2.2 Web和TTTP

HTTP(HyperText Transfer Protocol),是Web应用层的协议，一个网页由多个对象构成，一个对象可能是一个简单的文件如HTML页面，一个图片。大多数页面包含一个基本的HTML文件和多个引用的对象。比如一个网页由HTML和5张JPEG图片构成，那么它就有6个对象。

基本的HTML通过其他对象的URL来引用他们，URL由两部分组成 hostname 和 pathname，举例来说

```
http://www.someSchool.edu/someDepartment/picture.gif
```

`www.someSchool.edu`是hostname，而`/someDepartment/picture.gif/`则是pathname。

HTTP使用TCP作为传输层的协议，HTTP客户端首先初始化和server端的连接，这通过socket接口实现，类似的，server端也通过socket接口来发送响应的报文。

HTTP是一个无状态保存的协议，因为HTTP的server端并不保存任何客户端的信息。

在HTTP/1.0里面，请求一个文件就需要初始化关闭TCP连接，这使得每次请求都需要2个RTT加上一个发送时间

<div align=center>  
 
![](./IMG/2-2-2-none_persistence_connection.PNG)

</div>

这种非连续连接会使得服务器负担很重，所以在HTTP/1.1中改进出了连续连接的方式，它可以让连接在返回响应后仍保持一定的时间，默认的连续连接模式是流水的，这就可以使得客户端连续发送报文请求，每次的间隔只有1个RTT，并且server端一直处于有事做的状态。

#### HTTP报文

HTTP报文的格式的一个例子：

```
GET /somedir/page.html HTTP/1.1
Host: www.someschool.edu
Connection: close
User-agent: Mozilla/5.0
Accept-language: fr
```

HTTP的第一行是request line,剩下的行叫作header lines,request line 包含三部分:method, URL, version。

method除了最常用的GET外还有POST，HEAD，PUT和DELETE

下面是一个通用的报文格式

<div align=center>  
 
![](./IMG/2-2-3-general_HTTP_message.PNG)

</div>

因为上个例子里面，method是GET，所以body是空的，而对于POST而言，POST一般使用在用户填写一些信息的时候，body就不是空的了。

>  a request generated with a form does not necessarily use the POST method. Instead, HTML forms often use the GET method and include the inputted data (in the form fields) in the requested URL. 实际上一般用POST的也比较少；使用GET方法中的数据位表示也可。

对于其他方法而言 PUT: 代表存储上传一个对象到URL中, HEAD和DELTE顾名思义。

对于server端回复的报文，十分类似

```
HTTP/1.1 200 OK
Connection: close
Date: Tue, 18 Aug 2015 15:44:04 GMT
Server: Apache/2.2.3 (CentOS)
Last-Modified: Tue, 18 Aug 2015 15:11:03 GMT
Content-Length: 6821
Content-Type: text/html
(data data data data data ...)
```

只不过是把request line变成了status line,下面是一些常见的状态码:

| status code |  meaning |
| ---- | --------------------|
| 200  | OK                  |
| 301  | moved Permanently   |
| 400  | bad request         |
| 404  | not found           |
| 505  | version not support |


在谢希仁的书中提到，`4xx`表示客户的差错，`5xx`则表示服务器的问题。

#### Cookies

下面讨论Cookies，虽然HTTP是一个不保留状态的协议，但是对于server端而言，有时候保留客户端的信息也是一件需求，通过Cookies网站就可以追踪用户的信息了

一般而言，Cookies由四部分构成 
- HTTP响应时候的header line(Set Cookie: xxx)
- HTTP请求时候的header line(Cookie: xxx)
- 用户系统的Cookies文件
- 网站保存Cokkies的数据库

<div align=center>  
 
![](./IMG/2-2-4-cookies.PNG)

</div>

这样Amazon就可以知道给我推荐什么东西了，而且我买东西的时候只需傻瓜一件点击即可，登录的操作也免除了。

这也带来了关于隐私的讨论，我也很赞同隐私在这中间受到了一定的损害。

#### Web cache

Web cache或者说proxy server,代理服务器是拥有自己的硬盘空间来保存最近的请求对象数据，这样用户并不直接去访问server端，而是先看看web cache里面有没有，如果没有web cache就去访问server。

通常Web cache是ISP购买安装的，一般学校里面也会有，这样会大大降低网络的拥堵程度，减少请求的时间。

<div align=center>  
 
![](./IMG/2-2-5-web_cache.PNG)

</div>

#### conditional GET

就像内存和cache的关系一样，有更新的问题，这里才用的方法是发送GET的同时，包含这样一个header line:`If-Modified-Since`(由web cache发送给最终的服务器端),如果修改了就更新即可。