---
layout:     post
title:      Offer12. 矩阵中的路径
subtitle:   Array.easy
date:       2020-04-10
author:     GJXAIOU
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - 数组
	- 回溯
	- 完成
---

# Offer12. 矩阵中的路径

## 一、题目

请设计一个函数，用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径。路径可以从矩阵中的任意一格开始，每一步可以在矩阵中向左、右、上、下移动一格。如果一条路径经过了矩阵的某一格，那么该路径不能再次进入该格子。例如，在下面的 `3×4` 的矩阵中包含一条字符串 `“bfce”` 的路径（路径中的字母用加粗标出）。

```java
[["a","b","c","e"],
["s","f","c","s"],
["a","d","e","e"]]
```

但矩阵中不包含字符串 `“abfb”` 的路径，因为字符串的第一个字符 b 占据了矩阵中的第一行第二个格子之后，路径不能再次进入这个格子。

- 示例 1：

    输入：board = [["A","B","C","E"],["S","F","C","S"],["A","D","E","E"]], word = "ABCCED"
    输出：true

- 示例 2：

    输入：board = [["a","b"],["c","d"]], word = "abcd"
    输出：false

- 提示：

    1 <= board.length <= 200
    1 <= board[i].length <= 200

注意：本题与主站 79 题相同：https://leetcode-cn.com/problems/word-search/

## 二、解答

### 方法：回溯

**算法原理：**

- 深度优先搜索： 可以理解为暴力法遍历矩阵中所有字符串可能性。DFS 通过递归，先朝一个方向搜到底，再回溯至上个节点，沿另一个方向搜索，以此类推。
- 剪枝： 在搜索中，遇到 这条路不可能和目标字符串匹配成功 的情况（例如：此矩阵元素和目标字符不同或者此元素已被访问），则应立即返回，称之为 可行性剪枝 。

**算法剖析**：

- 递归参数： 当前元素在矩阵 board 中的行列索引 i 和 j ，当前目标字符在 word 中的索引 k 。
- 终止条件：
    - 返回 false ： ① 行或列索引越界 或 ② 当前矩阵元素与目标字符不同 或 ③ 当前矩阵元素已访问过 （③ 可合并至 ② ） 。
    - 返回 true ： 字符串 word 已全部匹配，即 `k = len(word) - 1` 。
- 递推工作：
    - 标记当前矩阵元素： 将 `board[i][j]`值暂存于变量 tmp ，并修改为字符 `'/'` ，代表此元素已访问过，防止之后搜索时重复访问。
    - 搜索下一单元格： 朝当前元素的 上、下、左、右 四个方向开启下层递归，使用 或 连接 （代表只需一条可行路径） ，并记录结果至 res 。
    - 还原当前矩阵元素： 将 tmp 暂存值还原至 `board[i][j]` 元素。
- 回溯返回值： 返回 res ，代表是否搜索到目标字符串。

```java
package array.easy;

/**
 * 比较典型的回溯
 */
public class Offer12 {
    // 方法二：DFS 方式
    public boolean exist(char[][] board, String word) {
        char[] words = word.toCharArray();
        for (int i = 0; i < board.length; i++) {
            for (int j = 0; j < board[0].length; j++) {
                if (dfs(board, words, i, j, 0)) {
                    return true;
                }
            }
        }
        return false;
    }

    boolean dfs(char[][] board, char[] word, int i, int j, int k) {
        if (i >= board.length || i < 0 || j >= board[0].length || j < 0 || board[i][j] != word[k]) {
            return false;
        }
        if (k == word.length - 1) {
            return true;
        }
        char tmp = board[i][j];
        board[i][j] = '0';

        int[][] direct = {{0, 1}, {1, 0}, {0, -1}, {-1, 0}};
        boolean res = false;
        for (int m = 0; m < 4; m++) {
            int newX = i + direct[m][0];
            int newY = j + direct[m][1];
            // 只要有一个方向的 dfs 结果为 true 则结果正确
            res = res || dfs(board, word, newX, newY, k + 1);
        }

        board[i][j] = tmp;
        return res;
    }
}

```



附加使用 used 数组的方法：

```java


/**
 * @Author GJXAIOU
 * Github 题解：https://github.com/GJXAIOU/LeetCode
 */
public class Solution {
    public boolean hasPath(char[] matrix, int rows, int cols, char[] str) {
        if (str.length > rows * cols) {
            return false;
        }
        // 重新构造成二维数组
        char[][] inputValue = new char[rows][cols];
        for (int i = 0; i < matrix.length; i++) {
            char value = matrix[i];
            int m = i / cols;
            int n = i % cols;
            inputValue[m][n] = value;
        }
        // 记录使用过的数字
        boolean[][] used = new boolean[rows][cols];
        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                if (dfs(inputValue, rows, cols, str, i, j, 0, used)) {
                    return true;
                }
            }
        }
        return false;
    }

    public boolean dfs(char[][] matrix, int rows, int cols, char[] str, int i, int j, int k,
                       boolean[][] used) {
        if (i >= rows || i < 0 || j >= cols || j < 0 || matrix[i][j] != str[k] || used[i][j] == true) {
            return false;
        }
        if (k == str.length -1) {
            return true;
        }
        used[i][j] = true;
        int[][] direct = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
        boolean res = false;
        for (int l = 0; l < 4; l++) {
            int newX = i + direct[l][0];
            int newY = j + direct[l][1];
            res = res || dfs(matrix, rows, cols, str, newX, newY, k + 1, used);
        }
        used[i][j] = false;
        return res;
    }
}
```



**更加推荐这种写法**

```java
package jd;


/**
 * @Author GJXAIOU
 * @Date 2020/9/18 20:54
 */
public class Solution {
    public boolean hasPath(char[] matrix, int rows, int cols, char[] str) {
        char[][] inputCharArray = new char[rows][cols];
        int begin = 0;
        for (int i = 0; i < inputCharArray.length; i++) {
            for (int j = 0; j < inputCharArray[0].length; j++) {
                inputCharArray[i][j] = matrix[begin++];
            }
        }

        int[][] used = new int[rows][cols];
        for (int i = 0; i < inputCharArray.length; i++) {
            for (int j = 0; j < inputCharArray[0].length; j++) {
                if (dfs(inputCharArray, i, j, str, 0, used)) {
                    return true;
                }
            }
        }
        return false;
    }

    public boolean dfs(char[][] matrix, int row, int col, char[] str, int len, int[][] used) {
        if (row >= matrix.length || row < 0 || col < 0 || col >= matrix[0].length || used[row][col] == 1 || matrix[row][col] != str[len]) {
            return false;
        }
        if (len == str.length - 1) {
            return true;
        }

        // 做选择
        used[row][col] = 1;
        int[][] direct = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};
        for (int i = 0; i < direct.length; i++) {
            int newX = row + direct[i][0];
            int newY = col + direct[i][1];
            if (dfs(matrix, newX, newY, str, len + 1, used)) {
                return true;
            }
        }
        used[row][col] = 0;
        return false;
    }
}
```

