# AlgorithmEasyDay07

[TOC]

## 一、前缀树

### （一）前缀树作用

- 基本作用

    **给定一系列的字符串，判断有没有以某些字符开头的字符串**；

- 扩充用法1

    已有一系列字符串中**是否包含某个字符串**

    方案：在每一个节点上加上一个数据项，该数据项表示由多少字符串是以当前字符结尾的。

- 扩充用法2

    给定一系列字符串，**查询有多少字符串是以当前字符作为前缀的**。

    方案：在每个节点上再加上一个数据项，该数据项表示加入所有字符串时候划过当前节点的次数；



### （二）生成前缀树

将字符串不断的加入前缀树中，默认刚开始树的结构只有头结点。然后在加入新字符串时候，判断当前有没有通向该结点的路径，如果没有就新建否则就复用。

![生成前缀树](AlgorithmEasyDay07.resource/%E7%94%9F%E6%88%90%E5%89%8D%E7%BC%80%E6%A0%91.png)

**示例问题**：

- 一个字符串类型的数组 arr1，另一个字符串类型的数组 arr2，arr2 中有哪些字符是在 arr1 中出现的？请打印。

- arr2 中有哪些字符是作为 arr1 中某个字符串前缀出现的？请打印。

- arr2 中有哪些字符是作为 arr1 中某个字符串前缀出现的？请打印 arr2 中出现次数最大的前缀。



代码

```java
package com.gjxaiou.easy.class_07;

public class Code_01_TrieTree {

	public static class TrieNode {
		// 建立的时候经过结点的次数
		public int path;
		// 有多少字符串是以该结点结尾的
		public int end;
		// 一共有多少路，这里是 26 个字母，所有规定了 26 条路，通过标记该路的子节点是否为空来判断树是否存在
		public TrieNode[] nexts;

		public TrieNode() {
			path = 0;
			end = 0;
			nexts = new TrieNode[26];
		}
	}

	public static class Trie {
		private TrieNode root;
// 创建头结点
		public Trie() {
			root = new TrieNode();
		}

		public void insert(String word) {
			if (word == null) {
				return;
			}
			char[] chs = word.toCharArray();
			TrieNode node = root;
			int index = 0;
			for (int i = 0; i < chs.length; i++) {
				index = chs[i] - 'a';
				// 是否有指向该结点的路，没有就新建。
				if (node.nexts[index] == null) {
					node.nexts[index] = new TrieNode();
				}
				node = node.nexts[index];
				node.path++;
			}
			node.end++;
		}

		public void delete(String word) {
			if (search(word) != 0) {
				char[] chs = word.toCharArray();
				TrieNode node = root;
				int index = 0;
				for (int i = 0; i < chs.length; i++) {
					index = chs[i] - 'a';
					// 删除肯定 path -1 ，如果为 0，则下面结点就找不到了，置为空
					if (--node.nexts[index].path == 0) {
						node.nexts[index] = null;
						return;
					}
					node = node.nexts[index];
				}
				node.end--;
			}
		}
// 是否包括该字符串（扩展一）
		public int search(String word) {
			if (word == null) {
				return 0;
			}
			char[] chs = word.toCharArray();
			TrieNode node = root;
			int index = 0;
			for (int i = 0; i < chs.length; i++) {
				index = chs[i] - 'a';
				if (node.nexts[index] == null) {
					return 0;
				}
				node = node.nexts[index];
			}
			return node.end;
		}

		public int prefixNumber(String pre) {
			if (pre == null) {
				return 0;
			}
			char[] chs = pre.toCharArray();
			TrieNode node = root;
			int index = 0;
			for (int i = 0; i < chs.length; i++) {
				index = chs[i] - 'a';
				if (node.nexts[index] == null) {
					return 0;
				}
				node = node.nexts[index];
			}
			return node.path;
		}
	}

	public static void main(String[] args) {
		Trie trie = new Trie();
		System.out.println(trie.search("zuo"));
		trie.insert("zuo");
		System.out.println(trie.search("zuo"));
		trie.delete("zuo");
		System.out.println(trie.search("zuo"));
		trie.insert("zuo");
		trie.insert("zuo");
		trie.delete("zuo");
		System.out.println(trie.search("zuo"));
		trie.delete("zuo");
		System.out.println(trie.search("zuo"));
		trie.insert("zuoa");
		trie.insert("zuoac");
		trie.insert("zuoab");
		trie.insert("zuoad");
		trie.delete("zuoa");
		System.out.println(trie.search("zuoa"));
		System.out.println(trie.prefixNumber("zuo"));

	}

}

```

