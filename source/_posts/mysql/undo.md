
---
title: undolog、redolog、binlog的用处
date: 2023-06-08T23:54:24+02:00
tags: mysql
categories: mysql
---

总结：
binlog 一致性。用于主从复制和数据恢复
redolog 保证持久性。log用于保证持久化，恢复在内存更新后，还没来得及刷到磁盘的数据
undolog 原子性。用于实现事务回滚和mvcc多版本并发控制

### binlog（逻辑日志）
一致性
定义：记录数据表结构和数据变更的二进制文件

### redolog（物理日志）
持久性
mysql不能每更新一条数据，就持久化到磁盘，因为会带来严重性能问题。所以是先更新到缓存，再特定时机下，再刷新到磁盘，为了防止内存数据丢失，会使用redolog作为事务日志，在内存更新完成后，写入redolog buffer，然后数据会写入到redolog file磁盘

redolog buffer写入磁盘的模式：
- 0（延迟写）每秒刷新写入磁盘，若系统崩溃，出现1s数据丢失
- 1（实时写）每次提交都会写入磁盘， 不会有数据丢失，但io性能差
- 2（实时写，延迟刷新）每次提交到os buffer, 然后每秒将os buffer日志写到磁盘

注：redolog的大小是固定的，可能被覆盖

### undolog（逻辑日志）
原子性
undolog存储数据的逻辑变化日志，
参考：https://blog.csdn.net/Huangjiazhen711/article/details/127900821

多版本并发控制mvcc，
MVCC在mysql中的实现依赖的是undo log与read view
undo log记录某行数据的多个版本的数据；read view用来判断当前版本数据的可见性。
read view可以用来判断可以看到哪些事务。根据他保存的创建时活跃事务id列表，创建时的最大事务id，创建时的最小事务id。

RC在每次语句执行，都会重新创建一份ReadView
RR下，事务开始创建ReadView，一直到事务结束

undo log的删除。当该undolog没有出现在其他事务的readview时，说明事务已提交，且没有其他事务依赖，innodb后台的清除purge线程会遍历删除undolog。聚簇索引中deleted标识为1的也会被删除。均为异步操作。


注：事务隔离性是通过mvcc+排他锁实现的