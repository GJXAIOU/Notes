---
layout:     post
title:      Offer52.两个链表的第一个公共节点
subtitle:   List.easy
date:       2020-02-28
author:     GJXAIOU
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - 链表
	- 分治算法
	- 完成
---

# Offer52.两个链表的第一个公共节点

### 一、题目
输入两个链表，找出它们的第一个公共节点。

如下面的两个链表：

<img src="Offer52.%E4%B8%A4%E4%B8%AA%E9%93%BE%E8%A1%A8%E7%9A%84%E7%AC%AC%E4%B8%80%E4%B8%AA%E5%85%AC%E5%85%B1%E8%8A%82%E7%82%B9.resource/160_statement.png" alt="img" style="zoom: 67%;" />

在节点 c1 开始相交。

- 示例 1：

<img src="Offer52.%E4%B8%A4%E4%B8%AA%E9%93%BE%E8%A1%A8%E7%9A%84%E7%AC%AC%E4%B8%80%E4%B8%AA%E5%85%AC%E5%85%B1%E8%8A%82%E7%82%B9.resource/160_statement-1590152904101.png" alt="img" style="zoom:67%;" />

> 输入：intersectVal = 8, listA = [4,1,8,4,5], listB = [5,0,1,8,4,5], skipA = 2, skipB = 3
> 输出：Reference of the node with value = 8
> 输入解释：相交节点的值为 8 （注意，如果两个列表相交则不能为 0）。从各自的表头开始算起，链表 A 为 [4,1,8,4,5]，链表 B 为 [5,0,1,8,4,5]。在 A 中，相交节点前有 2 个节点；在 B 中，相交节点前有 3 个节点

- 示例 2：

    <img src="Offer52.%E4%B8%A4%E4%B8%AA%E9%93%BE%E8%A1%A8%E7%9A%84%E7%AC%AC%E4%B8%80%E4%B8%AA%E5%85%AC%E5%85%B1%E8%8A%82%E7%82%B9.resource/160_example_2.png" alt="img" style="zoom: 67%;" />

> 输入：intersectVal = 2, listA = [0,9,1,2,4], listB = [3,2,4], skipA = 3, skipB = 1
> 输出：Reference of the node with value = 2
> 输入解释：相交节点的值为 2 （注意，如果两个列表相交则不能为 0）。从各自的表头开始算起，链表 A 为 [0,9,1,2,4]，链表 B 为 [3,2,4]。在 A 中，相交节点前有 3 个节点；在 B 中，相交节点前有 1 个节点。

- 示例 3：

    <img src="Offer52.%E4%B8%A4%E4%B8%AA%E9%93%BE%E8%A1%A8%E7%9A%84%E7%AC%AC%E4%B8%80%E4%B8%AA%E5%85%AC%E5%85%B1%E8%8A%82%E7%82%B9.resource/160_example_3.png" alt="img" style="zoom: 67%;" />

> 输入：intersectVal = 0, listA = [2,6,4], listB = [1,5], skipA = 3, skipB = 2
> 输出：null
> 输入解释：从各自的表头开始算起，链表 A 为 [2,6,4]，链表 B 为 [1,5]。由于这两个链表不相交，所以 intersectVal 必须为 0，而 skipA 和 skipB 可以是任意值。
> 解释：这两个链表不相交，因此返回 null。

- 注意：

    - 如果两个链表没有交点，返回 null.
    - 在返回结果后，两个链表仍须保持原有的结构。
    - 可假定整个链表结构中没有循环。
    - 程序尽量满足 O(n) 时间复杂度，且仅用 O(1) 内存。


本题与主站 160 题相同：https://leetcode-cn.com/problems/intersection-of-two-linked-lists/

## 二、解答

### 方法二：双指针法

长链表先走与短链表差值的步数，然后一起走，如果在 null 之前相等则相交

```java
package list.easy;

/**
 * @Author GJXAIOU
 * @Date 2020/2/28 12:29
 */
public class Offer52 {
    class ListNode {
        int val;
        ListNode next;

        ListNode(int x) {
            val = x;
            next = null;
        }
    }

    public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        if (headA == null || headB == null) {
            return null;
        }
        ListNode cur1 = headA;
        ListNode cur2 = headB;

        // count 是两个链表差值关系
        int count = 0;
        while (cur1 != null) {
            count++;
            cur1 = cur1.next;
        }

        while (cur2 != null) {
            count--;
            cur2 = cur2.next;
        }
        
        // 如果走到最后两个节点不相等，则两个链表不想交
        if (cur1 != cur2) {
            return null;
        }

        // 在定位哪一个是长链表，哪一个是短链表
        // cur1 指向长链表的头部，cur2 指向短链表的头部
        cur1 = count > 0 ? headA : headB;
        cur2 = cur1 == headA ? headB : headA;
        count = Math.abs(count);

        // 长的先走 count 步，然后短的再走，最后返回的 cur1 就是他们进入的第一个相交节点
        while (count != 0) {
            count--;
            cur1 = cur1.next;
        }

        while (cur1 != cur2) {

            cur1 = cur1.next;
            cur2 = cur2.next;
        }
        return cur1;
    }
}

```



**精简版本**

node1 到达链表 headA 的末尾时，重新定位到链表 headB 的头结点；当 node2 到达链表 headB 的末尾时，重新定位到链表 headA 的头结点。

这样，当它们相遇时，所指向的结点就是第一个公共结点。

-  时间复杂度：O(M+N)。
-  空间复杂度：O(1)。

![Offer52](Offer52.%E4%B8%A4%E4%B8%AA%E9%93%BE%E8%A1%A8%E7%9A%84%E7%AC%AC%E4%B8%80%E4%B8%AA%E5%85%AC%E5%85%B1%E8%8A%82%E7%82%B9.resource/Offer52.gif)

```java
package list.easy;


/**
 * @Author GJXAIOU
 * @Date 2020/2/28 12:29
 */
public class Offer52 {
    class ListNode {
        int val;
        ListNode next;

        ListNode(int x) {
            val = x;
            next = null;
        }
    }

    // 方法二：
    public ListNode getIntersectionNode2(ListNode headA, ListNode headB) {
        if (headA == null || headB == null) {
            return null;
        }
        ListNode cur1 = headA;
        ListNode cur2 = headB;
        while (cur1 != cur2) {
            cur1 = cur1 == null ? headB : cur1.next;
            cur2 = cur2 == null ? headA : cur2.next;
        }
        return cur1;
    }

}

```

