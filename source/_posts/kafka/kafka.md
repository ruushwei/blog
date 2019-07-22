---
title: kafka基本概念、原理
date: 2018-02-09 17:23:22
tags:
- kafka
categories: kafka
keywords:
- kafka
---

今天公司内部琼琼分享了kafka，也是我一直想听的，本文主要资料来源于该同学的分享，稍作整理，重点的个人理解，留作记录，否则怕过几天就忘记了。。


### 一、简介

| 概念    | 定义                                       | 理解                                       |
| ----- | ---------------------------------------- | ---------------------------------------- |
| kafka | 分布式，基于发布/订阅的消息系统                         | 集群，时间驱动                                  |
| 使用语言  | scala                                    | scala可能确实不错                              |
| 特点    | 快速, 可扩展, 可持久化,  高吞吐量, 近实时, 支持消息分区, 批量读写消息 |                                          |
| 设计目标  | 高吞吐率                                     | 利用磁盘顺序读写的特性；topic可以划分为多个partition，提高并发性；支持数据批量发送和拉取 |
|       | 访问与持久化常数时间复杂度                            | 未得证                                      |
|       | 同步和异步复制两种HA                              | HA: 高可用性                                 |
|       | 分布式消费 ,保证每个Partition消息顺序传输               | 一个group里不会有多个consumer消费一个partition，所以可以保证一个partition消息顺序性 |
|       | 支持在线水平扩展                                 |                                          |



<!--more-->

### 二、基础概念

| 概念             | 理解                                       |
| -------------- | ---------------------------------------- |
| 消息             | 数据单元， 一串字节构成，其中主要由key和value，  通过key将消息路由到指定分区，value是消息的有效负载 |
| topic          | 一个主题，可以看作是一系列消息的集合                       |
| partition      | **topic可以划分多个分区，每个分区自己是有序的，但整个topic无法保证有序性** |
| broker         | Kafka集群中的一个节点, 一台机器                      |
| 副本             | 每个partition可以有多个副本， 分为Leader副本和Follower副本，所有的读写请求都由Leader副本进行，Follower副本仅仅从Leader副本把数据同步到本地 |
| HW             | （HightWatermark）标记一个特殊的offset，当消费者处理消息时只能拉到HW之前的消息 |
| ISR集合          | 表示目前可用，且消息量与Leader副本相差不多的副本集合。满足（1）该副本所在的节点与zk保持连接；（2）副本最后一条消息的offset与Leader副本的offset之间的差值不能超过指定的阈值。 |
| 生产者            | 生产消息，并将消息按照一定规则推送到topic所在的分区             |
| 消费者            | 从topic中拉取消息，并对消息进行消费                     |
| Consumer Group | **1个 Consumer Group 有 n个 consumer ,  1个 consumer 有 1个 Consumer Group, 如果 n partition and n+1 consumer ,then 1 consumer don't work** |



### 三、demo



1、producer



一个异步发送消息的例子

```Java
public class myProducer {
    public static void main(String [] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "127.0.0.1:9092");
        props.put("client.id", "myProducer");
        props.put("key.serializer", "org.apache.kafka.common.serialization.IntegerSerializer");
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
        KafkaProducer producer = new KafkaProducer<>(props);
        String topic = "myTest";
        boolean isAsync = true;//异步发送
        int messageNo = 1;
        while (true) {
            String messageStr = "Message_" + messageNo;
            long startTime = System.currentTimeMillis();
            if (isAsync) {
                ProducerRecord<Integer, String> record = new ProducerRecord(topic, messageNo, messageStr);
                producer.send(record, new myProducerCallBack(startTime, messageNo, messageStr));

            }
            messageNo ++;
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }
}
class myProducerCallBack implements Callback {
    private final long startTime;

    private final int key;

    private final String message;

    public myProducerCallBack(long startTime, int key, String message) {
        this.startTime = startTime;
        this.key = key;
        this.message = message;
    }

    @Override
    public void onCompletion(RecordMetadata recordMetadata, Exception e) {
        long time = System.currentTimeMillis() - startTime;
        if (Objects.nonNull(recordMetadata)) {
            System.out.println("message key=" + key + ", message=" + message + ", send to partition=" + recordMetadata
                    .partition() + ", offset=" + recordMetadata.offset() + ", cost time=" + time);
        } else {
            e.printStackTrace();
        }

    }
}
```



