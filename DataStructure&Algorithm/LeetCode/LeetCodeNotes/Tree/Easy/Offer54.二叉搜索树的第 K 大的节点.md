---
layout:     post
title:      Offer54.二叉搜索树的第k大节点
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

# Offer54.二叉搜索树的第k大节点

## 一、题目

给定一棵二叉搜索树，请找出其中第k大的节点。

 

- 示例 1:

> 输入: root = [3,1,4,null,2], k = 1
>    3
>   / \
>  1   4
>   \
>    2
> 输出: 4

- 示例 2:

> 输入: root = [5,3,6,2,4,null,null,1], k = 3
>        5
>       / \
>      3   6
>     / \
>    2   4
>   /
>  1
> 输出: 4

- 限制：

1 ≤ k ≤ 二叉搜索树元素个数

## 二、解答

**二叉搜索树的中序遍历为递增序列**

#### 方法：二叉树中序遍历

**区别**：中序遍历选择第倒数第 K 位或者反中序遍历选第 K 位。

```java
package tree.easy;

import java.util.ArrayList;
import java.util.List;

/**
 * @Author GJXAIOU
 * @Date 2020/2/24 16:42
 */
public class Offer54 {
    static class TreeNode {
        int val;
        TreeNode left;
        TreeNode right;

        TreeNode(int x) {
            val = x;
        }
    }

    // 方法一：中序排列然后取倒数第 K 值，或者逆中序排列（右中左）然后取第 K 值。
    public int kthLargest(TreeNode root, int k) {
        if (root == null) {
            return 0;
        }
        List<Integer> resList = new ArrayList<>();
        inOder(root, resList);
        return Integer.valueOf(resList.get(resList.size() - k));
    }

    public void inOder(TreeNode root, List<Integer> resList) {
        if (root == null) {
            return;
        }
        inOder(root.left, resList);
        resList.add(root.val);
        inOder(root.right, resList);
    }

    // 方法二：计数不记值
    int count;
    int result = -1;

    public int kthLargest2(TreeNode root, int k) {
        count = k;
        kthLargest(root);
        return result;
    }

    public void kthLargest(TreeNode root) {
        if (root != null) {
            kthLargest(root.right);
            if (count == 1) {
                result = root.val;
                count--;
                return;
            }
            count--;
            kthLargest(root.left);
        }
    }
}
```

