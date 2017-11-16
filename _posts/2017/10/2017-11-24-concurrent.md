---
layout: post
title:  Java 高并发综合
category: java
tags: [java]
---

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

Reentrantlock提供了公平和非公平两种锁。公平锁是对锁的获取是先进先出，而非公平锁是可以插队的。当然从性能上分析，非公平锁的性能要好得多。因此，在无特殊需要，应该优选非公平锁，但是synchronized提供锁业不是绝对公平的。Reentrantlock在构造的时候可以指定锁是否公平。

在使用重入锁时，一定要在程序最后释放锁。一般释放锁的代码要写在finally里。否则，如果程序出现异常，Loack就永远无法释放了。synchronized的锁是JVM最后自动释放的。

经典使用方式如下：

```java
try {
 if (lock.tryLock(5, TimeUnit.SECONDS)) { //如果已经被lock，尝试等待5s，看是否可以获得锁，如果5s后仍然无法获得锁则返回false继续执行
 // lock.lockInterruptibly();可以响应中断事件
 try { 
 //操作
 } finally {
 lock.unlock();
 }
 }
} catch (InterruptedException e) {
 e.printStackTrace(); //当前线程被中断时(interrupt)，会抛InterruptedException 
}
```

Reentrantlock提供了非常丰富的锁控制功能，灵活应用这些控制方法，可以提高应用程序的性能。不过这里并非是极力推荐使用Reentrantlock。重入锁算是JDK中提供的高级开发工具。

## ReadWriteLock读写锁

读写分离是一种非常常见的数据处理思想。在sql中应该算是必须用到的技术。ReadWriteLock是在JDK5中提供的读写分离锁。读写分离锁可以有效地帮助减少锁竞争，以提升系统性能。读写分离使用场景主要是如果在系统中，读操作次数远远大于写操作。使用方式如下：

```java
private ReentrantReadWriteLock readWriteLock = new ReentrantReadWriteLock();
private Lock readLock = readWriteLock.readLock();
private Lock writeLock = readWriteLock.writeLock();
public Object handleRead() throws InterruptedException {
    try {
        readLock.lock();
        Thread.sleep(1000);
        return value;
    }finally{
        readLock.unlock();
    }
}
public Object handleRead() throws InterruptedException {
    try {
        writeLock.lock();
        Thread.sleep(1000);
        return value;
    }finally{
        writeLock.unlock();
    }
}
```

## Condition对象

Conditiond对象用于协调多线程间的复杂协作。主要与锁相关联。通过Lock接口中的newCondition()方法可以生成一个与Lock绑定的Condition实例。Condition对象和锁的关系就如用Object.wait()、Object.notify()两个函数以及synchronized关键字一样。

这里可以把ArrayBlockingQueue的源码摘出来看一下：

```java
public class ArrayBlockingQueue extends AbstractQueue
implements BlockingQueue, java.io.Serializable {
/** Main lock guarding all access */
final ReentrantLock lock;
/** Condition for waiting takes */
private final Condition notEmpty;
/** Condition for waiting puts */
private final Condition notFull;

public ArrayBlockingQueue(int capacity, boolean fair) {
    if (capacity <= 0)
        throw new IllegalArgumentException();
    this.items = new Object[capacity];
    lock = new ReentrantLock(fair); 
    notEmpty = lock.newCondition(); // 生成与Lock绑定的Condition
    notFull =  lock.newCondition();
}

public void put(E e) throws InterruptedException {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == items.length)
            notFull.await();
        insert(e);
    } finally {
        lock.unlock();
    }
}

private void insert(E x) {
    items[putIndex] = x;
    putIndex = inc(putIndex);
    ++count;
    notEmpty.signal(); // 通知
}

public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == 0) // 如果队列为空
            notEmpty.await();  // 则消费者队列要等待一个非空的信号
        return extract();
    } finally {
        lock.unlock();
    }
}

private E extract() {
    final Object[] items = this.items;
    E x = this.<E>cast(items[takeIndex]);
    items[takeIndex] = null;
    takeIndex = inc(takeIndex);
    --count;
    notFull.signal(); // 通知put() 线程队列已有空闲空间
    return x;
}

// other code
}
```

## Semaphore信号量

