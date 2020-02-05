# AlgorithmMediumDay05

[TOC]



## 一、判断一棵树是否为完全二叉树

```java
package nowcoder.advanced.advanced_class_05;

import java.util.LinkedList;
import java.util.Queue;

public class Code_01_IsBSTAndCBT {

    public static class Node {
        public int value;
        public Node left;
        public Node right;

        public Node(int data) {
            this.value = data;
        }
    }

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
            if (cur2 != null) {
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
            if (pre != null && pre.value > cur1.value) {
                res = false;
            }
            pre = cur1;
            cur1 = cur1.right;
        }
        return res;
    }

    /**
     * 判断是否为完全二叉树，这里定义空树为完全二叉树
     *
     * @param head
     * @return
     */
    public static boolean isCBT(Node head) {
        if (head == null) {
            return true;
        }
        Queue<Node> queue = new LinkedList<Node>();
        // 当一个节点左右两个孩子补全的时候，开启判断下面所有结点是否都为叶节点的过程
        boolean leaf = false;
        Node left = null;
        Node right = null;
        queue.offer(head);
        while (!queue.isEmpty()) {
            head = queue.poll();
            left = head.left;
            right = head.right;
            // 当发现某个节点的左右孩子补全的时候，leaf 过程开启，发现接下来的节点左孩子或者右孩子存在，则不是叶节点，返回 false
            // || 后面表示如果当前结点有右孩子没有左孩子直接返回 false（情况一）
            if ((leaf && (left != null || right != null)) || (left == null && right != null)) {
                return false;
            }
            if (left != null) {
                queue.offer(left);
            }
            if (right != null) {
                queue.offer(right);
                // else 等价于 if(left ==  null || right == null)
            } else {
                leaf = true;
            }
        }
        return true;
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

## 二、舞会最大活跃度

一个公司的上下级关系是一棵多叉树，这个公司要举办晚会，你作为组织者已经摸清了大家的心理：**一个员工的直** **接上级如果到场，这个员工肯定不会来**。每个员工都有一个活跃度的值（值越大，晚会上越活跃），**你可以给某个员工发邀请函以决定谁来**，怎么让舞会的气氛最活跃？返回最大的活跃值。

- 举例1：



![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e8cc6064d0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



如果邀请A来，那么其直接下属BCD一定不会来，你可以邀请EFGHJKL中的任意几个来，如果都邀请，那么舞会最大活跃度为`A(2)+E(9)+F(11)+G(2)+H(4)+J(7)+K(13)+L(5)`；但如果选择不邀请A来，那么你可以邀请其直接下属BCD中任意几个来，比如邀请B而不邀请CD，那么B的直接下属E一定不回来，但CD的直接下属你可以选择性邀请。

- 举例2

给定一个矩阵来表述这种关系
matrix =
{
1,6
1,5
1,4
}
这个矩阵的含义是：
matrix[0] = {1 , 6}，表示0这个员工的直接上级为1,0这个员工自己的活跃度为6
matrix[1] = {1 , 5}，表示1这个员工的直接上级为1（他自己是这个公司的最大boss）,1这个员工自己的活跃度为5
matrix[2] = {1 , 4}，表示2这个员工的直接上级为1,2这个员工自己的活跃度为4
为了让晚会活跃度最大，应该让1不来，0和2来。最后返回活跃度为10



**大前提**：如果你知道每个员工来舞会或不来舞会对舞会活跃值的影响，那么舞会最大活跃值就容易得知了。比如是否邀请A来取决于：B来或不来两种情况中选择对舞会活跃值增益最大的那个+C来或不来两种情况中选择对舞会活跃值增益最大的那个+D来或不来两种情况中选择对舞会活跃值增益最大的那个；同理，对于任意一名员工，是否邀请他来都是用此种决策。



**列出可能性**：来或不来。

![image-20200110184548796](AlgorithmMediumDay05.resource/image-20200110184548796.png)

**子过程要收集的信息**：返回子员工来对舞会活跃值的增益值和不来对舞会的增益值中的较大值。

示例代码：

```java
package nowcoder.advanced.advanced_class_05;

import java.awt.*;
import java.util.ArrayList;
import java.util.List;

public class Code_04_MaxHappy {
    // 比较容易理解的方法
    public static class Node {
        public int huo;
        public List<Node> nexts;

        public Node(int huo) {
            this.huo = huo;
            nexts = new ArrayList<>();
        }
    }

    // 主程序
    public static int getMaxHuo(Node head) {
        ReturnData data = process(head);
        return Math.max(data.bu_lai_huo, data.lai_huo);
    }

    public static class ReturnData {
        public int lai_huo;
        public int bu_lai_huo;

        public ReturnData(int lai_huo, int bu_lai_huo) {
            this.lai_huo = lai_huo;
            this.bu_lai_huo = bu_lai_huo;
        }
    }

