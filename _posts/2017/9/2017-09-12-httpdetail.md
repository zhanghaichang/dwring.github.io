---
layout: post
title: HTTP协议详解
category: other
tags: [other]
---
http使用面向连接的TCP作为传输层协议。http本身无连接。

*   请求报文
    CRLF是回车换行
    ![这里写图片描述](http://img.blog.csdn.net/20170815173046559?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzUxMTYzNTM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
    方法为GET的请求报文

```
GET /search?hl=zh-CN&source=hp&q=domety&aq=f&oq= HTTP/1.1  
Accept: image/gif, image/x-xbitmap, image/jpeg, image/pjpeg, application/vnd.ms-excel, application/vnd.ms-powerpoint, 
application/msword, application/x-silverlight, application/x-shockwave-flash, */*  
Referer: <a href="http://www.google.cn/">http://www.google.cn/</a>  
Accept-Language: zh-cn  
Accept-Encoding: gzip, deflate  
User-Agent: Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 2.0.50727; TheWorld)  
Host: <a href="http://www.google.cn">www.google.cn</a>  
Connection: Keep-Alive  
Cookie: PREF=ID=80a06da87be9ae3c:U=f7167333e2c3b714:NW=1:TM=1261551909:LM=1261551917:S=ybYcq2wpfefs4V9g; 
NID=31=ojj8d-IygaEtSxLgaJmqSjVhCspkviJrB6omjamNrSm8lZhKy_yMfO2M4QMRKcH1g0iQv9u-2hfBW7bUFwVh7pGaRUb0RnHcJU37y-
FxlRugatx63JLv7CWMD6UB_O_r
```

方法为POST的请求报文

```
POST /search HTTP/1.1  
Accept: image/gif, image/x-xbitmap, image/jpeg, image/pjpeg, application/vnd.ms-excel, application/vnd.ms-powerpoint, 
application/msword, application/x-silverlight, application/x-shockwave-flash, */*  
Referer：<ahref="http://www.google.cn/">http://www.google.cn/</a>  
Accept-Language: zh-cn  
Accept-Encoding: gzip, deflate  
User-Agent: Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 2.0.50727; TheWorld)  
Host: <a href="http://www.google.cn">www.google.cn</a>  
Connection: Keep-Alive  
Cookie: PREF=ID=80a06da87be9ae3c:U=f7167333e2c3b714:NW=1:TM=1261551909:LM=1261551917:S=ybYcq2wpfefs4V9g; 
NID=31=ojj8d-IygaEtSxLgaJmqSjVhCspkviJrB6omjamNrSm8lZhKy_yMfO2M4QMRKcH1g0iQv9u-2hfBW7bUFwVh7pGaRUb0RnHcJU37y-
FxlRugatx63JLv7CWMD6UB_O_r  

hl=zh-CN&source=hp&q=domety
```

### 方法

*   OPTIONS：这个方法可使服务器传回该资源所支持的所有HTTP请求方法。用’*’来代替资源名称，向Web服务器发送OPTIONS请求，可以测试服务器功能是否正常运作。
*   HEAD：与GET方法一样，都是向服务器发出指定资源的请求。只不过服务器将不传回资源的本文部分。它的好处在于，使用这个方法可以在不必传输全部内容的情况下，就可以获取其中“关于该资源的信息”（元信息或称元数据）。
*   <font color="greep">**GET**</font>：向指定的资源发出“显示”请求。使用GET方法应该只用在读取数据，而不应当被用于产生“副作用”的操作中，例如在Web Application中。其中一个原因是GET可能会被网络蜘蛛等随意访问。参见安全方法
*   <font color="greep">**POST**</font>：向指定资源提交数据，请求服务器进行处理（例如提交表单或者上传文件）。数据被包含在请求本文中。这个请求可能会创建新的资源或修改现有资源，或二者皆有。
*   PUT：向指定资源位置上传其最新内容。
*   DELETE：请求服务器删除Request-URI所标识的资源。
*   TRACE：回显服务器收到的请求，主要用于测试或诊断。
*   CONNECT：HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器。通常用于SSL加密服务器的链接（经由非加密的HTTP代理服务器）。
    虽然HTTP的请求方式有8种，但是我们在实际应用中常用的也就是get和post，其他请求方式也都可以通过这两种方式间接的来实现。

### URL

URL一般的组成成分是<协议>://<主机>:<端口号>/<路径>

*   协议

http——超文本传输协议资源
https——用安全套接字层传送的超文本传输协议
ftp——文件传输协议
mailto——电子邮件地址
ldap——轻型目录访问协议搜索
file——当地电脑或网上分享的文件
news——Usenet新闻组
gopher——Gopher协议
telnet——Telnet协议

*   主机-是指在因特网上的域名
*   端口有时可省略
*   路径
    绝对URL（absolute URL）显示文件的完整路径，这意味着绝对URL本身所在的位置与被引用的实际文件的位置无关。
    相对URL（relative URL）以包含URL本身的文件夹的位置为参考点，描述目标文件夹的位置。
    如果路径省略URL就指到因特网上的某个主页。

```
https://zhidao.baidu.com/
https://zhidao.baidu.com/question/1742817.html
```

第一个URL省略了路径，代表百度知道的主页。
第二个是文件1742817.html的相对路径，指出了他的位置。
它们都使用https协议。端口号省略了。

### 版本号

以前使用的协议是HTTP/1.0 ,现在升级为HTTP/1.1。两个的区别是什么？

*   请求一个万维网文档需要的时间是**2*RTT+文档传输时间**。因为要和服务器建立TCP连接需要3次握手，在第三次握手的时候捎带了发送请求相关的数据，然后HTTP服务器响应报文总共是四次交互，也就是2*RTT时间。再加上一些其他的开销，万维网服务器要服务大量的客户，所以每次浏览都需要建立连接，HTTP/1.0中这种**非持续连接（短链接）**服务器负担很重。HTTP/1.1使用了**持续连接（长链接）**，服务器在发送响应后仍然保持这条连接。
    持续链接还分为流水线方式和非流水线方式。非流水线方式规定客户发送浏览请求得到响应后才能发送下一个。流水线方式客户不用等到响应就可以发送下一个请求，服务器收到请求后就可以连续响应，不用等待，节省了时间。

*   HTTP 1.1的持续连接，也需要增加新的请求头来帮助实现。

    　　例如，Connection请求头的值为Keep-Alive时，客户端通知服务器返回本次请求结果后保持连接；Connection请求头的值为close时，客户端通知服务器返回本次请求结果后关闭连接。

*   HTTP 1.1还提供了与身份认证、状态管理和Cache缓存等机制相关的请求头和响应头。

### HTTP报首部字段

从上面看`HTTP`一共有四种类型的首部字段`通用首部字段`，`请求首部字段`，`响应首部字段`，`实体首部字段`。

*   通用首部字段：请求报文和响应报文两方都会使用的首部。
*   请求首部字段：从客户端向服务器发送请求报文时使用的首部。
*   响应首部字段：从服务器向客户端返回响应报文时使用的首部。
*   实体首部字段：针对请求报文和响应报文的实体部分使用的首部。

#### HTTP/1.1 首部字段

*   **通用首部字段**

| 首部字段名 | 说明 |
| :-- | :-- |
| Cache | 控制缓存的行为 |
| Connection | 逐跳首部、连接的管理 |
| Date | 创建报文的日期时间 |
| Pragma | 报文指令 |
| Trailer | 报文末端的首部一览 |
| Transfer-Encoding | 指定报文主体的传输编码方式 |
| Upgrade | 升级为其他协议 |
| Via | 代理服务器的相关信息 |
| Warning | 错误通知 |

*   **请求首部字段**

| 首部字段名 | 说明 |
| :-- | :-- |
| Accept | 用户代理可处理的媒体类型 |
| Accept-Charset | 优先的字符集 |
| Accept-Encoding | 优先的内容编码 |
| Accept-Language | 优先的语言（自然语言） |
| Authorization | Web认证信息 |
| Expect | 期待服务器的特定行为 |
| From | 用户的电子邮箱地址 |
| Host | 请求资源所在服务器 |
| if-Match | 比较实体标记（ETag） |
| if-Modified-Since | 比较资源的更新时间 |
| if-None-Match | 比较实体标记（与if-Match相反） |
| if-Range | 资源未更新时发送实体Byte的范围请求 |
| if-Unmodified-Since | 比较资源的更新时间（与if-Modified-Since相反） |
| Max-Forwards | 最大传输逐跳数 |
| Proxy-Authorization | 代理服务器要求客户端的认证信息 |
| Range | 实体的字节范围请求 |
| Referer | 对请求中URI的原始获取方法 |
| TE | 传输编码的优先级 |
| User-Agent | HTTP客户端程序的信息 |

*   **响应首部字段**

| 首部字段名 | 说明 |
| :-- | :-- |
| Accept-Ranges | 是否接受字节范围请求 |
| Age | 推算资源创建经过时间 |
| ETag | 资源的匹配信息 |
| Location | 令客户端重定向至指定的URI |
| Proxy-Authenticate | 代理服务器对客户端的认证信息 |
| Reter-After | 对再次发起请求的时机要求 |
| Server | HTTP服务器的安装信息 |
| Vary | 代理服务器缓存的管理信息 |
| WWW-Authenticate | 服务器对客户端的认证信息 |

*   **实体首部字段**

| 首部字段名 | 说明 |
| :-- | :-- |
| Allow | 资源可支持的HTTP方法 |
| Content-Encoding | 实体主体的适用的编码方式 |
| Content-Language | 实体主体的自然语言 |
| Content-Length | 实体主体的大小（单位：字节） |
| Content-Location | 替代对应资源的URI |
| Content-MD5 | 实体主体的报文摘要 |
| Content-Range | 实体主体的位置范围 |
| Content-Type | 实体主体的媒体类型 |
| Expires | 实体主体过期的日期时间 |
| Last-Modified | 资源的最后修改日期时间 |

### http操作过程

http是面向事物的应用层协议。每个万维网站点都有一个服务器进程，不断监听tcp 80端口，以便发现有浏览器向他发出连接请求，一旦建立连接，浏览器就向万维网服务器发出某个页面的浏览请求。浏览器与服务器必须按照规定的格式和遵循一定的规则，这些规则就是超文本传输协议http。
用HTTP/1.0说明用户发出浏览请求（在浏览器地址输入URL或者鼠标点击可选事件，浏览器会自动找到所要连接的页面）后的事件。
1\. 浏览器分析URL。
2\. 向DNS请求解析域名的IP地址。
3\. 得到IP地址。
3\. 浏览器服务器建立TCP连接（IP地址+端口号）。
4\. 发出取文件命令如上面URL中 GET /question/1742817.html
5\. 服务器做出响应吧1742817.html发送给浏览器。
6\. 释放TCP连接。
7\. 浏览器显示html中的文本。

*   响应报文
    ![这里写图片描述](http://img.blog.csdn.net/20170815213210179?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzUxMTYzNTM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
    ![这里写图片描述](http://img.blog.csdn.net/20170815211707737?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzUxMTYzNTM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 状态码和短语

1xx：指示信息–表示请求已接收，继续处理。
2xx：成功–表示请求已被成功接收、理解、接受。
3xx：重定向–要完成请求必须进行更进一步的操作。
4xx：客户端错误–请求有语法错误或请求无法实现。
5xx：服务器端错误–服务器未能实现合法的请求。

常见状态代码、状态描述的说明如下。

200 OK：客户端请求成功。
400 Bad Request：客户端请求有语法错误，不能被服务器所理解。
401 Unauthorized：请求未经授权，这个状态代码必须和WWW-Authenticate报头域一起使用。
403 Forbidden：服务器收到请求，但是拒绝提供服务。
404 Not Found：请求资源不存在，举个例子：输入了错误的URL。
500 Internal Server Error：服务器发生不可预期的错误。
503 Server Unavailable：服务器当前不能处理客户端的请求，一段时间后可能恢复正常，举个例子：HTTP/1.1 200 OK（CRLF）。

### GET方法和POST方法的区别

[参考链接](http://www.cnblogs.com/biyeymyhjob/archive/2012/07/28/2612910.html)
1.GET提交，请求的数据会附在URL之后（就是把数据放置在HTTP协议头＜request-line＞中），以?分割URL和传输数据，多个参数用&连接;例如：login.action?name=hyddd&password=idontknow&verify=%E4%BD%A0 %E5%A5%BD。如果数据是英文字母/数字，原样发送，如果是空格，转换为+，如果是中文/其他字符，则直接把字符串用BASE64加密，得出如： %E4%BD%A0%E5%A5%BD，其中％XX中的XX为该符号以16进制表示的ASCII。

POST提交：把提交的数据放置在是HTTP包的包体＜request-body＞中。上文示例中红色字体标明的就是实际的传输数据

因此，GET提交的数据会在地址栏中显示出来，而POST提交，地址栏不会改变

2.传输数据的大小：

首先声明,HTTP协议没有对传输的数据大小进行限制，HTTP协议规范也没有对URL长度进行限制。 而在实际开发中存在的限制主要有：

GET:特定浏览器和服务器对URL长度有限制，例如IE对URL长度的限制是2083字节(2K+35)。对于其他浏览器，如Netscape、FireFox等，理论上没有长度限制，其限制取决于操作系统的支持。

因此对于GET提交时，传输数据就会受到URL长度的限制。

POST:由于不是通过URL传值，理论上数据不受限。但实际各个WEB服务器会规定对post提交数据大小进行限制，Apache、IIS6都有各自的配置。

3.安全性：
POST的安全性要比GET的安全性高。注意：这里所说的安全性和上面GET提到的“安全”不是同个概念。上面“安全”的含义仅仅是不作数据修改，而这里安全的含义是真正的Security的含义，比如：通过GET提交数据，用户名和密码将明文出现在URL上，因为(1)登录页面有可能被浏览器缓存， (2)其他人查看浏览器的历史纪录，那么别人就可以拿到你的账号和密码了。