结果为：

```java
0
1
0
1
0
0
3
```



## 二、贪心

### （一）问题：金条分割代价最小（哈夫曼编码）

一块金条切成两半，是需要花费和长度数值一样的铜板的。比如长度为 20 的金条，不管切成长度多大的两半，都要花费 20 个铜板。一群人想整分整块金 条，怎么分最省铜板？ 例如,给定数组{10, 20, 30}，代表一共三个人，整块金条长度为 10+20+30=60. 金条要分成 10, 20, 30 三个部分。 如果， 先把长度 60 的金条分成 10 和 50，花费60 再把长度 50 的金条分成 20 和 30， 花费 50 一共花费 110 铜板。 但是如果， 先把长度 60 的金条分成 30 和30，花费 60 再把长度 30 金条分成 10 和 20，花费 30 一共花费 90 铜板。 输入一个数组，返回分割的最小代价。

**解答**

==本质上该题为 哈夫曼编码 问题==，也是比较经典的贪心问题。

哈夫曼编码问题示例：

最终结果为：{1,2,6,4,3,7,1,8}

**实现**：首先将所有元素形成小根堆，每次弹出最小的两个元素计算其和，然后将和再次放入堆中，以此反复直到只剩下一个元素，则所有的和为总代价。

步骤：

步一：拿出两个元素 1,1，和为 2，现在堆中元素为：2,6,4,3,7,8,2；

步二：拿出两个元素 2,2，和为 4，现在堆中元素为：6,4,3,7,8,4；

步三：拿出两个元素 3,4，和为 7，现在堆中元素为：6,7,8,4,7；

步四：拿出两个元素 4,6，和为 10，现在堆中元素为：7,8,7,10；

步五：拿出两个元素 7,7，和为 10，现在堆中元素为：8,10,14；

步六：拿出两个元素 8,10，和为 18，现在堆中元素为：18,14；

步六：拿出两个元素 18,14，和为 32，现在堆中元素为：32；

所有最终总代价为：2 + 4 + 7 + 10 + 14 + 18 + 32 



代码为：

```java
package com.gjxaiou.easy.class_07;

import java.util.Comparator;
import java.util.PriorityQueue;

public class Code_02_Less_Money {

	public static int lessMoney(int[] arr) {
		PriorityQueue<Integer> pQ = new PriorityQueue<>();
		for (int i = 0; i < arr.length; i++) {
			pQ.add(arr[i]);
		}
		int sum = 0;
		int cur = 0;
		while (pQ.size() > 1) {
			cur = pQ.poll() + pQ.poll();
			sum += cur;
			pQ.add(cur);
		}
		return sum;
	}

	public static class MinheapComparator implements Comparator<Integer> {

		@Override
		public int compare(Integer o1, Integer o2) {
			return o1 - o2; // < 0  o1 < o2  负数
		}

	}

	public static class MaxheapComparator implements Comparator<Integer> {

		@Override
		public int compare(Integer o1, Integer o2) {
			return o2 - o1; // <   o2 < o1
		}

	}

	public static void main(String[] args) {
		// solution
		int[] arr = { 6, 7, 8, 9 };
		System.out.println(lessMoney(arr));

		int[] arrForHeap = { 3, 5, 2, 7, 0, 1, 6, 4 };

		// min heap
		PriorityQueue<Integer> minQ1 = new PriorityQueue<>();
		for (int i = 0; i < arrForHeap.length; i++) {
			minQ1.add(arrForHeap[i]);
		}
		while (!minQ1.isEmpty()) {
			System.out.print(minQ1.poll() + " ");
		}
		System.out.println();

		// min heap use Comparator
		PriorityQueue<Integer> minQ2 = new PriorityQueue<>(new MinheapComparator());
		for (int i = 0; i < arrForHeap.length; i++) {
			minQ2.add(arrForHeap[i]);
		}
		while (!minQ2.isEmpty()) {
			System.out.print(minQ2.poll() + " ");
		}
		System.out.println();

		// max heap use Comparator
		PriorityQueue<Integer> maxQ = new PriorityQueue<>(new MaxheapComparator());
		for (int i = 0; i < arrForHeap.length; i++) {
			maxQ.add(arrForHeap[i]);
		}
		while (!maxQ.isEmpty()) {
			System.out.print(maxQ.poll() + " ");
		}
	}
}

```

结果为：

```java
60
0 1 2 3 4 5 6 7 
0 1 2 3 4 5 6 7 
7 6 5 4 3 2 1 0 
```



