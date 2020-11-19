---
layout:     post
title:      Offer09. 用两个栈实现队列
subtitle:   Stack.easy
date:       2020-02-24
author:     GJXAIOU
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - 栈
	- 完成
---



# Offer09. 用两个栈实现队列

## 一、题目

用两个栈实现一个队列。队列的声明如下，请实现它的两个函数 `appendTail` 和 `deleteHead` ，分别完成在队列尾部插入整数和在队列头部删除整数的功能。(若队列中没有元素，`deleteHead` 操作返回 -1 )

**示例 1：**

```
输入：
["CQueue","appendTail","deleteHead","deleteHead"]
[[],[3],[],[]]
输出：[null,null,3,-1]
```

**示例 2：**

```
输入：
["CQueue","deleteHead","appendTail","appendTail","deleteHead","deleteHead"]
[[],[],[5],[2],[],[]]
输出：[null,-1,null,null,5,2]
```

**提示：**

- `1 <= values <= 10000`
- `最多会对 appendTail、deleteHead 进行 10000 次调用`



## 二、解答

首先将数据放入 push 栈中，形成：5,4,3,2,1，然后将其全部拿出 push 到 pop 栈中，变成了 1,2,3,4，5；然后在 pop 栈中依次从栈顶取出所有元素即可；
**要求**：如果 push 栈中决定往 pop 栈中倒数据，则一次必须倒完；
如果 pop 栈中仍有数据，则 push 栈不能往 pop 栈中倒数据；

```java
package stack.easy;

import java.util.Stack;

/**
 * @Author GJXAIOU
 * @Date 2020/2/24 16:17
 */
public class Offer09 {
    class CQueue {
        Stack<Integer> pushStack;
        Stack<Integer> popStack;

        public CQueue() {
            pushStack = new Stack<>();
            popStack = new Stack<>();
        }

        public void appendTail(int value) {
            pushStack.push(value);
        }

        // 删除就是如果 pop 栈不为空弹出头，如果为空则将 push 栈中所有元素一次性倒入 pop 栈中，然后弹出。
        public int deleteHead() {
            if (pushStack.isEmpty() && popStack.isEmpty()) {
                return -1;
            } else if (popStack.isEmpty()) {
                while (!pushStack.isEmpty()) {
                    popStack.push(pushStack.pop());
                }
            }
            return popStack.pop();
        }
    }

/**
 * Your CQueue object will be instantiated and called as such:
 * CQueue obj = new CQueue();
 * obj.appendTail(value);
 * int param_2 = obj.deleteHead();
 */
}

```

