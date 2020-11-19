---
layout:     post
title:      Offer17.打印从 1 到最大的 n 位数
subtitle:   在数组中找出两数和为目标值的下标
date:       2020-02-24
author:     GJXAIOU
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - 剑指 Offer
    - Array
    - Easy 
---

# Offer17.打印从 1 到最大的 n 位数

## 一、题目

输入数字 n，按顺序打印出从 1 到最大的 n 位十进制数。比如输入 3，则打印出 1、2、3 一直到最大的 3 位数 999。

- 示例 1:

> 输入: n = 1
> 输出: [1,2,3,4,5,6,7,8,9]

- 说明：
    - 用返回一个整数列表来代替打印
    - n 为正整数

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/da-yin-cong-1dao-zui-da-de-nwei-shu-lcof
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 二、解答

```java
package array.easy;

/**
 * @Author GJXAIOU
 * @Date 2020/2/24 13:04
 */
public class Offer17 {

    public int[] printNumbers(int n) {

        if (n == 0) {
            return new int[0];
        }
        // 求得 n 位数对应的整数最大值
        int maxValue = 0;
        for (int i = 0; i < n; i++) {
            maxValue = maxValue * 10 + 9;
        }

        int[] resArray = new int[maxValue];
        for (int i = 0; i < maxValue; i++) {
            resArray[i] = i + 1;
        }
        return resArray;
    }
}

```

