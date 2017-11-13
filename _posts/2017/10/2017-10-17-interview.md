---
layout: post
title:  BO、VO、PO、DTO、POJO......的概念
category: other
tags: [other]
---

在Java使用中经常会碰到BO、VO、DAO、POJO等各种O，那么具体这些O代表的是什么？

## 1.PO
(persistant object) 持久对象
PO是Persistant Object的缩写，即持久化对象，通常对应数据模型，可以简单的理解为一个PO实例对应数据库中的一条记录，操作该实例即可以操作数据库中对应的数据。PO只封装数据库中对应的记录，不应该包含对数据库的操作。

## 2.VO
(view object) 值对象
VO(也可以理解为View Object视图对象)，通常用于封装页面上要展示的数据，在视图层传递，可以由PO转换而来，但是不能包含数据操作。

## 3.DO
（Domain Object）领域对象
DO就是从现实世界中抽象出来的有形或无形的业务实体。一般和数据中的表结构对应。

## 4.TO
(Transfer Object) 数据传输对象
TO就是在应用程序不同 tie( 关系 ) 之间传输的对象

## 5.BO
(business object) 业务对象
BO通常对应业务模型，封装业务数据，在业务服务层使用。BO中可以包含多个PO，封装业务数据。

## 6.DTO
(Data Transfer Object) 数据传输对象
DTO数据传输对象通常用于封装服务与服务之间、分层与分层之间要传输的数据，不应该包含业务逻辑，对要传输的数据起到承载的作用。

## 7.POJO
(plain ordinary java object) 简单无规则 java 对象
POJO是一个只有属性及属性setter和getter方法的基本JavaBean，是一个中间对象，可以用于多种用途，用于数据传输它就是DTO，用于数据展示它就是VO。

## 8.DAO
(data access object) 数据访问对象
是一个 sun 的一个标准 j2ee 设计模式， 这个模式中有个接口就是 DAO ，它负持久层的操作。为业务层提供接口。此对象用于访问数据库。通常和 PO 结合使用， DAO 中包含了各种数据库的操作方法。通过它的方法 , 结合 PO 对数据库进行相关的操作。夹在业务逻辑与数据库资源中间。配合 VO, 提供数据库的 CRUD 操作。
