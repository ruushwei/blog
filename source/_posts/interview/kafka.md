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

针对消息有序的业务需求，还分为全局有序和局部有序。
- 全局有序：一个Topic下的所有消息都需要按照生产顺序消费。要满足全局有序，需要1个Topic只能对应1个Partition。consumer内部单线程。
- 局部有序：一个Topic下的消息，只需要满足同一业务字段的要按照生产顺序消费。例如：Topic消息是订单的流水表，包含订单orderId，业务要求同一个orderId的消息需要按照生产顺序进行消费。
	在发消息的时候指定Partition Key，Kafka对其进行Hash计算，根据计算结果决定放入哪个Partition。这样Partition Key相同的消息会放在同一个Partition。consumer内部单线程消费或接亲缘型线程池（内存队列）。此时，Partition的数量仍然可以设置多个，提升Topic的整体吞吐量。

### - kafka如何保证消息可靠性（不丢消息）

1. 多副本写入 + 数据持久化，通过后台线程将消息持久化到磁盘
2. 可设置写入副本情生产者ack机制，况，ack=0 可不写入就成功，ack=1 写入leader副本，ack=all 写入全部副本
3. 生产者重试机制，出现网络问题可自动重试，可设置重试间隔
4. 消费者使用手动提交，可保证消费一次

### - kafka如何保证高可用

副本机制。  
Kafka在0.8版本之前是没有HA机制来确保高可用的，当某一个broker挂掉，partition就挂了。即存在单点故障。
0.8之后提供了副本机制，副本机制会将 一个broker下某个topic的一个partition放入到另外一个broker里，这个备份的分区和原分区都叫做副本（replica）。在所有的副本里，只能有一个leader，其余的副本都作为follower，同一时间内只有leader负责读写，follower不起任何作用。这样做的原因是为了确保消费者单调读 且 确保能立即读取写入的信息（Read-your-writes）。其他所有的follower会异步的拉去leader消息，拉的快的会进入ISR里，拉的慢得超过replica.lag.time.max.ms 配置的超时值的，会被踢进OSR里，这个过程是动态的，如果一个follower开始没跟上leader的消息写入速度，被踢出了ISR，等到跟上后又会重新进入ISR。当leader挂掉之后，为了保证高可用性，从ISR中获取一个副本，升格为leader。如果ISR全挂了，有两种策略，一是等待ISR第一个恢复的副本，二是开启unclean从OSR中选择一个副本作为leader。为保证高可用 可以选择第二种牺牲一致性的方式。


### - kafka服务端架构
服务端是reactor模式，**selector + threadPool**   
acceptor里有个selector, 每个processor中有一个selector，都是NIO,IO多路复用     
processor会通过注册/取消 OP_READ _ 事件，保证每个连接上只有一个请求和一个对应的响应，从而实现每个连接请求的顺序性。  

kafka是自身实现的nio框架，rocketmq是使用的netty

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
- 顺序读写。Kafka的message是不断追加到本地磁盘文件末尾的，而不是随机的写入，这使得Kafka写入吞吐量得到了显著提升 。
- 页缓存。kafka重度依赖页缓存技术，kafka只是将数据写入页缓存（内存）中而不直接操作磁盘，由操作系统决定什么时候把页缓存中数据刷到磁盘上。同时写入页缓存的数据是按照磁盘顺序去写入的，因此刷到磁盘上的速度也较快。当读操作发生时，先从页缓存中查询是否有所需信息，若没有才会调度磁盘。
- 零拷贝。这里的零拷贝并非指一次拷贝都没有，而是避免了在内核空间和用户空间之间的拷贝。页缓存 -> socket缓冲区。
- 分区分段 + 索引。通过这种分区分段的设计，Kafka的message消息实际上是分布式存储在一个一个小的segment中的，每次文件操作也是直接操作的segment。为了进一步的查询优化，Kafka又默认为分段后的数据文件建立了索引文件，就是文件系统上的.index文件。这种分区分段+索引的设计，不仅提升了数据读取的效率，同时也提高了数据操作的并行度。
- 批量压缩。批量的消息可以通过压缩的形式传输并且在日志中也可以保持压缩格式，直到被消费者解压缩，减少网络IO.

#### - controller broker机制
1. 本身也是一个普通的Broker，Kafka集群中始终只有一个Controller Broker。
2. controller broker具体作用： 
         创建、删除主题，增加分区并分配leader分区
         集群Broker管理（新增 Broker、Broker 主动关闭、Broker 故障)
         preferred leader选举
         分区重分配
3. Broker选举：第一个启动的broker会在zk中创建临时节点/controller，利用zk的强一致性，成为唯一的控制器节点，而其余的broker会监听该节点。随后由controller决定每个broker中副本的master，每当broker挂掉，controller会检测其副本，重新决定出master。当控制器broker断开时，/controller临时节点会删除，其余连接的broker将收到事件通知，抢占式注册为新的controller。

#### - kafka怎么记录消费offset：

kafka在消费的时候，会将消费的offset保存在kafka的特定topic中（_consumer_offsets）。当消费者需要恢复消费进度，从该内部topic读取上次提交的偏移量，从而恢复进度，继续消费。

rocketmq中，每个消费者组会在broker中创建一个消费进度管理器，用于保存每个消费者消费每个messagequeue的消费进度。

#### - rocketmq & kafka
rocketmq功能性和可靠性更强一些。kafka性能吞吐更强一些。

- 数据存储形式
kafka是存partition维度的数据顺序写入，每个分区是一个队列，分segment存储。
rocketmq是broker不分topic的所有数据顺序写入commitlog, 每个消费者组对应一个consumerqueue, cq里有消费的offset标识，每个cq里有多个messagequeue 类似kafka的partition，用于提升顺序消费的并发性。
这里明显rocketmq在读取的时候可能导致对磁盘的随机读，影响读取性能，但使用了缓存、批量读取、内存等提高读取性能。
- 性能
Kafka单机写入 TPS 号称在百万条/秒；
RocketMQ 大约在10万条/秒。
结论：追求性能的话，Kafka单机性能更高。
- 可靠性
RocketMQ支持异步/同步刷盘;异步/同步Replication；
Kafka使用异步刷盘方式，异步Replication。
结论：RocketMQ所支持的同步方式提升了数据的可靠性。
- 实时性
均支持pull长轮询，RocketMQ消息实时性更好
结论：RocketMQ 胜出。
- 支持的队列数
Kafka单机超过64个队列/分区，消息发送性能降低严重；
RocketMQ 单机支持最高5万个队列，性能稳定
结论：长远来看，RocketMQ 胜出，这也是适合业务处理的原因之一
- 消息顺序性
Kafka 某些配置下，支持消息顺序，但是一台Broker宕机后，就会产生消息乱序；
RocketMQ支持严格的消息顺序，在顺序消息场景下，一台Broker宕机后，
发送消息会失败，但是不会乱序；
结论：RocketMQ 胜出
- 消费失败重试机制
Kafka消费失败不支持重试
RocketMQ消费失败支持定时重试，每次重试间隔时间顺延。
- 定时/延时消息
Kafka不支持定时消息；
RocketMQ支持定时消息
- 分布式事务消息
Kafka不支持分布式事务消息；
RocketMQ支持分布式事务消息
- 消息查询机制
Kafka不支持消息查询
RocketMQ支持根据Message Id查询消息，也支持根据消息内容查询消息
- 消息回溯
Kafka理论上可以按照Offset来回溯消息
RocketMQ支持按照时间来回溯消息，精度毫秒，例如从一天之前的某时某分某秒开始重新消费消息.