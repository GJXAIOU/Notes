---
layout:     post
title:      Offer57. 和为s的两个数字
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

# Offer57. 和为 s 的两个数字

## 一、题目

输入一个**递增排序**的数组和一个数字 s，在数组中查找两个数，使得它们的和正好是 s。如果有多对数字的和等于s，则输出任意一对即可。

- 示例 1：

> 输入：nums = [2,7,11,15], target = 9
> 输出：[2,7] 或者 [7,2]

- 示例 2：

> 输入：nums = [10,26,30,31,47,60], target = 40
> 输出：[10,30] 或者 [30,10]

- 限制：

> 1 <= nums.length <= 10^5
> 1 <= nums[i] <= 10^6

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/he-wei-sde-liang-ge-shu-zi-lcof
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 二、解答

### 方法一：直接搜索

时间复杂度为 O(N^2)，空间复杂度为 O(1)

代码忽略。

### 方法二：Hash 表

遍历一遍数组，遍历一遍 Hash 表，时间复杂度为：O(N)，空间复杂度为 O(N)。

代码忽略

### 方法三：双指针

开头结尾双指针判别，时间复杂度 O(N)，空间复杂度 O(1)。

```java
package array.easy;

/**
 * @Author GJXAIOU
 * @Date 2020/2/25 10:31
 */
public class Offer57 {
    // 方法二：双指针
    public int[] twoSum(int[] nums, int target) {
        // 如果 target < nums[0] 即数组最小值，则不可能
        if (target < nums[0]) {
            return nums;
        }
        int left = 0;
        int right = nums.length - 1;
        while (left < right) {
            if (nums[left] + nums[right] == target) {
                return new int[]{nums[left], nums[right]};
            } else if (nums[left] + nums[right] < target) {
                left++;
            } else {
                right--;
            }
        }
        return nums;
    }
}

```





### 题目变更：

#### 题目描述

输入一个递增排序的数组和一个数字S，在数组中查找两个数，使得他们的和正好是S，如果有多对数字的和等于S，输出两个数的乘积最小的。

#### 输出描述:

```
对应每个测试案例，输出两个数，小的先输出。
```



```java
链接：https://www.nowcoder.com/questionTerminal/390da4f7a00f44bea7c2f3d19491311b?f=discussion&toCommentId=6359558
来源：牛客网

import java.util.ArrayList;
 
public class Solution {
    class Node {
        int val1;
        int val2;
 
        Node(int val1, int val2) {
            this.val1 = val1;
            this.val2 = val2;
        }
    }
 
    public ArrayList<Integer> FindNumbersWithSum(int[] array, int sum) {
        if (array == null || array.length < 2) {
            return new ArrayList<>();
        }
 
        int start = 0;
        int end = array.length - 1;
        int minValue = Integer.MAX_VALUE;
        ArrayList<Node> valueList = new ArrayList<>();
        ArrayList<Integer> resList = new ArrayList<>();
        while (start < end) {
            int startValue = array[start];
            int endValue = array[end];
            if (startValue + endValue == sum) {
                int value = startValue * endValue;
                if (value < minValue) {
                    // 因为数组本身有序，所以 startValue <= endValue
                    valueList.add(new Node(startValue, endValue));
                    minValue = value;
                }
                start++;
                end--;
            } else if (startValue + endValue < sum) {
                start++;
            } else {
                end--;
            }
        }
 
        if (valueList.size() >= 1) {
            resList.add(valueList.get(valueList.size() - 1).val1);
            resList.add(valueList.get(valueList.size() - 1).val2);
        }
        return resList;
    }
 
}
```

