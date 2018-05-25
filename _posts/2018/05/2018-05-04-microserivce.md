---
layout: post
title: 微服务架构参考技术栈
category: architecture
tags: [architecture]
---

近年，Spring Cloud俨然已经成为微服务开发的主流技术栈，在国内开发者社区非常火爆。

我近年一直在一线互联网公司（携程，拍拍贷等）开展微服务架构实践，根据我个人的一线实践经验和我平时对Spring Cloud的调研，我认为Spring Cloud技术栈中的有些组件离生产级开发尚有一定距离。

比方说Spring Cloud Config和Spring Cloud Sleuth都是Pivotal自研产品，尚未得到大规模企业级生产应用，很多企业级特性缺失（具体见我后文描述）。另外Spring Cloud体系还缺失一些关键的微服务基础组件，比如Metrics监控，健康检查和告警等。
所以我在参考Spring Cloud微服务技术栈的基础上，结合自身的实战落地经验，也结合国内外一线互联网公司（例如Netflix，点评，携程，Zalando等）的开源实践，综合提出更贴近国内技术文化特色的轻量级的微服务参考技术栈。希望这个参考技术栈对一线的架构师（或者是初创公司）有一个好的指导，能够少走弯路，快速落地微服务架构。

这个参考技术栈和总体架构如下图所示：

