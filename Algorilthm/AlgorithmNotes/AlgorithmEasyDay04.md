# AlgorithmEasyDay04


## 二叉树

### 实现二叉树的先序、中序、后序遍历，包括递归方式和非递归方式


**使用递归的方式：**

二叉树的示例结构为：
![二叉树结构]($resource/%E4%BA%8C%E5%8F%89%E6%A0%91%E7%BB%93%E6%9E%84.png)

实际函数访问结点的顺序为：1 2 4 4 4 2 5 5 5 2 1 3 6 6 6 3 7 7 7 3 1 结束
**重点：**
- 把打印时机放在第一次访问该结点的时候就是先序遍历；放在第二次访问该结点的时候就是中序遍历，放在第三次访问该结点的时候就是后续遍历； 
所有先序遍历是：1 2 4 5 3 6 7；
中序遍历：4 2 5 1 6 3 1；
后序遍历：4 5 2 6 7 3 1；


**非递归方式：**
因为递归使用的是栈，这里只需要自己实现一个栈结构即可；
先序遍历：首先在栈中放入头结点，然后弹出头结点进行打印，如果该节点的右结点不为空，压入右结点，如果左结点不为空压入左结点，然后弹出栈顶，再次循环；
示例为例：先压入 1 ，然后弹出 1，先压入节点 1 的右孩子 3，然后压入左孩子 2，然后弹出栈顶 2，然后压入节点 2 的右孩子和左孩子，现在栈中顺序为：4 5 3，然后弹出 4，压入 4 的右孩子和左孩子，发现都没有，弹出栈顶 5，压入 5 的右孩子和左孩子，发现都没有，弹出栈顶 3，然后压入 3 的右孩子和左孩子，然后弹出栈顶 6，然后压入 6 的右、左孩子，都没有，弹出 7，压入右孩子、左孩子，没有，弹出发现为空，结束；


中序遍历：当前结点不为空的时候，将当前结点压入栈中，结点指针左移，指向左结点，直到当前结点为空，则从栈中将栈顶弹出打印，然后指针右移；
示例为例：首先头结点 1 不为空，将该结点压入，然后指向节点 2，然后压入结点 2，然后压入结点 4，然后指向 null，以为栈不等于空，还得遍历，进入 else，弹出栈顶为 结点 4 ，指针指向 4，然后指向结点 4 的右结点为 null，然后再次弹出节点 2，然后指向结点 2 的右子节点 5，然后。。。。


