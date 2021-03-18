---
title: 知识储备-雯
date: 2020-03-10 23:47:33
---

<!-- toc -->

## Java集合

### - ArrayList 和 LinkedList的区别
1. 数据结构  数组 / 双向链表 
2. 是否支持快速随机访问，arraylist支持，linkedlist不支持
3. 随机插入删除速度, ArrayList慢因为需要整体位移，linkedlist快

### - 谈谈对hashMap的了解

基本：底层数据结构, 数组 + 链表，长度超过8变为红黑树，非线程安全
加分：HashMap在put时, 经过了两次hash，一个是JDK自带的对对象key的hash，一个是内部扰动函数hash，做高16位与低16位异或，加大散列的效果
加分：HashMap 的长度为什么是2的幂次方。1. 散列值用之前要对数组长度取模，求桶位(n - 1) & hash 更高效；2. 扩容的时候也可以用 hash值与扩容后的最高位 & 判断节点是在高位还是低位，更高效。
加分：HashMap 非线程安全，多线程操作导致死循环问题，1.7  并发下头插形成循环链表，1.8 用尾插修复， 但仍可能造成丢失数据。

白话：hashmap是非线程安全的集合。底层是数组+链表，链表长度超过8会转成红黑树。hashmap中table的长度总是2的幂次方，这样可以使用&运算代替取余运算(%)来提高效率，在求桶位和扩容时获取节点新位置在左边还是右边的时候。

### - loadFactor和threshold的关系?   

size: 当前共有多少个KV对
capacity: 当前HashMap的容量是多少, 桶位数, table大小, 默认是16
loadFactor: 负载因子，默认0.75
threshold: loadFactor* capacity，

白话：负载因子就是控制碰撞几率的，调的小，碰撞的概率小，调的大，碰的概率就大。阈值就是扩容时机。阈值等于负载因子乘以table大小

### - 谈谈对ConcurrentHashMap的了解

1.7   结构： 分段的数组+链表; 
      并发：分段锁，锁一段数组 segment，默认有16个segment, 并发度只有默认是16
      线程安全: segment上锁，继承ReentrantLock可重入锁

1.8   结构： Node数组+链表 + 红黑树; 
      并发： 并发控制使用 synchronized 和 CAS
      线程安全: CAS和synchronized来保证并发安全, synchronized只锁定当前链表或红黑二叉树的首节点, 只要hash不冲突，就不会产生并发，效率又提升N倍

白话：线程安全的map。1.7版本是使用分段锁实现的线程安全，segment上加锁，在构建的时候可以设置大小，默认是16，就是并发度默认只有16. 1.8版本的线程安全是用synchronized+cas实现的，使用synchronized锁住链表或红黑树的头结点，使用cas来进行size++和首个节点入桶。1.8的并发度是随着桶位增加而增加的，所以并发效率会随扩容提升很多倍。

### - ArrayBlockingQueue和LinkedBlockingQueue的实现原理

ArrayBlockingQueue是object数组实现的线程安全的有界阻塞队列    
1.数组实现：使用数组实现循环队列    
2.有界：内部为数组实现，一旦创建完成，数组的长度不能再改变    
3.线程安全：使用了 ReentrantLock 和 两个 condition(notEmpty, notFull) 来保证线程安全   
4.阻塞队列：先进先出。当取出元素的时候，若队列为空，wait直到队列非空（notEmpty）；当存储元素的时候，若队列满，wait直到队列有空闲（notFull）


LinkedBlockingQueue是一个单向链表实现的阻塞队列   
1.链表实现： head是链表的表头，last是链表的表尾。取出数据时，都是从表头head处取出，出队； 新增数据时，都是从表尾last处插入，入队。   
2.可有界可无界：可以在创建时指定容量大小，防止队列过度膨胀。如果未指定队列容量，默认容量大小为Integer.MAX_VALUE  
3.线程安全：使用了 两个 ReentrantLock 和 两个 condition，putLock是插入锁，takeLock是取出锁；notEmpty是“非空条件”，notFull是“未满条件”。通过它们对链表进行并发控制   
4.阻塞队列：先进先出。当取出元素的时候，若队列为空，wait直到队列非空（notEmpty）；当存储元素的时候，若队列满，wait直到队列有空闲（notFull）

