---
layout: post
title: Java Web系列：Spring MVC基础
category: java
tags: [java]
---
## 1.Web MVC基础

MVC的本质是表现层模式，我们以视图模型为中心，将视图和控制器分离出来。就如同分层模式一样，我们以业务逻辑为中心，把表现层和数据访问层代码分离出来是一样的方法。框架只能在技术层面上给我们帮助，无法在思考和过程上帮助我们，而我们很多人都不喜欢思考和尝试。

![Java Web系列：Spring MVC基础](http://static.codeceo.com/images/2016/01/194803-20151226125914406-227481745.png)

## 2.实现Web MVC的基础

实现Web MVC基础可以概括为1个前段控制器和2个映射。

**（1）前端控制器FrontController**

ASP.NET和JSP都是以Page路径和URL一一对应，Web MVC要通过URL映射Controller和View，就需要一个前端控制器统一接收和解析请求，再根据的URL将请求分发到Controller。由于ASP.NET和Java分别以IHttpHandler和Servlet作为核心，因此ASP.NET MVC和Spring MVC分别使用实现了对应接口的MvcHandler和DispatcherServlet作为前段控制器。

![Java Web系列：Spring MVC基础](http://static.codeceo.com/images/2016/01/194803-20151228105951917-1044478022.png)

ASP.NET中通过HttpModule的实现类处理URL映射，UrlRoutingModule根据URL将请求转发给前端控制器MvcHandler。Spring MVC中，则根据URL的配置，直接将请求转发给前端控制器DispatcherServlet。

**（2）URL和Contrller的映射**

ASP.NET MVC将URL和Controller的映射规则存储在RouteCollection中，前端控制器MvcHandler通过IController接口查找控制器。Spring MVC则通过RequestMapping和Controller注解标识映射规则，无需通过接口依赖实现控制i器。

**（3）URL和View的映射**

ASP.NET MVC 默认通过RazorViewEngine来根据URL和视图名称查找视图，核心接口是IViewEngine。Spring MVC 通过internalResourceViewResolver根据URL和视图名称查找视图，核心接口是ViewResolver。

## 3.Spring MVC的基础配置

（1）前端控制器DispatcherServlet初始化：AbstractAnnotationConfigDispatcherServletInitializer

ASP.NET MVC初始化需要我们在HttpApplication.Application_Start方法中注册默认的URL和Controller规则，Spring MVC由于采用注解映射URL和Controller，因此没有对应的步骤。ASP.NET在根web.config中配置了UrlRoutingModule可以将请求转发给MvcHandler，Spring MVC我们需要我们配置DispatcherServlet以及其对应的URL来达到接管所有请求的目的，Spring已经利用Servlet3.0定义的ServletContainerInitializer机制，为我们提供了内置的AbstractAnnotationConfigDispatcherServletInitializer，只要只需要像继承HttpApplication的MvcApplication一样，写一个MvcInitializer。

```java
package s4s;

import org.springframework.web.servlet.support.AbstractAnnotationConfigDispatcherServletInitializer;

public class MvcInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[] { };
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[] { MvcConfig.class };
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }

}
```

（2）URL和View的映射：WebMvcConfigurerAdapter

ASP.NET的RazorViewEngine内置了View的Path和扩展名.cshtml的规则。Spring MVC的internalResourceViewResolver没有提供默认值，一般我们会指定将View放置在统一的视图目录中，使用特定的扩展名。Spring同样提供了内置的WebMvcConfigurerAdapter，我们只需写一个自己的MvcConfig继承它，重写configureViewResolvers方法即可。

```java
package s4s;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.EnableWebMvc;
import org.springframework.web.servlet.config.annotation.ViewResolverRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
import org.springframework.web.servlet.view.InternalResourceViewResolver;

@EnableWebMvc
@ComponentScan
@Configuration
public class MvcConfig extends WebMvcConfigurerAdapter {

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setPrefix("/WEB-INF/views/");
        viewResolver.setSuffix(".jsp");
        registry.viewResolver(viewResolver);
    }
}
```

## 4.Spring MVC的Controller、Model和View

**（1）URL和Controller的映射：**

Spring MVC和ASP.NET MVC的不同，不通过IController接口标识Controller，也不通过RouteCollection定义URL和Controller，取而代之的是两个注解：Controller和RequestMapping。因此我们在普通的POJO类上应用@Controller和@RequestMapping即可。

```java
package s4s;

import javax.validation.Valid;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.stereotype.Controller;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class MyController {

    @ResponseBody
    @RequestMapping(value = "/")
    public String home() {
        return "home";
    }

    @RequestMapping(value = "/register")
    public String register(@ModelAttribute("model") RegisterUserModel model) {
        return "register";
    }

    @RequestMapping(value = "/register", method = RequestMethod.POST)
    public String register(@ModelAttribute("model") @Valid RegisterUserModel model, BindingResult result) {
        if (!result.hasErrors()) {
            return "redirect:/account";
        }
        return "register";
    }
}
```

**（2）Model：**

通过使用@ModelAttribute、@Valid和BindingResult参数，我们可以指定Model的Name是否参与验证并获取验证结果。为在Model上使用注解验证，还需要引入validation-api和hibernate-validator。

ASP.NET将视图最终编译为WebViewPage<object>，View和Model是一一对应并且类型匹配的，Model可以是任意的POCO。Spring MVC中View和Model是一对多的，提供了ModelMap和其子类ModelAndView提供类似ASP.NET MVC中ViewResult的功能。ModelMap的基类是LinkedHashMap<String, Object>。

Spring MVC中没有ViewResult类型。在Spring MVC中，我们一般返回String类型，可以有多种含义：

a.返回View的名称。

b.返回文本：在Action上应用@ResponseBody注解时。

c.返回跳转：以”redirect:”开头时。如：return “redirect:/success”

模型的验证：

（1）在Model字段上使用JSR-303定义的注解（需要引入hibernate validator）。

（2）在Controller的Model参数上应用@ModelAttribute、@Valid

（3）在View中使用<form:errors>标签

Spring MVC需要添加jstl和spring的tag支持才能完成模型相关的操作。由于Spring MVC中的View和ASP.NET MVC中的区别较大，没有办法指定View持有的Model类型也就没有了智能提示和错误检测的优势。

```java
package s4s;

import javax.validation.constraints.NotNull;
import javax.validation.constraints.Size;

public class RegisterUserModel {
    @Size(max = 20, min = 5)
    private String userName;
    @Size(max = 20, min = 5)
    private String password;
    @NotNull
    private String confirmPassword;

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getConfirmPassword() {
        return confirmPassword;
    }

    public void setConfirmPassword(String confirmPassword) {
        this.confirmPassword = confirmPassword;
    }
}
```

register.jsp

```java
<%@ page language="java" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
<%@ taglib uri="http://www.springframework.org/tags" prefix="s"%>
<%@ taglib uri="http://www.springframework.org/tags/form" prefix="form"%>
<!DOCTYPE HTML>
<html>
<head>
<title>Getting Started: Serving Web Content</title>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
    <h2>Register</h2>
    <form:form modelAttribute="model">
        <s:bind path="*">
            <c:if test="${status.error}">
                <div id="message" class="error">Form has errors</div>
            </c:if>
        </s:bind>
        <div>
            <form:label path="userName">userName</form:label>
            <form:input path="userName" />
            <form:errors path="userName" cssClass="error" />
        </div>
        <div>
            <form:label path="password">password</form:label>
            <form:password path="password" />
            <form:errors path="password" cssClass="error" />
        </div>
        <div>
            <form:label path="confirmPassword">confirmPassword</form:label>
            <form:password path="confirmPassword" />
            <form:errors path="confirmPassword" cssClass="error" />
        </div>
        <input type="submit" value="submit">
    </form:form>
</html>
```

## 5.Spring MVC的初始化机制

Spring实现了Servlet 3.0规范定义的javax.servlet.ServletContainerInitializer接口并通过javax.servlet.annotation.HandlesTypes注解引用了WebApplicationInitializer接口。因此在Servlet容器初始化时，在当前class path路径下的WebApplicationInitializer实现类的onStartup方法会自动执行（这和ASP.NET的Application_Start作用类似，在系列中的[Java Web基础](http://www.cnblogs.com/easygame/p/5055088.html)时曾经提到过）。

ASP.NET中我们在Application_Start中初始化依赖注入容器。在Spring MVC中，我们实现WebApplicationInitializer接口同样可以执行依赖注入的初始化。在Web环境中，我们使用的ApplicationContext接口的实现类为基于注解的AnnotationConfigWebApplicationContext（在系列中的[Spring依赖注入基础](http://www.codeceo.com/article/java-web-spring-di.html)中曾经提到过），但我们无需直接实现WebApplicationInitializer并手动初始化AnnotationConfigWebApplicationContext对象，因为Spring已经定义了AbstractAnnotationConfigDispatcherServletInitializer作为WebApplicationInitializer接口的实现类，已经包含了AnnotationConfigWebApplicationContext的初始化。

采用基于Annotation注解时可以通过@Configurateion指定POJO来替代web.xml配置依赖注入。同样，@ComponentScan可以替代web.xml中的扫描配置功能，使用ComponentScan配合Configurateion可以达到0xml配置的方式。上文中提到的Contrller相关的注解，都是启用ComponentScan后才会被扫描生效。

AbstractAnnotationConfigDispatcherServletInitializer类的父类AbstractDispatcherServletInitializer中已经包含DispatcherServlet的初始化。相关类图如下：

![Java Web系列：Spring MVC基础](http://static.codeceo.com/images/2016/01/194803-20151227095754968-387757003.png)

## 5.Spring MVC的Action Filter

.NET MVC提供了众多Filter接口和一个ActionFilterAttribute抽象类作为Filter的基础,其中以实现了IAuthorizationFilter接口的AuthorizeAttribute拦截器最为我们熟知。Spring MVC则提供了基于HandlerInterceptor接口的众多接口、抽象类和实现类，其中也有和.NET MVC类似的权限验证UserRoleAuthorizationInterceptor拦截器。内置的拦截器可以满足大部分需求，为了省事图就画在一张上了，上面是Spring MVC的，下面是.NET MVC的。

![Java Web系列：Spring MVC基础](http://static.codeceo.com/images/2016/01/194803-20151228152512042-633287775.png)

## 总结

（1）MVC实现的要点是前端控制器、URL和Controller的映射、URL和View的映射

（2）MvcHandler和DispatcherServlet

（3）ServletContainerInitializer和HttpApplication.Application_Start

（4）RazorViewEngine和internalResourceViewResolver

（5）IMvcFilter和HandlerInterceptor

目前没有找到类似ASP.NET中的从特性（注解）生成客户端JavaScript验证的方式，如果大家有相关资料分享，提前谢谢大家。

## 参考：

（1）http://www.ibm.com/developerworks/cn/java/j-lo-jsr303/index.html

（2）http://spring.oschina.mopaas.com/validation.html#validation-binder

（3）http://www.mkyong.com/spring-mvc/spring-3-mvc-and-jsr303-valid-example/

**随笔里的文章都是干货，都是解决实际问题的，这些问题我在网上找不到更好的解决方法，才会有你看到的这一篇篇随笔，因此如果这篇博客内容对您稍有帮助或略有借鉴，请您推荐它帮助更多的人。如果你有提供实际可行的更好方案，请推荐给我。**
