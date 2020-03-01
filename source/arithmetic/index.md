---
title: 算法
date: 2019-09-09 11:14:51
---

<!-- toc -->


### - 单例模式

#### DCL
```java    
public class Singleton {  
    private volatile static Singleton singleton;  
    private Singleton (){}  
    public static Singleton getSingleton() {  
    if (singleton == null) {  
        synchronized (Singleton.class) {  
        if (singleton == null) {  
            singleton = new Singleton();  
        }  
        }  
    }  
    return singleton;  
    }  
}  
```

volatile 防止指令重排序

#### 枚举
```java
public enum Singleton{
    INSTANCE;
}
```

### - 排序算法，归并、快速
#### 归并
```java
public class Sort {

    public static void MergeSort(int[] arr, int low, int high)
    {
        //使用递归的方式进行归并排序，所需要的空间复杂度是O（N+logN）
        int mid = (low + high)/2;
        if(low < high)
        {
            //递归地对左右两边进行排序
            MergeSort(arr, low, mid);
            MergeSort(arr, mid+1, high);
            //合并
            merge(arr, low, mid, high);
        }
    }
    
    //merge函数实际上是将两个有序数组合并成一个有序数组
    //因为数组有序，合并很简单，只要维护几个指针就可以了
    private static void merge(int[] arr, int low, int mid, int high)
    {
        //temp数组用于暂存合并的结果
        int[] temp = new int[high - low + 1];
        //左半边的指针
        int i = low;
        //右半边的指针
        int j = mid+1;
        //合并后数组的指针
        int k = 0;
        
        //将记录由小到大地放进temp数组
        for(; i <= mid && j <= high; k++)
        {
            if(arr[i] < arr[j])
                temp[k] = arr[i++];
            else
                temp[k] = arr[j++];
        }
        
        //接下来两个while循环是为了将剩余的（比另一边多出来的个数）放到temp数组中
        while(i <= mid)
            temp[k++] = arr[i++];
        
        while(j <= high)
            temp[k++] = arr[j++];
        
        //将temp数组中的元素写入到待排数组中
        for(int l = 0; l < temp.length; l++)
            arr[low + l] = temp[l];
    }
    
}
```

#### 快排
```java
public class QuickSort implements IArraySort {

    @Override
    public int[] sort(int[] sourceArray) throws Exception {
        // 对 arr 进行拷贝，不改变参数内容
        int[] arr = Arrays.copyOf(sourceArray, sourceArray.length);

        return quickSort(arr, 0, arr.length - 1);
    }

    private int[] quickSort(int[] arr, int left, int right) {
        if (left < right) {
            int partitionIndex = partition(arr, left, right);
            quickSort(arr, left, partitionIndex - 1);
            quickSort(arr, partitionIndex + 1, right);
        }
        return arr;
    }

    private int partition(int[] arr, int left, int right) {
        // 设定基准值（pivot）
        int pivot = left;
        int index = pivot + 1;
        for (int i = index; i <= right; i++) {
            if (arr[i] < arr[pivot]) {
                swap(arr, i, index);
                index++;
            }
        }
        swap(arr, pivot, index - 1);
        return index - 1;
    }

    private void swap(int[] arr, int i, int j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
}
```

### - 二叉树的打印，前中后序遍历, 递归&非递归

#### 非递归先序遍历
```java
public List<Integer> preOrder(BinaryTree node) {

    List<Integer> list=new ArrayList<Integer>();
    if(node==null) return list;
    Stack<BinaryTree> stack= new Stack<BinaryTree>();
    while(node!=null || !stack.empty()) {
        if(node!=null){
            list.add(node.val);
            stack.push(node);
            node=node.left;
        } else {
            node=stack.pop();
            node=node.right;
        }
   } 
   return list;
}
```

#### 非递归中序遍历

```java
public List<Integer> inOrder(BinaryTree node) {

    List<Integer> list=new ArrayList<Integer>();
    if(node==null) return list;
    Stack<BinaryTree> stack= new Stack<BinaryTree>();
    while(node!=null || !stack.empty()) {
        if(node!=null){
            stack.push(node);
            node=node.left;
        } else {
            node=stack.pop();
            list.add(node.val);
            node=node.right;
        }
   } 
   return list;
}

```

