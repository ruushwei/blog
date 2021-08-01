---
title: MVCC-2021
date: 2021-08-01T10:54:24+02:00
tags: mysql
categories: mysql
---


### 总结
MVCC的核心实现主要基于两部分：多事务并发操作数据与一致性读实现。

RC的本质：每一条SELECT都可以看到其他已经提交的事务对数据的修改，只要事务提交，其结果都可见，与事务开始的先后顺序无关。
RR的本质：第一条SELECT生成ReadView前，已经提交的事务的修改可见。

### 多事务并发操作数据
多事务并发操作数据核心基于Undo log进行实现，Undo log可以用来做事务的回滚操作，保证事务的原子性。
同时可以用来构建数据修改之前的版本，支持多版本读。

InnoDB中，每一行记录都有两个隐藏列：DATA_TRX_ID和DATA_ROLL_PTR。(若没有主键，则还有一个隐藏主键)
DATA_TRX_ID：记录最近更新这条记录的事务ID(6字节)
DATA_ROLL_PTR：指向该行回滚段的指针，通过指针找到之前版本，通过链表形式组织(7字节)
DB_ROW_ID：行标识（隐藏单增ID），没有主键时主动生成(6字节)
当存在多个事务进行并发操作数据时，不同事务对同一行的更新操作产生多个版本，通过回滚指针将这些版本链接成一条Undo Log链。

操作过程如下：
1、将待操作的行加排他锁。
2、将该行原本的值拷贝到Undo Log中，DB_TRX_ID和DB_ROLL_PTR保持不变。（形成历史版本）
3、修改该行的值，更新该行的DATA_TRX_ID为当前操作事务的事务ID，将DATA_ROLL_PTR指向第二步拷贝到Undo Log链中的旧版本记录。（通过DB_ROLL_PTR可以找到历史记录）
4、记录Redo Log，包括Undo Log中的修改。
INSERT操作：产生新的记录，其DATA_TRX_ID为当前插入记录的事务ID。
DELETE操作：软删除，将DATA_TRX_ID记录下删除该记录的事务ID，真正删除操作在事务提交时完成。


### 一致性读实现
在InnoDB中，对于不同的事务隔离级别，一致性读实现均不相同，具体如下：
READ UNCOMMITED隔离级别：直接读取版本的最新记录。
SERIALIZABLE隔离级别：通过加锁互斥访问数据实现。
READ COMMITED和REPEATABLE READ隔离级别：使用版本链实现。(ReadView，可读性视图)
对于RC与RR隔离级别，实现一致性读都是通过ReadView，也就是今天的重点，什么是ReadView？

MVCC ReadView
ReadView是事务开启时，当前所有活跃事务（还未提交的事务）的一个集合，ReadView数据结构决定了不同事务隔离级别下，数据的可见性。

up_limit_id：最先开始的事务，该SQL启动时，当前事务链表中最小的事务id编号，也就是当前系统中创建最早但还未提交的事务
low_limit_id：最后开始的事务，该SQL启动时，当前事务链表中最大的事务id编号，也就是最近创建的除自身以外最大事务编号
m_ids：当前活跃事务ID列表，所有事务链表中事务的id集合
注：ID越小，事务开始的越早；ID越大，事务开始的越晚

1、下面所说的db_trx_id，是来自于数据行中的db_trx_id字段，并非开启了一个事务分配的ID，分配的事务ID只有操作了数据行，才会更新数据行中的db_trx_id字段
2、ReadView是与SQL绑定的，而并不是事务，所以即使在同一个事务中，每次SQL启动时构造的ReadView的up_trx_id和low_trx_id也都是不一样的
up_limit_id表示“低水位”，即当时活跃事务列表的最小事务id（最早创建的事务），如果读取出来的数据行上的的db_trx_id小于up_limit_id，则说明这条记录的最后修改在ReadView创建之前，因此这条记录可以被看见。

low_limit_id表示“高水位”，即当前活跃事务的最大id（最晚创建的事务），如果读取出来的数据行上的的db_trx_id大于low_limit_id，则说明这条记录的最后修改在ReadView创建之后，因此这条记录肯定不可以被看见。

如果读取出来的数据行上的的db_trx_id在low_limit_id和up_limit_id之间，则查找该数据上的db_trx_id是否在ReadView的m_ids列表中：

如果存在，则表示这条记录的最后修改是在ReadView创建之时，被另外一个活跃事务所修改，所以这条记录也不可以被看见。
如果不存在，则表示这条记录的最后修改在ReadView创建之前，所以可以看到。

REPEATABLE READ下的ReadView生成
每个事务首次执行SELECT语句时，会将当前系统所有活跃事务拷贝到一个列表中生成ReadView。

每个事务后续的SELECT操作复用其之前生成的ReadView。

UPDATE,DELETE,INSERT对一致性读snapshot无影响。

示例：事务A，B同时操作同一行数据

若事务A的第一个SELECT在事务B提交之前进行，则即使事务B修改记录后先于事务A进行提交，事务A后续的SELECT操作也无法读到事务B修改后的数据。
若事务A的第一个SELECT在事务B修改数据并提交事务之后，则事务A能读到事务B的修改。
针对RR隔离级别，在第一次创建ReadView后，这个ReadView就会一直持续到事务结束，也就是说在事务执行过程中，数据的可见性不会变，所以在事务内部不会出现不一致的情况。

READ COMMITED下的ReadView生成
每次SELECT执行，都会重新将当前系统中的所有活跃事务拷贝到一个列表中生成ReadView。

针对RC隔离级别，事务中的每个查询语句都单独构建一个ReadView，所以如果两个查询之间有事务提交了，两个查询读出来的结果就不一样。
