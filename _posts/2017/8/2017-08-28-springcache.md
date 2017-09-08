---
layout: post
title: Spring Cache
category: java
tags: [java]
---
`缓存`是实际工作中非常常用的一种提高性能的方法, 我们会在许多场景下来使用缓存。

本文通过一个简单的例子进行展开，通过对比我们原来的自定义缓存和 spring 的基于注释的 cache 配置方法，展现了 spring cache 的强大之处，然后介绍了其基本的原理，扩展点和使用场景的限制。通过阅读本文，你应该可以短时间内掌握 spring 带来的强大缓存技术，在很少的配置下即可给既有代码提供缓存能力。

## 概述

Spring 3.1 引入了激动人心的基于注释（annotation）的缓存（cache）技术，它本质上不是一个具体的缓存实现方案（例如EHCache 或者 OSCache），而是一个对缓存使用的抽象，通过在既有代码中添加少量它定义的各种 annotation，即能够达到缓存方法的返回对象的效果。

Spring 的缓存技术还具备相当的灵活性，不仅能够使用 SpEL（Spring Expression Language）来定义缓存的 key 和各种 condition，还提供开箱即用的缓存临时存储方案，也支持和主流的专业缓存例如 EHCache 集成。

其特点总结如下：

*   通过少量的配置 annotation 注释即可使得既有代码支持缓存
*   支持开箱即用 Out-Of-The-Box，即不用安装和部署额外第三方组件即可使用缓存
*   支持 Spring Express Language，能使用对象的任何属性或者方法来定义缓存的 key 和 condition
*   支持 AspectJ，并通过其实现任何方法的缓存支持
*   支持自定义 key 和自定义缓存管理者，具有相当的灵活性和扩展性

本文将针对上述特点对 Spring cache 进行详细的介绍，主要通过一个简单的例子和原理介绍展开，然后我们将一起看一个比较实际的缓存例子，最后会介绍 spring cache 的使用限制和注意事项。好吧，让我们开始吧

## 我们以前如何自己实现缓存的呢

这里先展示一个完全自定义的缓存实现，即不用任何第三方的组件来实现某种对象的内存缓存。

场景如下：

> 对一个账号查询方法做缓存，以账号名称为 key，账号对象为 value，当以相同的账号名称查询账号的时候，直接从缓存中返回结果，否则更新缓存。账号查询服务还支持 reload 缓存（即清空缓存）

首先定义一个实体类：账号类，具备基本的 id 和 name 属性，且具备 getter 和 setter 方法

```java
public class Account {

    private int id;
    private String name;

    public Account(String name) {
        this.name = name;
    }
    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }

}
```

然后定义一个缓存管理器，这个管理器负责实现缓存逻辑，支持对象的增加、修改和删除，支持值对象的泛型。如下：

```java
import com.google.common.collect.Maps;

import java.util.Map;

/**
 * @author wenchao.ren
 *         2015/1/5.
 */
public class CacheContext<T> {

    private Map<String, T> cache = Maps.newConcurrentMap();

    public T get(String key){
        return  cache.get(key);
    }

    public void addOrUpdateCache(String key,T value) {
        cache.put(key, value);
    }

    // 根据 key 来删除缓存中的一条记录
    public void evictCache(String key) {
        if(cache.containsKey(key)) {
            cache.remove(key);
        }
    }

    // 清空缓存中的所有记录
    public void evictCache() {
        cache.clear();
    }

}
```

好，现在我们有了实体类和一个缓存管理器，还需要一个提供账号查询的服务类，此服务类使用缓存管理器来支持账号查询缓存，如下：

```java
import com.google.common.base.Optional;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;

/**
 * @author wenchao.ren
 *         2015/1/5.
 */
@Service
public class AccountService1 {

    private final Logger logger = LoggerFactory.getLogger(AccountService1.class);

    @Resource
    private CacheContext<Account> accountCacheContext;

    public Account getAccountByName(String accountName) {
        Account result = accountCacheContext.get(accountName);
        if (result != null) {
            logger.info("get from cache... {}", accountName);
            return result;
        }

        Optional<Account> accountOptional = getFromDB(accountName);
        if (!accountOptional.isPresent()) {
            throw new IllegalStateException(String.format("can not find account by account name : [%s]", accountName));
        }

        Account account = accountOptional.get();
        accountCacheContext.addOrUpdateCache(accountName, account);
        return account;
    }

    public void reload() {
        accountCacheContext.evictCache();
    }

    private Optional<Account> getFromDB(String accountName) {
        logger.info("real querying db... {}", accountName);
        //Todo query data from database
        return Optional.fromNullable(new Account(accountName));
    }

}
```