#### 非递归后序遍历
```java
public List<Integer> postOrder(BinaryTree node) {

    List<Integer> list=new ArrayList<Integer>();
    if(node==null) return list;
    Stack<BinaryTree> stack= new Stack<BinaryTree>();
    while(node!=null || !stack.empty()) {
        if(node!=null){
            list.add(0, node.val);
            stack.push(node);
            node=node.right;
        } else {
            node=stack.pop();
            node=node.left;
        }
   } 
   return list;
}
```

### - 剑指offer

Java版题解: https://juejin.im/post/5d06e058e51d4510664d16d0

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

Tree
#### 022-从上往下打印二叉树 * 

```java
public class Solution {
    public ArrayList<Integer> PrintFromTopToBottom(TreeNode root) {
        ArrayList<Integer> result = new ArrayList<Integer>();
        if(root == null) return result;
        Queue<TreeNode> queue = new LinkedList<TreeNode>();
        queue.offer(root);
        while(!queue.isEmpty()) {
            TreeNode temp = queue.poll();
            result.add(temp.val);
            if(temp.left != null) queue.offer(temp.left);
            if(temp.right != null) queue.offer(temp.right);
        }
        return result;
    }
}
```

#### 060-把二叉树打印成多行 

```java
ArrayList<ArrayList<Integer> > Print(TreeNode pRoot) {
    ArrayList<Integer> list = new ArrayList<Integer>();
    if(pRoot == null){
        return list;
    }
    Queue<TreeNode> queue = new LinkedList();
    //根节点入队列
    queue.offer(pRoot);
    //当队列不是空
    while(!queue.isEmpty()){
        //放里面，记录当前节点个数
        int size  = queue.size();
        ArrayList<Integer> temp = new ArrayList();
        //遍历一次是一层
        for(int i=0;i<size;i++){
            TreeNode node = queue.poll();
            if(node.left!=null){
                queue.offer(node.left);
            }
            if(node.right!=null){
                queue.offer(node.right);
            }
            temp.add(node.val);
        }
        list.add(temp);
    }
    return list;
}
```

#### 059-按之字形顺序打印二叉树 * 

按行打印的部分，按奇偶来改变头插尾插


#### 023-二叉搜索树的后序遍历序列 * 

同非递归二叉树的后序遍历， 思路: 先序是 abc, acb, 的反转 bca即需要的后序遍历。使用栈， 与先序写法差不多，只是先push right, 再left, list.add用头插. 

#### 024-二叉树中和为某一值的路径

回溯
```java
public ArrayList<ArrayList<Integer>> FindPath(TreeNode root, int target) {
    ArrayList<ArrayList<Integer>> result = new ArrayList<>();
    if (root == null) {
        return result;
    }
    find(root, target, 0, new ArrayList<Integer>(), result);
    return result;
}

private void find(TreeNode node, int target, int sum, ArrayList<Integer> path, ArrayList<ArrayList<Integer>> result) {

    sum += node.val;
    path.add(node.val);

    if (node.left == null && node.right == null && sum == target) {
        result.add(new ArrayList<>(path));
        path.remove(path.size() - 1);
        return;
    }
    
    if (node.left != null) {
        find(node.left, target, sum, path, result);
    }
    if (node.right != null) {
        find(node.right, target, sum, path, result);
    }
    path.remove(path.size() - 1);
}
```

#### 038-二叉树的深度

递归
```java
public int TreeDepth(TreeNode root) {
    if (root == null) {
        return 0;
    }
    
    if (root.left == null && root.right == null) {
        return 1;
    }
    
    int leftDepth = TreeDepth(root.left);
    int rightDepth = TreeDepth(root.right);
    
    return 1 + Integer.max(leftDepth, rightDepth);
}
```

Stack & Queue
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

Hash Table
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

搜索算法
#### 037-数字在排序数组中出现的次数（二分查找）

```java
public class Solution {
   public int GetNumberOfK(int [] array , int k) {
       int leftIndex = -1,start=0,end=array.length-1,rightIndex=-1;
       while(start <= end)
       {
           int mid = (start+end)/2;
           if(array[mid] > k)
           {
               end = mid-1;
           }else if(array[mid] < k){
               start = mid+1;
           }else{
               leftIndex = mid;
               end = mid-1;
           }
       }
       start = 0;
       end = array.length-1;
       while(start <= end)
       {
           int mid = (start+end)/2;
           if(array[mid] > k)
           {
               end = mid-1;
           }else if(array[mid] < k){
               start = mid+1;
           }else{
               rightIndex = mid;
               start = mid+1;
           }
       }
       if(array.length == 0 || rightIndex == -1)
           return 0;
       return rightIndex-leftIndex+1;
   }
}
```