这里异步发送和同步发送的区别？

如果是同步的话，发送速度慢，且异步不会有顺序性问题，我觉得都用异步就好了。



2、consumer



一个同步接收消息的例子

```Java
public class myConsumer {
    public static void main(String[] args) {
        Properties props = new Properties();
        props.put("bootstrap.servers", "127.0.0.1:9092");
        props.put("group.id", "test");
        props.put("enable.auto.commit", "true");
        props.put("auto.commit.interval.ms", "1000");
        props.put("session.timeout.ms", "30000");
        props.put("key.deserializer", "org.apache.kafka.common.serialization.IntegerDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        consumer.subscribe(Arrays.asList("myTest"));
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(100);
            for (ConsumerRecord<String, String> record : records) {
                System.out.println("myConsumer record = " + record);
            }
        }
    }
}
```



如果是异步呢？

如果多个partition的话，肯定要异步多线程起多个consumer去拉和处理。






### 四、原理解析



我以 生产者 -> 服务器 -> 消费者 的顺序写



#### 1、生产者原理



我们主要就是要知道他的 设计架构 和 如何保证的发送到同一分区消息的顺序性,



##### (1)设计架构



生产者工作主要涉及两个线程协同工作，主线程将数据封装为ProducerRecord并通过send()方法将消息放入RecordAccumulator中暂存，sender线程负责将消息构成请求，并最终执行网络I/O线程，将RecordAccumulator暂存的消息发送出去。



