---
layout: post
title: Java 引用类型简述
category: java
tags: [java]
---

## 强引用 ( Strong Reference )

强引用是使用最普遍的引用。如果一个对象具有强引用，那垃圾回收器绝不会回收它。当内存空间不足，Java虚拟机宁愿抛出OutOfMemoryError错误，使程序异常终止，也不会靠随意回收具有强引用的对象来解决内存不足的问题。 ps：强引用其实也就是我们平时A a = new A()这个意思。

*   强引用特性
    *   强引用可以直接访问目标对象。
    *   强引用所指向的对象在任何时候都不会被系统回收。
    *   强引用可能导致内存泄漏。

## Final Reference

*   当前类是否是finalizer类，注意这里finalizer是由JVM来标志的( 后面简称f类 )，并不是指java.lang.ref.Fianlizer类。但是f类是会被JVM注册到java.lang.ref.Fianlizer类中的。

① 当前类或父类中含有一个参数为空，返回值为void的名为finalize的方法。
② 并且该finalize方法必须非空

*   GC 回收问题
    *   对象因为Finalizer的引用而变成了一个临时的强引用，即使没有其他的强引用，还是无法立即被回收；
    *   对象至少经历两次GC才能被回收，因为只有在FinalizerThread执行完了f对象的finalize方法的情况下才有可能被下次GC回收，而有可能期间已经经历过多次GC了，但是一直还没执行对象的finalize方法；
    *   CPU资源比较稀缺的情况下FinalizerThread线程有可能因为优先级比较低而延迟执行对象的finalize方法；
    *   因为对象的finalize方法迟迟没有执行，有可能会导致大部分f对象进入到old分代，此时容易引发old分代的GC，甚至Full GC，GC暂停时间明显变长，甚至导致OOM；
    *   对象的finalize方法被调用后，这个对象其实还并没有被回收，虽然可能在不久的将来会被回收。

