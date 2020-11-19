---
layout:     post
title:      面试题59 - I. 滑动窗口的最大值
subtitle:   Array.easy
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



# [面试题59 - I. 滑动窗口的最大值](https://leetcode-cn.com/problems/hua-dong-chuang-kou-de-zui-da-zhi-lcof/)

## 一、题目

给定一个数组 `nums` 和滑动窗口的大小 `k`，请找出所有滑动窗口里的最大值。

**示例:**

```
输入: nums = [1,3,-1,-3,5,3,6,7], 和 k = 3
输出: [3,3,5,5,6,7] 
解释: 

  滑动窗口的位置                最大值
---------------               -----
[1  3  -1] -3  5  3  6  7       3
 1 [3  -1  -3] 5  3  6  7       3
 1  3 [-1  -3  5] 3  6  7       5
 1  3  -1 [-3  5  3] 6  7       5
 1  3  -1  -3 [5  3  6] 7       6
 1  3  -1  -3  5 [3  6  7]      7
```

**提示：**

你可以假设 *k* 总是有效的，在输入数组不为空的情况下，1 ≤ k ≤ 输入数组的大小。

注意：本题与主站 239 题相同：https://leetcode-cn.com/problems/sliding-window-maximum/

## 二、解答

### 方法一：滑动窗口

#### （一）滑动窗口的概念

**就是一个数组------有 L 和 R 指 针，默认两个指针均位于数组的最左边即下标为 `-1` 的位置， 当有数字进入时 R 向右移动。当有数字删除时则 L 向右移动。且 L 和 R 不会回退且 L 不能到 R 右边。**

思路： **双端队列（即双向链表）**：双端队列中既需要放置数据又需要放置位置下标，本质上**存放下标**就行，对应值根据下标从数组中就能获取。

![增加元素过程](Offer59I.%E6%BB%91%E5%8A%A8%E7%AA%97%E5%8F%A3%E7%9A%84%E6%9C%80%E5%A4%A7%E5%80%BC.resource/%E5%A2%9E%E5%8A%A0%E5%85%83%E7%B4%A0%E8%BF%87%E7%A8%8B.jpg)

**上图运行逻辑为：** 如果想实现窗口内最大值的更新结构： 使得双端队列保证从大到小的顺序 

- 当放入元素时候 ：从尾部添加

    （R增加） **头部始终存放的是当前最大的元素**--- 如果即将进入双端队列的元素（因为 R 增加而从数组中取出放入双端队列中的元素）比上一个进入双端队列的元素小，则从尾部进入，新进入的元素直接连接在后面，否则 原来双端队列的尾部一直弹出（包含相等情况，因为晚过期） 直到为即将要放入的元素找到合适的位置，或者整个队列为空，然后放入新加入的元素。（见上面示例）

- 当删除元素时候：从头部删除

    （L增加） ---L向右移---（index下标一定得保留）**则需要检查当前头部元素index是否过期 若过期则需要从头部进行弹出**

![img](Offer59I.%E6%BB%91%E5%8A%A8%E7%AA%97%E5%8F%A3%E7%9A%84%E6%9C%80%E5%A4%A7%E5%80%BC.resource/20180621232650862.png)

**时间复杂度**：因为从到到位滑过，每个数只会进队列一次，出队列一次，在队列中删除的数是不找回的，因此时间复杂度为：$$O(N)$$

```java
package array.easy;

import java.util.LinkedList;

/**
 * @author GJXAIOU
 * @create 2020/04/13 21:06
 */
public class Offer59I {

    public int[] maxSlidingWindow(int[] nums, int k) {
        if (nums == null || k < 1 || nums.length < k) {
            return new int[0];
        }

        LinkedList<Integer> maxList = new LinkedList<Integer>();
        int[] res = new int[nums.length - k + 1];
        int index = 0;
        for (int i = 0; i < nums.length; i++) {
            // 更新双端队列，如果双端队列不为空，并且尾结点(存的是下标)对应数组中的值是否小于等于当前值
            while (!maxList.isEmpty() && nums[maxList.peekLast()] <= nums[i]) {
                maxList.pollLast();
            }
            // 上面一直弹出，直到不符合然后加上当前值。
            maxList.addLast(i);
          
// 当过期的时候（当窗口形成之后再扩充才算过期）即窗口长度>k，窗口形成过程中不会过期, i-k 表示过期的下标
            if (maxList.peekFirst() == i - k) {
                maxList.pollFirst();
            }
            // 判断下标过期
            if (i >= k - 1) {
                // 当窗口已经形成了，记录每一步的res
                res[index++] = nums[maxList.peekFirst()];
            }
        }
        return res;
    }
}

```



