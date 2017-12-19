---
layout: post
title: Java Spring的 JavaConfig 注解
category: java
tags: [java]
---
传统spring一般都是基于xml配置的，不过后来新增了许多JavaConfig的注解。特别是springboot，基本都是清一色的java config，不了解一下，还真是不适应。这里备注一下。

![](http://static.codeceo.com/images/2015/01/java-programmer.png "java-programmer")

## @RestController

spring4为了更方便的支持restfull应用的开发，新增了RestController的注解，比Controller注解多的功能就是给底下的RequestMapping方法默认都加上ResponseBody注解，省得自己再去每个去添加该注解。

## @Configuration

这个标注该类是spring的配置类，本身自带Component注解

## @ImportResource

### 对应的xml

```java
<import resource="applicationContext-ehcache.xml"/>
```

### 存在的必要性

这个是兼容传统xml配置的，毕竟JavaConfig还不是万能的，比如 [JavaConfig不能很好地支持aop:advisor和tx:advice](http://stackoverflow.com/questions/14068525/javaconfig-replacing-aopadvisor-and-txadvice) ， [Introduce @EnableAspectJAutoProxy (equivalent to aop:aspectj-autoproxy)](https://jira.spring.io/browse/SPR-8138) ， [Introduce @Configuration-based equivalent to aop:config XML element](https://jira.spring.io/browse/SPR-8148)

## @ComponentScan

对应的xml

```java
<context:component-scan base-package="com.xixicat.app"/>
```

该配置自动包含了如下配置的功能：

```java
<context:annotation-config/>
```

就是向Spring容器注册AutowiredAnnotationBeanPostProcessor( 使用@Autowired必须注册 )、CommonAnnotationBeanPostProcessor( 使用@Resource 、@PostConstruct、@PreDestroy等必须注册 )、PersistenceAnnotationBeanPostProcessor( 使用@PersistenceContext必须注册 ) 以及RequiredAnnotationBeanPostProcessor( 使用@Required必须注册 )这4个BeanPostProcessor。

值得注意的是 [Spring3.1RC2版本](https://jira.spring.io/browse/SPR-8808) 是不允许注解Configuration的类在ComponentScan指定的包范围内的，否则会报错。


## @Bean

对应的xml如下：

```java
<bean id="objectMapper" class="org.codehaus.jackson.map.ObjectMapper" />
```

## @EnableWebMvc

对应的xml如下：

```java
<mvc:annotation-driven />
```

该配置自动注册DefaultAnnotationHandlerMapping( 来注册handler method和request的mapping关系 )与AnnotationMethodHandlerAdapter( 在实际调用handler method前对其参数进行处理 )两个bean，以支持@Controller注解的使用。

主要的作用如下：

*   可配置的ConversionService(方便进行自定义类型转换)
*   支持用@NumberFormat格式化数字类型字段
*   支持用@DateTimeFormat格式化Date,Calendar以及Joda Time字段( 如果classpath有Joda Time的话 )
*   支持@Valid的参数校验( 如果JSR-303相关provider有在classpath的话 )
*   支持@RequestBody/@ResponseBody注解的XML读写( 如果JAXB在classpath的话 )
*   支持@RequestBody/@ResponseBody注解的JSON读写( 如果<span class="wp_keywordlink">[Jackson](http://www.codeceo.com/article/java-json-jackson.html "Jackson")</span>在classpath的话 )

## @ContextConfiguration

主要在junit测试时指定java config

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration({
    "classpath*:spring/*.xml",
    "classpath:applicationContext.xml",
    "classpath:applicationContext-rabbitmq.xml",
    "classpath:applicationContext-mail.xml",
    "classpath:applicationContext-medis.xml",
    "classpath:applicationContext-mybatis.xml"})
@TransactionConfiguration(transactionManager = "mybatisTransactionManager", defaultRollback = false)
public class AppBaseTest {
   //......
}
```

## @ResponseStatus

主要是rest开发用，注解返回的http返回码，具体值看org.springframework.http.HttpStatus枚举。一般post方法返回HttpStatus.CREATED，DELETE和PUT方法返回HttpStatus.OK。还可以配置异常处理，见@ExceptionHandler和@ControllerAdvice

## @ExceptionHandler

主要用来处理指定的异常，返回返回指定的<span class="wp_keywordlink">[HTTP状态码](http://www.codeceo.com/article/http-code-learn.html "HTTP状态码")</span>，省得每个controller的方法自己去try catch。一般可以为每个应用定义一个异常基类，然后再定义业务异常，这样这里就可以统一捕获业务异常。

```java
    @ExceptionHandler(BizException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public @ResponseBody
    ReturnMessage bizExceptionHandler(Exception ex) {
        logger.error(ex.getMessage(),ex);
        return new ReturnMessage(HttpStatus.BAD_REQUEST.value(),ex.getMessage());
    }
```

不过值得注意的是这种方法仅限于controller的方法调用链产生的异常，如果在spring里头还使用了定时任务啥的，该注解是不会拦截到的。

## @ControllerAdvice

配合@ExceptionHandler使用的，用来拦截controller的方法。

```java
@ControllerAdvice
public class ErrorController {

    private static final Logger logger = LoggerFactory.getLogger(ErrorController.class);

    @ExceptionHandler(BizException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public @ResponseBody
    ReturnMessage bizExceptionHandler(Exception ex) {
        logger.error(ex.getMessage(),ex);
        return new ReturnMessage(HttpStatus.BAD_REQUEST.value(),ex.getMessage());
    }

    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public @ResponseBody
    ReturnMessage serverExceptionHandler(Exception ex) {
        logger.error(ex.getMessage(),ex);
        return new ReturnMessage(HttpStatus.INTERNAL_SERVER_ERROR.value(),ex.getMessage());
    }
}
```
