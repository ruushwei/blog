---
title: 常见基础算法
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

#### 43. 字符串相乘

两个长度分别为 n 和 m 的数相乘，长度不会超过 n + m。
因此我们可以创建一个长度为 n + m 的数组 res 存储结果。
nums[i] * nums[j] 的结果 位于 res[i+j+1], 如果 res[i+j+1] > 10, 则进位到 res[i+j]

```java
public String multiply(String n1, String n2) {
    int n = n1.length(), m = n2.length();
    int[] res = new int[n + m];
    for (int i = n - 1; i >= 0; i--) {
        for (int j = m - 1; j >= 0; j--) {
            int a = n1.charAt(i) - '0';
            int b = n2.charAt(j) - '0';
            int r = a * b;
            r += res[i + j + 1];
            res[i + j + 1] = r % 10;
            res[i + j] += r / 10;
        }
    }
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < n + m; i++) {
        if (sb.length() == 0 && res[i] == 0) continue;
        sb.append(res[i]);
    }
    return sb.length() == 0 ? "0" : sb.toString();
}
```

