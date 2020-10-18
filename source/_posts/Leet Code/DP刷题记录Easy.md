---
title: DP 刷题记录 Easy
date: 2017-10-11 21:28:29
tags: LeetCode
Catrgories:
	-LeetCode

---



- [ ] House robber

```java
package Classify.DP;

import org.junit.jupiter.api.Test;

public class HouseRobber {

    /**
     * 解法一：思路就是一个二维的动态规划问题，然后0代表不偷当前的房子，1就是偷
     * 那么就可以写出两个地推公式出来，这次不偷的话就是看上一次的最多的情况，然后如果这次偷那么就上一次肯定没有偷，就可以用上一次的没有偷的加上本次的
     * @param nums
     * @return
     */
    public int rob(int[] nums) {
        if (nums.length == 0) {
            return 0;
        }
        int[][] dp = new int[nums.length][2];
        dp[0][0] = 0;
        dp[0][1] = nums[0];
        int i;
        for (i = 1; i < nums.length; i++) {
            dp[i][0] = Math.max(dp[i - 1][0], dp[i - 1][1]);
            dp[i][1] = dp[i - 1][0] + nums[i];
        }
        return Math.max(dp[i-1][0], dp[i-1][1]);
    }

    /**
     * 解法二：也就是使用上面的方法然后进行降为，从二维到一维，空间复杂度成 o one
     * 观察可以发现我们每次进行递推的时候我们只用到了上一步的偷了或者没偷的情况所以我们直接就可以用两个变量来代替
     * @param nums
     * @return
     */
    public int rob_1(int[] nums){
        int[] dp=new int[2];
        dp[0] = 0;
        dp[1] = nums[0];
        int tmp;
        for (int i = 1; i < nums.length; i++) {
            tmp = dp[0];
            dp[0] = Math.max(dp[0], dp[1]);
            dp[1] = tmp + nums[i];
        }
        return Math.max(dp[0], dp[1]);
    }

    @Test
    public void fun(){
        System.out.println(rob_1(new int[]{1,2,3,43}));
    }

}
```



- [ ] Maximum Subarray

      ```java
      package Classify.DP;

      import org.junit.jupiter.api.Test;

      public class MaximumSubarray {
          public int maxSubArray(int[] A) {
              if (A == null || A.length == 0)
                  return 0;
              int global = A[0];
              int local = A[0];
              for (int i = 1; i < A.length; i++) {
                  local = Math.max(A[i], local + A[i]);
                  global = Math.max(local, global);
              }
              return global;
          }

          @Test
          public void fun() {
              System.out.println(maxSubArray(new int[]{-2, 1, -3, 4, -1, 2, 1, -5, 4}));
          }
      }
      ```



- [ ] Best Time to Buy and Sell Stock

      ```java
      package Classify.DP;

      import org.junit.jupiter.api.Test;

      public class BestTimeBuyAndSellStock {

          /**
           * 思路：这个动态规划问题就是定义 dp 数组就是当前的最大利润。那么递推式就是上一次的最大利润和今天与昨天的差值利润之间的较大的那个，如果是
           * 昨天收盘加上今天的的利润还不如仅仅今天的利润大就直接拿今天的否则就拿昨天的
           * 也就是这一行！
           * dp[1] = Math.max(dp[0] + prices[i] - prices[i - 1], prices[i] - prices[i - 1]);
           * @param prices
           * @return
           */
          public int maxProfit(int[] prices) {
              if (prices.length == 0) {
                  return 0;
              }
              int[] dp = new int[2];
              int result = 0;
              for (int i = 1; i < prices.length; i++) {
                  dp[1] = Math.max(dp[0] + prices[i] - prices[i - 1], prices[i] - prices[i - 1]);
                  dp[0] = dp[1];
                  result = Math.max(dp[1], result);
              }
              return result;
          }

          @Test
          public void fun(){
              System.out.println(maxProfit(new int[]{7, 1, 5, 3, 6, 4}));
          }
      }
      ```



- [ ] Climbing Stairs

      ```java
      package Classify.DP;

      import org.junit.jupiter.api.Test;

      public class ClimbingStairs {


          /**
           *  Time Limit Exceeded  暴力破解法
           */
          private int count = 0;

          public int climbStairs(int n) {
              find(0, n);
              return count;
          }

          private void find(int n,int level) {
              if (n == level) {
                  count++;
                  return;
              } else if (n > level) {
                  return;
              }
              find(n + 1,level);
              find(n + 2, level);
          }


          /**
           * 上面的暴力破解的方式导致超时了，其实一般来说绝大多数的 dp 问题都可以使用暴力破解的方法来解决，也就是上面的递归枚举，但是我们知道 dp 的出现就是
           * 为了记录一些东西防止重复计算的，也就是为枚举剪枝，所以说还是要用枚举，这里我们定义了 dp 的含义就是到达当前位置的方法数，而递推公式写法就是
           * 到达当前位置要么是跳两部要么是跳一步来的。所以递推公式就是
           * dp[i] = dp[i - 2] + dp[i - 1];
           * 最重要的就是首先要确定 dp 数组的含义
           * @param n
           * @return
           */

          public int climbStairs_1(int n) {
              int[] dp = new int[n+2];
              dp[0] = 0;
              dp[1] = 1;
              int i;
              for (i = 2; i <= n+1; i++) {
                  dp[i] = dp[i - 2] + dp[i - 1];
              }
              return dp[i-1];
          }


          @Test
          public void fun(){
              System.out.println(climbStairs_1(10));

          }
      }
      ```

以上的题全部都是 Easy 的题，然后也就是我们发现最后我们都可以转为一维的 dp 问题，我们都没设计二维的问题。下一篇开始就是 Mid 的题，估计主要就是类似背包问题的那一类二维的不可分解的问题。