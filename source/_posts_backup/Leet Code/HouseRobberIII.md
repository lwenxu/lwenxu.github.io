---
title: HouseRobberIII
date: 2017-09-27 22:12:07
tags: 动态规划
Categories:
	-LeetCode

---

The thief has found himself a new place for his thievery again. There is only one entrance to this area, called the "root." Besides the root, each house has one and only one parent house. After a tour, the smart thief realized that "all houses in this place forms a binary tree". It will automatically contact the police if two directly-linked houses were broken into on the same night.

Determine the maximum amount of money the thief can rob tonight without alerting the police.

Example 1:

```
     3
    / \
   2   3
    \   \ 
     3   1

```

Maximum amount of money the thief can rob = 3 + 3 + 1 = 7.

 

Example 2:

```
     3
    / \
   4   5
  / \   \ 
 1   3   1

```

Maximum amount of money the thief can rob = 4 + 5 = 9.



所以说纯粹是怎么多怎么来，那么这种问题是很典型的递归问题，我们可以利用回溯法来做，因为当前的计算需要依赖之前的结果，那么我们对于某一个节点，如果其左子节点存在，我们通过递归调用函数，算出不包含左子节点返回的值，同理，如果右子节点存在，算出不包含右子节点返回的值，那么此节点的最大值可能有两种情况，一种是该节点值加上不包含左子节点和右子节点的返回值之和，另一种是左右子节点返回值之和不包含当期节点值，取两者的较大值返回即可：

```java
//直接使用递归
public int rob(TreeNode root) {
    if (root == null) {
        return 0;
    }
    int var = 0;
    if (root.left != null) {
        var += rob(root.left.left) + rob(root.left.right);
    }
    if (root.right != null) {
        var += rob(root.right.left) + rob(root.right.right);
    }
    return Math.max(root.val + var, rob(root.right) + rob(root.left));
}
```

 

由于上面的方法超时了，所以我们必须优化上面的方法，上面的方法重复计算了很多地方，比如要完成一个节点的计算，就得一直找左右子节点计算，我们可以把已经算过的节点用哈希表保存起来，以后递归调用的时候，现在哈希表里找，如果存在直接返回，如果不存在，等计算出来后，保存到哈希表中再返回，这样方便以后再调用，参见代码如下：

 ```java
    //使用hash 表对多余的递归进行优化
    public int rob_1(TreeNode root) {
        HashMap<TreeNode, Integer> map = new HashMap<>();
        return dfs(root, map);
    }

    private int dfs(TreeNode root, HashMap<TreeNode, Integer> map) {
        if (root == null) {
            return 0;
        }
        if (map.containsKey(root)) {
            return map.get(root);
        }
        int var = 0;
        if (root.left != null) {
            var += dfs(root.left.right, map) + dfs(root.left.left, map);
        }
        if (root.right != null) {
            var += dfs(root.right.left, map) + dfs(root.right.right, map);
        }
        int max = Math.max(var + root.val, dfs(root.left, map) + dfs(root.right, map));
        map.put(root, max);
        return max;
    }
 ```





下面再来看一种方法，这种方法的递归函数返回一个大小为2的一维数组res，其中res[0]表示不包含当前节点值的最大值，res[1]表示包含当前值的最大值，那么我们在遍历某个节点时，首先对其左右子节点调用递归函数，分别得到包含与不包含左子节点值的最大值，和包含于不包含右子节点值的最大值，那么当前节点的res[0]就是左子节点两种情况的较大值加上右子节点两种情况的较大值，res[1]就是不包含左子节点值的最大值加上不包含右子节点值的最大值，和当前节点值之和，返回即可，参见代码如下：

```Java
public int rob_2(TreeNode root) {
        int[] result = dfs_1(root);
        return Math.max(result[0], result[1]);
    }

    private int[] dfs_1(TreeNode root) {
        if (root == null) {
            return new int[]{0, 0};
        }
        int[] left = dfs_1(root.left);
        int[] right = dfs_1(root.right);
        return new int[]{ Math.max(left[0], left[1]) + Math.max(right[0], right[1]),root.val + left[0] + right[0]};
    }
```







整理于：http://www.cnblogs.com/grandyang/p/5275096.html