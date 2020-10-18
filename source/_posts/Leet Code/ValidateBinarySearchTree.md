---
title: ValidateBinarySearchTree
date: 2017-09-02 11:16:10
tags:
---
import org.junit.jupiter.api.Test;
import org.omg.CORBA.INTERNAL;

<pre>
   Given a binary tree, determine if it is a valid binary search tree (BST).

     Assume a BST is defined as follows:

     The left subtree of a node contains only nodes with keys less than the node's key.
     The right subtree of a node contains only nodes with keys greater than the node's key.
     Both the left and right subtrees must also be binary search trees.
     Example 1:
     2
     / \
     1   3
     Binary tree [2,1,3], return true.
     Example 2:
     1
     / \
     2   3
     Binary tree [1,2,3], return false.
</pre>

错误解法，只判断了每一个小的子树是搜索二叉树，而忽略了整个 root 的情况

```java
public class ValidateBinarySearchTree {

    public boolean isValidBST(TreeNode root) {
        if (root==null){
            return true;
        }
        if (root.left==null&&root.right==null){
            return true;
        }
        int leftValue=(root.left==null)?Integer.MIN_VALUE:root.left.val;
        int rightValue=(root.right==null)? Integer.MAX_VALUE:root.right.val;
        if(leftValue<root.val&&rightValue>root.val){
            return isValidBST(root.left)&&isValidBST(root.right);
        }
        return false;
    }

    @Test
    void test(){
        TreeNode root=new TreeNode(0);
        TreeNode left=new TreeNode(-1);
        TreeNode right=new TreeNode(3);
        root.left=left;
        // root.right=right;
        System.out.println(isValidBST(root));
    }
}
```

通过的代码就是使用的搜索二叉树的中序遍历，他的终须遍历的话刚好出来结果就是
左边的树必须小于右边的数，这样就能很轻松的判断出来是否合法
```java
public class ValidateBinarySearchTree {

    public boolean isValidBST(TreeNode root) {
        if (root==null){
            return true;
        }
        if (root.left==null&&root.right==null){
            return true;
        }
        List<TreeNode> list=new ArrayList<>();
        inOrderTraversal(root,list);
        for (int i = 1; i < list.size(); i++) {
            if (list.get(i).val <= list.get(i - 1).val){
                return false;
            }
        }
        return true;
    }

    public void inOrderTraversal(TreeNode root,List<TreeNode> list){
        if (root==null){
            return ;
        }
        inOrderTraversal(root.left,list);
        list.add(root);
        inOrderTraversal(root.right,list);
    }

    @Test
    void test(){
        TreeNode root=new TreeNode(0);
        TreeNode left=new TreeNode(-1);
        TreeNode right=new TreeNode(3);
        root.left=left;
        // root.right=right;
        System.out.println(isValidBST(root));
    }
}

```

以及终须遍历衍生出来的新方法，也就是不使用一个 list 集合来存储前后之间的关系

```java
class ValidateBinarySearchTree2 {

    TreeNode pre=null;
    public boolean isValidBST(TreeNode root) {
        //这是一个中序遍历。然后我们在每次遍历的时候都存储了前一个元素，避免了使用一个 list 集合
        if (root!=null){
            //首先遍历左子树，左子树出问题直接 false
            if (!isValidBST(root.left)){
                return false;
            }
            //核心判断条件，是不是前面的数大于后面的数
            if (pre!=null&&pre.val>=root.val){
                return false;
            }
            //最后就是右子树，右子树由于是最后一道所以可以直接返回他的结果
            return isValidBST(root.right);
        }
        return true;
    }


    @Test
    void test(){
        TreeNode root=new TreeNode(0);
        TreeNode left=new TreeNode(-1);
        TreeNode right=new TreeNode(3);
        root.left=left;
        // root.right=right;
        System.out.println(isValidBST(root));
    }
}
```
