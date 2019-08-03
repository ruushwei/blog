---
title: 最长公共考点
date: 2019-07-01T12:54:24+02:00
tags: 
- 面试
- 算法
categories: 算法
---

<!-- toc -->   

#### 最长无重复子串

https://www.nowcoder.com/discuss/196946

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        int res = 0;
        Map<Character, Integer> map = new HashMap();
        char[] chars = s.toCharArray();
        int len = chars.length;
        
        for (int left=0, j=0; left<len && j<len; j++) {
            if (map.containsKey(chars[j])) {
                left = Math.max(left, map.get(chars[j]) + 1);
            }
            map.put(chars[j], j);
            res = Math.max(res, j-left+1);
        }
        return res;
    }
}
```

#### 最长公共子串



#### 最长公共子序列



