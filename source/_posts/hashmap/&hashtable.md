---
title: Hashmap源码解析-与hashtable区别
date: 2019-05-26T12:54:24+02:00
tags: 
- java 
- hashmap
categories: java
---

HashMap是非线程安全的，HashTable是线程安全的。

<!--more-->

### 系列目录

1. [总览&目录](../总结目录)
2. [链表节点NODE](../链表节点NODE)
3. [构造函数](../构造函数)
4. [扩容函数](../扩容函数)
5. [put](../put)
6. [remove](../remove)
7. [get](../get)
8. [遍历](../遍历)
9. [&hashtable](../&hashtable)



#### hashmap & hashtable

（1）HashMap是非线程安全的，HashTable是线程安全的。

（2）因为线程安全、哈希效率的问题，HashMap效率比HashTable的要高

（3）HashMap的键和值都允许有null存在，而HashTable则都不行。

（4）HashMap的迭代器(Iterator)是fail-fast迭代器，而Hashtable的enumerator迭代器不是fail-fast的