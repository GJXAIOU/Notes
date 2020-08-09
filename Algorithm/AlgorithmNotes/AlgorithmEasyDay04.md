# AlgorithmEasyDay04：二叉树

[TOC]

## ==一、实现二叉树的先序、中序、后序遍历==

二叉树的示例结构为：

![二叉树结构](AlgorithmEasyDay04.resource/%E4%BA%8C%E5%8F%89%E6%A0%91%E7%BB%93%E6%9E%84.png)

实际函数访问结点的顺序为：`1 2 4 4 4 2 5 5 5 2 1 3 6 6 6 3 7 7 7 3 1` 

### （一）使用递归的方式

**以先序遍历二叉树为例**，可以发现递归方式==首先尝试打印当前结点的值，随后尝试打印左子树，打印完左子树后尝试打印右子树==，递归过程的 `base case` 是当某个结点为空时停止子过程的展开。这种递归尝试是由二叉树本身的结构所决定的，因为二叉树上的任意结点都可看做一棵二叉树的根结点（即使是叶子结点，也可以看做是一棵左右子树为空的二叉树根结点）。

观察先序、中序、后序三个递归方法你会发现，不同点在于打印当前结点的值这一操作的时机。**你会发现==每个结点会被访问三次==**：进入方法时算一次、递归处理左子树完成之后返回时算一次、递归处理右子树完成之后返回时算一次。因此在 `preOrderRecursive` 中**将打印语句放到方法开始时就产生了先序遍历**；在`midOrderRecursive`中，**将打印语句放到递归处理左子树完成之后就产生了中序遍历**；同理**放在第三次访问时候打印就是后续遍历**。

**实现代码：**

```java
package com.gjxaiou.easy.day04;

import java.util.Stack;

/**
 * 递归版和非递归版本的先序、中序、后序遍历
 * @author GJXAIOU
 */
public class PreInPosTraversal {

    public static class Node {
        public int value;
        public Node left;
        public Node right;

        public Node(int data) {
            this.value = data;
        }
    }

    /**
     * 递归版本实现先序、中序、后序遍历，唯一变化就是 print() 函数位置不同。
     */
    // 先序遍历：中（当前节点）、左、右
    public static void preOrderRecur(Node head) {
        if (head == null) {
            return;
        }
        System.out.print(head.value + " ");
        preOrderRecur(head.left);
        preOrderRecur(head.right);
    }

    // 中序遍历：左、中（当前结点）、右
    public static void inOrderRecur(Node head) {
        if (head == null) {
            return;
        }
        inOrderRecur(head.left);
        System.out.print(head.value + " ");
        inOrderRecur(head.right);
    }

    // 后序遍历：左、右、中（当前结点）
    public static void posOrderRecur(Node head) {
        if (head == null) {
            return;
        }
        posOrderRecur(head.left);
        posOrderRecur(head.right);
        System.out.print(head.value + " ");
    }

    public static void main(String[] args) {
        Node head = new Node(5);
        head.left = new Node(3);
        head.right = new Node(8);
        head.left.left = new Node(2);
        head.left.right = new Node(4);
        head.left.left.left = new Node(1);
        head.right.left = new Node(7);
        head.right.left.left = new Node(6);
        head.right.right = new Node(10);
        head.right.right.left = new Node(9);
        head.right.right.right = new Node(11);

        // 递归版本
        System.out.println("==============递归版本==============");
        System.out.print("pre-order: ");
        preOrderRecur(head);
        System.out.println();
        System.out.print("in-order: ");
        inOrderRecur(head);
        System.out.println();
        System.out.print("pos-order: ");
        posOrderRecur(head);
        System.out.println();
    }
}
```

程序测试执行结果：

```java
==============递归版本==============
pre-order: 5 3 2 1 4 8 7 6 10 9 11 
in-order:  1 2 3 4 5 6 7 8 9 10 11 
pos-order: 1 2 4 3 6 7 9 11 10 8 5 
```




### （二）非递归方式
因为递归使用的是栈，这里只需要自己实现一个栈结构即可；

- **先序遍历**
  ==拿到一棵树的根结点后，首先打印该结点的值，然后将其非空右孩子、非空左孩子依次压栈。栈非空循环：从栈顶弹出结点（一棵子树的根节点）并打印其值，再将其非空右孩子、非空左孩子依次压栈==。即压栈的顺序和打印的顺序是相反的，压栈是先根结点，然后有右孩子就压右孩子、有左孩子就压左孩子，这是利用栈的后进先出。每次获取到一棵子树的根节点之后就可以获取其左右孩子，因此无需保留其信息，直接弹出并打印，然后保留其左右孩子到栈中即可。
  **举个栗子**：先压入 1 ；然后弹出 1，先压入节点 1 的右孩子 3，然后压入左孩子 2，然后弹出栈顶 2，然后压入节点 2 的右孩子和左孩子，现在栈中顺序为：4 5 3，然后弹出 4，压入 4 的右孩子和左孩子，发现都没有，弹出栈顶 5，压入 5 的右孩子和左孩子，发现都没有，弹出栈顶 3，然后压入 3 的右孩子和左孩子，然后弹出栈顶 6，然后压入 6 的右、左孩子，都没有，弹出 7，压入右孩子、左孩子，没有，弹出发现为空，结束；


- **中序遍历**
  对于一棵树，将该树的左边界全部压栈，`root` 的走向是只要左孩子不为空就走向左孩子。当左孩子为空时弹出栈顶结点（此时该结点是一棵左子树为空的树的根结点，根据中序遍历可以直接打印该结点，然后中序遍历该结点的右子树）打印，如果该结点的右孩子非空（说明有右子树），那么将其右孩子压栈，这个右孩子又可能是一棵子树的根节点，因此将这棵子树的左边界压栈，这时回到了开头，以此类推。

  ==当前结点不为空的时候，将当前结点压入栈中，结点指针左移，指向左结点，直到当前结点为空，则从栈中将栈顶弹出打印，然后指针右移==；因为中序遍历输出顺序为左、中、右，所以先压当前的，然后压左边，弹出，在押右边。
  **举个栗子**：首先头结点 1 不为空，将该结点压入，然后指向节点 2，然后压入结点 2，然后压入结点 4，然后指向 null，因为栈不等于空，还得遍历，进入 else，弹出栈顶为 结点 4 ，指针指向 4，然后指向结点 4 的右结点为 null，然后再次弹出节点 2，然后指向结点 2 的右子节点 5，然后。。。。

<img src="AlgorithmEasyDay04.resource/LeetCode94.gif" alt="LeetCode94" style="zoom:67%;" />


