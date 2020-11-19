---
layout:     post
title:      Offer47.礼物的最大价值
subtitle:   Array.medium
date:       2020-02-09
author:     GJXAIOU
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - 数组
	- 深度优先搜索
	- 动态规划
	- 完成
---



# Offer47. 礼物的最大价值

## 一、题目

在一个 m*n 的棋盘的每一格都放有一个礼物，每个礼物都有一定的价值（价值大于 0）。你可以从棋盘的左上角开始拿格子里的礼物，并每次向右或者向下移动一格、直到到达棋盘的右下角。给定一个棋盘及其上面的礼物的价值，请计算你最多能拿到多少价值的礼物？

 

**示例 1:**

```
输入: 
[
  [1,3,1],
  [1,5,1],
  [4,2,1]
]
输出: 12
解释: 路径 1→3→5→2→1 可以拿到最多价值的礼物
```

提示：

- `0 < grid.length <= 200`
- `0 < grid[0].length <= 200`



## 二、解答

### 一、 DFS

运行超时

```java
package array.medium;

/**
 * @author GJXAIOU
 * @create 2020/04/12 13:54
 */
public class Offer47 {
    // 方法一：DFS
    // 这里没有限定只能访问一次,所以不必要使用 boolean[][] visited;
    public int maxValue(int[][] grid) {
        int cow = grid.length;
        int column = grid[0].length;

        return dfs(grid, 0, 0, cow, column, 0);
    }

	// 结果是一个确定的值，所以使用 int 返回即可。
    public static int dfs(int[][] grid, int i, int j, int cow, int column, int res) {
        if (i >= cow || j >= column) {
            return res;
        }
        res += grid[i][j];
        return Math.max(dfs(grid, i + 1, j, cow, column, res),
                dfs(grid, i, j + 1, cow, column, res));
        // 因为是 DFS，不是回溯，所有不用恢复
    }
}
```



### 二、动态规划

> 题目说明：从棋盘的左上角开始拿格子里的礼物，并每次向右或者向下移动一格、直到到达棋盘的右下角。
> 根据题目说明，易得某单元格只可能从上边单元格或左边单元格到达。

- 设 `f(i,j)` 为从棋盘左上角走至单元格 `(i ,j)` 的礼物最大累计价值，易得到以下递推关系：`f(i,j)` 等于 `f(i,j-1)` 和 `f(i−1,j)` 中的较大值加上当前单元格礼物价值 `grid(i,j)` 。
    $f(i,j) = \max[f(i,j-1), f(i-1,j)] + grid(i,j)$

- 因此，可用动态规划解决此问题，以上公式便为此问题的转移方程。

![image-20200412134454685](Offer47.%E7%A4%BC%E7%89%A9%E7%9A%84%E6%9C%80%E5%A4%A7%E4%BB%B7%E5%80%BC.resource/image-20200412134454685.png)

**动态规划解析：**

- 状态定义： 设动态规划矩阵 dp ，**dp(i,j) 代表从棋盘的左上角开始，到达单元格 (i,j) 时能拿到礼物的最大累计价值。**
- 转移方程：
    - 当 i = 0 且 j = 0 时，为起始元素；
    - 当 i = 0 且 $j \ne 0$ 时，为矩阵第一行元素，只可从左边到达；
    - 当 $i \ne 0$ 且 $j = 0$ 时，为矩阵第一列元素，只可从上边到达；
    - 当 $i \ne 0$ 且 $j \ne 0$ 时，可从左边或上边到达；
        $dp(i,j)= \begin{cases} grid(i,j) & {,i=0, j=0}\\ grid(i,j) + dp(i,j-1) & {,i=0, j \ne 0}\\ grid(i,j) + dp(i-1,j) & {,i \ne 0, j=0}\\ grid(i,j) + \max[dp(i-1,j),dp(i,j-1)]& ,{i \ne 0, j \ne 0} \end{cases}$

- 初始状态： `dp[0][0] = grid[0][0]` ，即到达单元格 (0,0) 时能拿到礼物的最大累计价值为 `grid[0][0]` ；
- 返回值： `dp[m-1][n-1]` ，m,n 分别为矩阵的行高和列宽，即返回 dp 矩阵右下角元素。

**空间复杂度降低：**

- 由于 `dp[i][j]` 只与 `dp[i-1][j]dp[i−1][j]` ,  `grid[i][j]` 有关系，因此可以将原矩阵 grid 用作 dp 矩阵，即直接在 grid 上修改即可。
- 应用此方法可省去 dp 矩阵使用的额外空间，因此空间复杂度从 O(MN) 降至 O(1) 。

![Offer47](Offer47.%E7%A4%BC%E7%89%A9%E7%9A%84%E6%9C%80%E5%A4%A7%E4%BB%B7%E5%80%BC.resource/Offer47.gif)

**复杂度分析：**

- 时间复杂度 O(MN) ： M,N 分别为矩阵行高、列宽；动态规划需遍历整个 grid 矩阵，使用 O(MN) 时间。

- 空间复杂度 O(1) ： 原地修改使用常数大小的额外空间。

**代码：**

>  先初始化矩阵第一行，再从矩阵第二行开始遍历递推，即可避免以上问题。

```java
package array.medium;

/**
 * @author GJXAIOU
 * @create 2020/04/12 13:54
 */
public class Offer47 {
    public int maxValue(int[][] grid) {
        return dp(grid, 0, 0);
    }

    public int dp(int[][] grid, int i, int j) {
        // 为 DP 数组赋初始值，第一行和第一列
        int dp[][] = new int[grid.length + 1][grid[0].length + 1];
        dp[0][0] = grid[0][0];
        for (int k = 1; k < grid.length; k++) {
            dp[k][0] = dp[k - 1][0] + grid[k][0];
        }
        for (int m = 1; m < grid[0].length; m++) {
            dp[0][m] = dp[0][m - 1] + grid[0][m];
        }

        for (i = 1; i < grid.length; i++) {
            for (j = 1; j < grid[0].length; j++) {
                dp[i][j] = Math.max(dp[i][j - 1], dp[i - 1][j]) + grid[i][j];
            }
        }
        return dp[grid.length - 1][grid[0].length - 1];
    }
}

```

