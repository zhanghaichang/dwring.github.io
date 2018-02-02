---
layout: post
title: 架构师成长之路(4)--知识体系（方法）
category: other
tags: [other]
---

要成为优秀合格的架构师，必须具备前瞻性的眼光和系统性的思考能力。而拥有这些能力的前提是你必须完善自己的知识体系。
互联网思维不是工具，它是世界观。这篇文章之后，你可以尝试构建自己的知识体系了。愿每个人都可以像一个U盘一样，自带系统随处插拔。愿每个人都可以和别人不一样。

## 1、通用技能：

我是谁：思维方式，不将就认真做事的人

如何做事：
1）整体把握，找到方法论（解决方案），
2）思路：分而治之，优先排列，计划进行（排期完成）。
3）及时沟通，反馈，勇于承担责任
4）团队意识

成长：
1）和优秀的人在一起
2）不断学习充电

完成定义：了解基础原理，自测通过，及时跟踪反馈问题，文档更新

熟练定义：绕开问题，而不是解决问题。

## 2、专业技能表

文档：每一项技能，熟读官方文档

基础知识：

1）网络知识，http原理，tcp基础知识

2）office能力：熟练使用excel，ppt

3）php基础知识：php，linux，mysql，nginx/apache，代理负载均衡

4）java基础知识：Java高级特性和类库，Java网络与服务器编程, TCP/IP协议，以及线程，I/O模型，框架。

5）数据结构算法，设计模式

6）研发能力：**瀑布模型：需求->需求分析->设计->开发->测试->上线->运维/运营**

调试和解决问题能力

敏捷思想：快速迭代，任务细分，wiki更新

web安全：

1）web安全

2）安全维度：漏洞，风险，事件

安全问题：xss，sql注入，ddos攻击

安全书：

《黑客攻防技术宝典（Web实战篇）》,《白帽子讲Web安全》,《Web前端黑客技术揭秘》,《Web之困》,《SQL注入攻击与防御》


#### 研发清单

* 编码环境
Python
Linux/UNIX
前端
爬虫进阶
调度
并发
数据结构
数据存储及处理
DevOps
调试
算法
持续集成
安全

协作
类似Trello的在线协同平台
Slack
微信
立会
设计思想
人人都是架构师：具备架构思想是一件多酷的事
实战出真知
如何设计
松耦合、紧内聚
单元与单元属性
生产者与消费者
结构
队列
LRU
分布式
存储
计算
资源考虑
CPU
内存
带宽
粗暴美学/暴力美学
大数据，先考虑run it，然后才能知道规律在哪
「run it优先」能快速打通整体，洞察问题
「run it优先」能摆脱细节（繁枝末节）的束缚
「run it优先」能快速迭代出伟大的v1
一个字总结
美
牛人1,2,3
1研究：研究东西，有足够洞察力，研究水准不错
2研发：Hack Idea自己有魄力实现，不懂研发的黑客如同不会游泳的海盗
3工程：研发出来的需要实战、需要工程化，否则只是玩具，而不能成为真的武器



### 3、网站架构知识

- 
架构演进

- 初始阶段：LAMP,部署在一台服务器
- 应用服务器和数据服务器分离
- 使用缓存改善性能
- 使用集群改善并发
- 数据库地读写分离
- 使用反向代理和cdn加速
- 使用分布式文件和分布式数据库
- 业务拆分
- 分布式服务

- 
架构模式

- 分层：横向分层：应用层，服务层，数据层
- 分割：纵向分割：拆分功能和服务
- 分布式
- 分布式应用和服务
- 
分布式静态资源

- 分布式数据和存储
- 分布式计算

- 集群：提高并发和可用性
- 缓存：优化系统性能
- cdn
- 方向代理访问资源
- 本地缓存
- 分布式缓存

- 异步：降低系统的耦合性 
- 提供系统的可用性
- 加快响应速度

- 冗余：冷备和热备，保证系统的可用性
- 自动化：发布，测试，部署，监控，报警，失效转移，故障恢复
- 安全：

- 架构核心要素

- 高性能：网站的灵魂
- 性能测试
- 前端优化
- 应用优化
- 数据库优化

- 可用性：保证服务器不宕机，一般通过冗余部署备份服务器来完成
- 负载均衡
- 数据备份
- 自动发布
- 灰度发布
- 监控报警

- 伸缩性：建集群，是否快速应对大规模增长的流量，容易添加新的机器
- 集群
- 负载均衡
- 缓存负载均衡

