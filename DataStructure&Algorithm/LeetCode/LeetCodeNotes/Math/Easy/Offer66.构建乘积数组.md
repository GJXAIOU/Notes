---
layout:     post
title:      Offer66. 构建乘积数组
subtitle:   Math.easy
date:       2020-04-12
author:     GJXAIOU
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - 数学
	- 位运算
	- 完成
---



# Offer66. 构建乘积数组

## 一、题目

给定一个数组 `A[0,1,…,n-1]`，请构建一个数组 `B[0,1,…,n-1]`，其中 `B` 中的元素 `B[i]=A[0]×A[1]×…×A[i-1]×A[i+1]×…×A[n-1]`。不能使用除法。

 

**示例:**

```
输入: [1,2,3,4,5]
输出: [120,60,40,30,24]
```

 

**提示：**

- 所有元素乘积之和不会溢出 32 位整数
- `a.length <= 100000`

## 二、解答


通过 `B[i]=A[0]×A[1]×…×A[i-1]×A[i+1]×…×A[n-1]`，我们发现 **B[i] 就是 A[i] 左边所有元素的积 乘 A[i] 右边所有元素的积**。

对称遍历

- 从左往右遍历累乘，结果保存在数组 ret 中，此时 ret[i] 表示，A[i] 左边所有元素的乘积
- 然后从右往左遍历累乘，获取A[i] 右边所有元素的乘积
- 两边遍历之后得到的 ret，就是最终结果

**代码将第二步更改了，更加容易理解**。

```java
package math.easy;

/**
 * @author GJXAIOU
 * @create 2020/04/12 21:24
 */
public class Offer66 {
    public int[] constructArr(int[] a) {
        int[] b = new int[a.length];
        int leftMultiply = 1;
        int[] left = new int[a.length];
        int rightMultiply = 1;
        int[] right = new int[a.length];
        // 分别求出 i 值左边的乘积和右边乘积（均不包括 i 值）
        for (int i = 0; i < a.length; i++) {
            left[i] = leftMultiply;
            leftMultiply *= a[i];
        }
        for (int i = a.length - 1; i >= 0; i--) {
            right[i] = rightMultiply;
            rightMultiply *= a[i];

        }

        for (int i = 0; i < b.length; i++) {
            b[i] = left[i] * right[i];
        }
        return b;
    }

}
```

