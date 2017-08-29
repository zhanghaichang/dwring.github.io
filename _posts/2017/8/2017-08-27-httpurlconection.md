---
layout: post
title: 详解HttpURLConnection
category: java
tags: [java]
---
# 请求响应流程

![请求响应流程](http://img.blog.csdn.net/20150129101639203?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd294dWVsaXV5dW4=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)


# 设置连接参数的方法

*   setAllowUserInteraction
*   setDoInput
*   setDoOutput
*   setIfModifiedSince
*   setUseCaches
*   setDefaultAllowUserInteraction
*   setDefaultUseCaches

# 设置请求头或响应头

HTTP请求允许一个key带多个用逗号分开的values，但是HttpURLConnection只提供了单个操作的方法：

*   setRequestProperty(key,value)
*   addRequestProperty(key,value)

setRequestProperty和addRequestProperty的区别就是，setRequestProperty会覆盖已经存在的key的所有values，有清零重新赋值的作用。而addRequestProperty则是在原来key的基础上继续添加其他value。

# 发送URL请求

建立实际连接之后，就是发送请求，把请求参数传到服务器，这就需要使用outputStream把请求参数传给服务器：

*   getOutputStream 

# 获取响应

请求发送成功之后，即可获取响应的状态码，如果成功既可以读取响应中的数据，获取这些数据的方法包括：

*   getContent
*   getHeaderField
*   getInputStream 

对于大部分请求来说，getInputStream和getContent是用的最多的。

相应的信息头用以下方法获取：

*   getContentEncoding
*   getContentLength
*   getContentType
*   getDate
*   getExpiration
*   getLastModifed 

# HttpURLConnection

任何网络连接都需要经过socket才能连接，HttpURLConnection不需要设置socket，所以，HttpURLConnection并不是底层的连接，而是在底层连接上的**一个请求**。这就是为什么HttpURLConneciton只是一个**<span style="color:red;">抽象类</span>**，自身不能被实例化的原因。HttpURLConnection只能通过URL.openConnection()方法创建具体的实例。

虽然**底层**的网络连接可以被多个HttpURLConnection实例共享，但每一个HttpURLConnection实例只能发送一个请求。请求结束之后，应该调用HttpURLConnection实例的InputStream或OutputStream的close()方法以释放请求的网络资源，不过这种方式对于持久化连接没用。对于持久化连接，得用disconnect()方法关闭底层连接的socket。

## 创建HttpURLConnection

<div style="background:#FAFAFA;">

<span style="color:#2B91AF;"></span>

<pre name="code" class="java">URL url = new URL("http://localhost:8080/xxx.do");  

URLConnection rulConnection = url.openConnection();// 此处的urlConnection对象实际上是根据URL的  
// 请求协议(此处是http)生成的URLConnection类  
// 的子类HttpURLConnection,故此处最好将其转化  
// 为HttpURLConnection类型的对象,以便用到  
// HttpURLConnection更多的API.如下:  

HttpURLConnection httpUrlConnection = (HttpURLConnection) rulConnection;  </pre>

</div>

## 设置HttpURLConnection参数

<div style="background:#FAFAFA;">

<span style="color:#2B91AF;"></span>

<pre name="code" class="java">// 设定请求的方法为"POST"，默认是GET  
httpUrlConnection.setRequestMethod("POST");  

// 设置是否向httpUrlConnection输出，因为这个是post请求，参数要放在  
// http正文内，因此需要设为true, 默认情况下是false;  
httpUrlConnection.setDoOutput(true);  

// 设置是否从httpUrlConnection读入，默认情况下是true;  
httpUrlConnection.setDoInput(true);  

// Post 请求不能使用缓存  
httpUrlConnection.setUseCaches(false);  

// 设定传送的内容类型是可序列化的java对象  
// (如果不设此项,在传送序列化对象时,当WEB服务默认的不是这种类型时可能抛java.io.EOFException)  
httpUrlConnection.setRequestProperty("Content-type", "application/x-java-serialized-object");  

// 连接，从上述url.openConnection()至此的配置必须要在connect之前完成，  
httpUrlConnection.connect();  </pre>

</div>

## URLConnection建立连接

<div style="background:#FAFAFA;">

<span style="color:#2B91AF;"></span>

</div>

<pre name="code" class="java">// 此处getOutputStream会隐含的进行connect(即：如同调用上面的connect()方法，  
// 所以在开发中不调用上述的connect()也可以)。  
OutputStream outStrm = httpUrlConnection.getOutputStream();  </pre>

<pre name="code" class="java">

getInputStream()也是同理。

</pre>

## HttpURLConnection发送请求

<div style="background:#FAFAFA;">

<span style="color:#2B91AF;"></span>

<pre name="code" class="java">// 现在通过输出流对象构建对象输出流对象，以实现输出可序列化的对象。  
ObjectOutputStream objOutputStrm = new ObjectOutputStream(outStrm);  

// 向对象输出流写出数据，这些数据将存到内存缓冲区中  
objOutputStrm.writeObject(new String("我是测试数据"));  

// 刷新对象输出流，将任何字节都写入潜在的流中（些处为ObjectOutputStream）  
objOutputStm.flush();  

// 关闭流对象。此时，不能再向对象输出流写入任何数据，先前写入的数据存在于内存缓冲区中,  
// 在调用下边的getInputStream()函数时才把准备好的http请求正式发送到服务器  
objOutputStm.close();  </pre>

</div>

## HttpURLConneciton获取响应

<div style="background:#FAFAFA;">

<span style="color:#2B91AF;"> </span><span style="color:#008200;">// </span><span style="color:#008200;">调用</span><span style="color:#008200;">HttpURLConnection</span><span style="color:#008200;">连接对象的</span><span style="color:#008200;">getInputStream()</span><span style="color:#008200;">函数</span><span style="color:#008200;">,</span>  

InputStream inStrm = httpConn.getInputStream(); 

</div>

## 设置POST参数

<div style="background:#FAFAFA;">

<span style="color:#2B91AF;"></span>

<pre name="code" class="java">OutputStream os = httpConn.getOutputStream();  
             String param = new String();  
             param = "CorpID=" + CorpID +  
                     "&LoginName=" + LoginName+  
                     "&send_no=" + phoneNumber +  
                     "&msg=" + java.net.URLEncoder.encode(msg,"GBK"); ;  
             os.write(param.getBytes());  </pre>

</div>

## 超时设置，防止 网络异常的情况下，可能会导致程序僵死而不继续往下执行

<div style="background:#FAFAFA;">

<span style="color:#2B91AF;"></span>

<div style="background:#FAFAFA;">

System.setProperty(<span style="color:blue;">"sun.net.client.defaultConnectTimeout"</span>, <span style="color:blue;">"30000"</span>);  

System.setProperty(<span style="color: blue;">"sun.net.client.defaultReadTimeout"</span>, <span style="color: blue;">"30000"</span>);  

<span style="color:#2B91AF;"> </span>

其中： sun.net.client.defaultConnectTimeout：连接主机的超时时间（单位：毫秒）  

sun.net.client.defaultReadTimeout：从主机读取数据的超时时间（单位：毫秒）  

JDK <span style="color:#C00000;">1.5</span>以前的版本，只能通过设置这两个系统属性来控制网络超时。在<span style="color:#C00000;">1.5</span>中，还可以使用HttpURLConnection的父类URLConnection的以下两个方法：  

setConnectTimeout：设置连接主机超时（单位：毫秒）  

setReadTimeout：设置从主机读取数据超时（单位：毫秒）  

例如：  

HttpURLConnection urlCon = (HttpURLConnection)url.openConnection();  

urlCon.setConnectTimeout(<span style="color:#C00000;">30000</span>);  

urlCon.setReadTimeout(<span style="color:#C00000;">30000</span>);  

---

#### URLConnection和HttpURLConnection使用的都是java.net中的类，属于标准的java接口。HttpURLConnection继承自URLConnection,差别在与HttpURLConnection仅仅针对Http连接。

#### 基本步骤:
    # 1) 创建 URL 以及 URLConnection / HttpURLConnection 对象
    #2) 设置连接参数
    #3) 连接到服务器
    #4) 向服务器写数据
    #5)从服务器读取数据

```java
public void urlConnection()
   {
	   String urltext = "";
	   try {
//		  方法一：
		  URL url = new URL(urltext);
		  URLConnection conn = url.openConnection();//取得一个新的链接对指定的URL
		  conn.connect();//本方法不会自动重连
		  InputStream is = conn.getInputStream();
		  is.close();//关闭InputStream
//		  方法二：
		  URL url2 = new URL(urltext);
		  InputStream is2 = url2.openStream();
		  is2.close();//关闭InputStream
		  //URL对象也提供取得InputStream的方法。URL.openStream()会打开自动链接，所以不需要运行openConnection

		  //方法三：本方法同一，但是openConnection返回值直接转为HttpsURLConnection，
          //这样可以使用一些Http连接特有的方法,如setRequestMethod
		  URL url3 = new URL(urltext);
		  HttpsURLConnection conn3 =(HttpsURLConnection)url.openConnection();
		  conn3.setRequestMethod("POST");
		  //允许Input、Output,不使用Cache
		  conn3.setDoInput(true);
		  conn3.setDoOutput(true);
		  conn3.setUseCaches(false);
		  /*
		   * setRequestProperty
		   */
		  conn3.setRequestProperty("Connection", "Keep-Alive");
		  conn3.setRequestProperty("Charset", "UTF-8");
		  conn3.setRequestProperty("Content-type", "multipart/form-data;boundary=*****");
		  //在与服务器连接之前，设置一些网络参数
		  conn3.setConnectTimeout(10000);

		  conn3.connect();
		// 与服务器交互:向服务器端写数据，这里可以上传文件等多个操作
		  OutputStream outStream = conn3.getOutputStream();
		  ObjectOutputStream objOutput = new ObjectOutputStream(outStream);
		  objOutput.writeObject(new String("this is a string…"));
		  objOutput.flush();

		  // 处理数据, 取得响应内容
		  InputStream is3 = conn.getInputStream();
		  is3.close();//关闭InputStream
	   } catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
   }
```
