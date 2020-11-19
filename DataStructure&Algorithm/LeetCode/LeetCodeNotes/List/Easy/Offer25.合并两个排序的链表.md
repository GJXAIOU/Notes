---
layout:     post
title:      Offer25.合并两个排序的链表
subtitle:   List.easy
date:       2020-02-24
author:     GJXAIOU
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - 链表
	- 双指针
	- 完成
---

# Offer25.合并两个排序的链表

## 一、题目

输入两个==递增==排序的链表，合并这两个链表并使新链表中的节点仍然是递增排序的。

- 示例1：

    >输入：1->2->4, 1->3->4
    >输出：1->1->2->3->4->4

- 限制：

    0 <= 链表长度 <= 1000

注意：本题与主站 21 题相同：https://leetcode-cn.com/problems/merge-two-sorted-lists/

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/he-bing-liang-ge-pai-xu-de-lian-biao-lcof
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 二、解答

### 方法：双指针

逐个比较值：那个小哪个链接在后面。==新建一个头结点==。

```java
package list.easy;

/**
 * @Author GJXAIOU
 * @Date 2020/2/24 15:47
 */
public class Offer25 {
    class ListNode {
        int val;
        ListNode next;

        ListNode(int x) {
            val = x;
        }
    }

    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        if (l1 == null) {
            return l2;
        }
        if (l2 == null) {
            return l1;
        }

        // 新建一个链表头
        ListNode preHead = new ListNode(0);
        ListNode head = preHead;
        // 比较两个值相等，哪个小哪个跟在链表后面
        while (l1 != null && l2 != null) {
            if (l1.val <= l2.val) {
                head.next = l1;
                l1 = l1.next;
            } else {
                head.next = l2;
                l2 = l2.next;
            }
            head = head.next;
        }
        // 两个总得有一个剩余
        while (l1 != null) {
            head.next = l1;
            l1 = l1.next;
            head = head.next;
        }
        while (l2 != null) {
            head.next = l2;
            l2 = l2.next;
            head = head.next;
        }
        return preHead.next;
    }
}
```
