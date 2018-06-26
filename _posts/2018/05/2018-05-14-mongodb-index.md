---
layout: post
title: MongoDB索引的种类与使用
category: database
tags: [database]
---

一：索引的种类
-------

**1：_id索引**：是绝大多数集合默认建立的索引，对于每个插入的数据，MongoDB都会自动生成一条唯一的_id字段  
**2：单键索引**：

    1.单键索引是最普通的索引
    2.与_id索引不同，单键索引不会自动创建 如：一条记录，形式为：{x:1,y:2,z:3}
    db.imooc_2.getIndexes()//查看索引 
    db.imooc_2.ensureIndex({x:1})//创建索引，索引可以重复创建，若创建已经存在的索引，则会直接返回成功。
    db.imooc_2.find()//查看数据

**3：多键索引**  
多键索引与单键索引创建形式相同，区别在于字段的值。  
1）单键索引：值为一个单一的值，如字符串，数字或日期。  
2）多键索引：值具有多个记录，如数组。

    db.imooc_2.insert({x:[1,2,3,4,5]})//插入一条数组数据

**4：复合索引**：查询多个条件时,建立复合索引  
例如{x:1,y:2,z:3}这样一条数据，要按照x与y的值进行查询，就需要创建复合索引。

    db.imooc_2.ensureIndex({x:1,y:1}) #1升序，-1降序
    db.imooc_2.find({x:1,y:2}) #使用复合索引查询

**5：过期索引**  
在一段时间后会过期的索引  
在索引过期后，相应的数据会被删除  
适合存储在一段时间之后会失效的数据，比如用户的登录信息、存储的日志等。  
db.imooc_2.ensureIndex({time:1},{expireAfterSeconds:10}) #创建过期索引，time-字段，expireAfterSeconds在多少秒后过期，单位：秒

    db.imooc_2.ensureIndex({time:1},{expireAfterSeconds:30}) #time索引30秒后失效
    db.imooc_2.insert({time:new Date()}) #new Date()自动获取当前时间，ISODate
    db.imooc_2.find() #可看到刚才insert的值

过30秒后再find，刚才的数据就已经不存在了。

过期索引的限制：  
1.存储在过期索引字段的值必须是指定的时间类型，必须是ISODate或者ISODate数组，不能使用时间戳，否则不能自动删除。  
例如 >db.imooc_2.insert({time:1})，这种是不能被自动删除的  
2.如果指定了ISODate数组，则按照最小的时间进行删除。  
3.过期索引不能是复合索引。因为不能指定两个过期时间。  
4.删除时间是不精确的。删除过程是由MongoDB的后台进程每60s跑一次的，而且删除也需要一定时间，所以存在误差

**6：全文索引**：对字符串与字符串数组创建全文课搜索的索引  
不适用全文索引：查找困难，效率低下，需要正则匹配，逐条扫描。  
使用全文索引：简单查询即可查询需要的结果

创建方式

    db.articles.ensureIndex({key:"text"}) #key-字段名，value-固定字符串text
    上述指令表示，在articles这个集合的key字段上创建了一个全文索引
    
    db.articles.ensureIndex({key1:"text",key2:"text"}) #在多个字段上创建全文索引
    
    对于nosql数据库，每个记录存储的key可能都是不同的，如果要在所有的key上建立全文索引，一个一个写很麻烦，mongodb可以通过下面指令完成：
    db.articles.ensureIndex({"$**":"text"}) #给所有字段建立全文索引

全文索引的创建：  
1：可以为一个字段创建全文索引  
2：可以为多个字段创建全文索引  
3：可以为集合中所有的字段创建全文索引  
注意：上面三种创建全文索引的方式，前两个方式类似，第三个需要一个特殊的字符串来表示——”$**”，我想如果集合中就两三个字段，也可以使用2来创建这样的全文索引，如果这个集合总就一个字段使用1也是可以的，3仅仅是为了统一化而已。

全文索引的查找:  
1：使用全文索引查询不需要指定全文索引的字段名字——直接使用`$text,$search`即可  
2：在MongoDB中每个数据集合只允许创建一个全文索引，不过这个全文索引可以针对一个、多个、全部的数据集合的字段来创建。  
3：查询多个关键词，可以使用空格将多个关键词分开——空格——或的关系  
4：指定不包含的字段使用-来表示—— -:非的关系  
5：引号包括起来代表与的关系—— \\”\\”:与的关系

    db.articles.find({$text:{$search:"coffee"}})
    db.articles.find({$text:{$search:"aa bb cc"}}) #空格代表或操作，aa或bb或cc
    db.articles.find({$text:{$search:"aa bb -cc"}}) #-号为非操作，即不包含cc的
    db.articles.find({$text:{$search: "\"aa\" \"bb\" \"cc\""}}) #加双引号可以提供与关系操作

相似度查询：

搜索排序，查询结果与你查询条件越相关的越排在前面。  
MongoDB中可以使用$meta操作符完成，格式：

    {score:{$meta: "textScore"}}

在全文搜索的格式中加入这样一个条件，如下：

    db.imooc_2.find({$text:{$search:"aa bb"}},{score:{$meta:"textScore"}})

搜索出的结果会多出一个score字段，这个得分越高，相关度越高。  
还可以对查询出的结果根据得分进行排序：

    db.imooc_2.find({$text:{$search:"aa bb"}},{score:{$meta:"textScore"}}).sort({score:{$meta:"textScore"}})

