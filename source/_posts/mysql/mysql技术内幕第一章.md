---
title: mysql技术内幕第一章、mysql体系结构和存储引擎
date: 2018-05-21T22:54:24+02:00
tags: 
- mysql
- innodb
categories: mysql
---









### Mysql体系结构





![mysql体系结构](https://ws2.sinaimg.cn/large/006tKfTcly1frkj9mvm0dj30g20ay76c.jpg)

MySQL由以下几部分组成：

- 连接池组件。
- 管理服务和工具组件。
- SQL接口组件。
- 查询分析器组件。
- 优化器组件。
- 缓冲（Cache）组件。
- 插件式存储引擎。
- 物理文件。



从图1-1还可以看出，MySQL区别于其他数据库的最重要的特点就是其插件式的表存储引擎。MySQL插件式的存储引擎架构提供了一系列标准的管 理和服务支持，这些标准与存储引擎本身无关，可能是每个数据库系统本身都必需的，如SQL分析器和优化器等，而存储引擎是底层物理结构的实现，每个存储引 擎开发者都可以按照自己的意愿来进行开发。

**注意：存储引擎是基于表的，而不是数据库。请牢牢记住图1-1所示的MySQL体系结构图，它对于你以后深入了解MySQL有极大的帮助**



<!--more-->

### 存储引擎比较



#### InnoDB存储引擎

InnoDB存储引擎支持事务，主要面向在线事务处理（OLTP）方面的应用。其特点是行锁设计、支持外键，并支持类似于Oracle的非锁定读，即默认情况下读取操作不会产生锁。MySQL 在Windows版本下的InnoDB是默认的存储引擎，同时InnoDB默认地被包含在所有的MySQL二进制发布版本中。

InnoDB存储引擎将数据放在一个逻辑的表空间中，这个表空间就像黑盒一样由InnoDB自身进行管理。从MySQL 4.1（包括4.1）版本开始，它可以将每个InnoDB存储引擎的表单独存放到一个独立的ibd文件中。与Oracle类似，InnoDB存储引擎同样 可以使用裸设备（row disk）来建立其表空间。
InnoDB通过使用多版本并发控制（MVCC）来获得高并发性，并且实现了SQL标准的4种隔离级别，默认为REPEATABLE级别。同时使 用一种被称为next-key locking的策略来避免幻读（phantom）现象的产生。除此之外，InnoDB储存引擎还提供了插入缓冲（insert buffer）、二次写（double write）、自适应哈希索引（adaptive hash index）、预读（read ahead）等高性能和高可用的功能。
对于表中数据的存储，InnoDB存储引擎采用了聚集（clustered）的方式，这种方式类似于Oracle的索引聚集表（index organized table，IOT）。每张表的存储都按主键的顺序存放，如果没有显式地在表定义时指定主键，InnoDB存储引擎会为每一行生成一个6字节的 ROWID，并以此作为主键。



#### MyISAM存储引擎

MyISAM存储引擎是MySQL官方提供的存储引擎。其特点是不支持事务、表锁和全文索引，对于一些OLAP（Online Analytical Processing，在线分析处理）操作速度快。除Windows版本外，是所有MySQL版本默认的存储引擎。
MyISAM存储引擎表由MYD和MYI组成，MYD用来存放数据文件，MYI用来存放索引文件。可以通过使用myisampack工具来进一步 压缩数据文件，因为myisampack工具使用赫夫曼（Huffman）编码静态算法来压缩数据，因此使用myisampack工具压缩后的表是只读 的，当然你也可以通过myisampack来解压数据文件。
在MySQL 5.0版本之前，MyISAM默认支持的表大小为4G，如果需要支持大于4G的MyISAM表时，则需要制定MAX_ROWS和 AVG_ROW_LENGTH属性。从MySQL 5.0版本开始，MyISAM默认支持256T的单表数据，这足够满足一般应用的需求。
注意：对于MyISAM存储引擎表，MySQL数据库只缓存其索引文件，数据文件的缓存交由操作系统本身来完成，这与其他使用LRU算法缓存数据 的大部分数据库大不相同。此外，在MySQL 5.1.23版本之前，无论是在32位还是64位操作系统环境下，缓存索引的缓冲区最大只能设置为4G。在之后的版本中，64位系统可以支持大于4G的索 引缓冲区







![存储引擎之间的比较](https://ws3.sinaimg.cn/large/006tKfTcly1frkj9r68rpj31kw0rlwju.jpg)