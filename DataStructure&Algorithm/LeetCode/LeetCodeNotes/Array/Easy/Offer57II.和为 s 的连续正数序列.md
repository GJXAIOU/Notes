---
layout:     post
title:      Offer57II. 和为 s 的连续正数序列
subtitle:   在数组中找出两数和为目标值的下标
date:       2020-02-25
author:     GJXAIOU
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - 剑指 Offer
    - 数组
	- 滑动窗口
    - Easy 
---

# Offer57 - II. 和为s的连续正数序列

## 一、题目

输入一个正整数 `target` ，**输出所有和**为 `target` 的连续正整数序列（至少含有两个数）。

序列内的数字由小到大排列，不同序列按照首个数字从小到大排列。

- 示例 1：

> 输入：target = 9
> 输出：[[2,3,4],[4,5]]

- 示例 2：

> 输入：target = 15
> 输出：[[1,2,3,4,5],[4,5,6],[7,8]]

- 限制：

1 <= target <= 10^5

## 二、解答

### 方法：滑动窗口

**使用两个指针作为窗口的两个边界，左闭右开。**

```java
package array.easy;


import java.util.ArrayList;

/**
 * @Author GJXAIOU
 * @Date 2020/2/25 14:19
 */
public class Offer57II {
    public int[][] findContinuousSequence(int target) {
        if (target < 3) {
            return new int[1][0];
        }
        ArrayList<int[]> resList = new ArrayList<>();
        int left = 1;
        int right = 2;
        // 初始和为 1
        int curSum = left;

        // 因为如果 target 的 mid 位置值 + （mid + 1） 位置值肯定大于 target，所以到 mid 位置即可
        int mid = (left + target) >>> 1;
        // 区间为左闭右开
        while (left < mid) {
           // 和不断往右相加，如果和等于 target 则符合一种情况
            curSum += right;
            right++;
            if (curSum == target) {
                resList.add(count(left, right));
            }
            while (curSum > target) {
                curSum -= left;
                left++;
                if (curSum == target) {
                    resList.add(count(left, right));
                }
            }
        }
        return resList.toArray(new int[0][]);
    }

    public int[] count(int left, int right) {
        int i = 0;
        int length = right - left;
        int[] temp = new int[length];
        while (i < length) {
            temp[i] = i + left;
            i++;
        }
        return temp;
    }
}
```

**方案二**：自己思路

==结果不同，下面的方法返回值中窗口长度大于等于2，即不能 目标值为 K，最后一个窗口也为 K==

```java

import java.util.ArrayList;

/**
 * @Author GJXAIOU
 * Github 题解：https://github.com/GJXAIOU/LeetCode
 */
public class Solution {
    public ArrayList<ArrayList<Integer>> FindContinuousSequence(int sum) {
        // 合法性判断
        if (sum < 1) {
            return new ArrayList<>();
        }

        int winBegin = 1;
        int winEnd = 2;
        int curSum = 1;
        ArrayList<ArrayList<Integer>> resList = new ArrayList<>();
        // 因为 sum 的中间值 + 中间值后面一个值和肯定大于 sum,例如 100 的中间值 50 + 后面一个值 51 = 101 > 100
        int mid = (winBegin + sum) >>> 1;
        while (winBegin < mid) {
            if (curSum < sum){
                curSum += winEnd;
                winEnd++;
            } else if (curSum == sum) {
                ArrayList<Integer> tempList = new ArrayList<>();
                for (int i = winBegin; i < winEnd; i++) {
                    tempList.add(i);
                }
                resList.add(tempList);
                // 添加之后将最前面的数减了，否则就是死循环了
                curSum -= winBegin;
                winBegin++;
            }
            while (curSum > sum) {
                curSum -= winBegin;
                winBegin++;
            }
        }
        return resList;
    }
}
```