- **后序遍历**
  
  
  - **思路一**：准备两个栈，一个栈用来保存遍历时的结点信息，另一个栈用来排列后根顺序（根节点先进栈，右孩子再进，左孩子最后进）。
  - **思路二**：只用一个栈。借助两个变量`head`和`stackTopNode`，head 代表最近一次打印过的结点，`stackTopNode`代表栈顶结点。首先将根结点压栈，此后栈非空循环，令 `stackTopNode` 等于栈顶元素（`stackTopNode = stack.peek()`）执行以下三个分支：
    
    - `stackTopNode`的左右孩子是否与`head`相等，如果都不相等，说明`stackTopNode`的左右孩子都不是最近打印过的结点，同时因为左右孩子分别为左右子树的根节点，根据后序遍历的特点（左右中），左右子树肯定都没打印过，那么将左孩子压栈（打印左子树）。
      
      - 分支 1 没有执行说明`stackTopNode`的左孩子要么不存在；要么左子树刚打印过了；要么右子树刚打印过了。这时如果是前两种情况中的一种，那就轮到打印右子树了，因此如果`stackTopNode`的右孩子非空就压栈。
      - 如果前两个分支都没执行，说明`stackTopNode`的左右子树都打印完了，因此弹出并打印`stackTopNode`结点，更新一下`head`。
  
  **整体思路：** 后序遍历是左右中打印（需要用到两个栈，代码方式二是用的一个栈，不太好理解）；因为先序遍历是中、左、右，就是当前结点先压右孩子然后压左孩子，那么先使用中右左顺序（就是当前结果先压左孩子然后压右孩子）压入所有元素，同时将原来使用打印的语句更改为：将该元素存放到另一个栈中，但是不打印，全部遍历完成之后，将栈中的元素全部打印出来即可，这时候的顺序就是左右中了即为后序遍历；

**非递归方式实现**

```java
package com.gjxaiou.easy.day04;

import java.util.Stack;

/**
 * 递归版和非递归版本的先序、中序、后序遍历
 * @author GJXAIOU
 */
public class PreInPosTraversal {

    public static class Node {
        public int value;
        public Node left;
        public Node right;

        public Node(int data) {
            this.value = data;
        }
    }

    /**
     * 非递归版本实现先序、中序、后序遍历
     */
    // 非递归版本实现先序遍历
    public static void preOrderUnRecur(Node head) {
        System.out.print("pre-order: ");
        if (head != null) {
            // 准备一个栈
            Stack<Node> stack = new Stack<Node>();
            // 首先放入头结点
            stack.add(head);
            while (!stack.isEmpty()) {
                head = stack.pop();
                System.out.print(head.value + " ");
                if (head.right != null) {
                    stack.push(head.right);
                }
                if (head.left != null) {
                    stack.push(head.left);
                }
            }
        }
        System.out.println();
    }

    // 非递归方式实现中序遍历
    public static void inOrderUnRecur(Node head) {
        System.out.print("in-order: ");
        if (head != null) {
            Stack<Node> stack = new Stack<Node>();
            while (!stack.isEmpty() || head != null) {
                // 首先压入头结点然后一直压入所有的左孩子
                if (head != null) {
                    stack.push(head);
                    head = head.left;
                    // 弹出栈顶然后一直弹出右孩子
                } else {
                    head = stack.pop();
                    System.out.print(head.value + " ");
                    head = head.right;
                }
            }
        }
        System.out.println();
    }

    // 后序遍历方式一：先使用中左右的顺序将元素压入栈中，然后遍历栈弹出即可
    public static void posOrderUnRecur1(Node head) {
        System.out.print("pos-order: ");
        if (head != null) {
            Stack<Node> stack1 = new Stack<Node>();
            Stack<Node> stack2 = new Stack<Node>();
            stack1.push(head);
            // 这里和前序遍历类似，只不过左右节点压入顺序相反
            while (!stack1.isEmpty()) {
                head = stack1.pop();
                // 将打印语句替换为压栈语句
                stack2.push(head);
                if (head.left != null) {
                    stack1.push(head.left);
                }
                if (head.right != null) {
                    stack1.push(head.right);
                }
            }

            // 逐个弹出栈 stack2 中元素即为左、右、中顺序
            while (!stack2.isEmpty()) {
                System.out.print(stack2.pop().value + " ");
            }
        }
        System.out.println();
    }

    // 后续遍历方式二：使用一个栈
    // head 表示最近被打印的结点
    public static void posOrderUnRecur2(Node head) {
        System.out.print("pos-order: ");
        if (head != null) {
            Stack<Node> stack = new Stack<Node>();
            stack.push(head);
            Node stackTopNode = null;
            while (!stack.isEmpty()) {
                // stackTopNode 表示栈顶结点
                stackTopNode = stack.peek();
                // 如果栈顶结点的左右结点和最近打印的结点都不相等：说明栈顶结点的左右孩子都不是最近打印的结点，同样由于左右孩子分别为左右子树的头结点，
                // 根据后序遍历的特点（左右中），则左右子树都没有被打印过，所以压入左子树。
                if (stackTopNode.left != null && head != stackTopNode.left && head != stackTopNode.right) {
                    stack.push(stackTopNode.left);
                    // 如果上面没有执行：左子树不存在或者左子树刚刚打印过或者右子树刚刚打印过，如果是前两种情况，就接着打印右子树
                } else if (stackTopNode.right != null && head != stackTopNode.right) {
                    stack.push(stackTopNode.right);
                    // 上面还没有执行说明左右子树都空或者都打印完了，弹出该结点打印，然后更新。
                } else {
                    System.out.print(stack.pop().value + " ");
                    head = stackTopNode;
                }
            }
        }
        System.out.println();
    }

    public static void main(String[] args) {
        Node head = new Node(5);
        head.left = new Node(3);
        head.right = new Node(8);
        head.left.left = new Node(2);
        head.left.right = new Node(4);
        head.left.left.left = new Node(1);
        head.right.left = new Node(7);
        head.right.left.left = new Node(6);
        head.right.right = new Node(10);
        head.right.right.left = new Node(9);
        head.right.right.right = new Node(11);

        // 非递归版本
        System.out.println("============非递归版本=============");
        preOrderUnRecur(head);
        inOrderUnRecur(head);
        posOrderUnRecur1(head);
        posOrderUnRecur2(head);
    }
}
```
程序运行结果：
```java
============非递归版本=============
pre-order: 5 3 2 1 4 8 7 6 10 9 11 
in-order:  1 2 3 4 5 6 7 8 9 10 11 
pos-order: 1 2 4 3 6 7 9 11 10 8 5 
pos-order: 1 2 4 3 6 7 9 11 10 8 5
```



## 二、直观的打印一颗二叉树

 * 在节点值两边加上特定的字符串标记来区分孩子和位置以及之间的位置关系：
 * HXH：表示头结点 X；vYv：表示节点 Y 是左下方最近节点的孩子；^Z^：表示节点 Z 是左上方最近节点的孩子
 * 遍历树的顺序为：右子树 -> 根 -> 左子树；
 * 避免节点值长度不同影响对其，规定每个节点值长度为固定值（这里规定为 10）