    // 递归函数
    public static ReturnData process(Node head) {
        // 来的时候默认就是结果包含自己的
        int lai_huo = head.huo;
        int bu_lai_huo = 0;
        for (int i = 0; i < head.nexts.size(); i++) {
            Node next = head.nexts.get(i);
            ReturnData nextData = process(next);
            lai_huo += nextData.bu_lai_huo;
            bu_lai_huo += Math.max(nextData.lai_huo, nextData.bu_lai_huo);
        }
        return new ReturnData(lai_huo, bu_lai_huo);
    }


    public static int maxHappy(int[][] matrix) {
        int[][] dp = new int[matrix.length][2];
        boolean[] visited = new boolean[matrix.length];
        int root = 0;
        for (int i = 0; i < matrix.length; i++) {
            if (i == matrix[i][0]) {
                root = i;
            }
        }
        process(matrix, dp, visited, root);
        return Math.max(dp[root][0], dp[root][1]);
    }

    public static void process(int[][] matrix, int[][] dp, boolean[] visited, int root) {
        visited[root] = true;
        dp[root][1] = matrix[root][1];
        for (int i = 0; i < matrix.length; i++) {
            if (matrix[i][0] == root && !visited[i]) {
                process(matrix, dp, visited, i);
                dp[root][1] += dp[i][0];
                dp[root][0] += Math.max(dp[i][1], dp[i][0]);
            }
        }
    }

    public static void main(String[] args) {
        int[][] matrix = {{1, 8}, {1, 9}, {1, 10}};
        System.out.println(maxHappy(matrix));
    }
}

```



## 三、判断一棵树是否为平衡二叉树

- 左树是否平衡，左树的高度
- 右树是否平衡，右树的高度
- 左树，右树都平衡，比较高度差



```java
package nowcoder.advanced.advanced_class_05;

public class Code_02_IsBalancedTree {

    public static class Node {
        public int value;
        public Node left;
        public Node right;

        public Node(int data) {
            this.value = data;
        }
    }

    public static boolean isBalance(Node head) {
        boolean[] res = new boolean[1];
        res[0] = true;
        getHeight(head, 1, res);
        return res[0];
    }

    public static class ReturnType {
        public int level;
        public boolean isB;

        public ReturnType(int l, boolean is) {
            level = l;
            isB = is;
        }
    }

    // process(head, 1)

    public static ReturnType process(Node head, int level) {
        if (head == null) {
            return new ReturnType(level, true);
        }
        ReturnType leftSubTreeInfo = process(head.left, level + 1);
        if (!leftSubTreeInfo.isB) {
            return new ReturnType(level, false);
        }
        ReturnType rightSubTreeInfo = process(head.right, level + 1);
        if (!rightSubTreeInfo.isB) {
            return new ReturnType(level, false);
        }
        if (Math.abs(rightSubTreeInfo.level - leftSubTreeInfo.level) > 1) {
            return new ReturnType(level, false);
        }
        return new ReturnType(Math.max(leftSubTreeInfo.level, rightSubTreeInfo.level), true);
    }