现在我们开始写一个测试类，用于测试刚才的缓存是否有效

```java
import org.junit.Before;
import org.junit.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import static org.junit.Assert.*;

public class AccountService1Test {

    private AccountService1 accountService1;

    private final Logger logger = LoggerFactory.getLogger(AccountService1Test.class);

    @Before
    public void setUp() throws Exception {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext1.xml");
        accountService1 = context.getBean("accountService1", AccountService1.class);
    }

    @Test
    public void testInject(){
        assertNotNull(accountService1);
    }

    @Test
    public void testGetAccountByName() throws Exception {
        accountService1.getAccountByName("accountName");
        accountService1.getAccountByName("accountName");

        accountService1.reload();
        logger.info("after reload ....");

        accountService1.getAccountByName("accountName");
        accountService1.getAccountByName("accountName");
    }
}
```

按照分析，执行结果应该是：首先从数据库查询，然后直接返回缓存中的结果，重置缓存后，应该先从数据库查询，然后返回缓存中的结果. 查看程序运行的日志如下：

```
00:53:17.166 [main] INFO  c.r.s.cache.example1.AccountService - real querying db... accountName
00:53:17.168 [main] INFO  c.r.s.cache.example1.AccountService - get from cache... accountName
00:53:17.168 [main] INFO  c.r.s.c.example1.AccountServiceTest - after reload ....
00:53:17.168 [main] INFO  c.r.s.cache.example1.AccountService - real querying db... accountName
00:53:17.169 [main] INFO  c.r.s.cache.example1.AccountService - get from cache... accountName
```

可以看出我们的缓存起效了，但是这种自定义的缓存方案有如下劣势：

*   缓存代码和业务代码耦合度太高，如上面的例子，AccountService 中的 getAccountByName（）方法中有了太多缓存的逻辑，不便于维护和变更
*   不灵活，这种缓存方案不支持按照某种条件的缓存，比如只有某种类型的账号才需要缓存，这种需求会导致代码的变更
*   缓存的存储这块写的比较死，不能灵活的切换为使用第三方的缓存模块

如果你的代码中有上述代码的影子，那么你可以考虑按照下面的介绍来优化一下你的代码结构了，也可以说是简化，你会发现，你的代码会变得优雅的多！

## Spring cache是如何做的呢

我们对AccountService1 进行修改，创建AccountService2:

```java
import com.google.common.base.Optional;
import com.rollenholt.spring.cache.example1.Account;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

/**
 * @author wenchao.ren
 *         2015/1/5.
 */
@Service
public class AccountService2 {

    private final Logger logger = LoggerFactory.getLogger(AccountService2.class);

    // 使用了一个缓存名叫 accountCache
    @Cacheable(value="accountCache")
    public Account getAccountByName(String accountName) {

        // 方法内部实现不考虑缓存逻辑，直接实现业务
        logger.info("real querying account... {}", accountName);
        Optional<Account> accountOptional = getFromDB(accountName);
        if (!accountOptional.isPresent()) {
            throw new IllegalStateException(String.format("can not find account by account name : [%s]", accountName));
        }

        return accountOptional.get();
    }

    private Optional<Account> getFromDB(String accountName) {
        logger.info("real querying db... {}", accountName);
        //Todo query data from database
        return Optional.fromNullable(new Account(accountName));
    }

}
```

我们注意到在上面的代码中有一行：

```
     @Cacheable(value="accountCache")

```