![总体架构](https://upload-images.jianshu.io/upload_images/8122772-7dcd40768ee78fc9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

主要包含11大核心组件，分别是：

##### 核心支撑组件

1.  服务网关Zuul
2.  服务注册发现Eureka+Ribbon
3.  服务配置中心Apollo
4.  认证授权中心Spring Security OAuth
5.  服务框架Spring MVC/Boot

##### 监控反馈组件

1.  数据总线Kafka
2.  日志监控ELK
3.  调用链监控CAT
4.  Metrics监控KairosDB
5.  健康检查和告警ZMon
6.  限流熔断和流聚合Hystrix/Turbine

#### 一、核心支撑组件

##### 服务网关Zuul

```
2013年左右，infoq曾经对前Netflix架构总监Adrian Cockcroft有过一次专访[附录1]，其中有问Adrian：“Netflix开源这么多项目，你认为哪一个是最不可或缺的(MOST Indispensable)”，Adrian回答说：“在NetflixOSS开源项目中，有一个容易被忽略，但是Netflix最强大的基础服务之一，它就是Zuul网关服务。Zuul网关主要用于智能路由，同时也支持认证，区域和内容感知路由，将多个底层服务聚合成统一的对外API。Zuul网关的一大亮点是动态可编程，配置可以秒级生效”。从Adrian的回答中，我们可以感受到Zuul网关对微服务基础架构的重要性。

```



![服务网关](https://upload-images.jianshu.io/upload_images/8122772-50bfe9125907f38e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/411)



Zuul在英文中是一种怪兽，星际争霸中虫族里头也有Zuul，Netflix为网关起名Zuul，寓意看门神兽

Zuul网关在Netflix经过生产级验证，在纳入Spring Cloud体系之后，在社区中也有众多成功的应用。Zuul网关在携程（日流量超50亿），拍拍贷等公司也有成功的落地实践，是微服务基础架构中网关一块的首选。其它开源产品像Kong或者Nginx等也可以改造支持网关功能，但是较复杂门槛高一点。

Zuul网关虽然不完全支持异步，但是同步模型反而使它简单轻量，易于编程和扩展，当然同步模型需要做好限流熔断（和限流熔断组件Hystrix配合），否则可能造成资源耗尽甚至雪崩效应（cascading failure）。

##### 服务注册发现Eureka + Ribbon

```
针对微服务注册发现场景，社区里头的开源产品当中，经过生产级大流量验证的，目前只有Netflix Eureka一个，它也已经纳入Spring Cloud体系，在社区中有众多成功应用，例如携程Apollo配置中心也是使用Eureka做软负载。其它产品如Zookeeper/Etcd/Consul等，都是比较通用的产品，还需要进一步封装定制才可生产级使用。Eureka支持跨数据中心高可用，但它是AP最终一致系统，不是强一致性系统。

```


![Eureka + Ribbon](https://upload-images.jianshu.io/upload_images/8122772-4b01b56c506cae08.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)



Eureka是阿基米德洗澡时发现浮力原理时发出的惊叹声，在微服务中寓意发现

Ribbon是可以和Eureka配套对接的客户端软负载库，在Eureka的配合下能够支持多种灵活的动态路由和负载均衡策略。内部微服务直连可以直接走Ribbon客户端软负载，网关上也可以部署Ribbon，这时网关相当于一个具有路由和软负载能力的超级客户端。

##### 服务配置中心Apollo

```
Spring Cloud体系里头有个Spring Cloud Config产品，但是功能远远达不到生产级，只能小规模场景下用，中大规模企业级场景不建议采用。携程框架研发部开源的Apollo是一款在携程和其它众多互联网公司生产落地下来的产品，开源两年多，目前在github上有超过4k星，非常成功，文档齐全也是它的一大亮点，推荐作为企业级的配置中心产品。Apollo支持完善的管理界面，支持多环境，配置变更实时生效，权限和配置审计等多种生产级功能。Apollo既可以用于连接字符串等常规配置场景，也可用于发布开关（Feature Flag）和业务配置等高级场景。在《微服务架构实战160讲》课程中，第二个模块就配置中心相关主题，会深度剖析携程Apollo的架构和实践。

```

![服务配置中心Apollo](https://upload-images.jianshu.io/upload_images/8122772-b1538341e565dbaa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)


##### 认证授权中心Spring Security OAuth

目前开源社区还没有特别成熟的微服务安全认证中心产品，之前我工作过的一些中大型互联网公司，比如携程，唯品会等，在这一块基本都是定制自研的，但是对一般企业来说，定制自研还是有门槛的。OAuth2是一种基于令牌Token的授权框架，已经得到众多大厂（Google, Facebook, Twitter, Microsoft等）的支持，可以认为是事实上的微服务安全协议标准，适用于开放平台联合登录，现代微服务安全（包括单页浏览器App/无线原生App/服务器端WebApp接入微服务，以及微服务之间调用等场景），和企业内部应用认证授权(IAM/SSO)等多种场景。

Spring Security OAuth2是Spring Security基础上的一个扩展，支持四种主要的OAuth2 Flows，基本可以作为微服务认证授权中心的推荐产品。但是Spring Security OAuth2还只是一个框架，不是一个端到端的开箱即用的产品，企业级应用仍需在其上进行定制，例如提供Web端管理界面，对接企业内部的用户认证登录系统，使用Cache缓存令牌，和微服务网关对接等，才能作为生产级使用。


![Spring Security OAuth](https://upload-images.jianshu.io/upload_images/8122772-69a7d50a2da5d71f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/468)


Spring Security OAuth2 是 Spring Security框架的一个扩展

##### 服务框架Spring/Boot

Spring可以说是史上最成功的Web App/API开发框架之一，它融入了Java社区中多年来沉淀下来的最佳实践，虽然有将近15年历史，但目前的社区活跃度仍呈上升趋势。Spring Boot在Spring的基础上进一步打包封装，提供更贴心的Starter工程，自启动能力，自动依赖管理，基于代码的配置等特性进一步降低接入门槛。另外Spring Boot也提供actuator这样的生产级监控特性，支持DevOps研发模式，它是微服务开发框架的推荐首选。

REST契约规范Swagger和Spring有比较好的集成，使得Spring也支持契约驱动开发(Contract Driven Development)模型。对于一些中大规模的企业，如果业务复杂团队较多，考虑到互操作性和集成成本，建议采用契约驱动开发模型，也就是开发时先定义Swagger契约，然后再通过契约生成服务端接口和客户端，再实现服务端业务逻辑，这种开发模型能够标准化接口，降低系统间集成成本，对于多团队协同并行开发非常重要。


#### 二、监控反馈组件

##### 数据总线Kafka

最初由Linkedin研发并在其内部大规模成功应用，然后在Apache上开源的Kafka，是业内数据总线(Databus)一块的标配，几乎每一家互联网公司都可以看到Kafka的身影。Kafka堪称开源项目的一个经典成功案例，其创始人团队从Linkedin离职后还专门成立了一家叫confluent的企业软件服务公司，围绕Kafka周边提供配套和增值服务。在监控一块，日志和Metrics等数据可以通过Kafka做收集、存储和转发，相当于中间增加了一个大容量缓冲，能够应对海量日志数据的场景。除了日志监控数据收集，Kafka在业务大数据分析，IoT等场景都有广泛应用。如果对Kafka进行适当定制增强，还可以用于传统消息中间件场景。

Kafka的特性是大容量，高吞吐，高可用，数据可重复消费，可水平扩展，支持消费者组等。Kafka尤其适用于不严格要求实时和不丢数据的大数据日志场景。


##### 日志监控ELK

ELK（ElasticSearch/Logstash/Kibana）是日志监控一块的标配技术栈，几乎每一家互联网公司都可以看到ELK的身影，据称携程是国内ELK的最大用户，每日增量日志数据量达到80~90TB。ELK已经非常成熟，基本上是开箱即用，后续主要的工作在运维、治理和调优。ELK一般和Kafka配套使用，因为日志分词操作还是比较耗时的，Kafka主要作为前置缓冲，起到流量消峰作用，抵消日志流量高峰和消费（分词建索引）的不匹配问题。一旦反向索引建立，日志检索是非常快的，所以日志检索快和灵活是ElasticSearch的最大亮点。另外ELK还有大容量，高吞吐，高可用，可水平扩容等企业级特性。

创业公司起步期，考虑到资源时间限制，调用链监控和Metrics监控可以不是第一优先级，但是ELK是必须搭一套的，应用日志数据一定要收集并建立索引，基本能够覆盖大部分Trouble Shooting场景（业务，性能，程序bug等）。另外用好ELK的关键是治理，需要制定一些规则（比如只收集Warn级别以上日志），对应用的日志数据量做好监控，否则开发人员会滥用，什么垃圾数据都往ELK里头丢，造成大量空间被浪费，严重的还可能造成性能可用性问题。

![日志监控ELK](https://upload-images.jianshu.io/upload_images/8122772-19073045dcfe6500.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/631)


##### 调用链监控CAT

Spring Cloud支持基于Zipkin的调用链监控，我个人基于实践经验认为Zipkin还不能算一款企业级调用链监控产品，充其量只能算是一个半成品，很多重要的企业级特性缺失。Zipkin最早是由Twitter在消化Google Dapper论文的基础上研发，在Twitter内部有较成功应用，但是在开源出来的时候把不少重要的统计报表功能给阉割了（因为依赖于一些比较重的大数据分析平台），只是开源了一个半成品，能简单查询和呈现可视化调用链，但是细粒度的调用性能数据报表没有开源。

Google大致在2007年左右开始研发称为Dapper的调用链监控系统，但在远远早于这个时间（大致在2002左右），eBay就已经有了自己的调用链监控系统CAL（Centralized Application Logging），Google和eBay的设计思路大致相同，但是也有一些差别。CAL在eBay有大规模成功应用，被称为是eBay的四大神器之一（另外三个是DAL，Messaging和SOA）。开源调用链监控系统CAT的作者吴其敏（我曾经和他同事，习惯叫他老吴），曾经在eBay工作近十年，期间深入消化吸收了CAL的设计。2011年后老吴离开eBay去了点评，用三年时间在点评再造了一款调用链监控产品CAT（Centralized Application Tracking），CAT具有CAL的基因和影子，同时也融入了老吴在点评的探索实践和创新。

CAT是一款更完整的企业级调用链监控产品，甚至已经接近一个APM（Application Performance Management）产品的范畴，它不仅支持调用链的查询和可视化，还支持细粒度的调用性能数据统计报表，这块是CAT和市面上其它开源调用链监控产品最本质的差异点，实际上开发人员大部分时间用CAT是看性能统计报表（主要是CAT的Transaction和Problem报表），这些报表相当于给了开发人员一把尺子，可以自助测量并持续改进应用性能。另外CAT还支持应用报错大盘，自助告警等功能，也是企业级监控非常实用的功能。

CAT在点评，携程，陆金所，拍拍贷等公司有成功落地案例，因为是国产调用链监控产品，界面展示和功能等更契合国内文化，更易于在国内公司落地。个人推荐CAT作为微服务调用链监控的首选。至于社区里头有人提到CAT的侵入性问题，我觉得是要一分为二看，有利有弊，有耦合性但是性能更好，一般企业中基础架构团队会使用CAT统一为基础组件埋点，开发人员一般不用自己埋点；另外企业用了一款调用链监控产品以后，一般是不会换的，开发人员用习惯就好了，侵入不是大问题。

![调用链监控CAT](https://upload-images.jianshu.io/upload_images/8122772-1d25d6bfcf294aa0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)


CAT的Transaction报表

##### Metrics监控KariosDB

除了日志和调用链，Metrics也是应用监控的重要关注点。互联网应用提倡度量驱动开发（Metrics Driven Development），也就是说开发人员不仅要关注功能实现，做好单元测试（TDD），还要做好业务层（例如注册，登录和下单数等）和应用层（例如调用数，调用延迟等）的监控埋点，这个也是DevOps（开发即运维）理念的体现，DevOps要求开发人员必须关注运维需求，监控埋点是一种生产级运维需求。

Metrics监控产品底层依赖于时间序列数据库（TSDB），最近比较热的开源产品有Prometheus和InfluxDB，社区用户数量和反馈都不错，可以采纳。但是这些产品分布式能力比较弱，定制扩展门槛比较高，一般建议刚起步量不大的公司采用。如果企业业务和团队规模发展到一定阶段，建议考虑支持分布式能力的时间序列监控产品，例如KairosDB或者OpenTSDB，我本人对这两款产品都有一些实践经验，KariosDB基于Cassandra，相对更轻量一点，建议中大规模公司采用，如果你们公司已经采用Hadoop/HBase，则OpenTSDB也是不错选择。

KairosDB一般也和Kafka配套使用，Kafka作为前置缓冲。另外注意使用KariosDB打点的话tag的值不能太离散，否则会有查询性能问题，这个和KariosDB底层存储结构有关系。Grafana是Metrics展示标配，可以和KariosDB无缝集成。

![Metrics监控KariosDB](https://upload-images.jianshu.io/upload_images/8122772-672ba043cadcc84a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

Grafana是Metrics展示标配，和主流时间序列数据库都可以集成

##### 健康检查和告警ZMon

除了上述监控手段，我们仍需要健康检查和告警系统作为配套的监控手段。ZMon是德国电商公司Zalando开源的一款健康检查和告警平台，具备强大灵活的监控告警能力。ZMon本质上可以认为是一套分布式监控任务调度平台，它提供众多的Check脚本（也可以自己再定制扩展），能够对各种硬件资源或者目标服务（例如HTTP端口，Spring的Actuator端点，KariosDB中的Metrics，ELK中的错误日志等等）进行定期的健康检查和告警，它的告警逻辑和策略采用Python脚本实现，开发人员可以实现自助式告警。ZMon同时适用于系统，应用，业务，甚至端用户体验层的监控和告警。



![健康检查和告警ZMon](https://upload-images.jianshu.io/upload_images/8122772-cdcbfa9ce1b9ca7e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)



ZMon分布式监控告警系统架构，底层基于KairosDB时间序列数据库

##### 限流熔断和流聚合Hystrix+Turbine

2010年左右，Netflix也饱受分布式微服务系统中雪崩效应（Cascading Failure）的困扰，于是专门启动了一个叫做弹性工程的项目来解决这个问题，Hystrix就是弹性工程最终落地下来的一个产品。Hystrix在Netflix微服务系统中大规模推广应用后，雪崩效应问题基本得到解决，整个体统更具弹性。之后Netflix把Hystrix开源贡献给了社区，短期获得社区的大量正面反馈，目前Hystrix在github上有超过1.3万颗星，据说支持奥巴马总统选举的系统也曾使用Hystrix进行限流熔断保护[参考附录2]，可见限流熔断是分布式系统稳定性的强需求，Netflix很好的抓住了这个需求并给出了经过生产级验证的解决方案。Hystrix已经被纳入Spring Cloud体系，它是Java社区中限流熔断组件的首选（目前还看不到第二个更好的产品）。

Turbine是和Hystrix配套的一个流聚合服务，能够对Hystrix监控数据流进行聚合，聚合以后可以在Hystrix Dashboard上看到集群的流量和性能情况。


![Hystrix+Turbine](https://upload-images.jianshu.io/upload_images/8122772-f158a5c850699ba9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)



Hystrix在英文中是豪猪兽的意思，豪猪兽通过身上的刺保护自己，Netflix为限流熔断组件起名Hystrix，寓意Hystrix能够保护微服务调用。

#### 总结

1.  技术栈没有好坏之分，只有适合一说。本文推荐的技术栈主要基于我个人的实践和总结，但是未必适合所有场景，毕竟每个企业的上下文各不相同。作为架构师你可以参考我推荐的技术栈，但不可拘泥照搬，你必须在深入理解分布系统原理的基础上，再结合企业实际场景灵活应用。
2.  在整个互联网基础技术平台体系中，还有消息，任务，数据访问层，发布系统，容器云平台，分布式事务，分布式一致性，测试，CI/CD等其它重要主题，请大家持续关注。
