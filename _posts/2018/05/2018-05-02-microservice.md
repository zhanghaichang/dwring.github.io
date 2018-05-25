---
layout: post
title: 微服务理论与实践(一)
category: architecture
tags: [architecture]
---

微服务最早由Martin Fowler与James Lewis于2014年共同提出，微服务架构风格是一种使用一套小服务来开发单个应用的方式途径，每个服务运行在自己的进程中，并使用轻量级机制通信，通常是HTTP API，这些服务基于业务能力构建，并能够通过自动化部署机制来独立部署，这些服务使用不同的编程语言实现，以及不同数据存储技术，并保持最低限度的集中式管理。


## 架构的背景及需求


### 一.背景

# ![](https://img-blog.csdn.net/20161030232937464?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

业务架构是战略，应用架构是战术，技术架构是装备。

在开发服务端企业应用时，需要支持各种客户段，包括PC桌面浏览器，移动浏览器及原生移动应用，应用还需要向第三方提供可访问的API，并通过WebSevice或者消息代理与其他应用进行集成。应用通过业务逻辑，访问数据库，与其他服务交换信息，并返回一条HTML/XML/JSON响应，来处理请求。

应用采用多层架构或六角架构，主要由以下不同组建组成：

* **展现组件:** 负责处理http请求，并响应html或者JSON/XML

* **业务逻辑:** 应用的业务逻辑

* **数据库访问逻辑:** 用于访问数据库的数据访问对象

* **应用集成逻辑:** 消息层，例如Spring Integration

### 二.应用的部署架构需求是什么？

* 应用需要由一个开发者团队专门负责

* 团队新成员可以快速上手，完成开发任务

* 应用可以很容易的进行理解和修改

* 对应用能够进行持续的部署

* 需要在多台机器上部署应用的副本，从而保证应用的可用性和可扩展性的要求

* 可以使用各种新技术



## 单体架构模式


### 1.单体架构模式方案

