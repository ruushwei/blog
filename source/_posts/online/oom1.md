---
title: 一次线上OOM问题排查
date: 2021-03-18T12:54:24+02:00
tags: 
- 线上问题
categories: 线上问题
---

<!-- toc -->

一、问题

一个查询 服务 jvm 偶现oom，容器异常退出，过会儿自动重新拉起进程，容器不变
​二、时间线
20210308 怀疑是cpu飙升导致
观察cpu出现飙升， 通过 top 命令查到进程 pid 通过 top -H -p <pid>, 看到下图


![](https://ipic-1252327316.cos.ap-beijing.myqcloud.com/image/jvm/top.png)

**耗cpu的线程都为GC线程，排除是cpu飙升的原因**

20210311 复现oom 分析dump文件
本想通过sz 将dump文件拉到本地，但太费劲了，dump文件为16G, 即使分段也十分费劲，询问基础架构同学，提供了公司的dump文件分析工具 jifa
jifa 一个可以快速将dump文件快速上传到云端，且在远端使用MAT进行解析的工具，非常好使
将dump文件上传到S3云端，再根据返回的链接，直接跳转到jifa 平台，默认弹出刚才文件从S3导入的选择，选择导入，即会从S3导入到公司 jifa 平台。
选择对应机器的dump文件，进行在线分析，加载后即可得到线上MAT分析结果
从Dominator Tree中可看到有一个线程占用了96%的内存，且是线程中的一个ArrayList 占用着几乎这96%内存，点开发现全是 DotVO实体

![](https://ipic-1252327316.cos.ap-beijing.myqcloud.com/image/jvm/mat1.jpg)

该ArrayList 中存在 1.4 亿 个 DotVO实体


![](https://ipic-1252327316.cos.ap-beijing.myqcloud.com/image/jvm/mat2.png)

查该问题线程的堆栈，发现停在了 AssetsSecondInfoVO.result2VO


![](https://ipic-1252327316.cos.ap-beijing.myqcloud.com/image/jvm/mat3.png)

此处的逻辑为一个时间范围的中间数据补齐


![](https://ipic-1252327316.cos.ap-beijing.myqcloud.com/image/jvm/mat4.png)


看DotVO 中的时间 竟然还有50多万s 的时间。。。


![](https://ipic-1252327316.cos.ap-beijing.myqcloud.com/image/jvm/mat6.jpg)

查hive表数据，确实是有很大的时长的数据，比如视频时长为28s, 但是hive表中时间数据有16亿s的。。此时做中间时间补齐，则数据量会爆炸
询问数仓同学，是脏数据，是个已知问题
fix: 添加对下游数据正确性的校验逻辑 & 数仓同学推动上报源改造