后序遍历：是左右中打印（需要用到两个栈，代码方式二是用的一个栈，不太好理解）；先序遍历是中、左、右，就是当前结点先压右孩子然后压左孩子，那么中右左就是当前结果先压左孩子然后压右孩子，然后将原来使用打印的语句更改为：将该元素存放到另一个栈中，但是不打印，全部遍历完成之后，将栈中的元素全部打印出来即可；
```java
package nowcoder.easy.day04;

import java.util.Stack;

public class PreInPosTraversal {

	public static class Node {
		public int value;
		public Node left;
		public Node right;

		public Node(int data) {
			this.value = data;
		}
	}
	// 前序遍历
	public static void preOrderRecur(Node head) {
		if (head == null) {
			return;
		}
		System.out.print(head.value + " ");
		preOrderRecur(head.left);
		preOrderRecur(head.right);
	}

	public static void inOrderRecur(Node head) {
		if (head == null) {
			return;
		}
		inOrderRecur(head.left);
		System.out.print(head.value + " ");
		inOrderRecur(head.right);
	}

	public static void posOrderRecur(Node head) {
		if (head == null) {
			return;
		}
		posOrderRecur(head.left);
		posOrderRecur(head.right);
		System.out.print(head.value + " ");
	}


	/**
	 * 非递归版
	 * @param head
	 */
	public static void preOrderUnRecur(Node head) {
		System.out.print("pre-order: ");
		if (head != null) {
			// 准备一个栈
			Stack<Node> stack = new Stack<Node>();
			// 放入头结点
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

	public static void inOrderUnRecur(Node head) {
		System.out.print("in-order: ");
		if (head != null) {
			Stack<Node> stack = new Stack<Node>();
			while (!stack.isEmpty() || head != null) {
				if (head != null) {
					stack.push(head);
					head = head.left;
				} else {
					head = stack.pop();
					System.out.print(head.value + " ");
					head = head.right;
				}
			}
		}
		System.out.println();
	}

	public static void posOrderUnRecur1(Node head) {
		System.out.print("pos-order: ");
		if (head != null) {
			Stack<Node> s1 = new Stack<Node>();
			Stack<Node> s2 = new Stack<Node>();
			s1.push(head);
			while (!s1.isEmpty()) {
				head = s1.pop();
				s2.push(head);
				if (head.left != null) {
					s1.push(head.left);
				}
				if (head.right != null) {
					s1.push(head.right);
				}
			}
			while (!s2.isEmpty()) {
				System.out.print(s2.pop().value + " ");
			}
		}
		System.out.println();
	}

	// 另一种实现后续，使用一个栈
	public static void posOrderUnRecur2(Node h) {
		System.out.print("pos-order: ");
		if (h != null) {
			Stack<Node> stack = new Stack<Node>();
			stack.push(h);
			Node c = null;
			while (!stack.isEmpty()) {
				c = stack.peek();
				if (c.left != null && h != c.left && h != c.right) {
					stack.push(c.left);
				} else if (c.right != null && h != c.right) {
					stack.push(c.right);
				} else {
					System.out.print(stack.pop().value + " ");
					h = c;
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

		// recursive
		System.out.println("==============recursive==============");
		System.out.print("pre-order: ");
		preOrderRecur(head);
		System.out.println();
		System.out.print("in-order: ");
		inOrderRecur(head);
		System.out.println();
		System.out.print("pos-order: ");
		posOrderRecur(head);
		System.out.println();

		// unrecursive
		System.out.println("============unrecursive=============");
		preOrderUnRecur(head);
		inOrderUnRecur(head);
		posOrderUnRecur1(head);
		posOrderUnRecur2(head);

	}

}

```

### 如何直观的打印一颗二叉树
```java
package nowcoder.easy.day04;

public class PrintBinaryTree {

	public static class Node {
		public int value;
		public Node left;
		public Node right;

		public Node(int data) {
			this.value = data;
		}
	}

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
		Node head = new Node(1);
		head.left = new Node(-222222222);
		head.right = new Node(3);
		head.left.left = new Node(Integer.MIN_VALUE);
		head.right.left = new Node(55555555);
		head.right.right = new Node(66);
		head.left.left.right = new Node(777);
		printTree(head);

		head = new Node(1);
		head.left = new Node(2);
		head.right = new Node(3);
		head.left.left = new Node(4);
		head.right.left = new Node(5);
		head.right.right = new Node(6);
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

注释：打印结果中， `HXH`表示头结点 X，`VYV`表示 Y 是左下方最近结点的孩子；`^Z^` 表示 Z 是左上方最近结点的孩子；


### 在二叉树中找到一个节点的后继节点
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
该结构比普通二叉树节点结构多了一个指向父节点的parent指针。假设有一 棵Node类型的节点组成的二叉树，树中每个节点的parent指针都正确地指向 自己的父节点，头节点的parent指向null。只给一个在二叉树中的某个节点 node，请实现返回node的后继节点的函数。**在二叉树的中序遍历的序列中， node的下一个节点叫作node的后继节点，node 的前一个结点叫做 node 的前驱结点**。

还是以上面的树形结构为例，中序遍历结果为：4 2 5 1 6 3 7，则 1 的后继结点为 6，前驱结点为 5；

**解答：**
一般方法：可以通过给定结点的 parent 指针一直找到根结点，然后从根节点中序遍历整个树，最后根据中序遍历的结果得到每个结点的后继结点；
时间复杂度为：O(N)


**进阶：** 
例如 5 的后继结点为 1，能不能通过 5 2 1 这样的结构找到，即当前结点距离后继结点的距离长度就是所要的复杂度；
方案：如果一个结点有右子树，则后继结点就是右子树上左边的结点；例：1 的后继为 6；
如果没有右子树，则使用 parent 指针一直向上找（指向当前结点的指针和 parent 指针同时向上移动），一直到指向的结点是该结点父结点的左孩子就停止，则该父结点就是所求结点的后继结点；
示例图：
![题目三寻找后继结点示例]($resource/%E9%A2%98%E7%9B%AE%E4%B8%89%E5%AF%BB%E6%89%BE%E5%90%8E%E7%BB%A7%E7%BB%93%E7%82%B9%E7%A4%BA%E4%BE%8B.png)
以结点 11 为例，其父结点为 5,11 不是 5 的左结点，向上，5 的父结点为 2,5 不是 2 的左结点，向上，2 是 1 的左结点，则节点 11 的后继结点为 1；

**补充：**如果要求前驱结点，方案如下：
如果结点有左子树，该结点的前驱就是左子树的最右边结点；
如果结点没有左子树，就一直往上找，直到指向的结点是其父结点的右孩子为止；

```java
package nowcoder.easy.day04;