全排列
#### 027-字符串的排列（回溯）  * 

```java
public ArrayList<String> Permutation(String str) {
    ArrayList<String> res = new ArrayList<String>();
    if (str.length() < 1) {
        return res;
    }
    char[] chars = str.toCharArray();
    StringBuilder cur = new StringBuilder();
    helper(res, cur, chars, new boolean[chars.length]);
    return res;
}

private void helper(ArrayList<String> res, StringBuilder cur, char[] chars, boolean[] visited) {
    if (cur.length() == chars.length) {
        res.add(cur.toString());
        return;
    }

    for (int i=0; i < chars.length; i++) {
        char c = chars[i];
        if (visited[i]) {
            continue;
        }

        cur.append(c);
        visited[i] = true;
        helper(res, cur, chars, visited);
        cur.deleteCharAt(cur.length()-1);
        visited[i] = false;
    }
}
```

动态规划
#### 030-连续子数组的最大和

```java
public int FindGreatestSumOfSubArray(int[] array) {
	//设置指针从0开始向右移动，如果当前指针 + 前面指针代表的和为sum，计算max
	//如果sum > 0 移动 sum = sum
	//如果素sum < 0 移动 sum归零 重写跳转指针
	if(array == null){
		return 0;
	}
	int i = 0;
	int max = Integer.MIN_VALUE;
	int sum = 0;
	while(i < array.length){
		sum = array[i] + sum;
		max = max > sum ? max : sum;
		if(sum < 0){
			sum = 0;
		}
		i++;
	}
	return max;
}
```

排序

#### 029-最小的K个数(快速排序) * 

```java
import java.util.ArrayList;
public class Solution {

    public ArrayList<Integer> GetLeastNumbers_Solution1(int [] arr, int k) {
        ArrayList<Integer> kNumbers = new ArrayList<Integer>();
        if (arr == null || k <= 0 || k > arr.length) return kNumbers;
        int left = 0;
        int right = arr.length - 1;
        int index = partition(arr, left, right);

        while(index!=k-1){
            
            if(index<k-1){
                start=index+1;
                index=partition(arr, left, right);
            }else{
                end=index-1;
                index=partition(arr, left, right);
            }
        }
        for(int i=0;i<k;i++){
            kNumbers.add(arr[i]);
        }
        return kNumbers;
    }
    
    private int partition(int[] arr, int left, int right) {
        // 设定基准值（pivot）
        int pivot = left;
        int index = pivot + 1;
        for (int i = index; i <= right; i++) {
            if (arr[i] < arr[pivot]) {
                swap(arr, i, index);
                index++;
            }
        }
        swap(arr, pivot, index - 1);
        return index - 1;
    }

    private void swap(int[] arr, int i, int j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
}
```

其他算法

#### 013-调整数组顺序使奇数位于偶数前面 *

```java
public void reOrderArray(int [] array) {
    if (array == null || array.length < 2) {
        return;
    }
    int n = array.length;
    for (int i = 1; i < n; i++) {
        // 当前元素是奇数，就移动到奇数序列
        if (array[i] % 2 != 0) {
            int value = array[i];
            int cur = i;
            while (cur > 0 && (array[cur - 1] % 2 == 0)) {
                array[cur] = array[cur - 1];
                cur--;
            }
            array[cur] = value;
        }
        // 当前元素是偶数，无须移动
    }
}
```


#### 028-数组中出现次数超过一半的数字

```java
public int MoreThanHalfNum_Solution(int [] array) {
    if (array == null || array.length <= 0) {
        return 0;
    }
    int num = array[0];
    int count = 1;
    
    for (int i=1; i<array.length; i++) {
        int temp = array[i];
        if (temp == num) {
            count++;
        } else {
            count--;
        }
        
        if (count == 0) {
            num = temp;
            count = 1;
        }
    }
    count = 0;
    for (int i=0; i<array.length; i++) {
        if (array[i] == num) {
            count++;
        }
    }
    int half = array.length / 2 + 1;
    if (count >= half) {
        return num;
    }
    return 0;
}
```

