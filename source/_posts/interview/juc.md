---
title: java并发
date: 2021-07-29 11:10:43
---

<!-- toc -->


### - Java并发容器有哪些

concurrentHashmap:   线程安全的HashMap, Node数组+链表 + 红黑树  
copyonwriteArrayList:   线程安全的List，在读多写少的场合性能非常好    
concurrentLinkedQueue:   高效的并发队列，使用链表实现。
可以看做一个线程安全的 LinkedList   
BlockingQueue(接口)   
1 ArrayBlockingQueue:  有界队列实现类 底层Object数组    
2 LinkedBlockingQueue: 单向链表  
3 PriorityBlockingQueue: 支持优先级的无界阻塞队列 

### - 谈谈对ConcurrentHashMap的了解

1.7   结构： 分段的数组+链表; 
      并发：分段锁，锁一段数组 segment，默认有16个segment, 并发度只有默认是16
      线程安全: segment上锁，继承ReentrantLock可重入锁

1.8   结构： Node数组+链表 + 红黑树; 
      并发： 并发控制使用 synchronized 和 CAS
      线程安全: CAS和synchronized来保证并发安全, synchronized只锁定当前链表或红黑二叉树的首节点, 只要hash不冲突，就不会产生并发，效率又提升N倍

get是怎么做的
1. hash。 hashcode高低16位异或得到hash值，(h ^ h >>> 16) & 2147483647;
2. 桶位节点就是要找节点，直接返回该节点
3. 如果节点hash值小于0，说明是已迁移的节点或者红黑树bin，调用node.find(), TreeBin 和 ForwardingNode 都继承Node重写find方法
4. 剩下情况就是链表，遍历判断是否节点存在

注：ForwordingNode的hash值为-1，红黑树的根结点的hash值为-2。 TreeBin 和 ForwardingNode 都继承Node重写find方法

put是怎么做的
1. hash。 hashcode高低16位异或得到hash值，(h ^ h >>> 16) & 2147483647;
2. 桶位为空，cas设置新node，U.compareAndSetObject
3. 其他操作，加synchronized锁住头结点，如果是树，调用添加到树中方法，如果是链表，添加到尾端。
4. 判断是否到了8个，链表变树； 判断size是否到了阈值，触发扩容。size总数增加cas (U.compareAndSetLong)

resize是怎么做的
1. newTable, 大小是旧表的2倍
2. 头结点加锁，不允许其他线程更改，但可get，
3. 判断桶中各节点位置，复制一模一样的节点到新表（非直接移过去，还要get），到新表可能在旧桶位也可能在新桶位
4. 分配结束之后将头结点设置为fwd节点（指向新表）

白话：线程安全的map。1.7版本是使用分段锁实现的线程安全，segment上加锁，在构建的时候可以设置大小，默认是16，就是并发度默认只有16. 1.8版本的线程安全是用synchronized+cas实现的，使用synchronized锁住链表或红黑树的头结点，使用cas来进行size++和首个节点入桶。1.8的并发度是随着桶位增加而增加的，所以并发效率会随扩容提升很多倍。


### - 谈谈对Java内存模型的了解


jmm是为了解决线程间通信问题，线程间通信通常有两种解决方法，共享内存或通知机制， jmm使用了共享内存的方式，jmm定义了一套happens-before规则来规范多线程下的执行顺序和多线程下变量的可见性问题，happens-before规则底层是通过禁止部分编译器和处理器的指令重排序实现的，happens-before有八个，分别是

程序顺序规则：一个线程中的每个操作，发生在该线程中任意后续操作之前    
监视器锁规则：对一个锁的解锁，发生在随后对这个锁的加锁之前     
volatile变量规则：对一个volatile域的写，发生在任意后续对这个volatile域的读之前   
传递性：如果A发生在B之前，B发生在C之前，那么A一定发生在C之前    
线程启动规则：Thread对象的start()方法先行发生于此线程的每个一个动作；  
线程中断规则：对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生；  
线程终结规则：线程中所有的操作都先行发生于线程的终止检测，我们可以通过Thread.join()方法结束、Thread.isAlive()的返回值手段检测到线程已经终止执行；    
对象终结规则：一个对象的初始化完成先行发生于他的finalize()方法的开始； 

定义：JMM是什么，Java内存模型是要解决什么问题:    
JMM通过happens-before规则简单易懂让Java程序员理解JMM提供的内存可见性保证。

