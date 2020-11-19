---
layout:     post
title:      Offer03. 数组中重复的数字
subtitle:   在数组中找出两数和为目标值的下标
date:       2020-02-25
author:     GJXAIOU
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - 剑指 Offer
    - Array
    - Easy 
---



#  Offer03. 数组中重复的数字

## 一、题目

找出数组中重复的数字。

**在一个长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内。数组中某些数字是重复的**，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。

- 示例 1：

> 输入：
> [2, 3, 1, 0, 2, 5, 3]
> 输出：2 或 3 

- 限制：

`2 <= n <= 100000`

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/shu-zu-zhong-zhong-fu-de-shu-zi-lcof
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 二、解答

**方法一：HashMap**

时间复杂度和空间复杂度都是 O(N)

```java
package array.easy;

import java.util.Arrays;
import java.util.HashMap;

/**
 * @Author GJXAIOU
 * @Date 2020/2/25 9:38
 */
public class Offer03 {
    // 方法一：使用 HashMap
    public int findRepeatNumber(int[] nums) {
        if (nums.length == 0 || nums.length == 1) {
            return -1;
        }

        HashMap<Integer, Integer> resMap = new HashMap<>();
        for (int i = 0; i < nums.length; i++) {
            if (resMap.containsKey(nums[i])) {
                int oldValue = resMap.get(nums[i]);
                int newValue = oldValue + 1;
                if (newValue >= 2) {
                    return nums[i];
                } else {
                    resMap.replace(nums[i], oldValue, newValue);
                }
            } else {
                resMap.put(nums[i], 1);
            }
        }
        return -1;
    }

}

```



**方法二：排序**

时间复杂度为 O(NlogN)，空间复杂度为  O(1)

```java
package array.easy;

import java.util.Arrays;
import java.util.HashMap;

/**
 * @Author GJXAIOU
 * @Date 2020/2/25 9:38
 */
public class Offer03 {

    // 方法二：排序然后双指针
    public int findRepeatNumber2(int[] nums) {
        if (nums.length == 0 || nums.length == 1) {
            return -1;
        }
        Arrays.sort(nums);
        int fast = 1;
        int slow = 0;
        while (fast < nums.length) {
            if (nums[fast] == nums[slow]) {
                return nums[fast];
            } else {
                fast++;
                slow++;
            }
        }
        return -1;
    }
}

```



**方法三：类似桶排序**

时间复杂度为 O(N)，空间复杂度为 O(1)

- 题目中指出 在一个长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内 。 因此，我们可以利用数组下标作为 HashMap 的 key ，具体实现方法是遍历数组并通过交换操作使索引与值一一对应，（即 nums[i] = i ），这样就能通过索引找到对应值。

- 算法原理： 遍历数组，每次遇到索引为 i 的新数字 nums[i] 时，将其交换至索引为 nums[i] 的 nums[nums[i]] 处。而当遍历遇到一个重复数字 x 时，一定有 nums[x] == x （因为第一次遇到 x 时已经将其交换至 nums[x] 处了）。利用以上方法，即可得到一组重复数字。

- 算法流程：
    - 遍历数组 nums ，设索引初始值为 i = 0:
        - 若 nums[i] == i ： 说明此数字已在对应索引位置，无需交换，因此执行 i += 1 与 continue ；
        - 若 nums[nums[i]] == nums[i] ： 说明索引 nums[i] 处的元素值也为 nums[i]，即找到一组相同值，返回此值 nums[i]；
        - 否则： 当前数字是第一次遇到，因此交换索引为 i 和 nums[i] 的元素值，将此数字交换至对应索引位置。
    - 若遍历完毕尚未返回，则返回 −1 ，代表数组中无相同值。
      

![Offer03](Offer03.%E6%95%B0%E7%BB%84%E4%B8%AD%E9%87%8D%E5%A4%8D%E7%9A%84%E6%95%B0%E5%AD%97.resource/Offer03.gif)

```java
package array.easy;

import java.util.Arrays;
import java.util.HashMap;

/**
 * @Author GJXAIOU
 * @Date 2020/2/25 9:38
 */
public class Offer03 {

    // 方法三：桶排序，将 nums[i] 元素放置在 i 位置上
    public int findRepeatNumber3(int[] nums) {
        if (nums.length == 0 || nums.length == 1) {
            return -1;
        }
        int i = 0;
        while (i < nums.length) {
            if (nums[i] == i) {
                i++;
                continue;
            }
            if (nums[nums[i]] == nums[i]) {
                return nums[i];
            }
            int tmp = nums[i];
            nums[i] = nums[tmp];
            nums[tmp] = tmp;
        }
        return -1;
    }
}

```
