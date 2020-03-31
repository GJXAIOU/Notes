# AlgorithmMediumDay03

[TOC]



## 一、Morris遍历

介绍一种时间复杂度O(N)，额外空间复杂度O(1)的二叉树的遍历方式，N为二叉树的节点个数（不要求是完全二叉树）

利用 Morris 遍历实现二叉树的先序、中序、后序遍历，时间复杂度为 O（N），额外空间复杂度为 O（1）。

**经典二叉树，由于没有指向父节点的指针，故遍历时都需要一个栈**（递归：系统递归函数帮助压栈，非递归：自己压）来保存有关父节点的信息，都会造成O(H)的额外空间复杂度，H为二叉树高度。

```java
// 普通递归版
 public static void process(Node head) {
        if (head == null) {
            return;
        }
        // 1，打印放在这里为先序遍历
        //System.out.println(head.value);
        process(head.left);
        // 2，打印放在这里为中序遍历
        //System.out.println(head.value);
        process(head.right);	
        // 3,打印放在这里为后序遍历(第三次回到自己节点时候打印)
        //System.out.println(head.value);
 }
```

Morris 遍历是利用了二叉树中空闲的结点，例如末子节点的两个引用都是指向的位置为空，但是引用仍然占着空间。利用空闲的空间完成回到上级的操作，即修改原来二叉树结构来实现。



普通遍历递归，可以三次来到当前节点，按打印时机可分为前中后序。

morris遍历：如果有左子树，可以来到当前节点两次，若没有左子树，来到当前节点一次，可以利用左子树最右节点右指针指向谁来标记第一次还是第二次到这个节点（左子树最右指针指向null，第一次到，指向当前节点，即指向自己，第二次到）。但是在遍历右子树时，无法第三次回到自己。

morris遍历第一次到时打印，**先序**。第二次到时打印（没有左子树的一次可以理解为直到第一次与第二次重叠在一起），**中序**。

morris**后序**，**只关注能来到两次的节点**，在第二次来到这个节点的时候，逆序打印其左子树的右边界，然后整个函数退出之前单独打印整个树的右边界。

![image-20200103191511519](AlgorithmMediumDay03.resource/image-20200103191511519.png)

逆序打印：利用链表的reverse修改指针指向，打印后，改回指针。

示例拓扑：

![image-20200103192107163](AlgorithmMediumDay03.resource/image-20200103192107163.png)

**所有先序、中序、后序是经典遍历基础是选择打印时机不同的产生的结果**

morris遍历的空间复杂度：O（1）

- 来到的当前节点记为cur，如果cur无左孩子，cur向右移动（cur = cur.right）

- 如果cur有左孩子，找到 cur 左子树上最右的节点，记为 mostright
    - 若 mostright 的 right 指针指向空，让其指向 cur，然后 cur 向左移动（cur = cur.left）
    - 若 mostright 的 right 指针指向 cur，让其指向空，cur 向右移动

![image-20200103185903277](AlgorithmMediumDay03.resource/image-20200103185903277.png)

```java
package com.gjxaiou.advanced.day03;

/**
 * @Author GJXAIOU
 * @Date 2020/1/3 18:49
 */
public class MorrisTraversal {

    public static class Node {
        public int value;
        Node left;
        Node right;

        public Node(int data) {
            this.value = data;
        }
    }

    // Morris 遍历改为先序遍历
    public static void morrisPre(Node head) {
        if (head == null) {
            return;
        }
        Node cur = head;
        Node mostRight = null;
        while (cur != null) {
            mostRight = cur.left;
            // 如果当前结点的左孩子不为空，找到该结点左子树的最右节点
            if (mostRight != null) {
                while (mostRight.right != null && mostRight.right != cur) {
                    mostRight = mostRight.right;
                }
                // 如果最右结点的 right 指向 null，让其指向 cur，然后 cur 向左移动
                if (mostRight.right == null) {
                    mostRight.right = cur;
                    System.out.print(cur.value + " ");
                    cur = cur.left;
                    continue;
                    // 最右结点的 right 指向 cur，则改为指向 null
                } else {
                    mostRight.right = null;
                }
                // 此 else 表示当前结点没有左子树的时候，可以认为其第一次到达和第二次到达是重在一起的，没有左孩子，当前结点向右移动
            } else {
                System.out.print(cur.value + " ");
            }
            cur = cur.right;
        }
        System.out.println();
    }

    // Morris 中序遍历
    public static void morrisIn(Node head) {
        if (head == null) {
            return;
        }
        // 开始 cur 指向 head
        Node cur = head;
        Node mostRight = null;
        while (cur != null) {
            mostRight = cur.left;
            // 节点 cur 左孩子不为空则走这里
            if (mostRight != null) {
                // 不断找左子树最右的节点
                while (mostRight.right != null && mostRight.right != cur) {
                    mostRight = mostRight.right;
                }
                // 情况二的第一种可能性
                if (mostRight.right == null) {
                    mostRight.right = cur;
                    cur = cur.left;
                    continue;
                    // 情况二的第二种可能性
                } else {
                    mostRight.right = null;
                }
            }
            // 在节点往右跑之前打印
            System.out.print(cur.value + " ");
            // 当没有左孩子的时候
            cur = cur.right;
        }
        System.out.println();
    }


    // Morris 实现后续遍历
    public static void morrisPos(Node head) {
        if (head == null) {
            return;
        }
        Node cur = head;
        Node mostRight = null;
        while (cur != null) {
            mostRight = cur.left;
            // if 表示一个节点可以回到自己两次
            if (mostRight != null) {
                while (mostRight.right != null && mostRight.right != cur) {
                    mostRight = mostRight.right;
                }
                if (mostRight.right == null) {
                    mostRight.right = cur;
                    cur = cur.left;
                    continue;
                    // 发现是回到自己两次
                } else {
                    mostRight.right = null;
                    // 打印这个结点的整棵树右边界逆序打印
                    printEdge(cur.left);
                }
            }
            cur = cur.right;
        }
        // 函数结束之前打印整个树右边界
        printEdge(head);
        System.out.println();
    }

    public static void printEdge(Node head) {
        Node tail = reverseEdge(head);
        Node cur = tail;
        while (cur != null) {
            System.out.print(cur.value + " ");
            cur = cur.right;
        }
        reverseEdge(tail);
    }

    public static Node reverseEdge(Node from) {
        Node pre = null;
        Node next = null;
        while (from != null) {
            next = from.right;
            from.right = pre;
            pre = from;
            from = next;
        }
        return pre;
    }
}
```





## 二、搜索二叉树

搜索二叉树的定义：对于一棵二叉树中的任意子树，其左子树上的所有数值小于头结点的数值，其右子树上所有的数值大于头结点的数值，并且树中不存在数值相同的结点。也称二叉查找树。

**详细解释见**： AlgorithmEasyDay03.md



