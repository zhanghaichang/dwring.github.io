---
layout: post
title: Java 异步回调机制实例解析
category: java
tags: [java]
---

什么是回调？今天傻傻地截了张图问了下，然后被陈大牛回答道“就一个回调…”。此时千万个草泥马飞奔而过

哈哈，看着源码，享受着这种回调在代码上的作用，真是美哉。不妨总结总结。

## 一、什么是回调

回调，回调。要先有调用，才有调用者和被调用者之间的回调。所以在百度百科中是这样的：

> 软件模块之间总是存在着一定的接口，从调用方式上，可以把他们分为三类：同步调用、回调和异步调用。

回调是一种特殊的调用，至于三种方式也有点不同。

1、同步回调，即阻塞，单向。

2、回调，即双向（类似自行车的两个齿轮）。

3、异步调用，即通过异步消息进行通知。

## 二、CS中的异步回调（java案例）

比如这里模拟个场景：客户端发送msg给服务端，服务端处理后（5秒），回调给客户端，告知处理成功。代码如下：

回调接口类：

```java
/**
 * @author Jeff Lee
 * @since 2015-10-21 21:34:21
 * 回调模式-回调接口类
 */
public interface CSCallBack {
    public void process(String status);
}
```

模拟客户端：

```java
/**
 * @author Jeff Lee
 * @since 2015-10-21 21:25:14
 * 回调模式-模拟客户端类
 */
public class Client implements CSCallBack {

    private Server server;

    public Client(Server server) {
        this.server = server;
    }

    public void sendMsg(final String msg){
        System.out.println("客户端：发送的消息为：" + msg);
        new Thread(new Runnable() {
            @Override
            public void run() {
                server.getClientMsg(Client.this,msg);
            }
        }).start();
        System.out.println("客户端：异步发送成功");
    }

    @Override
    public void process(String status) {
        System.out.println("客户端：服务端回调状态为：" + status);
    }
}
```

模拟服务端:

```java
/**
 * @author Jeff Lee
 * @since 2015-10-21 21:24:15
 * 回调模式-模拟服务端类
 */
public class Server {

    public void getClientMsg(CSCallBack csCallBack , String msg) {
        System.out.println("服务端：服务端接收到客户端发送的消息为:" + msg);

        // 模拟服务端需要对数据处理
        try {
            Thread.sleep(5 * 1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("服务端:数据处理成功，返回成功状态 200");
        String status = "200";
        csCallBack.process(status);
    }
}
```

测试类：

```java
/**
 * @author Jeff Lee
 * @since 2015-10-21 21:24:15
 * 回调模式-测试类
 */
public class CallBackTest {
    public static void main(String[] args) {
        Server server = new Server();
        Client client = new Client(server);

        client.sendMsg("Server,Hello~");
    }
}
```

运行下测试类 — 打印结果如下：

> 客户端：发送的消息为：Server,Hello~
> 客户端：异步发送成功
> 服务端：服务端接收到客户端发送的消息为:Server,Hello~
> 
> （这里模拟服务端对数据处理时间，等待5秒）
> 服务端：数据处理成功，返回成功状态 200
> 客户端：服务端回调状态为：200

一步一步分析下代码，核心总结如下

> 1、接口作为方法参数，其实际传入引用指向的是实现类
> 
> 2、Client的sendMsg方法中，参数为final，因为要被内部类一个新的线程可以使用。这里就体现了异步。
> 
> 3、调用server的getClientMsg()，参数传入了Client本身（对应第一点）。

还有值得一提的是

— 开源代码都在我的[gitHub](https://github.com/JeffLi1993)上哦~

## 三、回调的应用场景

回调目前运用在什么场景比较多呢？从操作系统到开发者调用：

> 1、Windows平台的消息机制
> 
> 2、异步调用微信接口，根据微信返回状态对出业务逻辑响应。
> 
> 3、Servlet中的Filter(过滤器)是基于回调函数，需容器支持。

补充：其中 Filter(过滤器)和Interceptor(拦截器)的区别，拦截器基于是Java的反射机制，和容器无关。但与回调机制有异曲同工之妙。

总之，这设计让底层代码调用高层定义（实现层）的子程序，增强了程序的灵活性。

## 四、模式对比

上面讲了Filter和Intercepter有着异曲同工之妙。其实接口回调机制和一种<span class="wp_keywordlink">[设计模式](http://www.codeceo.com/article/category/develop/design-patterns "设计模式")</span>—观察者模式也有相似之处：

观察者模式：

GOF说道 — “定义对象的一种一对多的依赖关系，当一个对象的状态发送改变的时候，所有对他依赖的对象都被通知到并更新。”它是一种模式，是通过接口回调的方法实现的，即它是一种回调的体现。

接口回调：

与观察者模式的区别是，它是种原理，而非具体实现。

## 五、心得

总结四步走：

> 机制，即是原理。
> 
> 模式，即是体现。
> 
> 记住具体场景，常见模式。
> 
> 然后深入理解原理。
