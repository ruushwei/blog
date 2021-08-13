---
title: java
date: 2021-07-29 11:10:43
---

<!-- toc -->
### - 常见Java集合类

**非线程安全**  

ArrayList:  底层数组  
LinkedList:  底层双向链表  
HashMap: (*重要必考，后面单独问*)   
数组加链表，链表长度 > 8转化为红黑树  

### - ArrayList 和 LinkedList的区别
1. 数据结构  数组 / 双向链表 
2. 是否支持快速随机访问
3. 随机插入删除速度, ArrayList需要整体位移

### - ArrayBlockingQueue和LinkedBlockingQueue的实现原理

**ArrayBlockingQueue是object数组实现的线程安全的有界阻塞队列** 
1.数组实现：使用数组实现循环队列    
2.有界：内部为数组实现，一旦创建完成，数组的长度不能再改变    
3.线程安全：使用了 ReentrantLock 和 两个 condition(notEmpty, notFull) 来保证线程安全   
4.阻塞队列：先进先出。当取出元素的时候，若队列为空，wait直到队列非空（notEmpty）；当存储元素的时候，若队列满，wait直到队列有空闲（notFull）

**LinkedBlockingQueue是一个单向链表实现的阻塞队列** 
1.链表实现： head是链表的表头， last是链表的表尾。取出数据时，都是从表头head处取出，出队； 新增数据时，都是从表尾last处插入，入队。   
2.可有界可无界：可以在创建时指定容量大小，防止队列过度膨胀。如果未指定队列容量，默认容量大小为Integer.MAX_VALUE  
3.线程安全：使用了 两个 ReentrantLock 和 两个 condition，putLock是插入锁，takeLock是取出锁；notEmpty是“非空条件”，notFull是“未满条件”。通过它们对链表进行并发控制   
4.阻塞队列：先进先出。当取出元素的时候，若队列为空，wait直到队列非空（notEmpty）；当存储元素的时候，若队列满，wait直到队列有空闲（notFull）

**PriorityBlockingQueue**
PriorityBlockingQueue(具有优先级的无限阻塞队列). 是一个支持优先级的无界阻塞队列，内部结构是数组实现的二叉堆
线程安全的排序。

**SynchronousQueue**
SynchronousQueue (SynchronousQueue是一个不存储元素的阻塞队列。每一个put操作必须等待一个take操作，否则不能继续添加元素), 每一个线程的入队操作必须等待另一个线程相应的出队（take）操作，相反，每一个线程的出队操作必须等待另一个线程相应的入队操作。

Executors.newCachedThreadPool() 使用的SynchronousQueue， 实际就不想往队列里放元素，就是想有多少任务就生成多少线程去做。线程拉满，不做任务堆积。 但newCachedThreadPool(). maxThreadSize 是Integer.MAX_VALUE, 容易内存溢出。

**白话**
ArrayBlockingQueue和LinkedBlockingQueue都是线程安全的阻塞队列。 ArrayBlockingQueue底层是数组实现，是有界的队列，线程安全是使用一个 ReentrantLock 和 两个 condition实现的。 LinkedBlockingQueue底层是单向链表实现，是可有界可无界的队列，线程安全是使用两个 ReentrantLock 和 两个 condition实现的。

为什么ArrayBlockingQueue是一个ReentrantLock，LinkedBlockingQueue是两个？
ArrayBlockingQueue在插入、取出的时候，都会对index做变更，有并发问题，需要同步处理。 
LinkedBlockingQueue从队头取出，从队尾插入，链表结构所以两种操作可并行。所以用两个lock, 提升并发能力，还用了一个AtomicInteger count; // 记录总数

### - 阻塞队列原理

核心思想就是：空就不让你取，满就不让你加。
进行操作的方法，在操作前，都必须加锁。
主要就是一个 ReentrantLock，和两个由它创建的 Condition：notFull、notEmpty。
然后，在take、put、enqueue、dequeue 四个方法里面对这两个信号条件进行控制。


### - 项目中用到过哪些数据结构

queue, map, set, list, 堆（priorityQueue）


### - BIO、NIO、AIO 的区別和联系
https://mp.weixin.qq.com/s/QrE-PLCppwzfX3JtvWx5QA

BIO：同步、阻塞。服务器实现模式为一个连接一个线程,服务器端为每一个客户端的连接请求都需要启动一个线程进行处理