Java内存模型是在讲线程间通信机制。 线程间通信有两种：共享内存和消息传递.     
Java采用的是共享内存的模型实现的线程之间通信, 隐式进行, 对程序员透明。    
JMM通过控制主内存与每个线程的本地内存之间的交互，决定一个线程对共享变量的写入何时对另一个线程可见。解决多线程之间可见性问题。   
（本地内存是JMM的一个抽象概念，并不真实存在。它涵盖了缓存，写缓冲区、寄存器以及其他的硬件和编译器优化）

怎么做，怎么实现共享内存模型的通信: 
对于Java程序员来说，happens-before规则简单易懂，它避免Java程序员为了理解JMM提供的内存可见性保证而去学习复杂的重排序规则以及这些规则的具体实现方法。  
一个happens-before规则对应于一个或多个编译器和处理器重排序规则. 
1. happens-before规则
2. 禁止指令重排序（根据happens-before规则）
 - 编译器重排序，JMM会禁止特定类型的编译器重排序。   
 - 处理器重排序，JMM的处理重排序规则会要求java编译器在生成指令序列时，插入特定类型的内存屏障指令，通过内存屏障指令来禁止特定类型的处理器重排序。为程序员提供一致的内存可见性保证。

### - volatile原理

1. Volatile的特征   
A、禁止指令重排（有例外）   
B、可见性
2. Volatile的内存语义  
当写一个volatile变量时，JMM会把线程对应的本地内存中的共享变量值刷新到主内存。   
当读一个volatile变量时，JMM会把线程对应的本地内存置为无效，线程接下来将从主内存中读取共享变量
3. Volatile的重排序   
两个volatile变量操作不能够进行重排序；

白话：volatile有两个特性，可见性和禁止指令重排序。可见性，当一个线程更新volatile变量后，这个变量会立即写入到主内存中，其他线程读取时会从主内存中读取最新的值。禁止指令重排序，两个对volatile变量的操作不能被重排序，底层是通过内存栅栏实现的。


### - java的乐观锁CAS锁原理

CAS英文全称Compare and Swap，直白翻译过来即比较并交换，是一种无锁算法，在不使用锁即没有线程阻塞下实现多线程之间的变量同步，基于处理器的读-改-写原子指令来操作数据，可以保证数据在并发操作下的一致性。

CAS包含三个操作数：内存位置V，预期值A，写入的新值B。在执行数据操作时，当且仅当V的值等于A时，CAS才会通过原子操作方式用新值B来更新V的值（无论操作是否成功都会返回）。

CAS的含义是：我认为V的值应该为A，如果是，那么将V的值更新为B，否则不修改并告诉V的值实际为多少

CAS是怎么获取到预期值的?  通过unsafe类获取到的
```
public final int getAndAddInt(Object var1, long var2, int var4) {
   int var5;
   do {
      var5 = this.getIntVolatile(var1, var2);
   } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

   return var5;
}
```

CAS修改的值为什么要是volatile? 可见性

在compareAndSet的操作中，JNI借助CPU指令完成的，属于原子操作，保证多个线程在执行过程中看到同一个变量的修改值  
unsafe：操作内存内存空间   
valueOffset：value在内存中的偏移量（把存储单元的实际地址与其所在段的段地址之间的距离称为段内偏移），这里我们可以简单认为是内存地址，在类加载的时候，通过unsafe获取到value的偏移量。  
value： 顾名思义代表存储值，被volatile修饰，保证在线程间的可见性   

CAS问题：
1.ABA问题。CAS需要在操作值的时候检查内存值是否发生变化，没有发生变化才会更新内存值。但是如果内存值原来是A，后来变成了B，然后又变成了A，那么CAS进行检查时会发现值没有发生变化，但是实际上是有变化的。ABA问题的解决思路就是在变量前面添加版本号，每次变量更新的时候都把版本号加一，这样变化过程就从“A－B－A”变成了“1A－2B－3A”。     
JDK从1.5开始提供了AtomicStampedReference类来解决ABA问题，具体操作封装在compareAndSet()中。compareAndSet()首先检查当前引用和当前标志与预期引用和预期标志是否相等，如果都相等，则以原子方式将引用值和标志的值设置为给定的更新值。      
2.循环时间长开销大。CAS操作如果长时间不成功，会导致其一直自旋，给CPU带来非常大的开销。    
3.只能保证一个共享变量的原子操作。对一个共享变量执行操作时，CAS能够保证原子操作，但是对多个共享变量操作时，CAS是无法保证操作的原子性的。     
Java从1.5开始JDK提供了AtomicReference类来保证引用对象之间的原子性，可以把多个变量放在一个对象里来进行CAS操作。   
    
