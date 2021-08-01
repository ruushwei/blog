---
title: kafka
date: 2021-08-01 11:10:43
---

<!-- toc -->

### - mq使用场景

* 异步
    * 上游不需要同步拿到结果，执行过程又耗时
    * 好处：异步可以让服务端的线程不同步阻塞，提高服务端的吞吐量
    * eg: 视频制作
* 解耦
    * 多个下游相同依赖
    * 好处：不与依赖方耦合，不用额外开发
    * eg: 下单成功消息，不同的依赖方做不同的事情
* 削峰
    * 请求高峰期
    * 好处：保护下层存储
    * eg: 下单

### - kafka如何保证消息顺序性

1. kafka保证单partition有序，多partition之间没有顺序
2. kafka服务端先会有个requestQueue将请求排序，排序后单partition去做顺序写入持久化，读的时候partition维度顺序读

### - kafka如何保证消息可靠性

1. 多副本写入
2. 数据持久化，通过后台线程将消息持久化到磁盘
3. 生产者确认，可确认成功写入n个副本再ack（可配）
4. 消费组回复消费成功ack，确保消费一次

### - kafka如何保证高可用

副本机制。  
Kafka在0.8版本之前是没有HA机制来确保高可用的，当某一个broker挂掉，partition就挂了。即存在单点故障。
0.8之后提供了副本机制，副本机制会将 一个broker下某个topic的一个partition放入到另外一个broker里，这个备份的分区和原分区都叫做副本（replica）。在所有的副本里，只能有一个leader，其余的副本都作为follower，同一时间内只有leader负责读写，follower不起任何作用。这样做的原因是为了确保消费者单调读 且 确保能立即读取写入的信息（Read-your-writes）。其他所有的follower会异步的拉去leader消息，拉的快的会进入ISR里，拉的慢得超过replica.lag.time.max.ms 配置的超时值的，会被踢进OSR里，这个过程是动态的，如果一个follower开始没跟上leader的消息写入速度，被踢出了ISR，等到跟上后又会重新进入ISR。当leader挂掉之后，为了保证高可用性，从ISR中获取一个副本，升格为leader。如果ISR全挂了，有两种策略，一是等待ISR第一个恢复的副本，二是开启unclean从OSR中选择一个副本作为leader。为保证高可用 可以选择第二种牺牲一致性的方式。


### - kafka整体架构

![](https://ipic-1252327316.cos.ap-beijing.myqcloud.com/image/kafka/kafka%E6%9E%B6%E6%9E%84.jpg)

### - kafka服务端架构
服务端是reactor模式，**selector + threadPool**   
acceptor里有个selector, 每个processor中有一个selector，都是NIO,IO多路复用     
processor会通过注册/取消 OP_READ _ 事件，保证每个连接上只有一个请求和一个对应的响应，从而实现每个连接请求的顺序性。  

![](https://ipic-1252327316.cos.ap-beijing.myqcloud.com/image/kafka/kafka%E6%9C%8D%E5%8A%A1%E7%AB%AF%E6%9E%B6%E6%9E%84.jpeg)


### - kafka 基本特性
1. 高吞吐量、低延迟
kafka每秒可以处理几十万条消息，它的延迟最低只有几毫秒，每个topic可以分多个partition, consumer group 对partition进行consume操作。
2. 可扩展性
kafka集群支持热扩展
3. 持久性、可靠性
消息被持久化到本地磁盘，并且支持数据备份防止数据丢失
4. 容错性
允许集群中节点失败（若副本数量为n,则允许n-1个节点失败）
5. 高并发
支持数千个客户端同时读写


### - kafka 如何保证高性能
1. 分区
kafka将topic分区，每一个broker里面会保存topic的不同分区，这样就可以让一个消费者组同时消费不同的分区，提高吞吐。一个消费组下只应有一个消费者去消费一个分区。理想情况下，一个消费者对应一个partition性能最佳。
2. 页缓存+顺序写
kafka重度依赖页缓存技术，kafka只是将数据写入页缓存（内存）中而不直接操作磁盘，由操作系统决定什么时候把页缓存中数据刷到磁盘上。同时写入页缓存的数据是按照磁盘顺序去写入的，因此刷到磁盘上的速度也较快。当读操作发生时，先从页缓存中查询是否有所需信息，若没有才会调度磁盘。
3. Sendfile零拷贝
若页缓存中没有所需信息，需要去调度磁盘。原本网络io操作需要四个步骤 ： 硬盘 -> 内核态 PageCache -> 用户态程序读取 -> 内核态socket写入 -> 拷贝至网卡。Sendfile优化后，使用零拷贝可以将buffer从内核态和用户态间任意切换。PageCache仅会传递一个文件描述符给socket，就等效于直接从PageCache中拷贝至网卡。少两步IO操作。
4. 支持压缩


#### - controller broker机制
1. 本身也是一个普通的Broker，Kafka集群中始终只有一个Controller Broker。
2. controller broker具体作用： 
         创建、删除主题，增加分区并分配leader分区
         集群Broker管理（新增 Broker、Broker 主动关闭、Broker 故障)
         preferred leader选举
         分区重分配
3. Broker选举：第一个启动的broker会在zk中创建临时节点/controller，利用zk的强一致性，成为唯一的控制器节点，而其余的broker会监听该节点。随后由controller决定每个broker中副本的master，每当broker挂掉，controller会检测其副本，重新决定出master。当控制器broker断开时，/controller临时节点会删除，其余连接的broker将收到事件通知，抢占式注册为新的controller。

