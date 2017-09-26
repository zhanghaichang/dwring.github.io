---
layout: post
title: 关于浏览器缓存，cookie , session
category: other
tags: [other]
---
缓存分为2种：

1\.  强缓存： 直接从本地缓存中取资源，不会和服务器通信，返回的http状态码是200（from cache）；

2\.  协商缓存：通过服务器来告知是否能用本地缓存。先和服务器通信，如果返回可以使用本地缓存的指示，再从本地缓存中取（服务器不会返回被请求资源，返回的状态码是403（Not Modified））；如果不可以使用本地缓存就会返回最新的资源；

浏览器发起第二次相同的请求时会先判断能不能使用强缓存，不行的话，再判断能不能使用协商缓存（如果没有设置强缓存，协商缓存也不会生效）。

<span style="font-size: 16px;">一、强缓存</span>

<span style="font-size: 16px;">强缓存是由headers中的Expires和cache-control决定的，后者优先级高于前者。</span>

![](http://images2015.cnblogs.com/blog/553390/201701/553390-20170111154818681-1174872722.png)

Expries是http1.0提出的，表示失效时间（GMT格式），只有在这个时间之前的请求才可以用强缓存。

<style><!-- p.p1 {margin: 0.0px 0.0px 0.0px 0.0px; font: 13.0px 'PingFang SC'} --></style>

<span style="color: #333399; font-size: 14px;">第一次向服务器请求一个资源后，浏览器不仅会把资源保存起来，也会保存Reponse Headers，包括其中的Expires。第二次发请求时先去缓存中寻找这个资源，取到Expires，与当前的请求时间做对比，如果在Expires之前，则从缓存中取，否则重新向服务器请求，Expires在重新加载时被更新。</span>

cache-control可以有多个值：

cache-control: max-age=111000  -->表示自第一次收到响应后的111000ms以后可以用缓存

cache-control: no-cache   -->禁止使用强缓存

cache-control: no-store  -->禁止使用缓存，每次都要去服务器重新请求

cache-control: private  -->只允许被终端用户的浏览器端缓存

cache-control: public -->可以被所有用户缓存，保存终端用户和CDN等代理服务器

由于Expries是个绝对时间，由于各个客户端之间有时差就会导致缓存不一致的问题，所以http 1.1提出了<span style="color: #333399;">cache-control，是个相对时间，在第二次发请求时取到缓存中的max-age和第一次的请求时间计算出资源过期时间，与当前的请求时间对比决定是否使用缓存</span>。

<span style="font-size: 16px;">二、协商缓存 </span>

有两组headers值：Last-Modified / If-Modified-Since 和 Etag / If-None-Match，后者优先级高于前者。

第一次和第二次的同一个请求：

![](http://images2015.cnblogs.com/blog/553390/201701/553390-20170111171248931-939266891.png)     ![](http://images2015.cnblogs.com/blog/553390/201701/553390-20170111171453713-1917269453.png)

1\. Last-Modified / If-Modified-Since

<span style="color: #333399;">第一次请求时返回的Response Headers中用Last-Modified表示请求的资源在服务器上最新的修改时间，第二次请求时在Request Headers中用If-Modified-Since带上这个值发到服务器，服务器对比这个值和这个资源市价上的最新修改时间决定是否直接返回403还是返回资源。</span>当返回403时，表示资源没有更新，所以浏览器缓存中的Last-Modified也就不用更新了。

但是Last-Modified的问题在于有时服务器上资源其实有变化，但是最后修改时间却没有变化，所以有了Etag / If-None-Match来管理协商缓存。

2. Etag / If-None-Match

<span style="color: #333399;">Etag是服务器根据被请求资源生成的一个唯一标识字符串，只要资源发生变化，Etag就会变，跟资源的最新修改时间没有关系，能弥补Last-Modified的不足。与Last-Modified类似，第二次请求时请求头会带上If-None-Match标识的Etag值，区别是由于服务器每次会根据资源重新生成一个Etag，再拿它跟浏览器传过来的Etag对比，如果一致则返回403</span>，所以由于每次Etag都会重新生成，所以浏览器缓存中的Etag也必须每次都更新。

一般Last-Modified和Etag是同时启用的，但是对于分布式系统多同机器间文件的Last-Modified必须一致，以免因为负载均衡到不同机器导致比对不一致，分布式系统尽量关闭Etag,因为每台机器生成的Etag也不一致。

另外当使用F5刷新时会跳过强缓存，当强制刷新时，强缓存和协商缓存都会跳过。其他操作行为如前进后退，地址栏回车都会按正常流程走。

浏览器默认都会缓存图片，js，css等静态文件，也可以通过待会再响应头中设置是否要启用缓存，或是通过服务器专门的配置文件统一设置Expires, Cache-control等。

3\. 发布更新时，为了避免缓存，采用a.css?v1.1 和 a_v1.1.css的区别（ 覆盖式发布 和 非覆盖式发布）见[https://www.zhihu.com/question/20790576](https://www.zhihu.com/question/20790576)

<span style="font-size: 16px;">一、Cookie, Session</span>

<span style="font-size: 16px;">cookie和session都是为了弥补http协议的无状态特性，对server端来说无法知道两次http请求是否来自同一个用户，利用cookie和session就可以让用户只登录一次，server就知道某个请求是否需用重新登录。</span>

1\. cookie：客户端第一次正常访问服务器，服务器在response headers中返回与用户信息相关的cookie，客户端收到后把cookie保存在本地，下次再发请求时会在request headers中带上这个cookie，服务器收到这个cookie就知道用户状态了。cookie可以设置过期时间，默认值是-1，表示关闭浏览器时cookie就会失效，值为0时表示立马失效，相当于删除cookie（cookie没有删除的方法），服务器和客户端都可以设置cookie，但不可以操作另一个域名下的cookie。

2\. session: 客户端第一次正常访问服务器，服务器生成一个sessionid来标识用户并保存用户信息（服务器有一个专门的地方来保存所有用户的sessionId），在response headers中作为cookie的一个值返回，客户端收到后把cookie保存在本地，下次再发请求时会在request headers中带上这个sessionId，服务器通过查找这个sessionId就知道用户状态了，并更新sessionId的最后访问时间。sessionId也会可以设置失效时间，比如如果60分钟内某个session都没有被更新，服务器就会删除这个它。

总言之cookie是保存在客户端，session是存在服务器，session依赖于cookie。