    public static int getHeight(Node head, int level, boolean[] res) {
        if (head == null) {
            return level;
        }
        int lH = getHeight(head.left, level + 1, res);
        if (!res[0]) {
            return level;
        }
        int rH = getHeight(head.right, level + 1, res);
        if (!res[0]) {
            return level;
        }
        if (Math.abs(lH - rH) > 1) {
            res[0] = false;
        }
        return Math.max(lH, rH);
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



## 四、数组最大异或和

给定一个数组，求子数组的最大异或和。
一个数组的异或和为，数组中所有的数异或起来的结果。

```java
package nowcoder.advanced.advanced_class_05;

public class Code_05_Max_EOR {
    // 最暴力解法：O(N^3)
    public static int getMaxE(int[] arr) {
        int max = Integer.MIN_VALUE;
        // 分别计算 0 ~ i，1 ~ i。。。i ~ i 的异或和
        for (int i = 0; i < arr.length; i++) {
            for (int start = 0; start <= i; start++) {
                int res = 0;
                // 针对上面的每一个子数组求异或和
                for (int k = start; k <= i; k++) {
                    res ^= arr[k];
                }
                max = Math.max(max, res);
            }
        }
        return max;
    }


    // 优化方法：O（N^2）
    // 异或运算满足交换律与结合律： 若 E1 ^ E2 = E3，则 E1 = E2 ^ E3，E2 = E1 ^ E3；
    public static int getMaxE2(int[] arr) {
        int max = Integer.MIN_VALUE;
        // 准备一个辅助数组，里面放置
        int[] dp = new int[arr.length];
        int eor = 0;
        for (int i = 0; i < arr.length; i++) {
            // eor 每次都异或新数，最终得到 eor 就是 0 ~ i 的异或和
            eor ^= arr[i];
            max = Math.max(max, eor);
            // 下面计算 start ~ i 的计算结果，例如 2 ~ i 的结果为 0 ~ i 异或结果再异或 0 ~ 2 位置上值
            for (int start = 1; start <= i; start++) {
                // curEor 就是 start ~ i 的异或结果
                int curEor = eor ^ dp[start - 1];
                max = Math.max(max, curEor);
            }
            dp[i] = eor;
        }
        return max;
    }

    // 再次优化：前缀树 O（N^2）
    public static class Node {
        // 因为是前缀树，所以只有通向 0 或者 1 的路
        public Node[] nexts = new Node[2];
    }

    public static class NumTrie {
        public Node head = new Node();

        public void add(int num) {
            Node cur = head;
            // 因为加入的 int 类型，依次判断每一位的值，然后建立前缀树
            for (int move = 31; move >= 0; move--) {
                // 获取的是 int 类型符号位数，并且和 1 相与，如果符号位上为 0 结果为 0，反之如果为 1 则结果为 1；
                int path = ((num >> move) & 1);
                // 当前结点走向 path 的路是否为空，如果没有就新建
                cur.nexts[path] = cur.nexts[path] == null ? new Node() : cur.nexts[path];
                cur = cur.nexts[path];
            }
        }

        // num 为从 0 ~ i 的异或结果
        public int maxXor(int num) {
            Node cur = head;
            int res = 0;
            for (int move = 31; move >= 0; move--) {
                // 依次从最高位开始提取每一位上数
                int path = (num >> move) & 1;
                // 第一个符号为要选路，因为符号位应该走能保证异或之后值为 0 的路；符号位为 0 则应该选择 0 这条路，返回选择 1 这条路；
                // 如果不是符号位，因为保证最大，所以要选择能保证异或结果为 1 的路，所以选择的路值和原来值相反。
                int best = move == 31 ? path : (path ^ 1);
                // 如果有走向 best 的路则走 best 路，如果没有只能走另一条路
                best = cur.nexts[best] != null ? best : (best ^ 1);
                // 设置答案中每一位的值
                res |= (path ^ best) << move;
                cur = cur.nexts[best];
            }
            return res;
        }

    }

    public static int maxXorSubarray(int[] arr) {
        if (arr == null || arr.length == 0) {
            return 0;
        }
        int max = Integer.MIN_VALUE;
        int eor = 0;
        NumTrie numTrie = new NumTrie();
        numTrie.add(0);
        for (int i = 0; i < arr.length; i++) {
            // eor 是 0 ~ i 异或结果
            eor ^= arr[i];
            max = Math.max(max, numTrie.maxXor(eor));
            numTrie.add(eor);
        }
        return max;
    }

    // for test
    public static int comparator(int[] arr) {
        if (arr == null || arr.length == 0) {
            return 0;
        }
        int max = Integer.MIN_VALUE;
        for (int i = 0; i < arr.length; i++) {
            int eor = 0;
            for (int j = i; j < arr.length; j++) {
                eor ^= arr[j];
                max = Math.max(max, eor);
            }
        }
        return max;
    }

    // for test
    public static int[] generateRandomArray(int maxSize, int maxValue) {
        int[] arr = new int[(int) ((maxSize + 1) * Math.random())];
        for (int i = 0; i < arr.length; i++) {
            arr[i] = (int) ((maxValue + 1) * Math.random()) - (int) (maxValue * Math.random());
        }
        return arr;
    }

    // for test
    public static void printArray(int[] arr) {
        if (arr == null) {
            return;
        }
        for (int i = 0; i < arr.length; i++) {
            System.out.print(arr[i] + " ");
        }
        System.out.println();
    }

    // for test
    public static void main(String[] args) {
        int testTime = 500000;
        int maxSize = 30;
        int maxValue = 50;
        boolean succeed = true;
        for (int i = 0; i < testTime; i++) {
            int[] arr = generateRandomArray(maxSize, maxValue);
            int res = maxXorSubarray(arr);
            int comp = comparator(arr);
            if (res != comp) {
                succeed = false;
                printArray(arr);
                System.out.println(res);
                System.out.println(comp);
                break;
            }
        }
        System.out.println(succeed ? "Nice!" : "Fucking fucked!");
    }
}

```



## 六、完全二叉树结点个数

求一棵完全二叉树的节点个数，要求时间复杂度低于O(N)



```java
package nowcoder.advanced.advanced_class_05;

public class Code_06_CompleteTreeNodeNumber {

    public static class Node {
        public int value;
        public Node left;
        public Node right;

        public Node(int data) {
            this.value = data;
        }
    }

    public static int nodeNum(Node head) {
        if (head == null) {
            return 0;
        }
        return bs(head, 1, mostLeftLevel(head, 1));
    }

    public static int bs(Node node, int l, int h) {
        if (l == h) {
            return 1;
        }
        if (mostLeftLevel(node.right, l + 1) == h) {
            return (1 << (h - l)) + bs(node.right, l + 1, h);
        } else {
            return (1 << (h - l - 1)) + bs(node.left, l + 1, h);
        }
    }

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

