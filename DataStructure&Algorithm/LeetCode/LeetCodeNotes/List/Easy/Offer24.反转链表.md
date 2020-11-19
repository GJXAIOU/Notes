---
layout:     post
title:      Offer24.反转链表
subtitle:   List.easy
date:       2020-02-24
author:     GJXAIOU
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - 链表
	- 完成
---

# Offer24.反转链表

## 一、题目

定义一个函数，输入一个链表的头节点，反转该链表并输出反转后链表的头节点。

- 示例：

> 输入: 1->2->3->4->5->NULL
> 输出: 5->4->3->2->1->NULL

- 限制：

    0 <= 节点个数 <= 5000

**注意**：本题与主站 206 题相同：https://leetcode-cn.com/problems/reverse-linked-list/

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/fan-zhuan-lian-biao-lcof
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 二、解答

```java
package list.easy;

/**
 * @Author GJXAIOU
 * @Date 2020/2/24 15:39
 */
public class Offer24 {
    class ListNode {
        int val;
        ListNode next;

        ListNode(int x) {
            val = x;
        }
    }


    public ListNode reverseList(ListNode head) {
        if (head == null) {
            return head;
        }

        ListNode pre = null;
        ListNode cur = head;
        ListNode temp = null;
        while (cur != null) {
            temp = cur.next;
            cur.next = pre;
            pre = cur;
            cur = temp;
        }
        return pre;
    }
}

```

