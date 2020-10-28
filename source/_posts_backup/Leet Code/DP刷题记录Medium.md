---
title: DP刷题记录Medium
date: 2017-10-11 22:09:34
tags: LeetCode
Catrgories:
	-LeetCode

---

- [ ] House Robber II

      ```java
      package Classify.DP.Medium;

      import org.junit.jupiter.api.Test;



      public class HouseRobberII {
          /**
           * 思路一：这道题的意思就是成一个环，也就是偷了第一个不能头最后一个。
           * 所以我们的思路就是让他从第一个偷到倒数第二个
           * 从第二个偷到最后一个
           * 算两个的最大值
           * 思路二：这种方法就是去掉头和尾，计算中间的最大值，然后加上第一个与最后一个中的最大值
           * @param nums
           * @return
           */
          public int rob(int[] nums) {
              if (nums.length == 0) {
                  return 0;
              } else if (nums.length == 1) {
                  return nums[0];
              } else if (nums.length == 2) {
                  return Math.max(nums[0], nums[1]);
              }
              int result1;
              int result2;
              int[] dp=new int[2];
              dp[0] = 0;
              dp[1] = nums[0];
              int tmp;
              for (int i = 1; i < nums.length-1; i++) {
                  tmp = dp[0];
                  dp[0] = Math.max(dp[0], dp[1]);
                  dp[1] = tmp + nums[i];
              }
              result1 = Math.max(dp[0], dp[1]);

              dp[0] = 0;
              dp[1] = nums[1];
              for (int i = 2; i < nums.length; i++) {
                  tmp = dp[0];
                  dp[0] = Math.max(dp[0], dp[1]);
                  dp[1] = tmp + nums[i];
              }

              result2 = Math.max(dp[0], dp[1]);
              return Math.max(result1, result2);
          }

          @Test
          public void fun(){
              System.out.println(rob(new int[]{1,1,1}));
          }
      }
      ```





- [ ] 2 Keys Keyboard

      ```java
      package Classify.DP.Medium;

      import org.junit.jupiter.api.Test;

      /**
       *
           Initially on a notepad only one character 'A' is present. You can perform two operations on this notepad for each step:
          
           Copy All: You can copy all the characters present on the notepad (partial copy is not allowed).
           Paste: You can paste the characters which are copied last time.
           Given a number n. You have to get exactly n 'A' on the notepad by performing the minimum number of steps permitted. Output the minimum number of steps to get n 'A'.
          
           Example 1:
           Input: 3
           Output: 3
           Explanation:
           Intitally, we have one character 'A'.
           In step 1, we use Copy All operation.
           In step 2, we use Paste operation to get 'AA'.
           In step 3, we use Paste operation to get 'AAA'.
           Note:
           The n will be in the range [1, 1000].
       */

      public class KeysKeyboard {
          /**
           * 这题的思路就是 dp 代表的就是当前的 A 的数量,然后我们优先考虑上一步复制过的情况，因为这样能够最快的填满 n 个 A ，但是使用这种情况要注意的就是
           * 我们可能上部赋值我们最后收不了尾，所以必须有一个判断就是我们能用上一步的复制的内容填满剩下的 A ，也就是 n % dp[i - 1] == 0
           * 里面的内容的意思就是我们使用了上一步的复制的内容那么我们的字符串就直接翻倍，而且我们进行了一次复制，那么我们 count  就得加2
           * @param n
           * @return
           */
          public int minSteps(int n) {
              if (n == 1) {
                  return 0;
              }
              int[] dp = new int[1000];
              dp[0] = 1;
              int paste = 1;
              int i = 0;
              int count = 1;
              while (dp[i] <= n) {
                  ++i;
                  if (n % dp[i - 1] == 0) {
                      paste = dp[i - 1];
                      dp[i] = dp[i - 1] * 2;
                      ++count;
                      if (paste != 1) {
                          ++count;
                      }
                  } else {
                      dp[i] = dp[i - 1] + paste;
                      ++count;
                  }
                  if (dp[i] == n) {
                      return count;
                  }
              }
              return count;
          }


          @Test
          public void fun() {
              System.out.println(minSteps(10));
          }
      }
      ```

