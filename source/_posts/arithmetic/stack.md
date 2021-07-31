---
title: 栈&队列算法
date: 2019-09-09 11:14:51
---

<!-- toc -->

#### 021-栈的压入、弹出序列 * 
循环压入序列，先压入，再判断是否可以弹出，如果可以，循环弹出，更新弹出序列下标，最终如果栈正好为空，则表示成功

```java
public boolean IsPopOrder(int [] pushA,int [] popA) {
    if (pushA.length == 0 || popA.length == 0 || popA.length != pushA.length)
        return false;
    Stack<Integer> stack = new Stack<>();
    int j = 0;
    for (int i = 0; i < pushA.length; i++) {
        stack.push(pushA[i]);

        while (!stack.isEmpty() && stack.peek() == popA[j]){
            stack.pop();
            j++;
        }
    }
    return stack.isEmpty();
}
```


#### 215. 数组中的第K个最大元素

在未排序的数组中找到第 k 个最大的元素。请注意，你需要找的是数组排序后的第 k 个最大的元素，而不是第 k 个不同的元素。

```java
public int findKthLargest(int[] nums, int k) {
    if (nums.length < k) {
        return -1;
    }
    Queue<Integer> queue = new PriorityQueue<Integer>((a, b) -> a - b);
    for (int i = 0; i < nums.length; i++) {
        int num = nums[i];
        queue.offer(num);
        if (queue.size() > k) {
            queue.poll();
        }
    }
    return queue.peek();
}
```

