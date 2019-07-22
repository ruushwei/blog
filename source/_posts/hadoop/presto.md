---
title: presto简介、与hive比较
date: 2018-03-29T14:54:24+02:00
tags:
- hadoop
categories:
- hadoop
---







最近在查询hive数据做展示的时候，读取hive数据，一开始使用hive查询，但速度非常慢，偶然发现可以使用presto引擎，速度相比hive要快上许多，这里进行一下比较和整理下大概原因。





<!--more-->

#### 一、 简介



![presto简介](https://ws3.sinaimg.cn/large/006tKfTcly1fptuxjp14aj31a00dawja.jpg)









#### 二、与hive比较



![hive_presto](https://ws4.sinaimg.cn/large/006tKfTcly1fptuxnaewnj31600xewli.jpg)





hive查询需要把数据先map， 按照查询条件为key， 取的字段为value，得到一条条数据，然后按key分类持久化到磁盘，然后再从磁盘读出来进行 count,sum，distinct等reduce操作，每一次map reduce都要写 读磁盘.

而且 将 sql分为多个语句，分的有先后顺序，需要等前面的 算完了，再进行下一步



presto的话，是纯内存的，不是mapreduce，但也是分解sql为多个任务，但是是并发进行，最后再串联，全程都是内存操作，所以很快。







实际执行效果如下图所示:

![presto执行运转](https://ws1.sinaimg.cn/large/006tKfTcly1fptuxkr6uaj312c0k8mz6.jpg)







