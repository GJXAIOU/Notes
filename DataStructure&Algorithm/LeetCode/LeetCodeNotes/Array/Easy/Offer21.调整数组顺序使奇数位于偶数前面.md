---
layout:     post
title:      Offer21. 调整数组顺序使奇数位于偶数前面
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



# Offer21. 调整数组顺序使奇数位于偶数前面

## 一、题目

输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有奇数位于数组的前半部分，所有偶数位于数组的后半部分。

**示例：**

```
输入：nums = [1,2,3,4]
输出：[1,3,2,4] 
注：[3,1,2,4] 也是正确的答案之一。
```

**提示：**

1. `1 <= nums.length <= 50000`
2. `1 <= nums[i] <= 10000`

## 二、解答

### 方法一：首尾双指针

- 定义头指针 left ，尾指针 right .

- left 一直往右移，直到它指向的值为偶数

- right 一直往左移， 直到它指向的值为奇数

- 交换 nums[left] 和 nums[right] .

- 重复上述操作，直到 left==right .

```java
package array.easy;


/**
 * @Author GJXAIOU
 * @Date 2020/2/25 15:57
 */
public class Offer21 {
    // 方法一：首位双指针，元素相对位置会变
    public int[] exchange(int[] nums) {
        if (nums.length == 0) {
            return new int[0];
        }

        int left = 0;
        int right = nums.length - 1;

        while (left < right) {
            if (nums[left] % 2 == 0 && nums[right] % 2 != 0) {
                swap(nums, left, right);
               // 即 right 为偶数时候
            } else if (nums[left] % 2 == 0) {
                right--;
                // 即 left 为奇数的时候
            } else if (nums[left] % 2 != 0) {
                left++;
            }
        }
        return nums;
    }

    public void swap(int[] nums, int left, int right) {
        nums[left] = nums[left] ^ nums[right];
        nums[right] = nums[left] ^ nums[right];
        nums[left] = nums[left] ^ nums[right];
    }
}
```



### 方法二：快慢双指针

- 定义快慢双指针 fast 和 low ，fast 在前， low 在后 .

- fast 的作用是向前搜索奇数位置，low 的作用是指向下一个奇数应当存放的位置

- fast 向前移动，当它搜索到奇数时，将它和 nums[low] 交换，此时 low 向前移动一个位置 .

- 重复上述操作，直到 fast 指向数组末尾 .

```java
package array.easy;


/**
 * @Author GJXAIOU
 * @Date 2020/2/25 15:57
 */
public class Offer21 {
    public void swap(int[] nums, int left, int right) {
        if (left == right) {
            return;
        }
        nums[left] = nums[left] ^ nums[right];
        nums[right] = nums[left] ^ nums[right];
        nums[left] = nums[left] ^ nums[right];
    }

    // 方法二：快慢指针
    public int[] exchange2(int[] nums) {
        int low = 0, fast = 0;
        while (fast < nums.length) {
            if (nums[fast] % 2 != 0) {
                swap(nums, low, fast);
                low++;
            }
            fast++;
        }
        return nums;
    }
}

```



**方法三**：本质上是一个简化版的荷兰国旗问题，就是将树分为 0 1 两部分（所有数 % 2），如果结果大于  0.5 放在左边，小于 0.5 放在右边。