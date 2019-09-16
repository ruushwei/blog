---
title: 面试
date: 2019-09-09 11:10:43
---

<!-- toc -->

### - 谈谈对springIoc的了解

   [如何完美的向面试官阐述你对IOC的理解？](https://www.zhihu.com/question/313785621 "知乎")
 
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



参考  
[解读 java 并发队列 BlockingQueue](https://javadoop.com/post/java-concurrent-queue "javadoop") 

### - 死锁产生的条件、如何避免

### - 项目中用到过哪些数据结构

### - BIO、NIO、AIO 的区別和联系

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