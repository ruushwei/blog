---
title: 排序算法
date: 2019-07-01T12:54:24+02:00
tags: 
- 面试
- 算法
categories: 算法
---

#### 1. 归并排序

##### 基本思想  
归并排序的主要思想是分治法。主要过程是：
将n个元素从中间切开，分成两部分。（左边可能比右边多1个数）
将步骤1分成的两部分，再分别进行递归分解。直到所有部分的元素个数都为1。
从最底层开始逐步合并两个排好序的数列

##### 时间复杂度  
归并排序的时间复杂度为O(nlogn)，推导过程较复杂，在此不多赘述。

##### 使用场景  
归并排序需要一个跟待排序数组同等空间的临时数组，因此，使用归并排序时需要考虑是否有空间上的限制。如果没有空间上的限制，归并排序是一个不错的选择

##### code
![](https://ipic-1252327316.cos.ap-beijing.myqcloud.com/image/merge.jpeg)

#### 2. 快速排序

##### 基本思想

快速排序（Quicksort）是对冒泡排序的一种改进。快速排序由C. A. R. Hoare在1962年提出。它的基本思想是：

从要排序的数据中取一个数为“基准数”。

通过一趟排序将要排序的数据分割成独立的两部分，其中左边的数据都比“基准数”小，右边的数据都比“基准数”大。

然后再按步骤2对这两部分数据分别进行快速排序，整个排序过程可以递归进行，以此达到整个数据变成有序序列。

该思想可以概括为：挖坑填数 + 分治法。

##### 时间复杂度

最好情况的时间复杂度为O(nlogn)，过程比较复杂，就不在此赘述。

最差情况下每一次取到的数（基准数）都是当前要比较的数中的最大/最小值，在这种情况下，每次都只能得到比上一次少1个数的子序列（即要么全比基准数大，要么全比基准小），此时相当于一个冒泡排序，比较的次数 = (n - 1) + (n - 2) + ... + 2 + 1 = (n - 1) * n / 2，此时的时间复杂度为：O(n^2)。最差情况一般出现在：待排序的数据本身已经是正序或反序排好了。

##### 使用场景

基本上在任何需要排序的场景都可以使用快速排序。虽然快速排序的最坏情况时间复杂度为O(n^2)，但是由于基本不会出现，因此可以放心的使用快速排序。在本人的电脑测试，100万的随机数字，快速排序大约耗时120毫秒。

##### code

```java
private static void quickSort(int[] arr, int left, int right) {
    if (arr == null || left >= right || arr.length <= 1)
        return;
    int mid = partition(arr, left, right);
    quickSort(arr, left, mid);
    quickSort(arr, mid + 1, right);
}

private static int partition(int[] arr, int left, int right) {
    int temp = arr[left];
    while (right > left) {
        // 先判断基准数和后面的数依次比较
        while (temp <= arr[right] && left < right) {
            --right;
        }
        // 当基准数大于了 arr[right]，则填坑
        if (left < right) {
            arr[left] = arr[right];
            ++left;
        }
        // 现在是 arr[right] 需要填坑了
        while (temp >= arr[left] && left < right) {
            ++left;
        }
        if (left < right) {
            arr[right] = arr[left];
            --right;
        }
    }
    arr[left] = temp;
    return left;
}
```