---
title: 知识储备
date: 2019-09-09 11:10:43
---

<!-- toc -->

### - 谈谈对springIoc的了解

   [如何完美阐述你对IOC的理解？](https://www.zhihu.com/question/313785621 "知乎")
 
   // 最底层map: Map<String, Object> singletonObjects = new ConcurrentHashMap(256);  

   个人理解：    
   1.控制反转是一种设计思想，将对象生命周期和依赖的控制由对象转给了ioc容器，对象由主动创建其它对象变为被动由ioc容器注入  
   2.springioc容器控制了对象的创建、销毁及整个生命周期，，可以动态注入依赖的对象  
   3.将对象与对象间解耦，对象不再直接创建依赖其它对象，都依赖springioc容器  

   扩展：[springIoc的初始化过程](../2019/09/10/spring/springioc初始化 "springIoc的初始化过程")

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

### - 常见Java集合类

**非线程安全**  

ArrayList:  底层数组  
LinkedList:  底层双向链表  
HashMap: (*重要必考，后面单独问*)   
数组加链表，链表长度 > 8转化为红黑树  

**线程安全**   

concurrentHashmap:   线程安全的HashMap, Node数组+链表 + 红黑树  
copyonwriteArrayList:   线程安全的List，在读多写少的场合性能非常好    
concurrentLinkedQueue:   高效的并发队列，使用链表实现。
可以看做一个线程安全的 LinkedList   
BlockingQueue(接口)   
1 ArrayBlockingQueue:  有界队列实现类 底层Object数组    
2 LinkedBlockingQueue: 单向链表  
3 PriorityBlockingQueue: 支持优先级的无界阻塞队列  

### - ArrayList 和 LinkedList的区别
1. 数据结构  数组 / 双向链表 
2. 是否支持快速随机访问
3. 随机插入删除速度, ArrayList需要整体位移

### - Java并发容器有哪些

concurrentHashmap:   线程安全的HashMap, Node数组+链表 + 红黑树  
copyonwriteArrayList:   线程安全的List，在读多写少的场合性能非常好    
concurrentLinkedQueue:   高效的并发队列，使用链表实现。
可以看做一个线程安全的 LinkedList   
BlockingQueue(接口)   
1 ArrayBlockingQueue:  有界队列实现类 底层Object数组    
2 LinkedBlockingQueue: 单向链表  
3 PriorityBlockingQueue: 支持优先级的无界阻塞队列  
 

### - ArrayBlockingQueue和LinkedBlockingQueue的实现原理

#### ArrayBlockingQueue是object数组实现的线程安全的有界阻塞队列 
1.数组实现：使用数组实现循环队列    
2.有界：内部为数组实现，一旦创建完成，数组的长度不能再改变    
3.线程安全：使用了 ReentrantLock 和 两个 condition(notEmpty, notFull) 来保证线程安全   
4.阻塞队列：先进先出。当取出元素的时候，若队列为空，wait直到队列非空（notEmpty）；当存储元素的时候，若队列满，wait直到队列有空闲（notFull）

#### LinkedBlockingQueue是一个单向链表实现的阻塞队列
1.链表实现： head是链表的表头， last是链表的表尾。取出数据时，都是从表头head处取出，出队； 新增数据时，都是从表尾last处插入，入队。   
2.可有界可无界：可以在创建时指定容量大小，防止队列过度膨胀。如果未指定队列容量，默认容量大小为Integer.MAX_VALUE  
3.线程安全：使用了 两个 ReentrantLock 和 两个 condition，putLock是插入锁，takeLock是取出锁；notEmpty是“非空条件”，notFull是“未满条件”。通过它们对链表进行并发控制   
4.阻塞队列：先进先出。当取出元素的时候，若队列为空，wait直到队列非空（notEmpty）；当存储元素的时候，若队列满，wait直到队列有空闲（notFull）

注：  
PriorityBlockingQueue 是一个支持优先级的无界阻塞队列，内部结构是数组实现的二叉堆，不深入；     ArrayBlockingQueue 为什么没有使用两个锁，没有找到满意的答案，有的说是入队出队时间短不必要引入额外的设计复杂度（操作count、同index位数据）, 参考https://www.zhihu.com/question/41941103 林哲民的回答

### - 死锁产生的条件、如何避免

什么是死锁：一组互相竞争资源的线程因互相等待，导致“永久”阻塞的现象
 
死锁四大条件：  
互斥条件：简单的说就是进程抢夺的资源必须是临界资源，一段时间内，该资源只能同时被一个进程所占有  

请求和保持条件：当一个进程持有了一个（或者更多）资源，申请另外的资源的时候发现申请的资源被其他进程所持有，当前进程阻塞，但不会是放自己所持有的资源    

不可抢占条件：进程已经获得的资源在未使用完毕的情况下不可被其他进程所抢占   

循环等待条件：发生死锁的时候，必然存在一个进程—资源的循环链 

预防死锁 或叫避免死锁   
破坏请求和保持条件：   
所有进程在开始运行之前，必须一次性获得所有资源，如果无法获得完全，释放已经获得的资源，等待；　　

破坏不可抢占条件：  
说起来简单，只要当一个进程申请一个资源，然而却申请不到的时候，必须释放已经申请到的所有资源。让别人去抢占。

破坏循环等待条件：   
设立一个规则，让进程获取资源的时候按照一定的顺序依次申请，不能违背这个顺序的规则。必须按照顺序申请和释放，想要申请后面的资源必须先把该资源之前的资源全部申请，想要申请前面的资源必须先把该资源之后的资源（前提是已获得）全部释放

破坏互斥条件：  
没法破坏，是资源本身的性质所引起的


### - 项目中用到过哪些数据结构

queue, map, set, list, 堆（priorityQueue）

### - BIO、NIO、AIO 的区別和联系
https://mp.weixin.qq.com/s/QrE-PLCppwzfX3JtvWx5QA

BIO：同步、阻塞。服务器实现模式为一个连接一个线程,服务器端为每一个客户端的连接请求都需要启动一个线程进行处理

NIO：多路复用技术，同步非阻塞。服务器实现模式为客户端的连接请求都会注册到多路复用器上,用同一个线程接收所有连接请求

### - 谈谈对Java NIO的了解



### - 遇到过的设计模式（项目、框架） 



### - 谈谈对hashMap的了解


### - 谈谈对ConcurrentHashMap的了解


### - 倒排索引原理 
### - 索引优缺点
### - 什么情况下索引失效
### - InnoDB 支持的索引类型
### - B+树和 B 树的区别，和红黑树的区别
### - 如何优化sql
### - 什么是覆盖索引
### - 为什么尽量不要用select *？

### - 乐观锁 、悲观锁 区别？
### - mysql的乐观锁、悲观锁实现
### - mysql数据库事务隔离级别，分別解決什么问题，next-key锁原理、如何解决幻读？

### - 谈谈对Java内存模型的了解
### - java的乐观锁CAS锁原理
### - volatile原理
### - sychronized原理
### - AQS同步器原理？clh队列？tryAcquire的过程？
### - AQS涉及类及原理
### - sychronized reentranlock 的区别
### - ThreadLocal原理

### - http1.X 和 http2.0 的区別
HTTP/2 的多路复用(Multiplexing) 则允许同时通过单一的 HTTP/2 连接发起多重的请求-响应消息
### - Http https 的区別
### - https过程

### - tcp/ip模型分层及各层协议举例

### - tcp如何保证可靠传输，tcp、udp区别
### - 为什么要三次握手，四次挥手
### - 四次挥手的closewait, timewait分别在哪，为什么timewait要等待2msl

### - 常见的负載均衡算法有哪些
### - 多个 RPC 请求进来，服务器怎么处理并发呢

### - 讲一下 Redis 的哨兵机制

### - redis支持的数据结构
### - redis的过期删除策略
### - redis的内存淘汰机制
### - 如果redis有热点key怎么解决
### - redis缓存击穿、缓存穿透、缓存雪崩怎么解决


### - 谈谈数据库分库分表

### - Java内存区域
### - 谈谈java的类加载机制
### - gc垃圾收集器及其优缺点
### - 标记算法和复制算法的区別，用在什么场合  
CMS 标记清除、Serial Old，Parallel old 标记整理：适用于存活对象比较多的场景
Serial、ParNew、PS 收集器 复制算法：用在那种不可达对象比较多的场合  
### - cms垃圾收集器的回收步骤及其优缺点？  
### - GC、G1 和 ZGC 的区別
### - 内存泄漏与内存溢出的区别

### - 有几种gc fail？  

### - MESI和内存屏障  


### - 策略模式和模板方法模式的区别  

### - 最有成就感的项⽬  

### - 有哪些处理线上问题的经验  
### - 线上需要关注哪些机器参数  

### - 请求幂等性
### - 分布式事务
### - 秒杀系统的设计
### - 项目中消息队列怎么用的？使用哪些具体业务场景

### - 线程池的创建使用、有哪些参数

### - 如何写自我介绍