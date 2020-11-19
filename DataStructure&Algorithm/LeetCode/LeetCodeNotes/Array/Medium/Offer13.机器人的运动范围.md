---
layout:     post
title:      Offer13. 机器人的运动范围
subtitle:   Array.medium
date:       2020-04-11
author:     GJXAIOU
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - 数组
	- 深度优先搜索
	- 完成
---

# Offer13. 机器人的运动范围

## 一、题目

地上有一个 `m` 行 `n` 列的方格，从坐标 `[0,0]` 到坐标 `[m-1,n-1]` 。一个机器人从坐标 `[0, 0] `的格子开始移动，它每次可以向左、右、上、下移动一格（不能移动到方格外），**也不能进入行坐标和列坐标的数位之和大于 k的格子**。例如，当 `k` 为 `18` 时，机器人能够进入方格 `[35, 37]` ，因为 `3+5+3+7=18`。但它不能进入方格 `[35, 38]`，因为 `3+5+3+8=19`。请问该机器人能够到达多少个格子？

**示例 1：**

```
输入：m = 2, n = 3, k = 1
输出：3
```

**示例 1：**

```
输入：m = 3, n = 1, k = 0
输出：1
```

**提示：**

- `1 <= n,m <= 100`
- `0 <= k <= 20`

## 二、解答

> 本题与 矩阵中的路径 类似，是**典型的矩阵搜索问题。此类问题通常可使用 深度优先搜索（DFS） 或 广度优先搜索（BFS） 解决**。在介绍 DFS / BFS 算法之前，为提升计算效率，首先讲述两项前置工作： 数位之和计算 、 搜索方向简化 。

### ==（一）各位之和计算==

- 设一数字 `x` ，向下取整除法符号 `/` ，求余符号 `%` ，则有：

    - `x % 10`：得到 `x` 的个位数字；
    - `x / 10` ： 令 `x` 的十进制数向右移动一位，即删除个位数字。

- 因此，可通过循环求得数位和 sum ，数位和计算的封装函数如下所示：

    ```java
    public static void main(String[] args){
    	int sum = 0;
        while(x != 0){
        	sum += x % 10;
            x = x / 10;
        }
        return sum;
    }
    ```

- 由于机器人每次只能移动一格（即只能从 x 运动至 $x \pm 1$），因此每次只需计算 x 到 $x \pm 1$ 的数位和增量。本题说明 $1 \leq n,m \leq 100$ ，**以下公式仅在此范围适用**。

- 数位和增量公式： 设 x 的数位和为 $s_x$ ,   x+1 的数位和为 $s_{x+1}$  ；

    - 当$ (x + 1) \odot 10 = 0$ 时： $s_{x+1} = s_x - 8$ ，例如 19, 20 的数位和分别为 10, 2 ；
    - 当 $(x + 1) \odot 10 \neq 0$ 时： $s_{x+1} = s_x + 1$ ，例如 1,2 的数位和分别为 1,2 。

> 以下代码为增量公式的三元表达式写法，将整合入最终代码中。

```java
(x + 1) % 10 != 0 ? s_x + 1 : s_x - 8
```

### （二）搜索方向简化

- 数位和特点： 根据数位和增量公式得知，**数位和每逢 进位 突变一次**。
- 解的三角形结构：
    - 根据数位和特点，矩阵中 **满足数位和的解** 构成的几何形状形如多个 **等腰直角三角形** ，每个三角形的直角顶点位于 0, 10, 20,... 等数位和突变的矩阵索引处 。
    - 三角形内的解虽然都满足数位和要求，但由于机器人每步只能走一个单元格，**而三角形间不一定是连通的，因此机器人不一定能到达，称之为 不可达解 ；同理，可到达的解称为 可达解** （本题求此解） 。
- 结论： 根据可达解的结构，易推出机器人可 仅通过向右和向下移动，访问所有可达解 。
    - 三角形内部： 全部连通，易证；
    - 两三角形连通处： 若某三角形内的解为可达解，则必与其左边或上边的三角形连通（即相交），即机器人必可从左边或上边走进此三角形。

> 图例展示了 n,m=20 ， k∈[6,19] 的可达解、不可达解、非解，以及连通性的变化。

![Offer13](Offer13.%E6%9C%BA%E5%99%A8%E4%BA%BA%E7%9A%84%E8%BF%90%E5%8A%A8%E8%8C%83%E5%9B%B4.resource/Offer13.gif)

### ==方法一：深度优先遍历 DFS==

- 深度优先搜索： 可以理解为暴力法模拟机器人在矩阵中的所有路径。DFS 通过递归，先朝一个方向搜到底，再回溯至上个节点，沿另一个方向搜索，以此类推。
- 剪枝： 在搜索中，遇到数位和超出目标值、此元素已访问，则应立即返回，称之为 可行性剪枝 。

