---
layout:     post
title:      Offer55-I:二叉树的深度
subtitle:   Tree.easy
date:       2020-02-24
author:     GJXAIOU
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - 栈
	- 深度优先搜索
	- 完成
---

# Offer55-I：二叉树的深度

## 一、题目

输入一棵二叉树的根节点，求该树的深度。从根节点到叶节点依次经过的节点（含根、叶节点）形成树的一条路径，最长路径的长度为树的深度。

例如：

给定二叉树 [3,9,20,null,null,15,7]，

    	3
       / \
      9  20
        /  \
       15   7
    返回它的最大深度 3 。
- 提示：
    - 节点总数 <= 10000
    - 注意：本题与主站 104 题相同：https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/

## 二、解答

整棵树的高度为左子树的高度和右子树高度的最大值 + 1

```java
package tree.easy;

/**
 * @Author GJXAIOU
 * @Date 2020/2/24 13:39
 */
public class Offer55 {
    class TreeNode {
        int val;
        TreeNode left;
        TreeNode right;

        TreeNode(int x) {
            val = x;
        }
    }

    public int maxDepth(TreeNode root) {

        if (root == null) {
            return 0;
        }
        int leftHeight = 0;
        int rightHeight = 0;
        if (root.left != null) {
            leftHeight = maxDepth(root.left);
        }
        if (root.right != null) {
            rightHeight = maxDepth(root.right);
        }
        return Math.max(leftHeight, rightHeight) + 1;
    }

}

```

