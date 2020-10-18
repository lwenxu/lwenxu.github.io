---
title: QueueReconstructionbyHeight
date: 2017-09-27 21:29:51
tags: 排序
Categories:
	-LeetCode

---

> Suppose you have a random list of people standing in a queue. Each person is described by a pair of integers (h, k), where h is the height of the person and k is the number of people in front of this person who have a height greater than or equal to h. Write an algorithm to reconstruct the queue.
>
> Note:
> The number of people is less than 1,100.
>
> Example
>
> Input:
> [[7,0], [4,4], [7,1], [5,0], [6,1], [5,2]]
>
> Output:
> [[5,0], [7,0], [5,2], [6,1], [4,4], [7,1]]



 这道题给了我们一个队列，队列中的每个元素是一个pair，分别为身高和前面身高不低于当前身高的人的个数，让我们重新排列队列，使得每个pair的第二个参数都满足题意。

 基本思路：
 首先按照高度排序，如果两个人的高度一样的话我们就就按照第二个数字排序
 最后把他们插入到一个新的数组里面，就按照他们的第二个数字的下标，插入到对应的下标


```java
package LeetCode;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.Comparator;
import java.util.List;



public class QueueReconstructionbyHeight {
    public int[][] reconstructQueue(int[][] people) {
        if (people == null || people.length == 0) {
            return people;
        }
        Arrays.sort(people, new Comparator<int[]>() {
            @Override
            public int compare(int[] p1, int[] p2) {
                return p1[0] == p2[0] ? p1[1] - p2[1] : p2[0] - p1[0];
            }
        });
        List<int[]> temp = new ArrayList<>();
        for (int[] aPeople : people) {
            if (people.length == aPeople[1]) {
                temp.add(aPeople);
            } else {
                temp.add(aPeople[1], aPeople);
            }
        }
        int ans[][] = new int[people.length][2];
        for (int i = 0; i < temp.size(); i++) {
            ans[i] = temp.get(i);
        }
        return ans;
    }
}
```