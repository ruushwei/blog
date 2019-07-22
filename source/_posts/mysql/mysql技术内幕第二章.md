---
title: mysql技术内幕第二章、Innodb存储引擎
date: 2018-05-22T22:54:24+02:00
tags: 
- mysql
- innodb
categories: mysql
---



### Innodb

#### Innodb体系结构



![存储引擎结构图](https://ws4.sinaimg.cn/large/006tKfTcgy1frt9cjky75j30v90ldn3d.jpg) 



InnoDB 存储引擎有多个内存块,这些内存块组成了一个大的内存池,主要负责如下工作:

- 维护所有进程/线程需要访问的多个内部数据结构
- 缓存磁盘上的数据, 方便快速读取, 同时在对磁盘文件修改之前进行缓存
- 重做日志(redo log)缓冲

<!--more-->

![innodb体系架构](https://ws4.sinaimg.cn/large/006tKfTcgy1frl2gc2w2tj30m80gs0u1.jpg)



后台线程的主要作用是负责刷新内存池中的数据,保证缓冲池中的内存缓存的是最新数据;将已修改数据文件刷新到磁盘文件;保证数据库发生异常时 InnoDB 能恢复到正常运行的状态.



#### 后台线程



InnoDB 使用的是多线程模型, 其后台有多个不同的线程负责处理不同的任务

1. Master Thread

这是最核心的一个线程,主要负责将缓冲池中的数据异步刷新到磁盘,保证数据的一致性,包括赃页的刷新、合并插入缓冲、UNDO 页的回收等.

2. IO Thread

在 InnoDB 存储引擎中大量使用了异步 IO 来处理写 IO 请求, IO Thread 的工作主要是负责这些 IO 请求的回调.

可以通过命令来观察 InnoDB 中的 IO Thread:

```
mysql> SHOW ENGINE INNODB STATUS\G
*************************** 1. row ***************************
  Type: InnoDB
  Name:
Status:
=====================================
2016-03-23 20:19:53 0x700000d51000 INNODB MONITOR OUTPUT
....
...
--------
FILE I/O
--------
I/O thread 0 state: waiting for i/o request (insert buffer thread)
I/O thread 1 state: waiting for i/o request (log thread)
I/O thread 2 state: waiting for i/o request (read thread)
I/O thread 3 state: waiting for i/o request (read thread)
I/O thread 4 state: waiting for i/o request (read thread)
I/O thread 5 state: waiting for i/o request (read thread)
I/O thread 6 state: waiting for i/o request (write thread)
I/O thread 7 state: waiting for i/o request (write thread)
I/O thread 8 state: waiting for i/o request (write thread)
I/O thread 9 state: waiting for i/o request (write thread)
......
----------------------------
END OF INNODB MONITOR OUTPUT
```

可以看到, InnoDB 共有10个 IO Thread, 分别是 4个 write、4个 read、1个 insert buffer和1个 log thread.

3. Perge Thread

事物被提交之后, undo log 可能不再需要,因此需要 Purge Thread 来回收已经使用比分配的 undo页. InnoDB 支持多个 Purge Thread, 这样做可以加快 undo 页的回收
InnoDB 引擎默认设置为4个 Purge Thread:

```
mysql> SHOW VARIABLES LIKE "innodb_purge_threads"\G
*************************** 1. row ***************************
Variable_name: innodb_purge_threads
        Value: 4
1 row in set (0.00 sec)
```

**undo日志用于存放数据修改被修改前的值.**



4. Page Cleaner Thread

Page Cleaner Thread 是新引入的,其作用是将之前版本中脏页的刷新操作都放入单独的线程中来完成,这样减轻了 Master Thread 的工作及对于用户查询线程的阻塞



<!--more-->

####  内存



1. 缓冲池

InnoDB 存储引擎是基于磁盘存储的,其中的记录按照页的方式进行管理,由于 CPU 速度和磁盘速度之间的鸿沟, InnoDB 引擎使用缓冲池技术来提高数据库的整体性能.

缓冲池简单来说就是一块内存区域.在数据库中进行读取页的操作,首先将从磁盘读到的页存放在缓冲池中,下一次读取相同的页时,首先判断该页是不是在缓冲池中,若在,称该页在缓冲池中被命中,直接读取该页.否则,读取磁盘上的页.

对于数据库中页的修改操作,首先修改在缓冲池中页,然后再以**一定的频率刷新**到磁盘,并不是每次页发生改变就刷新回磁盘.

缓冲池的大小直接影响数据库的整体性能,对于 InnoDB 存储引擎而言,缓冲池配置通过参数 `innodb_buffer_pool_size` 来设置. 

缓冲池中缓存的数据页类型有:索引页、数据页、 undo 页、插入缓冲、自适应哈希索引、 InnoDB 的锁信息、数据字典信息等.索引页和数据页占缓冲池的很大一部分.

下图显示 InnoDB 存储引擎总内存的结构情况.

![图片描述](https://segmentfault.com/img/bVtLPd)

2. 重做日志缓冲 redo log



InnoDB 存储引擎先将重做日志信息放入这个缓冲区,然后以一定频率将其刷新到重做日志文件.重做日志文件一般不需要设置得很大,因为在下列三种情况下重做日志缓冲中的内容会刷新到磁盘的重做日志文件中.

1. Master Thread 每一秒将重做日志缓冲刷新到重做日志文件
2. 每个事物提交时会将重做日志缓冲刷新到重做日志文件
3. 当重做日志缓冲剩余空间小于1/2时,重做日志缓冲刷新到重做日志文件



**redo log:  当数据库对数据做修改的时候，需要把数据页从磁盘读到buffer pool中，然后在buffer pool中进行修改，那么这个时候buffer pool中的数据页就与磁盘上的数据页内容不一致，称buffer pool的数据页为dirty page 脏数据，如果这个时候发生非正常的DB服务重启，那么这些数据还没在内存，并没有同步到磁盘文件中（注意，同步到磁盘文件是个随机IO），也就是会发生数据丢失，如果这个时候，能够在有一个文件，当buffer pool 中的data page变更结束后，把相应修改记录记录到这个文件（注意，记录日志是顺序IO），那么当DB服务发生crash的情况，恢复DB的时候，也可以根据这个文件的记录内容，重新应用到磁盘文件，数据保持一致.**



3. 额外的内存池

在 InnoDB 存储引擎中, 对一些数据结构本身的内存进行分配时,需要从额外的内存池中进行申请.例如,分配了缓冲池,但是每个缓冲池中的帧缓冲还有对应的缓冲控制对象,这些对象记录以一些诸如 **LRU, 锁,等待**等信息,而这个对象的内存需要从额外的内存池中申请.



#### 2、Checkpoint技术

如果 redo log 可以无限地增大，同时缓冲池也足够大，是不是就意味着可以不将缓冲池中的脏页刷新回磁盘上？宕机时，完全可以通过 redo log 来恢复整个数据库系统中的数据。

显然，上述的前提条件是不满足的，这也就引入了 checkpoint 技术



我理解就是将脏页定时不断的刷新到磁盘，或触发它刷新到磁盘，让他下回崩溃时恢复的时候，可以通过读更少的redolog来恢复数据库系统。

在InnoDB存储引擎内部，有两种Checkpoint，分别为：Sharp Checkpoint、Fuzzy Checkpoint

Sharp Checkpoint 发生在数据库关闭时将所有的脏页都刷新回磁盘，这是默认的工作方式，即参数innodb_fast_shutdown=1。但是若数据库在运行时也使用Sharp Checkpoint，那么数据库的可用性就会受到很大的影响。故在InnoDB存储引擎内部使用Fuzzy Checkpoint进行页的刷新，即只刷新一部分脏页，而不是刷新所有的脏页回磁盘。

**Fuzzy Checkpoint**：1、Master Thread Checkpoint；2、FLUSH_LRU_LIST Checkpoint；3、Async/Sync Flush Checkpoint；4、Dirty Page too much Checkpoint



**1、Master Thread Checkpoint** 

以每秒或每十秒的速度从缓冲池的脏页列表中刷新一定比例的页回磁盘，这个过程是异步的，此时InnoDB存储引擎可以进行其他的操作，用户查询线程不会阻塞。

**2、FLUSH_LRU_LIST Checkpoint**

因为InnoDB存储引擎需要保证LRU列表中需要有差不多100个空闲页可供使用。

**3、Async/Sync Flush Checkpoint**

指的是重做日志文件不可用的情况，这时需要强制将一些页刷新回磁盘，而此时脏页是从脏页列表中选取的。

**4、Dirty Page too much**

即脏页的数量太多，导致InnoDB存储引擎强制进行Checkpoint。





#### 3、Master Thread



工作原理

```
void master_thread(){
    goto loop;
loop:
for (int i=0;i<10;i++){
    thread_sleep(1) //sleep 1 second-->每秒执行操作(负载在情况下会延迟)
    do log buffer flush to disk  //重做日志缓冲刷新到磁盘，即使这个事务没有提交(总是)
    if ( last_ten_second_ios < 5% innodb_io_capacity) //如果当前的10次数小于(5% * 200=10)(innodb_io_capacity默认值是200)
        do merger 5% innodb_io_capacity insert buffer //执行10个合并插入缓冲的操作(5% * 200=10)
    if ( buf_get_modified_ratio_pct > innodb_max_dirty_pages_pct ) //如果缓冲池中的脏页比例大于innodb_max_dirty_pages_pct(默认是75时)
        do buffer pool plush 100% innodb_io_capacity dirty page //刷新200个脏页到磁盘
    else if enable adaptive flush  //如果开户了自适应刷新
        do buffer pool flush desired amount dirty page //通过判断产生redo log的速度决定最合适的刷新脏页的数量
    if ( no user activity ) //如果当前没有用户活动
        goto backgroud loop  //跳到后台循环
}

//每10秒执行的操作
if ( last_ten_second_ios < innodb_io_capacity)  //如果过去10内磁盘IO次数小于设置的innodb_io_capacity的值（默认是200）
    do buffer pool flush 100% innodb_io_capacity dirty page //刷新脏页的数量为innodb_io_capacity的值（默认是200）
do merger 5% innodb_io_capacity insert buffer  //合并插入缓冲是innodb_io_capacity的5%（10）（总是）
do log buffer flush to disk                    //重做日志缓冲刷新到磁盘，即使这个事务没有提交（总是）
do full purge       //删除无用的undo页 （总是）
if (buf_get_modified_ratio_pct > 70%)          //如果缓冲池中的胜页比例大于70%
    do buffer pool flush 100% innodb_io_capacity dirty page  //刷新200个脏页到磁盘
else
    do buffer pool flush 10% innodb_io_capacity dirty page   //否则刷新20个脏页到磁盘
goto loop
backgroud loop:   //后台循环
do full purge     //删除无用的undo页 （总是）
do merger 5% innodb_io_capacity insert buffer  //合并插入缓冲是innodb_io_capacity的5%（10）（总是）
if not idle:      //如果不空闲，就跳回主循环，如果空闲就跳入flush loop
goto loop:    //跳到主循环
else:
    goto flush loop
flush loop:  //刷新循环
do buf_get_modified_ratio_pct pool flush 100% innodb_io_capacity dirty page //刷新200个脏页到磁盘
if ( buf_get_modified_ratio_pct > innodb_max_dirty_pages_pct ) //如果缓冲池中的脏页比例大于innodb_max_dirty_pages_pct的值（默认75%）
    goto flush loop            //跳到刷新循环，不断刷新脏页，直到符合条件
    goto suspend loop          //完成刷新脏页的任务后，跳入suspend loop
suspend loop:
suspend_thread()               //master线程挂起，等待事件发生
waiting event
goto loop;
}
```





#### 4、Innodb 关键特性



##### (1)、insert buffer



insert buffer是一种特殊的数据结构（B+ tree）**并不是缓存的一部分，而是物理页**，当受影响的索引页不在buffer pool时缓存 [secondary index](https://dev.mysql.com/doc/refman/5.5/en/glossary.html#glos_secondary_index) pages的变化，当buffer page读入buffer pool时，进行合并操作，这些操作可以是 [`INSERT`](https://dev.mysql.com/doc/refman/5.5/en/insert.html), [`UPDATE`](https://dev.mysql.com/doc/refman/5.5/en/update.html), or [`DELETE`](https://dev.mysql.com/doc/refman/5.5/en/delete.html) operations (DML)

最开始的时候只能是insert操作，所以叫做insert buffer，现在已经改叫做change buffer了

insert buffer 只适用于 non-unique secondary indexes 也就是说**只能用在非唯一的索引上**，原因如下

1、primary key 是按照递增的顺序进行插入的，异常插入聚族索引一般也顺序的，非随机IO

2 写唯一索引要检查记录是不是存在，所以在修改唯一索引之前,必须把修改的记录相关的索引页读出来才知道是不是唯一、这样Insert buffer就没意义了，要读出来(随机IO)

所以只对非唯一索引有效

对于为非唯一索引，辅助索引的修改操作并非实时更新索引的叶子页，而是把若干对同一页面的更新缓存起来做，合并为一次性更新操 作，**减少IO，转随机IO为顺序IO,这样可以避免随机IO带来性能损耗**，提高数据库的写性能

**具体流程**

先判断要更新的这一页在不在缓冲池中

a、若在，则直接插入；

b、若不在，则将index page 存入Insert Buffer，按照Master Thread的调度规则来合并非唯一索引和索引页中的叶子结点



##### (2)、double write

**double write工作流程如下：**

当一系列机制（main函数触发、checkpoint等）触发数据缓冲池中的脏页进行刷新到data file的时候，并不直接写磁盘，而是会通过memcpy函数将脏页先复制到内存中的double write buffer，之后通过double write buffer再分两次、每次1MB顺序写入共享表空间的物理磁盘上。然后马上调用fsync函数，同步脏页进磁盘上。由于在这个过程中，double write页的存储时连续的，因此写入磁盘为顺序写，性能很高；完成double write后，再将脏页写入实际的各个表空间文件，这时写入就是离散的了。各模块协作情况如下图（第一步应为脏页产生的redo记录log buffer，然后log buffer写入redo log file，为简化次要步骤直接连线表示）：

**double write工作流程如下：**

当一系列机制（main函数触发、checkpoint等）触发数据缓冲池中的脏页进行刷新到data file的时候，并不直接写磁盘，而是会通过memcpy函数将脏页先复制到内存中的double write buffer，之后通过double write buffer再分两次、每次1MB顺序写入共享表空间的物理磁盘上。然后马上调用fsync函数，同步脏页进磁盘上。由于在这个过程中，double write页的存储时连续的，因此写入磁盘为顺序写，性能很高；完成double write后，再将脏页写入实际的各个表空间文件，这时写入就是离散的了。各模块协作情况如下图（第一步应为脏页产生的redo记录log buffer，然后log buffer写入redo log file，为简化次要步骤直接连线表示）：

![dbw](https://ws4.sinaimg.cn/large/006tKfTcgy1frt9ci2lxoj30i40b3abb.jpg)



**double write在恢复的时候是如何工作的？**

如果是写double write buffer本身失败，那么这些数据不会被写到磁盘，InnoDB此时会从磁盘载入原始的数据，然后通过InnoDB的事务日志来计算出正确的数据，重新写入到double write buffer。

如果double write buffer写成功的话，但是写磁盘失败，InnoDB就不用通过事务日志来计算了，而是直接用buffer的数据再写一遍。如上图中显示，在恢复的时候，InnoDB直接比较页面的checksum，如果不对的话，Innodb存储引擎可以从共享表空间的double write中找到该页的一个最近的副本，将其复制到表空间文件，再应用redo log，就完成了恢复过程。因为有副本所以也不担心表空间中数据页是否损坏，但InnoDB的恢复通常需要较长的时间



##### (3)、自适应哈希索引



InnoDB存储引擎会监控对表上各索引页的查询。如果观察到建立哈希索引可以带来速度提升，则建立哈希索引，称之为自适应哈希索引(Adaptive Hash Index, AHI)。AHI是通过缓冲池的B+树页构造而来，因此建立的速度很快，而且不需要对整张表构建哈希索引。InnoDB存储引擎会自动根据访问的频率和模式来自动地为某些热点页建立哈希索引。

AHI有一个要求，对这个页的连续访问模式必须是一样的。例如对于(a,b)这样的联合索引页，其访问模式可以是下面情况： 
1）where a=xxx 
2）where a =xxx and b=xxx 
访问模式一样是指查询的条件是一样的，若交替进行上述两种查询，那么InnoDB存储引擎不会对该页构造AHI。 
AHI还有下面几个要求： 
1)以该模式访问了100次 
2)页通过该模式访问了N次，其中N=页中记录*1/16

InnoDB存储引擎官方文档显示，启用AHI后，读取和写入速度可以提高2倍，辅助索引的连接操作性能可以提高5倍。AHI的设计思想是数据库自优化，不需要DBA对数据库进行手动调整。



```sql
mysql> show variables like 'innodb_adaptive_hash_index';
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| innodb_adaptive_hash_index | ON    |
+----------------------------+-------+
1 row in set (0.00 sec)
```





##### (4)、异步IO

与AIO对应的Sync IO，即每进行一次IO操作，需要等待此操作结束才能继续接下来的操作。但是如果用户发出的是一条索引扫描的查询，那么这条SQL查询语句可能需要扫描多个索引页，也就是需要进行多次的IO操作。在每扫描一个页并等待其完成后再进行下一次的扫描，这是没有必要的。用户可以在发出一个IO请求后立即再发出另一个IO请求，当全部IO请求发送完毕后，等待所有IO操作的完成，这就是AIO。

```Sql
mysql> show variables like 'innodb_use_native_aio';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_use_native_aio | ON    |
+-----------------------+-------+
1 row in set (0.00 sec)
```





##### (5)、刷新临接页



AIO另一个优势是进行IO Merge操作，也就是将多个IO合并为1个IO,这样可以提高IOPS的性能。

例如用户需要访问页的（space, offset）为： 
(8,6),(8,7),(8,8) 
每个页的大小为16KB，那么同步IO需要进行3次IO操作。而AIO会判断到这三个页是连续的（可以通过(space,offset)知道）。因此AIO底层会发送一个IO请求，从(8,6)开始，读取48KB的页。



```Sql
mysql> show variables like 'innodb_flush_neighbors';
+------------------------+-------+
| Variable_name          | Value |
+------------------------+-------+
| innodb_flush_neighbors | 1     |
+------------------------+-------+
1 row in set (0.00 sec)
```



若是固态硬盘，有超高的IOPS，建议设置为0.



#### 5、启动、关闭、恢复



InnoDB存储引擎是MySQL的存储引擎之一，因此InnoDB存储引擎的启动和关闭更准确地是指在MySQL实例的启动过程中对InnoDB表存储引擎的处理过程。



**参数innodb_fast_shutdown**

在关闭时，参数innodb_fast_shutdown影响着表的存储引擎为InnoDB的行为。该参数可取值为0、1、2。

1. 0代表当MySQL关闭时，InnoDB需要完成所有的full purge和merge insert buffer操作，这会需要一些时间，有时甚至需要几个小时来完成。如果在做InnoDB plugin升级，通常需要将这个参数调为0，然后再关闭数据库。
2. 1是该参数的默认值，表示不需要完成上述的full purge和merge insert buffer操作，但是在缓冲池的一些数据脏页还是会刷新到磁盘。
3. 2表示不完成full purge和merge insert buffer操作，也不将缓冲池中的数据脏页写回磁盘，而是将日志都写入日志文件。这样不会有任何事务会丢失，但是MySQL数据库下次启动时，会执行恢复操作（recovery）。

当正常关闭MySQL数据库时，下一次启动应该会很正常。但是，如果没有正常地关闭数据库，如用kill命令关闭数据库，在MySQL数据库运行过程中重启了服务器，或者在关闭数据库时将参数innodb_fast_shutdown设为了2，MySQL数据库下次启动时都会对InnoDB存储引擎的表执行恢复操作。



**参数innodb_force_recovery**

参数innodb_force_recovery影响了整个InnoDB存储引擎的恢复状况。该值默认为0，表示当需要恢复时执行所有的恢复操作。当不能进行有效恢复时，如数据页发生了corruption，MySQL数据库可能会宕机，并把错误写入错误日志中。

但是，在某些情况下，我们可能并不需要执行完整的恢复操作，我们自己知道如何进行恢复。比如正在对一个表执行alter table操作，这时意外发生了，数据库重启时会对InnoDB表执行回滚操作。对于一个大表，这需要很长时间，甚至可能是几个小时。这时我们可以自行进行恢复，例如可以把表删除，从备份中重新将数据导入表中，这些操作的速度可能要远远快于回滚操作。

innodb_force_recovery还可以设置为6个非零值：1～6。大的数字包含了前面所有小数字的影响，具体情况如下。

- 1（SRV_FORCE_IGNORE_CORRUPT）：忽略检查到的corrupt页。
- 2（SRV_FORCE_NO_BACKGROUND）：阻止主线程的运行，如主线程需要执行full purge操作，会导致crash。
- 3（SRV_FORCE_NO_TRX_UNDO）：不执行事务回滚操作。
- 4（SRV_FORCE_NO_IBUF_MERGE）：不执行插入缓冲的合并操作。
- 5（SRV_FORCE_NO_UNDO_LOG_SCAN）：不查看撤销日志（Undo Log），InnoDB存储引擎会将未提交的事务视为已提交。
- 6（SRV_FORCE_NO_LOG_REDO）：不执行前滚的操作。

需要注意的是，当设置参数innodb_force_recovery大于0后，可以对表进行select、create、drop操作，但insert、update或者delete这类操作是不允许的。



参考链接：

https://segmentfault.com/a/1190000004673132