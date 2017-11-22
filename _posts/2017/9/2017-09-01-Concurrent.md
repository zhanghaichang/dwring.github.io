---
layout: post
title: 并发编程-Concurrent用户指南
category: java
tags: [java]
---
本指南根据 Jakob Jenkov 最新博客翻译，请随时关注博客更新：[http://tutorials.jenkov.com/java-util-concurrent/index.html](http://tutorials.jenkov.com/java-util-concurrent/index.html)。
本指南已做成中英文对照阅读版的 pdf 文档，有兴趣的朋友可以去 [Java并发工具包java.util.concurrent用户指南中英文对照阅读版.pdf[带书签]](http://download.csdn.net/detail/defonds/8469189) 进行下载。

# 1. java.util.concurrent - Java 并发工具包

Java 5 添加了一个新的包到 Java 平台，java.util.concurrent 包。这个包包含有一系列能够让 Java 的并发编程变得更加简单轻松的类。在这个包被添加以前，你需要自己去动手实现自己的相关工具类。
本文我将带你一一认识 java.util.concurrent 包里的这些类，然后你可以尝试着如何在项目中使用它们。本文中我将使用 Java 6 版本，我不确定这和 Java 5 版本里的是否有一些差异。
我不会去解释关于 Java 并发的核心问题 - 其背后的原理，也就是说，如果你对那些东西感兴趣，参考《[Java 并发指南](http://tutorials.jenkov.com/java-concurrency/index.html)》。

## 半成品

本文很大程度上还是个 "半成品"，所以当你发现一些被漏掉的类或接口时，请耐心等待。在我空闲的时候会把它们加进来的。

# 2. 阻塞队列 BlockingQueue

java.util.concurrent 包里的 BlockingQueue 接口表示一个线程安放入和提取实例的队列。本小节我将给你演示如何使用这个 BlockingQueue。
本节不会讨论如何在 Java 中实现一个你自己的 BlockingQueue。如果你对那个感兴趣，参考《[Java 并发指南](http://tutorials.jenkov.com/java-concurrency/index.html)》

## BlockingQueue 用法

BlockingQueue 通常用于一个线程生产对象，而另外一个线程消费这些对象的场景。下图是对这个原理的阐述：

![blocking-queue](http://img.blog.csdn.net/20150302184203260?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZGVmb25kcw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
**一个线程往里边放，另外一个线程从里边取的一个 BlockingQueue。**
一个线程将会持续生产新对象并将其插入到队列之中，直到队列达到它所能容纳的临界点。也就是说，它是有限的。如果该阻塞队列到达了其临界点，负责生产的线程将会在往里边插入新对象时发生阻塞。它会一直处于阻塞之中，直到负责消费的线程从队列中拿走一个对象。
负责消费的线程将会一直从该阻塞队列中拿出对象。如果消费线程尝试去从一个空的队列中提取对象的话，这个消费线程将会处于阻塞之中，直到一个生产线程把一个对象丢进队列。

## BlockingQueue 的方法

BlockingQueue 具有 4 组不同的方法用于插入、移除以及对队列中的元素进行检查。如果请求的操作不能得到立即执行的话，每个方法的表现也不同。这些方法如下：

原文链接：[http://tutorials.jenkov.com/java-util-concurrent/index.html](http://tutorials.jenkov.com/java-util-concurrent/index.html)。
