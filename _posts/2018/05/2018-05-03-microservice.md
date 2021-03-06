---
layout: post
title: 微服务理论与实践(二)
category: architecture
tags: [architecture]
---

## 微服务架构的六种模式


### 1.微服务架构模式方案

用Scale Cube方法设计应用架构，将应用服务按功能拆分成一组相互协作的服务。每个服务负责一组特定、相关的功能。每个服务可以有自己独立的数据库，从而保证与其他服务解耦。

### 1.1 聚合器微服务设计模式

![](https://img-blog.csdn.net/20161030233712795?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

聚合器调用多个服务实现应用程序所需的功能。它可以是一个简单的Web页面，将检索到的数据进行处理展示。它也可以是一个更高层次的组合微服务，对检索到的数据增加业务逻辑后进一步发布成一个新的微服务，这符合DRY原则。另外，每个服务都有自己的缓存和数据库。如果聚合器是一个组合服务，那么它也有自己的缓存和数据库。聚合器可以沿X轴和Z轴独立扩展。

### 1.2 代理微服务设计模式

![](https://img-blog.csdn.net/20161030233844639?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

这是聚合器模式的一个变种，在这种情况下，客户端并不聚合数据，但会根据业务需求的差别调用不同的微服务。代理可以仅仅委派请求，也可以进行数据转换工作。

### 1.3 链式微服务设计模式

![](https://img-blog.csdn.net/20161030233956922?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

这种模式在接收到请求后会产生一个经过合并的响应，在这种情况下，服务A接收到请求后会与服务B进行[通信](http://www.cctime.com/)，类似地，服务B会同服务C进行通信。所有服务都使用同步消息传递。在整个链式调用完成之前，客户端会一直阻塞。因此，服务调用链不宜过长，以免客户端长时间等待。

### 1.4 分支微服务设计模式

![](https://img-blog.csdn.net/20161030234126455?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

这种模式是聚合器模式的扩展，允许同时调用两个微服务链

### 1.5 数据共享微服务设计模式

![](https://img-blog.csdn.net/20161030234406969?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

自治是微服务的设计原则之一，就是说微服务是全栈式服务。但在重构现有的“单体应用（monolithic application）”时，SQL数据库反规范化可能会导致数据重复和不一致。因此，在单体应用到微服务架构的过渡阶段，可以使用这种设计模式

### 1.6 异步消息传递微服务设计模式

![](https://img-blog.csdn.net/20161030234556585?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

虽然REST设计模式非常流行，但它是同步的，会造成阻塞。因此部分基于微服务的架构可能会选择使用消息队列代替REST请求/响应


## 微服务之间的交互


Microservice架构模式中的“开”是各个服务的内部实现，而其中的“闭”则是各个服务之间相互沟通的方式

微服务必须使用进程间通信机制来交互。微服务架构有两类IPC机制可选，异步消息机制和同步请求/响应机制。当设计服务的通信模式时，需要考虑几个问题：服务如何交互，每个服务如何标识API，如何升级API，以及如何处理部分失败。

### 1.API GateWay 模式

**1.1 背景**

当决定将应用当作成一组微服务时，需要决定应用客户端如何与服务端交互。方式有两种：

* （1）客户端与服务端直接通讯

* （2）采用API Gateway，客户端与API Gateway交互，API Gateway封装具体的服务细节，并提供API给各个客户端。

**1.2客户端与服务端直接通讯**

一个客户端可以直接给多个微服务中的任何一个发起请求。每一个微服务都会有一个对外服务端。这个URL可能会映射到微服务的负载均衡上，它再转发请求道具体的节点上。

**缺点：**

* 客户端的需求量与微服务暴露的细粒度的API数量不匹配，客户端需要发起多个请求才能获取需与的完整数据。

* 客户端请求的微服务的协议可能不是web友好型的，例如微服务是RPC或AMQP协议的，他们都不是web友好型的

* 很难重构微服务

**1.3 API Gateway**

APIGateway是一个服务器，也是进入系统的唯一节点。API Gateway封装系统内部的架构，对每个客户端提供API。

**1.3.1 API Gateway的目标**

支持API接口动态发布及运营，包括但不限于：

* **（1）安全认证**

  * a. 应用申请审核通过后生成公钥，开放平台需提供支持分布式系统的密钥管理服务可设置为两个安全等级：需授权访问和无需授权访问（后者即任意客户端都可以发起调用），默认所有API都需授权访问。

  * b. 非正常状态（禁用、停用、黑名单等）的应用直接抛异常不允许访问——熔断机制

  * c. 调用次数、调用频率、并发数可运行时控制，避免某请求量过大影响其他应用的调用。

  * d. 可对某个应用某个API设置强制熔断，所有请求无视阀值直接抛出异常。

  * **（2）API管理**

所有API可后台查询管理，包括动态发布、参数映射配置、后端服务接口配置、API禁用、启用，多版本、分组、分级别等。

* **（3）应用管理**

后台管理开放平台接入的应用（第三方应用），包括查询、禁用、启用、审核。

* **（4）计费收费**

  * a. API的调用统计，每笔请求时间，响应时间，响应状态。

  * b. API的计费计算，按照请求量和请求资源计费，实现多种计费模型。（预付费，后收费。按量，按时间周期。）

* **（5）熔断**

* **（6）流量统计及限流**

  * 支持现有子系统RPC协议的API动态发布及运营，外部请求透传。

  * 支持json、xml响应报文，可以请求时选取所需报文格式。

  * 支持动态直接将后端SOA服务暴露为API。

  * 支持动态将普通Web接口暴露为API。

  * 支持动态将MQ服务暴露为API。

  * 支持多个服务组合编排后暴露为API。

  * 请求的转发、合成和协议转换

  * 给客户端提供一个定制化的API

**1.3.2 API Gateway模式的优缺点**

**（1）优点：**

  * 特定的API。使客户端只需跟Gateway打交道，减少客户端与服务端的通讯次数，也简化了客户端的代码。

  * 去报客户端不需要受服实例位置的影响

  * 为每套客户端提供最优的API

  * 降低请求/往返次数

**（2）缺点**

  * API Gateway是一个高可用的组件，必须要开发、部署和管理。开发者必须要更新API Gateway来提供新服务提供点来支持新暴露的微服务。

  * API Gateway会造成多余的网络跳转


## 服务注册与发现


### 1.背景

  * 服务的客户端（包括API网关或者其他服务）如何获取服务端实例的位置

  * 每个服务端实例都会在特定的位置（主机及端口）通过HTTP/REST或者Thrift等方式发布一个远程API

  * 服务端实例的具体数量和位置会发生动态变化

  * 虚拟机与容器通常会被分配动态IP地址

### 2.方案

### 2.1 客户端服务发现

![](https://img-blog.csdn.net/20161031000015056?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)


向某一服务发送请求时，客户端会通过查询Servcie Registry，即服务注册表来回去该服务的具体位置/该注册表包含全部服务的位置。

### 2.2 客户端服务发现的优缺点

**（1）优点**

* 相比服务端发现，客户端服务发现的活动部件与网络中转数量更少。

**（2）缺点**

* 需要为应用程序中使用的每种语言建立客户端发现逻辑

### 2.3 服务端服务发现

![](https://img-blog.csdn.net/20161031000124620?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)


向某一服务发送请求时，客户端会通知在已知位置的路由器（或者负载均衡器）发送请求。路由器会访问服务注册表，并向可用的服务实例转发该请求。服务注册表也可能被内建于路由器之中。

### 2.4 服务端发现的优缺点

**（1）优点**

* 相较客户端发现，其客户端代码无需实现服务发现功能，只需要向路由机制发送请求即可。

**（2）缺点**

*  除非成为云环境的一部分，否则该路由机制必须作为另一个系统组件进行安装与配置。为实现可用性和一定的接入能力，还需要为其配置一定数量的副本。

* 相较于客户端发现，服务端发现需要更多的网络跳转

### 3.服务注册表

### 3.1 背景

* 服务的客户端需要使用客户端发现机制或者服务端发现机制，获取需要发送请求的服务实例的位置

* 每个服务实例都会在特定位置（主机与端口）通过HTTP/REST或者Thrift等方式发布一个远程API。

* 服务实力的数量和位置会动态的发生变化。虚拟机与容器通常会被分配一个动态的IP地址。

### 3.2 方案

建立一套服务注册表。即一个包括服务、服务的实例和起位置的数据库。各服务示例需要在启动时注册至服务注册表，并在关闭的时候进行注销。该服务的客户端或者路由器通过查询此服务注册表获取可用的服务实例

### 3.3 服务注册表模式的优缺点

**（1）优点**

* 服务的客户端及路由器可以获取可用的服务实例的位置

**（2）缺点**

* 除非服务注册表被内置于基础设施，否则其必须做为另外的基础设施组件进行安装，配置及管理。

* 需要解决各服务示例如何注册至注册表，如何从注册表中注销

## 3.4 服务注册表的注册方式

### 3.4.1 需求

* 各服务实例必须在启动时注册到服务注册表，并在关闭时从服务注册表中注销

* 崩溃的服务实例必须从服务注册表中注销

* 在运行但无力处理请求的服务实例必须从服务注册表中注销

### 3.4.2 方案

#### 3.4.2.1 自注册

一项服务实例必须可以自动注册到服务注册表中。在启动时，该服务实例将自身（主机与IP地址）注册至服务注册表，使自身可被发现。客户端必须定期更新其注册信息，确保注册表获悉其仍处于运行状态。在关闭时，服务实例从服务注册表中自动注销。这一流程通常由微服务底盘框架实现。

### 3.4.2.2 自注册的优缺点

**（1）优点**

* 服务实例了解自身的状态，因而能够实现比启动/停止更为负责的状态模型。

**（2）缺点**

* 将服务与注册表耦合起来

* 需要为编写服务时使用的每种语言分别实现服务注册逻辑。

* 正在运行但无法处理请求的服务示例往往无法自动在服务注册表中进行自我注销

### 3.4.2.3 第三方注册

由第三方负责将服务和服务实例在服务注册表中的注册，并在服务启动的时候将服务实例注册到服务注册表。而在服务实例关闭的时候在服务注册表中注销

### 3.4.2.4 第三方注册的优缺点

**（1）优点**

* 与自注册相比，服务端无需实现自动注册逻辑，复杂度较低

* 注册工具可以对服务进行健康检查，并根据健康检查注册或者注销该实例

**（2）缺点**

* 第三方注册只能了解服务实例的一些表层状态，例如其是否正在运行，并不能判断正在运行的服务是否能够处理请求

* 除非该注册工具属于基础设施的一部分，否则我们需要对其进行安装、配置及维护。由于其属于关键部件，需要保证其高可用。


### 微服务 技术栈

构建微服务时，我们深深进入了分析分布式系统 - 一个已经研究了40年以上的技术主题，复杂的自适应系统理论已经深入人心有很长的时间。从技术的角度来看，我们需要解决的事情如下，这也是我们进来要深入研究的微服务领域的技术栈：

**（1）部署**

**（2）交付**

**（3）API**

**（4）版本控制**

**（5）合同**

**（6）缩放/自动缩放**

**（7）服务发现**

**（8）负载均衡**

**（9）路由/自适应路由**

**（10）健康检查**

**（11）配置**

**（12）熔断器**

**（13）bulk-heads**

**（14）TTL / deadlining**

**（15）延迟跟踪**

**（16）服务因果跟踪**

**（17）分布式日志**

**（18）度量操作与收集**

![](https://img-blog.csdn.net/20170316000241766?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvc3VuaHVpbGlhbmc4NQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