```java
package com.gjxaiou.easy.day04;

/**
 * 最直观的打印二叉树
 *
 * @author GJXAIOU
 */
public class PrintBinaryTree {

    public static class Node {
        public int value;
        public Node left;
        public Node right;

        public Node(int data) {
            value = data;
        }
    }

    public static void printTree(Node head) {
        System.out.println("Binary Tree:");
        printInOrder(head, 0, "H", 10);
        System.out.println();
    }

    /**
     * @param head      ：传入的节点
     * @param treeHeight：层数（根节点为 0）
     * @param to       ：表示的特定节点  H表示根节点   ^表示父亲节点在左上方　v表示父亲节点在左下方
     * @param totalLength ：指定每一个节点打印的宽度(总宽度)
     */
    public static void printInOrder(Node head, int treeHeight, String to, int totalLength) {
        if (head == null) {
            return;
        }
        // 递归遍历右子树
        printInOrder(head.right, treeHeight + 1, "v", totalLength);

        // 在节点值两边加上标识符
        String val = to + head.value + to;
        int valueLength = val.length();
        // 节点值左边的空格数：（总长度 - 节点值长度）/ 2
        int leftSpaceLength = (totalLength - valueLength) / 2;
        // 节点值右边的空格数
        int rightSpaceLength = totalLength - valueLength - leftSpaceLength;
        val = getSpace(leftSpaceLength) + val + getSpace(rightSpaceLength);
        // treeHeight * totalLength 为打印的节点前空格长度
        System.out.println(getSpace(treeHeight * totalLength) + val);

        // 递归遍历左子树
        printInOrder(head.left, treeHeight + 1, "^", totalLength);
    }

    public static String getSpace(int num) {
        String space = " ";
        StringBuffer buf = new StringBuffer("");
        for (int i = 0; i < num; i++) {
            buf.append(space);
        }
        return buf.toString();
    }

    public static void main(String[] args) {
        Node head = new Node(1);
        head.left = new Node(-22);
        head.right = new Node(3);
        head.left.left = new Node(2);
        head.right.left = new Node(55);
        head.right.right = new Node(66);
        head.left.left.right = new Node(7);
        printTree(head);

        head = new Node(1);
        head.left = new Node(1);
        head.right = new Node(1);
        head.left.left = new Node(1);
        head.right.left = new Node(1);
        head.right.right = new Node(1);
        head.left.left.right = new Node(1);
        printTree(head);
    }
}
```
程序运行结果
```java
Binary Tree:
                       v66v   
             v3v    
                       ^55^   
   H1H    
            ^-22^   
                                 v7v    
                       ^2^    

Binary Tree:
                       v1v    
             v1v    
                       ^1^    
   H1H    
             ^1^    
                                 v1v    
                       ^1^    
```
注释：打印结果中， `HXH`表示头结点 X，`VYV`表示 Y 是左下方最近结点的孩子；`^Z^` 表示 Z 是左上方最近结点的孩子；


## 三、在二叉树中找到一个节点的后继节点
【题目】 现在有一种新的二叉树节点类型如下：
```java
public class Node { 
    public int value; 
    public Node left;
    public Node right; 
    public Node parent;
    public Node(int data) { 
        this.value = data; 
    }
}
```
> 该结构比普通二叉树节点结构多了一个指向父节点的 parent 指针。假设有一 棵 Node 类型的节点组成的二叉树，树中每个节点的 parent 指针都正确地指向自己的父节点，头节点的 parent 指向 null。只给一个在二叉树中的某个节点 node，请实现返回 node 的后继节点的函数。

【概念】==在二叉树中，前驱结点和后继结点是按照二叉树中两个结点被**中序遍历**的先后顺序来划分的==。以下面的树形结构为例，中序遍历结果为：`4 2 5 1 6 3 7`，则 `1` 的后继结点为 `6`，前驱结点为 `5`；

![二叉树结构](AlgorithmEasyDay04.resource/%E4%BA%8C%E5%8F%89%E6%A0%91%E7%BB%93%E6%9E%84.png)


【解答方法】
- **一般方法**：==通过给定结点的 parent 指针一直找到根结点，然后从根节点中序遍历整个树，在遍历到该结点的时候标记一下，那么下一个要打印的结点就是该结点的后继结点。最后根据中序遍历的结果得到每个结点的后继结点；==
	- 时间复杂度为：O(N)
- **改进解法**：
  例如 5 的后继结点为 1，能不能通过 5 2 1 这样的结构找到，即当前结点距离后继结点的距离长度就是所要的复杂度；**具体方案为：**
  - ==如果一个结点有右子树，则后继结点就是其右子树上最左边的结点==；例：1 的后继为 6；
  - ==如果没有右子树，则使用 parent 指针一直向上找（指向当前结点的指针和 parent 指针同时向上移动），一直到当前指向的结点是当前结点父结点的左孩子就停止，则该父结点就是所求结点的后继结点；==
  **示例图**：以结点 11 为例，其父结点为 5，11 不是 5 的左结点，向上，5 的父结点为 2，5 不是 2 的左结点，向上，2 是 1 的左结点，则节点 11 的后继结点为 1；
  ![题目三寻找后继结点示例](AlgorithmEasyDay04.resource/%E9%A2%98%E7%9B%AE%E4%B8%89%E5%AF%BB%E6%89%BE%E5%90%8E%E7%BB%A7%E7%BB%93%E7%82%B9%E7%A4%BA%E4%BE%8B.png)

### 补充问题：求直接前驱结点的方案如下

- 如果结点有左子树，该结点的前驱就是左子树的最右边结点；
- 如果结点没有左子树，就一直往上找，直到指向的结点是其父结点的右孩子为止；


```java
package com.gjxaiou.easy.day04;

/**
 * 求二叉树中某个节点的后继结点
 *
 * @author GJXAIOU
 */
public class SuccessorNode {

    public static class Node {
        public int value;
        public Node left;
        public Node right;
        public Node parent;

        public Node(int value) {
            this.value = value;
        }
    }

    public static Node getSuccessorNode(Node root) {
        if (root == null) {
            return root;
        }
        // 说明当前结点有右子树，则后继结点为右子树的最左结点
        if (root.right != null) {
            return getLeftMost(root.right);
        } else {
            Node parent = root.parent;
            // 当前节点是父结点的左孩子则停止；整棵树最右边的节点是没有后继的，例如节点7，最终一直上升到节点1，但是节点1的 parent指向null，所有最后返回o
            // null表示没有后继；
            while (parent != null && parent.left != root) {
                root = parent;
                parent = root.parent;
            }
            return parent;
        }
    }

    // 获取以该结点为根节点的树上最左的结点
    public static Node getLeftMost(Node node) {
        if (node == null) {
            return node;
        }
        while (node.left != null) {
            node = node.left;
        }
        return node;
    }

    //////  测试程序
    public static void main(String[] args) {
        Node head = new Node(6);
        head.parent = null;
        head.left = new Node(3);
        head.left.parent = head;
        head.left.left = new Node(1);
        head.left.left.parent = head.left;
        head.left.left.right = new Node(2);
        head.left.left.right.parent = head.left.left;
        head.left.right = new Node(4);
        head.left.right.parent = head.left;
        head.left.right.right = new Node(5);
        head.left.right.right.parent = head.left.right;
        head.right = new Node(9);
        head.right.parent = head;
        head.right.left = new Node(8);
        head.right.left.parent = head.right;
        head.right.left.left = new Node(7);
        head.right.left.left.parent = head.right.left;
        head.right.right = new Node(10);
        head.right.right.parent = head.right;

        Node test = head.left.left;
        System.out.println(test.value + " next: " + getSuccessorNode(test).value);
        test = head.left.left.right;
        System.out.println(test.value + " next: " + getSuccessorNode(test).value);
        test = head.left;
        System.out.println(test.value + " next: " + getSuccessorNode(test).value);
        test = head.left.right;
        System.out.println(test.value + " next: " + getSuccessorNode(test).value);
        test = head.left.right.right;
        System.out.println(test.value + " next: " + getSuccessorNode(test).value);
        test = head;
        System.out.println(test.value + " next: " + getSuccessorNode(test).value);
        test = head.right.left.left;
        System.out.println(test.value + " next: " + getSuccessorNode(test).value);
        test = head.right.left;
        System.out.println(test.value + " next: " + getSuccessorNode(test).value);
        test = head.right;
        System.out.println(test.value + " next: " + getSuccessorNode(test).value);
        test = head.right.right; // 10's next is null
        System.out.println(test.value + " next: " + getSuccessorNode(test));
    }
}
```
程序运行结果为：
```java
1 next: 2
2 next: 3
3 next: 4
4 next: 5
5 next: 6
6 next: 7
7 next: 8
8 next: 9
9 next: 10
10 next: null
```



