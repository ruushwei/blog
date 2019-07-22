---
title: Hashmap源码解析-remove
date: 2019-05-26T12:54:24+02:00
tags: 
- java 
- hashmap
categories: java

---

#### remove

1.找到待删除节点，也是分第一个节点就是，红黑树中找该节点，普通链表中找该节点

2.删除节点，分红黑树删除，头节点删除，中间节点删除(包括尾)

3.++modCount, --size

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



#### REMOVE

1.找到待删除节点，也是分第一个节点就是，红黑树中找该节点，普通链表中找该节点

2.删除节点，分红黑树删除，头节点删除，中间节点删除(包括尾)

3.++modCount, --size

```java
public V remove(Object key) {

    Node<K,V> e;

    return (e = removeNode(hash(key), key, null, false, true)) == null ?

        null : e.value;

}

@Override

public boolean remove(Object key, Object value) {

    //这里传入了value 同时matchValue为true

    return removeNode(hash(key), key, value, true, true) != null;

}
```



```java
final Node<K,V> removeNode(int hash, Object key, Object value,

                            boolean matchValue, boolean movable) {

    // p 是待删除节点的前置节点

    Node<K,V>[] tab; Node<K,V> p; int n, index;

    //如果哈希表不为空，则根据hash值算出的index下 有节点的话。

    if ((tab = table) != null && (n = tab.length) > 0 &&

        (p = tab[index = (n - 1) & hash]) != null) {

        //node是待删除节点

        Node<K,V> node = null, e; K k; V v;

        //如果链表头的就是需要删除的节点

        if (p.hash == hash &&

            ((k = p.key) == key || (key != null && key.equals(k))))

            node = p;//将待删除节点引用赋给node

        else if ((e = p.next) != null) {//否则循环遍历 找到待删除节点，赋值给node

            if (p instanceof TreeNode)

                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);

            else {

                do {

                    if (e.hash == hash &&

                        ((k = e.key) == key ||

                            (key != null && key.equals(k)))) {

                        node = e;

                        break;

                    }

                    p = e;

                } while ((e = e.next) != null);

            }

        }

        //如果有待删除节点node，  且 matchValue为false，或者值也相等

        if (node != null && (!matchValue || (v = node.value) == value ||

                                (value != null && value.equals(v)))) {

            if (node instanceof TreeNode)

                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);

            else if (node == p)//如果node ==  p，说明是链表头是待删除节点

                tab[index] = node.next;

            else//否则待删除节点在表中间

                p.next = node.next;

            ++modCount;//修改modCount

            --size;//修改size

            afterNodeRemoval(node);//LinkedHashMap回调函数

            return node;

        }

    }

    return null;

}
```