## 三、平衡二叉树/AVL树

### （一）平衡性

经典的平衡搜索二叉树结构：在满足**搜索二叉树的前提条件**下，对于一棵二叉树中的任意子树，其左子树和其右子树的高度相差不超过1。

![image-20200103211949929](AlgorithmMediumDay03.resource/image-20200103211949929.png)

因为默认情况下形成的搜索二叉树结构和数值的输入顺序有关，如果想要尽量保持平衡性则需要对树的结构进行调整。

### （二）典型搜索二叉树—AVL树、红黑树、SBT树的原理，增删改查都是 O（N）

#### 1.AVL树

AVL树是一种具有严苛平衡性的搜索二叉树。什么叫做严苛平衡性呢？那就是**所有子树的左子树和右子树的高度相差不超过1**。弊端是，每次发现因为插入、删除操作破坏了这种严苛的平衡性之后，都需要作出相应的调整以使其恢复平衡，调整较为频繁。

#### 2.红黑树

红黑树是每个节点都带有颜色属性的搜索二叉树，颜色或红色或黑色。在搜索二叉树强制一般要求以外，对于任何有效的红黑树我们增加了如下的额外要求:

- 性质1. 节点是红色或黑色。
- 性质2. 根节点是黑色。
- 性质3 每个叶节点（NIL节点，空节点）是黑色的。
- 性质4 每个红色节点的两个子节点都是黑色。(从每个叶子到根的所有路径上不能有两个连续的红色节点)
- 性质5. 从任一节点到其每个叶子的所有路径都包含相同数目的黑色节点。

这些约束强制了红黑树的关键性质: **从根到叶子的最长的可能路径不多于最短的可能路径的两倍长**。结果是这个树**大致上是平衡**的。因为操作比如插入、删除和查找某个值的最坏情况时间都要求与树的高度成比例，这个在高度上的理论上限允许红黑树在最坏情况下都是高效的，而不同于普通的二叉查找树。

要知道为什么这些特性确保了这个结果，注意到**性质4导致了路径不能有两个毗连的红色节点**就足够了。**最短的可能路径都是黑色节点，最长的可能路径有交替的红色和黑色节点**。因为根据性质5所有最长的路径都有相同数目的黑色节点，这就表明了没有路径能多于任何其他路径的两倍长。

**红黑树结构**

