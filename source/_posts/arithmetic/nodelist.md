---
title: 链表算法
date: 2019-09-09 11:14:51
---

<!-- toc -->


LinkedList   
#### 014-链表中倒数第k个结点
双指针，快指针先走k-1步，然后快慢指针一起走，当快指针走到最后一个节点，慢指针指向倒数第K个节点
```java
public class Solution {
    public ListNode FindKthToTail(ListNode head,int k) {
        if(head == null || k ==0 ){
            return null;
        }
        ListNode slow=head;
        ListNode fast=head;
        for(int i=0;i<k;i++){
            if(fast == null){
                return null;
            }
            fast=fast.next;
        }
        while(fast!=null){
            slow=slow.next;
            fast=fast.next;
        }
        return slow;
    }
}
```


#### 015-反转链表

1. 无需额外空间，依次反转，三个指针
2. 用栈

```java
public ListNode reverseList(ListNode head) {
        // 判断链表为空或长度为1的情况
    if(head == null || head.next == null){
        return head;
    }
    ListNode pre = null; // 当前节点的前一个节点
    ListNode cur = head;
    ListNode next = null; // 当前节点的下一个节点
    while( cur != null){
        next = cur.next; // 记录当前节点的下一个节点位置；
        cur.next = pre; // 让当前节点指向前一个节点位置，完成反转
        pre = cur; // pre 往右走
        cur = next;// 当前节点往右继续走
    }
    return pre;
}
```

#### 016-合并两个有序链表

和归并一样
```java
public ListNode Merge(ListNode list1, ListNode list2) {
    if (list1 == null) {
        return list2;
    } 
    if (list2 == null) {
        return list1;
    }
    
    ListNode headNode = new ListNode(-1);
    ListNode pNode = headNode;
    pNode.next = null;
    
    while (list1 != null && list2 != null) {
        if (list1.val <= list2.val) {
            pNode.next = list1;
            pNode = list1;
            list1 = list1.next;
        } else {
            pNode.next = list2;
            pNode = list2;
            list2 = list2.next;
        }
    }
    
    if (list1 != null) {
        pNode.next = list1;
    }
    if (list2 != null) {
        pNode.next = list2;
    }
    return headNode.next;
}
```

#### 036-两个链表的第一个公共结点
思路1： 长的先走k步, 再快慢一起
思路2： 两个指针从两个链表走，如果一个走完就走另一个，这样先走短的指针会先走长的k步，其实第二条的时候后半部分是一样的，前面的长度相同，所以这时候走到后面肯定是一样的.
```java
public ListNode FindFirstCommonNode(ListNode pHead1, ListNode pHead2) {
    if (pHead1 == null || pHead2 == null) return null;
    ListNode p1 = pHead1;
    ListNode p2 = pHead2;
    while (p1 != p2) {
        p1 = p1 == null ? pHead2 : p1.next;
        p2 = p2 == null ? pHead1 : p2.next;
    }
    return p1;
}
```

#### 055-链表中环的入口结点

先快慢节点，确定有环，并找到相交节点，再从相交节点和开始节点同时开始，当相等的时候，就是环的入口节点。

```java
public ListNode entryNodeOfLoop(ListNode pHead) {
    if(pHead==null|| pHead.next==null|| pHead.next.next==null) return null;
    ListNode fast=pHead.next.next;
    ListNode slow=pHead.next;
    /////先判断有没有环
    while(fast!=slow){
        if(fast.next!=null&& fast.next.next!=null){
            fast=fast.next.next;
            slow=slow.next;
        }else{
            //没有环,返回
            return null;
        }
    }
    //循环出来的话就是有环，且此时fast==slow.
    fast = pHead;

    while(fast!=slow){
        fast=fast.next;
        slow=slow.next;
    }

    return slow;
}
```

#### 056-删除链表中重复的结点

假的头结点，三个指针，pre、cur、next, 判断是相等去掉中间重复 还是 整体后移
```java
public ListNode deleteDuplication(ListNode pHead)
{
    ListNode head = new ListNode(-1);
    head.next = pHead;
    ListNode pre = head;
    ListNode cur = head.next;
    while (cur != null) {
        if (cur.next != null && cur.val == cur.next.val) {
            while (cur.next != null && cur.val == cur.next.val) {
                cur = cur.next;
            }
            pre.next = cur.next;
            cur = cur = cur.next;
        } else {
            pre = pre.next;
            cur = cur.next;
        }
    }
    return head.next;
}
```


#### 19. 删除链表的倒数第N个节点

```java
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode first = dummy;
    ListNode second = dummy;

    for (int i = 0; i < n + 1; i++) {
        first = first.next;
        if (first == null) {
            return dummy;
        }
    }

    while (first != null) {
        first = first.next;
        second = second.next;
    }
    second.next = second.next.next;
    return dummy.next;
}
```