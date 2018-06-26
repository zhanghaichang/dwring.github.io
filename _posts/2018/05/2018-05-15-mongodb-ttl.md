---
layout: post
title: MongoDB自动删除过期数据--TTL索引
category: database
tags: [database]
---


由于公司业务需求，对于3个月前的过期数据需要进行删除动作，以释放空间和方便维护
本来想的是使用crontab写个脚本定时执行，但是看到Mongo本身就有自动删除过期数据的功能，所以还是用一下吧这个方法就是使用TTL索引，后续我再写一个脚本定时删除的任务

**介绍：**

TTL索引是MongoDB中一种特殊的索引， 可以支持文档在一定时间之后自动过期删除，目前TTL索引只能在单字段上建立，
并且字段类型必须是date类型或者包含有date类型的数组（如果数组中包含多个date类型字段，则取最早时间为过期时间）
官网介绍链接：[https://docs.mongodb.com/v3.2/core/index-ttl/](https://docs.mongodb.com/v3.2/core/index-ttl/)

**机制：**

当你在集合中某一个字段建立TTL索引后，后台会有一个单线程，通过不断查询（默认60s一次）索引的值来判断document是否有过期，
并且删除文档的动作还依据mongod实例的负载情况，如果负载很高，可能会稍微延后一段时间再删除。
还有一个需要注意的地方，在复制集成员中，TTL后台线程只删除primary的过期数据，如果此实例变为secondary角色，则后台线程闲置

**创建TTL索引方法：**

和普通索引的创建方法一样，只是会多加一个属性而已

例：在log_events的集合中，createTime 字段上建立一小时后过期的TTL索引

```json
 >db.log_events.createIndex( { "createTime": 1 },     ---字段名称 
                                { expireAfterSeconds: 60*60 } )     ---过期时间（单位秒）
>db.log_events.getIndexes()     ---查看索引
\[
        {
                "v" : 1,
                "key" : {
                        "_id" : 1
                },
                "name" : "\_id\_",
                "ns" : "tt.t1"
        },
        {
                "v" : 1,
                "key" : {
                        "createTime" : 1
                },
                "name" : "createTime_1",
                "ns" : "tt.t1",
                "expireAfterSeconds" : 3600
        }
\]
```
**修改TTL索引的expireAfterSeconds属性值：**

注：如果想更改过期时间expireAfterSeconds，可以使用collMod方法，要不然你只能只用dropIndex(),createIndex()方法重建索引了，我想这样的方法在亿级数据量下是很头疼的

db.runCommand( { collMod: "log_events",     ---集合名
                index: { keyPattern: { createTime: 1 },     ---createTime为具有TTL索引的字段名
                          expireAfterSeconds: 7200          ---修改后的过期时间(秒)
                        }})

虽然上面的方法可以实现自动过期删除，但是如果白天业务很忙，频繁的删除数据势必会增加负载，所以我想着晚上定时删除过期数据（如果晚上业务量少的话）
方法如下：

增加一个expireTime字段（用于指定过期时间），expireAfterSeconds属性值设置为0，

注：上面的createTime字段就不需要再有TTL索引了，这个expireTime的时间就需要在插入时指定上

>db.log_events.createIndex( { "expireTime": 1 },     ---字段名称
                                { expireAfterSeconds: 0 } )     ---过期时间（单位秒）
>db.log_events.insert( {
  "expireTime": new Date('Jan 22, 2016 23:00:00'),     ---此文档将在2016-1-22的23点自动删除
  "logEvent": 2,
  "logMessage": "Success!"} )

  
  

这样我们就实现了，指定时间自动删除的动作了

  

**限制条件：**

有一下集中情况是无法使用TTL索引的

①TTL索引是单字段索引，混合索引不支持TTL，并且也会忽略expireAfterSeconds属性

②在_id 主键上不能建立TTL索引

③在capped collection中不能建立TTL索引，因为MongoDB不能从capped collection中删除文档

④你不能使用createIndex()去更改已经存在的TTL索引的expireAfterSeconds值，如果想更改expireAfterSeconds，可以使用collMod命令，
否则你只能删除索引，然后重建了

⑤你不能在已有索引的字段上再创建TTL索引了，如果你想把非TTL索引改为TTL索引，那就只能删除重建索引了

  

**验证：**

虽然已经实现了晚上集中自动删除的功能，但是还是担心删除过大数量时负荷问题，随进行了简单测试，一查看TTL索引在亿级别集合中删除140万过期数据的消耗

测试配置：

OS：Vm虚拟机

CPU: 4

内存：8

集合数据量：

\> db.t1.count()

104273617

因为我制造测试数据时，\_id是顺序增加的，所以我直接查看\_id=1500000的那笔数据的createTime，然后自己计算一下此createTime和当前时间的时间差，

随后根据这个时间差来更改expireAfterSeconds的值，以让这150万数据5分钟后过期并删除。

在修改完expireAfterSeconds后，就严密延时“ vmstat 1 ” 命令的输出数据；

我的测试结果：

删除操作整个过程在90秒左右完成；

CPU最高占用90%，平均在50%

内存占用3G

这个也是特别准确的模拟情况，只是粗略的了解一下TTL索引的资源消耗，以决定是不是需要这样的方式来实现删除过期数据

监控vmstat的截图：

![](https://img-blog.csdn.net/20170120142825150)