信号量为多线程协作提供了更为强大的控制方法。信号量是对锁的扩展。无论是内部锁synchronized还是重入锁ReentrantLock，一次都允许一个线程访问一个资源，而信号量却可以指定多个线程同时访问某一个资源。从构造函数可以看出：

```java
public Semaphore(int permits) {}
public Semaphore(int permits, boolean fair){} // 可以指定是否公平
```

permits指定了信号量的准入书，也就是同时能申请多少个许可。当每个线程每次只申请一个许可时，这就相当于指定了同时有多少个线程可以访问某一个资源。这里罗列一下主要方法的使用：

*   public void acquire() throws InterruptedException {} //尝试获得一个准入的许可。若无法获得，则线程会等待，知道有线程释放一个许可或者当前线程被中断。
*   public void acquireUninterruptibly(){} // 类似于acquire()，但是不会响应中断。
*   public boolean tryAcquire(){} // 尝试获取，如果成功则为true，否则false。这个方法不会等待，立即返回。
*   public boolean tryAcquire(long timeout, TimeUnit unit) throws InterruptedException {} // 尝试等待多长时间
*   public void release() //用于在现场访问资源结束后，释放一个许可，以使其他等待许可的线程可以进行资源访问。

下面来看一下JDK文档中提供使用信号量的实例。这个实例很好的解释了如何通过信号量控制资源访问。

```java
public class Pool {
private static final int MAX_AVAILABLE = 100;
private final Semaphore available = new Semaphore(MAX_AVAILABLE, true);
public Object getItem() throws InterruptedException {
    available.acquire();
    // 申请一个许可
    // 同时只能有100个线程进入取得可用项，
    // 超过100个则需要等待
    return getNextAvailableItem();
}

public void putItem(Object x) {
    // 将给定项放回池内，标记为未被使用
    if (markAsUnused(x)) {
        available.release();
        // 新增了一个可用项，释放一个许可，请求资源的线程被激活一个
    }
}

// 仅作示例参考，非真实数据
protected Object[] items = new Object[MAX_AVAILABLE]; // 用于对象池复用对象
protected boolean[] used = new boolean[MAX_AVAILABLE]; // 标记作用

protected synchronized Object getNextAvailableItem() {
    for (int i = 0; i < MAX_AVAILABLE; ++i) {
        if (!used[i]) {
            used[i] = true;
            return items[i];
        }
    }
    return null;
}

protected synchronized boolean markAsUnused(Object item) {
    for (int i = 0; i < MAX_AVAILABLE; ++i) {
        if (item == items[i]) {
            if (used[i]) {
                used[i] = false;
                return true;
            } else {
                return false;
            }
        }
    }
    return false;
}
}
```

此实例简单实现了一个对象池，对象池最大容量为100。因此，当同时有100个对象请求时，对象池就会出现资源短缺，未能获得资源的线程就需要等待。当某个线程使用对象完毕后，就需要将对象返回给对象池。此时，由于可用资源增加，因此，可以激活一个等待该资源的线程。

## <span class="wp_keywordlink">[ThreadLocal](http://www.codeceo.com/article/threadlocal-usage.html "ThreadLocal")</span>线程局部变量

在刚开始接触ThreadLocal，笔者很难理解这个线程局部变量的使用场景。当现在回过头去看，ThreadLocal是一种多线程间并发访问变量的解决方案。与synchronized等加锁的方式不同，ThreadLocal完全不提供锁，而使用了以空间换时间的手段，为每个线程提供变量的独立副本，以保障线程安全，因此它不是一种数据共享的解决方案。

ThreadLocal是解决线程安全问题一个很好的思路，ThreadLocal类中有一个Map，用于存储每一个线程的变量副本，Map中元素的键为线程对象，而值对应线程的变量副本，由于Key值不可重复，每一个“线程对象”对应线程的“变量副本”，而到达了线程安全。

特别值得注意的地方，从性能上说，ThreadLocal并不具有绝对的又是，在并发量不是很高时，也行加锁的性能会更好。但作为一套与锁完全无关的线程安全解决方案，在高并发量或者所竞争激烈的场合，使用ThreadLocal可以在一定程度上减少锁竞争。

下面是一个ThreadLocal的简单使用：