这个注释的意思是，当调用这个方法的时候，会从一个名叫 accountCache 的缓存中查询，如果没有，则执行实际的方法（即查询数据库），并将执行的结果存入缓存中，否则返回缓存中的对象。这里的缓存中的 key 就是参数 accountName，value 就是 Account 对象。“accountCache”缓存是在 spring*.xml 中定义的名称。我们还需要一个 spring 的配置文件来支持基于注释的缓存

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:cache="http://www.springframework.org/schema/cache"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context.xsd
           http://www.springframework.org/schema/cache
           http://www.springframework.org/schema/cache/spring-cache.xsd">

    <context:component-scan base-package="com.rollenholt.spring.cache"/>

    <context:annotation-config/>

    <cache:annotation-driven/>

    <bean id="cacheManager" class="org.springframework.cache.support.SimpleCacheManager">
        <property name="caches">
            <set>
                <bean class="org.springframework.cache.concurrent.ConcurrentMapCacheFactoryBean">
                    <property name="name" value="default"/>
                </bean>
                <bean class="org.springframework.cache.concurrent.ConcurrentMapCacheFactoryBean">
                    <property name="name" value="accountCache"/>
                </bean>
            </set>
        </property>
    </bean>

</beans>
```

注意这个 spring 配置文件有一个关键的支持缓存的配置项：

```
<cache:annotation-driven />
```

这个配置项缺省使用了一个名字叫 cacheManager 的缓存管理器，这个缓存管理器有一个 spring 的缺省实现，即 `org.springframework.cache.support.SimpleCacheManager`，这个缓存管理器实现了我们刚刚自定义的缓存管理器的逻辑，它需要配置一个属性 caches，即此缓存管理器管理的缓存集合，除了缺省的名字叫 default 的缓存，我们还自定义了一个名字叫 accountCache 的缓存，使用了缺省的内存存储方案 `ConcurrentMapCacheFactoryBea`n，它是基于 `java.util.concurrent.ConcurrentHashMap` 的一个内存缓存实现方案。

然后我们编写测试程序：

```java
 import org.junit.Before;
import org.junit.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import static org.junit.Assert.*;

public class AccountService2Test {

    private AccountService2 accountService2;

    private final Logger logger = LoggerFactory.getLogger(AccountService2Test.class);

    @Before
    public void setUp() throws Exception {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext2.xml");
        accountService2 = context.getBean("accountService2", AccountService2.class);
    }

    @Test
    public void testInject(){
        assertNotNull(accountService2);
    }

    @Test
    public void testGetAccountByName() throws Exception {
        logger.info("first query...");
        accountService2.getAccountByName("accountName");

        logger.info("second query...");
        accountService2.getAccountByName("accountName");
    }
}
```

上面的测试代码主要进行了两次查询，第一次应该会查询数据库，第二次应该返回缓存，不再查数据库，我们执行一下，看看结果

```
01:10:32.435 [main] INFO  c.r.s.c.example2.AccountService2Test - first query...
01:10:32.456 [main] INFO  c.r.s.cache.example2.AccountService2 - real querying account... accountName
01:10:32.457 [main] INFO  c.r.s.cache.example2.AccountService2 - real querying db... accountName
01:10:32.458 [main] INFO  c.r.s.c.example2.AccountService2Test - second query...
```

可以看出我们设置的基于注释的缓存起作用了，而在 AccountService.java 的代码中，我们没有看到任何的缓存逻辑代码，只有一行注释：@Cacheable(value="accountCache")，就实现了基本的缓存方案，是不是很强大？

## 如何清空缓存

好，到目前为止，我们的 spring cache 缓存程序已经运行成功了，但是还不完美，因为还缺少一个重要的缓存管理逻辑：清空缓存.

当账号数据发生变更，那么必须要清空某个缓存，另外还需要定期的清空所有缓存，以保证缓存数据的可靠性。

为了加入清空缓存的逻辑，我们只要对 AccountService2.java 进行修改，从业务逻辑的角度上看，它有两个需要清空缓存的地方

*   当外部调用更新了账号，则我们需要更新此账号对应的缓存
*   当外部调用说明重新加载，则我们需要清空所有缓存

我们在AccountService2的基础上进行修改，修改为AccountService3，代码如下：

```java
import com.google.common.base.Optional;
import com.rollenholt.spring.cache.example1.Account;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cache.annotation.CacheEvict;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

