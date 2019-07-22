---
title: Innodb索引原理及其优化
date: 2018-08-31T10:54:24+02:00
tags: 
- mysql
- 索引
categories: mysql
---



### 一、目的



了解索引结构，explain方法，优化原理

### 二、索引名词



#### **2.1 聚簇索引**

![1](https://ws2.sinaimg.cn/large/0069RVTdgy1fusqrrxoywj30k00fxdhg.jpg)



- 聚簇索引也称为聚集索引，聚类索引，簇集索引，聚簇索引的顺序就是数据的物理存储顺序。
- 由于聚簇索引规定数据在表中的物理存储顺序，因此一个表只能包含一个聚簇索引。
- Innodb聚簇索引根据每张表的主键构造一棵B+树，叶子节点中存放的即为整张表的行记录数据，也称为数据页。
- 对于主键的排序查找和范围查找速度非常快。叶子节点的数据就是用户所要查询的数据

<!--more-->

**把非顺序的字段当主键**，肯定会因为插入，删除，造成数据位置的更新，插入可能会页分裂，留下碎片，浪费资源空间。

我们都是自增id当主键，**把时间当做主键**呢？

1、自增id占用存储空间小，一索引页可存更多

2、时间会有并发问题，时间+随机数也行，但占的空间会更大，会增加读取索引页数，增加io

3、如果该表的需求主要就是按时间查找，用时间(+随机数)的方式也不错。 如果时间不常变化的话。



#### **2.2 非聚簇索引**

非聚簇索引记录的物理顺序和索引的顺序无关。（MyISAM存储引擎）



非聚簇索引和聚簇索引的优缺点？

聚簇索引，主键的插入速度要比非聚簇索引主键的插入速度慢很多，要维护数据的位置。

聚簇索引数据根据主键聚成堆了适合排序，非聚簇索引则没有按序存放，需要额外消耗资源来排序。  



我们其实很少用主键做范围查询



#### **2.3 辅助索引**



也称二级索引，叶子节点并不包含行记录的全部数据

辅助索引的书签就是相应行数据的聚集索引键

辅助索引的存在并不影响数据在聚集索引中的组织，因此每张表上可以有多个辅助索引。



为什么**辅助索引**叶子节点不直接指向数据页，**还要走一遍主键索引**？

这样的策略减少了当出现行移动或者数据页分裂时二级索引的维护工作



#### **2.4 覆盖索引**



回表：就是在使用辅助索引时，因为**辅助索引只存储了部分数据**，如果根据键值查找到的数据不能包括全部目标数据时（就是无法使用到覆盖索引），就需要通过二级索引的指针，也就是键值对中的值，来找到聚簇索引的全部数据，然后根据完整的数据取出所需要的列的过程就称之为回表。这种在二级索引中不能找到所有需要的数据列的现象，被称为非覆盖索引，反之称为覆盖索引。

因为回表本身是需要去另一个索引（聚簇索引）中查找数据的，性能必然会受到影响，那为了尽可能的提高性能就需要尽量的减少回表次数，所以可以试着将出现频率非常高的语句中所有使用到的列以合适的顺序建一个二级索引，这样所有需要的列都被这个二级索引覆盖了，就不需要回表了，从而一定程度上提高了性能



#### **2.5 联合索引**



也叫复合索引，多个字段组成的索引



![2](https://ws2.sinaimg.cn/large/0069RVTdgy1fusqrsqyn3j308z031q30.jpg)



有时和覆盖索引连用，强行联合，也可提高效率。



### 三、索引结构B+树



B+树的高度一般都在2-4层，这也就是说查找某一键值的行记录最多只需要2-4次IO。

![3](https://ws1.sinaimg.cn/large/0069RVTdgy1fusqrtea1nj319g0j2wgi.jpg)





### **四、网上一些优化建议分析**



**1、前导模糊查询不能使用索引。**

**select** * **from** doc **where** title **like** '%XX';



很明显，'%lice'  是不可能能使用到索引的

![4](https://ws1.sinaimg.cn/large/0069RVTdgy1fusqru5oerj30f306674p.jpg)





**2、在字段上进行计算不能命中索引**



**3、强制类型转换会全表扫描**

**explain select** *** **from campaignxxx **where creator**=22;      // 无法使用索引

​     

字段定义字符串用数字查询无法使用索引，定义数字用字符串查询可以使用。

**explain select** *** **from campaignxxx **where status**=**'32as'**;   // 可以使用索引



**4、更新十分频繁字段上不宜建立索引**

更新会变更 B+ 树，更新频繁的字段建立索引会大大降低数据库性能。



**5、数据区分度不高的字段上不宜建立索引**

- “性别”这种区分度不大的属性，建立索引是没有什么意义的，不能有效过滤数据，性能与全表扫描类似。
- 一般区分度在80%以上的时候就可以建立索引，区分度可以使用 count(distinct(列名))/count(*) 来计算。

 

意思我们campaignxxx表的business和status字段也没必要建索引吗

没必要的关键还是索引要回表，但索引过滤这一层如果确实筛掉很多，肯定还是有用的，还是要看业务场景，比如status字段，查64的和查32的， 64的活动数据库 2912w， 32的活动 41w， 就算多回了表也是快的，而我们主要是查32活动。



select count(1) from campaignxxx where status=32;

select count(1) from campaignxxx where discounttype=4;



**explain select** *** **from campaignxxx **where status**=32 **limit** 10000000;
**explain select** *** **from campaignxxx **where discounttype**=4 **limit** 10000000;

**explain select** *** **from campaignxxx **where status**=64 **limit** 10000000;
**explain select** *** **from campaignxxx **where discounttype**=3 **limit** 10000000;



对于占比多的部分，查询速度会慢。 **区分度不高但你查的是少的那部分，建索引也很有效。**



性别这种可能确实不适合。



**6、利用延迟关联或者子查询优化超多分页场景**

      一般分页：

**explain select** *** **from campaignxxx **order by modtime limit** 300000, 20;

优化分页:
**explain select** *** **from campaignxxx cb **join** (**select id from** campaignxxx **order by modtime limit** 300000, 20) a **on** cb.**id**=a.**id**;





### 五、索引使用的地方和值得优化的地方



**1、where** 

select xxx from  order where userid=123;



**2、order by** 

select * from  user where departmentid=1 order by userid ;



不给order by加索引，就需要mysql server层自己去排序，explain Extra会得到using filesort

当使用排序的空间超过sort_buffer_size的时候，会使用临时表，explain Extra会得到using temporary

![5](https://ws3.sinaimg.cn/large/0069RVTdgy1fusqrtunipj30g20aymyu.jpg)

**3、group by** 



group by 操作在没有合适的索引可用的时候，通常先扫描整个表提取数据并创建一个临时表，然后按照group by 指定的列进行排序

在执行计划中通常可以看到 using temporary; using filesort



**explain select discounttype**, *count*(1) **as** num **from** campaignxxx **group by discounttype**;

**explain select status**, *count*(1) **as** num **from** campaignxxx **group by status**;



**4、join on** 



不用说，明显是要再on的关联字段上加索引，但是感觉肯定是要加，但**加了具体是如何作用的**



**join的原理**

MySQL是只支持一种JOIN算法Nested-Loop Join（嵌套循环链接）

Simple Nested-Loop Join （最初始的方法）：

![6](https://ws1.sinaimg.cn/large/0069RVTdgy1fusqrsc1cnj30jn0bvwey.jpg)

通常两表关联，数量少的表为驱动表。（其实就是求交集）

从驱动表中取出R1匹配S表所有列，然后R2，R3,直到将R表中的所有数据匹配完，然后合并数据，可以看到这种算法要对S表进行**RN**次访问，虽然简单，但是相对来说开销还是太大了



Index Nested-Loop Join（有索引的情况， Table S有索引）

![7](https://ws2.sinaimg.cn/large/0069RVTdgy1fusqrunmzaj30jg0e4t9k.jpg)

索引嵌套联系由于非驱动表上有索引，所以比较的时候不再需要一条条记录进行比较，而可以通过索引来减少比较，从而加速查询。这也就是平时我们在做关联查询的时候必须要求关联字段有索引的一个主要原因。**R\*logS**



Block Nested-Loop Join（没有索引的时候）



![8](https://ws3.sinaimg.cn/large/0069RVTdgy1fusqrrjnvrj30jq0ecmyg.jpg)



使用join buffer将驱动表的查询JOIN相关列都给缓冲到了JOIN BUFFER当中，然后批量与非驱动表进行比较，这也来实现的话，可以将多次比较合并到一次，降低了非驱动表的访问频率。也就是只需要访问一次S表。这样来说的话，就不会出现多次访问非驱动表的情况了，也只有这种情况下才会访问join buffer。**R + Slogr**





### 六、如何查看sql索引使用情况，Explain性能分析



当想知道sql的真实执行情况，用explain获取执行计划。



#### 6.1、列信息

| 列            | 含义                                                         |
| ------------- | ------------------------------------------------------------ |
| id            | SELECT 查询的标识符. 每个 SELECT 都会自动分配一个唯一的标识符. |
| select_type   | SELECT 查询的类型.                                           |
| table         | 查询的是哪个表                                               |
| partitions    | 匹配的分区                                                   |
| type          | 表参与类型                                                   |
| possible_keys | 此次查询中可能选用的索引                                     |
| key           | 此次查询中确切使用到的索引                                   |
| key_len       | 索引使用的长度, 可以看使用了复合索引的哪部分                 |
| ref           | 哪个字段或常数与 key 一起被使用                              |
| rows          | 显示此查询一共扫描了多少行. 这个是一个估计值                 |
| filtered      | 表示此查询条件所过滤的数据的百分比                           |
| extra         | 额外的信息                                                   |



#### 6.2、type 表参与类型



官方给的定义是 join type

级别:  **ALL < index < range  < ref < eq_ref < const < system**



system: 表中只有一条数据. 这个类型是特殊的 const 类型

const: 针对主键或唯一索引的等值查询扫描, 最多只返回一行数据

**explain select** *** **from campaignxxx **where id**=269382403;



eq_ref： 使用了唯一或主键关联

**explain select** *** **from campaignxxx cb **join** campaignobx co **on** cb.**id**=co.**campaignid;**



ref:  使用了非唯一或非主键关联

**explain select** *** **from campaignxxx cb **join** campaignobx co **on** cb.**id**=co.**campaignid where** cb.**status**=32;



range: 索引范围查找

**explain select** *** **from campaignxxx **where modtime**>1535425311;



index: 全索引扫描

**explain select modtime from** campaignxxx **order by modtime limit** 10000000;

ALL: 全表扫描

**explain select** *** **from campaignxxx **order by modtime limit** 10000000;



#### 6.3、extra: 额外信息

using where： 代表发生了过滤 

using index：覆盖索引,  表示查询在索引树中就可查找所需数据, 不用扫描表数据文件, 往往说明性能不错



using filesort : 需额外的排序操作, 不能通过索引顺序达到排序效果

**explain select** *** **from campaignxxx **order by addtime**;

using temporary ： 有使用临时表, 一般出现于排序, 分组和多表 join 的情况, 查询效率不高, 建议优化

**explain select** *** **from campaignxxx **group by addtime**;



### 七、总结



使用explain命令、综合业务添加合适的索引





### 八、参考文献

<http://www.ywnds.com/?p=5202>

<https://segmentfault.com/a/1190000008131735#articleHeader6>

<https://www.cnblogs.com/shengdimaya/p/7123069.html>