白话：CAS是Compare and Swap，比较并交换，是一种乐观锁实现线程安全的方式，更加轻量。底层是通过cpu的原子指令实现的比较并替换。使用的时候参数有内存位置，预期值，写入的新值，当通过内存位置拿到的值和预期值相等时，就用新值进行替换，整个操作是原子的，cpu指令保证。

### - sychronized 和 valotile 的区别

1. volatile更轻量，性能更好，但volatile只能用于变量而synchronized关键字可以修饰方法以及代码块  
2. 多线程访问volatile关键字不会发生阻塞，而synchronized关键字可能会发生阻塞  
3. volatile关键字能保证数据的可见性，但不能保证数据的原子性 （eg： i++）.synchronized关键字两者都能保证
4. volatile关键字主要用于解决变量在多个线程之间的可见性，而 synchronized关键字解决的是多个线程之间访问资源的同步性



### - sychronized reentranlock 的区别

1. synchronized 依赖于 JVM 而 ReentrantLock 依赖于 API
2. 等待可中断  lock.lockInterruptibly()， 线程可以选择放弃等待，改为处理其他事情
3. 可实现公平锁
4. 可实现选择性通知（锁可以绑定多个条件）

### - 弱引用

强引用

任何被强引用指向的对象都不能被垃圾回收器回收。

软引用

如果有软引用指向这些对象，则只有在内存空间不足时才回收这些对象（回收发生在OutOfMemoryError之前）。

弱引用

如果一个对象只有弱引用指向它，垃圾回收器会立即回收该对象，这是一种急切回收方式。

虚引用

虚引等同于没有引用，拥有虚引用的对象可以在任何时候被垃圾回收器回收。

弱引用的出现就是为了垃圾回收服务的。它引用一个对象，但是并不阻止该对象被回收。
如果使用一个强引用的话，只要该引用存在，那么被引用的对象是不能被回收的。
弱引用则没有这个问题。在垃圾回收器运行的时候，如果一个对象的所有引用都是弱引用的话，该对象会被回收

