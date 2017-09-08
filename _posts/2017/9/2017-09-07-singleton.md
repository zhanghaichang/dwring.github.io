---
layout: post
title: 线程安全的单例
category: java
tags: [java]
---

不使用`synchronized`和`lock`，如何实现一个线程安全的单例？

回答最多的是静态内部类和枚举。很好，这两种确实可以实现。

### 枚举

```java
public enum Singleton {  
    INSTANCE;  
    public void whateverMethod() {  
    }  
}  
```

### 静态内部类

```java
public class Singleton {  
    private static class SingletonHolder {  
    private static final Singleton INSTANCE = new Singleton();  
    }  
    private Singleton (){}  
    public static final Singleton getInstance() {  
    return SingletonHolder.INSTANCE;  
    }  
}  
```java

还有人回答的很简单：饿汉。很好，这个也是对的。

### 饿汉

```java
public class Singleton {  
    private static Singleton instance = new Singleton();  
    private Singleton (){}  
    public static Singleton getInstance() {  
    return instance;  
    }  
}  
```

### 饿汉变种

```java
public class Singleton {  
    private static class SingletonHolder {  
    private static final Singleton INSTANCE = new Singleton();  
    }  
    private Singleton (){}  
    public static final Singleton getInstance() {  
    return SingletonHolder.INSTANCE;  
    }  
}  
```

（更多单例实现方式见：单例模式的七种写法）

> 问：这几种实现单例的方式的真正的原理是什么呢？
> 
> 答：以上几种实现方式，都是借助了`ClassLoader`的线程安全机制。

先解释清楚为什么说都是借助了`ClassLoader`。

从后往前说，先说两个**饿汉**，其实都是通过定义静态的成员变量，以保证`instance`可以在类初始化的时候被实例化。那为啥让`instance`在类初始化的时候被实例化就能保证线程安全了呢？因为类的初始化是由`ClassLoader`完成的，这其实就是利用了`ClassLoader`的线程安全机制啊。

再说**静态内部类**，这种方式和两种饿汉方式只有细微差别，只是做法上稍微优雅一点。这种方式是`Singleton`类被装载了，`instance`不一定被初始化。因为`SingletonHolder`类没有被主动使用，只有显示通过调用`getInstance`方法时，才会显示装载`SingletonHolder`类，从而实例化`instance`。。。但是，原理和饿汉一样。

最后说**枚举**，其实，如果把枚举类进行反序列化，你会发现他也是使用了`static`<span class="Apple-converted-space"> </span>`final`来修饰每一个枚举项。（详情见：深度分析Java的枚举类型—-枚举的线程安全性及序列化问题）

至此，我们说清楚了，各位看官的回答都是利用了`ClassLoader`的线程安全机制。至于为什么`ClassLoader`加载类是线程安全的，这里可以先直接回答：`ClassLoader`的`loadClass`方法在加载类的时候使用了`synchronized`关键字。也正是因为这样， 除非被重写，这个方法默认在整个装载过程中都是同步的（线程安全的）。（详情见：深度分析Java的ClassLoader机制（源码级别））

哈哈哈哈！！！~所以呢，这里可以说，大家的回答都只答对了一半。虽然没有显示使用`synchronized`和`lock`，但是还是间接的用到了！！！！

那么，这里再问一句：不使用synchronized和lock，如何实现一个线程安全的单例？

相关推荐

[单例模式的七种写法](http://mp.weixin.qq.com/s?__biz=MzI3NzE0NjcwMg==&mid=402230646&idx=1&sn=16b7dda4dd46de380ebeb850bbcbd63b&scene=21#wechat_redirect)

[设计模式（二）——单例模式](http://mp.weixin.qq.com/s?__biz=MzI3NzE0NjcwMg==&mid=402577543&idx=1&sn=41c4bf5f46d13806668edacce130468b&scene=21#wechat_redirect)

