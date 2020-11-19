---
layout:     post
title:      Offer04.二维数组中的查找
subtitle:   Array.easy
date:       2020-04-10
author:     GJXAIOU
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - 数组
	- 完成
---

# Offer04.二维数组中的查找

## 一、题目

在一个 `n * m` 的二维数组中，**每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序**。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

- 示例:

```java
现有矩阵 matrix 如下：

[
  [1,   4,  7, 11, 15],
  [2,   5,  8, 12, 19],
  [3,   6,  9, 16, 22],
  [10, 13, 14, 17, 24],
  [18, 21, 23, 26, 30]
]
给定 target = 5，返回 true。

给定 target = 20，返回 false。
```

- 限制：

    0 <= n <= 1000

    0 <= m <= 1000

注意：本题与主站 240 题相同：https://leetcode-cn.com/problems/search-a-2d-matrix-ii/

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/er-wei-shu-zu-zhong-de-cha-zhao-lcof
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 二、解答

**注意信息**：有序，所以从左下角或者右上角进行查找，下面代码是从右上角进行查找。初始位置在左上角，如果该值比 target 大就向左移动，如果小就向下移动。**相当于一种二分查找**。

```java
package array.easy;

/**
 * @Author:GJXAIOU
 */
public class Offer04 {

    public boolean findNumberIn2DArray(int[][] matrix, int target) {
        if (matrix == null || matrix.length <= 0 || matrix[0].length <= 0) {
            return false;
        }
        int row = matrix.length;
        int column = matrix[0].length;

        // 初始位置在左上角，如果该值比 target 大就向左移动，如果小就向下移动
        int i = 0;
        int j = column - 1;
        while ((i <= row - 1) && (j >= 0)) {
            if (matrix[i][j] == target) {
                return true;
            } else if (matrix[i][j] < target) {
                i++;
            } else {
                j--;
            }
        }

        return false;
    }

}


```