- 可扩展性：主要关注功能需求，应对业务的扩展，快速响应业务的变化。是否做法开闭原则，系统耦合依赖
- 分布式消息
- 服务化

- 安全性：网站的各种攻击，各种漏洞是否堵住，架构是否可以做到限流作用，防止ddos攻击。
- xss攻击
- sql注入
- csr攻击
- web防火墙漏洞
- 安全漏洞
- ssl

## 4. GitHub上整理的一些工具和资源

### 2.1技术站点

Hacker News：非常棒的针对编程的链接聚合网站

Programming reddit：同上

infoq：企业级应用，关注软件开发领域

OSChina：开源技术社区，开源方面做的不错哦

51cto，cnblogs：常见的技术社区，各有专长

stackoverflow：IT技术问答网站

GitHub：全球最大的源代码管理平台，很多知名开源项目都在上面，如Linux内核

OpenStack等免费的it电子书：http://it-ebooks.info/

DevStore：开发者服务商店


### 2.2  不错的书籍
 
《人件》，《人月神话》《代码大全2》,《计算机程序设计艺术》,《程序员的自我修养》,《程序员修炼之道》,
《高效能程序员的修炼（成为一名杰出的程序员其实跟写代码没有太大关系）》,《深入理解计算机系统》,《软件随想录》
《算法导论》（麻省理工学院出版社）， 《离线数学及其应用》，《设计模式》,《编程之美》,《黑客与画家》,《编程珠玑》,
《C++ Prime》,《Effective C++》,《TCP/IP详解》,《Unix 编程艺术》,《精神分析引论》弗洛伊德,《搞定：无压力工作的艺术》

### 2.3 平台工具（都是开源的好东东哦）

Redmine/Trac：项目管理平台
Jenkins/Jira(非开源)：持续集成系统（Apache Continuum，这个是Apache下的CI系统，还没来得及研究）
Sonar：代码质量管理平台
git，svn：源代码版本控制系统
GitLib/Gitorious：构建自己的GitHub服务器
gitbook：https://www.gitbook.io/写书的好东西，当然用来写文档也很不错的
Travis-ci：开源项目持续集成必备，和GitHub相结合，https://travis-ci.org

开源测试工具、社区（Selenium、OpenQA.org）

Puppet：一个自动管理引擎，可以适用于Linux、Unix以及Windows平台。所谓配置管理系统，就是管理机器里面诸如文件、用户、进程、软件包这些资源。无论是管理1台，还是上万台机器Puppet都能轻松搞定。

Nagios：系统状态监控报警，还有个Icinga(完全兼容nagios所有的插件,工作原理,配置文件以及方法,几乎一模一样。配置简单,功能强大)

Ganglia：分布式监控系统


fleet：分布式init系统

### 2.4 Web 服务器性能/压力测试工具/负载均衡器

http_load：程序非常小，解压后也不到100K

webbench：是Linux下的一个网站压力测试工具，最多可以模拟3万个并发连接去测试网站的负载能力

ab：ab是apache自带的一款功能强大的测试工具

Siege：一款开源的压力测试工具，可以根据配置对一个WEB站点进行多用户的并发访问，记录每个用户所有请求过程的相应时间，并在一定数量的并发访问下重复进行。

squid（前端缓存），nginx（负载），nodejs（没错它也可以，自己写点代码就能实现高性能的负载均衡器）：常用的负载均衡器

Piwik：开源网站访问量统计系统

ClickHeat：开源的网站点击情况热力图

HAProxy：高性能TCP /HTTP负载均衡器

ElasticSearch：搜索引擎基于Lucene

Page Speed SDK和YSLOW

HAR Viewer：HAR分析工具

protractor：E2E（end to end）自动化测试工具

### 2.5 Web 前端相关

GRUNT：js task runner

Sea.js：js模块化

knockout.js：MVVM开发前台，绑定技术

Angular.js：使用超动感HTML & JS开发WEB应用！

Highcharts.js，Flot：常用的Web图表插件

Raw：非常不错的一款高级数据可视化工具

Rickshaw：图标库，可用于构建实时图表

JavaScript InfoVis Toolkit：另一款Web数据可视化插件

Pdf.js，在html中展现pdf

ACE，CodeMirror：Html代码编辑器（ACE甚好啊）

NProcess：绚丽的加载进度条

impress.js：让你制作出令人眩目的内容展示效果(类似的还有reveal)

Threejs：3DWeb库

Hightopo：基于Html5的2D、3D可视化UI库

jQuery.dataTables.js：高度灵活的表格插件

Raphaël：js，canvas绘图库，后来发现百度指数的图形就是用它绘出来的