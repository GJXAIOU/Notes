# ListSummary

- 链表中的快慢结点

    ```java
     ListNode slow = head.next;
     ListNode fast = head.next.next;
    ```

- 如果返回一个链表并且是新建的话：**如果需要修改链表的话**。

    ```java
    // 新建一个链表头
    ListNode preHead = new ListNode(0);
    ListNode head = preHead;
    // 下面都使用 head 运算，最后返回 preHead.next
    return preHead.next;
    }
    ```

- 反转链表

    ```java
    public ListNode reverseList(ListNode head) {
        ListNode pre = null;
        ListNode cur = head;
        ListNode next = null;
        while (cur != null) {
            // 记录当前结点的下一个结点
            next = cur.next;
            // 将当前结点的下一节点指向前一个结点
            cur.next = pre;
            // pre 和 cur 结点均向后移动
            pre = cur;
            cur = next;
        }
        // 一定是返回 pre
        return pre;
    }
    ```

    