#### 041-和为S的连续正数序列(滑动窗口思想) *

```java
public ArrayList<ArrayList<integer> > FindContinuousSequence(int sum) {
	ArrayList<ArrayList<integer>> p=new ArrayList<ArrayList<integer>>();
	//有两个指针，一个指向头，一个指向尾
	//因为是连续的，构成等差数列，用等差数列的求和公式
	int low=1,hight=2;
	while(low<hight) {
		int temp=(low+hight)*(hight-low+1)/2;
		//如果相等，说明这个连续的数列可以构成和为sum
		if(temp==sum) {
			ArrayList<integer> a=new ArrayList<integer>();
			for(int i=low;i<=hight;i++) {
				a.add(i);
			}
			p.add(a);
			//继续找下一组
			low++;
		}else if (temp<sum) {
			hight++;
		}else {
			low++;
		}
	}
	return p;
}
```

#### 042-和为S的两个数字(双指针思想)  *

左右双指针
```java
public ArrayList<Integer> FindNumbersWithSum(int[] array, int sum) {
    ArrayList<Integer> list = new ArrayList<Integer>();
    int l = 0, r = array.length - 1;
    while (l < r) {
        if (array[l] + array[r] == sum) {
            list.add(array[l]);
            list.add(array[r]);
            break;
        } else if (array[l] + array[r] > sum) {
            r--;
        } else {
            l++;
        }
    }
    return list;
}
```

### - leetcode

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

#### 搜索旋转排序数组
搜索一个给定的目标值，如果数组中存在这个目标值，则返回它的索引，否则返回 -1 。

```java
public int search(int[] nums, int target) {
    // 判空
    if (nums.length == 0) return -1;
    if (nums.length == 1) return nums[0] == target ? 0 : -1;

    // 得到旋转点
    int rotateIndex = getRotateIndex(nums);

    // 如果rotateIndex0, 和
    if (rotateIndex == 0) {
        return searchTarget(nums, 0, nums.length - 1, target);
    } else {
        if (target <= nums[nums.length - 1]) {
            return searchTarget(nums, rotateIndex, nums.length - 1, target);
        } else {
            return searchTarget(nums, 0, rotateIndex, target);
        }
    }
}

// 二分查找target所在位置
private int searchTarget(int[] nums, int left, int right, int target) {

    while (left <= right) {
        int mid = (left + right) / 2;
        if (nums[mid] == target) {
            return mid;
        } else {
            if (nums[mid] > target) {
                right = mid -1;
            } else {
                left = mid + 1;
            }
        }
    }
    return -1;
}

// 获取旋转点
private int getRotateIndex(int[] nums) {
    int left = 0;
    int right = nums.length - 1;

    if (nums[left] < nums[right]) {
        return 0;
    }

    while (left <= right) {
        int mid = (left + right ) / 2;
        if (nums[mid] > nums[mid+1]) {
            return mid+1;
        } else {
            if (nums[mid] < nums[left]) {
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        }
    }
    return 0;
}
```

#### 最长回文子串
```java
public String longestPalindrome(String s) {
    if (s == null || s.length() == 0) return s;
    char[] cs = s.toCharArray();
    int len = cs.length;
    String maxStr = "";

    for (int i = 0; i < len; i++) {
        String curStr = midSpreed(cs, i, i);
        String midStr = midSpreed(cs, i, i + 1);
        maxStr = curStr.length() > maxStr.length() ? curStr : maxStr;
        maxStr = midStr.length() > maxStr.length() ? midStr : maxStr;
    }
    return maxStr;
}

private String midSpreed(char[] cs, int index, int indexNext) {
    while (index >=0 && indexNext < cs.length && cs[index] == cs[indexNext]) {
        index--;
        indexNext++;
    }

    StringBuilder sb = new StringBuilder();
    for (int i = index + 1; i < indexNext; i++) {
        sb.append(cs[i]);
    }
    return sb.toString();
}
```

#### 盛最多水的容器