```java
package com.gjxaiou.advanced.day03;

/**
 * Not implemented by zuochengyun
 * 
 * Red-Black tree implementation. From Introduction to Algorithms 3rd edition.
 * 
 * @author Ignas Lelys
 * @created May 6, 2011
 * 
 */
public class RedBlackTree extends AbstractSelfBalancingBinarySearchTree {

    protected enum ColorEnum {
        RED,
        BLACK
    };

    protected static final RedBlackNode nilNode = new RedBlackNode(null, null, null, null, ColorEnum.BLACK);

    /**
     * @see trees.AbstractBinarySearchTree#insert(int)
     */
    @Override
    public Node insert(int element) {
        Node newNode = super.insert(element);
        newNode.left = nilNode;
        newNode.right = nilNode;
        root.parent = nilNode;
        insertRBFixup((RedBlackNode) newNode);
        return newNode;
    }
    
    /**
     * Slightly modified delete routine for red-black tree.
     * 
     * {@inheritDoc}
     */
    @Override
    protected Node delete(Node deleteNode) {
        Node replaceNode = null; // track node that replaces removedOrMovedNode
        if (deleteNode != null && deleteNode != nilNode) {
            Node removedOrMovedNode = deleteNode; // same as deleteNode if it has only one child, and otherwise it replaces deleteNode
            ColorEnum removedOrMovedNodeColor = ((RedBlackNode)removedOrMovedNode).color;
        
            if (deleteNode.left == nilNode) {
                replaceNode = deleteNode.right;
                rbTreeTransplant(deleteNode, deleteNode.right);
            } else if (deleteNode.right == nilNode) {
                replaceNode = deleteNode.left;
                rbTreeTransplant(deleteNode, deleteNode.left);
            } else {
                removedOrMovedNode = getMinimum(deleteNode.right);
                removedOrMovedNodeColor = ((RedBlackNode)removedOrMovedNode).color;
                replaceNode = removedOrMovedNode.right;
                if (removedOrMovedNode.parent == deleteNode) {
                    replaceNode.parent = removedOrMovedNode;
                } else {
                    rbTreeTransplant(removedOrMovedNode, removedOrMovedNode.right);
                    removedOrMovedNode.right = deleteNode.right;
                    removedOrMovedNode.right.parent = removedOrMovedNode;
                }
                rbTreeTransplant(deleteNode, removedOrMovedNode);
                removedOrMovedNode.left = deleteNode.left;
                removedOrMovedNode.left.parent = removedOrMovedNode;
                ((RedBlackNode)removedOrMovedNode).color = ((RedBlackNode)deleteNode).color;
            }
            
            size--;
            if (removedOrMovedNodeColor == ColorEnum.BLACK) {
                deleteRBFixup((RedBlackNode)replaceNode);
            }
        }
        
        return replaceNode;
    }
    
    /**
     * @see trees.AbstractBinarySearchTree#createNode(int, trees.AbstractBinarySearchTree.Node, trees.AbstractBinarySearchTree.Node, trees.AbstractBinarySearchTree.Node)
     */
    @Override
    protected Node createNode(int value, Node parent, Node left, Node right) {
        return new RedBlackNode(value, parent, left, right, ColorEnum.RED);
    }
    
    /**
     * {@inheritDoc}
     */
    @Override
    protected Node getMinimum(Node node) {
        while (node.left != nilNode) {
            node = node.left;
        }
        return node;
    }

    /**
     * {@inheritDoc}
     */
    @Override
    protected Node getMaximum(Node node) {
        while (node.right != nilNode) {
            node = node.right;
        }
        return node;
    }
    
    /**
     * {@inheritDoc}
     */
    @Override
    protected Node rotateLeft(Node node) {
        Node temp = node.right;
        temp.parent = node.parent;
        
        node.right = temp.left;
        if (node.right != nilNode) {
            node.right.parent = node;
        }

        temp.left = node;
        node.parent = temp;

        // temp took over node's place so now its parent should point to temp
        if (temp.parent != nilNode) {
            if (node == temp.parent.left) {
                temp.parent.left = temp;
            } else {
                temp.parent.right = temp;
            }
        } else {
            root = temp;
        }
        
        return temp;
    }

    /**
     * {@inheritDoc}
     */
    @Override
    protected Node rotateRight(Node node) {
        Node temp = node.left;
        temp.parent = node.parent;

        node.left = temp.right;
        if (node.left != nilNode) {
            node.left.parent = node;
        }

        temp.right = node;
        node.parent = temp;

        // temp took over node's place so now its parent should point to temp
        if (temp.parent != nilNode) {
            if (node == temp.parent.left) {
                temp.parent.left = temp;
            } else {
                temp.parent.right = temp;
            }
        } else {
            root = temp;
        }
        
        return temp;
    }

    
    /**
     * Similar to original transplant() method in BST but uses nilNode instead of null.
     */
    private Node rbTreeTransplant(Node nodeToReplace, Node newNode) {
        if (nodeToReplace.parent == nilNode) {
            this.root = newNode;
        } else if (nodeToReplace == nodeToReplace.parent.left) {
            nodeToReplace.parent.left = newNode;
        } else {
            nodeToReplace.parent.right = newNode;
        }
        newNode.parent = nodeToReplace.parent;
        return newNode;
    }
    
    /**
     * Restores Red-Black tree properties after delete if needed.
     */
    private void deleteRBFixup(RedBlackNode x) {
        while (x != root && isBlack(x)) {
            
            if (x == x.parent.left) {
                RedBlackNode w = (RedBlackNode)x.parent.right;
                if (isRed(w)) { // case 1 - sibling is red
                    w.color = ColorEnum.BLACK;
                    ((RedBlackNode)x.parent).color = ColorEnum.RED;
                    rotateLeft(x.parent);
                    w = (RedBlackNode)x.parent.right; // converted to case 2, 3 or 4
                }
                // case 2 sibling is black and both of its children are black
                if (isBlack(w.left) && isBlack(w.right)) {
                    w.color = ColorEnum.RED;
                    x = (RedBlackNode)x.parent;
                } else if (w != nilNode) {
                    if (isBlack(w.right)) { // case 3 sibling is black and its left child is red and right child is black
                        ((RedBlackNode)w.left).color = ColorEnum.BLACK;
                        w.color = ColorEnum.RED;
                        rotateRight(w);
                        w = (RedBlackNode)x.parent.right;
                    }
                    w.color = ((RedBlackNode)x.parent).color; // case 4 sibling is black and right child is red
                    ((RedBlackNode)x.parent).color = ColorEnum.BLACK;
                    ((RedBlackNode)w.right).color = ColorEnum.BLACK;
                    rotateLeft(x.parent);
                    x = (RedBlackNode)root;
                } else {
                    x.color = ColorEnum.BLACK;
                    x = (RedBlackNode)x.parent;
                }
            } else {
                RedBlackNode w = (RedBlackNode)x.parent.left;
                if (isRed(w)) { // case 1 - sibling is red
                    w.color = ColorEnum.BLACK;
                    ((RedBlackNode)x.parent).color = ColorEnum.RED;
                    rotateRight(x.parent);
                    w = (RedBlackNode)x.parent.left; // converted to case 2, 3 or 4
                }
                // case 2 sibling is black and both of its children are black
                if (isBlack(w.left) && isBlack(w.right)) {
                    w.color = ColorEnum.RED;
                    x = (RedBlackNode)x.parent;
                } else if (w != nilNode) {
                    if (isBlack(w.left)) { // case 3 sibling is black and its right child is red and left child is black
                        ((RedBlackNode)w.right).color = ColorEnum.BLACK;
                        w.color = ColorEnum.RED;
                        rotateLeft(w);
                        w = (RedBlackNode)x.parent.left;
                    }
                    w.color = ((RedBlackNode)x.parent).color; // case 4 sibling is black and left child is red
                    ((RedBlackNode)x.parent).color = ColorEnum.BLACK;
                    ((RedBlackNode)w.left).color = ColorEnum.BLACK;
                    rotateRight(x.parent);
                    x = (RedBlackNode)root;
                } else {
                    x.color = ColorEnum.BLACK;
                    x = (RedBlackNode)x.parent;
                }
            }
            
        }
    }
    
    private boolean isBlack(Node node) {
        return node != null ? ((RedBlackNode)node).color == ColorEnum.BLACK : false;
    }
    
    private boolean isRed(Node node) {
        return node != null ? ((RedBlackNode)node).color == ColorEnum.RED : false;
    }

    /**
     * Restores Red-Black tree properties after insert if needed. Insert can
     * break only 2 properties: root is red or if node is red then children must
     * be black.
     */
    private void insertRBFixup(RedBlackNode currentNode) {
        // current node is always RED, so if its parent is red it breaks
        // Red-Black property, otherwise no fixup needed and loop can terminate
        while (currentNode.parent != root && ((RedBlackNode) currentNode.parent).color == ColorEnum.RED) {
            RedBlackNode parent = (RedBlackNode) currentNode.parent;
            RedBlackNode grandParent = (RedBlackNode) parent.parent;
            if (parent == grandParent.left) {
                RedBlackNode uncle = (RedBlackNode) grandParent.right;
                // case1 - uncle and parent are both red
                // re color both of them to black
                if (((RedBlackNode) uncle).color == ColorEnum.RED) {
                    parent.color = ColorEnum.BLACK;
                    uncle.color = ColorEnum.BLACK;
                    grandParent.color = ColorEnum.RED;
                    // grandparent was recolored to red, so in next iteration we
                    // check if it does not break Red-Black property
                    currentNode = grandParent;
                } 
                // case 2/3 uncle is black - then we perform rotations
                else {
                    if (currentNode == parent.right) { // case 2, first rotate left
                        currentNode = parent;
                        rotateLeft(currentNode);
                    }
                    // do not use parent
                    parent.color = ColorEnum.BLACK; // case 3
                    grandParent.color = ColorEnum.RED;
                    rotateRight(grandParent);
                }
            } else if (parent == grandParent.right) {
                RedBlackNode uncle = (RedBlackNode) grandParent.left;
                // case1 - uncle and parent are both red
                // re color both of them to black
                if (((RedBlackNode) uncle).color == ColorEnum.RED) {
                    parent.color = ColorEnum.BLACK;
                    uncle.color = ColorEnum.BLACK;
                    grandParent.color = ColorEnum.RED;
                    // grandparent was recolored to red, so in next iteration we
                    // check if it does not break Red-Black property
                    currentNode = grandParent;
                }
                // case 2/3 uncle is black - then we perform rotations
                else {
                    if (currentNode == parent.left) { // case 2, first rotate right
                        currentNode = parent;
                        rotateRight(currentNode);
                    }
                    // do not use parent
                    parent.color = ColorEnum.BLACK; // case 3
                    grandParent.color = ColorEnum.RED;
                    rotateLeft(grandParent);
                }
            }

        }
        // ensure root is black in case it was colored red in fixup
        ((RedBlackNode) root).color = ColorEnum.BLACK;
    }

    protected static class RedBlackNode extends Node {
        public ColorEnum color;

        public RedBlackNode(Integer value, Node parent, Node left, Node right, ColorEnum color) {
            super(value, parent, left, right);
            this.color = color;
        }
    }

}

```

