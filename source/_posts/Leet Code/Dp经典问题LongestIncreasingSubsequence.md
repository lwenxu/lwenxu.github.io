---
title: Dp经典问题LongestIncreasingSubsequence
date: 2017-09-27 21:26:51
tags: 动态规划
Categories:
	-LeetCode

---





> Given an unsorted array of integers, find the length of longest increasing subsequence.
>
>  For example,
>  Given [10, 9, 2, 5, 3, 7, 101, 18],
>  The longest increasing subsequence is [2, 3, 7, 101], therefore the length is 4. Note that there may be more than one LIS combination, it is only necessary for you to return the length.
>
>  Your algorithm should run in O(n2) complexity.
>
>  Follow up: Could you improve it to O(n log n) time complexity?
>
>  Credits:
>  Special thanks to @pbrother for adding this problem and creating all test cases.
>

 思路：
 这道题主要考察的就是动态规划
 如果使用暴力破解的方式要么就是超时要么就是空间超出限制
 然后动态规划的方法就是：使用一个 dp 数组然后在这个 dp 数组记录的内容就是党目前为止的最长的子序列

![](http://ojrw3x8j2.bkt.clouddn.com/17-9-27/81514595.jpg)

```java


package LeetCode;

import org.junit.jupiter.api.Test;

import java.util.Arrays;



public class LongestIncreasingSubsequence {
    public int lengthOfLIS(int[] nums) {
        if (nums.length == 0) {
            return 0;
        }
        int[] dp = new int[nums.length];
        Arrays.fill(dp,0);
        dp[0] = 1;
        int maxans = 1;
        for (int i = 1; i < dp.length; i++) {
            int maxval = 0;
            for (int j = 0; j < i; j++) {
                if (nums[i] > nums[j]) {
                    maxval = Math.max(maxval, dp[j]);
                }
            }
            dp[i] = maxval + 1;
            maxans = Math.max(maxans, dp[i]);
        }
        System.out.println(LeetcodeUtils.integerArrayToString(dp));;
        return maxans;
    }

    @Test
    public void fun(){
        lengthOfLIS(new int[]{4,10,4,3,8,9});
    }
}
```