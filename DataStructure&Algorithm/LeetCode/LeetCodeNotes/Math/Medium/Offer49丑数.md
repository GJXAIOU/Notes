---
layout:     post
title:      Offer49. 丑数
subtitle:   Math.medium
date:       2020-02-09
author:     GJXAIOU
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - 数学
	- 动态规划
	- 完成
---

# Offer49. 丑数

## 一、题目

我们把只包含因子 2、3 和 5 的数称作丑数（Ugly Number）。求按从小到大的顺序的第 n 个丑数。

 

**示例:**

```
输入: n = 10
输出: 12
解释: 1, 2, 3, 4, 5, 6, 8, 9, 10, 12 是前 10 个丑数。
```

**说明:** 

1. `1` 是丑数。
2. `n` **不超过**1690。

注意：本题与主站 264 题相同：https://leetcode-cn.com/problems/ugly-number-ii/

## 二、解答


**丑数的递推性质： 由于丑数只包含因子 2,3,5 ，因此较大的丑数一定能够通过 某较小丑数 × 某因子 得到。**

- 设已知长度为 n 的丑数序列 $x_1, x_2, ... , x_n$ ，求第 n+1 个丑数 $x_{n+1}$  。根根据递推性质，下一个丑数 $x_{n+1}$只可能是以下三种情况其中之一（索引 a, b, ca,b,c 为未知数）：
    $x_{n+1} = \begin{cases} x_{a} \times 2 & ,a \in [1, n] \\ x_{b} \times 3 & ,b \in [1, n] \\ x_{c} \times 5 & ,c \in [1, n] \end{cases}$

    由于 $x_{n+1}$是 最接近 $x_n$的丑数，因此索引 a, b, c 一定满足以下条件：
    $\begin{cases} x_{a} \times 2 > x_n \geq x_{a-1} \times 2 & ，即 x_a 为首个乘以 2 后大于 x_n 的丑数 \\ x_{b} \times 3 > x_n \geq x_{b-1} \times 3 & ，即 x_b 为首个乘以 3 后大于 x_n 的丑数 \\ x_{c} \times 5 > x_n \geq x_{c-1} \times 5 & ，即 x_c 为首个乘以 5 后大于 x_n 的丑数 \\ \end{cases}$

    若索引 a,b,c 满足以上条件，则可使用递推公式计算下个丑数 $x_{n+1}$，其为三种情况中的最小值，即：
    $x_{n+1} = \min(x_{a} \times 2, x_{b} \times 3, x_{c} \times 5)$

    ![image-20200412143602462](Offer49%E4%B8%91%E6%95%B0.resource/image-20200412143602462.png)

**动态规划解析：**

- 状态定义： 设动态规划列表 dp ，**dp[i] 代表第 i+1 个丑数**。
- 转移方程：
    当索引 a,b,c 满足以下条件时， dp[i] 为三种情况的最小值；
- 每轮计算 dp[i] 后，需要更新索引 a, b, c 的值，使其始终满足方程条件，以便下轮计算。实现方法：**分别独立判断 dp[i] 和 $dp[a] \times 2$ , $dp[b] \times 3$ , $dp[c] \times 5$ 的大小关系，若相等则将对应索引 a , b , c 加 1 。**
    $\begin{cases} dp[a] \times 2 > dp[i-1] \geq dp[a-1] \times 2 \\ dp[b] \times 3 > dp[i-1] \geq dp[b-1] \times 3 \\ dp[c] \times 5 > dp[i-1] \geq dp[c-1] \times 5 \\ \end{cases}$
    	


$dp[i] = \min(dp[a] \times 2, dp[b] \times 3, dp[c] \times 5)$


- 初始状态： dp[0]=1 ，即第一个丑数为 1 ；
- 返回值： dp[n−1] ，即返回第 n 个丑数。



![Offer49](Offer49%E4%B8%91%E6%95%B0.resource/Offer49.gif)

复杂度分析：

时间复杂度 O(N) ： 其中 N = n ，动态规划需遍历计算 dp 列表。
空间复杂度 O(N) ： 长度为 N 的 dp 列表使用 O(N) 的额外空间。

```java
package math.medium;

/**
 * @author GJXAIOU
 * @create 2020/04/12 14:43
 */
public class Offer49 {

    public int nthUglyNumber(int n) {
        // 初始索引均为 0
        int a = 0, b = 0, c = 0;
        int[] dp = new int[n];
        dp[0] = 1;
        for (int i = 1; i < n; i++) {
            int n2 = dp[a] * 2, n3 = dp[b] * 3, n5 = dp[c] * 5;
            // dp[i] 为其中最小值
            dp[i] = Math.min(Math.min(n2, n3), n5);
            if (dp[i] == n2) {
                a++;
            }
            if (dp[i] == n3) {
                b++;
            }
            if (dp[i] == n5) {
                c++;
            }
        }
        return dp[n - 1];
    }

}

```