### 方法二：动态规划

这是另一个 O(N) 的算法。本算法的优点是不需要使用 数组 / 列表 之外的任何数据结构。

算法的思想是将输入数组分割成有 k 个元素的块。
若 n % k != 0，则最后一块的元素个数可能更少。

![image-20200413220531028](Offer59I.%E6%BB%91%E5%8A%A8%E7%AA%97%E5%8F%A3%E7%9A%84%E6%9C%80%E5%A4%A7%E5%80%BC.resource/image-20200413220531028.png)

开头元素为 i ，结尾元素为 j 的当前滑动窗口可能在一个块内，也可能在两个块中。

![image-20200413220548653](Offer59I.%E6%BB%91%E5%8A%A8%E7%AA%97%E5%8F%A3%E7%9A%84%E6%9C%80%E5%A4%A7%E5%80%BC.resource/image-20200413220548653.png)

情况 1 比较简单。 建立数组 left， 其中 left[j] 是从块的开始到下标 j 最大的元素，方向 左->右。

![image-20200413220606753](Offer59I.%E6%BB%91%E5%8A%A8%E7%AA%97%E5%8F%A3%E7%9A%84%E6%9C%80%E5%A4%A7%E5%80%BC.resource/image-20200413220606753.png)

为了处理更复杂的情况 2，我们需要数组 right，其中 right[j] 是从块的结尾到下标 j 最大的元素，方向 右->左。right 数组和 left 除了方向不同以外基本一致。

![image-20200413220638602](Offer59I.%E6%BB%91%E5%8A%A8%E7%AA%97%E5%8F%A3%E7%9A%84%E6%9C%80%E5%A4%A7%E5%80%BC.resource/image-20200413220638602.png)

两数组一起可以提供两个块内元素的全部信息。考虑从下标 i 到下标 j的滑动窗口。 根据定义，right[i] 是左侧块内的最大元素， left[j] 是右侧块内的最大元素。因此滑动窗口中的最大元素为 max(right[i], left[j])。

![image-20200413220650458](Offer59I.%E6%BB%91%E5%8A%A8%E7%AA%97%E5%8F%A3%E7%9A%84%E6%9C%80%E5%A4%A7%E5%80%BC.resource/image-20200413220650458.png)

算法

算法十分直截了当：

从左到右遍历数组，建立数组 left。

从右到左遍历数组，建立数组 right。

建立输出数组 max(right[i], left[i + k - 1])，其中 i 取值范围为 (0, n - k + 1)。

```java
class Solution {
  public int[] maxSlidingWindow(int[] nums, int k) {
    int n = nums.length;
    if (n * k == 0) return new int[0];
    if (k == 1) return nums;

    int [] left = new int[n];
    left[0] = nums[0];
    int [] right = new int[n];
    right[n - 1] = nums[n - 1];
    for (int i = 1; i < n; i++) {
      // from left to right
      if (i % k == 0) left[i] = nums[i];  // block_start
      else left[i] = Math.max(left[i - 1], nums[i]);

      // from right to left
      int j = n - i - 1;
      if ((j + 1) % k == 0) right[j] = nums[j];  // block_end
      else right[j] = Math.max(right[j + 1], nums[j]);
    }

    int [] output = new int[n - k + 1];
    for (int i = 0; i < n - k + 1; i++)
      output[i] = Math.max(left[i + k - 1], right[i]);

    return output;
  }
}
```

