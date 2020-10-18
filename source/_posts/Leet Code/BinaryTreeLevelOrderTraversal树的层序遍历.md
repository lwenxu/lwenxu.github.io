---
title: BinaryTreeLevelOrderTraversal树的层序遍历
date: 2017-09-03 10:57:46
tags: LeetCode
categories:
    -LeetCode
---



<pre>
Given a binary tree, return the level order traversal of its nodes' values. (ie, from left to right, level by level).

For example:
Given binary tree [3,9,20,null,null,15,7],
    3
   / \
  9  20
    /  \
   15   7
return its level order traversal as:
[
  [3],
  [9,20],
  [15,7]
]
</pre>


基本思想就是每一次遍历的时候看当前的层数是不是大于已有的 list 集合数，不大于则需要拓展
否则就把获取当前层的 list 集合，然后王进放元素即可。这里需要注意的就是在这个种递归问题中
我们最重要的是考虑当前的问题，而不要考虑到下一层，否则会越来越乱。
```java
import org.junit.jupiter.api.Test;

import java.util.ArrayList;
import java.util.List;

public class BinaryTreeLevelOrderTraversal {
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> result=new ArrayList<>();

        levelOrder(root,result,0);

        return result;
    }

    private void levelOrder(TreeNode root, List<List<Integer>> result, int level){
        if (root==null){
            return;
        }
        if (level>result.size()-1){
            result.add(new ArrayList<>());
        }
        List<Integer> list=result.get(level);
        if (root!=null){
            list.add(root.val);
        }
        levelOrder(root.left,result,level+1);
        levelOrder(root.right,result,level+1);
    }

}
```
