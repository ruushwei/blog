---
title: 动态规划算法
date: 2019-09-09 11:14:51
---

<!-- toc -->


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
