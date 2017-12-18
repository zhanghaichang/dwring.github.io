---
layout: post
title: 从一个简单的Java单例示例谈谈并发
category: java
tags: [java]
---

单例模式可能是大家经常接触和使用的一个[设计模式](http://www.codeceo.com/article/category/develop/design-patterns "设计模式")，你可能会这么写

```java
public class UnsafeLazyInitiallization {
    private static UnsafeLazyInitiallization instance;
    private UnsafeLazyInitiallization() {
    }
    public static UnsafeLazyInitiallization getInstance(){
        if(instance==null){   //1:A线程执行
            instance=new UnsafeLazyInitiallization();  //2:B线程执行
        }
        return instance;
    }
}
```

上面代码大家应该都知道，所谓的线程不安全的懒汉单例写法。在UnsafeLazyInitiallization类中，假设A线程执行代码1的同时，B线程执行代码2，此时，线程A可能看到instance引用的对象还没有初始化。

你可能会说，线程不安全，我可以对getInstance()方法做同步处理保证安全啊，比如下面这样的写法

```java
 public class SafeLazyInitiallization {
        private static SafeLazyInitiallization instance;
        private SafeLazyInitiallization() {
        }
        public synchronized static SafeLazyInitiallization getInstance(){
            if(instance==null){
                instance=new SafeLazyInitiallization();
            }
            return instance;
        }
    }
```

这样的写法是保证了线程安全，但是由于getInstance()方法做了同步处理，synchronized将导致性能开销。如getInstance()方法被多个线程频繁调用，将会导致程序执行性能的下降。反之，如果getInstance()方法不会被多个线程频繁的调用，那么这个方案将能够提供令人满意的性能。

那么，有没有更优雅的方案呢？前人的智慧是伟大的，在早期的JVM中，synchronized存在巨大的性能开销，因此，人们想出了一个“聪明”的技巧——双重检查锁定。人们通过双重检查锁定来降低同步的开销。下面来让我们看看

```java
public class DoubleCheckedLocking { //1
    private static DoubleCheckedLocking instance; //2
    private DoubleCheckedLocking() {
    }
    public static DoubleCheckedLocking getInstance() { //3
        if (instance == null) { //4:第一次检查
            synchronized (DoubleCheckedLocking.class) { //5:加锁
                if (instance == null) //6:第二次检查
                    instance = new DoubleCheckedLocking(); //7:问题的根源出在这里
            } //8
        } //9
        return instance; //10
    } //11
}
```

如上面代码所示，如果第一次检查instance不为null，那么就不需要执行下面的加锁和初始化操作。因此，可以大幅降低synchronized带来的性能开销。双重检查锁定看起来似乎很完美，但这是一个错误的优化！为什么呢？在线程执行到第4行，代码读取到instance不为null时，instance引用的对象有可能还没有完成初始化。在第7行创建了一个对象，这行代码可以分解为如下的3行伪代码

```java
memory=allocate(); //1:分配对象的内存空间
ctorInstance(memory); //2:初始化对象
instance=memory; //3:设置instance指向刚分配的内存地址
```

上面3行代码中的2和3之间，可能会被重排序（在一些JIT编译器上，这种重排序是真实发生的，如果不了解重排序，后文JMM会详细解释）。2和3之间重排序之后的执行时序如下

```java
memory=allocate(); //1:分配对象的内存空间
instance=memory; //3:设置instance指向刚分配的内存地址，注意此时对象还没有被初始化
ctorInstance(memory); //2:初始化对象
```

回到示例代码第7行，如果发生重排序，另一个并发执行的线程B就有可能在第4行判断instance不为null。线程B接下来将访问instance所引用的对象，但此时这个对象可能还没有被A线程初始化。在知晓问题发生的根源之后，我们可以想出两个办法解决

*   不允许2和3重排序
*   允许2和3重排序，但不允许其他线程“看到”这个重排序

下面就介绍这两个解决方案的具体实现

**基于[Volatile](http://www.codeceo.com/article/java-volatile-var.html "Volatile")的解决方案**

对于前面的基于双重检查锁定的方案，只需要做一点小的修改，就可以实现线程安全的延迟初始化。请看下面的示例代码

```java
public class SafeDoubleCheckedLocking {
    private volatile static SafeDoubleCheckedLocking instance;

    private SafeDoubleCheckedLocking() {
    }

    public static SafeDoubleCheckedLocking getInstance() {

        if (instance == null) {

            synchronized (SafeDoubleCheckedLocking.class) {

                if (instance == null)

                    instance = new SafeDoubleCheckedLocking();//instance为volatile，现在没问题了

            }

        }

        return instance;

    }

}
```

当声明对象的引用为volatile后，前面伪代码谈到的2和3之间的重排序，在多线程环境中将会被禁止。

**基于类初始化的解决方案**

JVM在类的初始化阶段（即在Class被加载后，且被线程使用之前），会执行类的初始化。在执行类的初始化期间，JVM会去获取多个线程对同一个类的初始化。基于这个特性，实现的示例代码如下

```java
public class InstanceFactory {

    private InstanceFactory() {
    }

    private static class InstanceHolder {

        public static InstanceFactory instance = new InstanceFactory();

    }

    public static InstanceFactory getInstance() {

        return InstanceHolder.instance; //这里将导致InstanceHolder类被初始化

    }

}
```

这个方案的本质是允许前面伪代码谈到的2和3重排序，但不允许其他线程“看到”这个重排序。在InstanceFactory示例代码中，首次执行getInstance()方法的线程将导致InstanceHolder类被初始化。由于Java语言是多线程的，多个线程可能在同一时间尝试去初始化同一个类或接口（比如这里多个线程可能会在同一时刻调用getInstance()方法来初始化IInstanceHolder类）。Java语言规定，对于每一个类和接口C，都有一个唯一的初始化锁LC与之对应。从C到LC的映射，由JVM的具体实现去自由实现。JVM在类初始化期间会获取这个初始化锁，并且每个线程至少获取一次锁来确保这个类已经被初始化过了。

## **JMM**

也许你还存在疑问，前面谈的重排序是什么鬼？为什么volatile在某方面就能禁止重排序？现在引出本文的另一个话题JMM（Java Memory Model——Java内存模型）。什么是JMM呢？JMM是一个抽象概念，它并不存在。Java虚拟机规范中试图定义一种Java内存模型（JMM）来屏蔽掉各种硬件和操作系统的内存访问差异，以实现让Java程序在各种平台下都能达到一致的内存访问效果。在此之前，主流程序语言（如C/C++等）直接使用物理硬件和操作系统的内存模型，因此，会由于不同平台的内存模型的差异，有可能导致程序在一套平台上并发完全正常，而在另一套平台上并发访问却经常出错，因此在某些场景就必须针对不同的平台来编写程序。

Java线程之间的通信由JMM来控制，JMM决定一个线程共享变量的写入何时对另一个线程可见。JMM保证如果程序是正确同步的，那么程序的执行将具有顺序一致性。从抽象的角度看，JMM定义了线程和主内存之间的抽象关系：线程之间的共享变量（实例域、静态域和数据元素）存储在主内存（Main Memory）中，每个线程都有一个私有的本地内存（Local Memory），本地内存中存储了该线程以读/写共享变量的副本（局部变量、方法定义参数和异常处理参数是不会在线程之间共享，它们存储在线程的本地内存中）。从物理角度上看，主内存仅仅是虚拟机内存的一部分，与物理硬件的主内存名字一样，两者可以互相类比；而本地内存，可与处理器高速缓存类比。Java内存模型的抽象示意图如图所示

![enter image description here](http://static.codeceo.com/images/2016/08/81e47eb67a57c25eac63e12a2755bda2.jpg)

这里先介绍几个基础概念：8种操作指令、内存屏障、顺序一致性模型、as-if-serial、happens-before 、数据依赖性、 重排序。

**8种操作指令**

关于主内存与本地内存之间具体的交互协议，即一个变量如何从主内存拷贝到本地内存、如何从本地内存同步回主内存之类的实现细节，JMM中定义了以下8种操作来完成，虚拟机实现时必须保证下面提及的每种操作都是原子的、不可再分的（对于double和long类型的遍历来说，load、store、read和write操作在某些平台上允许有例外）：

*   lock(锁定)：作用于主内存的变量，它把一个变量标识为一条线程独立的状态。
*   unlock(解锁)：作用于主内存的变量，它把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。
*   read(读取)：作用于主内存的变量，它把一个变量的值从主内存传输到线程的本地内存中，以便随后的load动作使用。
*   load(载入)：作用于本地内存的变量，它把read操作从主内存中得到变量值放入本地内存的变量副本中。
*   use(使用)：作用于本地内存的变量，它把本地内存中一个变量的值传递给执行引擎，每当虚拟机遇到一个需要使用到变量的值的字节码指令时将会执行这个操作。
*   assign(赋值)：作用于本地内存的变量，它把一个从执行引擎接收到的值赋给本地内存的变量，每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作。
*   store(存储)：作用于本地内存的变量，它把本地内存中的一个变量的值传送到主内存中，以便随后的write操作使用。
*   write(写入)：作用于主内存的变量，它把store操作从本地内存中提到的变量的值放入到主内存的变量中。

如果要把一个变量从主内存模型复制到本地内存，那就要顺序的执行read和load操作，如果要把变量从本地内存同步回主内存，就要顺序的执行store和write操作。注意，Java内存模型只要求上述两个操作必须按顺序执行，而没有保证是连续执行。也就是说read与load之间、store与write之间是可插入其他指令的，如对主内存中的变量a、b进行访问时，一种可能出现的顺序是read a read b、load b、load a。

**内存屏障**

内存屏障是一组处理器指令（前面的8个操作指令），用于实现对内存操作的顺序限制。包括LoadLoad, LoadStore, StoreLoad, StoreStore共4种内存屏障。内存屏障存在的意义是什么呢？它是在Java编译器生成指令序列的适当位置插入内存屏障指令来禁止特定类型的处理器重排序，从而让程序按我们预想的流程去执行，内存屏障是与相应的内存重排序相对应的。JMM把内存屏障指令分为4类

![enter image description here](http://static.codeceo.com/images/2016/08/e5e6f56eea2d653571f36804f81b40bc.jpg)

StoreLoad Barriers是一个“全能型 ”的屏障，它同时具有其他3个屏障的效果。现在的多数处理器大多支持该屏障（其他类型的屏障不一定被所有处理器支持）。执行该屏障开销会很昂贵，因为当前处理器通常要把写缓冲区中的数据全部刷新到内存中。

**数据依赖性**

如果两个操作访问同一个变量，且这两个操作中有一个为写操作，此时这两个操作之间就存在数据依赖性。数据依赖性分3种类型：写后读、写后写、读后写。这3种情况，只要重排序两个操作的执行顺序，程序的执行结果就会被改变。编译器和处理器可能对操作进行重排序。而它们进行重排序时，会遵守数据依赖性，不会改变数据依赖关系的两个操作的执行顺序。

这里所说的数据依赖性仅针对单个处理器中执行的指令序列和单个线程中执行的操作，不同处理器之间和不同线程之间的数据依赖性不被编译器和处理器考虑。

**顺序一致性内存模型**

顺序一致性内存模型是一个理论参考模型，在设计的时候，处理器的内存模型和编程语言的内存模型都会以顺序一致性内存模型作为参照。它有两个特性：

*   一个线程中的所有操作必须按照程序的顺序来执行
*   （不管程序是否同步）所有线程都只能看到一个单一的操作执行顺序。在顺序一致性的内存模型中，每个操作必须原子执行并且立刻对所有线程可见。

从顺序一致性模型中，我们可以知道程序所有操作完全按照程序的顺序串行执行。而在JMM中，临界区内的代码可以重排序（但JMM不允许临界区内的代码“逸出”到临界区外，那样就破坏监视器的语义）。JMM会在退出临界区和进入临界区这两个关键时间点做一些特别处理，使得线程在这两个时间点具有与顺序一致性模型相同的内存视图。虽然线程A在临界区内做了重排序，但由于监视器互斥执行的特性，这里的线程B根本无法“观察”到线程A在临界区内的重排序。这种重排序既提高了执行效率，又没有改变程序的执行结果。像前面单例示例的类初始化解决方案就是采用了这个思想。

**as-if-serial**

as-if-serial的意思是不管怎么重排序，（单线程）程序的执行结果不能改变。编译器、runtime和处理器都必须遵守as-if-serial语义。为了遵守as-if-serial语义，编译器和处理器不会对存在数据依赖关系的操作做重排序。

as-if-serial语义把单线程程序保护了起来，遵守as-if-serial语义的编译器、runtime和处理器共同为编写单线程程序的[程序员](http://www.codeceo.com/ "程序员")创建了一个幻觉：单线程程序是按程序的顺序来执行的。as-if-serial语义使单线程程序员无需担心重排序会干扰他们，也无需担心内存可见性问题。

**happens-before**

happens-before是JMM最核心的概念。从JDK5开始，Java使用新的JSR-133内存模型，JSR-133 使用happens-before的概念阐述操作之间的内存可见性，如果一个操作执行的结果需要对另一个操作可见，那么这两个操作之间必须存在happens-before关系。

happens-before规则如下：

*   程序次序法则：线程中的每个动作 A 都 happens-before 于该线程中的每一个动作 B，其中，在程序中，所有的动作 B 都出现在动作 A 之后。（注：此法则只是要求遵循 as-if-serial语义）
*   监视器锁法则：对一个监视器锁的解锁 happens-before 于每一个后续对同一监视器锁的加锁。（显式锁的加锁和解锁有着与内置锁，即监视器锁相同的存储语意。）
*   volatile变量法则：对 volatile 域的写入操作 happens-before 于每一个后续对同一域的读操作。（原子变量的读写操作有着与 volatile 变量相同的语意。）（volatile变量具有可见性和读写原子性。）
*   线程启动法则：在一个线程里，对 Thread.start 的调用会 happens-before 于每一个启动线程中的动作。 线程终止法则：线程中的任何动作都 happens-before 于其他线程检测到这个线程已终结，或者从 Thread.join 方法调用中成功返回，或者 Thread.isAlive 方法返回false。
*   中断法则法则：一个线程调用另一个线程的 interrupt 方法 happens-before 于被中断线程发现中断（通过抛出InterruptedException， 或者调用 isInterrupted 方法和 interrupted 方法）。
*   终结法则：一个对象的构造函数的结束 happens-before 于这个对象 finalizer 开始。
*   传递性：如果 A happens-before 于 B，且 B happens-before 于 C，则 A happens-before 于 C。

happens-before与JMM的关系如下图所示

![enter image description here](http://static.codeceo.com/images/2016/08/c11a018eb307623fab26e5afbee89625.png)

as-if-serial语义和happens-before本质上一样，参考顺序一致性内存模型的理论，在不改变程序执行结果的前提下，给编译器和处理器以最大的自由度，提高并行度。

**重排序**

终于谈到我们反复提及的重排序了，重排序是指编译器和处理器为了优化程序性能而对指令序列进行重新排序的一种手段。重排序分3种类型。

*   编译器优化的重排序。编译器在不改变单线程程序语义（as-if-serial ）的前提下，可以重新安排语句的执行顺序。
*   指令级并行的重排序。现代处理器采用了指令级并行技术（Instruction Level Parallelism，ILP）来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对机器指令的执行顺序。
*   内存系统的重排序。由于处理器使用缓存和读/写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。

从Java源代码到最终实际执行的指令序列，会分别经历下面3种重排序

![enter image description here](http://static.codeceo.com/images/2016/08/de6d051a2d66e3e9e026220f5d4e6060.png)

上述的1属于编译器重排序，2和3属于处理器重排序。这些重排序可能会导致多线程程序出现内存可见性问题。对于编译器，JMM的编译器重排序规则会禁止特定类型的编译器重排序（不是所有的编译器重排序都要禁止）。对于处理器重排序，JMM的处理器重排序规则会要求Java编译器在生成指令序列时，插入特定类型的内存屏障指令，通过内存屏障指令来禁止特定类型的处理器重排序。

JMM属于语言级的内存模型，它确保在不同的编译器和不同的处理器平台之上，通过禁止特定类型的编译器重排序和处理器重排序，为程序员提供一致的内存可见性保证。

从JMM设计者的角度来说，在设计JMM时，需要考虑两个关键因素：

*   程序员对内存模型的使用。程序员希望内存模型易于理解，易于编程。程序员希望基于一个强内存模型(程序尽可能的顺序执行)来编写代码。
*   编译器和处理器对内存模型的实现。编译器和处理器希望内存模型对它们的束缚越少越好，这样它们就可以做尽可能多的优化(对程序重排序，做尽可能多的并发)来提高性能。编译器和处理器希望实现一个弱内存模型。

JMM设计就需要在这两者之间作出协调。JMM对程序采取了不同的策略：

*   对于会改变程序执行结果的重排序，JMM要求编译器和处理器必须禁止这种重排序。
*   对于不会改变程序执行结果的重排序，JMM对编译器和处理器不作要求（JMM允许这种重排序）。

介绍完了这几个基本概念，我们不难推断出JMM是围绕着在并发过程中如何处理原子性、可见性和有序性这三个特征来建立的：

*   原子性：由Java内存模型来直接保证的原子性操作就是我们前面介绍的8个原子操作指令，其中lock（lock指令实际在处理器上原子操作体现对总线加锁或对缓存加锁）和unlock指令操作JVM并未直接开放给用户使用，但是却提供了更高层次的字节码指令monitorenter和monitorexit来隐式使用这两个操作，这两个字节码指令反映到Java代码中就是同步块——synchronize关键字，因此在synchronized块之间的操作也具备原子性。除了synchronize，在Java中另一个实现原子操作的重要方式是自旋CAS，它是利用处理器提供的cmpxchg指令实现的。至于自旋CAS后面J.U.C中会详细介绍，它和volatile是整个J.U.C底层实现的核心。
*   可见性：可见性是指一个线程修改了共享变量的值，其他线程能够立即得知这个修改。而我们上文谈的happens-before原则禁止某些处理器和编译器的重排序，来保证了JMM的可见性。而体现在程序上，实现可见性的关键字包含了volatile、synchronize和final。
*   有序性：谈到有序性就涉及到前面说的重排序和顺序一致性内存模型。我们也都知道了as-if-serial是针对单线程程序有序的，即使存在重排序，但是最终程序结果还是不变的，而多线程程序的有序性则体现在JMM通过插入内存屏障指令，禁止了特定类型处理器的重排序。通过前面8个操作指令和happens-before原则介绍，也不难推断出，volatile和synchronized两个关键字来保证线程之间的有序性，volatile本身就包含了禁止指令重排序的语义，而synchronized则是由监视器法则获得。

## **J.U.C**

谈完了JMM，那么Java相关类库是如何实现的呢？这里就谈谈J.U.C（ java.util.concurrent），先来张J.U.C的思维导图

![enter image description here](http://static.codeceo.com/images/2016/08/913fd5ac37e3ecced00fc2fe6cda0f64.jpg)

不难看出，J.U.C由atomic、locks、tools、collections、[Executor](http://www.codeceo.com/article/java-executor-learning.html "Executor")这五部分组成。它们的实现基于volatile的读写和CAS所具有的volatile读和写。AQS（AbstractQueuedSynchronizer，队列同步器）、非阻塞数据结构和原子变量类，这些J.U.C中的基础类都是使用了这种模式实现的，而J.U.C中的高层类又依赖于这些基础类来实现的。从整体上看，J.U.C的实现示意图如下

![enter image description here](http://static.codeceo.com/images/2016/08/b1f654b37481509f47bf8218667b5714.png)

也许你对volatile和CAS的底层实现原理不是很了解，这里先这里先简单介绍下它们的底层实现

**volatile**

Java语言规范第三版对volatile的定义为：Java编程语言允许线程访问共享变量，为了确保共享变量能被准确和一致性的更新，线程应该确保通过排他锁单独获得这个变量。如果一个字段被声明为volatile，Java内存模型确保这个所有线程看到这个值的变量是一致的。而volatile是如何来保证可见性的呢？如果对声明了volatile的变量进行写操作，JVM就会向处理器发送一条Lock前缀的指令，将这个变量所在缓存行的数据写回到系统内存（Lock指令会在声言该信号期间锁总线/缓存，这样就独占了系统内存）。但是，就算是写回到内存，如果其他处理器缓存的值还是旧的，再执行计算操作就会有问题。所以，在多处理器下，为了保证各个处理器的缓存是一致的，就会实现缓存一致性协议，每个处理器通过嗅探在总线（注意处理器不直接跟系统内存交互，而是通过总线）上传播的数据来检查自己缓存的值是不是过期了，当处理器发现直接缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。

**CAS**

CAS其实应用挺广泛的，我们常常听到的悲观锁乐观锁的概念，乐观锁（无锁）指的就是CAS。这里只是简单说下在并发的应用，所谓的乐观并发策略，通俗的说，就是先进性操作，如果没有其他线程争用共享数据，那操作就成功了，如果共享数据有争用，产生了冲突，那就采取其他的补偿措施（最常见的补偿措施就是不断重试，治到成功为止，这里其实也就是自旋CAS的概念），这种乐观的并发策略的许多实现都不需要把线程挂起，因此这种操作也被称为非阻塞同步。而CAS这种乐观并发策略操作和冲突检测这两个步骤具备的原子性，是靠什么保证的呢？硬件，硬件保证了一个从语义上看起来需要多次操作的行为只通过一条处理器指令就能完成。

也许你会存在疑问，为什么这种无锁的方案一般会比直接加锁效率更高呢？这里其实涉及到线程的实现和线程的状态转换。实现线程主要有三种方式：使用内核线程实现、使用用户线程实现和使用用户线程加轻量级进程混合实现。而Java的线程实现则依赖于平台使用的线程模型。至于状态转换，Java定义了6种线程状态，在任意一个时间点，一个线程只能有且只有其中的一种状态，这6种状态分别是：新建、运行、无限期等待、限期等待、阻塞、结束。 Java的线程是映射到操作系统的原生线程之上的，如果要阻塞或唤醒一个线程，都需要操作系统来帮忙完成，这就需要从用户态转换到核心态中，因此状态转换需要耗费很多的处理器时间。对于简单的同步块（被synchronized修饰的方法），状态转换消耗的时间可能比用户代码执行的时间还要长。所以出现了这种优化方案，在操作系统阻塞线程之间引入一段自旋过程或一直自旋直到成功为止。避免频繁的切入到核心态之中。

但是这种方案其实也并不完美，在这里就说下CAS实现原子操作的三大问题

*   ABA问题。因为CAS需要在操作值的时候，检查值有没有变化，如果没有发生变化则更新，但是如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时会发现它的值没有变化，但是实际上发生变化了。ABA解决的思路是使用版本号。在变量前面追加上版本号，每次变量更新的时候把版本号加1。JDK的Atomic包里提供了一个类AtomicStampedReference来解决ABA问题。不过目前来说这个类比较“鸡肋”，大部分情况下ABA问题不会影响程序并发的正确性，如果需要解决ABA问题，改用原来的互斥同步可能会比原子类更高效。
*   循环时间长开销大。自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。所以说如果是长时间占用锁执行的程序，这种方案并不适用于此。
*   只能保证一个共享变量的原子操作。当对一个共享变量执行操作时，我们可以使用自旋CAS来保证原子性，但是对多个共享变量的操作时，自旋CAS就无法保证操作的原子性，这个时候可以用锁。

谈完了这两个概念，下面我们就来逐个分析这五部分的具体源码实现

**atomic**

atomic包的原子操作类提供了一种简单、性能高效、线程安全操作一个变量的方式。atomic包里一共13个类，属于4种类型的原子更新方式，分别是原子更新基本类型、原子更新数组、原子更新引用、原子更新属性。atomic包里的类基本使用Unsafe实现的包装类。

下面通过一个简单的CAS方式实现计数器（一个线程安全的计数器方法safeCount和一个非线程安全的计数器方法count）的示例来说下

```java
public class CASTest {

    public static void main(String[] args){
        final Counter cas=new Counter();
        List ts=new ArrayList(600);
        long start=System.currentTimeMillis();
        for(int j=0;j<100;j++){
            Thread t=new Thread(new Runnable() {
                @Override
                public void run() {
                    for(int i=0;i<10000;i++){
                        cas.count();
                        cas.safeCount();
                    }
                }
            });
            ts.add(t);
        }
        for(Thread t:ts){
            t.start();
        }

        for(Thread t:ts){
            try {
                t.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println(cas.i);
        System.out.println(cas.atomicI.get());
        System.out.println(System.currentTimeMillis()-start);
    }

}

public class Counter {
    public AtomicInteger atomicI=new AtomicInteger(0);
    public int i=0;

    /**
    * 使用CAS实现线程安全计数器
    */
    public void safeCount(){
        for(;;){
            int i=atomicI.get();
            boolean suc=atomicI.compareAndSet(i,++i);
            if(suc){
                break;
            }
        }
    }

    /**
    * 非线程安全计数器
    */
    public void count(){
        i++;
    }

}
```

safeCount()方法的代码块其实是getandIncrement()方法的实现，源码for循环体第一步优先取得atomicI里存储的数值，第二步对atomicI的当前数值进行加1操作，关键的第三步调用compareAndSet()方法来进行原子更新操作，该方法先检查当前数值是否等于current，等于意味着atomicI的值没有被其他线程修改过，则将atomicI的当前数值更新成next的值，如果不等compareAndSet()方法会返回false，程序则进入for循环重新进行compareAndSet()方法操作进行不断尝试直到成功为止。在这里我们跟踪下compareAndSet()方法如下

```java
public final boolean compareAndSet(int expect, int update) {
    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);
}
```

从上面源码我们发现是使用Unsafe实现的，其实atomic里的类基本都是使用Unsafe实现的。我们再回到这个本地方法调用，这个本地方法在openjdk中依次调用c++代码为unsafe.cpp、atomic.app和atomic_windows_x86.inline.hpp。关于本地方法实现的源代码这里就不贴出来了，其实大体上是程序会根据当前处理器的类型来决定是否为cmpxchg指令添加lock前缀。如果程序是在多处理器上运行，就为cmpxchg指令加上lock前缀（Lock Cmpxchg）。反之，如果程序是在单处理器上运行，就省略lock前缀（单处理器自身就会维护单处理器内的顺序一致性，不需要lock前缀提供的内存屏障效果）。

**locks**

锁是用来控制多个线程访问共享资源的形式，Java SE 5之后，J.U.C中新增了locks来实现锁功能，它提供了与synchronized关键字类似的同步功能。只是在使用时需要显示的获取和释放锁。虽然它缺少了隐式获取和释放锁的便捷性，但是却拥有了锁获取和释放的可操作性、可中断的获取锁及超时获取锁等多种synchronized关键字不具备的同步特性。

locks在这我们只介绍下核心的AQS(AbstractQueuedSynchronizer，队列同步器)，AQS是用来构建锁或者其他同步组件的基础框架，它使用一个用volatile修饰的int成员变量表示同步状态。通过内置的FIFO队列来完成资源获取线程的排队工作。同步器的主要使用方式是继承，子类通过继承同步器并实现它的抽象方法来管理同步状态，在抽象方法的实现过程免不了要对同步状态进行更改，这时候就会使用到AQS提供的3个方法：getState()、setState()和compareAndSetState()来进行操作，这是因为它们能够保证状态的改变是原子性的。为什么这么设计呢？因为锁是面向使用者的，它定义了使用者与锁交互的接口，隐藏了实现细节，而AQS面向的是锁的实现者，它简化了锁的实现方式，屏蔽了同步状态管理、线程的排队、等待与唤醒等底层操作。锁和AQS很好的隔离了使用者和实现者锁关注的领域。

现在我们就自定义一个独占锁来详细解释下AQS的实现机制

```java
public class Mutex implements Lock {
    private static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = -4387327721959839431L;

        protected boolean isHeldExclusively() {
            return getState() == 1;
        }

        public boolean tryAcquire(int acquires) {
            assert acquires == 1; // Otherwise unused
            if (compareAndSetState(0, 1)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
        }

        protected boolean tryRelease(int releases) {
            assert releases == 1; // Otherwise unused
            if (getState() == 0)
                throw new IllegalMonitorStateException();
            setExclusiveOwnerThread(null);
            setState(0);
            return true;
        }

        Condition newCondition() {
            return new ConditionObject();
        }
    }

    private final Sync sync = new Sync();

    public void lock() {
        sync.acquire(1);
    }

    public boolean tryLock() {
        return sync.tryAcquire(1);
    }

    public void unlock() {
        sync.release(1);
    }

    public Condition newCondition() {
        return sync.newCondition();
    }

    public boolean isLocked() {
        return sync.isHeldExclusively();
    }

    public boolean hasQueuedThreads() {
        return sync.hasQueuedThreads();
    }

    public void lockInterruptibly() throws InterruptedException {
        sync.acquireInterruptibly(1);
    }

    public boolean tryLock(long timeout, TimeUnit unit) throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(timeout));
    }
}
```

实现自定义组件的时候，我们可以看到，AQS可重写的方法是tryAcquire()——独占式获取同步状态、tryRelease()——独占式释放同步状态、tryAcquireShared()——共享式获取同步状态、tryReleaseShared ()——共享式释放同步状态、isHeldExclusively()——是否被当前线程所独占。这个示例中，独占锁Mutex是一个自定义同步组件，它在同一时刻只允许一个线程占有锁。Mutex中定义了一个静态内部类，该内部类继承了同步器并实现了独占式获取和释放同步状态。在tryAcquire()中，如果经过CAS设置成功（同步状态设置为1），则表示获取了同步状态，而在tryRelease()中，只是将同步状态重置为0。接着我们对比一下重入锁（ReentrantLock）的源码实现

```java
public class ReentrantLock implements Lock, java.io.Serializable {
    private static final long serialVersionUID = 7373984872572414699L;
    /** Synchronizer providing all implementation mechanics */
    private final Sync sync;

    /**
     * Base of synchronization control for this lock. Subclassed
     * into fair and nonfair versions below. Uses AQS state to
     * represent the number of holds on the lock.
     */
    abstract static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = -5179523762034025860L;

        /**
         * Performs {@link Lock#lock}. The main reason for subclassing
         * is to allow fast path for nonfair version.
         */
        abstract void lock();

        /**
         * Performs non-fair tryLock.  tryAcquire is
         * implemented in subclasses, but both need nonfair
         * try for trylock method.
         */
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }

        protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }

        protected final boolean isHeldExclusively() {
            // While we must in general read state before owner,
            // we don't need to do so to check if current thread is owner
            return getExclusiveOwnerThread() == Thread.currentThread();
        }

        final ConditionObject newCondition() {
            return new ConditionObject();
        }

        // Methods relayed from outer class

        final Thread getOwner() {
            return getState() == 0 ? null : getExclusiveOwnerThread();
        }

        final int getHoldCount() {
            return isHeldExclusively() ? getState() : 0;
        }

        final boolean isLocked() {
            return getState() != 0;
        }

        /**
         * Reconstitutes this lock instance from a stream.
         * @param s the stream
         */
        private void readObject(java.io.ObjectInputStream s)
            throws java.io.IOException, ClassNotFoundException {
            s.defaultReadObject();
            setState(0); // reset to unlocked state
        }
    }

    /**
     * Sync object for non-fair locks
     */
    static final class NonfairSync extends Sync {
    //todo sth...
    }
    //todo sth...
}
```

重入锁分公平锁和不公平锁，默认使用的是不公平锁，在这我们看到实现重入锁大体上跟我们刚才自定义的独占锁差不多，但是有什么区别呢？我们看看重入锁nonfairTryAcquire()方法实现：首先获取同步状态（默认是0），如果是0的话，CAS设置同步状态，非0的话则判断当前线程是否已占有锁，如果是的话，则偏向更新同步状态。从这里我们不难推断出重入锁的概念，同一个线程可以多次获得同一把锁，在释放的时候也必须释放相同次数的锁。通过对比相信大家对自定义一个锁有了一个初步的概念，也许你存在疑问我们重写的这几个方法在AQS哪地方用呢？现在我们来继续往下跟踪，我们深入跟踪下刚才自定义独占锁lock()方法里面acquire()的实现

```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

这个方法在AQS类里面，看到里面的tryAcquire(arg)大家也就明白了，tryAcquire(arg)方法获取同步状态，后面acquireQueued(addWaiter(Node.EXCLUSIVE), arg)方法就是说的节点构造、加入同步队列及在同步队列中自旋等待的AQS没暴露给我们的相关操作。大体的流程就是首先调用自定义同步器实现的tryAcquire()方法，该方法保证线程安全的获取同步状态，如果获取同步状态失败，则构造同步节点（独占式Node.EXCLUSIVE，同一时刻只能有一个线程成功获取同步状态）并通过addWaiter()方法将该节点加入到同步队列的尾部，最后调用acquireQueued()方法，使得该节点以“死循环”的方式获取同步状态。如果获取不到则阻塞节点中的线程，而被阻塞线程的唤醒主要靠前驱节点的出队或阻塞线程被中断来实现。也许你还是不明白刚才所说的，那么我们继续跟踪下addWaiter()方法的实现

```java
private Node addWaiter(Node mode) {
    Node node = new Node(Thread.currentThread(), mode);
    // Try the fast path of enq; backup to full enq on failure
    Node pred = tail;
    if (pred != null) {
        node.prev = pred;
        if (compareAndSetTail(pred, node)) {
            pred.next = node;
            return node;
        }
    }
    enq(node);
    return node;
}

private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) { // Must initialize
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

上面的代码通过使用compareAndSetTail()方法来确保节点能够被线程安全添加。在enq()方法中，同步器通过“死循环”来确保节点的正确添加，在”死循环“中只有通过CAS将节点设置成为尾节点之后，当前线程才能够从该方法返回，否则，当前线程不断地尝试重试设置。

在节点进入同步队列之后，发生了什么呢？现在我们继续跟踪下acquireQueued()方法

```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

从上面的代码我们不难看出，节点进入同步队列之后，就进入了一个自旋的过程，每个节点（或者说每个线程）都在自省的观察，当条件满足时（自己的前驱节点是头节点就进行CAS设置同步状态）就获得同步状态，然后就可以从自旋的过程中退出，否则依旧在这个自旋的过程中。

**collections**

从前面的思维导图我们可以看到并发容器包括链表、队列、HashMap等.它们都是线程安全的。

*   ConcurrentHashMap : 一个高效的线程安全的HashMap。
*   CopyOnWriteArrayList : 在读多写少的场景中,性能非常好,远远高于vector。
*   ConcurrentLinkedQueue : 高效并发队列,使用链表实现,可以看成线程安全的LinkedList。
*   BlockingQueue : 一个接口,JDK内部通过链表,数组等方式实现了这个接口,表示阻塞队列,非常适合用作数据共享 。
*   ConcurrentSkipListMap : 跳表的实现,这是一个Map,使用跳表数据结构进行快速查找 。

另外Collections工具类可以帮助我们将任意集合包装成线程安全的集合。在这里重点说下ConcurrentHashMap和BlockingQueue这两个并发容器。

我们都知道HashMap线程不安全的，而我们可以通过Collections.synchronizedMap(new HashMap<>())来包装一个线程安全的HashMap或者使用线程安全的HashTable，但是它们的效率都不是很好，这时候我们就有了ConcurrentHashMap。为什么ConcurrentHashMap高效且线程安全呢？其实它使用了锁分段技术来提高了并发的访问率。假如容器里有多把锁，每一把锁用于锁容器的一部分数据，那么当多线程访问容器里不同数据段的数据时，线程间就不会存在锁竞争，从而可以有效地提高并发访问效率，这就是锁分段技术。首先将数据分成一段段的存储，然后给每段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。而既然数据被分成了多个段，线程如何定位要访问的段的数据呢？这里其实是通过散列算法来定位的。

现在来谈谈阻塞队列，阻塞队列其实跟后面要谈的线程池息息相关的，JDK7提供了7个阻塞队列，分别是

*   ArrayBlockingQueue ：一个由数组结构组成的有界阻塞队列。
*   LinkedBlockingQueue ：一个由链表结构组成的有界阻塞队列。
*   PriorityBlockingQueue ：一个支持优先级排序的无界阻塞队列。
*   DelayQueue：一个使用优先级队列实现的无界阻塞队列。
*   SynchronousQueue：一个不存储元素的阻塞队列。
*   LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。
*   LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。

如果队列是空的，消费者会一直等待，当生产者添加元素时候，消费者是如何知道当前队列有元素的呢？如果让你来设计阻塞队列你会如何设计，让生产者和消费者能够高效率的进行通讯呢？让我们先来看看JDK是如何实现的。

使用通知模式实现。所谓通知模式，就是当生产者往满的队列里添加元素时会阻塞住生产者，当消费者消费了一个队列中的元素后，会通知生产者当前队列可用。通过查看JDK源码发现ArrayBlockingQueue使用了Condition来实现，代码如下：

```java
private final Condition notFull;
private final Condition notEmpty;

    public ArrayBlockingQueue(int capacity, boolean fair) {
        //省略其他代码
        notEmpty = lock.newCondition();
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

    public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == 0)
                notEmpty.await();
            return extract();
        } finally {
            lock.unlock();
        }
    }

    private void insert(E x) {
        items[putIndex] = x;
        putIndex = inc(putIndex);
        ++count;
        notEmpty.signal();
    }
```

当我们往队列里插入一个元素时，如果队列不可用，阻塞生产者主要通过LockSupport.park(this)来实现

```java
public final void await() throws InterruptedException {
    if (Thread.interrupted())
        throw new InterruptedException();
    Node node = addConditionWaiter();
    int savedState = fullyRelease(node);
    int interruptMode = 0;
    while (!isOnSyncQueue(node)) {
        LockSupport.park(this);
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    if (node.nextWaiter != null) // clean up if cancelled
        unlinkCancelledWaiters();
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
}
```

继续进入源码，发现调用setBlocker先保存下将要阻塞的线程，然后调用unsafe.park阻塞当前线程。

```java
public static void park(Object blocker) {
    Thread t = Thread.currentThread();
    setBlocker(t, blocker);
    unsafe.park(false, 0L);
    setBlocker(t, null);
}
```

unsafe.park是个native方法，代码如下：

```java
public native void park(boolean isAbsolute, long time);
```

park这个方法会阻塞当前线程，只有以下四种情况中的一种发生时，该方法才会返回。

*   与park对应的unpark执行或已经执行时。注意：已经执行是指unpark先执行，然后再执行的park。
*   线程被中断时。
*   如果参数中的time不是零，等待了指定的毫秒数时。
*   发生异常现象时。这些异常事先无法确定。

我们继续看一下JVM是如何实现park方法的，park在不同的操作系统使用不同的方式实现，在linux下是使用的是系统方法pthread_cond_wait实现。实现代码在JVM源码路径src/os/linux/vm/os_linux.cpp里的 os::PlatformEvent::park方法，代码如下：

```java
void os::PlatformEvent::park() {
            int v ;
            for (;;) {
                v = _Event ;
            if (Atomic::cmpxchg (v-1, &_Event, v) == v) break ;
            }
            guarantee (v >= 0, "invariant") ;
            if (v == 0) {
            // Do this the hard way by blocking ...
            int status = pthread_mutex_lock(_mutex);
            assert_status(status == 0, status, "mutex_lock");
            guarantee (_nParked == 0, "invariant") ;
            ++ _nParked ;
           while (_Event < 0) {          
           status = pthread_cond_wait(_cond, _mutex);        
           // for some reason, under 2.7 lwp_cond_wait() may return ETIME ...           
           // Treat this the same as if the wait was interrupted         
            if (status == ETIME) { status = EINTR; }        
            assert_status(status == 0 || status == EINTR, status, "cond_wait");        
           }    
           -- _nParked ;          
          // In theory we could move the ST of 0 into _Event past the unlock(),       
          // but then we'd need a MEMBAR after the ST.       
          _Event = 0 ;     
         status = pthread_mutex_unlock(_mutex);    
         assert_status(status == 0, status, "mutex_unlock");       
         }        
          guarantee (_Event >= 0, "invariant") ;
        }
    }
```

pthread_cond_wait是一个多线程的条件变量函数，cond是condition的缩写，字面意思可以理解为线程在等待一个条件发生，这个条件是一个全局变量。这个方法接收两个参数，一个共享变量_cond，一个互斥量_mutex。而unpark方法在linux下是使用pthread_cond_signal实现的。park 在windows下则是使用WaitForSingleObject实现的。

当队列满时，生产者往阻塞队列里插入一个元素，生产者线程会进入WAITING (parking)状态。

**executor**

Executor框架提供了各种类型的线程池，不同的线程池应用了前面介绍的不同的堵塞队列

![enter image description here](http://static.codeceo.com/images/2016/08/ccc9680e4fe52fdfb6e8dd1bec6c881b.jpg)

Executor框架最核心的类是ThreadPoolExecutor，它是线程池的实现类。 对于核心的几个线程池，无论是newFixedThreadPool()、newSingleThreadExecutor()还是newCacheThreadPool()方法，虽然看起来创建的线程具有完全不同的功能特点，但其内部均使用了ThreadPoolExecutor实现

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
            0L, TimeUnit.MILLISECONDS,
            new LinkedBlockingQueue());
}
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                    0L, TimeUnit.MILLISECONDS,
                    new LinkedBlockingQueue()));
}
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
            60L, TimeUnit.SECONDS,
            new SynchronousQueue());
}
```

*   newFixedThreadPool()方法的实现，它返回一个corePoolSize和maximumPoolSize一样的，并使用了LinkedBlockingQueue任务队列（无界队列）的线程池。当任务提交非常频繁时，该队列可能迅速膨胀，从而系统资源耗尽。
*   newSingleThreadExecutor()返回单线程线程池，是newFixedThreadPool()方法的退化，只是简单的将线程池数量设置为1。
*   newCachedThreadPool()方法返回corePoolSize为0而maximumPoolSize无穷大的线程池，这意味着没有任务的时候线程池内没有现场，而当任务提交时，该线程池使用空闲线程执行任务，若无空闲则将任务加入SynchronousQueue队列，而SynchronousQueue队列是直接提交队列，它总是破事线程池增加新的线程来执行任务。当任务执行完后由于corePoolSize为0，因此空闲线程在指定时间内（60s）被回收。对于newCachedThreadPool()，如果有大量任务提交，而任务又不那么快执行时，那么系统变回开启等量的线程处理，这样做法可能会很快耗尽系统的资源，因为它会增加无穷大数量的线程。

由以上线程池的实现可以看到，它们都只是ThreadPoolExecutor类的封装。我们看下ThreadPoolExecutor最重要的构造函数：

```java
public ThreadPoolExecutor(
            //核心线程池，指定了线程池中的线程数量
            int corePoolSize,
            //基本线程池，指定了线程池中的最大线程数量
            int maximumPoolSize,
            //当前线程池数量超过corePoolSize时，多余的空闲线程的存活时间，即多次时间内会被销毁。
            long keepAliveTime,
            //keepAliveTime的单位
            TimeUnit unit,
            //任务队列，被提交但尚未被执行的任务。
            BlockingQueue workQueue,
            //线程工厂，用于创建线程，一般用默认的即可
            ThreadFactory threadFactory,
            //拒绝策略，当任务太多来不及处理，如何拒绝任务。
            RejectedExecutionHandler handler)
```

ThreadPoolExecutor的任务调度逻辑如下

![enter image description here](http://static.codeceo.com/images/2016/08/beeb8bc577fcec5df499c8cf1f56a31f.jpg)

从上图我们可以看出，当提交一个新任务到线程池时，线程池的处理流程如下：

*   首先线程池判断基本线程池是否已满，如果没满，创建一个工作线程来执行任务。满了，则进入下个流程。
*   其次线程池判断工作队列是否已满，如果没满，则将新提交的任务存储在工作队列里。满了，则进入下个流程。
*   最后线程池判断整个线程池是否已满，如果没满，则创建一个新的工作线程来执行任务，满了，则交给饱和策略来处理这个任务。

下面我们来看看ThreadPoolExecutor核心调度代码

```java
public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        int c = ctl.get();
        /**
        * workerCountOf(c)获取当前线程池线程总数
        * 当前线程数小于corePoolSize核心线程数时，会将任务通过addWorker(command, true)方法直接调度执行。
        * 否则进入下个if，将任务加入等待队列
        **/
        if (workerCountOf(c) < corePoolSize) {
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        /**
        * workQueue.offer(command) 将任务加入等待队列。
        * 如果加入失败（比如有界队列达到上限或者使用了synchronousQueue）则会执行else。
        *
        **/
        if (isRunning(c) && workQueue.offer(command)) {
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))
                reject(command);
            else if (workerCountOf(recheck) == 0)
                addWorker(null, false);
        }
        /**
        * addWorker(command, false)直接交给线程池，
        * 如果当前线程已达到maximumPoolSize，则提交失败执行reject()拒绝策略。
        **/
        else if (!addWorker(command, false))
            reject(command);
    }
```

从上面的源码我们可以知道execute的执行步骤：

*   如果当前运行的线程少于corePoolSize，则创建新线程来执行任务（注意，执行这一步骤需要获取全局锁）。
*   如果运行的线程等于或多于corePoolSize，则将任务加入到BlockingQueue。
*   如果无法将任务假如BlockingQueue（队列已满），则创建新的线程来处理任务（注意，执行这一步骤需要获取全局锁）。
*   如果创建新线程将使当前运行的线程超出maximumPoolSize，任务将被拒绝，并调用RejectedExecutionHandler.rejectedExecution()方法。

ThreadPoolExecutor采取上述步骤的总体设计思路，是为了在执行execute()方法时，尽可能的避免获取全局锁（那将会是一个严重的 可伸缩瓶颈）。在ThreadPoolExecutor完成预热之后（当前运行的线程数大于等于corePoolSize），几乎所有的execute()方法调用都是执行步骤2，而步骤2不需要获取全局锁。

## 参考阅读

本文部分内容参考自《Java并发编程的艺术》、《深入理解Java虚拟机（第2版）》、《实战Java高并发程序设计》、《深入Java内存模型》、《Java并发编程实践》，感兴趣的可自行查阅。
