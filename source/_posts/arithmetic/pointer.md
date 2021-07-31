---
title: 指针算法
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