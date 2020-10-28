---
title: UniqueBinarySearchTreesII
date: 2017-09-02 10:25:20
tags: LeetCode
categories:
    -LeetCode
---

import org.junit.jupiter.api.Test;

题目的描述如下
<pre>
 * Given n, how many structurally unique BST's (binary search trees) that store values 1...n?

     For example,
     Given n = 3, there are a total of 5 unique BST's.

     1         3     3      2      1
     \       /     /      / \      \
     3     2     1      1   3      2
     /     /       \                 \
     2     1         2                 3
</pre>

也就是让生成搜索二叉树的所有可能情况，用一种遍历方式放在 list 集合中展示出来

这个地方的基本思路就是把这个问题分解成子问题，也就是先找出一个 root 节点，然后分别对左右的子树进行同样的查找操作，
这里的代码比较巧妙的一点就是他一开始存放的并不是已经遍历过的树，而是只存放了这个根结点，根结点的左右子树都放在了根结点的
内部了，所以最后才会有一个双重循环说白了就是循环的是当选这个 root 节点的时候左右子树的所有情况


```java
class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;

    TreeNode(int x) {
        val = x;
    }
}

public class UniqueBinarySearchTreesII {
    public List<TreeNode> generateTrees(int n) {
        if (n < 1) {
            return new ArrayList<TreeNode>();
        }
        return generate(1, n);
    }

    public List<TreeNode> generate(int start, int end) {
        List<TreeNode> lists = new ArrayList<>();
        if (start > end) {
            lists.add(null);
            return lists;
        }
        for (int i = start; i <= end; i++) {
            List<TreeNode> leftTree = generate(start, i - 1); //里面的存储结构其实就是只存了一个子树的 root 节点然后，root 节点里面带的有左右子节点的关系
            List<TreeNode> rightTree = generate(i+1, end);
            for (TreeNode leftNode : leftTree) {
                for (TreeNode rightNode : rightTree) {
                    TreeNode root = new TreeNode(i);
                    root.left = leftNode;
                    root.right = rightNode;
                    lists.add(root);
                }
            }
        }
        return lists;
    }

    @Test
    void fun() {
        // generateTrees(3);
        System.out.println(Arrays.toString(generateTrees(3).toArray()));
    }
}
```



