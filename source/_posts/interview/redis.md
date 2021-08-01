---
title: redis11
date: 2021-07-31 11:10:43
---

<!-- toc -->


### - redis支持的基础数据类型

SDS （Simple Dynamic String）是 Redis 最基础的数据结构。直译过来就是”简单的动态字符串“。  
链表：自定义双向链表：listNode   
跳跃表：skipList. 链表的一种，是一种利用空间换时间的数据结构。跳表平均支持 O(logN)，最坏O(N)复杂度的查找。   
压缩链表：为了尽可能节约内存设计出来的双向链表。   
字典： hash
int, intset: 数字集合


### - redis支持的对象及实现方式

**String**

最常规的get/set操作，value可以是数字和string
	一般做一些复杂的计数功能的缓存
sds

**hash**

value存放的是结构化的对象
比较方便的就是操作其中的某个字段
在做单点登录的时候，就是用这种数据结构存储用户信息，以cookieId作为key，设置30分钟为缓存过期时间，能很好的模拟出类似session的效果

Redis 的 hash 表 使用 ziplist 和 字典 实现的。
键值对的键和值都小于 64 个字节, 键值对的数量小于 512。
都满足的时候使用 ziplist，否则使用字典。

**list**

使用List的数据结构，可以做简单的消息队列的功能
还有一个是，利用lrange命令，做基于redis的分页功能，性能极佳，用户体验好
一个场景，很合适---取行情信息。就也是个生产者和消费者的场景。LIST可以很好的完成排队，先进先出的原则。

底层是一个 ziplist 或者 linkedlist。
当列表对象保存的字符串元素的长度都小于64字节。保存的元素数量小于512个。

**set**

set堆放的是一堆不重复值的集合。所以可以做全局去重的功能  
为什么不用JVM自带的Set进行去重？因为我们的系统一般都是集群部署，使用JVM自带的Set，比较麻烦，难道为了一个做一个全局去重，再起一个公共服务，太麻烦了   
另外，就是利用交集、并集、差集等操作，可以计算共同喜好，全部的喜好，自己独有的喜好等功能

Redis 的集合底层是一个 intset 或者 一个字典（hashtable）。
这个比较容易理解：
当集合都是整数且不超过512个的时候，就使用intset。
剩下都是用字典。

**zset (sorted set)**

sorted set多了一个权重参数score,集合中的元素能够按score进行排列。可以做排行榜应用，取TOP N操作。

Redis 的有序集合使用 ziplist 或者 skiplist 实现的。
元素小于 128 个
每个元素长度 小于 64 字节。
同时满足以上条件使用ziplist，否则使用skiplist。

### - rediis 为什么快
官方给出的数字是读写性能可以打到10W/秒  
1. Redis所有数据都是放在内存中的
2. Redis使用了单线程架构，预防了多线程可能产生的竞争问题
3. Redis的IO多路复用，实现Reactor模型

### - redis的过期删除策略

定时删除,用一个定时器来负责监视key,过期则自动删除。虽然内存及时释放，但是十分消耗CPU资源。在大并发请求下，CPU要将时间应用在处理请求，而不是删除key,因此没有采用这一策略.

定期删除，redis默认每个100ms检查，是否有过期的key,有过期key则删除。需要说明的是，redis不是每个100ms将所有的key检查一次，而是随机抽取进行检查(如果每隔100ms,全部key进行检查，redis岂不是卡死)。因此，如果只采用定期删除策略，会导致很多key到时间没有删除

于是，惰性删除派上用场。也就是说在你获取某个key的时候，redis会检查一下，这个key如果设置了过期时间那么是否过期了？如果过期了此时就会删除。

### - redis的内存淘汰机制

在Redis内存使用达到设定上限时，触发满容淘汰策略释放内存

Redis 目前提供8种策略：

volatile-lru：从已设置过期时间的数据集中挑选最近最少使用的数据淘汰 

volatile-random：从已设置过期时间的数据集中任意选择数据淘汰  

volatile-lfu：从已设置过期时间的数据集中挑选最不经常使用的数据淘汰  

volatile-ttl：从已设置过期时间的数据集中挑选将要过期的数据淘汰  

allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的key（这个是最常用的） 

allkeys-random：从数据集中任意选择数据淘汰 

allkeys-lfu：当内存不足以容纳新写入数据时，在键空间中，移除最不经常使用的key  

no-eviction：禁止驱逐数据，也就是说当内存不足以容纳新写入数据时，新写入操作会报错。这个应该没人使用吧！  


### - 如果redis有热点key怎么解决

本地缓存, 拆分：eg: 按日期拆分、根据业务拆分

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


### - redis执行一条命令的过程

整体：  
1. 第一步是建立连接阶段，响应了socket的建立，并且创建了client对象；
2. 第二步是处理阶段，从socket读取数据到输入缓冲区，然后解析并获得命令，执行命令并将返回值存储到输出缓冲区中；
3. 第三步是数据返回阶段，将返回值从输出缓冲区写到socket中，返回给客户端，最后关闭client