```java
public class TestNum {
 // 通过匿名内部类覆盖ThreadLocal的initialValue()方法，指定初始值
 private static ThreadLocal seqNum = new ThreadLocal() {
 public Integer initialValue() {
 return 0;
 }
 };
 // 获取下一个序列值
 public int getNextNum() {
 seqNum.set(seqNum.get() + 1);
 return seqNum.get();
}public static void main(String[] args) {
 TestNum sn = new TestNum();
 //3个线程共享sn，各自产生序列号
 TestClient t1 = new TestClient(sn);
 TestClient t2 = new TestClient(sn);
 TestClient t3 = new TestClient(sn);
 t1.start();
 t2.start();
 t3.start();
 }
private static class TestClient extends Thread {
 private TestNum sn;
public TestClient(TestNum sn) {
 this.sn = sn;
 }
public void run() {
 for (int i = 0; i < 3; i++) {
 // 每个线程打出3个序列值
 System.out.println("thread[" + Thread.currentThread().getName() + "] --> sn["
 + sn.getNextNum() + "]");
 }
 }
 }
 }
```

输出结果：

```java
thread[Thread-0] –> sn[1]
thread[Thread-1] –> sn[1]
thread[Thread-2] –> sn[1]
thread[Thread-1] –> sn[2]
thread[Thread-0] –> sn[2]
thread[Thread-1] –> sn[3]
thread[Thread-2] –> sn[2]
thread[Thread-0] –> sn[3]
thread[Thread-2] –> sn[3]
```

输出的结果信息可以发现每个线程所产生的序号虽然都共享同一个TestNum实例，但它们并没有发生相互干扰的情况，而是各自产生独立的序列号，这是因为ThreadLocal为每一个线程提供了单独的副本。

## 锁的性能和优化

“锁”是最常用的同步方法之一。在平常开发中，经常能看到很多同学直接把锁加很大一段代码上。还有的同学只会用一种锁方式解决所有共享问题。显然这样的编码是让人无法接受的。特别的在高并发的环境下，激烈的锁竞争会导致程序的性能下降德更加明显。因此合理使用锁对程序的性能直接相关。

**1、线程的开销**

在多核情况下，使用多线程可以明显提高系统的性能。但是在实际情况中，使用多线程的方式会额外增加系统的开销。相对于单核系统任务本身的资源消耗外，多线程应用还需要维护额外多线程特有的信息。比如，线程本身的元数据，线程调度，线程上下文的切换等。

**2、减小锁持有时间**

在使用锁进行并发控制的程序中，当锁发生竞争时，单个线程对锁的持有时间与系统性能有着直接的关系。如果线程持有锁的时间很长，那么相对地，锁的竞争程度也就越激烈。因此，在程序开发过程中，应该尽可能地减少对某个锁的占有时间，以减少线程间互斥的可能。比如下面这一段代码：

```java
public synchronized void syncMehod(){
beforeMethod();
mutexMethod();
afterMethod();
}
```

此实例如果只有mutexMethod()方法是有同步需要的，而在beforeMethod(),和afterMethod()并不需要做同步控制。如果beforeMethod(),和afterMethod()分别是重量级的方法，则会花费较长的CPU时间。在这个时候，如果并发量较大时，使用这种同步方案会导致等待线程大量增加。因为当前执行的线程只有在执行完所有任务后，才会释放锁。

下面是优化后的方案，只在必要的时候进行同步，这样就能明显减少线程持有锁的时间，提高系统的吞吐量。代码如下：

```java
public void syncMehod(){
beforeMethod();
synchronized(this){
mutexMethod();
}
afterMethod();
}
```

**3、减少锁粒度**

减小锁粒度也是一种削弱多线程锁竞争的一种有效手段，这种技术典型的使用场景就是ConcurrentHashMap这个类。在普通的HashMap中每当对集合进行add()操作或者get()操作时，总是获得集合对象的锁。这种操作完全是一种同步行为，因为锁是在整个集合对象上的，因此，在高并发时，激烈的锁竞争会影响到系统的吞吐量。

如果看过源码的同学应该知道HashMap是数组+链表的方式做实现的。ConcurrentHashMap在HashMap的基础上将整个HashMap分成若干个段(Segment)，每个段都是一个子HashMap。如果需要在增加一个新的表项，并不是将这个HashMap加锁，二十搜线根据hashcode得到该表项应该被存放在哪个段中，然后对该段加锁，并完成put()操作。这样，在多线程环境中，如果多个线程同时进行写入操作，只要被写入的项不存在同一个段中，那么线程间便可以做到真正的并行。具体的实现希望读者自己花点时间读一读ConcurrentHashMap这个类的源码，这里就不再做过多描述了。