详见：[JVM源码分析之FinalReference完全解读 - 你假笨](http://lovestblog.cn/blog/2015/07/09/final-reference/)

## 软引用 ( Soft Reference )

是用来描述一些还有用但并非必须的对象。对于软引用关联着的对象，在系统将要发生内存溢出异常之前，将会把这些对象列进回收范围之中进行第二次回收。如果这次回收还没有足够的内存，才会抛出内存溢出异常。
对于软引用关联着的对象，如果内存充足，则垃圾回收器不会回收该对象，如果内存不够了，就会回收这些对象的内存。在 JDK 1.2 之后，提供了 SoftReference 类来实现软引用。软引用可用来实现内存敏感的高速缓存。软引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收器回收，Java虚拟机就会把这个软引用加入到与之关联的引用队列中。
**注意：Java 垃圾回收器准备对SoftReference所指向的对象进行回收时，调用对象的 finalize() 方法之前，SoftReference对象自身会被加入到这个 ReferenceQueue 对象中，此时可以通过 ReferenceQueue 的 poll() 方法取到它们。**

```
/**
 * 软引用：对于软引用关联着的对象，在系统将要发生内存溢出异常之前，将会把这些对象列进回收范围之中进行第二次回收( 因为是在第一次回收后才会发现内存依旧不充足，才有了这第二次回收 )。如果这次回收还没有足够的内存，才会抛出内存溢出异常。
 * 对于软引用关联着的对象，如果内存充足，则垃圾回收器不会回收该对象，如果内存不够了，就会回收这些对象的内存。
 * 通过debug发现，软引用在pending状态时，referent就已经是null了。
 *
 * 启动参数：-Xmx5m
 *
 */
public class SoftReferenceDemo {

    private static ReferenceQueue<MyObject> queue = new ReferenceQueue<>();

    public static void main(String[] args) throws InterruptedException {
        Thread.sleep(3000);
        MyObject object = new MyObject();
        SoftReference<MyObject> softRef = new SoftReference(object, queue);
        new Thread(new CheckRefQueue()).start();

        object = null;
        System.gc();
        System.out.println("After GC : Soft Get = " + softRef.get());
        System.out.println("分配大块内存");

        /**
         * ====================== 控制台打印 ======================
         * After GC : Soft Get = I am MyObject.
         * 分配大块内存
         * MyObject's finalize called
         * Object for softReference is null
         * After new byte[] : Soft Get = null
         * ====================== 控制台打印 ======================
         *
         * 总共触发了 3 次 full gc。第一次有System.gc();触发；第二次在在分配new byte[5*1024*740]时触发，然后发现内存不够，于是将softRef列入回收返回，接着进行了第三次full gc。
         */
//        byte[] b = new byte[5*1024*740];

        /**
         * ====================== 控制台打印 ======================
         * After GC : Soft Get = I am MyObject.
         * 分配大块内存
         * Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
         *      at com.bayern.multi_thread.part5.SoftReferenceDemo.main(SoftReferenceDemo.java:21)
         * MyObject's finalize called
         * Object for softReference is null
         * ====================== 控制台打印 ======================
         *
         * 也是触发了 3 次 full gc。第一次有System.gc();触发；第二次在在分配new byte[5*1024*740]时触发，然后发现内存不够，于是将softRef列入回收返回，接着进行了第三次full gc。当第三次 full gc 后发现内存依旧不够用于分配new byte[5*1024*740]，则就抛出了OutOfMemoryError异常。
         */
        byte[] b = new byte[5*1024*790];

        System.out.println("After new byte[] : Soft Get = " + softRef.get());
    }

    public static class CheckRefQueue implements Runnable {

        Reference<MyObject> obj = null;

        @Override
        public void run() {
            try {
                obj = (Reference<MyObject>) queue.remove();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            if (obj != null) {
                System.out.println("Object for softReference is " + obj.get());
            }

        }
    }

    public static class MyObject {

        @Override
        protected void finalize() throws Throwable {
            System.out.println("MyObject's finalize called");
            super.finalize();
        }

        @Override
        public String toString() {
            return "I am MyObject.";
        }
    }
}

```

## 弱引用 ( Weak Reference )

用来描述非必须的对象，但是它的强度比软引用更弱一些，被弱引用关联的对象只能生存到下一次垃圾收集发生之前。当垃圾收集器工作时，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。一旦一个弱引用对象被垃圾回收器回收，便会加入到一个注册引用队列中。
**注意：Java 垃圾回收器准备对WeakReference所指向的对象进行回收时，调用对象的 finalize() 方法之前，WeakReference对象自身会被加入到这个 ReferenceQueue 对象中，此时可以通过 ReferenceQueue 的 poll() 方法取到它们。**

```
/**
 * 用来描述非必须的对象，但是它的强度比软引用更弱一些，被弱引用关联的对象只能生存到下一次垃圾收集发送之前。当垃圾收集器工作时，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。一旦一个弱引用对象被垃圾回收器回收，便会加入到一个注册引用队列中。
 */
public class WeakReferenceDemo {

    private static ReferenceQueue<MyObject> queue = new ReferenceQueue<>();

    public static void main(String[] args) {

        MyObject object = new MyObject();
        Reference<MyObject> weakRef = new WeakReference<>(object, queue);
        System.out.println("创建的弱引用为 : " + weakRef);
        new Thread(new CheckRefQueue()).start();

        object = null;
        System.out.println("Before GC: Weak Get = " + weakRef.get());
        System.gc();
        System.out.println("After GC: Weak Get = " + weakRef.get());

        /**
         * ====================== 控制台打印 ======================
         * 创建的弱引用为 : java.lang.ref.WeakReference@1d44bcfa
         * Before GC: Weak Get = I am MyObject
         * After GC: Weak Get = null
         * MyObject's finalize called
         * 删除的弱引用为 : java.lang.ref.WeakReference@1d44bcfa , 获取到的弱引用的对象为 : null
         * ====================== 控制台打印 ======================
         */
    }

    public static class CheckRefQueue implements Runnable {

        Reference<MyObject> obj = null;

        @Override
        public void run() {
            try {
                obj = (Reference<MyObject>)queue.remove();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            if(obj != null) {
                System.out.println("删除的弱引用为 : " + obj + " , 获取到的弱引用的对象为 : " + obj.get());

            }

        }
    }

    public static class MyObject {

        @Override
        protected void finalize() throws Throwable {
            System.out.println("MyObject's finalize called");
            super.finalize();
        }

        @Override
        public String toString() {
            return "I am MyObject";
        }
    }
}

```

## 虚引用 ( Phantom Reference )

PhantomReference 是所有“弱引用”中最弱的引用类型。不同于软引用和弱引用，虚引用无法通过 get() 方法来取得目标对象的强引用从而使用目标对象，观察源码可以发现 get() 被重写为永远返回 null。
那虚引用到底有什么作用？**其实虚引用主要被用来 跟踪对象被垃圾回收的状态，通过查看引用队列中是否包含对象所对应的虚引用来判断它是否 即将被垃圾回收，从而采取行动。它并不被期待用来取得目标对象的引用，而目标对象被回收前，它的引用会被放入一个 ReferenceQueue 对象中，从而达到跟踪对象垃圾回收的作用。**
当phantomReference被放入队列时，说明referent的finalize()方法已经调用，并且垃圾收集器准备回收它的内存了。
**注意：PhantomReference 只有当 Java 垃圾回收器对其所指向的对象真正进行回收时，会将其加入到这个 ReferenceQueue 对象中，这样就可以追综对象的销毁情况。这里referent对象的finalize()方法已经调用过了。**
所以具体用法和之前两个有所不同，它必须传入一个 ReferenceQueue 对象。当虚引用所引用对象准备被垃圾回收时，虚引用会被添加到这个队列中。
Demo1：

```
/**
 * 虚引用也称为幽灵引用或者幻影引用，它是最弱的一种引用关系。一个持有虚引用的对象，和没有引用几乎是一样的，随时都有可能被垃圾回收器回收。
 * 虚引用必须和引用队列一起使用，它的作用在于跟踪垃圾回收过程。
 * 当phantomReference被放入队列时，说明referent的finalize()方法已经调用，并且垃圾收集器准备回收它的内存了。
 */
public class PhantomReferenceDemo {

    private static ReferenceQueue<MyObject> queue = new ReferenceQueue<>();

    public static void main(String[] args) throws InterruptedException {
        MyObject object = new MyObject();
        Reference<MyObject> phanRef = new PhantomReference<>(object, queue);
        System.out.println("创建的虚拟引用为 : " + phanRef);
        new Thread(new CheckRefQueue()).start();

        object = null;

        int i = 1;
        while (true) {
            System.out.println("第" + i++ + "次GC");
            System.gc();
            TimeUnit.SECONDS.sleep(1);
        }

        /**
         * ====================== 控制台打印 ======================
         * 创建的虚拟引用为 : java.lang.ref.PhantomReference@1d44bcfa
         * 第1次GC
         * MyObject's finalize called
         * 第2次GC
         * 删除的虚引用为: java.lang.ref.PhantomReference@1d44bcfa , 获取虚引用的对象 : null
         * ====================== 控制台打印 ======================
         *
         * 再经过一次GC之后，系统找到了垃圾对象，并调用finalize()方法回收内存，但没有立即加入PhantomReference Queue中。因为MyObject对象重写了finalize()方法，并且该方法是一个非空实现，所以这里MyObject也是一个Final Reference。所以地刺GC完成的是Final Reference的事情。
         * 第二次GC时，该对象真处理PhantomReference，此时，将PhantomReference加入虚引用队列( PhantomReference Queue )。
         * 而且每次gc之间需要停顿一些时间，已给JVM足够的处理时间；如果这里没有TimeUnit.SECONDS.sleep(1); 可能需要gc到第5、6次才会成功。
         */

    }

    public static class MyObject {

        @Override
        protected void finalize() throws Throwable {
            System.out.println("MyObject's finalize called");
            super.finalize();
        }

        @Override
        public String toString() {
            return "I am MyObject";
        }
    }

    public static  class CheckRefQueue implements Runnable {

        Reference<MyObject> obj = null;

        @Override
        public void run() {
            try {
                obj = (Reference<MyObject>)queue.remove();
                System.out.println("删除的虚引用为: " + obj + " , 获取虚引用的对象 : " + obj.get());
                System.exit(0);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

```

Q：👆了解下System.gc()操作，如果连续调用，若前一次没完成，后一次可能会失效，所以连接调用System.gc()其实作用不大？
A：关于上面例子的问题我们要补充两点
① 首先我们先来看下System.gc()的doc文档：

```
    /**
     * Runs the garbage collector.
     * <p>
     * Calling the <code>gc</code> method suggests that the Java Virtual
     * Machine expend effort toward recycling unused objects in order to
     * make the memory they currently occupy available for quick reuse.
     * When control returns from the method call, the Java Virtual
     * Machine has made a best effort to reclaim space from all discarded
     * objects.
     * <p>
     * The call <code>System.gc()</code> is effectively equivalent to the
     * call:
     * <blockquote><pre>
     * Runtime.getRuntime().gc()
     * </pre></blockquote>
     *
     * @see     java.lang.Runtime#gc()
     */
    public static void gc() {
        Runtime.getRuntime().gc();
    }

```

当这个方法返回的时候，Java虚拟机已经尽最大努力去回收所有丢弃对象的空间了。
因此不存在这System.gc()操作连续调用时，若前一次没完成，后一次可能会失效的情况。以及“所以连接调用System.gc()其实作用不大”这个说法不对，应该说连续调用System.gc()对性能可定是有影响的，但作用之一就是可以清除“漂浮垃圾”。
② 同时需要特别注意的是对于已经没有地方引用的这些f对象，并不会在最近的那一次gc里马上回收掉，而是会延迟到下一个或者下几个gc时才被回收，因为执行finalize方法的动作无法在gc过程中执行，万一finalize方法执行很长呢，所以只能在这个gc周期里将这个垃圾对象重新标活，直到执行完finalize方法将Final Reference从queue里删除，这样下次gc的时候就真的是漂浮垃圾了会被回收。

Demo2:

```
public class PhantomReferenceDemo2 {

    public static void main(String[] args) {
        ReferenceQueue<MyObject> queue = new ReferenceQueue<>();
        MyObject object = new MyObject();
        Reference<MyObject> phanRef = new PhantomReference<>(object, queue);
        System.out.println("创建的虚拟引用为 : " + phanRef);
        object = null;
        System.out.println(phanRef.get());

        System.gc();

        System.out.println("referent : " + phanRef);
        System.out.println(queue.poll() == phanRef); //true

        /**
         * ====================== 控制台打印 ======================
         * 创建的虚拟引用为 : java.lang.ref.PhantomReference@1d44bcfa
         * null
         * referent : java.lang.ref.PhantomReference@1d44bcfa
         * true
         * ====================== 控制台打印 ======================
         *
         * 这里因为MyObject没有重写finalize()方法，所以这里的在System.gc()后就会处理PhantomReference加入到PhantomReference Queue中。
         */
    }

    public static class MyObject {

        @Override
        public String toString() {
            return "I am MyObject";
        }
    }
}

```
