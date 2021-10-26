---
title: Innodb 锁
date: 2021-09-02T10:54:24+02:00
tags: 
- mysql
- 锁
- 索引
categories: mysql
---

#### Innodb支持哪些锁
按兼容性分为 共享锁（Shared Locks，S锁）、排他锁（Exclusive Locks，X锁）
按锁范围分为：表级锁、行级锁
表锁：表共享读锁（Table Read Lock）、表排他写锁（Table Write Lock)、意向共享锁（intention shared lock, IS锁）、意向排他锁（intention exclusive lock, IX锁）
行锁：记录锁（Record Locks）、间隙锁（Gap Locks）、临键锁（Next-Key Locks）、插入意向锁（Insert Intention Locks）

#### 共享锁（S）、排他锁（X）
锁的模式，每种锁都有shared和exclusive两种模式。
特性：X锁与其他X,S锁不兼容，S与S锁兼容

#### 锁定读
SELECT ... FOR SHARE 和 SELECT ... FOR UPDATE
这两种锁定读在搜索时所遇到的每一条索引记录(index record)上设置共享锁或排它锁。

#### 意向锁（IS、IX）
意向锁，协调行锁和表锁之间的关系的，实现了“表锁是否冲突”的快速判断。协调表的读写锁和行的读写锁（不同粒度锁）之间关系。
场景：获取行的S锁，需要先获取表的IS锁；获取行的X锁，需要先获取表的IX锁。
注：与意向锁冲突的不是行的X锁，是表的X锁！
IX 与表的 X 和 S 冲突，意思当前表有行锁X锁，不能上表的X或S锁。
IS 与表的 X 冲突，意思当前表有行锁S锁，不能上表的X锁。
why意向锁：“表锁是否冲突”的快速判断。否则需要遍历表去判断行锁的存在，才能判断冲突。

#### 索引记录锁(Record Locks)
行锁，锁定的是索引记录。所谓的“锁定某个行”或“在某个行上设置锁”。就是在某个索引的特定索引记录（或称索引条目、索引项、索引入口）上设置锁。有shared或exclusive两种模式

#### 间隙锁(Gap Locks)
锁定尚未存在的记录，即索引记录之间的间隙。有shard或exclusive两种模式，但，两种模式没有任何区别，二者等价。
gap lock 之间可以共存 !  两个事务可以共同持有。gap只阻塞插入意向锁。
why gap lock: 唯一目的就是阻止其他事务向gap中插入数据行。解决 phantom row问题。
gap lock与 插入意向锁冲突。rc 没有gap lock， rr使用gap与插入意向锁的冲突解决幻读。

#### 下一个键锁(Next-Key Locks)
next-key lock 是 (索引记录上的索引记录锁) + (该索引记录**前面**的间隙上的锁) 二者的合体，它锁定索引记录以及该索引记录**前面**的间隙。有shard或exclusive两种模式。
next-key lock 带的 gap lock固定锁记录前面的间隙！

#### 插入意向锁(Insert Intention Locks)
特殊的gap lock。
插入行之前，INSERT操作会首先在索引记录之间的间隙上设置insert intention lock，该锁的范围是(插入值, 向下的一个索引值)。有shard或exclusive两种模式，但，两种模式没有任何区别，二者等价。
插入意向锁 会和 gap锁冲突！
插入意向锁 本身多个不会冲突！插入意向锁也不会阻塞gap锁，只有先gap, 再insert Intention Lock 会阻塞。
why插入意向锁: 隔离级别为RR时正是利用插入意向锁与gap锁的冲突，来解决幻读问题。

#### 不同索引加锁情况
1. 聚簇索引
    索引命中：对这个 id 聚簇索引加 X 锁
    索引未命中：对 id 这个聚簇索引加 GAP 锁，GAP的范围是两个索引的间隙
    范围条件： 对范围区间内命中的id聚簇索引加Next-Key 锁，即左开右闭的GAP锁+X锁。eg: UPDATE student SET age = 100 WHERE id < 3
2. 唯一索引
    索引命中：二级索引的叶子节点中保存了主键索引的位置，在给二级索引加锁的时候，对应聚簇索引也会一并加锁。加两个X锁。
    索引未命中：只会在二级索引加GAP锁，不会在聚簇索引上加锁。
    范围条件：对范围区间内命中的唯一索引加Next-Key 锁(对条件范围内的索引加GAP锁+X锁)，并且对命中索引对应的聚簇索引也会加 X锁。
3. 非唯一索引
    索引命中：对命中的非唯一索引加 Next-Key锁，在非唯一索引相邻区间加 GAP 锁。并且对命中索引对应的聚簇索引加X锁。
    索引未命中: 只会在二级索引加GAP锁，不会在聚簇索引上加锁。
    范围条件：与唯一索引的范围条件加锁类似，对命中的索引加Next-Key锁(对条件范围内的索引加GAP锁+X锁)，对应的聚簇索引加X锁。
4. 无索引
    在没有索引的时候，只能走聚簇索引，对表中的记录进行全表扫描。会给所有记录加行锁，所有聚簇索引之间会加上 GAP 锁。

#### 为什么无索引更新会锁全表？
在没有索引的时候，只能走聚簇索引，对表中的记录进行全表扫描。会给所有记录加行锁，所有聚簇索引和聚簇索引之间还会加上 GAP 锁。
eg: UPDATE student SET age = 100 WHERE age = 33;   age无索引。
why: RR 下要保证当前读时不出现幻读。需要锁全表！否则任何一个记录都可以变成age=33, 任何一个地方都可以insert一条age=33。再执行该sql的时候会发现多出记录了。

#### 死锁场景
死锁的可能性并不受隔离级别的影响。隔离级别改变的是读操作的行为，而死锁是由于写操作产生的。
1. 两个行锁顺序不同的获取
stu_no字段 是唯一索引_
```
session1 : update student set age = 88 where stu_no = 1;
session2 : update student set age = 99 where stu_no = 3;
session1 : update student set age = 99 where stu_no = 3;
session2 : update student set age = 88 where stu_no = 1;
```
死锁, 一个有stu_no=1的 X RECORD锁，一个有_stu_no=3的。_
2. 两个gap锁各自阻塞另一个事务的插入意向锁
stu_no字段 是唯一索引_
```
session1 : delete from student where stu_no = 8;
session2 : delete from student where stu_no= 7;
session1 : insert student (stu_no) values (6);
session2 : insert student (stu_no) values (9);
```
3. 两个insert duplicate失败会加S RECORD, 若session1 commit, 两个返回duplicate-key error，若session1 rollback, 两个都想获取X锁，但对方都不释放S锁
stu_no字段是唯一索引_
```
session1: insert student (stu_no) values (6);
session2: insert student (stu_no) values (6);
session3: insert student (stu_no) values (6);
session1: rollback;
session2: commit;
session3: commit;
```

参考文档：
https://mp.weixin.qq.com/s/IyeiP2t1TGxZlPqSPAWNZg
https://mp.weixin.qq.com/s/dRIfbVwAJfEuZ978VlyXIA
https://mp.weixin.qq.com/s/S9Fzwu7-g81DsWgjaARHvQ
