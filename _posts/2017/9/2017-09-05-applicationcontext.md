---
layout: post
title: 十二种获取Spring的上下文环境ApplicationContext的方法
category: java
tags: [java]
---
#### 方式一：
使用实例：UserDao userDao = (UserDao)SpringUtil.getBean("userDao");
```java
public class SpringUtil {

       public static ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
 
       public static Object getBean(String serviceName){
             return context.getBean(serviceName);
       }
}
```
#### 方式二：
注意：这个地方使用了Spring的注解@Component，如果不是使用annotation的方式，而是使用xml的方式管理Bean，记得写入配置文件：
<bean id="springContextUtil" class="com.xxx.util.SpringContextUtil" singleton="true" />
```java
@Component
public class SpringContextUtil implements ApplicationContextAware {

         private static ApplicationContext applicationContext; // Spring应用上下文环境         
         /*          
         * 实现了ApplicationContextAware 接口，必须实现该方法；           
         *通过传递applicationContext参数初始化成员变量applicationContext          
         */
         @Override         
         public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
               SpringContextUtil.applicationContext = applicationContext;
         }

         public static ApplicationContext getApplicationContext() {
                return applicationContext;
         }

          @SuppressWarnings("unchecked")
          public static <T> T getBean(String name) throws BeansException {
                     return (T) applicationContext.getBean(name);
           }

}
```
####  方式三：
```java
ClassPathXmlApplicationContext resource = new ClassPathXmlApplicationContext(new String[]{       
"applicationContext-ibatis-oracle.xml",       
"applicationContext.xml",       
"applicationContext-data-oracle.xml"
      }
); 

BeanFactory factory = resource; 
UserDao userDao = (UserDao) factory.getBean("userDao");
```
#### 方式四：
利用ClassPathResource,可以从classpath中读取XML文件。
加载一个xml文件org.springframework.beans.factory.config.PropertyPlaceholderConfigurer不起作用
```java
Resource cr = new ClassPathResource("applicationContext.xml"); 
BeanFactory bf=new XmlBeanFactory(cr); 
UserDao userDao = (UserDao)bf.getBean("userDao");
```
#### 方式五：
利用XmlWebApplicationContext读取
```java
XmlWebApplicationContext ctx = new XmlWebApplicationContext(); 
ctx.setConfigLocations(new String[] {"/WEB-INF/ applicationContext.xml"); 
ctx.setServletContext(pageContext.getServletContext()); 
ctx.refresh(); 
UserDao userDao = (UserDao ) ctx.getBean("userDao ");
```
#### 方式六：
利用FileSystemResource读取。值得注意的是：利用FileSystemResource，则配置文件必须放在project直接目录下，或者写明绝对路径，否则就会抛出找不到文件的异常。
```java
Resource rs = new FileSystemResource("D:/tomcat/webapps/test/WEB-INF/classes/ applicationContext.xml"); 
BeanFactory factory = new XmlBeanFactory(rs); 
UserDao userDao = (UserDao )factory.getBean("userDao");
```
#### 方式七：
利用FileSystemXmlApplicationContext读取。可以指定XML定义文件的相对路径或者绝对路径来读取定义文件。
```java
方法一：
String[] path={     "WebRoot/WEB-INF/applicationContext.xml",     "WebRoot/WEB-INF/applicationContext_task.xml"
   }; 

ApplicationContext context = new FileSystemXmlApplicationContext(path);

方法二：
String path="WebRoot/WEB-INF/applicationContext*.xml"; ApplicationContext context = new FileSystemXmlApplicationContext(path);

方法三：
 
ApplicationContext ctx = new FileSystemXmlApplicationContext("classpath:地址");
没有classpath的话就是从当前的工作目录
```
#### 方式八：
通过Spring提供的工具类获取ApplicationContext对象 
```java
ApplicationContext ac1 = WebApplicationContextUtils.getRequiredWebApplicationContext(ServletContext sc);
ApplicationContext ac2 = WebApplicationContextUtils.getWebApplicationContext(ServletContext sc);
ac1.getBean("beanId");
ac2.getBean("beanId");
```
说明：这种方式适合于采用Spring框架的B/S系统，通过ServletContext对象获取ApplicationContext对象，然后在通过它获取需要的类实例。
上面两个工具方式的区别是，前者在获取失败时抛出异常，后者返回null。
#### 方式九：
继承自抽象类ApplicationObjectSupport  
说明：抽象类ApplicationObjectSupport提供getApplicationContext()方法，可以方便的获取ApplicationContext。 
Spring初始化时，会通过该抽象类的setApplicationContext(ApplicationContext context)方法将ApplicationContext 对象注入。
#### 方式十：
继承自抽象类WebApplicationObjectSupport  
说明：类似上面方法，调用getWebApplicationContext()获取WebApplicationContext  
#### 方式十一：
实现接口ApplicationContextAware 
说明：实现该接口的setApplicationContext(ApplicationContext context)方法，并保存ApplicationContext 对象。Spring初始化时，会通过该方法将ApplicationContext对象注入。 
以下是实现ApplicationContextAware接口方式的代码，前面两种方法类似：
```java
public class SpringContextUtil implements ApplicationContextAware {
	// Spring应用上下文环境
	private static ApplicationContext applicationContext;
	/**
	 *  实现ApplicationContextAware接口的回调方法，设置上下文环境
        * @param applicationContext
	 */
	public void setApplicationContext(ApplicationContext applicationContext) {
		SpringContextUtil.applicationContext = applicationContext;
	}

	/** 
       * @return ApplicationContext 
       */
	public static ApplicationContext getApplicationContext() {
		return applicationContext;
	}

	/** 获取对象
       *  @param name 
       * @return Object 
       * @throws BeansException 
       */
	public static Object getBean(String name) throws BeansException {
		return applicationContext.getBean(name);
	}
}
```
虽然，spring提供的后三种方法可以实现在普通的类中继承或实现相应的类或接口来获取spring 的ApplicationContext对象，但是在使用是一定要注意实现了这些类或接口的普通java类一定要在Spring 的配置文件applicationContext.xml文件中进行配置。否则获取的ApplicationContext对象将为null。
#### 方式十二：通过Spring提供的ContextLoader
```java
WebApplicationContext wac = ContextLoader.getCurrentWebApplicationContext();
wac.getBean(beanID);
```
最后提供一种不依赖于servlet,不需要注入的方式。但是需要注意一点，在服务器启动时，Spring容器初始化时，不能通过以下方法获取Spring 容器，细节可以查看spring源码org.springframework.web.context.ContextLoader
