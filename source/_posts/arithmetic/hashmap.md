---
title: hashmap算法
date: 2019-09-09 11:14:51
---

<!-- toc -->


#### 034-第一个只出现一次的字符

1.使用hashmap
2.indexof lastindexof

```java
public class Solution {
    public int FirstNotRepeatingChar(String str) {
        for (int i = 0; i < str.length(); i++) {
            char t = str.charAt(i);
            if (str.indexOf(t) == i && str.lastIndexOf(t) == i){
                return i;
            }
        }
        return -1;
    }
}
```

#### 03-无重复字符的最长子串 * 

```java
public int lengthOfLongestSubstring(String s) {
    char[] chars = s.toCharArray();
    int len = chars.length;
    int left =0, right = 0;
    int max = 0;
    Integer index;
    Map<Character, Integer> indexMap = new HashMap();
    for (;right < len;right ++) {
        char c = chars[right];
        if ((index = indexMap.get(c)) != null && index >= left) {
            left = index + 1;
        }
        indexMap.put(c, right);
        max = Math.max(max, right - left + 1);
    }
    return max;
}
```
#### 146.实现lru

```java
public class LRUCache {

    class DNode {
        int key;
        int val;
        DNode pre;
        DNode next;
    }

    private DNode head;
    private DNode tail;
    private HashMap<Integer, DNode> cache = new HashMap<Integer, DNode>();
    private int capacity;
    private int size;

    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.size = 0;
        head = new DNode();
        tail = new DNode();
        head.next = tail;
        tail.pre = head;
    }
    
    public int get(int key) {
        // cache
        DNode node = cache.get(key);
        if (node == null) return -1;
        // 移到最前面
        moveToHead(node);
        return node.val;
    }
    
    public void put(int key, int value) {
        DNode node = cache.get(key);
        if (node != null) {
            node.val = value;
            moveToHead(node);
        } else {
            node = new DNode();
            node.key = key;
            node.val = value;
            cache.put(key, node);
            addFirst(node);
            size++;

            if (size > capacity) {
                cache.remove(tail.pre.key);
                removeNode(tail.pre);
                size--;
            }
        }
    }

    private void moveToHead(DNode node) {
        // 删除节点
        removeNode(node);
        // 放到首节点后面
        addFirst(node);
    }

    private void removeNode(DNode node) {
        DNode pre = node.pre;
        DNode next = node.next;
        pre.next = next;
        next.pre = pre;
    }

    private void addFirst(DNode node) {
        DNode next = head.next;
        node.next = next;
        node.pre = head;

        head.next = node;
        next.pre = node;
    }
}
```