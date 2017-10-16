---
layout: post
title: Spring session共享
category: java
tags: [java]
---

# session共享

## 集群session共享机制
现在集群中使用的Session共享机制有两种，分别是session复制和session粘性。
* Session复制
该种方式下，负载均衡器会根据各个node的状态，把每个request进行分发，使用这样的测试，必须在多个node之间复制用户的session，实时保持整个集群中用户的状态同步。其中jboss的实现原理是使用拦截器，根据用户的同步策略拦截request，做完同步处理后再交给server产生响应。

优点：session不会被绑定到具体的node，只要有一个node存活，用户状态就不会丢失，集群能够正常工作。
缺点：node之间通信频繁，响应速度有影响，高并发情况下性能下降比较厉害。

* Session粘性
该种方式下，当用户发出第一个request后，负载均衡器动态的把该用户分配至到某个节点，并记录该节点的jvm路由，以后该用户的所有的request都会绑定到这个jvm路由，用户只会和该server交互。
优点：响应速度快，多个节点之间无需通信
缺点：某个node死掉之后，它负责的所有用户都会丢失session。
改进：servlet容器在新建、更新或维护session时，向其它no de推送修改信息。这种方式在高并发情况下同样会影响效率。
以上这两种方式都需要负载均衡器和Servlet容器的支持，在部署时需要单独配置负载均衡器和Servelt容器。

* 基于分布式缓存的session共享机制
将会话Session统一存储在分布式缓存中，并使用Cookie保持客户端和服务端的联系，每一次会话开始就生成一个GUID作为SessionID，保存在客户端的Cookie中，在服务端通过SessionID向分布式缓存中获取session。
实现思路：通过一个Filter过滤所有的request请求，在Filter创建request和session的代理，通过代理使用分布式缓存对session进行操作。这样实现对现有应用中对request对象的操作透明。

## 一：spring-session 介绍 
* 1.简介 
session一直都是我们做集群时需要解决的一个难题，过去我们可以从serlvet容器上解决，比如开源servlet容器-tomcat提供的tomcat-redis-session-manager、memcached-session-manager。 或者通过nginx之类的负载均衡做ip_hash，路由到特定的服务器上.但是这两种办法都存在弊端。
spring-session是spring旗下的一个项目，把servlet容器实现的httpSession替换为spring-session，专注于解决 session管理问题。
* 2.支持功能 
1）轻易把session存储到第三方存储容器，框架提供了redis、jvm的map、mongo、gemfire、hazelcast、jdbc等多种存储session的容器的方式。 
2）同一个浏览器同一个网站，支持多个session问题。 
3）Restful API，不依赖于cookie。可通过header来传递jessionID 
4）WebSocket和spring-session结合，同步生命周期管理。
* 3.集成方式 
集成方式非常简单[文档](http://docs.spring.io/spring-session/docs/1.3.0.RELEASE/reference/html5/) 
主要分为以下几个集成步骤： 
1）引入依赖jar包 
2）注解方式或者 xml方式配置 特定存储容器的存储方式，如redis的xml配置方式 spring.xml
```xml
<context:annotation-config/>    
/**  初始化一切spring-session准备，且把springSessionFilter放入IOC          **/
<beanclass="org.springframework.session.data.redis.config.annotation.web.http.RedisHttpSessionConfiguration"/>   
/** 这是存储容器的链接池 **/ 
<beanclass="org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory"/>
```
我们知道Tomcat再启动的时候首先会去加载web.xml 文件，Tomcat启动的时候web.xml被加载的顺序：context-param -> listener -> filter -> servlet。我们在使用Spring Session的时候，我们配置了一个filter，配置代码如下：
```xml
<filter>
     <filter-name>springSessionRepositoryFilter</filter-name>
     <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
     <filter-name>springSessionRepositoryFilter</filter-name>
     <url-pattern>/*</url-pattern>
     <dispatcher>REQUEST</dispatcher>
     <dispatcher>ERROR</dispatcher>
</filter-mapping>
```
添加Maven依赖
```maven
<dependency>
 <groupId>org.springframework.session</groupId>
  <artifactId>spring-session-data-redis</artifactId>
  <version>1.3.1.RELEASE</version>
</dependency>
<dependency>
 <groupId>biz.paluch.redis</groupId>
  <artifactId>lettuce</artifactId>
  <version>3.5.0.Final</version>
</dependency>
```
 ### 如何在Redis中查看Session数据？
* （1）Http Session数据在Redis中是以Hash结构存储的。
* （2）可以看到，还有一个key="spring:session:expirations:1431577740000"的数据，是以Set结构保存的。这个值记录了所有session数据应该被删除的时间（即最新的一个session数据过期的时间）。