**4、锁分离**

在前面提起过ReadWriteLock读写锁，那么读写分离的延伸就是锁的分离。同样可以在JDK中找到锁分离的源码LinkedBlockingQueue。

```java
public class LinkedBlockingQueue extends AbstractQueue
implements BlockingQueue, java.io.Serializable {
/* Lock held by take, poll, etc /
private final ReentrantLock takeLock = new ReentrantLock();
/** Wait queue for waiting takes */
private final Condition notEmpty = takeLock.newCondition();

/** Lock held by put, offer, etc */
private final ReentrantLock putLock = new ReentrantLock();

/** Wait queue for waiting puts */
private final Condition notFull = putLock.newCondition();

public E take() throws InterruptedException {
    E x;
    int c = -1;
    final AtomicInteger count = this.count;
    final ReentrantLock takeLock = this.takeLock;
    takeLock.lockInterruptibly(); // 不能有两个线程同时读取数据
    try {
        while (count.get() == 0) { // 如果当前没有可用数据，一直等待put()的通知
            notEmpty.await();
        }
        x = dequeue(); // 从头部移除一项
        c = count.getAndDecrement(); // size减1
        if (c > 1)
            notEmpty.signal(); // 通知其他take()操作
    } finally {
        takeLock.unlock(); // 释放锁
    }
    if (c == capacity)
        signalNotFull(); // 通知put()操作，已有空余空间
    return x;
}

public void put(E e) throws InterruptedException {
    if (e == null) throw new NullPointerException();
    // Note: convention in all put/take/etc is to preset local var
    // holding count negative to indicate failure unless set.
    int c = -1;
    Node<E> node = new Node(e);
    final ReentrantLock putLock = this.putLock;
    final AtomicInteger count = this.count;
    putLock.lockInterruptibly(); // 不能有两个线程同时put数据
    try {
        /*
         * Note that count is used in wait guard even though it is
         * not protected by lock. This works because count can
         * only decrease at this point (all other puts are shut
         * out by lock), and we (or some other waiting put) are
         * signalled if it ever changes from capacity. Similarly
         * for all other uses of count in other wait guards.
         */
        while (count.get() == capacity) { // 队列满了 则等待
            notFull.await();
        }
        enqueue(node); // 加入队列
        c = count.getAndIncrement();// size加1
        if (c + 1 < capacity)
            notFull.signal(); // 如果有足够空间，通知其他线程
    } finally {
        putLock.unlock();// 释放锁
    }
    if (c == 0)
        signalNotEmpty();// 插入成功后，通知take()操作读取数据
}

// other code     
}
```

这里需要说明一下的就是，take()和put()函数是相互独立的，它们之间不存在锁竞争关系。只需要在take()和put()各自方法内部分别对takeLock和putLock发生竞争。从而，削弱了锁竞争的可能性。

**5、锁粗化**

上面说到的减小锁时间和粒度，这样做就是为了满足每个线程持有锁的时间尽量短。但是，在粒度上应该把握一个度，如果对用一个锁不停地进行请求、同步和释放，其本身也会消耗系统宝贵的资源，反而加大了系统开销。

我们需要知道的是，虚拟机在遇到一连串连续的对同一锁不断进行请求和释放的操作时，便会把所有的锁操作整合成对锁的一次请求，从而减少对锁的请求同步次数，这样的操作叫做锁的粗化。下面是一段整合实例演示：

```java
public void syncMehod(){
synchronized(lock){
method1();
}
synchronized(lock){
method2();
}
}
```

JVM整合后的形式：

```java
public void syncMehod(){
synchronized(lock){
method1();
method2();
}
}
```

因此，这样的整合给我们开发人员对锁粒度的把握给出了很好的演示作用。

## 无锁的并行计算

上面花了很大篇幅在说锁的事情，同时也提到过锁是会带来一定的上下文切换的额外资源开销，在高并发时，”锁“的激烈竞争可能会成为系统瓶颈。因此，这里可以使用一种非阻塞同步方法。这种无锁方式依然能保证数据和程序在高并发环境下保持多线程间的一致性。