加上.sort方法即可。

全局索引的限制：  
1.每次查询，只能指定一个$text查询  
2.`$text`查询不能出现在`$nor`查询中  
3\. 查询中如果包含了$text, hint不再起作用（强制指定索引hint）  
4\. MongoDB全文索引还不支持中文

**7：地理位置索引**  
将一些点的位置存储在MongoDB中，创建索引后，可以按照位置来查找其他点。

地理位置索引分为两类：  
1.2D索引，用于存储和查找平面上的点。  
2.2Dsphere索引，用于存储和查找球面上的点。  
例如：  
查找距离某个点一定距离内的点。  
查找包含在某区域内的点。

分为2种：2D平面地理位置索引 和 2D sphere 2D球面地里位置索引 2者的区别在于计算距离时使用的计算方式不同（平面距离还是球面距离）

2D地理位置索引创建方式

> db.collection.ensureIndex({w:”2d”})

2D地理位置索引的取值范围以及表示方法 经纬度\[经度,纬度\]  
经纬度取值范围 经度\[-180,180\] 纬度\[-90,90\]

    db.collection.insert({w:[180,90]})

2D地理位置查询有2种  
1.使用$near 查询距离某个点最近的点 ,默认返回最近的100个点

    db.collection.find({w:{$near:[x,y]}})

可以使用$maxDistance:x 限制返回的最远距离

    db.collection.find({w:{$near:[x,y],$maxDistance:z}})

2.使用$geoWithin 查询某个形状内的点  
形状的表示方式:

    1. $box 矩形，使用{$box:[[x1,y1],[x2,y2]]}
    2. $center 圆形，使用 {$center:[[x,y],r]}
    3. $polygon 多边形，使用 {$polygon:[[x1,y1],[x2,y2],[x3,y3]]}

mongodb geoWithin 查询

    查询矩形中的点
    db.collection.find({w:{$geoWithin:{$box:[[0,0],[3,3]]}}})
    查询圆中的点
    db.collection.find({w:{$geoWithin:{$center:[[0,0],5]}}})
    查询多边形中的点
    db.collection.find({w:{$geoWithin:{$polygon:[[0,0],[0,1],[2,5],[6,1]]}}})

mongodb geoNear 查询

    geoNear 使用 runCommand命令操作
    db.runCommand({
    geoNear:"collection名称",
    near:[x, y],
    minDistance:10（对2d索引无效，对2Dsphere有效'）
    maxDistance:10
    num:1 返回数量
    })

可返回最大距离和平均距离等数据.

返回的数据：  
results:查询到的数据；dis:距离，obj:数据记录  
stats:查询参数，maxDistance最大距离和avgDistance平均距离  
ok:1,查询成功

mongodb 2Dsphere索引详解

2Dsphere index create method  
use command:

    db.collection.ensureindex({key: '2dsphere'})

2Dsphere位置表示方式：  
GeoJSON：描述一个点，一条直线，多边形等形状。  
格式：

    {type:'', coordinates:[list]}

GeoJSON查询可支持多边形交叉点等，支持MaxDistance 和 MinDistance

索引属性
----

创建索引的格式：

    db.collection.ensureIndex({indexValue},{indexProperty})

其中：indexProperty比较重要的有  
1：名字

    db.collection.ensureIndex({indexValue},{name:})

MongoDB会自动的创建，规则是key\_1 或者 key\_-1 1或者-1代表排序方向，一般影响不大，长度一般有限制125字节  
为了见名知意我们可以自己来命名

自定义索引名称方式：

    db.imooc_2.ensureIndex({x:1,y:1,z:1,m:1},{name:"normal_index"})

删除索引

    db.imooc_dropIndex(indexName)

删除索引时，可以通过我们定义的名字来删除索引

    db.imooc_2.dropIndex("normal_index")

2：唯一性：不允许在同一个集合上插入具有同一唯一性的数据。

    db.imooc_2.ensureIndex({x:1,y:1,z:1,m:1},{unigue:true)

3：稀疏性

    db.collection.ensureIndex({},{sparse:true/false}) #指定索引是否稀疏

MongoDB索引默认是不稀疏的。  
稀疏性的不同代表了MongoDB在处理索引中存在但是文档中不存在的字段的两种不同的方法。  
例如，我们为一个collection的x字段指定了索引，但这个collection中可以插入如{y:1,z:1}这种不存在x字段的数据，  
如果索引为不稀疏的，mongodb依然会为这个数据创建索引，如果在创建索引时指定为稀疏索引，那么就可以避免这件事情发生了。

    db.imooc_2.insert({"m":1})
    db.imooc_2.insert({"n":1})

通过$exists可以判断字段是否存在，如

    db.imooc_2.find({m:{$exists:true}}) #筛选出有m字段的文档

给这个文档的m字段创建一个稀疏索引：

    db.imooc_2.ensureIndex({m:1},{sparse:true})

第二条文档不存在m字段，所以不会创建这个索引  
如果使用稀疏索引查找不存在稀疏索引字段的文档，mongodb则不会使用这个索引查找  
例如：

    db.imooc_2.find({m:{$exists:false}}) #可以查到数据

但如果我们通过hint强制使用索引，就不会查到数据了

    db.imooc_2.find({m:{$exists:false}}).hint("m_1") #查不出数据，因为n上并没有m字段的索引

4：是否定时删除（过期索引TTL）