![](https://ipic-1252327316.cos.ap-beijing.myqcloud.com/image/%E5%BC%BA%E8%BD%AF%E5%BC%B1%E8%99%9A.png)

### - ThreadLocal原理

用多线程多份数据，来避免线程不安全

每个Thread 维护一个 ThreadLocalMap 映射表，这个映射表的 key是 ThreadLocal 实例本身，value 是真正需要存储的 Object。也就是说 ThreadLocal 本身并不存储值，它只是作为一个 key 来让线程从 ThreadLocalMap 获取 value。

由于每一条线程均含有各自私有的ThreadLocalMap容器，这些容器相互独立互不影响，因此不会存在线程安全性问题，从而也无需使用同步机制来保证多条线程访问容器的互斥性。

ThreadLocalMap 是使用 ThreadLocal 的弱引用作为 Key的，弱引用的对象在 GC 时会被回收。  
Entry 继承 WeekReference<ThreadLocal<?>>，也就是说，一个Entry对象是由ThreadLocal对象和一个Object（ThreadLocal关联的对象）组成。   

为什么选择弱引用？   
为了应对非常大和长时间的用途，哈希表使用弱引用的 key
由于ThreadLocalMap的生命周期跟Thread一样长，如果都没有手动删除对应key，都会导致内存泄漏，但是使用弱引用可以多一层保障：弱引用ThreadLocal key不会内存泄漏，对应的value在下一次ThreadLocalMap调用set,get,remove的时候会被清除

内存泄漏问题  
在ThreadLocalMap中，只有key是弱引用，value仍然是一个强引用。当某一条线程中的ThreadLocal使用完毕，没有强引用指向它的时候，这个key指向的对象就会被垃圾收集器回收，从而这个key就变成了null；然而，此时value和value指向的对象之间仍然是强引用关系，只要这种关系不解除，value指向的对象永远不会被垃圾收集器回收，从而导致内存泄漏！

解决办法：
ThreadLocal提供了这个问题的解决方案，
其实，ThreadLocalMap的设计中已经考虑到这种情况，也加上了一些防护措施：在ThreadLocal的get(),set(),remove()的时候都会清除线程ThreadLocalMap里所有key为null的value。
但是这些被动的预防措施并不能保证不会内存泄漏：
使用static的ThreadLocal，延长了ThreadLocal的生命周期，可能导致的内存泄漏。
分配使用了ThreadLocal又不再调用get(),set(),remove()方法，那么就会导致内存泄漏。

ThreadLocal最佳实践   
综合上面的分析，我们可以理解ThreadLocal内存泄漏的前因后果，那么怎么避免内存泄漏呢？  
每次使用完ThreadLocal，都调用它的remove()方法，清除数据。   
在使用线程池的情况下，没有及时清理ThreadLocal，不仅是内存泄漏的问题，更严重的是可能导致业务逻辑出现问题。所以，使用ThreadLocal就跟加锁完要解锁一样，用完就清理。

### - AQS同步器原理？tryAcquire的过程？

**AQS使用一个volatile的int类型的成员变量来表示同步状态，通过内置的FIFO队列来完成资源获取的排队工作。AQS通过CAS完成对state值的修改

核心思想是，如果被请求的共享资源空闲，将当前请求资源的线程设置为有效的工作线程，将共享资源设置为锁定状态；如果共享资源被占用，将暂时获取不到锁的线程加入到队列中, 需要一定的阻塞等待唤醒机制机制来保证锁分配。这个机制主要用的是CLH队列实现的**

AQS是通过将每条请求共享资源的线程封装成一个节点来实现锁的分配。

通过简单的几行代码就能实现同步功能，这就是AQS的强大之处。

自定义同步器实现的相关方法也只是为了通过修改State字段来实现多线程的独占模式或者共享模式。自定义同步器需要实现以下方法（ReentrantLock需要实现的方法如下，并不是全部）：

ReentrantLock这类自定义同步器自己实现了获取锁和释放锁的方式，而其余的等待队列的处理、线程中断等功能，异常与性能处理，还有并发优化等细节工作，都是由AQS统一提供，这也是AQS的强大所在。对同步器这类应用层来说，AQS屏蔽了底层的，同步器只需要设计自己的加锁和解锁逻辑即可

一般来说，自定义同步器要么是独占方式，要么是共享方式，它们也只需实现tryAcquire-tryRelease、tryAcquireShared-tryReleaseShared中的一种即可。AQS也支持自定义同步器同时实现独占和共享两种方式，如ReentrantReadWriteLock。ReentrantLock是独占锁，所以实现了tryAcquire-tryRelease。

独占与共享最大不同就在各自的tryacquire里，对于独占来说只有true或false，只有一个线程得以执行任务；而对于共享锁的tryAcquireShared来说，线程数没达到限制都可以直接执行。
但本质上都是对AQS同步状态的修改，一个是0与1之间，另一个允许更多而已

应用：
1.ReentrantLock   
使用AQS保存锁重复持有的次数。当一个线程获取锁时，ReentrantLock记录当前获得锁的线程标识，用于检测是否重复获取，以及错误线程试图解锁操作时异常情况的处理。  
2.Semaphore   
使用AQS同步状态来保存信号量的当前计数。tryRelease会增加计数，acquireShared会减少计数。   
3.CountDownLatch   
使用AQS同步状态来表示计数。计数为0时，所有的Acquire操作（CountDownLatch的await方法）才可以通过

白话：AQS是jdk提供的一个同步器，可以很方便的生成自定义同步器。AQS内部使用一个volatile的state来表示同步状态，通过一个FIFO队列来做多线程获取资源的排队操作，AQS通过CAS来做state变量的修改。实现AQS只要实现其中判断获取锁和释放锁的方法即可，AQS内部会去做队列入队出队等复杂逻辑处理。使用AQS实现的同步器有ReentrantLock，Semaphore，CountDownLatch。

### - CountDownLatch、Semaphore、CyclicBarrier含义及实现原理

CountDownLatch   
一个或多个线程等待其他线程完成一些列操作
CountDownLatch是一个同步辅助类，当CountDownLatch类中的计数器减少为0之前所有调用await方法的线程都会被阻塞，如果计数器减少为0，则所有线程被唤醒继续运行。

典型应用场景

开始执行前等待n个线程完成各自任务：例如有一个任务想要往下执行，但必须要等到其他任务执行完毕后才可以继续往下执行。假如这个想要继续往下执行的任务调用一个CountDownLatch对象的await()方法，其他的任务执行完自己的任务后调用同一个CountDownLatch对象上的countDown()方法，这个调用await()方法的任务将一直阻塞等待，直到这个CountDownLatch对象的计数值减到0为止。

CyclicBarrier   
多个线程相互等待，直到到达同一个同步点，再继续一起执行。CyclicBarrier适用于多个线程有固定的多步需要执行，线程间互相等待，当都执行完了，在一起执行下一步。

CyclicBarrier和CountDownLatch的异同   
CountDownLatch 是一次性的，CyclicBarrier 是可循环利用的
CountDownLatch 参与的线程的职责是不一样的，有的在倒计时，有的在等待倒计时结束。
CyclicBarrier 参与的线程职责是一样的。

个人理解：
1. CountDownLatch 是当前线程等着别人做好再开始做。像做饭一样，买好菜。
   CountDownLatch内部是AQS做的同步，共享模式，共享释放，只有减到0才能获得
2. Semaphore 是多个线程去获取，有的话就有，没有就等着。 像买房摇号。
   Semaphore 内部是AQS做的同步，非0就可获得，0就不行了
3. CyclicBarrier 是各个线程都达到某个预设点的时候， 可以执行一段逻辑，然后打开所有线程的限制。 像赛马.
   CyclicBarrier 底层是依赖Reentrantlock保证同步 和 一个condition 来阻塞和放开早到达的多个线程，其实也是AQS

## java异步

### - Callable、Future、FutureTask、CompletableFuture分别是什么

Callable是一个接口，提供一个回调方法，可以放到executorService中

Future接口提供了三种功能：   
1）判断任务是否完成；     
2）能够中断任务；   
3）能够获取任务执行结果。   
   
FutureTask
可以看出RunnableFuture继承了Runnable接口和Future接口，而FutureTask实现了RunnableFuture接口，那就可以得出FutureTask即可以作为一个Runnable线程执行，又可以作为Future得到Callable返回值。    
FutureTask是Future的唯一实现类

listenableFuture
ListenableFuture和JDK原生Future最大的区别是前者做到了一个可以监听结果的Future   
void addListener(Runnable listener, Executor executor);

CompletableFuture
在JDK8中开始引入的，这个在一定程度上与ListenableFuture非常类似。比如说ListenableFuture的listener监听回调，在这个类中，相当于thenRun或者whneComplete操作原语

https://blog.csdn.net/Androidlushangderen/article/details/80372711


### - java异步编程，获取线程返回结果的方法#
Thread的join()方法实现

CountDownLatch实现

ExecutorService.submit方法实现
future.get()

FutureTask
futureTask.get()

listenableFuture
ListenableFuture和JDK原生Future最大的区别是前者做到了一个可以监听结果的Future
void addListener(Runnable listener, Executor executor);

CompletableFuture
CompletableFuture.supplyAsync可以用来异步执行一个带返回值的任务，调用completableFuture.get()
会阻塞当前线程，直到任务执行完毕，get方法才会返回。

## 线程池

### - 线程池的创建使用、有哪些参数

为什么要用线程池？    
	1. 降低资源消耗。 通过重复利用已创建的线程降低线程创建和销毁造成的消耗 （重复利用）    
	2. 提高响应速度。 当任务到达时，任务可以不需要的等到线程创建就能立即执行 （提前开始任务）    
	3. 提高线程的可管理性。 线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控  （控制线程数量）    

如何创建线程池    
	1. Executors - 不允许    
		CachedThreadPool 和 ScheduledThreadPool ： 允许创建的线程数量为 Integer.MAX_VALUE ，可能会创建大量线程，从而导致OOM    
		FixedThreadPool 和 SingleThreadExecutor ： 允许请求的队列长度为 Integer.MAX_VALUE ，可能堆积大量的请求，从而导致OOM    
	2. ThreadPoolExcutor    
		参数： 核心线程数， 最大线程数，非核心线程空闲存活时间，阻塞队列，拒绝策略，有必要的话还有ThreadFactory线程工厂.    
		好处：    
      - corePoolSize, maximumPoolSize 弹性控制线程数量，可伸缩，可扩容可释放      
      - keepAliveTime, TimeUnit.SECONDS 设置后在回收前可让其他任务使用，减少重新创建线程的开销     
      - BlockingQueue, 设置队列大小，起到缓冲的作用     
      - rejectHandler  

### - 线程池的原理  

1. 先讲构造参数     
corePoolSize： 线程池核心线程数最大值   
maximumPoolSize： 线程池最大线程数大小  
keepAliveTime： 线程池中非核心线程空 闲的存活时间大小  
unit： 线程空闲存活时间单位   
workQueue： 存放任务的阻塞队列  
threadFactory： 用于设置创建线程的工厂，可以给创建的线程设置有意义的名字，可方便排查问题。  
handler：  线城池的饱和策略事件，主要有四种类型。  

2. 再描述提交任务后的执行过程     
   - 提交一个任务，线程池里存活的核心线程数小于线程数corePoolSize时，线程池会创建一个核心线程去处理提交的任务。
   - 如果线程池核心线程数已满，即线程数已经等于corePoolSize，一个新提交的任务，会被放进任务队列workQueue排队等待执行。
   - 当线程池里面存活的线程数已经等于corePoolSize了,并且任务队列workQueue也满，判断线程数是否达到maximumPoolSize，即最大线程数是否已满，如果没到达，创建一个非核心线程执行提交的任务。
   - 如果当前的线程数达到了maximumPoolSize，还有新的任务过来的话，直接采用拒绝策略处理。

3. 四种拒绝策略

AbortPolicy(抛出一个异常，默认的)   
DiscardPolicy(直接丢弃任务)   
DiscardOldestPolicy（丢弃队列里最老的任务，将当前这个任务继续提交给线程池）   
CallerRunsPolicy（交给线程池调用所在的线程进行处理)  