![producer](https://ws1.sinaimg.cn/large/006tKfTcly1foaf3lvnujj31kw1b0tm5.jpg)



partition 选择： 当调用send()方法发消息但没有在ProducerRecord中指定partition字段时，KafkaProducer就会调用Partitioner.partition()方法进行选择。默认的实现类DefaultPartitioner会按照key的哈希值取模分区的个数来确定分区；如果没有指定key，会将消息均匀分配到topic对应的分区中，具体的根据内部一个counter取模。



**图上的长条都是队列，而且是Deque，可头插的队列.**



**RecordAccumulator**作为主线程和Sender线程之间的缓冲必须是线程安全的，其主要字段是一个以TopicPartition为key，ArrayDeque<RecordBatch>为value的ConcurrentMap，用于存放暂存的消息。Send()方法会调用RecordAccumulator.append()方法将消息放入batchs中。暂存在batchs中的RecordBatch在被Sender线程发送到服务器之前会调用RecordAccumulator.ready()方法获取符合发送消息条件的节点结合，具体条件为：

（1）Deque中有多个RecordBatch或者第一个RecordBatch是否满了；

（2）是否超时，RecordBatch缓存时间超过了设置的时间；

（3）BufferPool的空间耗尽了；

（4）Sender线程准备关闭；





**InFlightRequests**是NetWorkClient中的一个字段，只要作用是缓存已经发送出去单没有收到响应的ClientRequest，底层通过Map<String, Deque<NetworkClient.InFlightRequest>>来实现，已经成功处理的请求会从该字段中删除。下面就简单讲解一下Kafka如何利用RecordAccumulatorm， InFlightRequests等实现发送到同一分区消息的顺序性。

（1）调用RecordAccumulator.ready()方法获取可以发送到指定分区的RecordBatch时，只会获取Deque中的第一个RecordBatch;

（2）经过（1）中得到的RecordBatch通过NetWorkClient发向Kafka服务器指定Node之前会检查Metadata是否需要更新，连接是否成功，以及调用InFlightRequests.canSendMore()方法判断是否发送改请求。InFlightRequests.canSendMore() 只有在队列为空，或者队列的头部的请求已经完成完成的条件下才返回true。

（3）在NetWorkClient成功收到Kafka集群中的响应后。如果是成功响应，会遍历ClientRequest中的RecordBatch，执行RecordBatch.done()方法，并释放RecordBatch底层的ByteBuffer空间。RecordBatch.done()会执行Recordbatch中每条消息对应的callback函数。如果是异常响应，则尝试将RecordBatch放入RecordAccumulator中重新发送。如果重试次数已经达到上限，或者不允许重试，则直接调用RecordBatch.done()进行异常完成callback处理。



##### (2)同一分区消息的顺序性（重点）



1. 同步发送

同步发送的话， 中间的缓存里一定是只有一个的，下次send是在确定发送到kafka服务器成功之后才会进行，相当于整个topic是同步的，所以partition也肯定是同步的。



2. 异步发送



中间缓存里的队列是partition队列，这个队列里只有上一个recordBatch结束了，才会走下一个，保证顺序性，

如果失败，会插回partition队列的头部，详情尽在图中

![partition](https://ws1.sinaimg.cn/large/006tKfTcly1foaf3l9dy8j31gq14sk1v.jpg)





#### 2、kafka集群原理

kafka并不是分片的，是主从架构



##### (1)leader副本选举

如果某个分区的leader挂了，那么Follewer副本将会进行选举产生一个新的leader来进行读写操作。kafka中leader的选举是从该topic的ISR列表中进行挑选。通常，先使用ISR里的第一个副本，如果不行依次类推。如果ISR集合中的Follower全部宕机，会选择第一个恢复的副本（不一定是ISR）作为leader。通过ISR，kafka需要的冗余度较低，可以容忍的失败数较高。



##### (2)高性能策略

（1）顺序读写磁盘

​       Kafka的设计充分利用了磁盘顺序读写的性能，Partition相当于一个数据，Broker顺序Partition进行读写。同时，Consumer在进行消费时也顺序的读取这些数据。Kafka官方给出了顺序读写磁盘和随机读取磁盘的性能测试数据（Raid-5，7200rpm）：Sequence I/O: 600MB/s， Random I/O: 100KB/s。并且据资料显示，顺序读写磁盘很多时候比随机访问内存快的多。当对Partition没必要的数据进行删除时，Kafka也是通过整个文件的方式进行删除。具体的，Kafka将Partition分为多个Segment，每个Segment对应一个物理文件，删除是删除整个Segment文件。

​      Kafka也对Page Cache进行了充分的利用，写磁盘时只是将数据写入Page Cache，并不保证数据一定完全写入磁盘。当读操作发生时，先从PageCache中查找，如果发生缺页才进行磁盘调度，最终返回需要的数据。

（2）数据文件的索引

​      如上所述，Kafka将Partition分为多个Segment，每个Segment以该文件中最小的offset命令。这样在查找指定offset的Message的时候，用二分查找就可以定位到该Message在哪个Segment中。同时，为了进一步提高查找的效率，Kafka为每个分段后的数据文件建立了索引文件，文件名与数据文件的名字是一样的，只是文件扩展名为.index。索引文件索引的方式采用了稀疏存储的方式，每隔一定字节的数据建立一条索引。这样避免了索引文件占用过多的空间，从而可以将索引文件保留在内存中。



#### 3、消费者原理

##### (1)客户端

Consumer接口的是实现类是KafkaConsumer类，其工作流程大致如下图所示。可以看出KafkaConsumer通过SubscriptionState来管理订阅的topic集合及Partition的消费状态，通过ConsumerCoordinator与Kafka服务端的GroupCoordinator完成**Rebalance**操作及**offset**提交操作，Fetcher负责从Kafka中拉取数据并进行解析。上述操作的所有操作都是通过ConsumerNetworkClient进行缓存并发送的，在该类中还维护了定时任务队列用来完成心跳发送及AutoCommitTask任务。



这部分我理解的还不是很好，都是客户端的一些操作。

![consumer](https://ws2.sinaimg.cn/large/006tKfTcly1foaf3jgpu4j31kw0tyalo.jpg)



##### (2)offset - partition偏移管理 (重要)

新版的consumer为了缓解zk集群的压力，将consumer的offset信息保存在上述GroupCoordinator维护的内部主题中。该主题中保存了每个Consumer Group某一时刻提交的offset信息，其数据格式大致入下图所示，key为groupid,topic,partition组成，value为offset信息。并且通过配置compact策略，使得每个key对应的offset总是最新的。同时，kafka通过Math.abs(groupId.hashCode()) % numPartitions确定某个Consumer Group的partition offset信息存储的分区.

![consumer_off](https://ws2.sinaimg.cn/large/006tKfTcly1foaf3kgqrfj31kw0wrq85.jpg)

也就是consumer如果是更新的话，就会更新对应partition中对应key的value即offset数据，下次consumer再读的时候，返回consumer该offset



如果是consumer第一次获取数据的话，就需要服务端对这个新consumer的到来做一次rebalance了(重分配)，需要把partition再尽量均分给不同的consumer



##### (3)rebalance (重要)

在同一个Consumer Group中同一个topic的不同分区会被不同的消费者消费，并且当新的消费者加入或者离开时，会对消费的分区进行重分配，这个工作是由Rebalance操作完成。在开始对Rebalance操作进行介绍前，先说明



Rebalance操作发生的时机：

- 有新的消费者加入Consumer Group;
- Consumer Group中的消费者宕机下线；
- 有消费者主动退出Consumer Group；
- Consumer Group订阅的任一topic出现分区变化；
- 消费者调用unsubscribe取消对某一个topic的订阅；

​    

 Rebalance操作主要包括三个阶段：

（1）消费者查找管理当前Consumer Group的GroupCoordinator

​        该阶段会向Kafka集群中负载最小的Broker发送GroupCoordinatorRequest，并处理响应找到其所对应的GroupCoordinator的网络位置。

（2）Join Group阶段

​        在该阶段Consumer会向GroupCoordinator发送JoinGroupRequest，并处理各自的JoinGroupResponse。JoinGroupResponse的处理逻辑如下：

- 解析JoinGroupResponse，获取GroupCoordinator分配的memberId,generation等信息，更新到本地；
- 消费者根据leadId检测自己是不是Leader。如果是leader则进入onJoinLeader()，否者进入onJoinFollower()方法；
- leader根据group_protocol字段指定的Partition分配策略，查找响应的PartitionAssignor对象；
- leader会得到全部消费者订阅的topic，并将其添加到其SubscriptionState.groupSubscription集合中。Follower则只关心自己订阅的topic;
- 更新metadata;
- leader调用PartiionAssignor.assign()方法进行分区分配；

（3）Sync Group阶段

​        在该阶段leader会将其分配的结果封装为SyncGroupRequest发送给GroupCoordinator进行处理，并处理SyncGroupResponse。





具体实现大概就是GroupCoordinator之间一顿操作，将partition分配，具体的分配策略如下：



在Rebalance操作的Join Group阶段leader会调用PartiionAssignor.assign()进行分区分配。开发者可以通过继承PartiionAssignor接口自己实现分区分配策略，Kaka提供了两个默认实现RangeAssignor和RoundRobinAssignor。

(1) RangeAssignor

​       对于一个topic，n = 分区数/消费者数量，m=分区数%消费者数量，前m个消费者每个分配n+1个分区，后面的 （消费者数量-m）个消费者每个分配n个分区。比如有两个消费者c1,c2，有topic t1,t2，分别有三个分区。那么，按这种方式的分配结果是c1t1p1,c1t1p2,c2t1p3,c1t2p1,c1t2p2,c2t2p3。显然，这种方式在一定程度上是不公平的。

(2) RoundRobinAssignor

​       该分配器会把这个group订阅的所有TopicPartition排序，排序是先按topic排序，同一个topic的分区按partition id排序。然后，依次分配给消费者。上述例子利用RoundRobinAssignor的分配结果是：c1t1p1,c2t1p2,c1t1p3,c1t2p1,c2t2p2,c1t2p3



### 五、消息队列对比

|        | Kafka      | RabbitMQ   | ZeroMQ            | ActiveMQ      | RocketMQ   |
| ------ | ---------- | ---------- | ----------------- | ------------- | ---------- |
| 吞吐量    | 高（17.3W/s） | 低 (2.6W/s) | 高（“史上最快”） (29W/s) | 低（比RabbitMQ低） | 高（11.6w/s） |
| 可靠性    | 保证         | 保证         | 不保证               | 不保证           | 保证         |
| 持久化    | 支持(大量)     | 支持（少量）     | 不支持               | 支持（少量）        | 支持（大量）     |
| 消息回溯   | 支持         | 不支持        | 不支持               | 不支持           | 支持         |
| 只要开发语言 | Scala/Java | Erlang     | C                 | Java          | Java       |
| 性能稳定性  | 较差         | 好          | 很好                | 好             | 一般         |







### 六、配置信息



#### 1、producer

| 配置项                                      | 默认值                | 作用                                       |
| ---------------------------------------- | ------------------ | ---------------------------------------- |
| `batch.num.messages`                     | 200                | `一个batch发送消息的数量`                         |
| batch.num.messages                       | 200                | 采用异步模式时，一个batch缓存的消息数量。达到这个数量值时producer才会发送消息。 |
| batch.size                               | 16384              | 批量提交的batch字节大小                           |
| buffer.memory                            | 33554432           | BufferPool缓存大小                           |
| [client.id](http://client.id/)           | 生成一个id             | 标识生产者的一个字符串                              |
| compression.type                         | none               | Producer用于压缩数据的压缩类型,默认是无压缩。正确的选项值是none、gzip、snappy。 |
| interceptor.classes                      |                    | 消息拦截类的list                               |
| linger.ms                                | 0                  | 数据缓存的最大延迟时间                              |
| message.send.max.retries                 | 3                  | 消息发送失败后重试次数                              |
| metadata.broker.list                     |                    | 生产者获取元数据地址                               |
| partitioner.class                        | DefaultPartitioner | 分区分配策略，默认是kafka.producer.DefaultPartitioner，取模 |
| queue.buffering.max.messages             | 10000              | 异步模式下缓冲的消息的最大数量                          |
| queue.buffering.max.ms                   | 5000               | 异步模式下缓冲数据的最大时间                           |
| [queue.enqueue.timeout.ms](http://queue.enqueue.timeout.ms/) | -1                 | 异步模式下，消息进入队列的等待时间。若是设置为0，则消息不等待，如果进入不了队列，则直接被抛弃 |
| request.required.acks                    | 0                  | 消息确认模式，0：不保证消息到达确认；1：发送消息，等待leader收到确认；-1：发送消息，等待leader收到确认，并进行复制操作后才返回。 |
| request.timeout.ms                       | 10000              | 发送消息最长等待时间                               |
| send.buffer.bytes                        | 100*1024           | socket的缓存大小                              |
| topic.metadata.refresh.interval.ms       | 600*1000           | 生产者定时更新topic元信息的时间间隔                     |





#### 2、consumer

| 配置项                                      | 默认值                | 作用                                       |
| ---------------------------------------- | ------------------ | ---------------------------------------- |
| `batch.num.messages`                     | 200                | `一个batch发送消息的数量`                         |
| batch.num.messages                       | 200                | 采用异步模式时，一个batch缓存的消息数量。达到这个数量值时producer才会发送消息。 |
| batch.size                               | 16384              | 批量提交的batch字节大小                           |
| buffer.memory                            | 33554432           | BufferPool缓存大小                           |
| [client.id](http://client.id/)           | 生成一个id             | 标识生产者的一个字符串                              |
| compression.type                         | none               | Producer用于压缩数据的压缩类型,默认是无压缩。正确的选项值是none、gzip、snappy。 |
| interceptor.classes                      |                    | 消息拦截类的list                               |
| linger.ms                                | 0                  | 数据缓存的最大延迟时间                              |
| message.send.max.retries                 | 3                  | 消息发送失败后重试次数                              |
| metadata.broker.list                     |                    | 生产者获取元数据地址                               |
| partitioner.class                        | DefaultPartitioner | 分区分配策略，默认是kafka.producer.DefaultPartitioner，取模 |
| queue.buffering.max.messages             | 10000              | 异步模式下缓冲的消息的最大数量                          |
| queue.buffering.max.ms                   | 5000               | 异步模式下缓冲数据的最大时间                           |
| [queue.enqueue.timeout.ms](http://queue.enqueue.timeout.ms/) | -1                 | 异步模式下，消息进入队列的等待时间。若是设置为0，则消息不等待，如果进入不了队列，则直接被抛弃 |
| request.required.acks                    | 0                  | 消息确认模式，0：不保证消息到达确认；1：发送消息，等待leader收到确认；-1：发送消息，等待leader收到确认，并进行复制操作后才返回。 |
| request.timeout.ms                       | 10000              | 发送消息最长等待时间                               |
| send.buffer.bytes                        | 100*1024           | socket的缓存大小                              |
| topic.metadata.refresh.interval.ms       | 600*1000           | 生产者定时更新topic元信息的时间间隔                     |
