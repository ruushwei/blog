---
title: gc调优（parnew+cms）
date: 2020-04-14T12:54:24+02:00
tags: jvm
categories: jvm   
---

### 1如何做gc调优
调优目的：减少系统停顿时间
优化目标：减少younggc、减少cmsgc、减少fullgc
Old GC：只清理老年代空间的 GC 事件，只有 CMS 的并发收集是这个模式。
Full GC：清理整个堆的 GC 事件，包括新生代、老年代、元空间等 。

#### 1.1younggc原因
1.请求中生成的对象多。新生代空间较小，Eden区很快被填满，就会导致频繁young GC     
#### 1.1如何减少younggc
1.增大新生代空间
2.减少请求中生成对象大小
1.2cmsgc原因
1.有过多的对象到达了老年代，频繁触发cms gc
1.2如何减少cmsgc
1.增大老年代大小
2.减少进入老年代的对象大小, 提高晋升年龄阈值（如果小的话，比如改成了3、4）

#### 1.3fullgc原因
1.metaspace扩容

2.CMS GC时出现promotion failed(晋升失败)和concurrent mode failure 触发串行full gc。

    promotion failed: 当新生代发生垃圾回收，老年代有足够的空间可以容纳晋升的对象，但是由于空闲空间的碎片化，导致晋升失败，此时会触发单线程且带压缩动作的Full GC
    concurrent mode failure: 发生的原因一般是CMS正在进行，但是由于老年代空间不足，需要尽快回收老年代里面的不再被使用的对象，这时停止所有的线程，同时终止CMS，直接进行Serial Old GC
3.统计得到的Young GC晋升到老年代的平均大小大于老年代的剩余空间；

4.主动触发Full GC，

    程序代码执行System.gc(); 
    命令行中dump整个堆: 执行jmap -histo:live [pid]

#### 1.3如何减少fullgc

原因1. 启动时设置好metaspace，避免因扩容导致fullgc
    "-XX:MetaspaceSize=256M",
    "-XX:MaxMetaspaceSize=256M"

原因2&3. 减少cms gc

•降低触发CMS GC的阈值, 以保证有足够的空间。即参数-XX:CMSInitiatingOccupancyFraction的值，让CMS GC尽早执行。
•增大老年代空间
•让对象尽量在新生代回收，避免进入老年代

原因4. 避免使用System.gc()

### 2.Jvm如何内存分配
eg: 16G内存机器，9G老年代，4G新生代，3G直接内存

### 3.gc调优案例
#### 3.1younggc
1.请求中生成的对象大，eg:历史数据因历史原因对应很多或很大的返回结果，频繁访问造成gc较多，可以在场景允许的情况下将这部分数据下掉或者做个截断处理。减少younggc的频率和复制对象所花费的时间。
2.增大新生代大小
#### 3.2fullgc
1.服务启动时metaspace扩容触发fullgc，导致刚启动时服务不稳定，jvm启动参数中设置好Metaspace的size和最大size。
2.服务偶尔发生concurrent mode failure。服务瞬间的可用性下降严重。修改jvm参数，降低触发cms gc的阈值（90%->75%），避免触发串行full gc。
#### 3.3cmsgc
1.增大old区。肯定是有用的，至少能减少触发的频率
2.也可能是晋升年龄阈值比较小。-XX:MaxTenuringThreshold，默认15, 可能有人改小了，让年龄小的就直接到old区了，比如改成3、4。

### 备注：cms内存碎片问题
通常 CMS 的 GC 过程基于标记清除算法，不带压缩动作，导致越来越多的内存碎片需要压缩。常见以下场景会触发内存碎片压缩
新生代 Young GC 出现新生代晋升担保失败（promotion failed)）

程序主动执行System.gc()

可通过参数 CMSFullGCsBeforeCompaction 的值，设置多少次 Full GC 触发一次压缩。默认值为 0，代表每次进入 Full GC 都会触发压缩，带压缩动作的算法为上面提到的单线程 Serial Old 算法，暂停时间（STW）时间非常长，需要尽可能减少压缩时间。

