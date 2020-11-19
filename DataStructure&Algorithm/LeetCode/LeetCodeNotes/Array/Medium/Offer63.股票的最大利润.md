---
layout:     post
title:      Offer63. 股票的最大利润
subtitle:   Array.medium
date:       2020-02-09
author:     GJXAIOU
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - 数组
	- 动态规划
	- 完成
---



# Offer63. 股票的最大利润

## 一、题目

假设把某股票的价格按照时间先后顺序存储在数组中，请问买卖该股票一次可能获得的最大利润是多少？

**示例 1:**

```
输入: [7,1,5,3,6,4]
输出: 5
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
     注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格。
```

**示例 2:**

```
输入: [7,6,4,3,1]
输出: 0
解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。
```

**限制：**

```
0 <= 数组长度 <= 10^5
```

**注意：**本题与主站 121 题相同：https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/

## 二、解答

动态规划解析：

- 状态定义： 设动态规划列表 dp ，==**dp[i] 代表以 prices[i] 为结尾的子数组的最大利润（以下简称为 “前 i 日的最大利润” 。**==

- 转移方程： 由于题目限定 “买卖该股票一次” ，因此前 i 日最大利润 dp[i] 等于前 i - 1 日最大利润 dp[i-1] 和第 i 日卖出的最大利润中的最大值。
    $前 i 日最大利润 = \max(前 (i-1) 日最大利润, 第 i 日价格 - 前 i 日最低价格)$

    $dp[i] = \max(dp[i - 1], prices[i] - \min(prices[0:i]))$

- 初始状态： dp[0] = 0 ，即首日利润为 0 ；
- 返回值： dp[n - 1] ，其中 n 为 dp 列表长度。

**效率优化：**

- 时间复杂度降低： 前 i 日的最低价格 $\min(prices[0:i])$ 时间复杂度为 O(i) 。而在遍历 prices 时，可以借助一个变量（记为成本 cost ）每日更新最低价格。优化后的转移方程为：
    $dp[i] = \max(dp[i - 1], prices[i] - \min(cost, prices[i])$


- 空间复杂度降低： 由于 dp[i] 只与 dp[i−1] , prices[i] , cost 相关，因此可使用一个变量（记为利润 profit ）代替 dp 列表。优化后的转移方程为：
    $profit = \max(profit, prices[i] - \min(cost, prices[i])$


**复杂度分析：**
时间复杂度 O(N) ： 其中 N 为 prices 列表长度，动态规划需遍历 prices 。
空间复杂度 O(1) ： 变量 cost 和 profit 使用常数大小的额外空间。



```java
package array.medium;

/**
 * @author GJXAIOU
 * @create 2020/04/12 14:51
 */
public class Offer63 {
    // 自己的动态规划
    public int maxProfit(int[] prices) {
        if (prices.length == 0 || prices == null) {
            return 0;
        }
        // dp[i] 表示前 i 日的最大利润
        int[] dp = new int[prices.length + 1];
        // 便于理解，这里的 dp[0] 不使用，dp[1] 就是表示前 1 天可以获取的利润，就是 0
        dp[1] = 0;
        // 保存前 i 天的最低成本，默认成本就是第一天成本
        int lowestCost = prices[0];
        for (int i = 2; i <= prices.length; i++) {
            lowestCost = Math.min(lowestCost, prices[i - 1]);
            dp[i] = Math.max(dp[i - 1], prices[i - 1] - lowestCost);
        }
        return dp[prices.length];
    }

    // 题解的 DP
    public int maxProfit2(int[] prices) {
        int cost = Integer.MAX_VALUE, profit = 0;
        for (int price : prices) {
            cost = Math.min(cost, price);
            profit = Math.max(profit, price - cost);
        }
        return profit;
    }

}

```