public class PrintBinaryTree {

	public static class Node {
		public int value;
		public Node left;
		public Node right;

		public Node(int data) {
			this.value = data;
		}
	}

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
		Node head = new Node(1);
		head.left = new Node(-222222222);
		head.right = new Node(3);
		head.left.left = new Node(Integer.MIN_VALUE);
		head.right.left = new Node(55555555);
		head.right.right = new Node(66);
		head.left.left.right = new Node(777);
		printTree(head);

		head = new Node(1);
		head.left = new Node(2);
		head.right = new Node(3);
		head.left.left = new Node(4);
		head.right.left = new Node(5);
		head.right.right = new Node(6);
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


### 介绍二叉树的序列化和反序列化

**序列化：** 就是将在内存中构建的二叉树，怎么使用文件进行存储（一般变成字符串进行处理）
**反序列化：** 就是将文件中序列化的二叉树取出重新还原成二叉树；
**方案**
- 使用先序遍历、中序遍历、后续遍历其中之一，同样反序列化也应该使用同样方式；
- 使用按层序列化；

**以先序遍历为例：**
树的结构为：
![题目四序列化示例]($resource/%E9%A2%98%E7%9B%AE%E5%9B%9B%E5%BA%8F%E5%88%97%E5%8C%96%E7%A4%BA%E4%BE%8B.png)

先序遍历结果：`1_2_4_#_#_5_#_#_3_6_#_#_7_#_#_`
其中：`#`表示该结点为空，`_`表示值的结束符，这两个都可以使用其他符号进行代替；如果不使用`#`则无法表示整个树都是一样的情况，；
反序列化：从数组的第一个元素开始，因为是先序遍历，因此该结点一定是根节点，然后开始搭建左子树，1 的左孩子是 2，2 的左孩子是 4，然后是 #，则 4 的左孩子没有，返回到节点 4，下一个还是# ，则 4 的右孩子也没有，则回到 2，。。。。

按层序列化：上面图片中的二叉树按层遍历的结果为：`1_2_3_5_#_#_6_#_#_#_#_`

```java
package nowcoder.easy.day04;

import java.util.LinkedList;
import java.util.Queue;

public class SerializeAndReconstructTree {

	public static class Node {
		public int value;
		public Node left;
		public Node right;

		public Node(int data) {
			this.value = data;
		}
	}

	public static String serialByPre(Node head) {
		if (head == null) {
			return "#!";
		}
		String res = head.value + "!";
		res += serialByPre(head.left);
		res += serialByPre(head.right);
		return res;
	}

	public static Node reconByPreString(String preStr) {
		// 首先将字符串分割
		String[] values = preStr.split("!");
		Queue<String> queue = new LinkedList<String>();
		for (int i = 0; i != values.length; i++) {
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

	public static String serialByLevel(Node head) {
		if (head == null) {
			return "#!";
		}
		String res = head.value + "!";
		Queue<Node> queue = new LinkedList<Node>();
		queue.offer(head);
		while (!queue.isEmpty()) {
			head = queue.poll();
			if (head.left != null) {
				res += head.left.value + "!";
				queue.offer(head.left);
			} else {
				res += "#!";
			}
			if (head.right != null) {
				res += head.right.value + "!";
				queue.offer(head.right);
			} else {
				res += "#!";
			}
		}
		return res;
	}

	public static Node reconByLevelString(String levelStr) {
		String[] values = levelStr.split("!");
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
		Node head = null;
		printTree(head);

		String pre = serialByPre(head);
		System.out.println("serialize tree by pre-order: " + pre);
		head = reconByPreString(pre);
		System.out.print("reconstruct tree by pre-order, ");
		printTree(head);

		String level = serialByLevel(head);
		System.out.println("serialize tree by level: " + level);
		head = reconByLevelString(level);
		System.out.print("reconstruct tree by level, ");
		printTree(head);

		System.out.println("====================================");

		head = new Node(1);
		printTree(head);

		pre = serialByPre(head);
		System.out.println("serialize tree by pre-order: " + pre);
		head = reconByPreString(pre);
		System.out.print("reconstruct tree by pre-order, ");
		printTree(head);

		level = serialByLevel(head);
		System.out.println("serialize tree by level: " + level);
		head = reconByLevelString(level);
		System.out.print("reconstruct tree by level, ");
		printTree(head);

		System.out.println("====================================");

		head = new Node(1);
		head.left = new Node(2);
		head.right = new Node(3);
		head.left.left = new Node(4);
		head.right.right = new Node(5);
		printTree(head);

		pre = serialByPre(head);
		System.out.println("serialize tree by pre-order: " + pre);
		head = reconByPreString(pre);
		System.out.print("reconstruct tree by pre-order, ");
		printTree(head);

		level = serialByLevel(head);
		System.out.println("serialize tree by level: " + level);
		head = reconByLevelString(level);
		System.out.print("reconstruct tree by level, ");
		printTree(head);

		System.out.println("====================================");

		head = new Node(100);
		head.left = new Node(21);
		head.left.left = new Node(37);
		head.right = new Node(-42);
		head.right.left = new Node(0);
		head.right.right = new Node(666);
		printTree(head);

		pre = serialByPre(head);
		System.out.println("serialize tree by pre-order: " + pre);
		head = reconByPreString(pre);
		System.out.print("reconstruct tree by pre-order, ");
		printTree(head);

		level = serialByLevel(head);
		System.out.println("serialize tree by level: " + level);
		head = reconByLevelString(level);
		System.out.print("reconstruct tree by level, ");
		printTree(head);

		System.out.println("====================================");

	}
}

```

### 折纸问题
【题目】 请把一段纸条竖着放在桌子上，然后从纸条的下边向上方对折1次，压出折痕后展开。此时 折痕是凹下去的，即折痕突起的方向指向纸条的背面。如果从纸条的下边向上方连续对折2 次，压出折痕后展开，此时有三条折痕，从上到下依次是下折痕、下折痕和上折痕。
给定一 个输入参数N，代表纸条都从下边向上方连续对折N次，请从上到下打印所有折痕的方向。 例如：N=1时，打印： down N=2时，打印： down down up


```java
package nowcoder.easy.day04;

public class PaperFolding {

	public static void printAllFolds(int N) {
		printProcess(1, N, true);
	}

	public static void printProcess(int i, int N, boolean down) {
		if (i > N) {
			return;
		}
		printProcess(i + 1, N, true);
		System.out.println(down ? "down " : "up ");
		printProcess(i + 1, N, false);
	}

	public static void main(String[] args) {
		int N = 4;
		printAllFolds(N);
	}
}

```

### 判断一棵二叉树是否是平衡二叉树

平衡二叉树：对于任意一棵子树，左子树和右子树的高度差不能超过 1；
**思路：**
首先将问题分解：只要保证以每个结点为根节点的树是否平衡；
判断顺序，首先判断该结点的左子树是否平衡，然后判断该结点的右子树是否平衡，如果两个子树都平衡，分别计算左树和右树的高度；因此设计的递归函数的返回值应该是树是否是平稳的，同时返回数的高度；

**注意点：**
因为使用递归进行二叉树的遍历的时候，每个递归函数会到一个节点 3 次：首先会到一次，然后访问左子树之后会回来一次，最后访问右子树之后会回来一次；
**心得：** 对于二叉树使用递归的时候，首先想办法收集一下左子树上的信息，然后想办法收集一下右子树上的信息，最后将这些信息进行整合皆可以得到该结点所在的整棵树符不符合标准；本题中的标准就是判断该数是否平衡；
```java
package nowcoder.easy.day04;

public class IsBalancedTree {

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


### 判断一棵树是否是搜索二叉树、判断一棵树是否是完全二叉树

**搜索二叉树 BST：** 任何一个结点的左子树的值都比该结点小，右子树的值都比该结点的值大；（不存在重复的结点，因为重复结点的值可以以 list 进行存放）

**解法：** 根据二叉树的中序遍历是否递增；
下面代码提供的是递归版本，如果想要非递归版本，只要将中序遍历的非递归版本的打印方法去掉，然后改成与上一个结点值的比较即可（可以采用一个变量保存上一个结点的值）；


**完全二叉树 CBT：** 
**解法：** 将二叉树按层遍历，如果结点有右孩子但是没有左孩子则一定不是完全二叉树，如果不符合上面，则接着判断是否是 有左孩子没有右孩子或者是两个都没有，如果是这种情况则该结点下面的所有节点都必须是叶子结点；否则就不是完全二叉树；

```java
package nowcoder.easy.day04;

import java.util.LinkedList;
import java.util.Queue;

public class IsBSTAndCBT {

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