算法解析：

- 递归参数： 当前元素在矩阵中的行列索引 i 和 j ，两者的数位和 sumI, sumJ。
- 终止条件： 当 ① 行列索引越界 或 ② 数位和超出目标值 k 或 ③ 当前元素已访问过 时，返回 0 ，代表不计入可达解。
- 递推工作：
    - 标记当前单元格 ：将索引 (i, j) 存入 Set `visited` 中，代表此单元格已被访问过。
    - 搜索下一单元格： **计算当前元素的 下、右 两个方向元素的数位和**，并开启下层递归 。
- 回溯返回值： 返回 `1 + 右方搜索的可达解总数 + 下方搜索的可达解总数`，代表从本单元格递归搜索的可达解总数。

![Offer13II](Offer13.%E6%9C%BA%E5%99%A8%E4%BA%BA%E7%9A%84%E8%BF%90%E5%8A%A8%E8%8C%83%E5%9B%B4.resource/Offer13II.gif)



- 复杂度分析：
    - 时间复杂度 O(MN)： 最差情况下，机器人遍历矩阵所有单元格，此时时间复杂度为 O(MN) 。
    - 空间复杂度 O(MN) ： 最差情况下，Set visited 内存储矩阵所有单元格的索引，使用 O(MN) 的额外空间。

```java
package array.medium;

/**
 * @author GJXAIOU
 * @create 2020/04/11 21:20
 */
public class Offer13 {
    // 方法：DFS
    boolean[][] visited;

    public int movingCount(int m, int n, int k) {
        visited = new boolean[m][n];
        // 返回的是个数即结果，不是每步的路径
        return dfs(m, n, k, 0, 0, 0, 0);
    }

    public int dfs(int m, int n, int k, int i, int j, int sumI, int sumJ) {
        // 如果越界或者该元素访问过就返回
        if (i >= m || j >= n || k < sumI + sumJ || visited[i][j]) {
            return 0;
        }
        // 做决定
        visited[i][j] = true;
        // 向下走和向右走决策
        return 1 + dfs(m, n, k, i + 1, j, (i + 1) % 10 != 0 ? sumI + 1 : sumI - 8, sumJ) + dfs(m,n, k, i,j + 1, sumI, (j + 1) % 10 != 0 ? sumJ + 1 : sumJ - 8);
    }

}

```

### 方法二：广度优先遍历 BFS

- BFS/DFS ： 两者目标都是遍历整个矩阵，不同点在于搜索顺序不同。DFS 是朝一个方向走到底，再回退，以此类推；BFS 则是按照“平推”的方式向前搜索。

- **BFS 实现： 通常利用队列实现广度优先遍历**。

    

    #### 算法解析：

    - 初始化： 将机器人初始点 (0, 0) 加入队列 queue ；
    - 迭代终止条件： queue 为空。代表已遍历完所有可达解。
    - 迭代工作：
        - 单元格出队： 将队首单元格的 索引、数位和 弹出，作为当前搜索单元格。
        - 判断是否跳过： 若 ① 行列索引越界 或 ② 数位和超出目标值 k 或 ③ 当前元素已访问过 时，执行 continue 。
        - 标记当前单元格 ：将单元格索引 (i, j) 存入 Set visited 中，代表此单元格 已被访问过 。
        - 单元格入队： 将当前元素的 下方、右方 单元格的 索引、数位和 加入 queue 。
    - 返回值： Set visited 的长度 len(visited) ，即可达解的数量。

Java 使用了辅助变量 res 统计可达解数量； Python 直接返回 Set 的元素数即可。

```java
package array.medium;

import java.util.LinkedList;
import java.util.Queue;

/**
 * @author GJXAIOU
 * @create 2020/04/11 21:20
 */
public class Offer13 {

    // 方法二：BFS
    public int movingCount2(int m, int n, int k) {
        boolean[][] visited = new boolean[m][n];
        int res = 0;
        Queue<int[]> queue = new LinkedList<int[]>();
        queue.add(new int[]{0, 0, 0, 0});
        while (queue.size() > 0) {
            int[] x = queue.poll();
            int i = x[0], j = x[1], si = x[2], sj = x[3];
            if (i >= m || j >= n || k < si + sj || visited[i][j]) {
                continue;
            }
            visited[i][j] = true;
            res++;
            queue.add(new int[]{i + 1, j, (i + 1) % 10 != 0 ? si + 1 : si - 8, sj});
            queue.add(new int[]{i, j + 1, si, (j + 1) % 10 != 0 ? sj + 1 : sj - 8});
        }
        return res;
    }
}
```

