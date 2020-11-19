---
layout:     post
title:      Offer39. 数组中出现次数超过一半的数字
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



# 面试题39. 数组中出现次数超过一半的数字

## 一、题目

数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。

你可以假设数组是非空的，并且给定的数组总是存在多数元素。

**示例 1:**

```
输入: [1, 2, 3, 2, 2, 2, 5, 4, 2]
输出: 2
```

**限制：**

```
1 <= 数组长度 <= 50000
```

注意：本题与主站 169 题相同：https://leetcode-cn.com/problems/majority-element/

 

## 二、解答

### 方法一：使用 HashMap 记录每次元素出现次数

```java
package array.easy;

import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;

/**
 * @Author GJXAIOU
 * @Date 2020/2/24 22:31
 */
public class Offer39 {
    // 方法一：Hash 计数
    public int majorityElement(int[] nums) {
        if (nums.length == 1) {
            return nums[0];
        }
        HashMap<Integer, Integer> map = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            if (map.containsKey(nums[i])) {
                int oldValue = map.get(nums[i]);
                int newValue = oldValue + 1;
                if (newValue > nums.length >>> 1) {
                    return nums[i];
                } else {
                    map.replace(nums[i], oldValue, newValue);
                }
            } else {
                map.put(nums[i], 1);
            }
        }
        return 0;
    }
}
```

### 方法二：排序之后的中间数

```java
package array.easy;

import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;

/**
 * @Author GJXAIOU
 * @Date 2020/2/24 22:31
 */
public class Offer39 {
    // 方法二：因为是众数，数量超过数组长度一半，则数组中间值一定为所求值
    public int majorityElement2(int[] nums) {
        Arrays.sort(nums);
        return nums[nums.length >> 1];
    }

}

```

### 方法三：位运算（放弃）

位运算的思想: 如果一个数字的出现次数超过了数组长度的一半, 那么这个数字二进制的各个bit的出现次数同样超过了数组长度的一半

```java
package array.easy;

import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;

/**
 * @Author GJXAIOU
 * @Date 2020/2/24 22:31
 */
public class Offer39 {
    // 方法三：位运算
    public int majorityElement3(int[] nums) {
        int[] bit = new int[32];
        int n = nums.length;
        for (int a : nums) {
            for (int i = 0; i < 32; i++) {
                //无符号右移; 负数的无符号右移移动的是补码还是原码?
                if (((a >>> i) & 1) == 1) {
                    bit[i]++;
                }
            }
        }

        int res = 0;
        for (int i = 0; i < 32; i++) {
            if (bit[i] > n / 2) {
                res = res | (1 << i);
            }
        }
        return res;
    }

}

```

