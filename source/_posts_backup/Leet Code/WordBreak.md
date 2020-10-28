---
title: WordBreak
date: 2017-09-04 20:52:16
tags: 动态规划
categories:
    -LeetCode
---
<pre>
Given a non-empty string s and a dictionary wordDict containing a list of non-empty words, determine if s can be segmented into a space-separated sequence of one or more dictionary words. You may assume the dictionary does not contain duplicate words.

For example, given
s = "leetcode",
dict = ["leet", "code"].

Return true because "leetcode" can be segmented as "leet code".
</pre>
这道题的思路是这样的，首先设置一个比字符串长一的数组，这个数组里面多的那个一就是存放空格的位置
假设我们空格在某个位置，然后这个数组的空格前一个位置表示空格之前的数组是字典里面的内容，然后
空格后面的字串被包含在字典里那么就说明在这个字串就是字典里面的内容

这道题说白了也是一个动态规划的问题，这类问题的思想就是存储上一步骤的东西，然后根据上一步骤的东西推算出下一步骤的结论
这里的存储的上一步骤的东西就是那个布尔数组，然后根据递推公式也就是 if 判断里面的东西得出本步骤的结论

```java
import java.util.List;

public class WordBreak {
    public boolean wordBreak(String s, List<String> wordDict) {
        if (s.length() == 0) {
            return true;
        }
        boolean[] res = new boolean[s.length() + 1];
        res[0] = true;
        for (int i = 0; i < s.length(); i++) {
            StringBuilder stringBuilder = new StringBuilder(s.substring(0, i + 1));
            for (int j = 0; j <= i; j++) {
                if (res[j] && wordDict.contains(stringBuilder.toString())) {
                    res[i + 1] = true;
                    break;
                }
                stringBuilder.deleteCharAt(0);
            }
        }
        return res[s.length()];
    }
}


```