![](https://ipic-1252327316.cos.ap-beijing.myqcloud.com/image/redis/22.png)

细节：  
1. 客户端向服务端发起建立 socket 连接的请求，那么监听套接字将产生 AE_READABLE 事件，触发连接应答处理器执行。处理器会对客户端的连接请求进行应答，然后创建客户端套接字，以及客户端状态，并将客户端套接字的 AE_READABLE 事件与命令请求处理器关联。
2. 客户端建立连接后，向服务器发送命令，那么客户端套接字将产生 AE_READABLE 事件，触发命令请求处理器执行，处理器读取客户端命令，然后传递给相关程序去执行。
3. 执行命令获得相应的命令回复，为了将命令回复传递给客户端，服务器将客户端套接字的 AE_WRITEABLE 事件与命令回复处理器关联。当客户端试图读取命令回复时，客户端套接字产生 AE_WRITEABLE 事件，触发命令回复处理器将命令回复全部写入到套接字中

![](https://ipic-1252327316.cos.ap-beijing.myqcloud.com/image/redis/33.png)

个人理解：  
1. 所有事件都是注册在一个aeEventLoop上，aeEventLoop里面是事件数组  
2. 事件监听的是socket的文件描述符，socket分server和client，连接事件监听serversocket, 建立client之后注册读事件监听clientsocket, 执行完之后注册写事件监听clientsocket  
3. 每个client有个输入输出缓冲区方便数据就绪
关键点就是他娘的用了IO多路复用，提升执行效率  

### - select epoll
select 是轮询所有的文件描述符
epoll 是走回调，能支持的数量更多，效率更高


### - redis主从同步过程

![](https://ipic-1252327316.cos.ap-beijing.myqcloud.com/image/redis/11.png)

对于master
1. 首先slave向master发起sync同步命令，这一步在slave启动后触发，master被动的将新的salve加入到自己的主备复制集群。
2. master在收到sync后开启BGSAVE命令，BGSAVE是Redis的一种全量模式的持久化机制
3. BGSAVE完成后，master将快照信息发给salve。
4. 发送期间，master收到来自客户端新的写命令，除了正常的相应之外，都存入一份back-log队列
5. 快照信息发送完成后，master继续发送backlog命令
6. backlog发送完成后，后续的写操作同时发送给salve，保持实时的异步复制

对于slave
1. 发送完sync命令后，继续对外提供服务。
2. 开始接收master的快照信息，此时，slave将现有数据清空，并将master快照写入内存
3. 接收backlog内容并执行，也就是回放，期间对外提供读请求
4. 继续接收后续来自master的命令副本，并继续回放，保持数据和master数据一致。

如果有多个slave节点并发发送sync命令给master，企图建立主备关系，只要第二个salve的sync命令发生在master完成BGSAVE之前，第二个salve将收到和第一个salve相同的快照和后续backlog，否则第二个salve的sync将触发master的第二次BGSAVE

断点续传 (部分重同步)
Redis支持PSYNC用于替代SYNC，做到基于断点续传的主备同步协议，master和slave两端通过维护一个offset记录当前已经同步过的命令，slave断开期间，master的客户端命令保存在缓存中，salve重连之后，告知master断开时的最新offset，master则将缓存中大于offset的数据发送给slave。

### - redis主从 故障转移 failover sentinel Raft协议

failover决策  
当一台master宕机后，要发起failover流程。其中只有一个sentinel节点可以作为failover的发起者，让他作为leader主导选举。
Redis的sentinel leader选举机制采用类似raft协议实现这个选举算法。

1. sentinelState的epoch变量类似于raft协议中的term（选举回合）
2. 每一个确认了master“客观不可用”的sentinel节点都会向周围广播自己的参选请求
3. 每一个收到参选请求的sentinel节点如果还没人向他发送过参选请求，它就将本回合的意向置为第一个参选sentinel并回复他，如果本回合内回复过意向，那么拒绝所有参选请求，并将已有意向回复给参选的的sentinel
4. 每个发送参选请求的sentinel节点，如果收到了超过一半的意向同意某个的sentinel参选（可能是本人），那么将成为leader，如果回合内进行了长时间还没有选出leader，那么进行下一个回合。

leader sentinel确定了之后，从所有的slave中依据一定的规则选一个新的master，并告知其他slave连接这个新的master。
新master的选举规则
1. 过滤不健康（主观下线，断线），5秒内没有回复过sentinel节点ping相应，与主节点失联超过
down-after-millisecond*10 时间的
2. 选择slave-priority最高的从节点列表，如果存在返回，否则继续
3. 选择复制偏移量最大的从节点，
4. 选择runid最小的从节点

如果多个回合还没有选出leader sentinel，怎么办？
选举过程非常快，一般谁先判断出主观下线谁就是leader