## 四、二叉树的序列化和反序列化

- **序列化：** 就是将在内存中构建的二叉树，怎么使用文件进行存储（一般变成字符串进行处理）；
- **反序列化：** 就是将文件中序列化的二叉树取出重新还原成二叉树；
- 二叉树的序列化要注意的两个点如下：
	-  每序列化一个结点数值之后都应该加上一个结束符表示一个结点序列化的终止，如`!`或者 `_` 等等。
	- 不能忽视空结点的存在，可以使用一个占位符如`#`表示空结点的序列化。

**解决方案：**

- 方案一：使用先序遍历、中序遍历、后续遍历其中之一，同样反序列化也应该使用对应的同样方式；
- 方案二：使用按层序列化；

**举个栗子：**这里以先序遍历方法作为测试，下面为测试树的结构：
![题目四序列化示例](AlgorithmEasyDay04.resource/%E9%A2%98%E7%9B%AE%E5%9B%9B%E5%BA%8F%E5%88%97%E5%8C%96%E7%A4%BA%E4%BE%8B.png)

先序遍历结果：`1_2_4_#_#_5_#_#_3_6_#_#_7_#_#_`
其中：`#`表示该结点为空，`_`表示值的结束符，这两个都可以使用其他符号进行代替；如果不使用 `#` 来表示空节点就无法表示整个树中结点值都是一样的情况；
**反序列化**：从数组的第一个元素开始，因为是先序遍历，因此该结点一定是根节点，然后开始搭建左子树，1 的左孩子是 2，2 的左孩子是 4，然后是 #，则 4 的左孩子没有，返回到节点 4，下一个还是# ，则 4 的右孩子也没有，则回到 2，。。。。

**按层序列化**：上面图片中的二叉树按层遍历的结果为：`1_2_3_4_#_#_5_#_#_#_#_`

```java
package com.gjxaiou.easy.day04;

import java.util.LinkedList;
import java.util.Queue;

/**
 * 序列号和反序列化二叉树
 *
 * @author GJXAIOU
 */
public class SerializeAndReconstructTree {

    public static class Node {
        public int value;
        public Node left;
        public Node right;

        public Node(int value) {
            this.value = value;
        }
    }

    // 使用递归版本的先序遍历实现二叉树的序列化
    public static String serialByPre(Node head) {
        if (head == null) {
            return "#_";
        }
        // 形成字符串格式为：结点值_
        String res = head.value + "_";
        res += serialByPre(head.left);
        res += serialByPre(head.right);
        return res;
    }

    // 对先序遍历序列化得到的字符串进行反序列化
    public static Node reconByPreString(String preStr) {
        // 首先将字符串分割
        String[] values = preStr.split("_");
        Queue<String> queue = new LinkedList<String>();
        for (int i = 0; i != values.length; i++) {
            // add() 和 offer() 都是向队列中添加一个元素，如果想在一个满的队列中加入一个新项，调用 add() 方法就会抛出一个 unchecked 异常，而调用
            //offer() 方法会返回 false
            queue.offer(values[i]);
        }
        return reconPreOrder(queue);
    }

    // 通过队列建出树
    public static Node reconPreOrder(Queue<String> queue) {
        String value = queue.poll();
        if (value.equals("#")) {
            return null;
        }
        Node head = new Node(Integer.valueOf(value));
        head.left = reconPreOrder(queue);
        head.right = reconPreOrder(queue);
        return head;
    }


    // 按层遍历二叉树的序列号
    public static String serialByLevel(Node head) {
        if (head == null) {
            return "#_";
        }
        String res = head.value + "_";
        Queue<Node> queue = new LinkedList<Node>();
        // 首先加入头结点，弹出，加入左结点、加入右结点
        queue.offer(head);
        while (!queue.isEmpty()) {
            head = queue.poll();
            if (head.left != null) {
                res += head.left.value + "_";
                queue.offer(head.left);
            } else {
                res += "#_";
            }
            if (head.right != null) {
                res += head.right.value + "_";
                queue.offer(head.right);
            } else {
                res += "#_";
            }
        }
        return res;
    }

    // 按层遍历二叉树的反序列化
    public static Node reconByLevelString(String levelStr) {
        String[] values = levelStr.split("_");
        int index = 0;
        Node head = generateNodeByString(values[index++]);
        Queue<Node> queue = new LinkedList<Node>();
        if (head != null) {
            queue.offer(head);
        }
        Node node = null;
        while (!queue.isEmpty()) {
            node = queue.poll();
            node.left = generateNodeByString(values[index++]);
            node.right = generateNodeByString(values[index++]);
            if (node.left != null) {
                queue.offer(node.left);
            }
            if (node.right != null) {
                queue.offer(node.right);
            }
        }
        return head;
    }

    public static Node generateNodeByString(String val) {
        if (val.equals("#")) {
            return null;
        }
        return new Node(Integer.valueOf(val));
    }

    // 打印二叉树树形结构--------
    public static void printTree(Node head) {
        System.out.println("Binary Tree:");
        printInOrder(head, 0, "H", 10);
        System.out.println();
    }

    public static void printInOrder(Node head, int height, String to, int len) {
        if (head == null) {
            return;
        }
        printInOrder(head.right, height + 1, "v", len);
        String val = to + head.value + to;
        int lenM = val.length();
        int lenL = (len - lenM) / 2;
        int lenR = len - lenM - lenL;
        val = getSpace(lenL) + val + getSpace(lenR);
        System.out.println(getSpace(height * len) + val);
        printInOrder(head.left, height + 1, "^", len);
    }

    public static String getSpace(int num) {
        String space = " ";
        StringBuffer buf = new StringBuffer("");
        for (int i = 0; i < num; i++) {
            buf.append(space);
        }
        return buf.toString();
    }
    //-----------------------------

    public static void main(String[] args) {
        Node head = null;
        printTree(head);

        String pre = serialByPre(head);
        System.out.println("通过先序遍历序列化二叉树: " + pre);
        head = reconByPreString(pre);
        System.out.print("通过先序遍历反序列二叉树, ");
        printTree(head);

        String level = serialByLevel(head);
        System.out.println("通过层序遍历序列化二叉树: " + level);
        head = reconByLevelString(level);
        System.out.print("通过层序遍历反序列化二叉树, ");
        printTree(head);
        System.out.println("====================================");

        head = new Node(1);
        printTree(head);

        pre = serialByPre(head);
        System.out.println("通过先序遍历序列化二叉树: " + pre);
        head = reconByPreString(pre);
        System.out.print("通过先序遍历反序列二叉树, ");
        printTree(head);

        level = serialByLevel(head);
        System.out.println("通过层序遍历序列化二叉树: " + level);
        head = reconByLevelString(level);
        System.out.print("通过层序遍历反序列化二叉树, ");
        printTree(head);

        System.out.println("====================================");

        head = new Node(1);
        head.left = new Node(2);
        head.right = new Node(3);
        head.left.left = new Node(4);
        head.right.right = new Node(5);
        printTree(head);

        pre = serialByPre(head);
        System.out.println("通过先序遍历序列化二叉树: " + pre);
        head = reconByPreString(pre);
        System.out.print("通过先序遍历反序列二叉树, ");
        printTree(head);

        level = serialByLevel(head);
        System.out.println("通过层序遍历序列化二叉树: " + level);
        head = reconByLevelString(level);
        System.out.print("通过层序遍历反序列化二叉树, ");
        printTree(head);

        System.out.println("====================================");
    }
}
```
程序运行结果
```java
Binary Tree:

通过先序遍历序列化二叉树: #_
通过先序遍历反序列二叉树, Binary Tree:

通过层序遍历序列化二叉树: #_
通过层序遍历反序列化二叉树, Binary Tree:

====================================
Binary Tree:
   H1H    

通过先序遍历序列化二叉树: 1_#_#_
通过先序遍历反序列二叉树, Binary Tree:
   H1H    

通过层序遍历序列化二叉树: 1_#_#_
通过层序遍历反序列化二叉树, Binary Tree:
   H1H    

====================================
Binary Tree:
                       v5v    
             v3v    
   H1H    
             ^2^    
                       ^4^    

通过先序遍历序列化二叉树: 1_2_4_#_#_#_3_#_5_#_#_
通过先序遍历反序列二叉树, Binary Tree:
                       v5v    
             v3v    
   H1H    
             ^2^    
                       ^4^    

通过层序遍历序列化二叉树: 1_2_3_4_#_#_5_#_#_#_#_
通过层序遍历反序列化二叉树, Binary Tree:
                       v5v    
             v3v    
   H1H    
             ^2^    
                       ^4^    

====================================
```





