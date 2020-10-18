---
title: PalindromicSubstrings
date: 2017-10-12 19:58:32
tags: LeetCode
categories: -leetcode
---

```java
package Classify.DP.Medium;

import org.junit.jupiter.api.Test;

public class PalindromicSubstrings {

    /**
     * 基本思路：这里的 dp 方程的每一个元素就代表我要以当前元素作为回文子串的结尾时候的回文子串的数量
     * 那么递推公式就是以上一个元素结尾时候的子串数量加上本次的结尾的子串的数量就能获得总数量了
     * 而判断当前结尾的回文子串就是判断到对称的元素，然后翻转操作做判断即可
     * @param s
     * @return
     */

    public int countSubstrings(String s) {
        int[] dp = new int[s.length()];
        dp[0] = 1;
        for (int i = 1; i < dp.length; i++) {
            dp[i] = dp[i - 1] + currentCount(s, i);
        }
        return dp[s.length() - 1];
    }

    private int currentCount(String string, int i) {
        int count = 0;
        for (int j = i; j >= 0; --j) {
            if (string.charAt(i) != string.charAt(j)) {
                continue;
            }
            if (new StringBuilder(string.substring(j, i + 1)).reverse().toString().equals(string.substring(j, i + 1))) {
                ++count;
            }
        }
        return count;
    }

    @Test
    public void fun() {
        System.out.println(countSubstrings("aaa"));
    }
} 
```

