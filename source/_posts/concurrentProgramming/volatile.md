---
title: volatile可见性、内存屏障
date: 2019-06-02T12:54:24+02:00
tags: 
- java
- 并发
categories: 并发
---

#### 总结

volatile关键字具有许多特性，其中最重要的特性就是保证了用volatile修饰的变量对所有线程的可见性。利用内存屏障使得读写、读读、写写 都不能同时发生, 且不能指令重排。

#### 可见性
当一个线程修改了变量的值，新的值会立刻同步到主内存当中。而其他线程读取这个变量的时候，也会从主内存中拉取最新的变量值。

得益于java语言的先行发生原则（happens-before）
对于一个volatile变量的写操作先行发生于后面对这个变量的读操作。

#### 内存屏障

内存屏障（Memory Barrier）是一种CPU指令。**禁止重排序，控制执行顺序，刷新缓存与获取最新主存数据。**

<!--more-->

内存屏障也称为内存栅栏或栅栏指令，是一种屏障指令，它使CPU或编译器对屏障指令之前和之后发出的内存操作执行一个排序约束。 这通常意味着在屏障之前发布的操作被保证在屏障之后发布的操作之前执行。

Java内存屏障主要有Load和Store两类。 

**对Load Barrier来说，在读指令前插入读屏障，可以让高速缓存中的数据失效，重新从主内存加载数据**

**对Store Barrier来说，在写指令之后插入写屏障，能让写入缓存的最新数据写回到主内存**

内存屏障共分为四种类型：

- LoadLoad屏障：
抽象场景：Load1; LoadLoad; Load2
Load1 和 Load2 代表两条读取指令。在Load2要读取的数据被访问前，保证Load1要读取的数据被读取完毕。

- StoreStore屏障：
抽象场景：Store1; StoreStore; Store2
Store1 和 Store2代表两条写入指令。在Store2写入执行前，保证Store1的写入操作对其它处理器可见

- LoadStore屏障：
抽象场景：Load1; LoadStore; Store2
在Store2被写入前，保证Load1要读取的数据被读取完毕。

- StoreLoad屏障：
抽象场景：Store1; StoreLoad; Load2
在Load2读取操作执行前，保证Store1的写入对所有处理器可见。StoreLoad屏障的开销是四种屏障中最大的。

简单总结就是读和写之间都支持通过屏障，等前一个完全结束再进行之后的操作

#### volatile涉及内存屏障

1. 在每个volatile写操作前插入StoreStore屏障，在写操作后插入StoreLoad屏障; 
- 意思等别人写完自己再写，别人等自己写完之后才能读


2. 在每个volatile读操作前插入LoadLoad屏障，在读操作后插入LoadStore屏障。
- 意思等别人读完自己再读，别人等自己读完之后才能写

**读写、读读、写写 都不能同时发生, 且不能重排序**