## 五、折纸问题

【**题目**】 请把一段纸条竖着放在桌子上，然后从纸条的下边向上方对折 1 次，压出折痕后展开。此时 折痕是凹下去的，即折痕突起的方向指向纸条的背面。如果从纸条的下边向上方连续对折 2 次，压出折痕后展开，此时有三条折痕，从上到下依次是下折痕、下折痕和上折痕。
给定一 个输入参数 N，代表纸条都从下边向上方连续对折 N 次，请从上到下打印所有折痕的方向。 例如：N = 1时，打印： down ；N = 2 时，打印： down down up

【**解答**】本质上就是将下面这棵二叉树按照  左-> 中  -> 右 的顺序进行遍历；

![生成的树结构_20200113150339](AlgorithmEasyDay04.resource/%E7%94%9F%E6%88%90%E7%9A%84%E6%A0%91%E7%BB%93%E6%9E%84_20200113150339.jpg)

```java
package com.gjxaiou.easy.day04;

/**
 * 折纸问题
 *
 * @author GJXAIOU
 */
public class PaperFolding {

    public static void printAllFolds(int foldTotalTime) {
        printProcess(1, foldTotalTime, true);
    }

    public static void printProcess(int currentTime, int foldTotalTime, boolean down) {
        if (currentTime > foldTotalTime) {
            return;
        }
        // 按照中序遍历顺序
        printProcess(currentTime + 1, foldTotalTime, true);
        System.out.print(down ? "down " : "up ");
        printProcess(currentTime + 1, foldTotalTime, false);
    }

    public static void main(String[] args) {
        printAllFolds(1);
        System.out.println();
        printAllFolds(2);
        System.out.println();
        printAllFolds(3);
        System.out.println();
        printAllFolds(4);
    }
}
```
程序运行结果
```java
down 
down down up 
down down up down down up up 
down down up down down up up down down down up up down up up 
```



## ==六、判断一棵二叉树是否是平衡二叉树==

### （一）平衡二叉树概念

当二叉树的任意一棵子树的左子树的高度和右子树的高度相差不超过 1 时，该二叉树为平衡二叉树。

### （二）解答方案：确保以每个结点为根节点的树都是平衡二叉树

将问题分解为：只要保证以二叉树的每个结点为根节点的树是否平衡；而遍历到每个结点时，要想知道以该结点为根结点的子树是否是平衡二叉树，首先判断该结点的左子树是否平衡，然后判断该结点的右子树是否平衡，如果两个子树都平衡，分别计算左子树和右子树的高度。因此我们要收集两个信息：

- 该结点的左子树、右子树是否是平衡二叉树。

- 左右子树的高度分别是多少，相差是否超过 1。

那么我们来到某个结点时（子过程），我们需要向上层（父过程）**返回的信息就是该结点为根结点的树是否是平衡二叉树以及以该结点为根的树高度**，这样的话，父过程就能继续向上层返回应该收集的信息。

### （三）方案改进：通过递归收集左右子树信息

因为使用递归进行二叉树的遍历的时候，每个递归函数会到一个节点 3 次：首先会到一次，然后访问左子树之后会回来一次，最后访问右子树之后会回来一次； 对于二叉树使用递归的时候，首先想办法收集一下左子树上的信息，然后想办法收集一下右子树上的信息，最后将这些信息进行整合皆可以得到该结点所在的整棵树符不符合标准；本题中的标准就是判断该数是否平衡；

```java
package com.gjxaiou.easy.day04;

/**
 * 判断是否为平衡树
 *
 * @author GJXAIOU
 */
public class IsBalancedTree {

    public static class Node {
        public int value;
        public Node left;
        public Node right;

        public Node(int value) {
            this.value = value;
        }
    }

    public static boolean isBalance(Node head) {
        // 这里需要使用数组，因为作为参数传入 getHeight 函数，同时在该函数中修改了值，需要同步修改这里的值返回值才会变化。
        boolean[] res = new boolean[1];
        res[0] = true;
        getHeight(head, 1, res);
        return res[0];
    }

    public static int getHeight(Node head, int level, boolean[] res) {
        if (head == null) {
            return level;
        }
        int leftHeight = getHeight(head.left, level + 1, res);
        if (!res[0]) {
            return level;
        }
        int rightHeight = getHeight(head.right, level + 1, res);
        if (!res[0]) {
            return level;
        }
        // 如果左右高度差大于 1 则返回 false
        if (Math.abs(leftHeight - rightHeight) > 1) {
            res[0] = false;
        }
        return Math.max(leftHeight, rightHeight);
    }

    public static void main(String[] args) {
        Node head = new Node(1);
        head.left = new Node(2);
        head.right = new Node(3);
        head.left.left = new Node(4);
        head.left.right = new Node(5);
        head.right.left = new Node(6);
        head.right.right = new Node(7);

        System.out.println(isBalance(head));
    }
}
```
程序运行结果
```java
true
```

### ==注：递归过程总结==

递归很好用，该题中的递归用法也是一种经典用法，可以高度套路：

1.  分析问题的解决需要哪些步骤（这里是遍历每个结点，确认每个结点为根节点的子树是否为平衡二叉树）
2.  确定递归：父问题是否和子问题相同
3.  子过程要收集哪些信息
4.  本次递归如何利用子过程返回的信息得到本过程要返回的信息
5.  处理 `base case`

  