/**
 * @author wenchao.ren
 *         2015/1/5.
 */
@Service
public class AccountService3 {

    private final Logger logger = LoggerFactory.getLogger(AccountService3.class);

    // 使用了一个缓存名叫 accountCache
    @Cacheable(value="accountCache")
    public Account getAccountByName(String accountName) {

        // 方法内部实现不考虑缓存逻辑，直接实现业务
        logger.info("real querying account... {}", accountName);
        Optional<Account> accountOptional = getFromDB(accountName);
        if (!accountOptional.isPresent()) {
            throw new IllegalStateException(String.format("can not find account by account name : [%s]", accountName));
        }

        return accountOptional.get();
    }

    @CacheEvict(value="accountCache",key="#account.getName()")
    public void updateAccount(Account account) {
        updateDB(account);
    }

    @CacheEvict(value="accountCache",allEntries=true)
    public void reload() {
    }

    private void updateDB(Account account) {
        logger.info("real update db...{}", account.getName());
    }

    private Optional<Account> getFromDB(String accountName) {
        logger.info("real querying db... {}", accountName);
        //Todo query data from database
        return Optional.fromNullable(new Account(accountName));
    }
}
```

我们的测试代码如下：

```java
import com.rollenholt.spring.cache.example1.Account;
import org.junit.Before;
import org.junit.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AccountService3Test {

    private AccountService3 accountService3;

    private final Logger logger = LoggerFactory.getLogger(AccountService3Test.class);

    @Before
    public void setUp() throws Exception {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("applicationContext2.xml");
        accountService3 = context.getBean("accountService3", AccountService3.class);
    }

    @Test
    public void testGetAccountByName() throws Exception {

        logger.info("first query.....");
        accountService3.getAccountByName("accountName");

        logger.info("second query....");
        accountService3.getAccountByName("accountName");

    }

    @Test
    public void testUpdateAccount() throws Exception {
        Account account1 = accountService3.getAccountByName("accountName1");
        logger.info(account1.toString());
        Account account2 = accountService3.getAccountByName("accountName2");
        logger.info(account2.toString());

        account2.setId(121212);
        accountService3.updateAccount(account2);

        // account1会走缓存
        account1 = accountService3.getAccountByName("accountName1");
        logger.info(account1.toString());
        // account2会查询db
        account2 = accountService3.getAccountByName("accountName2");
        logger.info(account2.toString());

    }

    @Test
    public void testReload() throws Exception {
        accountService3.reload();
        // 这2行查询数据库
        accountService3.getAccountByName("somebody1");
        accountService3.getAccountByName("somebody2");

        // 这两行走缓存
        accountService3.getAccountByName("somebody1");
        accountService3.getAccountByName("somebody2");
    }
}
```

在这个测试代码中我们重点关注`testUpdateAccount()`方法，在测试代码中我们已经注释了在update完account2以后，再次查询的时候，account1会走缓存，而account2不会走缓存，而去查询db，观察程序运行日志，运行日志为：

```log
01:37:34.549 [main] INFO  c.r.s.cache.example3.AccountService3 - real querying account... accountName1
01:37:34.551 [main] INFO  c.r.s.cache.example3.AccountService3 - real querying db... accountName1
01:37:34.552 [main] INFO  c.r.s.c.example3.AccountService3Test - Account{id=0, name='accountName1'}
01:37:34.553 [main] INFO  c.r.s.cache.example3.AccountService3 - real querying account... accountName2
01:37:34.553 [main] INFO  c.r.s.cache.example3.AccountService3 - real querying db... accountName2
01:37:34.555 [main] INFO  c.r.s.c.example3.AccountService3Test - Account{id=0, name='accountName2'}
01:37:34.555 [main] INFO  c.r.s.cache.example3.AccountService3 - real update db...accountName2
01:37:34.595 [main] INFO  c.r.s.c.example3.AccountService3Test - Account{id=0, name='accountName1'}
01:37:34.596 [main] INFO  c.r.s.cache.example3.AccountService3 - real querying account... accountName2
01:37:34.596 [main] INFO  c.r.s.cache.example3.AccountService3 - real querying db... accountName2
01:37:34.596 [main] INFO  c.r.s.c.example3.AccountService3Test - Account{id=0, name='accountName2'}
```

我们会发现实际运行情况和我们预估的结果是一致的。

## 如何按照条件操作缓存

前面介绍的缓存方法，没有任何条件，即所有对 accountService 对象的 getAccountByName 方法的调用都会起动缓存效果，不管参数是什么值。

> 如果有一个需求，就是只有账号名称的长度小于等于 4 的情况下，才做缓存，大于 4 的不使用缓存

虽然这个需求比较坑爹，但是抛开需求的合理性，我们怎么实现这个功能呢？

通过查看`CacheEvict`注解的定义，我们会发现：

```java
/**
 * Annotation indicating that a method (or all methods on a class) trigger(s)
 * a cache invalidate operation.
 *
 * @author Costin Leau
 * @author Stephane Nicoll
 * @since 3.1
 * @see CacheConfig
 */
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface CacheEvict {

    /**
     * Qualifier value for the specified cached operation.
     * <p>May be used to determine the target cache (or caches), matching the qualifier
     * value (or the bean name(s)) of (a) specific bean definition.
     */
    String[] value() default {};

    /**
     * Spring Expression Language (SpEL) attribute for computing the key dynamically.
     * <p>Default is "", meaning all method parameters are considered as a key, unless
     * a custom {@link #keyGenerator()} has been set.
     */
    String key() default "";

    /**
     * The bean name of the custom {@link org.springframework.cache.interceptor.KeyGenerator} to use.
     * <p>Mutually exclusive with the {@link #key()} attribute.
     */
    String keyGenerator() default "";

    /**
     * The bean name of the custom {@link org.springframework.cache.CacheManager} to use to
     * create a default {@link org.springframework.cache.interceptor.CacheResolver} if none
     * is set already.
     * <p>Mutually exclusive with the {@link #cacheResolver()}  attribute.
     * @see org.springframework.cache.interceptor.SimpleCacheResolver
     */
    String cacheManager() default "";

    /**
     * The bean name of the custom {@link org.springframework.cache.interceptor.CacheResolver} to use.
     */
    String cacheResolver() default "";

    /**
     * Spring Expression Language (SpEL) attribute used for conditioning the method caching.
     * <p>Default is "", meaning the method is always cached.
     */
    String condition() default "";

    /**
     * Whether or not all the entries inside the cache(s) are removed or not. By
     * default, only the value under the associated key is removed.
     * <p>Note that setting this parameter to {@code true} and specifying a {@link #key()}
     * is not allowed.
     */
    boolean allEntries() default false;

    /**
     * Whether the eviction should occur after the method is successfully invoked (default)
     * or before. The latter causes the eviction to occur irrespective of the method outcome (whether
     * it threw an exception or not) while the former does not.
     */
    boolean beforeInvocation() default false;
}
```

定义中有一个`condition`描述：

> Spring Expression Language (SpEL) attribute used for conditioning the method caching.Default is "", meaning the method is always cached.

我们可以利用这个方法来完成这个功能，下面只给出示例代码：

```java
@Cacheable(value="accountCache",condition="#accountName.length() <= 4")// 缓存名叫 accountCache 
public Account getAccountByName(String accountName) {
    // 方法内部实现不考虑缓存逻辑，直接实现业务
    return getFromDB(accountName);
}
```

注意其中的 `condition=”#accountName.length() <=4”`，这里使用了 SpEL 表达式访问了参数 accountName 对象的 length() 方法，条件表达式返回一个布尔值，true/false，当条件为 true，则进行缓存操作，否则直接调用方法执行的返回结果。