![](https://img-blog.csdn.net/20161030233449747?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

（1） 单个java WAR文件

（2） 单个Rails或者NodeJS代码目录层级

### 2.单体架构模式的优缺点

#### （1）优点:

* **为人所熟知：** 现有的大部分工具、应用服务器、框架和脚本都是这种应用程序；

* **IDE友好：** 像NetBeans、Eclipse、IntelliJ这些开发环境都是针对开发、部署、调试这样的单个应用而设计的；

* **易于测试：** 单体应用一旦部署，所有的服务或特性就都可以使用了，这简化了测试过程，因为没有额外的依赖，每项测试都可以在部署完成后立刻开始；

* **容易部署：** 只需将单个归档文件复制到单个目录下。

#### （2）缺点

* **代码库巨大**

* **过载的IDE** 代码库越大，IDE性能越差，影响开发者效率

* **过载的Web容器** 应用越大，服务的启动时间越长，影响服务的开发和部署速度

* **持续部署苦难** 巨大的单体服务本身就是频繁部署的一大障碍，部署时间长，为了部署一个组件往往要部署全部应用。导致不必要的服务停机，并增加部署的风险

* **应用扩展困难** 单体服务只能进行一维伸缩

* **难于进行模块化开发** 单体服务由于模块无法划分是规模化开发的障碍，无法做到一个团队负责一个模块。

* **需要长期关注同一套技术栈** 单体架构迫使我们长期使用在开发初期选定的技术堆栈（在某些情况下，可能是某些技术的特定版本）。单体应用是渐进采用新技术的障碍



## 微服务架构的基本能力和优缺点


### 1.微服务架构模式方案

微服务架构采用Scale Cube方法设计应用架构，将应用服务按功能拆分成一组相互协作的服务。每个服务负责一组特定、相关的功能。每个服务可以有自己独立的数据库，从而保证与其他服务解耦。

### 2.微服务架构的基本能力

![](https://img-blog.csdn.net/20161030235025738?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

#### 2.1 Restful 轻量级通讯的首选方式

在微服务架构下，推崇使用轻量级的方式进行通讯。我们选择Restful的进行通讯。每个微服务都统一对外提供rest服务。无论前端调用后端服务还是后端之间的服务调用都统一走restful，这样就统一了协议栈。微服务架构可以支持各种异构系统服务间的交互。

#### 2.2 RPC通讯

统一的RPC框架是服务化首先要解决的问题，它为团队提供了一整套序列化，反序列化，网络框架，连接池，线程池超时处理等业务处理之外的能力。目前的RPC框架包括dubbo/dubbox，motan，thrift，grpc，Karyon/Ribbon



#### 2.3 服务的注册与发现

服务之间需要创建一种服务发现机制，用于帮助服务之间互相感知彼此的存在。服务启动时会将自身的服务信息注册到注册中心，并订阅自己需要消费的服务。

![](https://img-blog.csdn.net/20161030235145935?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

#### 2.4 负载均衡

服务高可用的保证手段，为了保证高可用，每一个微服务都需要部署多个服务实例来提供服务。此时客户端进行服务的负载均衡。

##### 2.4.1负载均衡的常见策略

##### 2.4.1.1随机

把来自网络的请求随机分配给内部中的多个服务器。

##### 2.4.1.2轮询

每一个来自网络中的请求，轮流分配给内部的服务器，从1到N然后重新开始。此种负载均衡算法适合服务器组内部的服务器都具有相同的配置并且平均服务请求相对均衡的情况。

##### 2.4.1.3加权轮询

根据服务器的不同处理能力，给每个服务器分配不同的权值，使其能够接受相应权值数的服务请求。例如：服务器A的权值被设计成1，B的权值是3，C的权值是6，则服务器A、B、C将分别接受到10%、30％、60％的服务请求。此种均衡算法能确保高性能的服务器得到更多的使用率，避免低性能的服务器负载过重。

##### 2.4.1.4 IP Hash

这种方式通过生成请求源IP的哈希值，并通过这个哈希值来找到正确的真实服务器。这意味着对于同一主机来说他对应的服务器总是相同。使用这种方式，你不需要保存任何源IP。但是需要注意，这种方式可能导致服务器负载不平衡。

##### 2.4.1.5最少连接数

客户端的每一次请求服务在服务器停留的时间可能会有较大的差异，随着工作时间加长，如果采用简单的轮循或随机均衡算法，每一台服务器上的连接进程可能会产生极大的不同，并没有达到真正的负载均衡。最少连接数均衡算法对内部中需负载的每一台服务器都有一个数据记录，记录当前该服务器正在处理的连接数量，当有新的服务连接请求时，将把当前请求分配给连接数最少的服务器，使均衡更加符合实际情况，负载更加均衡。此种均衡算法适合长时处理的请求服务，如FTP。

#### 2.5 容错

在调用服务集群时，如果一个微服务调用异常，如超时，连接异常，网络异常等，则根据容错策略进行服务容错。目前支持的服务容错策略有快速失败，失效切换。如果连续失败多次则直接熔断，不再发起调用。这样可以避免一个服务异常拖垮所有依赖于他的服务。

#### 2.5.1 容错策略

##### 2.5.1.1 快速失败

服务只发起一次待用，失败立即报错。通常用于非幂等下性的写操作

##### 2.5.1.2 失效切换

服务发起调用，当出现失败后，重试其他服务器。通常用于读操作，但重试会带来更长时间的延迟。重试的次数通常是可以设置的

##### 2.5.1.3 失败安全

失败安全， 当服务调用出现异常时，直接忽略。通常用于写入日志等操作。

##### 2.5.1.4 失败自动恢复

当服务调用出现异常时，记录失败请求，定时重发。通常用于消息通知。

##### 2.5.1.5 forking Cluster

并行调用多个服务器，只要有一个成功，即返回。通常用于实时性较高的读操作。可以通过forks=n来设置最大并行数。

##### 2.5.1.6 广播调用

广播调用所有提供者，逐个调用，任何一台失败则失败。通常用于通知所有提供者更新缓存或日志等本地资源信息。

#### 2.5.2 熔断

有一些异常比较顽固，突然发生，无法预测，而且很难恢复，并且还会导致级联失败（举个例子，假设一个服务集群的负载非常高，如果这时候集群的一部分挂掉了，还占了很大一部分资源，整个集群都有可能遭殃）。如果我们这时还是不断进行重试的话，结果大多都是失败的。因此，此时我们的应用需要立即进入失败状态(fast-fail)，并采取合适的方法进行恢复。


##### 2.5.2.1 Circuit Breaker（熔断器）原理

![](https://img-blog.csdn.net/20161030235447647?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

我们可以用状态机来实现CircuitBreaker，它有以下三种状态：

* 关闭( **Closed** )：默认情况下Circuit Breaker是关闭的，此时允许操作执行。CircuitBreaker内部记录着最近失败的次数，如果对应的操作执行失败，次数就会续一次。如果在某个时间段内，失败次数（或者失败比率）达到阈值，CircuitBreaker会转换到开启( **Open** )状态。在开启状态中，Circuit Breaker会启用一个超时计时器，设这个计时器的目的是给集群相应的时间来恢复故障。当计时器时间到的时候，CircuitBreaker会转换到半开启( **Half-Open** )状态。

* 开启( **Open** )：在此状态下，执行对应的操作将会立即失败并且立即抛出异常。

* 半开启( **Half-Open** )：在此状态下，Circuit Breaker会允许执行一定数量的操作。如果所有操作全部成功，CircuitBreaker就会假定故障已经恢复，它就会转换到关闭状态，并且重置失败次数。如果其中  **任意一次**  操作失败了，Circuit Breaker就会认为故障仍然存在，所以它会转换到开启状态并再次开启计时器（再给系统一些时间使其从失败中恢复）

#### 2.5.3 Dubbo的集群容错

当我们的系统中用到Dubbo的集群环境,因为各种原因在集群调用失败时，Dubbo提供了多种容错方案，缺省为failover重试。

```
通过以下方式设置

<dubbo:servicecluster="failsafe"/>

或：

<dubbo:referencecluster="failsafe"/>

集群容错模式：

FailoverCluster

失败自动切换，当出现失败，重试其它服务器。(缺省)

通常用于读操作，但重试会带来更长延迟。

可通过retries="2"来设置重试次数(不含第一次)。正是文章刚开始说的那种情况.

FailfastCluster

快速失败，只发起一次调用，失败立即报错。

通常用于非幂等性的写操作，比如新增记录。

FailsafeCluster

失败安全，出现异常时，直接忽略。

通常用于写入审计日志等操作。

FailbackCluster

失败自动恢复，后台记录失败请求，定时重发。

通常用于消息通知操作。

Forking Cluster

并行调用多个服务器，只要一个成功即返回。

通常用于实时性要求较高的读操作，但需要浪费更多服务资源。

可通过forks="2"来设置最大并行数。

BroadcastCluster

广播调用所有提供者，逐个调用，任意一台报错则报错。(2.1.0开始支持)

通常用于通知所有提供者更新缓存或日志等本地资源信息。

重试次数配置如：(failover集群模式生效)
```

### 2.6 限流和降级

保证核心服务的稳定性。为了保证核心服务的稳定性，随着访问量的不断增加，需要为系统能够处理的服务数量设置一个极限阀值，超过这个阀值的请求则直接拒绝。同时，为了保证核心服务的可用，可以对否些非核心服务进行降级，通过限制服务的最大访问量进行限流，通过管理控制台对单个微服务进行人工降级

### 2.7 SLA

SLA：Service-LevelAgreement的缩写，意思是服务等级协议。是关于[网络服务](http://baike.baidu.com/view/1279152.htm)供应商和客户间的一份合同，其中定义了[服务类型](http://baike.baidu.com/view/3872009.htm)、服务质量和客户付款等术语。

典型的SLA包括以下项目：

* 分配给客户的最小[带宽](http://baike.baidu.com/view/10821.htm)；

* 客户带宽极限；

* 能同时服务的客户数目；

* 在可能影响用户行为的网络变化之前的通知安排；

* 拨入访问可用性；

* 运用统计学；

* 服务供应商支持的最小网络利用性能，如99.9%[有效工作时间](http://baike.baidu.com/view/1588784.htm)或每天最多为1分钟的停机时间；

* 各类客户的流量优先权；

* 客户技术支持和服务；

* 惩罚规定，为服务供应商不能满足 SLA需求所指定。

### 3.微服务架构模式的优缺点

**（1）优点**

* 1.每个微服务相对较小 开发者易于理解 IDE处理效率快，利于提高劳动生产率，Web容器压力小，容器启动速度快，易于提供劳动生产率和生产环境部署速度

* 2.每个微服务都可以独立部署，简化了部署新服务版本的流程

* 3.易于规模化开发，多个开发团队可以并行开发，每个团队负责一项服务

* 4.改善故障隔离。一个服务宕机不会影响其他的服务

* 5.每个服务可以独立的进行开发和部署

* 6.无需长期使用同一套技术栈

**（2）缺点**

* 1.开发者需要应对创建分布式系统所产生的额外的复杂因素，目前的IDE主要面对的是单体工程程序，无法显示支持分布式应用的开发，测试工作更加困难，需要采用服务间的通讯机制，很难在不采用分布式事务的情况下跨服务实现功能，跨服务实现要求功能要求团队之间的紧密协作

* 2.部署复杂

* 3.内存占用量更高