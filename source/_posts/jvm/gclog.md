---
title: gc日志
date: 2019-02-14T12:54:24+02:00
tags: jvm
categories: jvm

---



### 一、gc日志

虽然有falcon，但gc日志在查问题时对一次gc更加直观

<!--more-->

#### 1、初步观察

略





#### 2、简单聊下 ParNew and CMS



我们使用的垃圾收集器是 parnew + cms ， 使用参数 -XX:+UseConcMarkSweepGC

配置不同的垃圾收集器：经典回顾：[JVM-探究（二）：JVM实验和 GC 日志解读](http://jacobchang.cn/experiment-on-jvm.html)

![129584597](https://ws2.sinaimg.cn/large/006tKfTcgy1g076ggt3tpj30xk0k4wi0.jpg)



ParNew and CMS  响应时间优先的选择

"Concurrent Mark and Sweep" 是CMS的全称

年轻代：采用 stop-the-world mark-copy 算法；

年老代：采用 Mostly Concurrent mark-sweep 算法；

设计目标：年老代收集的时候避免长时间的暂停， 响应时间优先。



[吞吐量 VS 暂停时间](http://ifeve.com/useful-jvm-flags-part-6-throughput-collector/)

[7种垃圾收集器比较](https://crowhawk.github.io/2017/08/15/jvm_3/)



能够达成该目标主要因为以下两个原因：

1  它不会花时间整理压缩年老代，而是维护了一个叫做 free-lists 的数据结构，该数据结构用来管理那些回收再利用的内存空间；

2  mark-sweep分为多个阶段，其中一大部分阶段GC的工作是和Application threads的工作同时进行的（当然，gc线程会和用户线程竞争CPU的时间），默认的GC的工作线程为你服务器物理CPU核数的1/4；

补充：当你的服务器是多核同时你的目标是低延时，那该GC的搭配则是你的不二选择。





#### 3、打印gc日志命令

| 命令                           | 作用（默认都为关闭）                |
| ------------------------------ | ----------------------------------- |
| -XX:+PrintGCDetails            | 1打印GC的详细信息                   |
| -XX:+PrintHeapAtGC             | 2在GC前后输出GC堆的概要状况         |
| -XX:+PrintTenuringDistribution | 3打印GC后新生代各个年龄对象的大小   |
| -XX:+PrintGCTimeStamps         | 4输出GC的时间戳（以基准时间的形式） |
| -XX:+PrintGCDateStamps         | 5输出GC的时间戳（以日期的形式）     |



示例:

![129564986](https://ws1.sinaimg.cn/large/006tKfTcgy1g076ghxamhj31hc0klajs.jpg)





#### 4、younggc

![129485142](https://ws3.sinaimg.cn/large/006tKfTcgy1g076h4epr3j31hc0o6aks.jpg)



2016-08-23T02:23:07.219-0200**1**: 64.322**2**:[GC**3**(Allocation Failure**4**) 64.322: [ParNew**5**: 613404K->68068K**6**(613440K)**7**, 0.1020465 secs**8**] 10885349K->10880154K**9**(12514816K)**10**, 0.1021309 secs**11**][Times: user=0.78 sys=0.01, real=0.11 secs]**12**



1. 2016-08-23T02:23:07.219-0200 – GC发生的时间；
2. 64.322 – GC开始，相对JVM启动的相对时间，单位是秒；
3. GC – 区别MinorGC和FullGC的标识，这次代表的是MinorGC;
4. Allocation Failure – MinorGC的原因，在这个case里边，由于年轻代不满足申请的空间，因此触发了MinorGC;    minor gc的原因都有什么？ 
5. ParNew – 收集器的名称，它预示了年轻代使用一个并行的 mark-copy stop-the-world 垃圾收集器；
6. 613404K->68068K – 收集前后年轻代的使用情况；
7. (613440K) – 整个年轻代的容量；
8. 0.1020465 secs – 表示该区域这次 GC 使用的时间
9. 10885349K->10880154K – 收集前后整个堆的使用情况；
10. (12514816K) – 整个堆的容量；
11. 0.1021309 secs –  表示 Java堆 这次 GC 使用的时间. ParNew收集器标记和复制年轻代活着的对象所花费的时间（包括和老年代通信的开销、对象晋升到老年代时间、垃圾收集周期结束一些最后的清理对象等的花销）；
12. [Times: user=0.78 sys=0.01, real=0.11 secs] – GC事件在不同维度的耗时，具体的用英文解释起来更加合理:
    - user – Total CPU time that was consumed by Garbage Collector threads during this collection
    - sys – Time spent in OS calls or waiting for system event
    - real – Clock time for which your application was stopped. With Parallel GC this number should be close to (user time + system time) divided by the number of threads used by the Garbage Collector. In this particular case 8 threads were used. Note that due to some activities not being parallelizable, it always exceeds the ratio by a certain amount.





younggc的原因：

Allocation Failure  新生代没有足够的空间分配对象

GCLocker Initiated GC    如果线程执行在JNI临界区时，刚好需要进行GC，此时GC locker将会阻止GC的发生，同时阻止其他线程进入JNI临界区，直到最后一个线程退出临界区时触发一次GC





#### 5、cmsgc & fullgc



```
2013-11-27T04:00:12.819+0800: 38892.743: [GC [1 CMS-initial-mark: 1547313K(2146304K)] 1734957K(4023680K), 0.1390860 secs] [Times: user=0.14 sys=0.00, real=0.14 secs]
2013-11-27T04:00:12.958+0800: 38892.883: [CMS-concurrent-mark-start]
2013-11-27T04:00:19.231+0800: 38899.155: [CMS-concurrent-mark: 6.255/6.272 secs] [Times: user=8.49 sys=1.57, real=6.27 secs]
2013-11-27T04:00:19.231+0800: 38899.155: [CMS-concurrent-preclean-start]
2013-11-27T04:00:19.250+0800: 38899.175: [CMS-concurrent-preclean: 0.018/0.019 secs] [Times: user=0.02 sys=0.00, real=0.02 secs]
2013-11-27T04:00:19.250+0800: 38899.175: [CMS-concurrent-abortable-preclean-start]
 CMS: abort preclean due to time 2013-11-27T04:00:25.252+0800: 38905.176: [CMS-concurrent-abortable-preclean: 5.993/6.002 secs] [Times: user=6.97 sys=2.16, real=6.00 secs]
2013-11-27T04:00:25.253+0800: 38905.177: [GC[YG occupancy: 573705 K (1877376 K)]38905.177: [Rescan (parallel) , 0.3685690 secs]38905.546: [weak refs processing, 0.0024100 secs]38905.548: [cla
ss unloading, 0.0177600 secs]38905.566: [scrub symbol & string tables, 0.0154090 secs] [1 CMS-remark: 1547313K(2146304K)] 2121018K(4023680K), 0.4229380 secs] [Times: user=1.41 sys=0.01, real=
0.43 secs]
2013-11-27T04:00:25.676+0800: 38905.601: [CMS-concurrent-sweep-start]
2013-11-27T04:00:26.436+0800: 38906.360: [CMS-concurrent-sweep: 0.759/0.760 secs] [Times: user=1.06 sys=0.48, real=0.76 secs]
2013-11-27T04:00:26.436+0800: 38906.360: [CMS-concurrent-reset-start]
2013-11-27T04:00:26.441+0800: 38906.365: [CMS-concurrent-reset: 0.005/0.005 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
```





[cms gc 过程](http://www.cnblogs.com/zhangxiaoguang/p/5792468.html)

[cms gc 和 full gc的关系](https://www.zhihu.com/question/41922036)

[新生代老年代gc日志](https://t.hao0.me/jvm/2016/03/15/jvm-gc-log.html)

[foreground collect](https://mp.weixin.qq.com/s/W_URLiVuCFS58AiJS257vw)



fullgc 的原因

1. 没有配置 -XX:+DisableExplicitGC情况下System.gc()可能会触发FullGC；
2. Promotion failed； （cms下会触发cms gc, 非full gc）
3. concurrent mode failure；
4. Metaspace Space使用达到MaxMetaspace阈值； (cms 下回触发cms gc, 非full gc)
5. 执行jmap -histo:live或者jmap -dump:live



Promoration failure

有足够的空间，但是由于碎片化严重，无法容纳新生代中晋升的对象，发生晋升失败， 触发cms gc的foreground cmsgc

CMS提供了两个参数来对碎片进行整理和压缩 

-XX:+UseCMSCompactAtFullCollection这个设置的作用是在进行FullGC的时候对碎片进行整理和压缩。

-XX:CMSFullGCsBeforeCompaction=*这个参数是设置在进行多少次FullGC的时候对老年代的内存进行一次碎片整理压缩. 



```
2013-11-27T03:00:53.638+0800: 35333.562: [GC 35333.562: [ParNew (promotion failed): 1877376K->1877376K(1877376K), 15.7989680 secs]35349.361: [CMS: 2144171K->2129287K(2146304K), 10.4200280 sec
s] 3514052K->2129287K(4023680K), [CMS Perm : 119979K->118652K(190132K)], 26.2193500 secs] [Times: user=30.35 sys=5.19, real=26.22 secs]
```





Concurrent Mode Failure

此错误就是在CMS GC的过程中，又有对象需要放到old区，但是old区空间不足了，放不下了就报了此错误。

导致**concurrent mode failure**的原因有是： there was not enough space in the CMS generation to promote the worst case surviving young generation objects. We name this failure as “full promotion guarantee failure” 

解决的方案有： The concurrent mode failure can either be avoided **increasing the tenured generation size** or **initiating the CMS collection at a lesser heap occupancy** by setting CMSInitiatingOccupancyFraction to a lower value and setting UseCMSInitiatingOccupancyOnly to true.

![img](https://upload-images.jianshu.io/upload_images/3288959-0ce34715b04ebf1b.png)



systm.gc

```
2013-07-21T17:44:01.554+0800: 50.568: [Full GC (System) 50.568: [CMS: 943772K->220K(2596864K), 2.3424070 secs] 1477000K->220K(4061184K), [CMS Perm : 3361K->3361K(98304K)], 2.3425410 secs] [Times: user=2.33 sys=0.01, real=2.34 secs]
```