---
title: 剑指offer常考
date: 2019-07-01T12:54:24+02:00
tags: 
- 面试
- 算法
categories: 算法
---

[TOC]     



#### 二叉树分层打印 （mi, toutiao）
直接上之字形吧

```java
public ArrayList<ArrayList<Integer>> Print(TreeNode pRoot) {
    ArrayList<ArrayList<Integer>> result = new ArrayList();
        
    if (pRoot == null) {
        return result;
    }
        
    boolean orderAsc = true;
    BlockingQueue<TreeNode> queue = new LinkedBlockingQueue();
    queue.offer(pRoot);
        
    while (!queue.isEmpty()) {
            
        int count = queue.size();
        ArrayList<Integer> cengNodes = new ArrayList();
        for (int i=0; i<count; i++) {
            TreeNode node = queue.poll();
            if (orderAsc) {
                cengNodes.add(node.val);
            } else {
                cengNodes.add(0, node.val);
            }
                
            if (node.left != null) {
                queue.offer(node.left);
            }
                
            if (node.right != null) {
                queue.offer(node.right);
            }
        }
        orderAsc = !orderAsc;
        result.add(cengNodes);
    }
        
    return result;
}
```


#### 字符串前n位排列组合 （amazon）

```java
import java.util.ArrayList;
import java.util.TreeSet;
 
public class Solution {
    public ArrayList<String> Permutation(String str) {
        ArrayList<String> result = new ArrayList();
        if (str == null || str.length() == 0) {
            return result;
        }
        char[] chars = str.toCharArray();
        TreeSet<String> tree = new TreeSet();
        Permutation(tree, chars, 0);
        result.addAll(tree);
        return result;
    }
     
     public void Permutation(TreeSet<String> tree, char[] chars, int begin) {
         if (begin == chars.length - 1) {
             tree.add(new String(chars));
             return;
         }
          
         for (int i = begin; i< chars.length; i++) {
             swap(chars, i, begin);
             Permutation(tree, chars, begin + 1);
             swap(chars, i, begin);
         }
         
    }
     
     private void swap(char [] chs, int i, int j){
        char ch = chs[i];
        chs[i] = chs[j];
        chs[j] = ch;
    }
}
```


#### 判断序列是否为栈的合法弹出序列（toutiao）


```java
import java.util.ArrayList;
import java.util.Stack;
 
public class Solution {
    public boolean IsPopOrder(int [] pushA,int [] popA) {
         
        int n = pushA.length;
        int m = popA.length;
         
        if (m == 0 || n == 0) {
            return false;
        }
         
        Stack<Integer> stack = new Stack<>();
        int length = pushA.length;
        int popIndex = 0;
        for (int i=0; i<n; i++) {
            stack.push(pushA[i]);
            while(!stack.isEmpty() && stack.peek() == popA[popIndex]) {
                stack.pop();
                popIndex++;
            }
        }
         
        if (popIndex == n) {
            return true;
        }
        return false;
    }
}
```