```java
public int maxArea(int[] height) {
    int left = 0, right = height.length - 1;
    int maxAreaSize = 0;
    int h = 0, w = 0;
    while (left < right) {
        h = Integer.min(height[left], height[right]);
        w = right - left;
        maxAreaSize = Integer.max(w * h, maxAreaSize);
        if (height[left] < height[right]) {
            left++;
        } else {
            right--;
        }
    }
    return maxAreaSize;
}
```

#### 三数之和

```java
public List<List<Integer>> threeSum(int[] nums) {
    List<List<Integer>> result = new ArrayList();
    // 排序
    Arrays.sort(nums);
    int l,r,sum;
    // 固定最左边的数字, 保证不重复
    for (int i=0; i<nums.length; i++) {

        if(i > 0 && nums[i] == nums[i-1]) continue;

        l = i+1;
        r = nums.length - 1;
        while (l < r) {
            sum = nums[i] + nums[l] + nums[r];
            if (sum == 0) {
                result.add(Arrays.asList(nums[i],nums[l],nums[r]));
                while (l < r && nums[l] == nums[l + 1]) l++;   // 去重
                while (l < r && nums[r] == nums[r - 1]) r--;   // 去重
                l++;
            } else if (sum < 0) {
                l++;
            } else {
                r--;
            }
        }
    }
    return result;
}
```

#### 电话号码的字母组合

```java
class Solution {
    
    Map<String, String> phone = new HashMap<String, String>() {{
        put("2", "abc");
        put("3", "def");
        put("4", "ghi");
        put("5", "jkl");
        put("6", "mno");
        put("7", "pqrs");
        put("8", "tuv");
        put("9", "wxyz");
    }};

    public List<String> letterCombinations(String digits) {
        List<String> result = new ArrayList<>();
        if (digits == null || digits.length() == 0) {
            return result;
        }
        helper(digits, "", result);
        return result;
    }

    public void helper(String digits, String combination, List<String> result) {
        if ("".equals(digits)) {
            result.add(combination);
            return;
        }

        String digit = digits.substring(0, 1);
        String letters = phone.get(digit);

        for (int i = 0; i < letters.length(); i++) {
            String letter = letters.substring(i, i + 1);
            helper(digits.substring(1), combination + letter, result);
        }
    }
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

#### 22. 有效括号
给出 n 代表生成括号的对数，请你写出一个函数，使其能够生成所有可能的并且有效的括号组合。

```java
public List<String> generateParenthesis(int n) {
    List<String> ans = new ArrayList();
    if (n <= 0) {
        return ans;
    }
    backtrack(ans, "" , 0, 0, n);
    return ans;
}

public void backtrack(List<String> result, String cur, int open, int close, int max) {
    if (cur.length() == max * 2) {
        result.add(cur);
        return;
    }

    if (open < max) {
        backtrack(result, cur + "(", open + 1, close, max);
    }

    if (close < open) {
        backtrack(result, cur + ")", open, close + 1, max);
    }
}
```

#### 31. 下一个排列

实现获取下一个排列的函数，算法需要将给定数字序列重新排列成字典序中下一个更大的排列。

如果不存在下一个更大的排列，则将数字重新排列成最小的排列（即升序排列）。

```java
public void nextPermutation(int[] nums) {
    if (nums.length <= 0) return;
    int len = nums.length;
    for (int i = len - 1; i >= 0; i--) {
        if (i == 0) {
            Arrays.sort(nums);
        } else {
            if (nums[i] > nums[i-1]) {
                // 说明有逆序存在, 则下一个是将i-1 转变成i-1 后面比它大的数字
                Arrays.sort(nums, i, len);
                for (int j=i;j<len;j++) {
                    if (nums[j]>nums[i-1]) {
                        swap(nums,i-1,j);
                        return;
                    }
                }
            }
        }
    }
}