**Java中红黑树的使用**

Java中红黑树的实现有`TreeSet`和`TreeMap`，前者结点存储的是单一数据，而后者存储的是``的形式。

```java
public static void main(String[] args) {
    TreeMap<Integer,String> treeMap = new TreeMap();
    treeMap.put(5, "tom");
    treeMap.put(11, "jack");
    treeMap.put(30,"tony");
    treeMap.put(18, "alice");
    treeMap.put(25, "jerry");

    //红黑树中最右边的结点
    System.out.println(treeMap.lastEntry());
    System.out.println(treeMap.lastKey());
    //红黑树最左边的结点
    System.out.println(treeMap.firstKey());
    //如果有13这个key，那么返回这条记录，否则返回树中比13大的key中最小的那一个
    System.out.println(treeMap.ceilingEntry(13));
    //如果有21这个key，那么返回这条记录，否则返回树中比21小的key中最大的那一个
    System.out.println(treeMap.floorEntry(21));
    //比11大的key中，最小的那一个
    System.out.println(treeMap.higherKey(11));
    //比25小的key中，最大的那一个
    System.out.println(treeMap.lowerKey(25));
    //遍历红黑树，是按key有序遍历的
    for (Map.Entry<Integer, String> record : treeMap.entrySet()) {
        System.out.println("age:"+record.getKey()+",name:"+record.getValue());
    }
}
```

`TreeMap`的优势是`key`在其中是有序组织的，因此增加、删除、查找`key`的时间复杂度均为`log(2,N)`。

#### 3.SBT树

它是由中国广东中山纪念中学的陈启峰发明的。陈启峰于2006年底完成论文《Size Balanced Tree》，并在2007年的全国青少年信息学奥林匹克竞赛冬令营中发表。**相比红黑树、AVL树等自平衡二叉查找树，SBT更易于实现**。**据陈启峰在论文中称，SBT是“目前为止速度最快的高级二叉搜索树”**。**SBT能在O(log n)的时间内完成所有二叉搜索树(BST)的相关操作**，而与普通二叉搜索树相比，SBT仅仅加入了简洁的核心操作Maintain。由于SBT赖以保持平衡的是size域而不是其他“无用”的域，它可以很方便地实现动态顺序统计中的select和rank操作。

**SBT树的性质**是：对于数中任意结点，以该结点为根节点的子树的结点个数不能比以该结点的叔叔结点为根节点的子树的结点个数大。