白话: ArrayBlockingQueue和LinkedBlockingQueue都是线程安全的阻塞队列。 ArrayBlockingQueue底层是数组实现，是有界的队列，线程安全是使用一个 ReentrantLock 和 两个 condition实现的。 LinkedBlockingQueue底层是单向链表实现，是可有界可无界的队列，线程安全是使用两个 ReentrantLock 和 两个 condition实现的。

## Java并发类

### - volatile原理

1. Volatile的特征   
A、可见性
B、禁止指令重排
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
unsafe：操作内存内存空间的类     
```
public final int getAndAddInt(Object var1, long var2, int var4) {
   int var5;
   do {
      var5 = this.getIntVolatile(var1, var2);
   } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

   return var5;
}
```

valueOffset：value在内存中的偏移量（把存储单元的实际地址与其所在段的段地址之间的距离称为段内偏移），这里我们可以简单认为是内存地址，在类加载的时候，通过unsafe获取到value的偏移量。  
value： 顾名思义代表存储值，被volatile修饰，保证在线程间的可见性   

CAS修改的值为什么要是volatile? 可见性
   
白话：CAS是Compare and Swap，比较并交换，是一种乐观锁实现线程安全的方式，更加轻量。底层是通过cpu的原子指令实现的比较并替换。使用的时候参数有内存位置，预期值，写入的新值，当通过内存位置拿到的值和预期值相等时，就用新值进行替换，整个操作是原子的，cpu指令保证。

### - sychronized 和 valotile 的区别

1. volatile更轻量，性能更好，但volatile只能用于变量而synchronized关键字可以修饰方法以及代码块  
2. 多线程访问volatile关键字不会发生阻塞，而synchronized关键字可能会发生阻塞  
3. volatile关键字能保证数据的可见性，但不能保证数据的原子性 （eg： i++）.synchronized关键字两者都能保证
4. volatile关键字主要用于解决变量在多个线程之间的可见性，而 synchronized关键字解决的是多个线程之间访问资源的同步性

白话: 1. 3.即可

### - sychronized reentranlock 的区别

1. synchronized 依赖于 JVM实现线程安全 而 ReentrantLock 依赖于 API，底层依赖AQS
2. 等待可中断  lock.lockInterruptibly()， 线程可以选择放弃等待，改为处理其他事情
3. 可实现公平锁
4. 可实现选择性通知（锁可以绑定多个条件）

### - ThreadLocal原理

每个Thread 维护一个 ThreadLocalMap 映射表，这个映射表的 key是 ThreadLocal 实例本身，value 是真正需要存储的 Object。也就是说 ThreadLocal 本身并不存储值，它只是作为一个 key 来让线程从 ThreadLocalMap 获取 value。

由于每一条线程均含有各自私有的ThreadLocalMap容器，这些容器相互独立互不影响，因此不会存在线程安全性问题，从而也无需使用同步机制来保证多条线程访问容器的互斥性。

白话：用多线程多份数据，来避免线程不安全


### - AQS同步器原理？

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

### - CountDownLatch、CyclicBarrier区别
CountDownLatch   
一个或多个线程等待其他线程完成一些列操作
CountDownLatch是一个同步辅助类，当CountDownLatch类中的计数器减少为0之前所有调用await方法的线程都会被阻塞，如果计数器减少为0，则所有线程被唤醒继续运行。

典型应用场景

开始执行前等待n个线程完成各自任务：例如有一个任务想要往下执行，但必须要等到其他任务执行完毕后才可以继续往下执行。假如这个想要继续往下执行的任务调用一个CountDownLatch对象的await()方法，其他的任务执行完自己的任务后调用同一个CountDownLatch对象上的countDown()方法，这个调用await()方法的任务将一直阻塞等待，直到这个CountDownLatch对象的计数值减到0为止。

