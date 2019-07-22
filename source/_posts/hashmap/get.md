---
title: Hashmap源码解析-get
date: 2019-05-26T12:54:24+02:00
tags: 
- java 
- hashmap
categories: java


---



#### get



查找过程和删除基本差不多， 找到返回节点，否则返回null

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


```java
public V get(Object key) {

    Node<K,V> e;

    //传入扰动后的哈希值 和 key 找到目标节点Node

    return (e = getNode(hash(key), key)) == null ? null : e.value;

}

//传入扰动后的哈希值 和 key 找到目标节点Node

final Node<K,V> getNode(int hash, Object key) {

    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;

    //查找过程和删除基本差不多， 找到返回节点，否则返回null

    if ((tab = table) != null && (n = tab.length) > 0 &&

        (first = tab[(n - 1) & hash]) != null) {

        if (first.hash == hash && // always check first node

            ((k = first.key) == key || (key != null && key.equals(k))))

            return first;

        if ((e = first.next) != null) {

            if (first instanceof TreeNode)

                return ((TreeNode<K,V>)first).getTreeNode(hash, key);

            do {

                if (e.hash == hash &&

                    ((k = e.key) == key || (key != null && key.equals(k))))

                    return e;

            } while ((e = e.next) != null);

        }

    }

    return null;

}

public boolean containsKey(Object key) {

    return getNode(hash(key), key) != null;

}

public boolean containsValue(Object value) {

    Node<K,V>[] tab; V v;

    //遍历哈希桶上的每一个链表

    if ((tab = table) != null && size > 0) {

        for (int i = 0; i < tab.length; ++i) {

            for (Node<K,V> e = tab[i]; e != null; e = e.next) {

                //如果找到value一致的返回true

                if ((v = e.value) == value ||

                    (value != null && value.equals(v)))

                    return true;

            }

        }

    }

    return false;

}
```