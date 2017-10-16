---
layout: post
title: Spring session集群共享
category: java
tags: [java]
---

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

## spring-session 介绍 
* 1.简介 
Spring Session是Spring的项目之一，(GitHub地址)[https://github.com/spring-projects/spring-session]
Spring Session提供了一套创建和管理Servlet HttpSession的方案。Spring Session提供了集群Session（Clustered Sessions）功能，默认采用外置的Redis来存储Session数据，以此来解决Session共享的问题。
session一直都是我们做集群时需要解决的一个难题，过去我们可以从serlvet容器上解决，比如开源servlet容器-tomcat提供的tomcat-redis-session-manager、memcached-session-manager。 或者通过nginx之类的负载均衡做ip_hash，路由到特定的服务器上.但是这两种办法都存在弊端。
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
* 注解 SpringSessionConfig.java
```java
package com.dwring.config;

import java.util.HashSet;
import java.util.Set;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisClusterConfiguration;
import org.springframework.data.redis.connection.RedisNode;
import org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory;
import org.springframework.session.data.redis.config.ConfigureRedisAction;
import org.springframework.session.data.redis.config.annotation.web.http.EnableRedisHttpSession;

/**
 * @ClassName SpringSessionConfig
 * @Description TODO
 * @author zhanghaichang
 * @date: 2017年9月28日 下午12:45:47
 *
 */
@Configuration
@EnableRedisHttpSession(maxInactiveIntervalInSeconds = 1800)
public class SpringSessionConfig {
	
	 /**
     * The Spring configuration is responsible for creating a Servlet Filter that
     * replaces the HttpSession implementation with an implementation backed by Spring Session.
     */
    @Bean
    public LettuceConnectionFactory connectionFactory() {
        LettuceConnectionFactory connectionFactory = new LettuceConnectionFactory(redisClusterConfiguration());
        return connectionFactory;
    }
    
    /**
     * 让Spring Session不再执行config命令
     * @return
     */
    @Bean
    public static ConfigureRedisAction configureRedisAction() {
        return ConfigureRedisAction.NO_OP;
    }

	/**
	 * redis集群配置
	 * 
	 * 配置redis集群的结点及其它一些属性
	 * 
	 * @return
	 */
	private RedisClusterConfiguration redisClusterConfiguration() {
		RedisClusterConfiguration redisClusterConfig = new RedisClusterConfiguration();
		redisClusterConfig.setClusterNodes(getClusterNodes());
		redisClusterConfig.setMaxRedirects(3);
		return redisClusterConfig;

	}


	/**
	 * redis集群节点IP和端口的添加
	 * 
	 * 节点：RedisNode redisNode = new RedisNode("127.0.0.1",6379);
	 * 
	 * @return
	 */
	private Set<RedisNode> getClusterNodes() {
		// 添加redis集群的节点
		Set<RedisNode> clusterNodes = new HashSet<RedisNode>();
		clusterNodes.add(new RedisNode("192.168.44.128", 6379));
		clusterNodes.add(new RedisNode("192.168.44.128", 7000));
		clusterNodes.add(new RedisNode("192.168.44.128", 7001));
		clusterNodes.add(new RedisNode("192.168.44.128", 7002));
		clusterNodes.add(new RedisNode("192.168.44.128", 7003));
		clusterNodes.add(new RedisNode("192.168.44.128", 7004));
		clusterNodes.add(new RedisNode("192.168.44.128", 7005));
		return clusterNodes;
	}
}
```
```xml
<context:annotation-config/>    
/**  初始化一切spring-session准备，且把springSessionFilter放入IOC          **/
<bean class="org.springframework.session.data.redis.config.annotation.web.http.RedisHttpSessionConfiguration"/>   
/** 这是存储容器的链接池 **/ 
<bean class="org.springframework.data.redis.connection.lettuce.LettuceConnectionFactory">
    <property name="hostName" value="127.0.0.1"/>
    <property name="port" value="6379"/>
    <property name="password" value="xxxxxxx"></property>
</bean>
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
## Spring Session原理
* 1.前面集成spring-sesion的第二步中，编写了一个配置类RedisHttpSessionConfig，它包含注解@EnableRedisHttpSession，并通过@Bean注解注册了一个RedisConnectionFactory到Spring容器中。
而@EnableRedisHttpSession注解通过Import，引入了RedisHttpSessionConfiguration配置类。该配置类通过@Bean注解，向Spring容器中注册了一个SessionRepositoryFilter（SessionRepositoryFilter的依赖关系：SessionRepositoryFilter --> SessionRepository --> RedisTemplate --> RedisConnectionFactory）
```java
package org.springframework.session.data.redis.config.annotation.web.http;  
  
@Configuration  
@EnableScheduling  
public class RedisHttpSessionConfiguration implements ImportAware, BeanClassLoaderAware {  
    //......  
      
    @Bean  
    public RedisTemplate<String,ExpiringSession> sessionRedisTemplate(RedisConnectionFactory connectionFactory) {  
        //......  
        return template;  
    }  
      
    @Bean  
    public RedisOperationsSessionRepository sessionRepository(RedisTemplate<String, ExpiringSession> sessionRedisTemplate) {  
        //......  
        return sessionRepository;  
    }  
      
    @Bean  
    public <S extends ExpiringSession> SessionRepositoryFilter<? extends ExpiringSession> springSessionRepositoryFilter(SessionRepository<S> sessionRepository, ServletContext servletContext) {  
        //......  
        return sessionRepositoryFilter;  
    }  
}  
```
* 2.集成spring-sesion的第四步中，我们编写了一个SpringSessionInitializer 类，它继承自AbstractHttpSessionApplicationInitializer。该类不需要重载或实现任何方法，它的作用是在Servlet容器初始化时，从Spring容器中获取一个默认名叫sessionRepositoryFilter的过滤器类（之前没有注册的话这里找不到会报错），并添加到Servlet过滤器链中。
```java
package org.springframework.session.web.context;  
  
/** 
 * Registers the {@link DelegatingFilterProxy} to use the 
 * springSessionRepositoryFilter before any other registered {@link Filter}.  
 * 
 * ...... 
 */  
@Order(100)  
public abstract class AbstractHttpSessionApplicationInitializer implements WebApplicationInitializer {  
  
    private static final String SERVLET_CONTEXT_PREFIX = "org.springframework.web.servlet.FrameworkServlet.CONTEXT.";  
  
    public static final String DEFAULT_FILTER_NAME = "springSessionRepositoryFilter";  
  
    //......  
  
    public void onStartup(ServletContext servletContext)  
            throws ServletException {  
        beforeSessionRepositoryFilter(servletContext);  
        if(configurationClasses != null) {  
            AnnotationConfigWebApplicationContext rootAppContext = new AnnotationConfigWebApplicationContext();  
            rootAppContext.register(configurationClasses);  
            servletContext.addListener(new ContextLoaderListener(rootAppContext));  
        }  
        insertSessionRepositoryFilter(servletContext);//注册一个SessionRepositoryFilter  
        afterSessionRepositoryFilter(servletContext);  
    }  
  
    /** 
     * Registers the springSessionRepositoryFilter 
     * @param servletContext the {@link ServletContext} 
     */  
    private void insertSessionRepositoryFilter(ServletContext servletContext) {  
        String filterName = DEFAULT_FILTER_NAME;//默认名字是springSessionRepositoryFilter  
        DelegatingFilterProxy springSessionRepositoryFilter = new DelegatingFilterProxy(filterName);//该Filter代理会在初始化时从Spring容器中查找springSessionRepositoryFilter，之后实际会使用SessionRepositoryFilter进行doFilter操作         
        String contextAttribute = getWebApplicationContextAttribute();  
        if(contextAttribute != null) {  
            springSessionRepositoryFilter.setContextAttribute(contextAttribute);  
        }  
        registerFilter(servletContext, true, filterName, springSessionRepositoryFilter);  
    }  
}  
```
SessionRepositoryFilter是一个优先级最高的javax.servlet.Filter，它使用了一个SessionRepositoryRequestWrapper类接管了Http Session的创建和管理工作。注意下面给出的是简化过的示例代码，与spring-session项目的源代码有所差异。
```java
@Order(SessionRepositoryFilter.DEFAULT_ORDER)  
public class SessionRepositoryFilter implements Filter {  
  
        public doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {  
                HttpServletRequest httpRequest = (HttpServletRequest) request;  
                SessionRepositoryRequestWrapper customRequest =  
                        new SessionRepositoryRequestWrapper(httpRequest);  
  
                chain.doFilter(customRequest, response, chain);  
        }  
  
        // ...  
}  
public class SessionRepositoryRequestWrapper extends HttpServletRequestWrapper {  
  
        public SessionRepositoryRequestWrapper(HttpServletRequest original) {  
                super(original);  
        }  
  
        public HttpSession getSession() {  
                return getSession(true);  
        }  
  
        public HttpSession getSession(boolean createNew) {  
                // create an HttpSession implementation from Spring Session  
        }  
  
        // ... other methods delegate to the original HttpServletRequest ...  
}  
```
* 3.好了，剩下的问题就是，如何在Servlet容器启动时，加载下面两个类。幸运的是，这两个类由于都实现了WebApplicationInitializer接口，会被自动加载。
WebInitializer，负责加载配置类。它继承自AbstractAnnotationConfigDispatcherServletInitializer，实现了WebApplicationInitializer接口
SpringSessionInitializer，负责添加sessionRepositoryFilter的过滤器类。它继承自AbstractHttpSessionApplicationInitializer，实现了WebApplicationInitializer接口

在Servlet3.0规范中，Servlet容器启动时会自动扫描javax.servlet.ServletContainerInitializer的实现类，在实现类中我们可以定制需要加载的类。在spring-web项目中，有一个ServletContainerInitializer实现类SpringServletContainerInitializer，它通过注解@HandlesTypes(WebApplicationInitializer.class)，让Servlet容器在启动该类时，会自动寻找所有的WebApplicationInitializer实现类。

```java
package org.springframework.web;  
  
@HandlesTypes(WebApplicationInitializer.class)  
public class SpringServletContainerInitializer implements ServletContainerInitializer {  
  
    /** 
     * Delegate the {@code ServletContext} to any {@link WebApplicationInitializer} 
     * implementations present on the application classpath. 
     * 
     * <p>Because this class declares @{@code HandlesTypes(WebApplicationInitializer.class)}, 
     * Servlet 3.0+ containers will automatically scan the classpath for implementations 
     * of Spring's {@code WebApplicationInitializer} interface and provide the set of all 
     * such types to the {@code webAppInitializerClasses} parameter of this method. 
     * 
     * <p>If no {@code WebApplicationInitializer} implementations are found on the 
     * classpath, this method is effectively a no-op. An INFO-level log message will be 
     * issued notifying the user that the {@code ServletContainerInitializer} has indeed 
     * been invoked but that no {@code WebApplicationInitializer} implementations were 
     * found. 
     * 
     * <p>Assuming that one or more {@code WebApplicationInitializer} types are detected, 
     * they will be instantiated (and <em>sorted</em> if the @{@link 
     * org.springframework.core.annotation.Order @Order} annotation is present or 
     * the {@link org.springframework.core.Ordered Ordered} interface has been 
     * implemented). Then the {@link WebApplicationInitializer#onStartup(ServletContext)} 
     * method will be invoked on each instance, delegating the {@code ServletContext} such 
     * that each instance may register and configure servlets such as Spring's 
     * {@code DispatcherServlet}, listeners such as Spring's {@code ContextLoaderListener}, 
     * or any other Servlet API componentry such as filters. 
     * 
     * @param webAppInitializerClasses all implementations of 
     * {@link WebApplicationInitializer} found on the application classpath 
     * @param servletContext the servlet context to be initialized 
     * @see WebApplicationInitializer#onStartup(ServletContext) 
     * @see AnnotationAwareOrderComparator 
     */  
    @Override  
    public void onStartup(Set<Class<?>> webAppInitializerClasses, ServletContext servletContext)  
            throws ServletException {  
        //......  
    }  
  
}  
```
 ## 如何在Redis中查看Session数据？
* （1）Http Session数据在Redis中是以Hash结构存储的。
* （2）可以看到，还有一个key="spring:session:expirations:1431577740000"的数据，是以Set结构保存的。这个值记录了所有session数据应该被删除的时间（即最新的一个session数据过期的时间）。