## 七、判断是否是搜索二叉树或者完全二叉树

### （一）搜索二叉树 （BST：Binary Search Tree）
**二叉查找树**也称为**二叉搜索树**、**有序二叉树**（ordered binary tree）或**排序二叉树**（sorted binary tree），是指一棵空树或者具有下列性质的二叉树：

1. 若任意节点的左子树不空，则左子树上所有节点的值均小于它的根节点的值；
2. 若任意节点的右子树不空，则右子树上所有节点的值均大于或等于它的根节点的值；
3. 任意节点的左、右子树也分别为二叉查找树；
4. 并且整棵树上任意两个结点的值不同（重复的结点的值可以通过 List 的形式进行存放）。

二叉查找树相比于其他数据结构的优势在于查找、插入的[时间复杂度较低。为 $O(log^N)$，最坏情况为：O(N)



【**解答方案**】 **根据二叉树的中序遍历看是否递增**
根据定义，==搜索二叉树的**中序遍历**打印将是一个升序序列==。因此我们可以利用二叉树的中序遍历的非递归方式，比较中序遍历时相邻两个结点的大小，只要有一个结点的值小于其后继结点的那就不是搜索二叉树。
下面代码提供的是递归版本，如果想要非递归版本，只要将中序遍历的非递归版本的打印方法去掉，然后改成与上一个结点值的比较即可（可以采用一个变量保存上一个结点的值）；


### （二）完全二叉树 CBT 
【**完全二叉树 CBT 概念**】

==如果二叉树上某个结点有右孩子无左孩子则一定不是完全二叉树；如果二叉树上某个结点有左孩子而没有右孩子，那么该结点所在的那一层上，该结点右侧的所有结点应该都是叶子结点，否则不是完全二叉树。==

【**解答方案**】 将二叉树**按层遍历**，如果结点有右孩子但是没有左孩子则一定不是完全二叉树，如果不符合上面，则接着判断是否是有左孩子没有右孩子或者是两个都没有，如果是这种情况则该结点下面的所有节点都必须是叶子结点；否则就不是完全二叉树；


```java
package com.gjxaiou.easy.day04;

import java.util.LinkedList;
import java.util.Queue;

/**
 * 判断是否为搜索二叉树、完全二叉树
 *
 * @author GJXAIOU
 */
public class IsBSTAndCBT {

    public static class Node {
        public int value;
        public Node left;
        public Node right;

        public Node(int data) {
            this.value = data;
        }
    }

    /**
     * 判断是否为搜索二叉树
     * 方法：中序遍历为递增
     */
    public static boolean isBST(Node head) {
        if (head == null) {
            return true;
        }
        boolean res = true;
        Node pre = null;
        Node cur1 = head;
        Node cur2 = null;
        while (cur1 != null) {
            cur2 = cur1.left;
            // 如果当前结点的左孩子存在
            if (cur2 != null) {
                // 当前结点的左孩子的右孩子不为空并且不等于当前结点
                while (cur2.right != null && cur2.right != cur1) {
                    cur2 = cur2.right;
                }
                if (cur2.right == null) {
                    cur2.right = cur1;
                    cur1 = cur1.left;
                    continue;
                } else {
                    cur2.right = null;
                }
            }
            // 中序遍历之后如果不是递增数列，就不是搜索二叉树
            if (pre != null && pre.value > cur1.value) {
                res = false;
            }
            pre = cur1;
            cur1 = cur1.right;
        }
        return res;
    }

    /**
     * 判断是否为完全二叉树
     */
    public static boolean isCBT(Node head) {
        if (head == null) {
            return true;
        }
        Queue<Node> queue = new LinkedList<Node>();
        boolean leaf = false;
        Node left = null;
        Node right = null;
        queue.offer(head);
        while (!queue.isEmpty()) {
            head = queue.poll();
            left = head.left;
            right = head.right;
            // 有右孩子没有左孩子一定不是；如果两个孩子都没有，该结点下面结点必须是叶子结点；有左孩子没有右孩子则下面所有节点都是叶子节点。
            if ((leaf && (left != null || right != null)) || (left == null && right != null)) {
                return false;
            }
            if (left != null) {
                queue.offer(left);
            }
            if (right != null) {
                queue.offer(right);
            } else {
                // 左等于空或者右等于空 则开启。因为上面代码已经去掉左等于空的情况，因此这里只需要判断右是否为空；
                leaf = true;
            }
        }
        return true;
    }
    

    // 测试程序：打印二叉树
    public static void printTree(Node head) {
        System.out.println("Binary Tree:");
        printInOrder(head, 0, "H", 17);
        System.out.println();
    }

    public static void printInOrder(Node head, int height, String to, int len) {
        if (head == null) {
            return;
        }
        printInOrder(head.right, height + 1, "v", len);
        String val = to + head.value + to;
        int lenM = val.length();
        int lenL = (len - lenM) / 2;
        int lenR = len - lenM - lenL;
        val = getSpace(lenL) + val + getSpace(lenR);
        System.out.println(getSpace(height * len) + val);
        printInOrder(head.left, height + 1, "^", len);
    }

    public static String getSpace(int num) {
        String space = " ";
        StringBuffer buf = new StringBuffer("");
        for (int i = 0; i < num; i++) {
            buf.append(space);
        }
        return buf.toString();
    }

    public static void main(String[] args) {
        Node head = new Node(4);
        head.left = new Node(2);
        head.right = new Node(6);
        head.left.left = new Node(1);
        head.left.right = new Node(3);
        head.right.left = new Node(5);

        printTree(head);
        System.out.println(isBST(head));
        System.out.println(isCBT(head));

    }
}
```
程序运行结果
```java
Binary Tree:
                        v6v       
                                         ^5^       
       H4H       
                                         v3v       
                        ^2^       
                                         ^1^       

true
true
```



### （一）问题变形：求一棵二叉树的最远距离

【注】如果在二叉树中，小明从结点 A 出发，既可以往上走到达它的父结点，又可以往下走到达它的子结点，那么小明从结点 A 走到结点 B 最少要经过的结点个数（包括 A 和 B）叫做 A 到 B 的距离，任意两结点所形成的距离中，最大的叫做树的最大距离。

**高度套路化**：

大前提：如果对于以该树的任意结点作为头结点的子树中，如果我们能够求得所有这些子树的最大距离，那么答案就在其中。

对于该树的任意子树，其最大距离的求解分为以下三种情况：

- 该树的最大距离是左子树的最大距离。
- 该树的最大距离是右子树的最大距离。
- 该树的最大距离是从左子树的最深的那个结点经过该树的头结点走到右子树的最深的那个结点。

要从子树收集的信息：

- 子树的最大距离
- 子树的深度

示例代码：

```java
package nowcoder.advanced.advanced_class_05;

public class Code_03_MaxDistanceInTree {

    public static class Node {
        public int value;
        public Node left;
        public Node right;

        public Node(int data) {
            this.value = data;
        }
    }

    public static int maxDistance(Node head) {
        int[] record = new int[1];
        return posOrder(head, record);
    }

    // 最大距离，高度
    public static class ReturnType {
        public int maxDistance;
        public int h;

        public ReturnType(int m, int h) {
            this.maxDistance = m;
            ;
            this.h = h;
        }
    }

    public static ReturnType process(Node head) {
        if (head == null) {
            return new ReturnType(0, 0);
        }
        ReturnType leftReturnType = process(head.left);
        ReturnType rightReturnType = process(head.right);

        // 可能性 3，可能性 1，可能性 2；
        int includeHeadDistance = leftReturnType.h + 1 + rightReturnType.h;
        int p1 = leftReturnType.maxDistance;
        int p2 = rightReturnType.maxDistance;
        // 最终距离
        int resultDistance = Math.max(Math.max(p1, p2), includeHeadDistance);
        // 最大深度，左右最大深度 + 自己
        int hitself = Math.max(leftReturnType.h, leftReturnType.h) + 1;
        return new ReturnType(resultDistance, hitself);
    }

    public static int posOrder(Node head, int[] record) {
        if (head == null) {
            record[0] = 0;
            return 0;
        }
        int lMax = posOrder(head.left, record);
        int maxfromLeft = record[0];
        int rMax = posOrder(head.right, record);
        int maxFromRight = record[0];
        int curNodeMax = maxfromLeft + maxFromRight + 1;
        record[0] = Math.max(maxfromLeft, maxFromRight) + 1;
        return Math.max(Math.max(lMax, rMax), curNodeMax);
    }

    public static void main(String[] args) {
        Node head1 = new Node(1);
        head1.left = new Node(2);
        head1.right = new Node(3);
        head1.left.left = new Node(4);
        head1.left.right = new Node(5);
        head1.right.left = new Node(6);
        head1.right.right = new Node(7);
        head1.left.left.left = new Node(8);
        head1.right.left.right = new Node(9);
        System.out.println(maxDistance(head1));

        Node head2 = new Node(1);
        head2.left = new Node(2);
        head2.right = new Node(3);
        head2.right.left = new Node(4);
        head2.right.right = new Node(5);
        head2.right.left.left = new Node(6);
        head2.right.right.right = new Node(7);
        head2.right.left.left.left = new Node(8);
        head2.right.right.right.right = new Node(9);
        System.out.println(maxDistance(head2));
    }
}

```

> 高度套路化：列出可能性->从子过程收集的信息中整合出本过程要返回的信息->返回



### （二）最大搜索二叉子树

【**题目**】

给定一棵二叉树的头节点 head，所有结点的值都是不一样的，请返回最大搜索二叉子树的大小（即含有结点最多的搜索二叉树）

【总结】

==求一棵树的最大XXXXX，转换为求以每个结点为根节点树的最大XXXX，最终答案一定在其中。== 即这类题一般都有一个**大前提**：**假设对于以树中的任意结点为头结点的子树，我们都能求得其最大搜索二叉子树的结点个数，那么答案一定就在其中**。

这种题目的解题过程分为三步:

- **列出所有可能性**；
- **列出结点需要的信息，并整合信息(成一个结构体)**；
- **改递归 ，先假设左和右都给我信息(黑盒)，然后怎么利用左边和右边的信息组出来我该返回的信息，最后`basecase`(边界)填什么**；



**步骤一：可能性**

而对于以任意结点为头结点的子树，其最大搜索二叉子树的求解分为三种情况（**列出可能性**）：

- 整棵树的最大搜索二叉子树存在于左子树中。这要求其左子树中存在最大搜索二叉子树，而其右子树不存在。
- 整棵树的最大搜索二叉子树存在于右子树中。这要求其右子树中存在最大搜索二叉子树，而其左子树不存在。
- 最整棵二叉树的最大搜索二叉子树就是其本身。这需要其左子树就是一棵搜索二叉子树（左树头结点为 `node.left`）且左子树的最大值结点比头结点小、其右子树就是一棵搜索二叉子树（右结点的头结点为 `node.right`）且右子树的最小值结点比头结点大。

**步骤二：**要想区分这三种情况，我们需要收集的信息：

左边搜索二叉树大小、右边搜索二叉树大小、左边搜索二叉树头部、右边搜索二叉树头部、左树最大值，右树最小值，==但是求得每个结点之后进行递归，要创建结点唯一要搜索的结构，所有要化简==

- 子树中是否存在最大搜索二叉树，如果存在则记录搜索二叉树的大小；
- 子树的头结点，搜索二叉树头结点；
- 子树的最大值结点
- 子树的最小值结点

**步骤三**：因此我们就可以开始我们的高度套路了：

1. 将要从子树收集的信息封装成一个`ReturnData`，代表处理完这一棵子树要向上级返回的信息。
2. 假设我利用子过程收集到了子树的信息，接下来根据子树的信息和分析问题时列出的情况加工出当前这棵树要为上级提供的所有信息，并返回给上级（**整合信息**）。
3. 确定`base case`，子过程到子树为空时，停。

根据上面高度套路的分析，可以写出解决这类问题高度相似的代码：

```java
package com.gjxaiou.advanced.day04;

public class BiggestSubBSTInTree {

    // 树节点的结构
    public static class Node {
        public int value;
        public Node left;
        public Node right;

        public Node(int data) {
            this.value = data;
        }
    }

    // 对于递归函数返回的结构类型
    public static class ReturnType {
        // 左右搜索二叉子树节点数的最大值
        public int size;
        // 左右搜索二叉子树节点数较大的子树的头结点
        public Node head;
        // 右树的最小值
        public int min;
        // 左树的最大值
        public int max;

        public ReturnType(int size, Node head, int min, int max) {
            this.size = size;
            this.head = head;
            this.min = min;
            this.max = max;
        }
    }

    /**
     * 方式一：使用后续遍
     */
    public static Node biggestSubBST1(Node head) {
        return process(head).head;
    }

    // 采用后序遍历的方式进行递归，因为需要左右的信息来构造头部的信息
    public static ReturnType process(Node head) {
        if (head == null) {
            // 返回最大值，比较谁小的时候不会影响比较结果
            return new ReturnType(0, null, Integer.MAX_VALUE, Integer.MIN_VALUE);
        }

        // 分别将左右孩子传入之后返回左树和右树的信息
        Node left = head.left;
        ReturnType leftSubTressInfo = process(left);

        Node right = head.right;
        ReturnType rightSubTressInfo = process(right);

        // 可能性 3
        int includeItSelf = 0;
        // 如果左树上最大搜索二叉子树的头部是该节点的左孩子，右树上最大搜索二叉子树的头部是该节点的右孩子，同时左树上最大值值小于该结点，右树上值大于该节点
        if (leftSubTressInfo.head == left
                && rightSubTressInfo.head == right
                && head.value > leftSubTressInfo.max
                && head.value < rightSubTressInfo.min
        ) {
            includeItSelf = leftSubTressInfo.size + 1 + rightSubTressInfo.size;
        }
        // 可能性 1
        int p1 = leftSubTressInfo.size;
        // 可能性 2
        int p2 = rightSubTressInfo.size;
        // 该节点返回的最大值是三种可能性中最大值
        int maxSize = Math.max(Math.max(p1, p2), includeItSelf);
        // 返回值头部 P1 大说明来自左子树最大搜索二叉树头部，P2 大则说明......
        Node maxHead = p1 > p2 ? leftSubTressInfo.head : rightSubTressInfo.head;
        if (maxSize == includeItSelf) {
            maxHead = head;
        }

        return new ReturnType(maxSize,
                maxHead,
                Math.min(Math.min(leftSubTressInfo.min, rightSubTressInfo.min), head.value),
                Math.max(Math.max(leftSubTressInfo.max, rightSubTressInfo.max), head.value));
    }


    /*
     优化方式：将三个值以数组形式存放，
     */
    public static Node biggestSubBST(Node head) {
        // 0->size, 1->min, 2->max
        int[] record = new int[3];
        return posOrder(head, record);
    }

    public static Node posOrder(Node head, int[] record) {
        if (head == null) {
            record[0] = 0;
            record[1] = Integer.MAX_VALUE;
            record[2] = Integer.MIN_VALUE;
            return null;
        }
        int value = head.value;
        Node left = head.left;
        Node right = head.right;
        Node lBST = posOrder(left, record);
        int lSize = record[0];
        int lMin = record[1];
        int lMax = record[2];
        Node rBST = posOrder(right, record);
        int rSize = record[0];
        int rMin = record[1];
        int rMax = record[2];
        record[1] = Math.min(rMin, Math.min(lMin, value)); // lmin, value, rmin -> min
        record[2] = Math.max(lMax, Math.max(rMax, value)); // lmax, value, rmax -> max
        if (left == lBST && right == rBST && lMax < value && value < rMin) {
            record[0] = lSize + rSize + 1;
            return head;
        }
        record[0] = Math.max(lSize, rSize);
        return lSize > rSize ? lBST : rBST;
    }


    // for test -- print tree
    public static void printTree(Node head) {
        System.out.println("Binary Tree:");
        printInOrder(head, 0, "H", 17);
        System.out.println();
    }

    public static void printInOrder(Node head, int height, String to, int len) {
        if (head == null) {
            return;
        }
        printInOrder(head.right, height + 1, "v", len);
        String val = to + head.value + to;
        int lenM = val.length();
        int lenL = (len - lenM) / 2;
        int lenR = len - lenM - lenL;
        val = getSpace(lenL) + val + getSpace(lenR);
        System.out.println(getSpace(height * len) + val);
        printInOrder(head.left, height + 1, "^", len);
    }

    public static String getSpace(int num) {
        String space = " ";
        StringBuffer buf = new StringBuffer("");
        for (int i = 0; i < num; i++) {
            buf.append(space);
        }
        return buf.toString();
    }

    public static void main(String[] args) {

        Node head = new Node(6);
        head.left = new Node(1);
        head.left.left = new Node(0);
        head.left.right = new Node(3);
        head.right = new Node(12);
        head.right.left = new Node(10);
        head.right.left.left = new Node(4);
        head.right.left.left.left = new Node(2);
        head.right.left.left.right = new Node(5);
        head.right.left.right = new Node(14);
        head.right.left.right.left = new Node(11);
        head.right.left.right.right = new Node(15);
        head.right.right = new Node(13);
        head.right.right.left = new Node(20);
        head.right.right.right = new Node(16);

        printTree(head);
        Node bst = biggestSubBST(head);
        Node node = biggestSubBST1(head);
        printTree(bst);
        System.out.println("=====================================");
        printTree(node);
    }
}
```



## 八、已知一棵完全二叉树，求其节点的个数

要求：时间复杂度低于 O(N)，N 为这棵树的节点个数

### （一）获取完全二叉树的高度

**遍历到左子树的最左边结点就得到了树的高度**

### （二）满二叉树结点数目

如果我们遍历二叉树的每个结点来计算结点个数，那么时间复杂度将是 $O(N^2)$，我们可以利用满二叉树的结点个数为 $2^{h} -1$（h 为树的层数）来加速这个过程。

### （三）计算过程

- 首先对于完全二叉树，如果其右子树的最左结点在树的最后一层，那么其左子树肯定是满二叉树，且高度为 h - 1（注意：是左子树的高度为 h - 1）；
- 否则其右子树肯定是满二叉树，且高度为 h - 2（如果右子树的高度为 `h - 1`，则说明左子树是满二叉树，如果右子树高度为 `h - 2` 则说明右子树是满二叉树）。也就是说，对于一个完全二叉树结点个数的求解，我们可以分解求解过程：1个根结点+ 一棵满二叉树（高度为 h - 1 或者 h - 2）+ 一棵完全二叉树（高度为 h - 1）。前两者的结点数是可求的（$1 + 2^{level} - 1$ = $2^{level}$），后者就又成了求一棵完全二叉树结点数的问题了，可以使用递归。

![image-20200219120345996](AlgorithmEasyDay04.resource/image-20200219120345996.png)

**结论：** 对于满二叉树，如果树的高度为 L，则其节点数为 ${2}^{L} -1$，这里采用的方法如果一共有 N 个结点，每一层都只会遍历一个，不是左孩子就是右孩子，则一共 $O(log_{2}^{N})$，到了某个结点之后，一直往下遍历的时间复杂度也是 $O(log_{2}^{N})$，最终结果就是两者乘积；

```java
package com.gjxaiou.easy.day04;

/**
 * 计算完全二叉树的结点个数
 *
 * @author GJXAIOU
 */
public class CompleteTreeNodeNumber {

    public static class Node {
        public int value;
        public Node left;
        public Node right;

        public Node(int data) {
            value = data;
        }
    }

    public static int nodeNum(Node head) {
        if (head == null) {
            return 0;
        }
        return bs(head, 1, mostLeftLevel(head, 1));
    }

    /**
     * @param node：表示当前节点
     * @param level：当前节点在第几层
     * @param treeHeight：整棵树的高度，为定值，左子树的最左边结点所在层即为树的高度
     * @return ：以这个结点为头的子树一共有多少个节点
     */
    public static int bs(Node node, int level, int treeHeight) {
        // 等于树的高度相当于最后一层，以该结点为头结点的树只有自身，所以结点数为 1
        if (level == treeHeight) {
            return 1;
        }
        // 如果 node 的右子树上的左边界层数到 treeHeight 层，即左子树是满二叉树，右子树为完全二叉树
        if (mostLeftLevel(node.right, level + 1) == treeHeight) {
            // 1 << (treeHeight - level)表示当前节点的左子树和当前节点的节点个数和，2^(treeHeight - level)个
            // 因为右孩子也是一个完全二叉树，使用递归求其总节点，就是后面部分；
            return (1 << (treeHeight - level)) + bs(node.right, level + 1, treeHeight);
            // 左子树是完全二叉树，右子树为满二叉树
        } else {
            // 没有到 treeHeight 层，则右树的高度比左树少一个，1 << (treeHeight - level - 1))
            //就是右树所有节点加上当前节点个数，然后后面是左树也是完全二叉树，递归求解；
            return (1 << (treeHeight - level - 1)) + bs(node.left, level + 1, treeHeight);
        }
    }

    // 获取以该结点为根的最左边结点所在的层数，就是当前树的高度
    public static int mostLeftLevel(Node node, int level) {
        while (node != null) {
            level++;
            node = node.left;
        }
        return level - 1;
    }

    public static void main(String[] args) {
        Node head = new Node(1);
        head.left = new Node(2);
        head.right = new Node(3);
        head.left.left = new Node(4);
        head.left.right = new Node(5);
        head.right.left = new Node(6);
        System.out.println(nodeNum(head));
    }
}

```
程序运行结果为 `6`