NIO：多路复用技术，同步非阻塞。服务器实现模式为客户端的连接请求都会注册到多路复用器上,用同一个线程接收所有连接请求

AIO：异步非阻塞IO

BIO里用户最关心“我要读”，NIO里用户最关心”我可以读了”，在AIO模型里用户更需要关注的是“读完了” 

### - 谈谈对Java NIO的了解
NIO（Non-blocking I/O，在Java领域，也称为New I/O），是一种同步非阻塞的I/O模型，也是I/O多路复用的基础，已经成为解决高并发与大量连接、I/O处理问题的有效方式。

传统的BIO模型严重依赖于线程。但线程是很”贵”的资源。创建和销毁成本很高，本身占用较大内存，切换成本是很高。 

NIO由原来占用线程的阻塞读写变成了单线程轮询事件，找到可以进行读写的网络描述符进行读写。大大地节约了线程，为处理海量连接提供了可能

NIO的主要事件有几个：读就绪、写就绪、有新连接到来。

首先需要注册当这几个事件到来对应的处理器。然后在合适的时机告诉事件选择器
；其次，用一个死循环选择就绪的事件，会执行系统调用（Linux 2.6之前是select、poll（轮询），2.6之后是epoll（通知））

总结NIO带来了什么：   
- 事件驱动模型  
- 避免多线程
- 单线程处理多任务
- 非阻塞I/O，I/O读写不再阻塞，而是返回0
- 基于block的传输，通常比基于流的传输更高效(buffer)
- 更高级的IO函数，zero-copy
- IO多路复用大大提高了Java网络应用的可伸缩性和实用性
- JavaNIO使用channel、buffer传输，更高效
  
白话：nio 的核心在selector，相比于BIO一连接一线程，nio 一线程可以监听多个网络连接的文件描述符，监听处理其中的事件。这样很好的节省了线程资源，会增加服务端的并发能力和吞吐。

### - 谈谈对hashMap的了解

基本：底层数据结构, 数组 + 链表，长度超过8变为红黑树，非线程安全
加分：HashMap在put时, 经过了两次hash，一个是JDK自带的对对象key的hash，一个是内部扰动函数hash，做高16位与低16位异或，加大散列的效果
加分：HashMap 的长度为什么是2的幂次方。1. 散列值用之前要对数组长度取模，求桶位(n - 1) & hash 更高效；2. 扩容的时候也可以用 hash值与扩容后的最高位 & 判断节点是在高位还是低位，更高效。
加分：HashMap 非线程安全，多线程操作导致死循环问题，1.7  并发下头插形成循环链表，1.8 用尾插修复， 但仍可能造成丢失数据。

HashMap.Node<K, V>[] table;

get是怎么做的
1. hash。 hashcode高低16位异或得到hash值， (h = key.hashCode()) ^ h >>> 16
2. 根据hash值拿到桶位第一个节点，判断是否可以直接返回
3. 判断节点中结构类型是树还是链表，树则调用树查询方法，链表则遍历链表比较

put是怎么做的
1. hash。 hashcode高低16位异或得到hash值， (h = key.hashCode()) ^ h >>> 16
2. 根据hash值拿第一个桶位的节点，如果为空则新建节点插入
3. 判断节点结构是树还是链表，树则调用树插入的方法, 链表则添加到表尾，如果链表节点到了8个，则触发treeifyBin() ！如果整体节点数量 > threshold, 则resize()

resize是怎么做的
1. new HashMap.Node[newCap]，遍历每一个桶位
2. 如果只有一个节点直接赋到新桶位，新桶位计算方式：e.hash & newCap - 1
3. 如果节点数据结构为树，则走树的split()
4. 如果节点数据结构为链表，遍历链表，判断属于新桶还是老桶，通过：e.hash & oldCap(2^n), 通过最高位的0，1值

白话：hashmap是非线程安全的集合。底层是数组+链表，链表长度超过8会转成红黑树。hashmap中table的长度总是2的幂次方，这样可以使用&运算代替取余运算(%)来提高效率，在求桶位和扩容时获取节点新位置在左边还是右边的时候。

### - loadFactor和threshold的关系?   

size: 当前共有多少个KV对
capacity: 当前HashMap的容量是多少, 桶位数, table大小, 默认是16
loadFactor: 负载因子，默认0.75
threshold: loadFactor* capacity，

白话：负载因子就是控制碰撞几率的，调的小，碰撞的概率小，调的大，碰的概率就大。阈值就是扩容时机。阈值等于负载因子乘以table大小