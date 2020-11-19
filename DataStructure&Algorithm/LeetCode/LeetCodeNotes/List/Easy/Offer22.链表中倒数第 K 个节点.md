---
layout:     post
title:      Offer22. 链表中倒数第k个节点
subtitle:   List.easy
date:       2020-02-24
author:     GJXAIOU
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - 剑指 Offer
    - 链表
	- 双指针
    - Easy 
---

# Offer22. 链表中倒数第k个节点

## 一、题目

输入一个链表，输出该链表中倒数第 `k` 个节点。为了符合大多数人的习惯，本题从 `1` 开始计数，即链表的尾节点是倒数第 `1` 个节点。例如，一个链表有 `6` 个节点，从头节点开始，它们的值依次是 `1、2、3、4、5、6`。这个链表的倒数第 `3` 个节点是值为 `4` 的节点。

- 示例：

    给定一个链表: `1->2->3->4->5`, 和 `k = 2`。

    返回链表 `4->5`。

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/lian-biao-zhong-dao-shu-di-kge-jie-dian-lcof
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。



## 二、解答

### 方法：快慢指针

- 定义两个指针，快指针 fast， 慢指针 slow .
- 让 fast 先向前移动 k 个位置，然后 slow 和 fast 再一起向前移动 .
- 当 fast 到达链表尾部，返回 low .

```java
package list.easy;

import java.util.List;

/**
 * @Author GJXAIOU
 * @Date 2020/2/24 13:18
 */
public class Offer22 {
    class ListNode {
        int val;
        ListNode next;

        ListNode(int x) {
            val = x;
        }
    }

    public ListNode getKthFromEnd(ListNode head, int k) {
        if (head == null) {
            return head;
        }

        ListNode slow = head;
        ListNode fast = head;
        // 快指针先跑 K 步
        while (k != 0) {
            fast = fast.next;
            k--;
        }
        // 快慢指针一起跑
        while (fast != null) {
            fast = fast.next;
            slow = slow.next;
        }
        return slow;
    }
}

```
