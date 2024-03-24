---
title: MQ
date: 2021-08-01 11:10:43
---

<!-- toc -->

### - mq使用场景

* 异步
    * 上游不需要同步拿到结果，执行过程又耗时
    * 好处：异步可以让服务端的线程不同步阻塞，提高服务端的吞吐量
* 解耦
    * 多个下游相同依赖
    * 好处：不与依赖方耦合，不用额外开发
* 削峰
    * 请求高峰期
    * 好处：保护下层存储

### - 如何保证消息顺序性

针对消息有序的业务需求，还分为全局有序和局部有序。
- 全局有序：一个Topic下的所有消息都需要按照生产顺序消费。要满足全局有序，需要1个Topic只能对应1个Partition。consumer内部单线程。
- 局部有序：一个Topic下的消息，只需要满足同一业务字段的要按照生产顺序消费
    - producer按shardingkey分片的方式发到不同的分区
    - consumer单线程消费分区或保证相同的sharding规则下顺序线程模型串行消费

### - 如何保证消息可靠性（不丢）
刷盘+重试、确认

rocketmq
- 多副本机制、持久化机制
- 生产者发消息重试&ack，生产者发送消息自动重试到最大次数，发送时可配置同步刷盘再返回ack
- broker发消息重试&ack，消费者接收到消息后会发送ack到broker否则broker也会重试

kafka
1. 多副本写入 + 数据持久化，通过后台线程将消息持久化到磁盘
2. 可设置写入副本情生产者ack机制，况，ack=0 可不写入就成功，ack=1 写入leader副本，ack=all 写入全部副本
3. 生产者重试机制，出现网络问题可自动重试，可设置重试间隔
4. 消费者使用手动提交，可保证消费一次


### - 如何保证高可用

- 分布式部署多分区
- 主从复制, 副本机制
- 同步复制，可配置broker节点之间同步复制
- 故障切换与恢复，自动检测触发故障转移


注：kafka副本机制。  
Kafka在0.8版本之前是没有HA机制来确保高可用的，当某一个broker挂掉，partition就挂了。即存在单点故障。
0.8之后提供了副本机制，副本机制会将 一个broker下某个topic的一个partition放入到另外一个broker里，这个备份的分区和原分区都叫做副本（replica）。在所有的副本里，只能有一个leader，其余的副本都作为follower，同一时间内只有leader负责读写，follower不起任何作用。这样做的原因是为了确保消费者单调读 且 确保能立即读取写入的信息（Read-your-writes）。其他所有的follower会异步的拉去leader消息，拉的快的会进入ISR里，拉的慢得超过replica.lag.time.max.ms 配置的超时值的，会被踢进OSR里，这个过程是动态的，如果一个follower开始没跟上leader的消息写入速度，被踢出了ISR，等到跟上后又会重新进入ISR。当leader挂掉之后，为了保证高可用性，从ISR中获取一个副本，升格为leader。如果ISR全挂了，有两种策略，一是等待ISR第一个恢复的副本，二是开启unclean从OSR中选择一个副本作为leader。为保证高可用 可以选择第二种牺牲一致性的方式。


### - 如何保证高性能

kafka 
- 顺序读写。Kafka的message是不断追加到本地磁盘文件末尾的，而不是随机的写入，这使得Kafka写入吞吐量得到了显著提升 。
- 页缓存。kafka重度依赖页缓存技术，kafka只是将数据写入页缓存（内存）中而不直接操作磁盘，由操作系统决定什么时候把页缓存中数据刷到磁盘上。同时写入页缓存的数据是按照磁盘顺序去写入的，因此刷到磁盘上的速度也较快。当读操作发生时，先从页缓存中查询是否有所需信息，若没有才会调度磁盘。
- 零拷贝。这里的零拷贝并非指一次拷贝都没有，而是避免了在内核空间和用户空间之间的拷贝。页缓存 -> socket缓冲区。
- 分区分段 + 索引。通过这种分区分段的设计，Kafka的message消息实际上是分布式存储在一个一个小的segment中的，每次文件操作也是直接操作的segment。为了进一步的查询优化，Kafka又默认为分段后的数据文件建立了索引文件，就是文件系统上的.index文件。这种分区分段+索引的设计，不仅提升了数据读取的效率，同时也提高了数据操作的并行度。
- 批量压缩。批量的消息可以通过压缩的形式传输并且在日志中也可以保持压缩格式，直到被消费者解压缩，减少网络IO.

rocketmq
- 顺序读写。
- 可异步刷盘
- 零拷贝。
- 高效的消息存储结构。commitlog，cq indexfile，简化了存储优化了读写
- 批量处理。支持批量发送和消费，减少网络交互


#### - rocketmq & kafka
rocketmq功能性更丰富一些。kafka性能在topic不多的时候更优一些。

功能性
kafka不支持延时消息，不支持消息轨迹追踪和消息过滤，最新版本才开始支持事务消息。
rocketmq支持消息过滤，在broker做过滤，减少网络压力；在consumer端可自定义过滤条件。

性能
kafka因为是partition维度的文件存储，当topic多的时候随机写变多，性能下滑明显；相比rocketmq因为都是顺序写入commitlog，在topic多时对性能也不会有影响。