private void swap(int[] nums, int i, int j) {
    int temp = nums[i];
    nums[i] = nums[j];
    nums[j] = temp;
}
```

#### 34. 在排序数组中查找元素的第一个和最后一个位置

给定一个按照升序排列的整数数组 nums，和一个目标值 target。找出给定目标值在数组中的开始位置和结束位置。
你的算法时间复杂度必须是 O(log n) 级别。

```java
public int[] searchRange(int[] nums, int target) {
    if (nums.length == 0) {
        return new int[]{-1, -1};
    }

    int leftIndex = -1;
    int rightIndex = -1;
    int left = 0, right = nums.length - 1, mid = 0;

    while (left <= right) {
        mid = (left + right) / 2;
        if (nums[mid] == target && (mid == 0 || nums[mid - 1] < nums[mid])) {
            leftIndex = mid;
            break;
        } else {
            if (nums[mid] < target) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
    }

    left = leftIndex != -1 ? leftIndex : 0;
    right = nums.length - 1;

    while (left <= right) {
        mid = (left + right) / 2;
        if (nums[mid] == target && (mid == nums.length - 1 || nums[mid + 1] > nums[mid])) {
            rightIndex = mid;
            break;
        } else {
            if (nums[mid] > target) {
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        }
    }

    int[] result = new int[]{leftIndex, rightIndex};
    return result;
}
```
#### 39. 组合总和
给定一个无重复元素的数组 candidates 和一个目标数 target ，找出 candidates 中所有可以使数字和为 target 的组合。

candidates 中的数字可以无限制重复被选取。

```java
public List<List<Integer>> combinationSum(int[] candidates, int target) {
    List<List<Integer>> result = new ArrayList();
    List<Integer> cur = new ArrayList();
    helper(result, cur, 0, target, candidates, 0);
    return result;
}

private void helper(List<List<Integer>> result, List<Integer> cur, int sum, int target, int[] candidates, int index) {
    if (sum > target) {
        return;
    }

    if (sum == target) {
        result.add(new ArrayList(cur));
        return;
    }

    for (int i = 0;i < candidates.length; i++) {
        if (i < index) {
            continue;
        }
        cur.add(candidates[i]);
        helper(result, cur, sum + candidates[i], target, candidates, i);
        cur.remove(cur.size() - 1);
    }
}
```

#### 46. 全排列

```java
public List<List<Integer>> permute(int[] nums) {
    List<List<Integer>> result = new ArrayList();
    helper(result, new ArrayList(), nums, new int[nums.length]);
    return result;
}

private void helper(List<List<Integer>> result, List<Integer> cur, int[] nums, int[] index) {
    if (cur.size() == nums.length) {
        result.add(new ArrayList(cur));
        return;
    }

    for (int i=0; i < nums.length; i++) {
        if (index[i] == 1) {
            continue;
        }
        index[i] = 1;
        cur.add(nums[i]);
        helper(result, cur, nums, index);
        index[i] = 0;
        cur.remove(cur.size() - 1);
    }
}
```

#### 62. 不同路径

一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为“Start” ）。

```java
public int uniquePaths(int m, int n) {
    int[][] res = new int[m][n];
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (i == 0)
                res[0][j] = 1;
            else if (j == 0)
                res[i][0] = 1;
            else
                res[i][j] = res[i - 1][j] + res[i][j - 1];
        }
    }
    return res[m - 1][n - 1];
}

public int uniquePaths(int m, int n) {
    int[] dp = new int[n];
    for (int i = 0; i < n; i++) {
        dp[i] = 1;
    }

    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            dp[j] = dp[j] + dp[j - 1];
        }
    }
    return dp[n - 1];
}
```

#### 二叉树层次遍历

```java
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> levels = new ArrayList<List<Integer>>();
    if (root == null) return levels;
    Queue<TreeNode> queue = new LinkedList<TreeNode>();
    queue.add(root);
    int level = 0;
    while ( !queue.isEmpty() ) {
      // start the current level
      levels.add(new ArrayList<Integer>());

      // number of elements in the current level
      int level_length = queue.size();
      for(int i = 0; i < level_length; ++i) {
        TreeNode node = queue.remove();

        // fulfill the current level
        levels.get(level).add(node.val);

        // add child nodes of the current level
        // in the queue for the next level
        if (node.left != null) queue.add(node.left);
        if (node.right != null) queue.add(node.right);
      }
      // go to next level
      level++;
    }
    return levels;
}
```

#### 114. 二叉树展开为链表

直接展开
```java
public void flatten(TreeNode root) {
    TreeNode node = root;
    while (node != null) {
        
        // 
        if (node.left == null) {
            node = node.right;
            continue;
        }

        // 找到左分支的最右节点
        TreeNode pre = node.left;
        while (pre.right != null) {
            pre = pre.right;
        }        

        // pre 替换node 右分支的位置
        pre.right = node.right;
        node.right = node.left;
        node.left = null;
        node = node.right;
    }
}
```

先序遍历
```java
public static void preOrderStack(TreeNode root) {
    if (root == null) { 
        return;
    }
    Stack<TreeNode> s = new Stack<TreeNode>();
    while (root != null || !s.isEmpty()) {
        while (root != null) {
            System.out.println(root.val);
            s.push(root);
            root = root.left;
        }
        root = s.pop();
        root = root.right;
    }
}
```

#### 139. 单词拆分

给定一个非空字符串 s 和一个包含非空单词列表的字典 wordDict，判定 s 是否可以被空格拆分为一个或多个在字典中出现的单词。

说明：

拆分时可以重复使用字典中的单词。
你可以假设字典中没有重复的单词。

```java
public boolean wordBreak(String s, List<String> wordDict) {
    return helper(s, new HashSet(wordDict), 0, new Boolean[s.length()]);
}

