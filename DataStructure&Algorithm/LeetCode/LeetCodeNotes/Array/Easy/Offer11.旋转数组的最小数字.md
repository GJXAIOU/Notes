---
layout:     post
title:      Offer11. 旋转数组的最小数字
subtitle:   Array.easy
date:       2020-04-10
author:     GJXAIOU
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - 数组
	- 二分查找
	- 完成
---



# Offer11. 旋转数组的最小数字

## 一、题目

把一个数组**最开始的若干个元素搬到数组的末尾**，我们称之为数组的旋转。输入一个递增排序的数组的一个旋转，输出旋转数组的最小元素。例如，数组 `[3,4,5,1,2]` 为 `[1,2,3,4,5]` 的一个旋转，该数组的最小值为1。 

**示例 1：**

```
输入：[3,4,5,1,2]
输出：1
```

**示例 2：**

```
输入：[2,2,2,0,1]
输出：0
```

注意：本题与主站 154 题相同：https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array-ii/



## 二、解答

- 方法一：部分遍历

    理解：就是第二个非递减序列的开头

    ```java
    package array.easy;
    
    public class Offer11 {
        // 方法一：遍历或者排序得到最小值
    
        // 方法二：思路，求第二个非递减数组的开头
        public int minArray(int[] numbers) {
            if (numbers.length <= 0 || numbers == null){
                return 0;
            }
            if (numbers.length == 1){
                return numbers[0];
            }
            // 从第二个遍历，哪一个值比前一个小，则就是该值
            for (int i = 1; i < numbers.length; i++) {
                if (numbers[i] < numbers[i - 1]){
                    return numbers[i];
                }
            }
            // 如果上面都没有，则可能没有旋转，则最小值就是第一个值。
            return numbers[0];
        }
    }
    
    ```

    

- 方法二：二分法

    将遍历法的 线性级别 时间复杂度降低至 对数级别 。

    ![image-20200410212122044](Offer11.%E6%97%8B%E8%BD%AC%E6%95%B0%E7%BB%84%E7%9A%84%E6%9C%80%E5%B0%8F%E6%95%B0%E5%AD%97.resource/image-20200410212122044.png)

![image-20200410212142245](Offer11.%E6%97%8B%E8%BD%AC%E6%95%B0%E7%BB%84%E7%9A%84%E6%9C%80%E5%B0%8F%E6%95%B0%E5%AD%97.resource/image-20200410212142245.png)

```java
package array.easy;

public class Offer11 {
    // 方法三：二分查找
    public int minArray2(int[] numbers) {
        if (numbers.length <= 0 || numbers == null) {
            return 0;
        }
        int begin = 0;
        int end = numbers.length - 1;
        while (begin < end) {
            int mid = (begin + end) >>> 1;
            if (numbers[mid] < numbers[end]) {
                end = mid;
            } else if (numbers[mid] > numbers[end]) {
                begin = mid + 1;
            } else {
                end--;
            }
        }
        return numbers[begin];
    }
}
```

