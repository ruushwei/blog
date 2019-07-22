---
title: Hashmap源码解析-put
date: 2019-05-26T12:54:24+02:00
tags: 
- java 
- hashmap
categories: java
---


源码解析-put函数

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



**#### put**

1.构造函数后的hashmap都是空的，size=0,只有在第一次put的时候，才会调resize扩容

2.index = (n - 1) & hash 实际是做了一个取模, 因为n是2的n次方, 所以n-1二进制都是1

3.if index节点为空，则new Node

  else 哈希冲突

​    如果节点哈希值相等，key也相等，则是覆盖value操作   

​    如果节点是树, 红黑树先不深入，反正就是做了查找插入或替换

​    如果是链表小于8，就遍历，插入或替换，注插入后如果链表长度大于8了，还要做treeifyBin转换为红黑树操作

4.然后++modCount, if (++size > threshold) resize();

5.扰动函数

​    hashcode散列值分布再松散，要是只取最后几位的话，碰撞也会很严重.

​    右位移16位，正好是32bit的一半，自己的高半区和低半区做异或，就是为了混合原始哈希码的高位和低位，以此来加大低位的随机性。而且混合后的低位掺杂了高位的部分特征，这样高位的信息也被变相保留下来。

​    Peter Lawley的一篇专栏文章《An introduction to optimising a hashing strategy》里的的一个实验，实验证明：在没有扰动函数的情况下，发生了103次碰撞，接近30%。而在使用了扰动函数之后只有92次碰撞。碰撞减少了将近10%。看来扰动函数确实还是有功效的。

![](https://ipic-1252327316.cos.ap-beijing.myqcloud.com/image/sdja.jpg)



#### 源码:

```java
public V put(K key, V value) {

    //先根据key，取得hash值。 再调用上一节的方法插入节点

    return putVal(hash(key), key, value, false, true);

}

static final int hash(Object key) {

    int h;

    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);     // 扰动函数

}
```



```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,

                boolean evict) {

    //tab存放 当前的哈希桶， p用作临时链表节点  

    Node<K,V>[] tab; Node<K,V> p; int n, i;

    //如果当前哈希表是空的，代表是初始化

    if ((tab = table) == null || (n = tab.length) == 0)

        //那么直接去扩容哈希表，并且将扩容后的哈希桶长度赋值给n

        n = (tab = resize()).length;

    //如果当前index的节点是空的，表示没有发生哈希碰撞。 直接构建一个新节点Node，挂载在index处即可。

    //这里再啰嗦一下，index 是利用 哈希值 & 哈希桶的长度-1，替代模运算

    if ((p = tab[i = (n - 1) & hash]) == null)

        tab[i] = newNode(hash, key, value, null);

    else {//否则 发生了哈希冲突。

        //e

        Node<K,V> e; K k;

        //如果哈希值相等，key也相等，则是覆盖value操作

        if (p.hash == hash &&

            ((k = p.key) == key || (key != null && key.equals(k))))

            e = p;//将当前节点引用赋值给e

        else if (p instanceof TreeNode)//红黑树暂且不谈

            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);

        else {//不是覆盖操作，则插入一个普通链表节点

            //遍历链表

            for (int binCount = 0; ; ++binCount) {

                if ((e = p.next) == null) {//遍历到尾部，追加新节点到尾部

                    p.next = newNode(hash, key, value, null);

                    //如果追加节点后，链表数量》=8，则转化为红黑树

                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st

                        treeifyBin(tab, hash);

                    break;

                }

                //如果找到了要覆盖的节点

                if (e.hash == hash &&

                    ((k = e.key) == key || (key != null && key.equals(k))))

                    break;

                p = e;

            }

        }

        //如果e不是null，说明有需要覆盖的节点，

        if (e != null) { // existing mapping for key

            //则覆盖节点值，并返回原oldValue

            V oldValue = e.value;

            if (!onlyIfAbsent || oldValue == null)

                e.value = value;

            //这是一个空实现的函数，用作LinkedHashMap重写使用。

            afterNodeAccess(e);

            return oldValue;

        }

    }

    //如果执行到了这里，说明插入了一个新的节点，所以会修改modCount，以及返回null。

    //修改modCount

    ++modCount;

    //更新size，并判断是否需要扩容。

    if (++size > threshold)

        resize();

    //这是一个空实现的函数，用作LinkedHashMap重写使用。

    afterNodeInsertion(evict);

    return null;

}
```





putAll

```java
public void putAll(Map<? extends K, ? extends V> m) {

    //将另一个Map的所有元素加入表中，参数evict初始化时为false，其他情况为true

    putMapEntries(m, true);

}

//将另一个Map的所有元素加入表中，参数evict初始化时为false，其他情况为true

final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {

    //拿到m的元素数量

    int s = m.size();

    //如果数量大于0

    if (s > 0) {

        //如果当前表是空的

        if (table == null) { // pre-size

            //根据m的元素数量和当前表的加载因子，计算出阈值

            float ft = ((float)s / loadFactor) + 1.0F;

            //修正阈值的边界 不能超过MAXIMUM_CAPACITY

            int t = ((ft < (float)MAXIMUM_CAPACITY) ?

                        (int)ft : MAXIMUM_CAPACITY);

            //如果新的阈值大于当前阈值

            if (t > threshold)

                //返回一个 》=新的阈值的 满足2的n次方的阈值

                threshold = tableSizeFor(t);

        }

        //如果当前元素表不是空的，但是 m的元素数量大于阈值，说明一定要扩容。

        else if (s > threshold)

            resize();

        //遍历 m 依次将元素加入当前表中。

        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {

            K key = e.getKey();

            V value = e.getValue();

            putVal(hash(key), key, value, false, evict);

        }

    }

}
```