### （二）问题二：做项目完成最大收益

输入： 参数1，正数数组costs ；参数2，正数数组profits ；参数3， 正数k ；参数4，正数m ；costs[i]表示i号项目的花费， profits[i]表示i号项目在扣除花 费之后还能挣到的钱(利润)， k表示你不能并行、只能串行的最多 做k个项目， m表示你初始的资金 。说明：你每做完一个项目，马上获得的收益，可以支持你去做下 一个 项目。 输出： 你最后获得的最大钱数。

**解答**

首先每个项目包括两个属性：花费成本 cost 和利润 profile，然后将所有的项目放入数组中，示例如图所示：

![数组中项目结构](AlgorithmEasyDay07.resource/%E6%95%B0%E7%BB%84%E4%B8%AD%E9%A1%B9%E7%9B%AE%E7%BB%93%E6%9E%84.png)

然后将数组中所有的项目按照花费成本的大小形成一个小根堆，接着挨着从小根堆中弹出头部，只要花费成本小于初始资金（W）就都弹出，（弹出来的表示在当前资金下可以做的任务），然后将弹出的元素按照收益的高低放入大根堆中（则收益越高越优先做），每次只做大根堆的堆头部的任务，做完任务之后初始资金发生变化，再次从小根堆中弹出符合的元素进入大根堆，以此类推。直到大根堆中没有元素或者达到了最大 K 次就结束。



示例：以下任务的初始资金为：7

cost：    5   10    6   20

profile：3    2     4    9

首先将所有的项目放入小根堆，然后通过比较 cost 和初始资金，则首先放入大根堆中是： 6,4 和 5,3，先做堆顶项目6,4，做完之后收益变为：11，则任务 10,2 可以做了，现在大根堆中为：5，3；10，2，先做 5,3 任务，收益变为 14，没有新的任务可以加入大根堆中，则继续做大根堆中任务：10,2，昨晚之后收益变为：16，还是没有新的项目可以加入大根堆，这时候大根堆没有任务可以完成了，则最终停止。



代码为：

```java
package com.gjxaiou.easy.class_07;

import java.util.Comparator;
import java.util.PriorityQueue;

public class Code_03_IPO {
    public static class Node {
        public int p;
        public int c;

        public Node(int p, int c) {
            this.p = p;
            this.c = c;
        }
    }

    public static class MinCostComparator implements Comparator<Node> {
        @Override
        public int compare(Node o1, Node o2) {
            return o1.c - o2.c;
        }
    }

    public static class MaxProfitComparator implements Comparator<Node> {
        @Override
        public int compare(Node o1, Node o2) {
            return o2.p - o1.p;
        }
    }

    public static int findMaximizedCapital(int k, int W, int[] Profits, int[] Capital) {
        Node[] nodes = new Node[Profits.length];
        for (int i = 0; i < Profits.length; i++) {
            nodes[i] = new Node(Profits[i], Capital[i]);
        }

        PriorityQueue<Node> minCostQ = new PriorityQueue<>(new MinCostComparator());
        PriorityQueue<Node> maxProfitQ = new PriorityQueue<>(new MaxProfitComparator());
        for (int i = 0; i < nodes.length; i++) {
            minCostQ.add(nodes[i]);
        }
        for (int i = 0; i < k; i++) {
            while (!minCostQ.isEmpty() && minCostQ.peek().c <= W) {
                maxProfitQ.add(minCostQ.poll());
            }
            // 做不到 K 项目，即大根堆空了（资金不能做其他项目）也得停
            if (maxProfitQ.isEmpty()) {
                return W;
            }
            W += maxProfitQ.poll().p;
        }
        return W;
    }
}

```



### （三）问题：在一个数据流中，随时可以获取到中位数

在一个数据流中，随时可以获取到中位数：

代码：