private boolean helper(String s, Set<String> wordDict, int start, Boolean[] menu) {
    if (start == s.length()) {
        return true;
    }

    if (menu[start] != null) {
        return menu[start];
    }

    for(int end = start + 1; end <= s.length(); end++) {
        String sub = s.substring(start, end);
        if (wordDict.contains(sub) && helper(s, wordDict, end, menu)) {
            return menu[start] = true;
        }
    }
    return menu[start] = false;
}
```

#### 152. 乘积最大子序列

给定一个整数数组 nums ，找出一个序列中乘积最大的连续子序列（该序列至少包含一个数）。

```java
public int maxProduct(int[] nums) {
    int max = nums[0];
    int min = nums[0];
    int res = nums[0];
    for(int i=1; i<nums.length;i++) {
        if (nums[i] < 0) {
            int temp = max;
            max = min;
            min = temp;
        }
        max = Integer.max(max * nums[i], nums[i]);
        min = Integer.min(min * nums[i], nums[i]);
        res = Integer.max(max, res);
    }
    return res;
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

#### 236. 二叉树的最近公共祖先

给定一个二叉树, 找到该树中两个指定节点的最近公共祖先。

```java
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    Stack<TreeNode> stack = new Stack();
    Map<TreeNode, TreeNode> son2ParentMap = new HashMap();
    stack.push(root);
    son2ParentMap.put(root, null);
    while (!son2ParentMap.containsKey(p) || !son2ParentMap.containsKey(q)) {
        TreeNode node = stack.pop();
        if (node.left != null) {
            son2ParentMap.put(node.left, node);
            stack.push(node.left);
        }
        if (node.right != null) {
            son2ParentMap.put(node.right, node);
            stack.push(node.right);
        }
    }
    Set<TreeNode> sets = new HashSet();
    while (p != null) {
        sets.add(p);
        p = son2ParentMap.get(p);
    }
    while (!sets.contains(q)) {
        q = son2ParentMap.get(q);
    }
    return q;
}
```

#### 560. 和为K的子数组

```java
public int subarraySum(int[] nums, int k) {
    int count = 0;
    for (int i=0; i< nums.length;i++) {
        int sum = 0;
        for (int j=i;j < nums.length; j++) {
            sum += nums[j];
            if (sum == k) {
                count++;
            }
        }
    }
    return count;
}
```

#### 347. 前 K 个高频元素

```java
public List<Integer> topKFrequent(int[] nums, int k) {
    // 使用字典，统计每个元素出现的次数，元素为键，元素出现的次数为值
    HashMap<Integer,Integer> map = new HashMap();
    for(int num : nums){
        if (map.containsKey(num)) {
            map.put(num, map.get(num) + 1);
            } else {
            map.put(num, 1);
            }
    }
    // 遍历map，用最小堆保存频率最大的k个元素
    PriorityQueue<Integer> pq = new PriorityQueue<>(new Comparator<Integer>() {
        @Override
        public int compare(Integer a, Integer b) {
            return map.get(a) - map.get(b);
        }
    });
    for (Integer key : map.keySet()) {
        if (pq.size() < k) {
            pq.offer(key);
        } else if (map.get(key) > map.get(pq.peek())) {
            pq.poll();
            pq.offer(key);
        }
    }
    // 取出最小堆中的元素
    List<Integer> res = new ArrayList<>();
    while (!pq.isEmpty()) {
        res.add(pq.poll());
    }
    return res;
}
```