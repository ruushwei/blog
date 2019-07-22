---
title: zookeeper分享总结
date: 2017-07-19T12:54:24+02:00
tags:
- zookeeper
categories: zookeeper
---



给公司组内成员做的zookeeper的分享，研究了两三个星期，包含zookeeper的各个方面，由浅入深，没有探讨一致性算法，但对zookeeper的理解和使用、常用应用做了详细的整理和总结。



一.  定义

**官方定义：**

Zookeeper是分布式系统的**高性能协调服务**。

它将公共服务：像 **命名、配置管理、同步、**和 **群组服务** 暴露在一个简单的接口里。

你可以使用现成的Zookeeper去实现 **共识、群组管理、领导人选举**和 **presence protocols（存在，出席协议）**。

你没有必要从头开始实现它们，并且可以在它的基础之上建立自己的特定的需求。



**共识：**

就是在一个进程提出了一个值应当是什么后，每个进程都提出自己的提议，最终通过共识算法，使所有进程对这个值达成一致意见。



**自己理解**

zookeeper是  znode组成的文件系统 + watcher监听器 + 一致性协议zab协议



**为什么要使用zookeeper**：



如果应用程序自己去实现这些服务都不可避免的引入大量的工作。

zookeeper将协调服务与系统服务解耦，降低复杂度。

<!--more-->



二. znode

znode是一个跟**Unix文件系统**路径相似的节点，由一系列斜杠进行分割的路径表示。



