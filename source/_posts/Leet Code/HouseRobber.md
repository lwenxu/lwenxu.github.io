---
title: HouseRobber
date: 2017-09-07 21:29:06
tags: 动态规划
categories:
    -LeetCode
---

>You are a professional robber planning to rob houses along a street. Each house has a certain amount of money stashed, the only constraint stopping you from robbing each of them is that adjacent houses have security system connected and it will automatically contact the police if two adjacent houses were broken into on the same night.
Given a list of non-negative integers representing the amount of money of each house, determine the maximum amount of money you can rob tonight without alerting the police.


题目的意思大概就是有一个数组，数组里面装的就是正整数，你要去访问各个数组，但是这些数组相邻元素不可同时访问，从而获取最大的和
### 思路
还是 dp 动态规划 ：我们首先就想当前是头还是不偷，从而推断下一步，不管别的

```java
public class HouseRobber {
    public int rob(int[] nums) {
        int no=0;
        int yes=0;
        for (int num : nums) {
            int lastNo=no;
            no=Math.max(yes,no);
            yes=lastNo+num;
        }
        return Math.max(yes, no);
    }
    public int rob1(int[] nums) {
        int[][] dp=new int[nums.length+1][2];
        for (int i = 1; i < nums.length; i++) {
            dp[i][0]=Math.max(dp[i-1][0],dp[i-1][1]);
            dp[i][1]=dp[i-1][0]+nums[i];
        }
        return Math.max(dp[nums.length][0],dp[nums.length][1]);
    }
}
```