```java
package com.gjxaiou.easy.class_07;

import java.util.Arrays;
import java.util.Comparator;
import java.util.PriorityQueue;

public class Code_04_MadianQuick {

	public static class MedianHolder {
		private PriorityQueue<Integer> maxHeap = new PriorityQueue<Integer>(new MaxHeapComparator());
		private PriorityQueue<Integer> minHeap = new PriorityQueue<Integer>(new MinHeapComparator());

		private void modifyTwoHeapsSize() {
			if (this.maxHeap.size() == this.minHeap.size() + 2) {
				this.minHeap.add(this.maxHeap.poll());
			}
			if (this.minHeap.size() == this.maxHeap.size() + 2) {
				this.maxHeap.add(this.minHeap.poll());
			}
		}

		public void addNumber(int num) {
			if (this.maxHeap.isEmpty()) {
				this.maxHeap.add(num);
				return;
			}
			if (this.maxHeap.peek() >= num) {
				this.maxHeap.add(num);
			} else {
				if (this.minHeap.isEmpty()) {
					this.minHeap.add(num);
					return;
				}		
				if (this.minHeap.peek() > num) {
					this.maxHeap.add(num);
				} else {
					this.minHeap.add(num);
				}
			}
			modifyTwoHeapsSize();
		}

		public Integer getMedian() {
			int maxHeapSize = this.maxHeap.size();
			int minHeapSize = this.minHeap.size();
			if (maxHeapSize + minHeapSize == 0) {
				return null;
			}
			Integer maxHeapHead = this.maxHeap.peek();
			Integer minHeapHead = this.minHeap.peek();
			if (((maxHeapSize + minHeapSize) & 1) == 0) {
				return (maxHeapHead + minHeapHead) / 2;
			}
			return maxHeapSize > minHeapSize ? maxHeapHead : minHeapHead;
		}

	}

	public static class MaxHeapComparator implements Comparator<Integer> {
		@Override
		public int compare(Integer o1, Integer o2) {
			if (o2 > o1) {
				return 1;
			} else {
				return -1;
			}
		}
	}

	public static class MinHeapComparator implements Comparator<Integer> {
		@Override
		public int compare(Integer o1, Integer o2) {
			if (o2 < o1) {
				return 1;
			} else {
				return -1;
			}
		}
	}

	// for test
	public static int[] getRandomArray(int maxLen, int maxValue) {
		int[] res = new int[(int) (Math.random() * maxLen) + 1];
		for (int i = 0; i != res.length; i++) {
			res[i] = (int) (Math.random() * maxValue);
		}
		return res;
	}

	// for test, this method is ineffective but absolutely right
	public static int getMedianOfArray(int[] arr) {
		int[] newArr = Arrays.copyOf(arr, arr.length);
		Arrays.sort(newArr);
		int mid = (newArr.length - 1) / 2;
		if ((newArr.length & 1) == 0) {
			return (newArr[mid] + newArr[mid + 1]) / 2;
		} else {
			return newArr[mid];
		}
	}

	public static void printArray(int[] arr) {
		for (int i = 0; i != arr.length; i++) {
			System.out.print(arr[i] + " ");
		}
		System.out.println();
	}

	public static void main(String[] args) {
		boolean err = false;
		int testTimes = 200000;
		for (int i = 0; i != testTimes; i++) {
			int len = 30;
			int maxValue = 1000;
			int[] arr = getRandomArray(len, maxValue);
			MedianHolder medianHold = new MedianHolder();
			for (int j = 0; j != arr.length; j++) {
				medianHold.addNumber(arr[j]);
			}
			if (medianHold.getMedian() != getMedianOfArray(arr)) {
				err = true;
				printArray(arr);
				break;
			}
		}
		System.out.println(err ? "Oops..what a fuck!" : "today is a beautiful day^_^");

	}

}

```



### （四）问题：拼接字符串获得最低字典序

给定一个字符串类型的数组 strs，找到一种拼接方式，使得把所有字符串拼接起来之后形成的字符串具有最低的字典序。

**分析**

**此题的比较策略即为贪心策略**，但是不一定其它也是两者一样的。

贪心方案一（错误）：按照每个字符串字典序排序然后进行合并，例 “b” “ba” 排完之后是 “b”,”ba”，合并是：”bba”，但是实际上最小的是 “bab”;

贪心方案二：若 “str1 str2“ 拼接结果 <= “str2 str1” 拼接结果，则 str1 放在前面。

**判断自定义比较器的正确性**

自定义比较器正确的前提是：具有传递性并且结果值唯一，同时可以比较出所有值的大小，

例如：甲乙比较：甲 < 乙；乙丙比较：乙 < 丙；甲丙比较：丙 < 甲，结果具有随机性，就是比较不出来结果；

应该具有传递性，例如：甲 < 乙，乙 < 丙，然后甲 < 丙；

**字典序比较**

- 如果两个比较对象长度相等，这直接比较；
- 如果两个对象长度不等，短的后面补 ASCII 最小值，然后比较；

**证明比较器的传递性**

- 前言

    12,23 两个数字拼接为 1223，即 $$12 * 10^2 + 23$$，即 12 向左位移两位然后加上 23；

    a,b 两个字符串拼接为 ab，将字符串理解为 K 进制数，即 a 向左位移 b 的长度那么多位数然后加上 b  的值；

    所以：ab = $$a * k^{b 长度} + b$$，其中设 $$m(b) = k^{b 长度}$$，同理 $$m(c) = k^{c 长度}$$