https://mp.weixin.qq.com/s?__biz=MzA5MTkxMDQ4MQ==&mid=2648933151&idx=1&sn=2020066b974b5f4c0823abd419e8adae&chksm=88621b21bf159237bdacfb47bd1a344f7123aabc25e3607e78d936dd554412edce5dd825003d&token=995072421&lang=zh_CN#rd

如何自定义拒绝策略：实现RejectedExecutionHandler接口，实现rejectedExecution方法
public interface RejectedExecutionHandler {
    void rejectedExecution(Runnable r, 
    ThreadPoolExecutor executor);
}

### - 怎么配置参数

线程数：  

如果是CPU密集型应用，则线程池大小设置为N+1

如果是IO密集型应用，则线程池大小设置为2N+1

系统负载： 一个进程或线程正在被cpu执行或等待被cpu执行，则系统负载+1， 单核cpu负载小于1表示cpu可以在线程不等待的情况下处理完

IO密集：通常指网络IO


### - 阻塞队列原理

如果队列为空条件满足时，消费者一直等待，如果队列满条件满足时，生产者会一直等待，当条件不满足的时候，通过通知机制实现生产者和消费者间的通信。当消费者消费了一个队列中的元素后，会通知生产者当前队列可用。

JDK通过condition实现的通知机制，condition底层通过unsafe类的park、unpark方法实现的线程的阻塞和解除阻塞

### - 有哪些阻塞队列

ArrayBlockingQueue
ArrayBlockingQueue（有界队列）是一个用数组实现的有界阻塞队列，按FIFO排序量。

LinkedBlockingQueue
LinkedBlockingQueue（可设置容量队列）基于链表结构的阻塞队列，按FIFO排序任务，容量可以选择进行设置，不设置的话，将是一个无边界的阻塞队列，最大长度为Integer.MAX_VALUE，吞吐量通常要高于ArrayBlockingQuene；newFixedThreadPool线程池使用了这个队列

DelayQueue
DelayQueue（延迟队列）是一个任务定时周期的延迟执行的队列。根据指定的执行时间从小到大排序，否则根据插入到队列的先后排序。newScheduledThreadPool线程池使用了这个队列。

PriorityBlockingQueue
PriorityBlockingQueue（优先级队列）是具有优先级的无界阻塞队列；

SynchronousQueue
SynchronousQueue（同步队列）一个不存储元素的阻塞队列，每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQuene，newCachedThreadPool线程池使用了这个队列。