![20160125174013_212](https://ws1.sinaimg.cn/large/006tNc79ly1fhm4nwafvwj30ca071q35.jpg)





node可以指代**主机、服务器、集群成员、客户端进程**等等。

注意：引用时**路径必须是绝对**的，必须由斜杠字符 ( / ) 来开头。



既**像文件一样维护着数据、元信息、ACL、时间戳等数据结构**，又**像目录一样可以作为路径标识的一部分**

每个Znode由3部分组成:

**①** stat：此为状态信息, 描述该Znode的版本, 权限等信息

**②** data：与该Znode关联的数据

**③** children：该Znode下的子节点



![301534569026625](https://ws4.sinaimg.cn/large/006tKfTcly1fhm4nx8smcj30g006imy4.jpg)





zookeeper是**存储在内存**中的，所以它非常快，但内存有限，所以它存储的数据非常小，它存的是一些**管理调度数据**， 通常就是配置信息。



ZooKeeper的服务器和客户端都**严格限制Znode的数据大小至多1M**，但常规使用中应该远小于此值。



**zookeeper提供的操作均是原子操作**。操作不可分割，读写都是一次性操作节点全部的数据，读就是一下读全部，写就是一下写全部。



**znode节点类型**

节点类型分为**持久节点**（PERSISTENT ）、**临时节点（**EPHEMERAL），以及**时序节点**（SEQUENTIAL ）,一般是组合使用，可以生成以下4种节点类型



**持久节点（PERSISTENT）**

**创建后，就一直存在**，直到有删除操作来主动清除这个节点.



**持久顺序节点（PERSISTENT_SEQUENTIAL ）**

在创建子节点的时候，可以设置这个属性，那么在创建节点过程中，ZK会**自动为给定节点名加上一个数字后缀**，作为新的节点名。这个数字后缀的上限是整型的最大值。



**临时节点（EPHEMERAL ）**

生命周期和客户端会话绑定, **会话失效，而非连接断开**。另外，在 **临时节点下面不能创建子节点**



**临时顺序节点（EPHEMERAL_SEQUENTIAL）**       



 三.  客户端调用



   Java

![api](https://ws2.sinaimg.cn/large/006tNc79ly1fhm4nwttd0j30na0fhq7y.jpg)





```
// 创建一个与服务器的连接
ZooKeeper zk = new ZooKeeper("localhost:" + CLIENT_PORT,
       ClientBase.CONNECTION_TIMEOUT, new Watcher() {
           // 监控所有被触发的事件
           public void process(WatchedEvent event) {
               System.out.println("已经触发了" + event.getType() + "事件！");
           }
       });

// 创建一个目录节点
zk.create("/testRootPath", "testRootData".getBytes(), Ids.OPEN_ACL_UNSAFE,
  CreateMode.PERSISTENT);

// 创建一个子目录节点
zk.create("/testRootPath/testChildPathOne", "testChildDataOne".getBytes(),
  Ids.OPEN_ACL_UNSAFE,CreateMode.PERSISTENT);
System.out.println(new String(zk.getData("/testRootPath",false,null)));

// 取出子目录节点列表
System.out.println(zk.getChildren("/testRootPath",true));

// 修改子目录节点数据
zk.setData("/testRootPath/testChildPathOne","modifyChildDataOne".getBytes(),-1);
System.out.println("目录节点状态：["+zk.exists("/testRootPath",true)+"]");

// 创建另外一个子目录节点
zk.create("/testRootPath/testChildPathTwo", "testChildDataTwo".getBytes(),
  Ids.OPEN_ACL_UNSAFE,CreateMode.PERSISTENT);
System.out.println(new String(zk.getData("/testRootPath/testChildPathTwo",true,null)));

// 删除子目录节点
zk.delete("/testRootPath/testChildPathTwo",-1);
zk.delete("/testRootPath/testChildPathOne",-1);

// 删除父目录节点
zk.delete("/testRootPath",-1);

// 关闭连接
zk.close();
```





 命令行



```
[zk: localhost:2181(CONNECTED) 15] help             
ZooKeeper -server host:port cmd args                
	stat path [watch]                               //  exits
	set path data [version]                         //  setData
	ls path [watch]                                 //  getchildren
	delquota [-n|-b] path
	ls2 path [watch]                                //  exits + getchildren
	setAcl path acl
	setquota -n|-b val path
	history                    
	redo cmdno
	printwatches on|off
	delete path [version]                           //   delete
	sync path
	listquota path
	rmr path
	get path [watch]                                //   getData
	create [-s] [-e] path data acl
	addauth scheme auth
	quit
	getAcl path
	close
	connect host:port
```







四.  watch触发器



**监视器** ,当节点状态发生改变时将会触发watch所对应的操作。当watch被触发时，ZooKeeper将会向客户端发送且仅发送一条通知。



![301534579188980](https://ws4.sinaimg.cn/large/006tKfTcly1fhm4nxps0sj30pp04b0tq.jpg)





![20160125174013_212](https://ws1.sinaimg.cn/large/006tNc79ly1fhm4nwafvwj30ca071q35.jpg)







五.  服务端



Zookeeper有三种模式，**单机模式**、**伪集群模式**和**集群模式**。

■ 单机模式：Zookeeper只运行在一台服务器上，适合测试环境；
■ 伪集群模式：就是在一台物理机上运行多个Zookeeper 实例；
■ 集群模式：Zookeeper运行于一个集群上，适合生产环境，这个计算机集群被称为一个“集合体”（ensemble）



**配置伪集群**

1. 官网下载tar包

2. **解压**：sudo tar -zxvf zookeeper-3.4.10.tar.gz

   **重命名**：mv zookeeper-3.4.10 zk

   **配置文件**：

   每启动一个服务都需要各自的  .cfg配置文件  一个data文件夹，data文件夹下myid文件

3. .cfg配置文件

   conf 下根据自带的zoo_sample.cfg 创建 配置文件zoo1.cfg、zoo2.cfg、zoo3.cfg

   zoo1.cfg

```
The number of milliseconds of each tick
tickTime=2000
The number of ticks that the initial
synchronization phase can take
initLimit=10
The number of ticks that can pass between
syncLimit=5
the directory where the snapshot is stored.
dataDir=/usr/local/zk/data1                        //zoo2.cfg, zoo3.cfg需改data目录
the port at which the clients will connect
clientPort=2181                                    //zoo2.cfg, zoo3.cfg需改端口

server.1=localhost:2287:3387
server.2=localhost:2288:3388
server.3=localhost:2289:3389
```

zoo2.cfg、zoo3.cfg 与zoo1.cfg差不多

参数说明：  

```

tickTime：这个时间是作为 Zookeeper 服务器之间、客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。
dataDir：顾名思义就是 Zookeeper 保存数据的目录，默认情况下，Zookeeper 将写数据的日志文件也保存在这个目录里。
clientPort：这个端口就是客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。
initLimit：这个配置项是用来配置 Follower 服务器初始化连接leader时最长能忍受多少个心跳时间间隔数。当已经超过 10 个心跳的时间（也就是 tickTime）长度后 Zookeeper 服务器还没有收到客户端的返回信息，那么表明这个客户端连接失败。总的时间长度就是 5*2000=10 秒
syncLimit：这个配置项标识 Leader 与 Follower 之间请求和应答时间长度，最长不能超过多少个 tickTime 的时间长度，总的时间长度就是 2*2000=4 秒
server.A=B：C：D：其中 A 是一个数字，表示这个是第几号服务器；B 是这个服务器的 ip 地址；
  C 表示的是这个服务器与集群中的 Leader 服务器交换信息的端口 (leader开启这个端口的监听，与follower交换信息 ) ；
  D 表示的是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的 Leader，而这个端口就是用来执行选举时服务器相互通信的端口。
```

​       data文件夹、myid文件

4. myid 位于data文件夹下，data文件夹的位置在 .cfg文件中配置，如上。

​       myid 文件中为server.id对应id



5. start status stop

```
./zhServer.sh start conf/zoo.cfg

./zhServer.sh status conf/zoo.cfg

./zhServer.sh stop conf/zoo.cfg
```

六.  zookeeper的功能

zookeeper除了在znode上存储些配置数据还能做什么。



命名服务

客户端根据指定名字获取资源或服务地址，提供者等信息。被命名的实体通常可以是集群中的机器，提供的服务地址，远程对象等等。

较为常见的就是一些分布式服务框架中的服务地址列表。通过调用ZK提供的创建节点的API，能够很容易创建一个全局唯一的path，这个**path**就可以作为一个名称。



![20160125174013_212](https://ws1.sinaimg.cn/large/006tNc79ly1fhm4nwafvwj30ca071q35.jpg)



**举个例子：**

**Dubbo**中使用ZooKeeper来作为其命名服务，**维护全局的服务地址列表**

**服务提供者 启动的时候，向ZK上的指定节点/dubbo/${serviceName}/providers目录下写入自己的URL地址，这个操作就完成了服务的发布。 **

**服务消费者 启动的时候，订阅/dubbo/${serviceName}/providers目录下的提供者URL地址， 并向/dubbo/${serviceName} /consumers目录下写入自己的URL地址。 **



![dubbo](https://ws2.sinaimg.cn/large/006tNc79ly1fhqbvyduacj31eo0rw109.jpg)



数据发布与订阅（配置中心）



**发布者将数据发布到ZK节点上，供订阅者动态获取数据，实现配置信息的集中式管理和动态更新.**



**通常场景**

**应用来节点获取一次配置**，同时，**在节点上注册一个Watcher**，这样一来，**以后每次配置有更新的时候，都会实时通知到订阅的客户端，从来达到动态获取最新配置信息的目的**。 (就是有一个节点，大家一起读，一起改，都在这儿设置监视器)

注：**数据量很小，但是数据更新可能会比较快的场景。**



![配置中心](https://ws1.sinaimg.cn/large/006tNc79ly1fhqbvxmqn3j31kw0y4492.jpg)





**举三个例子**

**HBase**中，客户端就是连接一个Zookeeper，获得必要的HBase集群的配置信息，然后才可以进一步操作。

**Kafka**中，也使用Zookeeper来维护broker的信息。

**Dubbo**中也广泛的使用Zookeeper管理一些配置来实现服务治理。



负载均衡



分布式环境中，为了保证**高可用性**，通常同一个服务的 **提供方会部署多份**，达到对等服务。而消费者就须要在这些对等的服务器中**选择一个**来执行相关的业务逻辑。



Kafka 和 阿里开源的 metaq 都是通过zookeeper来做到 生产者、消费者的负载均衡



**举个例子kafka**



**生产者负载均衡：**

**生产者客户端 在发送消息须选择一台broker(就是一台生产者服务端server)上的一个分区来发送消息，**

**kafka 会把所有broker和对应分区信息全部注册到ZK指定节点上**

**生产者** 在通过ZK获取分区列表之后，会**按照brokerId和partition的顺序排列组织成一个有序的分区列表**，发送的时候按照从头到尾循环往复的方式选择一个分区来发送消息。



![负载均衡](https://ws1.sinaimg.cn/large/006tNc79ly1fhq9zlsdedj31ec0v8gu1.jpg)



**消费策略是**

group订阅topic，partition的增删对应group中消费者重新进行负载均衡。

一个partition对应一个consumer，group监听topic，此时新加一条消息的话，可能会新加一个topic中的partition索引，也可能会去掉一个topic中的partition索引。

新增partition的话，在group中找到一个consumer对应，然后在owners 和 offsets 添加分区和消费者对应的信息和消费偏移量的信息，此时讲道理消费者和生产者broker已经连接了。

如果不是新增partition，就是向partition中放消息的话，此时已经对应了consumer，就直接由那个consumer消费。

如果是删除了一个partion或broker， 对应topic下的索引删除，consumer不再消费对应partition，group会重新进行负载均衡。



分布式通知/协调



协调不同系统。

跟配置中心挺像，**配置中心是修改配置给所有人看，通知/协调是一个系统修改配置给另一个系统看**。

通过 **watcher注册与异步通知机制**，能够很好的实现分布式环境下**不同系统**之间的通知与协调，实现对数据变更的实时处理。

**使用方法通常是不同系统都对ZK上同一个znode进行注册，监听znode的变化（包括znode本身内容及子节点的），其中一个系统update了znode，那么另一个系统能够收到通知，并作出相应处理。**



![分布式通知协调](https://ws3.sinaimg.cn/large/006tNc79ly1fhqbvxxecij31kw0wbk13.jpg)



**举两个例子**

**调度系统**， 某系统有 **控制台** 和 **推送系统** 两部分组成，控制台的职责是控制推送系统进行相应的推送工作。管理人员在控制台作的一些操作，实际上是修改了ZK上某些节点的状态，而ZK就把这些变化通知给他们注册Watcher的客户端，即推送系统，于是，作出相应的推送任务.

**任务分发系统**，子任务启动后，到zk来注册一个临时节点，并且定时将自己的进度进行汇报（将进度写回这个临时节点），这样任务管理者就能够实时知道任务进度。



分布式锁



**这个锁是给一个分布式应用提供的**。

提供两类锁服务，一个是独占锁，另一个是时序锁。



**独占锁**就是从一堆客户端中选一个获得锁，只有一个获得。

**时序锁**是将这些客户端获取锁的顺序进行排序。



**独占锁通常的做法是把zk上的一个znode看作是一把锁，通过create znode的方式来实现。所有客户端都去创建 /distribute_lock 节点，最终成功创建的那个客户端也即拥有了这把锁。**

**时序锁是  /distribute_lock 已经预先存在，客户端在它下面创建临时有序节点（这个可以通过节点的属性控制：CreateMode.EPHEMERAL_SEQUENTIAL来指定）。Zk的父节点（/distribute_lock）维持一份sequence,保证子节点创建的时序性，从而也形成了每个客户端的全局时序**



分布式队列



两种，一种是常规的**先进先出队列**，另一种是要等到**队列成员聚齐之后的才统一按序执行。**

先进先出队列，和分布式锁服务中的控制时序场景基本原理一致。



第二种队列其实是在FIFO队列的基础上作了一个增强。

通常可以**在 /queue 这个znode下预先建立一个/queue/num 节点，并且赋值为n**（或者直接给/queue赋值n），表示队列大小，之后**每次有队列成员加入后，就判断下是否已经到达队列大小，决定是否可以开始执行了**。

这种用法的典型场景是，分布式环境中，**一个大任务Task A，需要在很多子任务完成**（或条件就绪）情况下才能进行。这个时候，凡是其中**一个子任务完成（就绪**），那么**就去 /taskList 下建立自己的临时时序节点**（CreateMode.EPHEMERAL_SEQUENTIAL），**当 /taskList 发现自己下面的子节点满足指定个数，就可以进行下一步按序进行处理**了。



集群管理与Master选举



利用ZooKeeper有两个特性，就可以实时另一种集群机器存活性监控系统：

1. 客户端在节点 x 上注册一个Watcher，那么如果 x?的子节点变化了，会通知该客户端。
2. 创建EPHEMERAL类型的节点，一旦客户端和服务器的会话结束或过期，那么该节点就会消失。



例如，监控系统在 /clusterServers 节点上注册一个Watcher，以后每动态加机器，那么就往 /clusterServers 下创建一个 EPHEMERAL类型的节点：/clusterServers/{hostname}. 这样，监控系统就能够实时知道机器的增减情况，至于后续处理就是监控系统的业务了。



Master选举则是zookeeper中最为经典的应用场景。

利用ZooKeeper的强一致性，能够保证在分布式高并发情况下节点创建的全局唯一性，即：同时有多个客户端请求创建 /currentMaster 节点，最终一定只有一个客户端请求能够创建成功。利用这个特性，就能很轻易的在分布式环境中进行集群选取了。

上文中提到，所有客户端创建请求，最终只有一个能够创建成功。在这里稍微变化下，就是允许所有请求都能够创建成功，但是得有个创建顺序，于是所有的请求最终在ZK上创建结果的一种可能情况是这样： /currentMaster/{sessionId}-1 ,?/currentMaster/{sessionId}-2 ,?/currentMaster/{sessionId}-3 ….. 每次选取序列号最小的那个机器作为Master，如果这个机器挂了，由于他创建的节点会马上小时，那么之后最小的那个机器就是Master了。





七.  一致性

* 顺序一致性

  一个客户端的更新将按照发送的顺序被写入到服务端。

* 并发一致性

  zookeeper 并不保证在某个时刻两个不同客户端具有一致的数据视图，因为网络的延迟一个客户端可能在另一个客户端得到修改通知之前进行更新。如果不同客户端读取到相同的值很重要，那么客户端应该在执行读取操作之前调用 sync() 方法，使得读操作的连接所连的 zookeeper 实例能与 leader 进行同步，从而能读到最新的内容。




 zab协议



**ZooKeeper Atomic Broadcast    ( ZooKeeper 原子消息广播协议 )**

**是zookeeper数据一致性的核心算法。**



功能实现

* 使用单一主进程来接收并处理客户端的所有事务请求，并采用原子广播协议，将服务器数据的状态变更以事务 Proposal（ 提议 ） 的形式广播到所有的副本进程上去。

* 保证一个全局的变更序列被顺序应用。

* 当前主进程出现异常情况的时候，依旧能够正常工作。

  



核心

​	所有事务请求必须由一个全局唯一的服务器来协调处理，这样的服务器被称为 Leader服务器，而余下的其他服务器为 Follower 服务器或Observer服务器。

​	Leader 服务器负责将一个客户端事务请求转换成一个事务proposal（提议），并将该 Proposal分发给集群中所有的Follower服务器。之后 Leader 服务器需要等待所有Follower 服务器的反馈,一旦超过半数的Follower服务器进行了正确的反馈后，那么 Leader 就会再次向所有的 Follower服务器分发Commit消息，要求其将前一个proposal进行提交。
