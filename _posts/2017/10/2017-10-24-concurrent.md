---
layout: post
title:  Java 高并发综合
category: java
tags: [java]
---
http://www.codeceo.com/article/talk-about-concurrency.html
在一般性开发中，笔者经常看到很多同学在对待java并发开发模型中只会使用一些基础的方法。比如<span class="wp_keywordlink">[Volatile](http://www.codeceo.com/article/java-volatile-var.html "Volatile")</span>，synchronized。像Lock和atomic这类高级并发包很多人并不经常使用。我想大部分原因都是来之于对原理的不属性导致的。在繁忙的开发工作中，又有谁会很准确的把握和使用正确的并发模型呢？

所以最近基于这个思想，本人打算把并发控制机制这部分整理成一篇文章。既是对自己掌握知识的一个回忆，也是希望这篇讲到的类容能帮助到大部分开发者。

并行程序开发不可避免地要涉及多线程、多任务的协作和数据共享等问题。在JDK中，提供了多种途径实现多线程间的并发控制。比如常用的：内部锁、重入锁、读写锁和信号量。

## Java内存模型

在java中，每一个线程有一块工作内存区，其中存放着被所有线程共享的主内存中的变量的值的拷贝。当线程执行时，它在自己的工作内存中操作这些变量。

为了存取一个共享的变量，一个线程通常先获取锁定并且清除它的工作内存区，这保证该共享变量从所有线程的共享内存区正确地装入到线程的工作内存区，当线程解锁时保证该工作内存区中变量的值协会到共享内存中。

当一个线程使用某一个变量时，不论程序是否正确地使用线程同步操作，它获取的值一定是由它本身或者其他线程存储到变量中的值。例如，如果两个线程把不同的值或者对象引用存储到同一个共享变量中，那么该变量的值要么是这个线程的，要么是那个线程的，共享变量的值不会是由两个线程的引用值组合而成。

一个变量时Java程序可以存取的一个地址，它不仅包括基本类型变量、引用类型变量，而且还包括数组类型变量。保存在主内存区的变量可以被所有线程共享，但是一个线程存取另一个线程的参数或者局部变量时不可能的，所以开发人员不必担心局部变量的线程安全问题。 

## volatile变量–多线程间可见

由于每个线程都有自己的工作内存区，因此当一个线程改变自己的工作内存中的数据时，对其他线程来说，可能是不可见的。为此，可以使用volatile关键字破事所有线程军读写内存中的变量，从而使得volatile变量在多线程间可见。

声明为volatile的变量可以做到如下保证：

> 1、其他线程对变量的修改，可以及时反应在当前线程中；
> 2、确保当前线程对volatile变量的修改，能及时写回到共享内存中，并被其他线程所见；
> 3、使用volatile声明的变量，编译器会保证其有序性。

## 同步关键字synchronized

同步关键字synchronized是Java语言中最为常用的同步方法之一。在JDK早期版本中，synchronized的性能并不是太好，值适合于锁竞争不是特别激烈的场合。在JDK6中，synchronized和非公平锁的差距已经缩小。更为重要的是，synchronized更为简洁明了，代码可读性和维护性比较好。

锁定一个对象的方法：

```java
public synchronized void method(){}
```

当method()方法被调用时，调用线程首先必须获得当前对象所，若当前对象锁被其他线程持有，这调用线程会等待，犯法结束后，对象锁会被释放，以上方法等价于下面的写法：

```java
public void method(){
synchronized(this){
// do something …
}
}
```

其次，使用synchronized还可以构造同步块，与同步方法相比，同步块可以更为精确控制同步代码范围。一个小的同步代码非常有离与锁的快进快出，从而使系统拥有更高的吞吐量。

```java
public void method(Object o){
// before
synchronized(o){
// do something ...
}
// after
}
```

synchronized也可以用于static函数：

```java
public synchronized static void method(){}
```

这个地方一定要注意，synchronized的锁是加在**当前Class对象**上，因此，所有对该方法的调用，都必须获得Class对象的锁。

虽然synchronized可以保证对象或者代码段的线程安全，但是仅使用synchronized还是不足以控制拥有复杂逻辑的线程交互。为了实现多线程间的交互，还需要使用Object对象的wait()和notify()方法。

典型用法：

```java
synchronized(obj){
    while(<?>){
        obj.wait();
        // 收到通知后，继续执行。
    }
}
```

在使用wait()方法前，需要获得对象锁。在wait()方法执行时，当前线程或释放obj的独占锁，供其他线程使用。

当等待在obj上线程收到obj.notify()时，它就能重新获得obj的独占锁，并继续运行。注意了，notify()方法是**随机唤起**等待在当前对象的某一个线程。

下面是一个阻塞队列的实现：

```java
public class BlockQueue{
 private List list = new ArrayList();

 public synchronized Object pop() throws InterruptedException{
 while (list.size()==0){
 this.wait();
 }
 if (list.size()>0){
 return list.remove(0);
 } else{
 return null;
 }
 }

 public synchronized Object put(Object obj){
 list.add(obj);
 this.notify();
 }

}
```

**synchronized配合wait()、notify()**应该是Java开发者必须掌握的基本技能。

## Reentrantlock重入锁

Reentrantlock称为重入锁。它比synchronized拥有更加强大的功能，它可以中断、可定时。在高并发的情况下，它比synchronized有明显的性能优势。

Reentrantlock提供了公平和非公平两种锁。公平锁是对锁的获取是先进先出，
