---
layout:     post
title:      Offer47.礼物的最大价值
subtitle:   Array.medium
date:       2020-04-13
author:     GJXAIOU
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - 数组
	- 完成
---

# Offer45.把数组排成最小的数

## 一、题目

输入一个正整数数组，把数组里所有数字拼接起来排成一个数，打印能拼接出的所有数字中最小的一个。

**示例 1:**

```
输入: [10,2]
输出: "102"
```

**示例 2:**

```
输入: [3,30,34,5,9]
输出: "3033459"
```

**提示:**

- `0 < nums.length <= 100`

**说明:**

- 输出结果可能非常大，所以你需要返回一个字符串而不是整数
- 拼接起来的数字可能会有前导 0，最后结果不需要去掉前导 0



## 二、解答

- 此题求拼接起来的 “最小数字” ，本质上是一个排序问题。

- 排序判断规则： 设 nums 任意两数字的字符串格式 x 和 y ，则

    - 若拼接字符串 x+y  > y+x ，则 m>n ；
    - 反之，若 x + y < y + x ，则 n < m ；

- 根据以上规则，套用任何排序方法对 nums 执行排序即可。

    ![image-20200413125920502](Offer45.%E6%8A%8A%E6%95%B0%E7%BB%84%E6%8E%92%E6%88%90%E6%9C%80%E5%B0%8F%E7%9A%84%E6%95%B0.resource/image-20200413125920502.png)

**算法流程：**

- 初始化： 字符串列表 strs ，保存各数字的字符串格式；

- 列表排序： 应用以上 “排序判断规则” ，对 strs 执行排序；

- 返回值： 拼接 strs 中的所有字符串，并返回。

    

**复杂度分析：**

- 时间复杂度 O(NlogN) ： N 为最终返回值的字符数量（ strs 列表的长度 ≤N ）；使用快排或内置函数的平均时间复杂度为 O(N log N)，最差为 O(N^2) 。
- 空间复杂度 O(N) ： 字符串列表 strs 占用线性大小的额外空间。

以快速排序为例：
需修改快速排序函数中的排序判断规则。字符串大小（字典序）对比的实现方法：Java 中使用 A.compareTo(B) 。

==**方法二就是直接使用了 内置数组排序，然后重新 compareTo 方法即可**==。

```java
package array.medium;

import java.util.Arrays;
import java.util.Comparator;

/**
 * @author GJXAIOU
 * @create 2020/04/13 12:54
 */
public class Offer45 {

    public String minNumber(int[] nums) {
        String[] copyNums = new String[nums.length];
        for (int i = 0; i < nums.length; i++)
            copyNums[i] = String.valueOf(nums[i]);
        // 对复制之后的数组进行排序
        fastSort(copyNums, 0, copyNums.length - 1);
        StringBuilder res = new StringBuilder();
        for (String s : copyNums)
            res.append(s);
        return res.toString();
    }

    void fastSort(String[] strArray, int left, int right) {
        if (left >= right) {
            return;
        }
        int i = left, j = right;
        String tmp = strArray[i];
        while (i < j) {
            // a.compareTo(b) 可以理解为 a - b,返回值是 ASCII 码差值
            while ((strArray[j] + strArray[left]).compareTo(strArray[left] + strArray[j]) >= 0 && i < j) {
                j--;
            }
            while ((strArray[i] + strArray[left]).compareTo(strArray[left] + strArray[i]) <= 0 && i < j) {
                i++;
            }
            tmp = strArray[i];
            strArray[i] = strArray[j];
            strArray[j] = tmp;
        }
        strArray[i] = strArray[left];
        strArray[left] = tmp;
        fastSort(strArray, left, i - 1);
        fastSort(strArray, i + 1, right);
    }

    // 方法二：
    public String minNumber2(int[] nums) {
        String[] copyNums = new String[nums.length];
        for (int i = 0; i < nums.length; i++)
            copyNums[i] = String.valueOf(nums[i]);
        // 对复制之后的数组进行排序
        // 可以使用内置排序：Arrays.sort(copyNums, (x, y) -> (x + y).compareTo(y + x)); 代替 fastSort
        Arrays.sort(copyNums, new MyComparator());
        StringBuilder res = new StringBuilder();
        for (String s : copyNums)
            res.append(s);
        return res.toString();
    }

    class MyComparator implements Comparator<String> {

        @Override
        public int compare(String o1, String o2) {
            return (o1 + o2).compareTo(o2 + o1);
        }
    }
}

```



### 上面方法二的第二种写法

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Comparator;

public class Solution {
    public String PrintMinNumber(int[] numbers) {

        String[] input = new String[numbers.length];
        for (int i = 0; i < numbers.length; i++) {
            input[i] = String.valueOf(numbers[i]);
        }

        Arrays.sort(input, new MyCompactor());
        StringBuilder res = new StringBuilder();
        for (String s : input) {
            res.append(s);
        }
        return res.toString();
    }

    class MyCompactor implements Comparator<String> {

        @Override
        public int compare(String o1, String o2) {
            return Integer.parseInt(o1 + o2) - Integer.parseInt(o2 + o1);
        }
    }
}
```

