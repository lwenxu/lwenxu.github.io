---
title: SymmetricTree
date: 2017-09-03 09:40:50
tags: LeetCode
categories:
    -LeetCode
---

<pre>
Given a binary tree, check whether it is a mirror of itself (ie, symmetric around its center).

For example, this binary tree [1,2,2,3,4,4,3] is symmetric:

1
/ \
2   2
/ \ / \
3  4 4  3
But the following [1,2,2,null,3,null,3] is not:
1
/ \
2   2
\   \
3    3
</pre>

错误的解法，就是遇到多个 null 值的时候就会出问题，也就是在有的时候虽然他不对称，
但是终须遍历的结果表明他就是对称的
```java
import org.junit.jupiter.api.Test;

import java.util.ArrayList;
import java.util.List;

public class SymmetricTree {
    public boolean isSymmetric(TreeNode root) {
        List<TreeNode> list = new ArrayList<>();
        inOrderTraversal(root, list);
        int center = list.size() / 2;
        return compare(list, center);
    }

    public boolean compare(List<TreeNode> list, int center) {
        for (int i = 0, j = list.size() - 1; i < center; i++, j--) {
            if (list.get(i).val != list.get(j).val) {
                return false;
            }
        }
        return true;
    }

    public void inOrderTraversal(TreeNode root, List<TreeNode> list) {
        if (root == null) {
            return;
        }
        inOrderTraversal(root.left, list);
        list.add(root);
        inOrderTraversal(root.right, list);
    }

    @Test
    void test() {

    }
}


```

好的解法就是使用递归，所谓对称简单来说就是 root 的左孩子和右孩子要一样才行