CyclicBarrier   
多个线程相互等待，直到到达同一个同步点，再继续一起执行。CyclicBarrier适用于多个线程有固定的多步需要执行，线程间互相等待，当都执行完了，在一起执行下一步。

CyclicBarrier是全部线程一起去执行下一步，CountDownLatch是只有当时等待的线程去执行下一步.

CyclicBarrier和CountDownLatch的异同

1.CyclicBarrier和CountDownLatch都可以通过一个条件去控制多个线程，然后在条件满足时做一些操作。CyclicBarrier在条件满足时可以让所有线程都继续运行，而CountDownLatch只能让一开始就在等待的线程去运行。

2.CyclicBarrier可以复用，开启屏障之后count会回归到初始值，可以进行下一次屏障拦截线程。而CountDownLatch只能使用一次，倒计时完毕后count的值不会变化。

白话：同CyclicBarrier和CountDownLatch的异同

## spring

### - 谈谈对springIoc的了解

   [如何完美阐述你对IOC的理解？](https://www.zhihu.com/question/313785621 "知乎")
 
   // 最底层map: Map<String, Object> singletonObjects = new ConcurrentHashMap(256);  

   个人理解：    
   1.控制反转是一种设计思想，将对象生命周期和依赖的控制由对象转给了ioc容器，对象由主动创建其它对象变为被动由ioc容器注入  
   2.ioc容器控制了对象的创建、销毁及整个生命周期 
   3.将对象与对象间解耦，对象不再直接创建依赖其它对象，都依赖springioc容器  

   扩展：[springIoc的初始化过程](../2019/09/10/spring/springioc初始化 "springIoc的初始化过程")

它负责业务对象的构建管理和业务对象间的依赖绑定。

白话：同个人理解

### - 谈谈对springAop的了解  

   AOP为面向切面编程，底层是通过动态代理实现，实质上就是将相同逻辑的重复代码横向抽取出来, 拦截对象方法，对方法进行改造、增强！比如在 **方法执行前**、**方法返回后**、**方法前后**、**方法抛出异常后** 等地方进行一定的增强处理，应用场景： 事务、日志、权限、监控打点

   基于动态代理来实现，在容器启动的时候生成代理实例。默认地，如果使用接口的，用 JDK 提供的动态代理实现，如果没有接口，使用 CGLIB 实现

   优点：每个关注点现在都集中于一处，而不是分散到多处代码中；服务模块更简洁，服务模块只需关注核心代码  

   @Pointcut(切点)：用于定义哪些方法需要被增强或者说需要被拦截  
   @Before、@After、@Around、@AfterReturning、@AfterThrowing（增强）: 添加到切点指定位置的一段逻辑代码

   注：@AfterReturning 比 @After 的方法参数多被代理方法的返回值

   参考：  
   [Spring AOP 使用介绍，从前世到今生](https://javadoop.com/post/spring-aop-intro "知乎")  
   [Spring AOP就是这么简单啦](https://juejin.im/post/5b06bf2df265da0de2574ee1 "掘金") 

白话：AOP为面向切面编程，底层是通过动态代理实现，实质上就是将相同逻辑的重复代码横向抽取出来, 拦截对象方法，对方法进行改造、增强. 应用场景比如日志、权限、监控打点。

## jvm

### - Java内存区域

#### 线程私有

程序计数器
存放当前线程所执行的字节码的行号指示器。如果线程执行的是一个Java程序，计数器记录正在执行的虚拟机字节码指令地址；正在执行的是native方法，则计数器的值为空

Java虚拟机栈

描述Java方法执行的内存模型，每个方法执行的同时都会创建一个栈帧来存储局部变量表（编译期可知的各种基本数据类型，对象引用【reference类型】，内存空间的分配是在编译期完成的，方法运行期间不会改变局部变量表的大小）、操作数栈、动态链接、方法出口等。

本地方法栈

为虚拟机使用到的native方法服务

#### 线程公有

Java堆（GC堆）

存放对象实例，所有的对象实例和数组都在堆上分配。程序运行时分支可能不一样，只有运行期间才能知道创建哪些对象，这部分内存的分配和回收是动态的

metaspace

存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码

白话：只说名称即可

### - 谈谈java的类加载机制

1. 过程   
	- 加载   
		通过全类名获取定义此类的二进制字节流   
		将字节流所代表的静态存储结构转换为metaspace的运行时数据结构   
		在内存中生成一个代表该类的 Class 对象,作为metaspace这些数据的访问入口   
	- 连接   
		验证   
      文件格式、元数据、字节码、符号引用验证   
	- 准备   
		正式为类变量分配内存并设置类变量初始值的阶段，设置数据类型默认的零值   
	- 解析   
		解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程   
	- 初始化   
		真正执行类中定义的 Java 程序代码   
      // 初始化阶段是执行类构造器 <clinit> ()方法的过程   
2. 双亲委派模式   
   Java的类加载使用双亲委派模式，即一个类加载器在加载类时，先把这个请求委托给自己的父类加载器去执行，如果父类加载器还存在父类加载器，就继续向上委托，直到顶层的启动类加载器。如果父类加载器能够完成类加载，就成功返回，如果父类加载器无法完成加载，那么子加载器才会尝试自己去加载。这种双亲委派模式的好处，一个可以避免类的重复加载，另外也避免了java的核心API被篡改。

   BootstrapClassLoader  
	ExtensionClassLoader  
	AppClassLoader(应用程序类加载器)   

白话：ClassLoader名不用记，别的要记

### - gc对象存活判定算法

1.1引用计数算法

给对象添加一个引用计数器，当有地方引用它时加1，当引用失效时计数减1，任何时刻计数器为0时的对象都不会再被引用。

缺点：存在两个对象不会被继续访问，但是两者循环调用的情况，由于存在循环调用导致两个对象都不会被回收。

1.2可达性分析算法

通过一系列被称作gc roots对象作为起点，从这些起点向下搜索，搜索所走过的路径成为引用链，当一个对象到gc roots没有任何引用链时证明此对象不可用。

Java中gc roots的对象包括：

虚拟机栈中引用的对象  
native方法中引用的对象   
类静态属性、常量中引用的对象 

白话：答：引用计数算法、可达性分析算法 名就行了

### - gc垃圾收集器及其优缺点

垃圾回收器种类:Serial、ParNew、Parallel Scavenge、Serial Old、Parallel Old、CMS(ConCurrent Mark Sweep)、G1(Garbage First)

我们通常使用parnew(多线程stw复制算法，新生代) + cms(标记清除，老年代）

### - 标记算法和复制算法的区別，用在什么场合  
CMS 标记清除;   
Serial Old，Parallel old 标记整理：适用于存活对象比较多的场景;    
Serial、ParNew、PS 收集器 复制算法：用在那种不可达对象比较多的场合;

### - cms垃圾收集器的回收步骤及其优缺点？  

cms回收器的标记过程:

CMS以获取最短回收停顿时间为目标的收集器,使用“标记-清除”算法,分为以下6个步骤

1.STW initial mark:**初始标记**,第一次暂停,从root标记old space存活对象(the set of objects reachable from roots (application code))

2.Concurrent marking:**并发标记**,从上一步得到的集合出发,遍历old space,标记存活对象 (all live objects that are transitively reachable from previous set)

3.Concurrent precleaning:**并发预清理**。并发的标记前一阶段被修改的对象(card table)

4.STW remark:**重新标记**。第二次暂停,检查,标记,检查脏页的对象,标记前一阶段被修改的对象 (revisiting any objects that were modified during the concurrent marking phase)

5.Concurrent sweeping:**并发清理**。运行过程中清理,扫描old space,释放不可到达对象占用的空间

6.Concurrent reset:**并发重置**。此次CMS结束后,重设CMS状态等待下次CMS的触发 

白话：步骤名称，细问再说

## mysql

### - 什么情况下索引失效

1. 不符合联合最左匹配
2. like前导模糊查询 （可以通过 REVERSE()函数来创建一个函数索引解决）
3. 字段类型不匹配
4. 索引字段施加函数
5. or的多个字段都要索引（只要有一个没有索引，就无法使用索引）
6. != , <> is null, is not null都会使索引失效

### - InnoDB 支持的索引类型

1. B+树索引：  
2. 全文索引：就是倒排索引    
仅能再char、varchar、text类型的列上面创建全文索引
但是面对高级的搜索还是略显简陋，且性能问题也是担忧。  
3. 哈希索引：
   哈希索引在InnoDB中只是一种系统自动优化的功能   
   Hash 索引仅仅能满足"=","IN"和"<>"查询，不能使用范围查询。   
   Hash 索引无法被用来避免数据的排序操作。     
   在MySQL运行的过程中，如果InnoDB发现，有很多SQL存在这类很长的寻路，并且有很多SQL会命中相同的页面(page)，InnoDB会在自己的内存缓冲区(Buffer)里，开辟一块区域，建立自适应哈希所有AHI，以加速查询。InnoDB的自使用哈希索引，更像“索引的索引”  

### - B+树和 B 树的区别，和红黑树的区别

为什么不用B树?:
因为B树的所有节点都是包含键和值的,这就导致了每个几点可以存储的内容就变少了,出度就少了,树的高度会增高,查询的 时候磁盘I/O会增多,影响性能。由于B+Tree内节点去掉了data域,因此可以拥有更大的出度,拥有更好的性能。

和红黑树:   
B+树跟红黑树不用比，B+树的高很低，红黑树比不了。

### - 平时怎么创建索引的

1. 选择区分度高的列做索引

2. 根据查询场景、查询语句的查询条件设计索引

3. 如果要为多列去创建索引，遵循最左匹配原则使用联合索引去创建

4. 在长字符类型的字段上使用前缀索引，减少空间占用

5. 扩展索引的时候尽量做追加，不是新建

### - 如何优化sql  

1. 让sql使用索引，如果查看sql使用索引情况，explain查看执行计划       
通过explain命令可以得到下面这些信息: 表的读取顺序，数据读取操作的操作类型哪些索引可以使用哪些索引被实际使用，表之间的引用，每张表有多少行被优化器查询等信息。 rows是核心指标，绝大部分rows小的语句执行一定很快。  
Extra字段几种需要优化的情况    
Using filesort 需要优化，MYSQL需要进行额外的步骤来对返回的行排序。   
Using temporary需要优化，发生这种情况一般都是需要进行优化的。mysql需要创建一张临时表用来处理此类查询  
Using where 表示 MySQL 服务器从存储引擎收到行后再进行“后过滤”

白话: 让sql使用索引, 通过explain查看sql执行计划，看是否正常使用索引，关注下possible keys, keys 是否使用到索引,rows扫描行数。还有Extra字段，如果有 Using filesort、  Using temporary、Using where都说明需要优化了。Using filesort说明服务器使用额外步骤做排序操作了，Using where也表示服务器做额外的过滤了，Using temporary表示新建了一张临时表来做查询。都需要优化。

### - 分库分表

根据userid取模做的分库分表

两个场景，业务解耦垂直拆分，读写性能瓶颈做水平拆分。

1. 垂直切分

业务维度切分，解耦

2. 水平切分

主库写入并发瓶颈，分库  
表数据量过大，影响查询性能和维护索引性能，分表


### - mysql数据库事务隔离级别，分別解決什么问题，next-key锁原理、如何解决幻读？

READ-UNCOMMITTED(未提交读): 可能会导致脏读、幻读或不可重复读  
READ-COMMITTED(提交读): 可以阻止脏读，但是幻读或不可重复读仍有可能发生  
REPEATABLE-READ(可重复读，mysql默认隔离级别): 可以阻止脏读和不可重复读，幻读通过mvcc解决了快照读，next-key锁解决了当前读
SERIALIZABLE(串行化读): 该级别可以防止脏读、不可重复读以及幻读

### - RR RC区别

区别就在于rr解决了不可重复读和幻读，怎么解决的，通过MVCC和next-key锁.    

不可重复读：rc级别下的mvcc总是读取数据行的最新快照，而rr级别下的mvcc，会在事务第一次select的时候，为数据行生成一个快照，后面每次都读这个快照，除非自己更新，所以rr下是可重复读，别的事务提交也无法影响你的事务。   

幻读：快照读下rr级别不会出现幻读，因为rr级别的mvcc读的是事务第一次读取时的快照；在当前读下rr级别使用了next-key锁（临键锁），临键锁包括行锁+间隙锁， 来避免两个当前读时有其它事务插入数据，所以当前读使用next-key锁解决的幻读。 最后备注下：如果是先快照读再当前读，影响行数不一致是否属于幻读，是有争议的但大多认为并不是幻读。   

## redis

### - redis支持的数据结构

String

最常规的get/set操作，value可以是数字和string
	一般做一些复杂的计数功能的缓存
hash

value存放的是结构化的对象
比较方便的就是操作其中的某个字段
在做单点登录的时候，就是用这种数据结构存储用户信息，以cookieId作为key，设置30分钟为缓存过期时间，能很好的模拟出类似session的效果

list

使用List的数据结构，可以做简单的消息队列的功能
还有一个是，利用lrange命令，做基于redis的分页功能，性能极佳，用户体验好
一个场景，很合适---取行情信息。就也是个生产者和消费者的场景。LIST可以很好的完成排队，先进先出的原则。

set

set堆放的是一堆不重复值的集合。所以可以做全局去重的功能  
为什么不用JVM自带的Set进行去重？因为我们的系统一般都是集群部署，使用JVM自带的Set，比较麻烦，难道为了一个做一个全局去重，再起一个公共服务，太麻烦了   
另外，就是利用交集、并集、差集等操作，可以计算共同喜好，全部的喜好，自己独有的喜好等功能

sorted set  

sorted set多了一个权重参数score,集合中的元素能够按score进行排列。可以做排行榜应用，取TOP N操作。

### - redis缓存击穿、缓存穿透、缓存雪崩怎么解决

**缓存穿透**

定义: 访问一个**不存在的key**，缓存不起作用，请求会穿透到DB，流量大时DB会挂掉。

解决方案：

1. 缓存空值 （值少时）
2. 布隆过滤器， 特性: 没有的肯定没有，有的不一定有

**缓存击穿**

定义：**并发性,一个存在的key，在缓存过期的一刻**，同时有**大量的并发请求**，这些请求都会击穿到DB，造成瞬时DB请求量大、压力骤增。

解决方案：

1. 分布式锁，在访问key之前，采用分布式锁SETNX（set if not exists）来设置另一个短期key来锁住当前key的访问，访问结束再删除该短期key
2. 双重缓存，设置不同过期，其实同时解决了key过热 和 缓存击穿问题，如果更新的操作确实很耗时，返回的有损请求比较多，那确实需要双重缓存了，再放一份到本地、分布式缓存的另一个key

**缓存雪崩**

定义：**大量的key**设置了相同的过期时间，或者某台服务器宕机，导致大量缓存在**同一时刻全部失效**，造成瞬时DB请求量大、压力骤增，引起雪崩。

解决方案：

1. 提前预防，主从加集群，主从可以一个实例挂了，另一个可以顶上。集群数据分片，即使一个分片上的主从都挂了，打到db的量也不会是全部
2. 将key的过期时间设置时添加一个随机时间，分散过期时间
3. 如果是热点key，可以加分布式锁，减少并发量
4. 二级缓存（本地缓存），减少db压力

击穿是单个，必定是热点key，雪崩是很多，不一定是热点key，对应的解决方案也有不一样的地方，雪崩有一个设置过期时间加随机数，雪崩用二级缓存比较好使，但击穿就没太大必要

## mq

### - mq使用场景

* 解耦
    * 多个下游相同依赖
    * 数据驱动的任务依赖（下游任务强依赖上游数据到达）
* 异步
    * 上游不关心执行结果
    * 上游关注执行结果，但执行时间很长（改为异步操作，再尝试获取结果）
* 削峰
    * 请求高峰期

### - kafka如何保证消息顺序性

1. 一个partition内保证顺序性
2. 可以通过message key来定义，同一个key的message可以保证只发送到同一个partition

### - kafka如何保证消息可靠性

1. 多副本写入
2. 数据持久化，通过后台线程将消息持久化到磁盘
3. 生产者确认，分片所有副本写入成功才返回。发送失败重试
4. 消费组回复消费成功ack，确保消费一次

## 网络

### - tcp/ip模型分层及各层协议举例

分层   
应用层   
   任务是通过应用进程间的交互来完成特定网络应用, dns, http, smtp  
传输层  
   负责向两台主机进程之间的通信提供通用的数据传输服务, tcp, udp  
网络层  
   选择合适的网间路由和交换结点， 确保数据及时传送, ip，ICMP  
数据链路层  
   两台主机之间的数据传输，这就需要使用专门的链路层的协议， ARP   

### - tcp如何保证可靠传输，tcp、udp区别

tcp与udp区别
1. 是否建立连接， udp不建立连接，tcp三次握手   
2. 是否可靠，udp不需要确认， tcp会有确认、重传、窗口、拥塞等机制   
3. 应用场景， udp一般用于即时通信，qq，直播； tcp用于文件传输，收发邮件、登录等  

### - 为什么要三次握手，四次挥手

为什么要三次握手  
	确认双发收发功能都正常   

为什么要四次挥手   
	确认双方都没有数据再发送

### - 四次挥手的closewait, timewait分别在哪，为什么timewait要等待2msl

2MSL是两倍的MSL(Maximum Segment Lifetime)，MSL指一个片段在网络中最大的存活时间，2MSL就是一个发送和一个回复所需的最大时间

如果直到2MSL，Client都没有再次收到FIN，那么Client推断ACK已经被成功接收，则结束TCP连接

TIME_WAIT状态就是用来重发可能丢失的ACK报文。在Client发送出最后的ACK回复，但该ACK可能丢失。Server如果没有收到ACK，将不断重复发送FIN片段。所以Client不能立即关闭，它必须确认Server接收到了该ACK


## 其他

### - 主从延迟怎么办

主从延迟的原因：   
主库写入数据并且生成binlog文件，从库异步读取更新，程序读从库但还没更新就去读取了   

解决方案：    
一、更新操作方面，做SQL优化，避免大批量更新   
二、查询场景，   
   1. 强制读主，对主库压力大，谨慎使用   
   2. 延迟读从，更新时写一个过期时间为主从延迟时间的缓存key，读的时候添加一个判断   
   3. 识别重试机制，通过消息发送的版本号与数据库数据版本对比识别最新数据，如果不一致，通过等待重试来保证   

白话：解决方案其实说二的1、2就够了，更新和识别重试比较少见

### - 谈谈对Java NIO的了解

NIO（Non-blocking I/O，在Java领域，也称为New I/O），是一种同步非阻塞的I/O模型，也是I/O多路复用的基础，已经成为解决高并发与大量连接、I/O处理问题的有效方式。   
传统的BIO模型严重依赖于线程。但线程是很”贵”的资源。创建和销毁成本很高，本身占用较大内存，切换成本是很高。    
NIO由原来占用线程的阻塞读写变成了单线程轮询事件，找到可以进行读写的网络描述符进行读写。大大地节约了线程，为处理海量连接提供了可能     

总结NIO带来了什么：    
- 事件驱动模型  
- 避免多线程
- 单线程处理多任务
- 非阻塞I/O，I/O读写不再阻塞，而是返回0


### - 谈谈对Java内存模型的了解

jmm是为了解决线程间通信问题，线程间通信通常有两种解决方法，共享内存或通知机制， jmm使用了共享内存的方式，jmm定义了一套happens-before规则来规范多线程下的执行顺序和多线程下变量的可见性问题，happens-before规则底层是通过禁止部分编译器和处理器的指令重排序实现的。

### - 如何创建线程池    
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