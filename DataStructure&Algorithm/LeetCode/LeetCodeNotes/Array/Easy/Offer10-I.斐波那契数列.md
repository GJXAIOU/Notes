---
layout:     post
title:      Offer10-I.斐波那契数列
subtitle:   Array.easy
date:       2020-04-10
author:     GJXAIOU
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - 数组
	- 动态规划
	- 完成
---

# Offer10-I.斐波那契数列

## 一、题目

写一个函数，输入 `n` ，求斐波那契（Fibonacci）数列的第 `n` 项。斐波那契数列的定义如下：

```java
F(0) = 0,   F(1) = 1
F(N) = F(N - 1) + F(N - 2), 其中 N > 1.
```

斐波那契数列由 0 和 1 开始，之后的斐波那契数就是由之前的两数相加而得出。

答案需要取模 1e9+7（1000000007），如计算初始结果为：1000000008，请返回 1。

 

**示例 1：**

```
输入：n = 2
输出：1
```

**示例 2：**

```
输入：n = 5
输出：5
```

 

**提示：**

- `0 <= n <= 100`

注意：本题与主站 509 题相同：https://leetcode-cn.com/problems/fibonacci-number/



## 二、解答

- 方法一：记忆化搜索

````java
package array.easy;

public class Offer10I {
    // 方案一：记忆化搜索
    public int fib(int n) {
        if (n <= 1) {
            return n;
        }

        // 使用一个数组来保存已经计算完成的结果
        int[] ans = new int[n + 1];
        ans[0] = 0;
        ans[1] = 1;
        if (ans[n] != 0) {
            return ans[n];
        } else {
            ans[n] = fib(n - 2) + fib(n - 1);
        }
        return ans[n];
    }
}
````



- 方案二：动态规划

    - 步骤一：确定几维表

        因为爆搜中只有一个变量，所以 DP 数组为一维表。

    - 步骤二：基本情况填写

        在 n 为 0 和 1 的时候值是知道了。

        ![image-20200410205502567](Offer10-I.%E6%96%90%E6%B3%A2%E9%82%A3%E5%A5%91%E6%95%B0%E5%88%97.resource/image-20200410205502567.png)

    - 步骤四：迭代关系

        `F(N) = F(N - 1) + F(N - 2)`, 其中 N > 1

    - 步骤五：要求的值：f(n)

```java
package array.easy;


public class Offer10I {

    // 方案二：动态规划
    public int fib2(int n) {
        if(n <= 1){
            return n;
        }
        int[] dp = new int[n + 1];
        dp[0] = 0;
        dp[1] = 1;
        for (int i = 2; i < dp.length; i++) {
            dp[i] = (dp[i - 1] + dp[i - 2]) % 1000000007;
        }
        return dp[n];
    }   
}

```