	public static boolean isCBT(Node head) {
		if (head == null) {
			return true;
		}
		Queue<Node> queue = new LinkedList<Node>();
		boolean leaf = false;
		Node l = null;
		Node r = null;
		queue.offer(head);
		while (!queue.isEmpty()) {
			head = queue.poll();
			l = head.left;
			r = head.right;
			if ((leaf && (l != null || r != null)) || (l == null && r != null)) {
				return false;
			}
			if (l != null) {
				queue.offer(l);
			}
			if (r != null) {
				queue.offer(r);
			} else {
				// 左等于空或者右等于空 则开启。因为上面代码已经去掉左等于空的情况，因此这里只需要判断右是否为空；
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


### 已知一棵完全二叉树，求其节点的个数
要求：时间复杂度低于O(N)，N为这棵树的节点个数

**结论：** 对于满二叉树，如果数的高度为 L，则其节点数为 ${2}^{L} -1$，这里采用的方法如果一共有 N 个结点，每一层都只会遍历一个，不是左孩子就是右孩子，则一共 $O(log_{2}^{N})$，到了某个结点之后，一直往下遍历的时间复杂度也是 $O(log_{2}^{N})$，最终结果就是两者乘积；

```java
package nowcoder.easy.day04;

public class CompleteTreeNodeNumber {

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

	// node 表示当前节点， level：当前节点在第几层，h:整棵树的高度，为定值；返回值是以这个节点为头的子树一共有多少个节点；
	public static int bs(Node node, int level, int h) {
		if (level == h) {
			return 1;
		}
		// node 的右子树上的左边界到 h 层
		if (mostLeftLevel(node.right, level + 1) == h) {
			// 1 << (h - level)表示当前节点的左子树和当前节点的节点个数和，2^(h - level)个
			// 因为右孩子也是一个完全二叉树，使用递归求其总节点，就是后面部分；
			return (1 << (h - level)) + bs(node.right, level + 1, h);
		} else {
			// 没有到 h 层，则右树的高度比左树少一个，1 << (h - level - 1))就是右树所有节点加上当前节点个数，然后后面是左树也是完全二叉树，递归求解；
			return (1 << (h - level - 1)) + bs(node.left, level + 1, h);
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