## 如果有多个参数，如何进行 key 的组合

我们看看`CacheEvict`注解的`key()`方法的描述：

> Spring Expression Language (SpEL) attribute for computing the key dynamically. Default is "", meaning all method parameters are considered as a key, unless a custom [{@link](mailto:%7B@link) #keyGenerator()} has been set.

假设我们希望根据对象相关属性的组合来进行缓存，比如有这么一个场景：

> 要求根据账号名、密码和是否发送日志查询账号信息

很明显，这里我们需要根据账号名、密码对账号对象进行缓存，而第三个参数“是否发送日志”对缓存没有任何影响。所以，我们可以利用 SpEL 表达式对缓存 key 进行设计

我们为Account类增加一个password 属性， 然后修改AccountService代码：

```java
 @Cacheable(value="accountCache",key="#accountName.concat(#password)") 
 public Account getAccount(String accountName,String password,boolean sendLog) { 
   // 方法内部实现不考虑缓存逻辑，直接实现业务
   return getFromDB(accountName,password); 
 }

```

注意上面的 key 属性，其中引用了方法的两个参数 accountName 和 password，而 sendLog 属性没有考虑，因为其对缓存没有影响。

```java
accountService.getAccount("accountName", "123456", true);// 查询数据库
accountService.getAccount("accountName", "123456", true);// 走缓存
accountService.getAccount("accountName", "123456", false);// 走缓存
accountService.getAccount("accountName", "654321", true);// 查询数据库
accountService.getAccount("accountName", "654321", true);// 走缓存
```

## 如何做到：既要保证方法被调用，又希望结果被缓存

根据前面的例子，我们知道，如果使用了 @Cacheable 注释，则当重复使用相同参数调用方法的时候，方法本身不会被调用执行，即方法本身被略过了，取而代之的是方法的结果直接从缓存中找到并返回了。

现实中并不总是如此，有些情况下我们希望方法一定会被调用，因为其除了返回一个结果，还做了其他事情，例如记录日志，调用接口等，这个时候，我们可以用 `@CachePut` 注释，这个注释可以确保方法被执行，同时方法的返回值也被记录到缓存中。

```java
@Cacheable(value="accountCache")
 public Account getAccountByName(String accountName) { 
   // 方法内部实现不考虑缓存逻辑，直接实现业务
   return getFromDB(accountName); 
 } 

 // 更新 accountCache 缓存
 @CachePut(value="accountCache",key="#account.getName()")
 public Account updateAccount(Account account) { 
   return updateDB(account); 
 } 
 private Account updateDB(Account account) { 
   logger.info("real updating db..."+account.getName()); 
   return account; 
 }
```

我们的测试代码如下

```java
Account account = accountService.getAccountByName("someone"); 
account.setPassword("123"); 
accountService.updateAccount(account); 
account.setPassword("321"); 
accountService.updateAccount(account); 
account = accountService.getAccountByName("someone"); 
logger.info(account.getPassword()); 
```

如上面的代码所示，我们首先用 getAccountByName 方法查询一个人 someone 的账号，这个时候会查询数据库一次，但是也记录到缓存中了。然后我们修改了密码，调用了 updateAccount 方法，这个时候会执行数据库的更新操作且记录到缓存，我们再次修改密码并调用 updateAccount 方法，然后通过 getAccountByName 方法查询，这个时候，由于缓存中已经有数据，所以不会查询数据库，而是直接返回最新的数据，所以打印的密码应该是“321”

## @Cacheable、@CachePut、@CacheEvict 注释介绍

*   @Cacheable 主要针对方法配置，能够根据方法的请求参数对其结果进行缓存
*   @CachePut 主要针对方法配置，能够根据方法的请求参数对其结果进行缓存，和 @Cacheable 不同的是，它每次都会触发真实方法的调用
    [-@CachEvict](mailto:-@CachEvict) 主要针对方法配置，能够根据一定的条件对缓存进行清空

## 基本原理

一句话介绍就是Spring AOP的动态代理技术。 如果读者对Spring AOP不熟悉的话，可以去看看官方文档

## 扩展性

直到现在，我们已经学会了如何使用开箱即用的 spring cache，这基本能够满足一般应用对缓存的需求。

但现实总是很复杂，当你的用户量上去或者性能跟不上，总需要进行扩展，这个时候你或许对其提供的内存缓存不满意了，因为其不支持高可用性，也不具备持久化数据能力，这个时候，你就需要自定义你的缓存方案了。

还好，spring 也想到了这一点。我们先不考虑如何持久化缓存，毕竟这种第三方的实现方案很多。

我们要考虑的是，怎么利用 spring 提供的扩展点实现我们自己的缓存，且在不改原来已有代码的情况下进行扩展。

首先，我们需要提供一个 `CacheManager` 接口的实现，这个接口告诉 spring 有哪些 cache 实例，spring 会根据 cache 的名字查找 cache 的实例。另外还需要自己实现 Cache 接口，Cache 接口负责实际的缓存逻辑，例如增加键值对、存储、查询和清空等。

利用 Cache 接口，我们可以对接任何第三方的缓存系统，例如 `EHCache`、`OSCache`，甚至一些内存数据库例如 `memcache` 或者 `redis` 等。下面我举一个简单的例子说明如何做。

```java
import java.util.Collection; 

 import org.springframework.cache.support.AbstractCacheManager; 

 public class MyCacheManager extends AbstractCacheManager { 
   private Collection<? extends MyCache> caches; 

   /** 
   * Specify the collection of Cache instances to use for this CacheManager. 
   */ 
   public void setCaches(Collection<? extends MyCache> caches) { 
     this.caches = caches; 
   } 

   @Override 
   protected Collection<? extends MyCache> loadCaches() { 
     return this.caches; 
   } 

 }
```

上面的自定义的 CacheManager 实际继承了 spring 内置的 AbstractCacheManager，实际上仅仅管理 MyCache 类的实例。

下面是MyCache的定义：

```java
import java.util.HashMap; 
 import java.util.Map; 

 import org.springframework.cache.Cache; 
 import org.springframework.cache.support.SimpleValueWrapper; 

 public class MyCache implements Cache { 
   private String name; 
   private Map<String,Account> store = new HashMap<String,Account>();; 

   public MyCache() { 
   } 

   public MyCache(String name) { 
     this.name = name; 
   } 

   @Override 
   public String getName() { 
     return name; 
   } 

   public void setName(String name) { 
     this.name = name; 
   } 

   @Override 
   public Object getNativeCache() { 
     return store; 
   } 

   @Override 
   public ValueWrapper get(Object key) { 
     ValueWrapper result = null; 
     Account thevalue = store.get(key); 
     if(thevalue!=null) { 
       thevalue.setPassword("from mycache:"+name); 
       result = new SimpleValueWrapper(thevalue); 
     } 
     return result; 
   } 

   @Override 
   public void put(Object key, Object value) { 
     Account thevalue = (Account)value; 
     store.put((String)key, thevalue); 
   } 

   @Override 
   public void evict(Object key) { 
   } 

   @Override 
   public void clear() { 
   } 
 }

```

上面的自定义缓存只实现了很简单的逻辑，但这是我们自己做的，也很令人激动是不是，主要看 get 和 put 方法，其中的 get 方法留了一个后门，即所有的从缓存查询返回的对象都将其 password 字段设置为一个特殊的值，这样我们等下就能演示“我们的缓存确实在起作用！”了。

这还不够，spring 还不知道我们写了这些东西，需要通过 spring*.xml 配置文件告诉它

```xml
 <cache:annotation-driven /> 

 <bean id="cacheManager" class="com.rollenholt.spring.cache.MyCacheManager">
     <property name="caches"> 
       <set> 
         <bean 
           class="com.rollenholt.spring.cache.MyCache"
           p:name="accountCache" /> 
       </set> 
     </property> 
   </bean> 

```

接下来我们来编写测试代码：

```java
Account account = accountService.getAccountByName("someone"); 
logger.info("passwd={}", account.getPassword()); 
account = accountService.getAccountByName("someone"); 
logger.info("passwd={}", account.getPassword()); 
```

上面的测试代码主要是先调用 getAccountByName 进行一次查询，这会调用数据库查询，然后缓存到 mycache 中，然后我打印密码，应该是空的；下面我再次查询 someone 的账号，这个时候会从 mycache 中返回缓存的实例，记得上面的后门么？我们修改了密码，所以这个时候打印的密码应该是一个特殊的值

## 注意和限制

### 基于 proxy 的 spring aop 带来的内部调用问题

上面介绍过 spring cache 的原理，即它是基于动态生成的 proxy 代理机制来对方法的调用进行切面，这里关键点是**对象的引用问题**.

如果对象的方法是内部调用（即 this 引用）而不是外部引用，则会导致 proxy 失效，那么我们的切面就失效，也就是说上面定义的各种注释包括 @Cacheable、@CachePut 和 @CacheEvict 都会失效，我们来演示一下。

```java
public Account getAccountByName2(String accountName) { 
   return this.getAccountByName(accountName); 
 } 

 @Cacheable(value="accountCache")// 使用了一个缓存名叫 accountCache 
 public Account getAccountByName(String accountName) { 
   // 方法内部实现不考虑缓存逻辑，直接实现业务
   return getFromDB(accountName); 
 }

```

上面我们定义了一个新的方法 getAccountByName2，其自身调用了 getAccountByName 方法，这个时候，发生的是内部调用（this），所以没有走 proxy，导致 spring cache 失效

要避免这个问题，就是要避免对缓存方法的内部调用，或者避免使用基于 proxy 的 AOP 模式，可以使用基于 aspectJ 的 AOP 模式来解决这个问题。

### @CacheEvict 的可靠性问题

我们看到，`@CacheEvict` 注释有一个属性 `beforeInvocation`，缺省为 false，即缺省情况下，都是在实际的方法执行完成后，才对缓存进行清空操作。期间如果执行方法出现异常，则会导致缓存清空不被执行。我们演示一下

```java
// 清空 accountCache 缓存
 @CacheEvict(value="accountCache",allEntries=true)
 public void reload() { 
   throw new RuntimeException(); 
 }

```

我们的测试代码如下：

```java
   accountService.getAccountByName("someone"); 
   accountService.getAccountByName("someone"); 
   try { 
     accountService.reload(); 
   } catch (Exception e) { 
    //...
   } 
   accountService.getAccountByName("someone"); 

```

注意上面的代码，我们在 reload 的时候抛出了运行期异常，这会导致清空缓存失败。上面的测试代码先查询了两次，然后 reload，然后再查询一次，结果应该是只有第一次查询走了数据库，其他两次查询都从缓存，第三次也走缓存因为 reload 失败了。

那么我们如何避免这个问题呢？我们可以用 @CacheEvict 注释提供的 beforeInvocation 属性，将其设置为 true，这样，在方法执行前我们的缓存就被清空了。可以确保缓存被清空。

### 非 public 方法问题

和内部调用问题类似，非 public 方法如果想实现基于注释的缓存，必须采用基于 AspectJ 的 AOP 机制

## Dummy CacheManager 的配置和作用

有的时候，我们在代码迁移、调试或者部署的时候，恰好没有 cache 容器，比如 memcache 还不具备条件，h2db 还没有装好等，如果这个时候你想调试代码，岂不是要疯掉？这里有一个办法，在不具备缓存条件的时候，在不改代码的情况下，禁用缓存。

方法就是修改 spring*.xml 配置文件，设置一个找不到缓存就不做任何操作的标志位，如下

```xml
   <cache:annotation-driven /> 

   <bean id="simpleCacheManager" class="org.springframework.cache.support.SimpleCacheManager"> 
     <property name="caches"> 
       <set> 
         <bean 
           class="org.springframework.cache.concurrent.ConcurrentMapCacheFactoryBean"
           p:name="default" /> 
       </set> 
     </property> 
   </bean> 

   <bean id="cacheManager" class="org.springframework.cache.support.CompositeCacheManager">
     <property name="cacheManagers"> 
       <list> 
         <ref bean="simpleCacheManager" /> 
       </list> 
     </property> 
     <property name="fallbackToNoOpCache" value="true" /> 
   </bean> 
```

注意以前的 cacheManager 变为了 simpleCacheManager，且没有配置 accountCache 实例，后面的 cacheManager 的实例是一个 CompositeCacheManager，他利用了前面的 simpleCacheManager 进行查询，如果查询不到，则根据标志位 fallbackToNoOpCache 来判断是否不做任何缓存操作。

## 使用 guava cache

```xml
<bean id="cacheManager" class="org.springframework.cache.guava.GuavaCacheManager">
    <property name="cacheSpecification" value="concurrencyLevel=4,expireAfterAccess=100s,expireAfterWrite=100s" />
    <property name="cacheNames">
        <list>
            <value>dictTableCache</value>
        </list>
    </property>
</bean>
```
