---
layout: post
title:  Spring Boot启动过程及回调接口汇总
category: java
tags: [java]
---

相关代码在： [https://github.com/chanjarster/spring-boot-all-callbacks](https://github.com/chanjarster/spring-boot-all-callbacks)

注：本文基于[spring-boot 1.4.1.RELEASE](https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE), [spring 4.3.3.RELEASE](https://github.com/spring-projects/spring-framework/tree/v4.3.3.RELEASE)撰写。

## 启动顺序

Spring boot的启动代码一般是这样的：

```
@SpringBootApplication
public class SampleApplication {
  public static void main(String[] args) throws Exception {
    SpringApplication.run(SampleApplication.class, args);
  }
}
```

### 初始化SpringApplication

1.  `SpringApplication#run(Object source, String... args)`[#L1174](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L1174)

2.  [SpringApplication#L1186](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L1186) -> `SpringApplication(sources)`[#L236](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L236)

    1.  `SpringApplication#initialize(Object[] sources)`[#L256](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L256) [javadoc](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/SpringApplication.html)

        1.  [SpringApplication#L257](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L257) 添加source（复数），`SpringApplication`使用source来构建Bean。一般来说在`run`的时候都会把[@SpringBootApplication](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/autoconfigure/SpringBootApplication.html)标记的类(本例中是SampleApplication)放到`sources`参数里，然后由这个类出发找到Bean的定义。

        2.  [SpringApplication#L261](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L261) 初始化[ApplicationContextInitializer](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationContextInitializer.html)列表（见附录）

        3.  [SpringApplication#L263](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L263) 初始化[ApplicationListener](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationListener.html)列表（见附录）

3.  [SpringApplication#L1186](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L1186) -> `SpringApplication#run(args)`[#L297](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L297)，进入运行阶段

### 推送ApplicationStartedEvent

`SpringApplication#run(args)`[#L297](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L297)

1.  [SpringApplication#L303](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L303) 初始化[SpringApplicationRunListeners](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplicationRunListeners.java) ([SpringApplicationRunListener](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/SpringApplicationRunListener.html)的集合)。它内部只包含[EventPublishingRunListener](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/context/event/EventPublishingRunListener.html)。

2.  [SpringApplication#L304](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L304) 推送[ApplicationStartedEvent](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/context/event/ApplicationStartedEvent.html)给所有的[ApplicationListener](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationListener.html)（见附录）。 下面是关心此事件的listener：

    1.  [LiquibaseServiceLocatorApplicationListener](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/liquibase/LiquibaseServiceLocatorApplicationListener.html)

    2.  [LoggingApplicationListener](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/logging/LoggingApplicationListener.html)（见附录）

### 准备Environment

`SpringApplication#run(args)`[#L297](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L297)->[#L308](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L308)->`prepareEnvironment(...)`[#L331](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L331)准备[ConfigurableEnvironment](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/env/ConfigurableEnvironment.html)。

1.  [SpringApplication#L335](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L335) 创建[StandardEnvironment](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/env/StandardEnvironment.html)（见附录）。

2.  [SpringApplication#L336](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L336) 配置[StandardEnvironment](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/env/StandardEnvironment.html)，将命令行和默认参数整吧整吧，添加到[MutablePropertySources](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/env/MutablePropertySources.html)。

3.  [SpringApplication#L337](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L337) 推送[ApplicationEnvironmentPreparedEvent](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/context/event/ApplicationEnvironmentPreparedEvent.html)给所有的[ApplicationListener](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationListener.html)（见附录）。下面是关心此事件的listener:

    1.  [BackgroundPreinitializer](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/autoconfigure/BackgroundPreinitializer.html)

    2.  [FileEncodingApplicationListener](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/context/FileEncodingApplicationListener.html)

    3.  [AnsiOutputApplicationListener](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/context/config/AnsiOutputApplicationListener.html)

    4.  [ConfigFileApplicationListener](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/context/config/ConfigFileApplicationListener.html)（见附录）

    5.  [DelegatingApplicationListener](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/context/config/DelegatingApplicationListener.html)

    6.  [ClasspathLoggingApplicationListener](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/logging/ClasspathLoggingApplicationListener.html)

    7.  [LoggingApplicationListener](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/logging/LoggingApplicationListener.html)

    8.  [ApplicationPidFileWriter](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/system/ApplicationPidFileWriter.html)

可以参考[官方文档](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#boot-features-external-config)了解[StandardEnvironment](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/env/StandardEnvironment.html)构建好之后，其[MutablePropertySources](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/env/MutablePropertySources.html)内部到底有些啥东东。

### 创建及准备ApplicationContext

`SpringApplication#run(args)`[#L297](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L297)

1.  [SpringApplication#L311](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L311)->`SpringApplication#createApplicationContext()`[#L583](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L583)创建[ApplicationContext](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationContext.html)。可以看到实际上创建的是[AnnotationConfigApplicationContext](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/AnnotationConfigApplicationContext.html)或[AnnotationConfigEmbeddedWebApplicationContext](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/context/embedded/AnnotationConfigEmbeddedWebApplicationContext.html)。

    1.  在构造[AnnotationConfigApplicationContext](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/AnnotationConfigApplicationContext.html)的时候，间接注册了一个[BeanDefinitionRegistryPostProcessor](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/support/BeanDefinitionRegistryPostProcessor.html)的Bean：[ConfigurationClassPostProcessor](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/ConfigurationClassPostProcessor.html)。经由[AnnotatedBeanDefinitionReader](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/AnnotatedBeanDefinitionReader.html)[构造函数](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/AnnotatedBeanDefinitionReader.java#L83)->[AnnotationConfigUtils.registerAnnotationConfigProcessors](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/AnnotationConfigUtils.java#L160)。

2.  [SpringApplication#L313](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L313)->`SpringApplication#prepareContext(...)`[#L344](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L344)准备[ApplicationContext](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationContext.html)

    1.  [SpringApplication#L347](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L347)->`context.setEnvironment(environment)`，把之前准备好的[Environment](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/env/Environment.html)塞给[ApplicationContext](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationContext.html)

    2.  [SpringApplication#L348](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L348)->`postProcessApplicationContext(context)`[#L605](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L605)，给[ApplicationContext](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationContext.html)设置了一些其他东西

    3.  [SpringApplication#L349](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L349)->`applyInitializers(context)`[#L630](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L630)，调用之前准备好的[ApplicationContextInitializer](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationContextInitializer.html)

    4.  [SpringApplication#L350](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L350)->`listeners.contextPrepared(context)`->[EventPublishingRunListener.contextPrepared](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/context/event/EventPublishingRunListener.java#L73)，但实际上啥都没做。

    5.  [SpringApplication#L366](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L366)->`load`[#L687](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L687)，负责将source(复数)里所定义的Bean加载到[ApplicationContext](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationContext.html)里，在本例中就是SampleApplication，这些source是在**初始化SpringApplication**阶段获得的。

    6.  [SpringApplication#L367](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L367)->`listeners.contextLoaded(context)`->[EventPublishingRunListener.contextLoaded](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/context/event/EventPublishingRunListener.java#L78)。

        1.  将[SpringApplication](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/SpringApplication.html)自己拥有的[ApplicationListener](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationListener.html)加入到[ApplicationContext](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationContext.html)

        2.  发送[ApplicationPreparedEvent](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/context/event/ApplicationPreparedEvent.html)。目前已知关心这个事件的有[ConfigFileApplicationListener](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/context/config/ConfigFileApplicationListener.html)、[LoggingApplicationListener](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/logging/LoggingApplicationListener.html)、[ApplicationPidFileWriter](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/system/ApplicationPidFileWriter.html)

要注意的是在这个阶段，[ApplicationContext](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationContext.html)里只有SampleApplication，SampleApplication是Bean的加载工作的起点。

### 刷新ApplicationContext

根据前面所讲，这里的[ApplicationContext](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationContext.html)实际上是[GenericApplicationContext](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/support/GenericApplicationContext.html)
->[AnnotationConfigApplicationContext](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/AnnotationConfigApplicationContext.html)或者[AnnotationConfigEmbeddedWebApplicationContext](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/context/embedded/AnnotationConfigEmbeddedWebApplicationContext.html)

`SpringApplication#run(args)`[#L297](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L297)
->[#L315](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L315)->`SpringApplication#refreshContext(context)`[#L370](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L370)
->[#L371](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L371)->`SpringApplication#refresh(context)`[#L759](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L759)
->[#L761](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L761)->`AbstractApplicationContext#refresh`[AbstractApplicationContext#L507](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L507)

1.  [AbstractApplicationContext#L510](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L510)->`AbstractApplicationContext#prepareRefresh()`[#L575](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L575)，做了一些初始化工作，比如设置了当前Context的状态，初始化propertySource（其实啥都没干），检查required的property是否都已在Environment中（其实并没有required的property可供检查）等。

2.  [AbstractApplicationContext#L513](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L513)->`obtainFreshBeanFactory()`[#L611](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L611)，获得[BeanFactory](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/BeanFactory.html)，实际上这里获得的是[DefaultListableBeanFactory](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/support/DefaultListableBeanFactory.html)

3.  [AbstractApplicationContext#L516](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L516)->`prepareBeanFactory(beanFactory)`[#L625](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L625)准备[BeanFactory](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/BeanFactory.html)

    1.  给beanFactory设置了ClassLoader

    2.  给beanFactory设置了[SpEL解析器](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/expression/StandardBeanExpressionResolver.html)

    3.  给beanFactory设置了[PropertyEditorRegistrar](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/PropertyEditorRegistrar.html)

    4.  给beanFactory添加了[ApplicationContextAwareProcessor](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/support/ApplicationContextAwareProcessor.java)（[BeanPostProcessor](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/config/BeanPostProcessor.html)的实现类），需要注意的是它是第一个被添加到[BeanFactory](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/BeanFactory.html)的[BeanPostProcessor](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/config/BeanPostProcessor.html)

    5.  给beanFactory设置忽略解析以下类的依赖：[ResourceLoaderAware](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ResourceLoaderAware.html)、[ApplicationEventPublisherAware](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationEventPublisherAware.html)、[MessageSourceAware](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/MessageSourceAware.html)、[ApplicationContextAware](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationContextAware.html)、[EnvironmentAware](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/EnvironmentAware.html)。原因是注入这些回调接口本身没有什么意义。

    6.  给beanFactory添加了以下类的依赖解析：[BeanFactory](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/BeanFactory.html)、[ResourceLoader](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/io/ResourceLoader.html)、[ApplicationEventPublisher](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationEventPublisher.html)、[ApplicationContext](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationContext.html)

    7.  给beanFactory添加[LoadTimeWeaverAwareProcessor](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/weaving/LoadTimeWeaverAwareProcessor.html)用来处理[LoadTimeWeaverAware](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/weaving/LoadTimeWeaverAware.html)的回调，在和AspectJ集成的时候会用到

    8.  把`getEnvironment()`作为Bean添加到beanFactory中，Bean Name: `environment`

    9.  把`getEnvironment().getSystemProperties()`作为Bean添加到beanFactory中，Bean Name: `systemProperties`

    10.  把`getEnvironment().getSystemEnvironment()`作为Bean添加到beanFactory中，Bean Name: `systemEnvironment`

4.  [AbstractApplicationContext#L520](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L520)->`postProcessBeanFactory(beanFactory)`，后置处理[BeanFactory](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/BeanFactory.html)，实际啥都没做

5.  [AbstractApplicationContext#L523](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L523)->`invokeBeanFactoryPostProcessors(beanFactory)`，利用[BeanFactoryPostProcessor](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/config/BeanFactoryPostProcessor.html)，对beanFactory做后置处理。调用此方法时有四个[BeanFactoryPostProcessor](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/config/BeanFactoryPostProcessor.html)：

    1.  [SharedMetadataReaderFactoryContextInitializer](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/SharedMetadataReaderFactoryContextInitializer.java)的内部类[CachingMetadataReaderFactoryPostProcessor](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/SharedMetadataReaderFactoryContextInitializer.java#L66)，是在 **创建及准备ApplicationContext 2.3** 时添加的：[#L57](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/SharedMetadataReaderFactoryContextInitializer.java#L57)

    2.  [ConfigurationWarningsApplicationContextInitializer](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/context/ConfigurationWarningsApplicationContextInitializer.html)的内部类[ConfigurationWarningsPostProcessor](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/context/ConfigurationWarningsApplicationContextInitializer.java#L75)，是在 **创建及准备ApplicationContext 2.3** 时添加的：[#L60](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/context/ConfigurationWarningsApplicationContextInitializer.java#L60)

    3.  [ConfigFileApplicationListener](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/context/config/ConfigFileApplicationListener.html)的内部类[PropertySourceOrderingPostProcessor](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/context/config/ConfigFileApplicationListener.java#L285)，是在 **创建及准备ApplicationContext 2.6** 时添加的：[#L158](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/context/config/ConfigFileApplicationListener.java#L158)->[#L199](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/context/config/ConfigFileApplicationListener.java#L199)->[#L244](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/context/config/ConfigFileApplicationListener.java#L244)

    4.  [ConfigurationClassPostProcessor](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/ConfigurationClassPostProcessor.html)，负责读取[BeanDefinition](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/config/BeanDefinition.html)是在 **创建及准备ApplicationContext 1.1** 时添加的

6.  [AbstractApplicationContext#L526](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L526)->`registerBeanPostProcessors(beanFactory)`，注册[BeanPostProcessor](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/config/BeanPostProcessor.html)

7.  [AbstractApplicationContext#L529](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L529)->`initMessageSource()`[#L704](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L704)，初始化[MessageSource](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/MessageSource.html)，不过其实此时的[MessageSource](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/MessageSource.html)是个Noop对象。

8.  [AbstractApplicationContext#L532](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L532)->`initApplicationEventMulticaster()`[#L739](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L739)，初始化[ApplicationEventMulticaster](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/event/SimpleApplicationEventMulticaster.html)。

9.  [AbstractApplicationContext#L535](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L535)->`onRefresh()`[#L793](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L793)，这个方法啥都没做

10.  [AbstractApplicationContext#L538](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L538)->`registerListeners()`[#L801](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L801)，把自己的[ApplicationListener](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationListener.html)注册到[ApplicationEventMulticaster](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/event/SimpleApplicationEventMulticaster.html)里，并且将之前因为没有[ApplicationEventMulticaster](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/event/SimpleApplicationEventMulticaster.html)而无法发出的[ApplicationEvent](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationEvent.html)发送出去。

11.  [AbstractApplicationContext#L541](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L541)->`finishBeanFactoryInitialization`[#L828](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L828)。注意[#L861](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L861)，在这一步的时候才会实例化所有non-lazy-init bean，这里说的实例化不只是new而已，注入、[BeanPostProcessor](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/config/BeanPostProcessor.html)都会执行。

12.  [AbstractApplicationContext#L544](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L544)->`finishRefresh()`[#L869](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L869)。

    1.  在[#L877](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/support/AbstractApplicationContext.java#L877)发送了[ContextRefreshedEvent](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/event/ContextRefreshedEvent.html)

### 调用 ApplicationRunner 和 CommandLineRunner

`SpringApplication#run(args)`[#L297](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L297)
->`afterRefresh(context, applicationArguments)`[#L316](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L316)
->`callRunners(context, args)`[#L771](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L771)
->[#L774](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L774) 先后调用了当前[ApplicationContext](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationContext.html)中的[ApplicationRunner](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/ApplicationRunner.html)和[CommandLineRunner](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/CommandLineRunner.html)。关于它们的相关文档可以看[这里](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#boot-features-command-line-runner)。

需要注意的是，此时的[ApplicationContext](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationContext.html)已经刷新完毕了，该有的Bean都已经有了。

### 推送ApplicationReadyEvent or ApplicationFailedEvent

`SpringApplication#run(args)`[#L297](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L297)->`listeners.finished(context, null)`[#L317](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/SpringApplication.java#L317)
间接地调用了`EventPublishingRunListener#getFinishedEvent`[EventPublishingRunListener#L96](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/context/event/EventPublishingRunListener.java#L96)，发送了[ApplicationReadyEvent](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/context/event/ApplicationReadyEvent.html)或[ApplicationFailedEvent](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/context/event/ApplicationFailedEvent.html)

## 回调接口

### ApplicationContextInitializer

[javadoc](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationContextInitializer.html) [相关文档](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#howto-customize-the-environment-or-application-context)

加载方式：读取`classpath*:META-INF/spring.factories`中key等于`org.springframework.context.ApplicationContextInitializer`的property列出的类

排序方式：[AnnotationAwareOrderComparator](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/annotation/AnnotationAwareOrderComparator.html)

已知清单1：spring-boot-1.4.1.RELEASE.jar!/META-INF/spring.factories

1.  [ConfigurationWarningsApplicationContextInitializer](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/context/ConfigurationWarningsApplicationContextInitializer.html)（优先级：0）

2.  [ContextIdApplicationContextInitializer](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/context/ContextIdApplicationContextInitializer.html)（优先级：Ordered.LOWEST_PRECEDENCE - 10）

3.  [DelegatingApplicationContextInitializer](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/context/config/DelegatingApplicationContextInitializer.html)（优先级：无=Ordered.LOWEST_PRECEDENCE）

4.  [ServerPortInfoApplicationContextInitializer](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/context/web/ServerPortInfoApplicationContextInitializer.html)（优先级：无=Ordered.LOWEST_PRECEDENCE）

已知清单2：spring-boot-autoconfigure-1.4.1.RELEASE.jar!/META-INF/spring.factories

1.  [SharedMetadataReaderFactoryContextInitializer](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/SharedMetadataReaderFactoryContextInitializer.java)（优先级：无=Ordered.LOWEST_PRECEDENCE）

2.  [AutoConfigurationReportLoggingInitializer](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/autoconfigure/logging/AutoConfigurationReportLoggingInitializer.html)（优先级：无=Ordered.LOWEST_PRECEDENCE）

### ApplicationListener

[javadoc](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationListener.html) [相关文档](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#howto-customize-the-environment-or-application-context)

加载方式：读取`classpath*:META-INF/spring.factories`中key等于`org.springframework.context.ApplicationListener`的property列出的类

排序方式：[AnnotationAwareOrderComparator](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/annotation/AnnotationAwareOrderComparator.html)

已知清单1：spring-boot-1.4.1.RELEASE.jar!/META-INF/spring.factories中定义的

1.  [ClearCachesApplicationListener](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot/src/main/java/org/springframework/boot/ClearCachesApplicationListener.java)（优先级：无=Ordered.LOWEST_PRECEDENCE）

2.  [ParentContextCloserApplicationListener](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/builder/ParentContextCloserApplicationListener.html)（优先级：Ordered.LOWEST_PRECEDENCE - 10）

3.  [FileEncodingApplicationListener](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/context/FileEncodingApplicationListener.html)（优先级：Ordered.LOWEST_PRECEDENCE）

4.  [AnsiOutputApplicationListener](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/context/config/AnsiOutputApplicationListener.html)（优先级：ConfigFileApplicationListener.DEFAULT_ORDER + 1）

5.  [ConfigFileApplicationListener](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/context/config/ConfigFileApplicationListener.html)（优先级：Ordered.HIGHEST_PRECEDENCE + 10）

6.  [DelegatingApplicationListener](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/context/config/DelegatingApplicationListener.html)（优先级：0)

7.  [LiquibaseServiceLocatorApplicationListener](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/liquibase/LiquibaseServiceLocatorApplicationListener.html)（优先级：无=Ordered.LOWEST_PRECEDENCE）

8.  [ClasspathLoggingApplicationListener](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/logging/ClasspathLoggingApplicationListener.html)（优先级：LoggingApplicationListener的优先级 + 1）

9.  [LoggingApplicationListener](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/logging/LoggingApplicationListener.html)（优先级：Ordered.HIGHEST_PRECEDENCE + 20）

已知清单2：spring-boot-autoconfigure-1.4.1.RELEASE.jar!/META-INF/spring.factories中定义的

1.  [BackgroundPreinitializer](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/autoconfigure/BackgroundPreinitializer.html)

### SpringApplicationRunListener

[javadoc](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/SpringApplicationRunListener.html)

加载方式：读取`classpath*:META-INF/spring.factories`中key等于`org.springframework.boot.SpringApplicationRunListener`的property列出的类

排序方式：[AnnotationAwareOrderComparator](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/annotation/AnnotationAwareOrderComparator.html)

已知清单：spring-boot-1.4.1.RELEASE.jar!/META-INF/spring.factories定义的

1.  org.springframework.boot.context.event.EventPublishingRunListener（优先级：0）

### EnvironmentPostProcessor

[EnvironmentPostProcessor](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/env/EnvironmentPostProcessor.html)可以用来自定义[StandardEnvironment](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/env/StandardEnvironment.html)（[相关文档](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#howto-customize-the-environment-or-application-context)）。

加载方式：读取`classpath*:META-INF/spring.factories`中key等于`org.springframework.boot.env.EnvironmentPostProcessor`的property列出的类

排序方式：[AnnotationAwareOrderComparator](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/annotation/AnnotationAwareOrderComparator.html)

已知清单：spring-boot-1.4.1.RELEASE.jar!/META-INF/spring.factories定义的

1.  [CloudFoundryVcapEnvironmentPostProcessor](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/cloud/CloudFoundryVcapEnvironmentPostProcessor.html)（优先级：ConfigFileApplicationListener.DEFAULT_ORDER - 1）

2.  [SpringApplicationJsonEnvironmentPostProcessor](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/env/SpringApplicationJsonEnvironmentPostProcessor.html)（优先级：Ordered.HIGHEST_PRECEDENCE + 5）

### BeanPostProcessor

[javadoc](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/config/BeanPostProcessor.html) [相关文档](http://docs.spring.io/spring/docs/4.3.3.RELEASE/spring-framework-reference/htmlsingle/#beans-factory-extension-bpp)

用来对Bean**实例**进行修改的勾子，根据Javadoc ApplicationContext会自动侦测到BeanPostProcessor Bean，然后将它们应用到后续创建的所有Bean上。

### BeanFactoryPostProcessor和BeanDefinitionRegistryPostProcessor

[相关文档](http://docs.spring.io/spring/docs/4.3.3.RELEASE/spring-framework-reference/htmlsingle/#beans-factory-extension-factory-postprocessors)

[PostProcessorRegistrationDelegate](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/support/PostProcessorRegistrationDelegate.java#L57)负责调用[BeanDefinitionRegistryPostProcessor](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/support/BeanDefinitionRegistryPostProcessor.html)和[BeanFactoryPostProcessor](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/config/BeanFactoryPostProcessor.html)。
[BeanDefinitionRegistryPostProcessor](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/support/BeanDefinitionRegistryPostProcessor.html)在[BeanFactoryPostProcessor](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/config/BeanFactoryPostProcessor.html)之前被调用。

### *Aware

*Aware是一类可以用来获得Spring对象的interface，这些interface都继承了[Aware](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/Aware.html)，已知的有：

*   [ApplicationEventPublisherAware](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationEventPublisherAware.html)

*   [NotificationPublisherAware](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/jmx/export/notification/NotificationPublisherAware.html)

*   [MessageSourceAware](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/MessageSourceAware.html)

*   [EnvironmentAware](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/EnvironmentAware.html)

*   [BeanFactoryAware](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/BeanFactoryAware.html)

*   [EmbeddedValueResolverAware](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/EmbeddedValueResolverAware.html)

*   [ResourceLoaderAware](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ResourceLoaderAware.html)

*   [ImportAware](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/ImportAware.html)

*   [LoadTimeWeaverAware](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/weaving/LoadTimeWeaverAware.html)

*   [BeanNameAware](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/BeanNameAware.html)

*   [BeanClassLoaderAware](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/BeanClassLoaderAware.html)

*   [ApplicationContextAware](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationContextAware.html)

## @Configuration 和 Auto-configuration

[@Configuration](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/Configuration.html)替代xml来定义[BeanDefinition](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/config/BeanDefinition.html)的一种手段。[Auto-configuration](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#using-boot-auto-configuration)也是定义[BeanDefinition](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/config/BeanDefinition.html)的一种手段。

这两者的相同之处有：

1.  都是使用[@Configuration](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/Configuration.html)注解的类，这些类里都可以定义[@Bean](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/Bean.html)、[@Import](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/Import.html)、[@ImportResource](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/ImportResource.html)。

2.  都可以使用[@Condition*](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#boot-features-condition-annotations)来根据情况选择是否加载

而不同之处有：

1.  加载方式不同：

    *   普通[@Configuration](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/Configuration.html)则是通过扫描package path加载的

    *   [Auto-configuration](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#using-boot-auto-configuration)的是通过读取`classpath*:META-INF/spring.factories`中key等于`org.springframework.boot.autoconfigure.EnableAutoConfiguration`的property列出的[@Configuration](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/Configuration.html)加载的

2.  加载顺序不同：普通[@Configuration](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/Configuration.html)的加载永远在[Auto-configuration](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#using-boot-auto-configuration)之前

3.  内部加载顺序可控上的不同：

    *   普通[@Configuration](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/Configuration.html)则无法控制加载顺序

    *   [Auto-configuration](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#using-boot-auto-configuration)可以使用[@AutoConfigureOrder](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/autoconfigure/AutoConfigureOrder.html)、[@AutoConfigureBefore](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/autoconfigure/AutoConfigureBefore.html)、[@AutoConfigureAfter](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/autoconfigure/AutoConfigureAfter.html)

以下情况下[Auto-configuration](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#using-boot-auto-configuration)会在普通[@Configuration](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/Configuration.html)前加载：

1.  [Auto-configuration](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#using-boot-auto-configuration)如果出现在最初的扫描路径里（@ComponentScan），就会被提前加载到，然后被当作普通的[@Configuration](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/Configuration.html)处理，这样[@AutoConfigureBefore](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/autoconfigure/AutoConfigureBefore.html)和[@AutoConfigureAfter](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/autoconfigure/AutoConfigureAfter.html)就没用了。参看例子代码里的InsideAutoConfiguration和InsideAutoConfiguration2。

2.  [Auto-configuration](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#using-boot-auto-configuration)如果提供[BeanPostProcessor](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/config/BeanPostProcessor.html)，那么它会被提前加载。参见例子代码里的BeanPostProcessorAutoConfiguration。

3.  [Auto-configuration](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#using-boot-auto-configuration)如果使用了[ImportBeanDefinitionRegistrar](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/ImportBeanDefinitionRegistrar.html)，那么[ImportBeanDefinitionRegistrar](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/ImportBeanDefinitionRegistrar.html)会被提前加载。参见例子代码里的ImportBeanDefinitionRegistrarAutoConfiguration。

4.  [Auto-configuration](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#using-boot-auto-configuration)如果使用了[ImportSelector](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/ImportSelector.html)，那么[ImportSelector](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/ImportSelector.html)会被提前加载。参见例子代码里的UselessDeferredImportSelectorAutoConfiguration。

参考[EnableAutoConfiguration](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/autoconfigure/EnableAutoConfiguration.html)和附录[EnableAutoConfigurationImportSelector](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/autoconfigure/EnableAutoConfigurationImportSelector.html)了解Spring boot内部处理机制

### AnnotatedBeanDefinitionReader

这个类用来读取[@Configuration](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/Configuration.html)和[@Component](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/stereotype/Component.html)，并将[BeanDefinition](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/config/BeanDefinition.html)注册到[ApplicationContext](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationContext.html)里。

### ConfigurationClassPostProcessor

[ConfigurationClassPostProcessor](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/ConfigurationClassPostProcessor.html)是一个[BeanDefinitionRegistryPostProcessor](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/support/BeanDefinitionRegistryPostProcessor.html)，负责处理[@Configuration](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/Configuration.html)。

需要注意一个烟雾弹：看[#L296](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/ConfigurationClassPostProcessor.java#L296)->[ConfigurationClassUtils#L209](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/ConfigurationClassUtils.java#L209)。而order的值则是在[ConfigurationClassUtils#L122](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/ConfigurationClassUtils.java#L122)从注解中提取的。
这段代码似乎告诉我们它会对[@Configuration](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/Configuration.html)进行排序，然后按次序加载。
实际上不是的，[@Configuration](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/Configuration.html)是一个递归加载的过程。在本例中，是先从SampleApplication开始加载的，而事实上在这个时候，也就只有SampleApplication它自己可以提供排序。
而之后则直接使用了[ConfigurationClassParser](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/ConfigurationClassParser.java)，它里面并没有排序的逻辑。

关于排序的方式简单来说是这样的：[@Configuration](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/Configuration.html)的排序根据且只根据[@Order](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/annotation/Order.html)排序，如果没有[@Order](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/annotation/Order.html)则优先级最低。

### ConfigurationClassParser

前面讲了[ConfigurationClassPostProcessor](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/ConfigurationClassPostProcessor.html)使用[ConfigurationClassParser](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/ConfigurationClassParser.java)，实际上加载[@Configuration](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/Configuration.html)的工作是在这里做的。

下面讲以下加载的顺序：

1.  以SampleApplication为起点开始扫描

2.  扫描得到所有的[@Configuration](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/Configuration.html)和[@Component](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/stereotype/Component.html)，从中读取[BeanDefinition](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/config/BeanDefinition.html)并导入：

    1.  如果[@Configuration](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/Configuration.html)注解了[@Import](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/Import.html)

        1.  如果使用的是[ImportSelector](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/ImportSelector.html)，则递归导入

        2.  如果使用的是[ImportBeanDefinitionRegistrar](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/ImportBeanDefinitionRegistrar.html)，则递归导入

        3.  如果使用的是[DeferredImportSelector](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/DeferredImportSelector.html)，则仅收集不导入

    2.  如果[@Configuration](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/Configuration.html)注解了[@ImportResource](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/ImportResource.html)，则递归导入

3.  迭代之前收集的[DeferredImportSelector](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/DeferredImportSelector.html)，递归导入

那[Auto-configuration](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#using-boot-auto-configuration)在哪里呢？
实际上是在第3步里，[@SpringBootApplication](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/autoconfigure/SpringBootApplication.html)存在注解[@EnableAutoConfiguration](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/autoconfigure/EnableAutoConfiguration.html)，它使用了[EnableAutoConfigurationImportSelector](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/autoconfigure/EnableAutoConfigurationImportSelector.html)，
[EnableAutoConfigurationImportSelector](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/autoconfigure/EnableAutoConfigurationImportSelector.html)是一个[DeferredImportSelector](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/DeferredImportSelector.html)，所以也就是说，[Auto-configuration](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#using-boot-auto-configuration)是在普通[@Configuration](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/Configuration.html)之后再加载的。

顺带一提，如果[Auto-configuration](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#using-boot-auto-configuration)里再使用[DeferredImportSelector](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/DeferredImportSelector.html)，那么效果和使用[ImportSelector](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/ImportSelector.html)效果是一样的，不会再被延后处理。参见例子代码里的UselessDeferredImportSelectorAutoConfiguration。

### EnableAutoConfigurationImportSelector

[EnableAutoConfigurationImportSelector](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/autoconfigure/EnableAutoConfigurationImportSelector.html)负责导入[Auto-configuration](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#using-boot-auto-configuration)。

它利用[AutoConfigurationSorter](https://github.com/spring-projects/spring-boot/blob/v1.4.1.RELEASE/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/AutoConfigurationSorter.java)对[Auto-configuration](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#using-boot-auto-configuration)进行排序。逻辑算法是：

1.  先根据类名排序

2.  再根据[@AutoConfigureOrder](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/autoconfigure/AutoConfigureOrder.html)排序，如果没有[@AutoConfigureOrder](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/autoconfigure/AutoConfigureOrder.html)则优先级最低

3.  再根据[@AutoConfigureBefore](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/autoconfigure/AutoConfigureBefore.html)，[@AutoConfigureAfter](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/autoconfigure/AutoConfigureAfter.html)排序

## 内置类说明

### LoggingApplicationListener

[LoggingApplicationListener](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/logging/LoggingApplicationListener.html)用来配置日志系统的，比如logback、log4j。Spring boot对于日志有[详细解释](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#boot-features-logging)，如果你想[自定义日志配置](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/reference/htmlsingle/#boot-features-custom-log-configuration)，那么也请参考本文中对于[LoggingApplicationListener](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/logging/LoggingApplicationListener.html)的被调用时机的说明以获得更深入的了解。

### StandardEnvironment

[StandardEnvironment](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/env/StandardEnvironment.html)有一个[MutablePropertySources](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/env/MutablePropertySources.html)，它里面有多个[PropertySource](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/env/PropertySource.html)，[PropertySource](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/env/PropertySource.html)负责提供property（即property的提供源），目前已知的[PropertySource](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/env/PropertySource.html)实现有：[MapPropertySource](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/env/MapPropertySource.html)、[SystemEnvironmentPropertySource](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/env/SystemEnvironmentPropertySource.html)、[CommandLinePropertySource](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/env/CommandLinePropertySource.html)等。当[StandardEnvironment](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/env/StandardEnvironment.html)查找property值的时候，是从[MutablePropertySources](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/env/MutablePropertySources.html)里依次查找的，而且一旦查找到就不再查找，也就是说如果要覆盖property的值，那么就得提供顺序在前的[PropertySource](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/env/PropertySource.html)。

### ConfigFileApplicationListener

[ConfigFileApplicationListener](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/context/config/ConfigFileApplicationListener.html)用来将`application.properties`加载到[StandardEnvironment](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/env/StandardEnvironment.html)中。

[ConfigFileApplicationListener](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/context/config/ConfigFileApplicationListener.html)内部使用了[EnvironmentPostProcessor](http://docs.spring.io/spring-boot/docs/1.4.1.RELEASE/api/org/springframework/boot/env/EnvironmentPostProcessor.html)（见附录）自定义[StandardEnvironment](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/core/env/StandardEnvironment.html)

### ApplicationContextAwareProcessor

[ApplicationContextAwareProcessor](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/support/ApplicationContextAwareProcessor.java)实现了[BeanPostProcessor](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/config/BeanPostProcessor.html)接口，根据javadoc这个类用来调用以下接口的回调方法：

1.  [EnvironmentAware](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/EnvironmentAware.html)

2.  [EmbeddedValueResolverAware](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/EmbeddedValueResolverAware.html)

3.  [ResourceLoaderAware](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ResourceLoaderAware.html)

4.  [ApplicationEventPublisherAware](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationEventPublisherAware.html)

5.  [MessageSourceAware](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/MessageSourceAware.html)

6.  [ApplicationContextAware](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/ApplicationContextAware.html)

### AnnotationConfigApplicationContext

根据[javadoc](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/AnnotationConfigApplicationContext.html)，这个类用来将[@Configuration](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/Configuration.html)和[@Component](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/stereotype/Component.html)作为输入来注册BeanDefinition。

特别需要注意的是，在[javadoc](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/AnnotationConfigApplicationContext.html)中讲到其支持@Bean的覆盖：

> In case of multiple @Configuration classes, @Bean methods defined in later classes will override those defined in earlier classes. This can be leveraged to deliberately override certain bean definitions via an extra @Configuration class.

它使用[AnnotatedBeanDefinitionReader](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/AnnotatedBeanDefinitionReader.html)来读取[@Configuration](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/Configuration.html)和[@Component](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/stereotype/Component.html)。

### AnnotatedBeanDefinitionReader

[AnnotatedBeanDefinitionReader](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/AnnotatedBeanDefinitionReader.html)在其构造函数内部间接([AnnotationConfigUtils#L145](https://github.com/spring-projects/spring-framework/blob/v4.3.3.RELEASE/spring-context/src/main/java/org/springframework/context/annotation/AnnotationConfigUtils.java#L145))的给[BeanFactory](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/BeanFactory.html)注册了几个与[BeanDefinition](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/config/BeanDefinition.html)相关注解的处理器。

1.  [ConfigurationClassPostProcessor](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/ConfigurationClassPostProcessor.html)

2.  [AutowiredAnnotationBeanPostProcessor](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/annotation/AutowiredAnnotationBeanPostProcessor.html)

3.  [RequiredAnnotationBeanPostProcessor](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/beans/factory/annotation/RequiredAnnotationBeanPostProcessor.html)

4.  [CommonAnnotationBeanPostProcessor](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/annotation/CommonAnnotationBeanPostProcessor.html)

5.  [EventListenerMethodProcessor](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/context/event/EventListenerMethodProcessor.html)

6.  [PersistenceAnnotationBeanPostProcessor](http://docs.spring.io/spring/docs/4.3.3.RELEASE/javadoc-api/org/springframework/orm/jpa/support/PersistenceAnnotationBeanPostProcessor.html)