证明过程：因为 ab <= ba，bc <= cb，证明：ac <= ca

因为 ab <= ba，所以 `a * m(b) + b <= b * m(a) + a`，等式两边同时减去 b 然后乘 c 得到：`a * m(b) *c <=(b * m(a) +a -b) * c`

因为 bc <= cb，所以 `b * m(c) + c <= c * m(b) + b`，等式两边同时减去 b 然后乘 a 得到：`(b * m(c) +(-b)) *a <= c * m(b) * a`

从上面两个等式推出：`(b * m(c) + (-b)) * a <= (b * m(a) + a - b) * c`，然后两边化简并且除以 b ，并且两边同时加上 a  和 c 之后得到 `a * m(c) + c <= c * m(a) + a)`，即 ac <= ca，证毕。



**证明任意两个交换都比现在的字典序大**

原来的：………k1 m1 m2 ………….mk k2…….

交换后：………k2 m1 m2 ………… mk k1…….

证明：因为 k1 m1 <= m1 k1 所以：

………k1 m1 m2 ………….mk k2……. 小于等于

………m1 k1 m2 ………….mk k2……. 小于等于

………m1 m2 k1 ………….mk k2……. 小于等于

。。。。。。。。。。。

………m1 m2 ………….mk k2 k1……. 小于等于

………m1 m2 ………….k2 mk k1……. 小于等于

。。。。。。。。。。。

………k2 m1 m2 ………….mk k1……. 

证毕，并可以证明多个数交换也符合（可以化简到两个元素交换）。

**判断贪心方案的正确性**

做题的时候不要妄图验证贪心方案的正确性，直接使用小数据样本输入该贪心方案和对数器的结果比较，如果小样本验证是正确的则默认该方案就是正确的。



代码：

```java
package com.gjxaiou.easy.class_07;

import java.util.Arrays;
import java.util.Comparator;

public class Code_05_LowestLexicography {

	public static class MyComparator implements Comparator<String> {
		// 自定义比较器
		@Override
		public int compare(String a, String b) {
			return (a + b).compareTo(b + a);
		}
	}

	public static String lowestString(String[] strs) {
		if (strs == null || strs.length == 0) {
			return "";
		}
		// 使用自己的比较器
		Arrays.sort(strs, new MyComparator());
		String res = "";
		for (int i = 0; i < strs.length; i++) {
			res += strs[i];
		}
		return res;
	}

	public static void main(String[] args) {
		String[] strs1 = { "jibw", "ji", "jp", "bw", "jibw" };
		System.out.println(lowestString(strs1));

		String[] strs2 = { "ba", "b" };
		System.out.println(lowestString(strs2));
	}
}

```

结果为：

```java
bwjibwjibwjijp
bab
```



### （五）问题：会议室安排宣讲次数最多

一些项目要占用一个会议室宣讲，会议室不能同时容纳两个项目 的宣讲。 给你每一个项目开始的时间和结束的时间(给你一个数 组，里面 是一个个具体的项目)，你来安排宣讲的日程，要求会 议室进行 的宣讲的场次最多。返回这个最多的宣讲场次。

**解答**

下面是不同的贪心策略：

方式一（错误）：最早开始的最先安排；

方式二（错误）：哪个会议持续时间长；

方式三（正确）：哪个项目最早结束，然后淘汰因为安排这个项目而耽误的其他的项目；

代码：

```java
package com.gjxaiou.easy.class_07;

import java.util.Arrays;
import java.util.Comparator;

public class Code_06_BestArrange {

	public static class Program {
		public int start;
		public int end;

		public Program(int start, int end) {
			this.start = start;
			this.end = end;
		}
	}

	public static class ProgramComparator implements Comparator<Program> {
		@Override
		public int compare(Program o1, Program o2) {
			return o1.end - o2.end;
		}
	}

	public static int bestArrange(Program[] programs, int currentTime) {
		Arrays.sort(programs, new ProgramComparator());
		int result = 0;
		for (int i = 0; i < programs.length; i++) {
			// 如果当前时间小于项目的开始时间，则项目可以安排
			if (currentTime <= programs[i].start) {
				// 项目可以做，即项目数 + 1;
				result++;
				// 当前时间来到项目的结束时间，表示项目做完了
				currentTime = programs[i].end;
			}
		}
		return result;
	}

	public static void main(String[] args) {

	}
}
```

