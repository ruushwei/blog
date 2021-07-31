---
title: 回溯算法
date: 2019-09-09 11:14:51
---

<!-- toc -->

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