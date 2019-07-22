---
title: gfs分享
date: 2017-12-02T14:54:24+02:00
tags:
  - hadoop
categories:
  - hadoop
---





Google file system



### 问题



问题： 如何高效可靠地存储如此大规模的数据 ？

GFS是Google为其内部应用设计的分布式存储系统



问题的关键点是 **高效 、 可靠、规模巨大**

传统操作系统的问题在于  1. 硬盘不够大，存不了那么多  2. 数据不安全，传上去，硬盘瞬间损坏了，数据就没了

<!--more-->

方法：

对于硬盘不够大 (规模巨大)：

1. 多加几块硬盘， 最好就是无限加硬盘，让空间接近无限大，也就是可扩展、可伸缩

对于数据不安全 (可靠)：

1. 冗余，数据备份，默认是3份,  写好三份，再返回完成

高效：

1. 写时多server数据水平复制


2. 读时就近读取


3. 等



面向应用： 数据规模巨大，不要求低延迟

<!--more-->



### 架构



**组件: master, chunkserver, client**



![架构](https://ws4.sinaimg.cn/large/006tNc79ly1fmhbhjbqugj30k80f60un.jpg)



**Master：**

存储系统元数据信息，主要包括namespace、文件chunk信息以及chunk多副本位置信息。Master是系统的中心节点，所有客户端的元数据访问，如列举目录下文件，获取文件属性等操作都是直接访问Master。除此之外，还承担了系统诸多的管理工作



**Chunkserver:**

是文件chunk的存储位置。每个数据节点挂载多个磁盘设备并将其格式化为本地文件系统, 将客户端写入数据以Chunk为单位存储，存储形式为本地文件



**Client:**

提供类POSIX文件接口，应用程序使用客户端与GFS交互 （[POSIX](https://baike.baidu.com/item/POSIX)表示[可移植操作系统接口](https://baike.baidu.com/item/%E5%8F%AF%E7%A7%BB%E6%A4%8D%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E6%8E%A5%E5%8F%A3)（Portable Operating System Interface of UNIX，缩写为 POSIX ），POSIX标准定义了操作系统应该为应用程序提供的接口标准），就是说操作gfs和操作本地文件系统的接口没太大区别





### Chunk



GFS以64MB为Chunk大小来划分文件，每一个Chunk又以Block为单位进行划分，大小为64KB，每一个Block对应一个32位的校验和。当读取一个Chunk副本时，Chunk Server会将读取的数据和校验和进行比较，如果不匹配，就会返回错误，客户端将选择其它Chunk Server上的副本。





![chunkBlock](https://ws2.sinaimg.cn/large/006tNc79ly1fmgjes47duj30e4051wet.jpg)







![chunkServer](https://ws3.sinaimg.cn/large/006tNc79ly1fmgjeudzddj30l60sg771.jpg)







### Master

不进行文件数据读写

##### 文件元数据

1. 文件系统的目录树 ， 存储为B树， 叶子节点代表普通文件，中间节点则代表目录文件

2. 文件元数据信息 ，包括：id、文件大小、创建时间、文件的chunk信息等，Master会将这些元数据信息进行持久化存储， 每一次元数据的更新操作都会先写日志，再应用到内存的B树 ，Master启动时会将所有元数据加载至内存中， 优点是元数据操作速度很快，缺点是限制了文件系统的可扩展性，如64GB内存的服务器最多可支持的文件数量约为 64GB/64B = 10亿

3. 定期进行日志回收， 将当前内存状态冻结并持久化存储到磁盘上，称为Checkpoint；然后，回收该时刻之前的所有日志。若系统此时重启，只需要：1. 将Checkpoint加载入内存；2. 重放Checkpoint点之后的日志即可在内存中重构最新状态。

   

####

#####Chunk管理

文件元数据中存储了file id至chunk映射关系。Chunk以多副本方式存储，Master还需要维护Chunk位置信息。Chunk的位置信息ChunkServer周期性汇报，Master只在内存中维护该信息，无需持久化。如果master挂掉，新master启动的时候，会询问chunkserver，拿到各个chunk的位置信息



![master内容](https://ws4.sinaimg.cn/large/006tNc79ly1fmgjetsb33j30p10ezwfz.jpg)



注： Master授予一个副本租约，这个副本成为primary副本, 租约在修改操作的顺序上起到重要作用。



### 租约



保证多个副本变更顺序的一致性



租约（Lease）是由GFS中心节点Master分配给chunk的某个副本的锁。持有租约的副本方可处理客户端的更新请求，客户端更新数据前会从Master获取该chunk持有租约的副本并向该副本发送更新请求。

通过主副本将并发的更新操作串行化



租约本质上是一种有时间限制的锁：租约的持有者（chunk的某个副本）需要定期向Master申请续约。如果超过租约的期限，那么该租约会被强制收回并重新分配给其他副本。



注：读操作不涉及租约



### 读

![gfs架构](https://ws3.sinaimg.cn/large/006tNc79ly1fmgjev8r60j30k008ugm7.jpg)

块句柄（chunk handle）全局唯一的64bit标识

chunk index 是客户端计算出来的



### 写



![gfs写入](https://ws3.sinaimg.cn/large/006tNc79ly1fmgjeswkz5j3088082glr.jpg)





1. 客户端向Master查询待写入的chunk的副本信息，
2. Master返回副本列表，第一项为主副本，即当前持有租约的副本；
3. 客户端将数据发送至chunk多副本，chunkserver会缓存这些数据，此时数据并不落盘；
4. 客户端向主副本发起Sync请求；
5. 主副本将数据写入本地的同时通知其他副本将数据写入各自节点，此时数据方才落盘；
6. 主副本等待所有从副本的sync响应；
7. 主副本给客户端返回写入成功响应



- 数据推送过程，客户端可以根据网络拓扑情况进行推送路径优化：客户端可以选择距离自己最近的副本推送数据，然后再由该**副本**选择下一个距离自己最近的副本进行数据推送，直到数据被扩散至所有副本，由于该过程仅仅是将数据推送给chunkserver并保存在内存缓冲区中，因此，无需保证数据推送顺序；
- sync (5) 是将上面推送的数据落盘，需要保证多副本上数据写入序列的一致性, 该指令必须由主副本来确定数据更新顺序然后将该顺序通知给其他从副本。



主副本先执行sync操作，成功的话再为这个操作分配一个序列号，给其他副本发写入请求，如果不成功，则直接返回客户端失败。



主副本失败或部分副本失败会返回给客户端失败，客户端会进行重发



### 数据一致性



GFS定义了几种一致性：

> **defined**：状态已定义，从客户端角度来看，客户端完全了解已写入集群的数据，例如，客户端串行写入且成功，此时的状态是defined
> **consistent**：客户端来看chunk多副本的数据完全一致，但不一定defined，如下图2，一般发生在多客户端并发更新时
> **unconsistent**：多副本数据不一致
> **undefined**：数据未定义





#### 状态



![一致性](https://ws4.sinaimg.cn/large/006tNc79ly1fmha92ko1cj30at04nq37.jpg)



 上图其实从左到右，从上到下分别是  串行写成功、 并发写成功、追加成功(包含重试情况)、 各种失败



串行写只要成功就不会有一致性问题，因为offset是确定的，只要写成功各个副本就是一样的，只是并发的话单个客户端不知道真实的数据情况



#### write

**串行Over-Write**

over-write由客户端指定文件更新offset。当客户端是串行更新时，客户端自己知道写入文件范围以及写入数据内容，且本次写入在数据服务器的多副本上均执行成功， 即使第一次不成功，重试之后成功因为写入位置偏移固定，也没有并发客户端跟你抢，所以，结果对于客户端来说就是明确的，且多副本上数据一致，故而结果是defined。

![串行写](https://ws2.sinaimg.cn/large/006tNc79ly1fmha93ifiwj30be08wdfu.jpg)





**并行Over-Write**

并行写入时多个客户端由于写入范围可能交叉而形成交织写。这时候，由于单个客户端无法决定写入顺序（只有主副本才能决定谁先写谁后写），因此，即使写入成功，客户端仍无法确定在并发写入时交叉部分最终写入结果，但是因为写入成功，所以多副本数据必然一致



![并行写](https://ws4.sinaimg.cn/large/006tNc79ly1fmha90p2vjj308d08u0sr.jpg)



图中红色部分代表并发追加的部分，这部分数据由于无法确定谁先谁后执行，因此结果不确定。但由于更新成功，因此，副本间数据是一致的，这就是consistent but undefined。

无论是穿行还是并行over-write，一旦失败，多个chunk副本上的数据可能都不一致了，其次，客户端从不同的副本上读出的数据也不一样（可能某些副本成功而某些副本失败），因此，必然也是undefined，也是inconsistent。





#### append

客户端append操作无需指定offset，由chunk主副本根据当前文件大小决定写入offset，在写入成功后将该offset返回给客户端。因此，客户端能够根据offset确切知道写入结果，无论是串行写入还是并发写入，其行为是defined。如下：



![追加](https://ws3.sinaimg.cn/large/006tNc79ly1fmha91lwt2j30cv07o74d.jpg)



**append并重试**

假设上面的append经历了一次重试，那可能实际chunk的布局如下：

![追加并重试](https://ws3.sinaimg.cn/large/006tNc79ly1fmha8yscgij30d506rwek.jpg)



重试：由于第一次写失败（错误可能发生在任意一个副本），导致了多副本之间从50至80的数据可能不一致。但接下来重试成功，从80至110之间的数据一致，因此，其状态是interspersed with inconsistent。









### snapshot 快照



Snapshot是对系统当前状态进行的一次拍照。用户可以在任意时刻回滚到快照的状态。GFS使用COW技术实现Snapshot。

COW原理是如果被Snapshot的文件有更新操作时，就将文件的要被更新的chunk复制一份，然后对复制的chunk进行更新，而原来的chunk作为快照数据被保留，以后要恢复到该快照时，直接将该chunk读出即可。

当GFS的Master节点收到Snapshot请求时：

1. 回收Snapshot请求覆盖的文件chunks上的租约，这样，接下来客户端要对文件修改时，就必须向Master申请，而此时master就可以对chunk进行复制；
2. Master在日志中记录本次Snapshot操作，然后在内存中执行Snapshot动作，具体是将被Snapshot的文件或目录的元数据复制一份，被复制出的文件与原始文件指向相同的chunk；
3. 假如客户端申请更新被Snapshot的文件内容，那么找到需要更新的Chunk，向其多个副本发送拷贝命令，在其本地创建出Chunk的副本Chunk’，之所以本地创建是因为可以避免跨节点之间的数据拷贝，节省网络带宽；
4. 客户端收到Master的响应后，表示该Chunk已经COW结束，接下来客户端的更新流程与正常的没有区别。






![快照](https://ws3.sinaimg.cn/large/006tNc79ly1fmha94flmtj30k008agm8.jpg)





### Master容错



![架构](https://ws4.sinaimg.cn/large/006tNc79ly1fmhbhjbqugj30k80f60un.jpg)



GFS Master的修改操作总是先记录操作日志，然后再修改内存，当Master发生故障重启时，可以通过磁盘中的操作日志恢复内存数据结构；

为了减少Master宕机恢复时间，Master会定期将内存中的数据以checkpoint文件的形式转储到磁盘中，从而减少回放的日志量。为了进一步提高Master的可靠性和可用性，GFS中还会执行实时热备，所有的元数据修改操作都必须保证发送到实时热备才算成功。



远程的实时热备将实时接收Master发送的操作日志并在内存中回放这些元数据操作。如果Master宕机，还可以秒级切换到实时备机继续提供服务。为了保证同一时刻只有一个Master，GFS依赖Google内部的Chubby服务进行选主操作。

Master需要持久化前两种元数据，即命令空间及文件到chunk之间的映射关系；对于第三种元数据，即Chunk副本的位置信息，Master可以选择不进行持久化，这是因为ChunkServer维护了这些信息，即使Master发生故障，也可以在重启时通过ChunkServer汇报来获取。





### 相关链接

<http://www.uml.org.cn/zjjs/201202172.asp>

<http://blog.luoyuanhang.com/2017/05/15/gfs-reading-notes/>

<https://www.youtube.com/watch?v=WLad7CCexo8>

<https://zhuanlan.zhihu.com/p/28155582>

<http://blog.csdn.net/zhangpan19910604/article/details/51271943>
