---
title: Target Sum
date: 2017-09-27 21:32:05
tags: 动态规划
Categories:
	-LeetCode

---

> You are given a list of non-negative integers, a1, a2, ..., an, and a target, S. Now you have 2 symbols + and -. For each integer, you should choose one from + and - as its new symbol.
>
> Find out how many ways to assign symbols to make sum of integers equal to target S.
>
> Example 1:
> Input: nums is [1, 1, 1, 1, 1], S is 3.
> Output: 5
> Explanation:
>
> -1+1+1+1+1 = 3
> +1-1+1+1+1 = 3
> +1+1-1+1+1 = 3
> +1+1+1-1+1 = 3
> +1+1+1+1-1 = 3
>
> There are 5 ways to assign symbols to make the sum of nums be target 3.
> Note:
> The length of the given array is positive and will not exceed 20.
> The sum of elements in the given array will not exceed 1000.
> Your output answer is guaranteed to be fitted in a 32-bit integer.



这题主要就是给一个算式，然后让我们填符号，要是最后的算式的结果等于一个特定的数，现在发现这种阶梯方法比较好就使用了动态规划，里面具体的东西就是递归求解了。

与此类似的体还有很多就是台阶问题等等，都可以使用类似的方法求解！

```java
package LeetCode;

public class TargetSum {
    int count = 0;

    public int findTargetSumWays(int[] nums, int S) {
        calculate(nums, 0, 0, S);
        return count;
    }

    private void calculate(int[] nums,int i,int sum,int S){
        if (i == nums.length) {
            if (sum == S) {
                count++;
            }
        }else {
            calculate(nums, i + 1, sum + nums[i], S);
            calculate(nums, i + 1, sum - nums[i], S);
        }
    }
}
```