![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e92dc69636?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



> 由于红黑树的实现较为复杂，因此现在工程中大多使用SBT树作为平衡二叉树的实现。



原版没有调整的搜索二叉树：不兼顾平衡性

```java
package com.gjxaiou.advanced.advanced_class_03;

/**
 * Abstract binary search tree implementation. Its basically fully implemented
 * binary search tree, just template method is provided for creating Node (other
 * trees can have slightly different nodes with more info). This way some code
 * from standart binary search tree can be reused for other kinds of binary
 * trees.
 *
 * @author Ignas Lelys
 * @created Jun 29, 2011
 */
public class AbstractBinarySearchTree {

    /**
     * Root node where whole tree starts.
     */
    public Node root;

    /**
     * Tree size.
     */
    protected int size;

    /**
     * Because this is abstract class and various trees have different
     * additional information on different nodes subclasses uses this abstract
     * method to create nodes (maybe of class {@link Node} or maybe some
     * different node sub class).
     *
     * @param value  Value that node will have.
     * @param parent Node's parent.
     * @param left   Node's left child.
     * @param right  Node's right child.
     * @return Created node instance.
     */
    protected Node createNode(int value, Node parent, Node left, Node right) {
        return new Node(value, parent, left, right);
    }

    /**
     * Finds a node with concrete value. If it is not found then null is
     * returned.
     *
     * @param element Element value.
     * @return Node with value provided, or null if not found.
     */
    public Node search(int element) {
        Node node = root;
        while (node != null && node.value != null && node.value != element) {
            if (element < node.value) {
                node = node.left;
            } else {
                node = node.right;
            }
        }
        return node;
    }

    /**
     * Insert new element to tree.
     *
     * @param element Element to insert.
     */
    public Node insert(int element) {
        if (root == null) {
            root = createNode(element, null, null, null);
            size++;
            return root;
        }

        Node insertParentNode = null;
        Node searchTempNode = root;
        // 一直找到不能再找了
        while (searchTempNode != null && searchTempNode.value != null) {
            insertParentNode = searchTempNode;
            if (element < searchTempNode.value) {
                searchTempNode = searchTempNode.left;
            } else {
                searchTempNode = searchTempNode.right;
            }
        }

        Node newNode = createNode(element, insertParentNode, null, null);
        if (insertParentNode.value > newNode.value) {
            insertParentNode.left = newNode;
        } else {
            insertParentNode.right = newNode;
        }
        size++;
        return newNode;
    }

    /**
     * Removes element if node with such value exists.
     *
     * @param element Element value to remove.
     * @return New node that is in place of deleted node. Or null if element for
     * delete was not found.
     */
    public Node delete(int element) {
        Node deleteNode = search(element);
        if (deleteNode != null) {
            return delete(deleteNode);
        } else {
            return null;
        }
    }

    /**
     * Delete logic when node is already found.
     *
     * @param deleteNode Node that needs to be deleted.
     * @return New node that is in place of deleted node. Or null if element for
     * delete was not found.
     */
    protected Node delete(Node deleteNode) {
        if (deleteNode != null) {
            Node nodeToReturn = null;
            if (deleteNode != null) {
                if (deleteNode.left == null) {
                    nodeToReturn = transplant(deleteNode, deleteNode.right);
                } else if (deleteNode.right == null) {
                    nodeToReturn = transplant(deleteNode, deleteNode.left);
                    // 如果要删除的节点的左子树和右子树都存在，见图片解析
                } else {
                    Node successorNode = getMinimum(deleteNode.right);
                    if (successorNode.parent != deleteNode) {
                        transplant(successorNode, successorNode.right);
                        successorNode.right = deleteNode.right;
                        successorNode.right.parent = successorNode;
                    }
                    transplant(deleteNode, successorNode);
                    successorNode.left = deleteNode.left;
                    successorNode.left.parent = successorNode;
                    nodeToReturn = successorNode;
                }
                size--;
            }
            return nodeToReturn;
        }
        return null;
    }

    /**
     * Put one node from tree (newNode) to the place of another (nodeToReplace).
     *
     * @param nodeToReplace Node which is replaced by newNode and removed from tree.
     * @param newNode       New node.
     * @return New replaced node.
     */
    private Node transplant(Node nodeToReplace, Node newNode) {
        if (nodeToReplace.parent == null) {
            this.root = newNode;
        } else if (nodeToReplace == nodeToReplace.parent.left) {
            nodeToReplace.parent.left = newNode;
        } else {
            nodeToReplace.parent.right = newNode;
        }
        if (newNode != null) {
            newNode.parent = nodeToReplace.parent;
        }
        return newNode;
    }

    /**
     * @param element
     * @return true if tree contains element.
     */
    public boolean contains(int element) {
        return search(element) != null;
    }

    /**
     * @return Minimum element in tree.
     */
    public int getMinimum() {
        return getMinimum(root).value;
    }

    /**
     * @return Maximum element in tree.
     */
    public int getMaximum() {
        return getMaximum(root).value;
    }

    /**
     * Get next element element who is bigger than provided element.
     *
     * @param element Element for whom descendand element is searched
     * @return Successor value.
     */
    // TODO Predecessor
    public int getSuccessor(int element) {
        return getSuccessor(search(element)).value;
    }

    /**
     * @return Number of elements in the tree.
     */
    public int getSize() {
        return size;
    }

    /**
     * Tree traversal with printing element values. In order method.
     */
    public void printTreeInOrder() {
        printTreeInOrder(root);
    }

    /**
     * Tree traversal with printing element values. Pre order method.
     */
    public void printTreePreOrder() {
        printTreePreOrder(root);
    }

    /**
     * Tree traversal with printing element values. Post order method.
     */
    public void printTreePostOrder() {
        printTreePostOrder(root);
    }

    /*-------------------PRIVATE HELPER METHODS-------------------*/

    private void printTreeInOrder(Node entry) {
        if (entry != null) {
            printTreeInOrder(entry.left);
            if (entry.value != null) {
                System.out.println(entry.value);
            }
            printTreeInOrder(entry.right);
        }
    }

    private void printTreePreOrder(Node entry) {
        if (entry != null) {
            if (entry.value != null) {
                System.out.println(entry.value);
            }
            printTreeInOrder(entry.left);
            printTreeInOrder(entry.right);
        }
    }

    private void printTreePostOrder(Node entry) {
        if (entry != null) {
            printTreeInOrder(entry.left);
            printTreeInOrder(entry.right);
            if (entry.value != null) {
                System.out.println(entry.value);
            }
        }
    }

    protected Node getMinimum(Node node) {
        while (node.left != null) {
            node = node.left;
        }
        return node;
    }

    protected Node getMaximum(Node node) {
        while (node.right != null) {
            node = node.right;
        }
        return node;
    }

    protected Node getSuccessor(Node node) {
        // if there is right branch, then successor is leftmost node of that
        // subtree
        if (node.right != null) {
            return getMinimum(node.right);
        } else { // otherwise it is a lowest ancestor whose left child is also
            // ancestor of node
            Node currentNode = node;
            Node parentNode = node.parent;
            while (parentNode != null && currentNode == parentNode.right) {
                // go up until we find parent that currentNode is not in right
                // subtree.
                currentNode = parentNode;
                parentNode = parentNode.parent;
            }
            return parentNode;
        }
    }

    // -------------------------------- TREE PRINTING
    // ------------------------------------

    public void printTree() {
        printSubtree(root);
    }

    public void printSubtree(Node node) {
        if (node.right != null) {
            printTree(node.right, true, "");
        }
        printNodeValue(node);
        if (node.left != null) {
            printTree(node.left, false, "");
        }
    }

    private void printNodeValue(Node node) {
        if (node.value == null) {
            System.out.print("<null>");
        } else {
            System.out.print(node.value.toString());
        }
        System.out.println();
    }

    private void printTree(Node node, boolean isRight, String indent) {
        if (node.right != null) {
            printTree(node.right, true, indent + (isRight ? "        " : " |      "));
        }
        System.out.print(indent);
        if (isRight) {
            System.out.print(" /");
        } else {
            System.out.print(" \\");
        }
        System.out.print("----- ");
        printNodeValue(node);
        if (node.left != null) {
            printTree(node.left, false, indent + (isRight ? " |      " : "        "));
        }
    }

    public static class Node {
        public Node(Integer value, Node parent, Node left, Node right) {
            super();
            this.value = value;
            this.parent = parent;
            this.left = left;
            this.right = right;
        }

        public Integer value;
        public Node parent;
        public Node left;
        public Node right;

        public boolean isLeaf() {
            return left == null && right == null;
        }

        @Override
        public int hashCode() {
            final int prime = 31;
            int result = 1;
            result = prime * result + ((value == null) ? 0 : value.hashCode());
            return result;
        }

        @Override
        public boolean equals(Object obj) {
            if (this == obj)
                return true;
            if (obj == null)
                return false;
            if (getClass() != obj.getClass())
                return false;
            Node other = (Node) obj;
            if (value == null) {
                if (other.value != null)
                    return false;
            } else if (!value.equals(other.value))
                return false;
            return true;
        }

    }
}

```



![image-20200103215345867](AlgorithmMediumDay03.resource/image-20200103215345867.png)

### （三）搜索二叉树调整的步骤：为了平衡性

右旋：头结点变成了新头结点的右孩子

左旋：头结点变成了新头结点的左孩子

![image-20200103223229101](AlgorithmMediumDay03.resource/image-20200103223229101.png)



#### 以 AVL 树为例

- 怎么发现不平衡的

![image-20200104113644523](AlgorithmMediumDay03.resource/image-20200104113644523.png)

- 发现之后的修改方式

    结点发现左右子树高度不平衡，修改，然后向父走，一级一级判断修改

- LL，RR，LR，RL型

    ![image-20200104114442568](AlgorithmMediumDay03.resource/image-20200104114442568.png)

#### 1.旋转过程——Rebalance

- 左旋：

![左旋](https://user-gold-cdn.xitu.io/2019/2/19/169045e933a810ae?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



- 右旋：

![右旋](https://user-gold-cdn.xitu.io/2019/2/19/169045e93488e2a3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



每种平衡二叉树都有自己的一套在插入、删除等操作改变树结构而破坏既定平衡性时的应对措施（但都是左旋操作和右旋操作的组合），以AVL数为例（有四种平衡调整操作，其中的数字只是结点代号而非结点数值）：

- `LL`调整：2号结点的左孩子的左孩子导致整个树不平衡，2号结点右旋一次

    

    ![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e937fe631e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

    

- `RR`调整：3号结点的右孩子的右孩子导致树不平衡，3号结点左旋一次：

    

    ![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e940a17348?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

    

- `LR`调整：先左后右

    

    ![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e941fb5487?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

    

- `RL`调整：先右后左：

    

    ![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e949b6084c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

    

> 红黑树的调整也是类似的，只不过调整方案更多。面试中一般不会让你手写红黑树（若有兴趣可参见文末附录），但我们一定能说清这些查找二叉树的性质，以及调整平衡的基本操作，再就是这些结构的使用。

AVL 树结构

```java
package com.gjxaiou.advanced.day03;

/**
 * Not implemented by zuochengyun
 * <p>
 * AVL tree implementation.
 * <p>
 * In computer science, an AVL tree is a self-balancing binary search tree, and
 * it was the first such data structure to be invented.[1] In an AVL tree, the
 * heights of the two child subtrees of any node differ by at most one. Lookup,
 * insertion, and deletion all take O(log n) time in both the average and worst
 * cases, where n is the number of nodes in the tree prior to the operation.
 * Insertions and deletions may require the tree to be rebalanced by one or more
 * tree rotations.
 *
 * @author Ignas Lelys
 * @created Jun 28, 2011
 */
public class AVLTree extends AbstractSelfBalancingBinarySearchTree {

    /**
     * @see trees.AbstractBinarySearchTree#insert(int)
     * <p>
     * AVL tree insert method also balances tree if needed. Additional
     * height parameter on node is used to track if one subtree is higher
     * than other by more than one, if so AVL tree rotations is performed
     * to regain balance of the tree.
     */
    @Override
    public Node insert(int element) {
        Node newNode = super.insert(element);
        rebalance((AVLNode) newNode);
        return newNode;
    }

    /**
     * @see trees.AbstractBinarySearchTree#delete(int)
     */
    @Override
    public Node delete(int element) {
        Node deleteNode = super.search(element);
        if (deleteNode != null) {
            Node successorNode = super.delete(deleteNode);
            if (successorNode != null) {
                // if replaced from getMinimum(deleteNode.right) then come back there and update 
                //heights
                AVLNode minimum = successorNode.right != null ?
                        (AVLNode) getMinimum(successorNode.right) : (AVLNode) successorNode;
                recomputeHeight(minimum);
                rebalance((AVLNode) minimum);
            } else {
                recomputeHeight((AVLNode) deleteNode.parent);
                rebalance((AVLNode) deleteNode.parent);
            }
            return successorNode;
        }
        return null;
    }

    /**
     * @see trees.AbstractBinarySearchTree#createNode(int, trees.AbstractBinarySearchTree.Node, trees.AbstractBinarySearchTree.Node, trees.AbstractBinarySearchTree.Node)
     */
    @Override
    protected Node createNode(int value, Node parent, Node left, Node right) {
        return new AVLNode(value, parent, left, right);
    }

    /**
     * Go up from inserted node, and update height and balance informations if needed.
     * If some node balance reaches 2 or -2 that means that subtree must be rebalanced.
     *
     * @param node Inserted Node.
     */
    // 结点发现左右子树高度不平衡，修改，然后向父走，一级一级判断修改
    private void rebalance(AVLNode node) {
        while (node != null) {

            Node parent = node.parent;
            // 左树和右树的高度：如果左孩子存在左树高度就是左孩子高度，不在则为 -1，右子树同理。
            int leftHeight = (node.left == null) ? -1 : ((AVLNode) node.left).height;
            int rightHeight = (node.right == null) ? -1 : ((AVLNode) node.right).height;
            int nodeBalance = rightHeight - leftHeight;
            // rebalance (-2 means left subtree outgrow, 2 means right subtree)
            if (nodeBalance == 2) {
                if (node.right.right != null) {
                    node = (AVLNode) avlRotateLeft(node);
                    break;
                } else {
                    node = (AVLNode) doubleRotateRightLeft(node);
                    break;
                }
            } else if (nodeBalance == -2) {
                if (node.left.left != null) {
                    node = (AVLNode) avlRotateRight(node);
                    break;
                } else {
                    node = (AVLNode) doubleRotateLeftRight(node);
                    break;
                }
            } else {
                updateHeight(node);
            }

            node = (AVLNode) parent;
        }
    }

    /**
     * Rotates to left side.
     */
    private Node avlRotateLeft(Node node) {
        Node temp = super.rotateLeft(node);

        updateHeight((AVLNode) temp.left);
        updateHeight((AVLNode) temp);
        return temp;
    }

    /**
     * Rotates to right side.
     */
    private Node avlRotateRight(Node node) {
        Node temp = super.rotateRight(node);

        updateHeight((AVLNode) temp.right);
        updateHeight((AVLNode) temp);
        return temp;
    }

    /**
     * Take right child and rotate it to the right side first and then rotate
     * node to the left side.
     */
    protected Node doubleRotateRightLeft(Node node) {
        node.right = avlRotateRight(node.right);
        return avlRotateLeft(node);
    }

    /**
     * Take right child and rotate it to the right side first and then rotate
     * node to the left side.
     */
    protected Node doubleRotateLeftRight(Node node) {
        node.left = avlRotateLeft(node.left);
        return avlRotateRight(node);
    }

    /**
     * Recomputes height information from the node and up for all of parents. It needs to be done
     * after delete.
     */
    private void recomputeHeight(AVLNode node) {
        while (node != null) {
            node.height = maxHeight((AVLNode) node.left, (AVLNode) node.right) + 1;
            node = (AVLNode) node.parent;
        }
    }

    /**
     * Returns higher height of 2 nodes.
     */
    private int maxHeight(AVLNode node1, AVLNode node2) {
        if (node1 != null && node2 != null) {
            return node1.height > node2.height ? node1.height : node2.height;
        } else if (node1 == null) {
            return node2 != null ? node2.height : -1;
        } else if (node2 == null) {
            return node1 != null ? node1.height : -1;
        }
        return -1;
    }

    /**
     * Updates height and balance of the node.
     *
     * @param node Node for which height and balance must be updated.
     */
    private static final void updateHeight(AVLNode node) {
        int leftHeight = (node.left == null) ? -1 : ((AVLNode) node.left).height;
        int rightHeight = (node.right == null) ? -1 : ((AVLNode) node.right).height;
        node.height = 1 + Math.max(leftHeight, rightHeight);
    }

    /**
     * Node of AVL tree has height and balance additional properties. If balance
     * equals 2 (or -2) that node needs to be re balanced. (Height is height of
     * the subtree starting with this node, and balance is difference between
     * left and right nodes heights).
     *
     * @author Ignas Lelys
     * @created Jun 30, 2011
     */
    protected static class AVLNode extends Node {
        public int height;

        public AVLNode(int value, Node parent, Node left, Node right) {
            super(value, parent, left, right);
        }
    }

}
```





## 四、跳表



![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e96cbc2217?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



跳表有着和红黑树、SBT树相同的功能，都能实现在`O(log(2,N))`内实现对数据的增删改查操作。但跳表不是以二叉树为原型的，其设计细节如下：

记该结构为`SkipList`，该结构中可以包含有很多结点（`SkipListNode`），每个结点代表一个被添加到该结构的数据项。当实例化`SkipList`时，该对象就会自带一个`SkipListNode`（不代表任何数据项的头结点）。

### （一）添加数据

当你向其中添加数据之前，首先会抛硬币，将第一次出现正面朝上时硬币被抛出的次数作为该数据的层数（`level`，**最小为1**），接着将数据和其层数封装成一个`SkipListNode`添加到`SkipList`中。结构初始化时，其头结点的层数为0，但每次添加数据后都会更新头结点的层数为所添数据中层数最大的。比如实例化一个`SkipList`后向其中添加一条层数为`3`的数据`7`：



![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e97278c2b4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



这时如果再添加一条层数为`2`的数据`5`呢？首先游标`curNode`会从`head`的最高层出发往右走，走到数据项为7的结点，发现`7>5`，于是又退回来走向下一层：



![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e9816ae3c1?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



接着再尝试往右走，还是发现`7>5`，于是还是准备走向下一层，但此时发现`curNode`所在层数`2`是数据项`5`的最高层，于是先建出数据项`5`的第二层，`curNode`再走向下一层：



![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e98dbd9a4d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



同样的，`curNode`尝试往右走，但发现`7>5`，`curNode`所在层为1，但数据`5`的第一层还没建，于是建出，`curNode`再往下走。当`curNode`走到`null`时，建出数据`5`根部的`null`：



![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e992ace6b1?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



至此层数为2的数据项`5`的添加操作完毕。

那如果添加一个层数较高的数据项该如何处理呢？以添加层数为4的数据`10`为例：



![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e9982e49b4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



添加操作对应的代码示例：

```java
import java.util.ArrayList;

/**
 * A stored structure.Its add,delete,update,find operation are log(2,N)
 *
 * @author zhenganwen
 */
public class SkipList {
    private SkipListNode head;
    private int maxLevel;
    private int size;
    public static final double PROBABILITY = 0.5;

    public SkipList() {
        this.head = new SkipListNode(Integer.MIN_VALUE);
        /**
         * the 0th level of each SkipListNode is null
         */
        this.head.nextNodes.add(null);
        this.maxLevel = 0;
        this.size = 0;
    }

    private class SkipListNode {
        int value;
        /**
         * nextNodes represent the all levels of a SkipListNode the element on
         * one index represent the successor SkipListNode on the indexth level
         */
        ArrayList<SkipListNode> nextNodes;

        public SkipListNode(int newValue) {
            this.value = newValue;
            this.nextNodes = new ArrayList<SkipListNode>();
        }
    }

    /**
     * put a new data into the structure->log(2,N)
     *
     * @param newValue
     */
    public void add(int newValue) {
        if (!contains(newValue)) {

            // generate the level
            int level = 1;
            while (Math.random() < PROBABILITY) {
                level++;
            }
            // update max level
            if (level > maxLevel) {
                int increment = level - maxLevel;
                while (increment-- > 0) {
                    this.head.nextNodes.add(null);
                }
                maxLevel = level;
            }
            // encapsulate value
            SkipListNode newNode = new SkipListNode(newValue);
            // build all the levels of new node
            SkipListNode cur = findInsertionOfTopLevel(newValue, level);
            while (level > 0) {
                if (cur.nextNodes.get(level) != null) {
                    newNode.nextNodes.add(0, cur.nextNodes.get(level));
                } else {
                    newNode.nextNodes.add(0, null);
                }
                cur.nextNodes.set(level, newNode);
                level--;
                cur = findNextInsertion(cur, newValue, level);
            }
            newNode.nextNodes.add(0, null);
            size++;
        }
    }

    /**
     * find the insertion point of the newNode's top level from head's maxLevel
     * by going right or down
     *
     * @param newValue newNode's value
     * @param level    newNode's top level
     * @return
     */
    private SkipListNode findInsertionOfTopLevel(int newValue, int level) {
        int curLevel = this.maxLevel;
        SkipListNode cur = head;
        while (curLevel >= level) {
            if (cur.nextNodes.get(curLevel) != null
                    && cur.nextNodes.get(curLevel).value < newValue) {
                // go right
                cur = cur.nextNodes.get(curLevel);
            } else {
                // go down
                curLevel--;
            }
        }
        return cur;
    }

    /**
     * find the next insertion from cur node by going right on the level
     *
     * @param cur
     * @param newValue
     * @param level
     * @return
     */
    private SkipListNode findNextInsertion(SkipListNode cur, int newValue,
                                           int level) {
        while (cur.nextNodes.get(level) != null
                && cur.nextNodes.get(level).value < newValue) {
            cur = cur.nextNodes.get(level);
        }
        return cur;
    }

    /**
     * check whether a value exists->log(2,N)
     *
     * @param value
     * @return
     */
    public boolean contains(int value) {
        if (this.size == 0) {
            return false;
        }
        SkipListNode cur = head;
        int curLevel = maxLevel;
        while (curLevel > 0) {
            if (cur.nextNodes.get(curLevel) != null) {
                if (cur.nextNodes.get(curLevel).value == value) {
                    return true;
                } else if (cur.nextNodes.get(curLevel).value < value) {
                    cur = cur.nextNodes.get(curLevel);
                } else {
                    curLevel--;
                }
            } else {
                curLevel--;
            }
        }

        return false;
    }

    public static void main(String[] args) {
        SkipList skipList = new SkipList();
        skipList.add(1);
        skipList.add(2);
        skipList.add(3);
        skipList.add(4);
        skipList.add(5);
        //mark a break point here to check the memory structure of skipList
        System.out.println(skipList);
    }

}

```

### （二）查找数据

查找数据项的操作和添加数据项的步骤类似，也是游标`curNode`从`head`的最高层出发，每次先尝试向右走来到`nextNode`，如果`nextNode`封装的数据大于查找的目标`target`或`nextNode`为空，那么`curNode`回退并向下走；如果`nextNode`封装的数据小于`target`，那么`curNode`继续向右走，直到`curNode`走到的结点数据与`target`相同表示找到了，否则`curNode`走到了某一结点的根部`null`，那么说明结构中不存在该数据。`->contains()`

### （三）删除数据

了解添加数据的过程之后，删除数据其实就是将逻辑倒过来：解除该数据结点的前后引用关系。下图是我在写好上述`add()`方法后，向其中放入`1、2、3、4、5`后形成的结构：



![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e9a1bc219d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



如果此时删除数据`3`：



![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e9a1e24f68?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



首先应该从`head`的最高层出发，通过向右或向下找到数据3的最高层（如图`2->3->5->6->7`），将该层移除整体结构并处理好该层上，其前后结点的关系。同样的逻辑，将数据`3`剩下的层移除。

示例代码：

```java
/**
     * delete skipListNode by the value
     *
     * @param value
     */
public void delete(int value) {
    //if exists
    if (contains(value)) {
        //find the node and its level
        SkipListNode deletedNode = head;
        int deletedLevels = maxLevel;
        //because exists,so must can find
        while (deletedLevels > 0) {
            if (deletedNode.nextNodes.get(deletedLevels) != null) {
                if (deletedNode.nextNodes.get(deletedLevels).value == value) {
                    deletedNode = deletedNode.nextNodes.get(deletedLevels);
                    break;
                } else if (deletedNode.nextNodes.get(deletedLevels).value < value) {
                    deletedNode = deletedNode.nextNodes.get(deletedLevels);
                } else {
                    deletedLevels--;
                }
            } else {
                deletedLevels--;
            }
        }
        //release the node and adjust the reference
        while (deletedLevels > 0) {
            SkipListNode pre = findInsertionOfTopLevel(value, deletedLevels);
            if (deletedNode.nextNodes.get(deletedLevels) != null) {
                pre.nextNodes.set(deletedLevels, deletedNode.nextNodes.get(deletedLevels));
            } else {
                pre.nextNodes.set(deletedLevels, null);
            }
            deletedLevels--;
        }

        size--;
    }
}

public static void main(String[] args) {
    SkipList skipList = new SkipList();
    skipList.add(1);
    skipList.add(2);
    skipList.add(3);
    skipList.add(4);
    skipList.add(5);
    //mark a break point here to check the memory structure of skipList
    skipList.delete(3);
    System.out.println(skipList);
}

```

### （四）遍历数据

需要遍历跳表中的数据时，我们可以根据每个数据的层数至少为1的特点（每个结点的第一层引用的是比该结点数据大的结点中数据最小的结点）。

示例代码：

```java
class SkipListIterator implements Iterator<Integer> {
    private SkipListNode cur;
    public SkipListIterator(SkipList skipList) {
        this.cur = skipList.head;
    }

    @Override
    public boolean hasNext() {
        return cur.nextNodes.get(1) != null;
    }

    @Override
    public Integer next() {
        int value = cur.nextNodes.get(1).value;
        cur = cur.nextNodes.get(1);
        return value;
    }
}

@Override
public String toString() {
    SkipListIterator iterator = new SkipListIterator(this);
    String res = "[ ";
    while (iterator.hasNext()) {
        res += iterator.next()+" ";
    }
    res += "]";
    System.out.println();
    return res;
}

public static void main(String[] args) {
    SkipList skipList = new SkipList();
    skipList.add(1);
    skipList.add(2);
    skipList.add(3);
    skipList.add(4);
    skipList.add(5);
    System.out.println(skipList);
    skipList.delete(3);
    System.out.println(skipList);
}
```



完整的代码程序

```java
package com.gjxaiou.advanced.day03;

import java.util.ArrayList;
import java.util.Iterator;

/**
 * 跳表
 */
public class Code_02_SkipList {

    public static class SkipListNode {
        public Integer value;
        // 该调表 ArrayList 的长度表示有多少层，nextNodes[0] 表示第一层上 SkipListNode 节点的下一层是什么
        public ArrayList<SkipListNode> nextNodes;

        public SkipListNode(Integer value) {
            this.value = value;
            nextNodes = new ArrayList<SkipListNode>();
        }
    }

    public static class SkipListIterator implements Iterator<Integer> {
        SkipList list;
        SkipListNode current;

        public SkipListIterator(SkipList list) {
            this.list = list;
            this.current = list.getHead();
        }

        @Override
        public boolean hasNext() {
            return current.nextNodes.get(0) != null;
        }

        @Override
        public Integer next() {
            current = current.nextNodes.get(0);
            return current.value;
        }
    }

    public static class SkipList {
        // head 就是巨小，根据最大的层数决定
        private SkipListNode head;
        private int maxLevel;
        private int size;
        // 以什么概率出 0，则 1- 概率出 1
        private static final double PROBABILITY = 0.5;

        public SkipList() {
            size = 0;
            maxLevel = 0;
            head = new SkipListNode(null);
            head.nextNodes.add(null);
        }

        public SkipListNode getHead() {
            return head;
        }

        public void add(Integer newValue) {
            if (!contains(newValue)) {
                size++;
                int level = 0;
                while (Math.random() < PROBABILITY) {
                    level++;
                }
                // 如果当前 level 大于之前最大的 maxLevel，则头部一定要增加
                while (level > maxLevel) {
                    head.nextNodes.add(null);
                    maxLevel++;
                }
                // 当新加入元素总是从头部 head 的最高层往下移动的
                SkipListNode newNode = new SkipListNode(newValue);
                SkipListNode current = head;
                // 同一层，如果下一个位置数比当前位置数小，则往右移动；如果大了就往下移动
                // 每次都是从最高层开始往下找
                int levelAll = maxLevel;
                do {
                    current = findNext(newValue, current, levelAll);
                    if (level >= levelAll) {
                        newNode.nextNodes.add(0, current.nextNodes.get(level));
                        current.nextNodes.set(level, newNode);
                        level--;
                    }
                } while (levelAll-- > 0);
            }
        }

        public void delete(Integer deleteValue) {
            if (contains(deleteValue)) {
                SkipListNode deleteNode = find(deleteValue);
                size--;
                int level = maxLevel;
                SkipListNode current = head;
                do {
                    current = findNext(deleteNode.value, current, level);
                    if (deleteNode.nextNodes.size() > level) {
                        current.nextNodes.set(level, deleteNode.nextNodes.get(level));
                    }
                } while (level-- > 0);
            }
        }

        // Returns the skiplist node with greatest value <= e
        private SkipListNode find(Integer e) {
            return find(e, head, maxLevel);
        }

        // Returns the skiplist node with greatest value <= e
        // Starts at node start and level
        private SkipListNode find(Integer e, SkipListNode current, int level) {
            do {
                current = findNext(e, current, level);
            } while (level-- > 0);
            return current;
        }

        // Returns the node at a given level with highest value less than e
        private SkipListNode findNext(Integer e, SkipListNode current, int level) {
            SkipListNode next = current.nextNodes.get(level);
            while (next != null) {
                // 拿出 next 值
                Integer value = next.value;
                // 如果当前值小于拿出来的值，找到了
                if (lessThan(e, value)) {
                    break;
                }
                // 如果不小于，则往右走，所以 current 就是在该 level 层中最后一个小于当前数的
                current = next;
                next = current.nextNodes.get(level);
            }
            return current;
        }

        public int size() {
            return size;
        }

        public boolean contains(Integer value) {
            SkipListNode node = find(value);
            return node != null && node.value != null && equalTo(node.value, value);
        }

        public Iterator<Integer> iterator() {
            return new SkipListIterator(this);
        }

        /******************************************************************************
         * Utility Functions *
         ******************************************************************************/

        private boolean lessThan(Integer a, Integer b) {
            return a.compareTo(b) < 0;
        }

        private boolean equalTo(Integer a, Integer b) {
            return a.compareTo(b) == 0;
        }

    }

    public static void main(String[] args) {

    }
}

```