**1、非阻塞同步/无锁**
非阻塞同步方式其实在前面的ThreadLocal中已经有所体现，每个线程拥有各自独立的变量副本，因此在并行计算时，无需相互等待。这里笔者主要推荐一种更为重要的、基于比较并交换（Compare And Swap）CAS算法的无锁并发控制方法。

CAS算法的过程：它包含3个参数CAS(V,E,N)。V表示要更新的变量，E表示预期值，N表示新值。仅当V值等于E值时，才会将V的值设为N，如果V值和E值不同，则说明已经有其他线程做了更新，则当前线程什么都不做。最后CAS返回当前V的真实值。CAS操作时抱着乐观的态度进行的，它总是认为自己可以成功完成操作。当多个线程同时使用CAS操作一个变量时，只有一个会胜出，并成功更新，其余俊辉失败。失败的线程不会被挂起，仅是被告知失败，并且允许再次尝试，当然也允许失败的线程放弃操作。基于这样的原理，CAS操作及时没有锁，也可以发现其他线程对当前线程的干扰，并且进行恰当的处理。

**2、原子量操作**

JDK的java.util.concurrent.atomic包提供了使用无锁算法实现的原子操作类，代码内部主要使用了底层native代码的实现。有兴趣的同学可以继续跟踪一下native层面的代码。这里就不贴表层的代码实现了。

下面主要以一个例子来展示普通同步方法和无锁同步的性能差距：

```java
public class TestAtomic {
private static final int MAX_THREADS = 3;
private static final int TASK_COUNT = 3;
private static final int TARGET_COUNT = 100 * 10000;
private AtomicInteger acount = new AtomicInteger(0);
private int count = 0;
synchronized int inc() {
    return ++count;
}

synchronized int getCount() {
    return count;
}

public class SyncThread implements Runnable {
    String name;
    long startTime;
    TestAtomic out;

    public SyncThread(TestAtomic o, long startTime) {
        this.out = o;
        this.startTime = startTime;
    }

    @Override
    public void run() {
        int v = out.inc();
        while (v < TARGET_COUNT) {
            v = out.inc();
        }
        long endTime = System.currentTimeMillis();
        System.out.println("SyncThread spend:" + (endTime - startTime) + "ms" + ", v=" + v);
    }
}

public class AtomicThread implements Runnable {
    String name;
    long startTime;

    public AtomicThread(long startTime) {
        this.startTime = startTime;
    }

    @Override
    public void run() {
        int v = acount.incrementAndGet();
        while (v < TARGET_COUNT) {
            v = acount.incrementAndGet();
        }
        long endTime = System.currentTimeMillis();
        System.out.println("AtomicThread spend:" + (endTime - startTime) + "ms" + ", v=" + v);
    }
}

@Test
public void testSync() throws InterruptedException {
    ExecutorService exe = Executors.newFixedThreadPool(MAX_THREADS);
    long startTime = System.currentTimeMillis();
    SyncThread sync = new SyncThread(this, startTime);
    for (int i = 0; i < TASK_COUNT; i++) {
        exe.submit(sync);
    }
    Thread.sleep(10000);
}

@Test
public void testAtomic() throws InterruptedException {
    ExecutorService exe = Executors.newFixedThreadPool(MAX_THREADS);
    long startTime = System.currentTimeMillis();
    AtomicThread atomic = new AtomicThread(startTime);
    for (int i = 0; i < TASK_COUNT; i++) {
        exe.submit(atomic);
    }
    Thread.sleep(10000);
}
}
```

测试结果如下：

```java
testSync():
SyncThread spend:201ms, v=1000002
SyncThread spend:201ms, v=1000000
SyncThread spend:201ms, v=1000001
testAtomic():
AtomicThread spend:43ms, v=1000000
AtomicThread spend:44ms, v=1000001
AtomicThread spend:46ms, v=1000002
```

相信这样的测试结果将内部锁和非阻塞同步算法的性能差异体现的非常明显。因此笔者更推荐直接视同atomic下的这个原子类。

## 结束语

终于把想表达的这些东西整理完成了，其实还有一些想CountDownLatch这样的类没有讲到。不过上面的所讲到的绝对是并发编程中的核心。也许有些读者朋友能在网上看到很多这样的知识点，但是个人还是觉得知识只有在对比的基础上才能找到它合适的使用场景。因此，这也是笔者整理这篇文章的原因，也希望这篇文章能帮到更多的同学。
