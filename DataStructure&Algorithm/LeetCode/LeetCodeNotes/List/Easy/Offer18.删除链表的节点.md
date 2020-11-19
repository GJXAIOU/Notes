---
layout:     post
title:      Offer18. 删除链表的节点
subtitle:   List.easy
date:       2020-01-30
author:     GJXAIOU
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - 链表
	- 完成
---

## Offer18. 删除链表的节点

## 一、题目

给定单向链表的头指针和一个要删除的节点的值，定义一个函数删除该节点。

返回删除后的链表的头节点。

**注意：**此题对比原题有改动

**示例 1:**

```
输入: head = [4,5,1,9], val = 5
输出: [4,1,9]
解释: 给定你链表中值为 5 的第二个节点，那么在调用了你的函数之后，该链表应变为 4 -> 1 -> 9.
```

**示例 2:**

```
输入: head = [4,5,1,9], val = 1
输出: [4,5,9]
解释: 给定你链表中值为 1 的第三个节点，那么在调用了你的函数之后，该链表应变为 4 -> 5 -> 9.
```

**说明：**

- 题目保证链表中节点的值互不相同
- 若使用 C 或 C++ 语言，你不需要 `free` 或 `delete` 被删除的节点

## 二、解答

### 方法：指针

要删除的元素值相应节点的位置有三处：

- 位于链表头部
- 位于链表中间
- 位于链表尾部

要删除目标节点，关键在于找到待删除节点的前驱节点，改变前驱节点的下一个节点的指向即可删除节点，所以使用双指针，一个指向前驱节点，一个指向当前节点

```java
package list.easy;

/**
 * @Author GJXAIOU
 * @Date 2020/2/25 10:51
 */
public class Offer18 {
    class ListNode {
        int value;
        ListNode next;

        ListNode(int x) {
            value = x;
        }
    }
    
    public ListNode deleteNode(ListNode head, int val) {
        if (head == null) {
            return head;
        }
        ListNode pre = null;
        ListNode cur = head;
        // 如果是头结点
        if (head.value == val) {
            return head.next;
        }

        // 找到要删除节点的前驱节点
        while (cur != null && (cur.value != val)) {
            pre = cur;
            cur = cur.next;
        }
        // 如果是最后一个，则将 pre.next 即最后一个结点置为 null
        if (cur.next == null) {
            pre.next = null;
            // 如果不是最后一个，跳过该结点
        } else {
            pre.next = cur.next;
        }
        return head;
    }
}
```

