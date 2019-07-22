---
title: Hashmap源码解析-链表节点NODE
date: 2019-05-26T12:54:24+02:00
tags: 
- java 
- hashmap
categories: java


---


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



#### 链表节点NODE

重点:

1.单链表

2.hashCode()是将key的hashCode和value的hashCode异或得到

<!--more-->


```java
static class Node<K,V> implements Map.Entry<K,V> {

    final int hash;   //哈希值

    final K key;      //key

    V value;          //value

    Node<K,V> next;   //链表后置节点

    Node(int hash, K key, V value, Node<K,V> next) {

        this.hash = hash;

        this.key = key;

        this.value = value;

        this.next = next;

    }

    public final K getKey()        { return key; }

    public final V getValue()      { return value; }

    public final String toString() { return key + "=" + value; }

    //每一个节点的hash值，是将key的hashCode 和 value的hashCode 亦或得到的。

    public final int hashCode() {

        return Objects.hashCode(key) ^ Objects.hashCode(value);

    }

    //设置新的value 同时返回旧value

    public final V setValue(V newValue) {

        V oldValue = value;

        value = newValue;

        return oldValue;

    }

    public final boolean equals(Object o) {

        if (o == this)

            return true;

        if (o instanceof Map.Entry) {

            Map.Entry<?,?> e = (Map.Entry<?,?>)o;

            if (Objects.equals(key, e.getKey()) &&

                Objects.equals(value, e.getValue()))

                return true;

        }

        return false;

    }

}

```