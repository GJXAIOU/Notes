# AlgorithmMediumDay02

[TOC]

## 一、找到新的被指的新类型字符

```java
package com.gjxaiou.advanced.day02;

public class Code_03_FindNewTypeChar {

    public static String pointNewchar(String s, int k) {
        if (s == null || s.equals("") || k < 0 || k >= s.length()) {
            return "";
        }
        char[] chas = s.toCharArray();
        int uNum = 0;
        for (int i = k - 1; i >= 0; i--) {
            if (!isUpper(chas[i])) {
                break;
            }
            uNum++;
        }
        if ((uNum & 1) == 1) {
            return s.substring(k - 1, k + 1);
        }
        if (isUpper(chas[k])) {
            return s.substring(k, k + 2);
        }
        return String.valueOf(chas[k]);
    }

    public static boolean isUpper(char ch) {
        return !(ch < 'A' || ch > 'Z');
    }

    public static void main(String[] args) {
        String str = "aaABCDEcBCg";
        System.out.println(pointNewchar(str, 7));
        System.out.println(pointNewchar(str, 4));
        System.out.println(pointNewchar(str, 10));

    }

}

```



## 二、字典树（前缀树）的实现

【题目】
字典树又称为前缀树或Trie树，是处理字符串常见的数据结构。
假设组成所有单词的字符仅是“a”~“z”，请实现字典树结构，
并包含以下四个主要功能。
`void insert(String word)`：添加word，可重复添加。
`void delete(String word)`：删除word，如果word添加过多次，仅删除一个。
`boolean search(String word)`：查询word是否在字典树中。
`int prefixNumber(String pre)`：返回以字符串pre为前缀的单词数

```java
package com.gjxaiou.advanced.day02;

public class Code_04_TrieTree {

	public static class TrieNode {
		public int path;
		public int end;
		public TrieNode[] map;

		public TrieNode() {
			path = 0;
			end = 0;
			map = new TrieNode[26];
		}
	}

	public static class Trie {
		private TrieNode root;

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
				if (node.map[index] == null) {
					node.map[index] = new TrieNode();
				}
				node = node.map[index];
				node.path++;
			}
			node.end++;
		}

		public void delete(String word) {
			if (search(word)) {
				char[] chs = word.toCharArray();
				TrieNode node = root;
				int index = 0;
				for (int i = 0; i < chs.length; i++) {
					index = chs[i] - 'a';
					if (node.map[index].path-- == 1) {
						node.map[index] = null;
						return;
					}
					node = node.map[index];
				}
				node.end--;
			}
		}

		public boolean search(String word) {
			if (word == null) {
				return false;
			}
			char[] chs = word.toCharArray();
			TrieNode node = root;
			int index = 0;
			for (int i = 0; i < chs.length; i++) {
				index = chs[i] - 'a';
				if (node.map[index] == null) {
					return false;
				}
				node = node.map[index];
			}
			return node.end != 0;
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
				if (node.map[index] == null) {
					return 0;
				}
				node = node.map[index];
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



## 三、数字的英文表达和中文表达

【题目】
给定一个32位整数num，写两个函数分别返回num的英文与中文表达字符串。
【举例】
num=319
英文表达字符串为：Three Hundred Nineteen
中文表达字符串为：三百一十九
num=1014
英文表达字符串为：One Thousand, Fourteen
中文表达字符串为：一千零十四
num=-2147483648
英文表达字符串为：Negative, Two Billion, One Hundred Forty Seven Million, Four
Hundred Eighty Three Thousand, Six Hundred Forty Eight
中文表达字符串为：负二十一亿四千七百四十八万三千六百四十八
num=0
英文表达字符串为：Zero
中文表达字符串为：零

```java
package com.gjxaiou.advanced.advanced_class_02;

public class Code_05_EnglishExpression {

	public static String num1To19(int num) {
		if (num < 1 || num > 19) {
			return "";
		}
		String[] names = { "One ", "Two ", "Three ", "Four ", "Five ", "Six ",
				"Seven ", "Eight ", "Nine ", "Ten ", "Eleven ", "Twelve ",
				"Thirteen ", "Fourteen ", "Fifteen ", "Sixteen ", "Sixteen ",
				"Eighteen ", "Nineteen " };
		return names[num - 1];
	}

	public static String num1To99(int num) {
		if (num < 1 || num > 99) {
			return "";
		}
		if (num < 20) {
			return num1To19(num);
		}
		int high = num / 10;
		String[] tyNames = { "Twenty ", "Thirty ", "Forty ", "Fifty ",
				"Sixty ", "Seventy ", "Eighty ", "Ninety " };
		return tyNames[high - 2] + num1To19(num % 10);
	}

	public static String num1To999(int num) {
		if (num < 1 || num > 999) {
			return "";
		}
		if (num < 100) {
			return num1To99(num);
		}
		int high = num / 100;
		return num1To19(high) + "Hundred " + num1To99(num % 100);
	}

	public static String getNumEngExp(int num) {
		if (num == 0) {
			return "Zero";
		}
		String res = "";
		if (num < 0) {
			res = "Negative, ";
		}
		if (num == Integer.MIN_VALUE) {
			res += "Two Billion, ";
			num %= -2000000000;
		}
		num = Math.abs(num);
		int high = 1000000000;
		int highIndex = 0;
		String[] names = { "Billion", "Million", "Thousand", "" };
		while (num != 0) {
			int cur = num / high;
			num %= high;
			if (cur != 0) {
				res += num1To999(cur);
				res += names[highIndex] + (num == 0 ? " " : ", ");
			}
			high /= 1000;
			highIndex++;
		}
		return res;
	}

	public static int generateRandomNum() {
		boolean isNeg = Math.random() > 0.5 ? false : true;
		int value = (int) (Math.random() * Integer.MIN_VALUE);
		return isNeg ? value : -value;
	}

	public static void main(String[] args) {
		System.out.println(getNumEngExp(0));
		System.out.println(getNumEngExp(Integer.MAX_VALUE));
		System.out.println(getNumEngExp(Integer.MIN_VALUE));
		int num = generateRandomNum();
		System.out.println(num);
		System.out.println(getNumEngExp(num));

	}

}

```



中文表达

```java
package com.gjxaiou.advanced.day02;

public class Code_06_ChineseExpression {

	public static String num1To9(int num) {
		if (num < 1 || num > 9) {
			return "";
		}
		String[] names = { "一", "二", "三", "四", "五", "六", "七", "八", "九" };
		return names[num - 1];
	}

	public static String num1To99(int num, boolean hasBai) {
		if (num < 1 || num > 99) {
			return "";
		}
		if (num < 10) {
			return num1To9(num);
		}
		int shi = num / 10;
		if (shi == 1 && (!hasBai)) {
			return "十" + num1To9(num % 10);
		} else {
			return num1To9(shi) + "十" + num1To9(num % 10);
		}
	}

	public static String num1To999(int num) {
		if (num < 1 || num > 999) {
			return "";
		}
		if (num < 100) {
			return num1To99(num, false);
		}
		String res = num1To9(num / 100) + "百";
		int rest = num % 100;
		if (rest == 0) {
			return res;
		} else if (rest >= 10) {
			res += num1To99(rest, true);
		} else {
			res += "零" + num1To9(rest);
		}
		return res;
	}

	public static String num1To9999(int num) {
		if (num < 1 || num > 9999) {
			return "";
		}
		if (num < 1000) {
			return num1To999(num);
		}
		String res = num1To9(num / 1000) + "千";
		int rest = num % 1000;
		if (rest == 0) {
			return res;
		} else if (rest >= 100) {
			res += num1To999(rest);
		} else {
			res += "零" + num1To99(rest, false);
		}
		return res;
	}

	public static String num1To99999999(int num) {
		if (num < 1 || num > 99999999) {
			return "";
		}
		int wan = num / 10000;
		int rest = num % 10000;
		if (wan == 0) {
			return num1To9999(rest);
		}
		String res = num1To9999(wan) + "万";
		if (rest == 0) {
			return res;
		} else {
			if (rest < 1000) {
				return res + "零" + num1To999(rest);
			} else {
				return res + num1To9999(rest);
			}
		}
	}

	public static String getNumChiExp(int num) {
		if (num == 0) {
			return "零";
		}
		String res = num < 0 ? "负" : "";
		int yi = Math.abs(num / 100000000);
		int rest = Math.abs((num % 100000000));
		if (yi == 0) {
			return res + num1To99999999(rest);
		}
		res += num1To9999(yi) + "亿";
		if (rest == 0) {
			return res;
		} else {
			if (rest < 10000000) {
				return res + "零" + num1To99999999(rest);
			} else {
				return res + num1To99999999(rest);
			}
		}
	}

	// for test
	public static int generateRandomNum() {
		boolean isNeg = Math.random() > 0.5 ? false : true;
		int value = (int) (Math.random() * Integer.MIN_VALUE);
		return isNeg ? value : -value;
	}

	public static void main(String[] args) {
		System.out.println(0);
		System.out.println(getNumChiExp(0));

		System.out.println(Integer.MAX_VALUE);
		System.out.println(getNumChiExp(Integer.MAX_VALUE));

		System.out.println(Integer.MIN_VALUE);
		System.out.println(getNumChiExp(Integer.MIN_VALUE));

		int num = generateRandomNum();
		System.out.println(num);
		System.out.println(getNumChiExp(num));

		num = generateRandomNum();
		System.out.println(num);
		System.out.println(getNumChiExp(num));

		num = generateRandomNum();
		System.out.println(num);
		System.out.println(getNumChiExp(num));

		num = generateRandomNum();
		System.out.println(num);
		System.out.println(getNumChiExp(num));

		System.out.println(getNumChiExp(10));
		System.out.println(getNumChiExp(110));
		System.out.println(getNumChiExp(1010));
		System.out.println(getNumChiExp(10010));
		System.out.println(getNumChiExp(1900000000));
		System.out.println(getNumChiExp(1000000010));
		System.out.println(getNumChiExp(1010100010));

	}
}

```



## 四、滑动窗口

**介绍窗口以及窗口内最大值或者最小值的更新结构（单调双向队列）**

### （一）滑动窗口的概念

**就是一个数组------有 L 和 R指 针，默认两个指针均位于数组的最左边即下标为 -1 的位置， 当有数字进入时 R 向右移动。当有数字删除时则 L 向右移动 且L 和R 不会回退且 L 不能到 R 右边。**

思路： **双端队列（链表）**：双端队列中既需要放置数据又需要放置位置下标，本质上存放下标就行，对应值根据下标从数组中就能获取。

![增加元素过程](AlgorithmMediumDay02.resource/%E5%A2%9E%E5%8A%A0%E5%85%83%E7%B4%A0%E8%BF%87%E7%A8%8B.jpg)



**上图运行逻辑为：** 如果想实现窗口内最大值的更新结构： 使得双端队列保证从大到小的顺序 

- 当放入元素时候 ：

    （R增加） **头部始终存放的是当前最大的元素**--- 如果即将进入双端队列的元素（因为 R 增加而从数组中取出放入双端队列中的元素）比上一个进入双端队列的元素小，则从尾部进入，新进入的元素直接连接在后面，否则 原来双端队列的尾部一直弹出（包含相等情况，因为晚过期） 直到为即将要放入的元素找到合适的位置，或者整个队列为空，然后放入新加入的元素。（见上面示例）

- 当删除元素时候：

    （L增加） ---L向右移---（index下标一定得保留）则需要检查当前头部元素index是否过期 若过期则需要从头部进行弹出

![img](AlgorithmMediumDay02.resource/20180621232650862.png)

**时间复杂度**：因为从到到位滑过，每个数只会进队列一次，出队列一次，在队列中删除的数是不找回的，因此时间复杂度为：$$O(N)$$



### （二）应用一：生成窗口最大值数组

**1.题目**：

有一个整型数组 arr 和一个大小为 w 的窗口从数组的最左边滑到最右边，窗口每次向右滑一个位置。

例如，数组为 [4,3,5,4,3,3,6,7]，窗口大小为 3 时候：

[4 3 5] 4 3 3 6 7   窗口中最大值为：5

4 [3 5 4] 3 3 6 7   窗口中最大值为：5

4 3 [5 4 3] 3 6 7   窗口中最大值为：5

4 3 5 [4 3 3] 6 7   窗口中最大值为：4

4 3 5 4 [3 3 6] 7   窗口中最大值为：6

4 3 5 4 3 [3 6 7]   窗口中最大值为：7

如果数组长度为 n，窗口大小为 w，则一共产生 n - w + 1 个窗口的最大值。

请实现一个函数：

输入：整型数组 arr，窗口大小为 w。

输出：一个长度为 n - w + 1 的数组 res，res[i]表示每一种窗口状态下的最大值。

上面的结果应该返回{5,5,5,4,6,7}

**代码：**

```java
package com.gjxaiou.advanced.day02;

import java.util.LinkedList;

/**
 * @Author GJXAIOU
 * @Date 2020/1/1 20:29
 */
public class SlidingWindowMaxArray {
    public static int[] getMaxWindow(int[] arr, int w) {
        if (arr == null || w < 1 || arr.length < w) {
            return null;
        }
        // LinkedList 就是一个标准的双向链表
        LinkedList<Integer> maxList = new LinkedList<Integer>();
        // 生成的结果数组
        int[] res = new int[arr.length - w + 1];
        int index = 0;
        for (int i = 0; i < arr.length; i++) {
            // 更新双端队列，如果双端队列不为空，并且尾结点(存的是下标)对应数组中的值是否小于等于当前值
            while (!maxList.isEmpty() && arr[maxList.peekLast()] <= arr[i]) {
                maxList.pollLast();
            }
            // 上面一直弹出，直到不符合然后加上当前值。
            maxList.addLast(i);
            // 上面加法是通用的，但是减法是针对该题定制的
            // 当过期的时候（当窗口形成之后再扩充才算过期）即窗口长度 > w，窗口形成过程中不会过期, i - w表示过期的下标
            if (maxList.peekFirst() == i - w) {
                maxList.pollFirst();
            }
            // 判断下标过期
            if (i >= w - 1) {
                // 当窗口已经形成了，记录每一步的res
                res[index++] = arr[maxList.peekFirst()];
            }
        }
        return res;
    }

    public static void printArray(int[] array) {
        for (int i = 0; i < array.length; i++) {
            System.out.print(array[i] + " ");
        }
        System.out.println();
    }

    public static void main(String[] args) {
        int[] arr = {4, 3, 5, 4, 3, 3, 6, 7};
        printArray(getMaxWindow(arr, 3));
    }
}
```



### （二）应用二：最大值-最小值<=num的子数组数量:

**题目**：给定数组 arr 和整数 num，共返回有多少个子数组满足如下情况：`max(arr[i...j]) - min(arr[i...j]) <= num`，其中 `max(arr[i...j])` 表示子数组 `arr[i...j]` 中最大值，`min(arr[i...j])` 表示子数组 `arr[i...j]` 中的最小值。

**要求**：如果数组长度为 N，请实现时间复杂度为 O（N）的解法。



- 子数组（必须连续）一共有：$$N^2$$ 个（0~1,0~2，。。。0~n；1~1，。。。）



**最简单思路： 暴力求解 两个for 依次遍历** 

```java
package com.gjxaiou.advanced.class02;

/**
 * @Author GJXAIOU
 * @Date 2020/1/1 20:59
 */
public class AllLessNumSubArray {
    // 暴力解法:O(N^3)
    public static int getNum1(int[] arr, int num) {
        int res = 0;
        // 双层 for 循环穷尽所有的子数组可能性。
        for (int start = 0; start < arr.length; start++) {
            for (int end = start; end < arr.length; end++) {
                if (isValid(arr, start, end, num)) {
                    res++;
                }
            }
        }
        return res;
    }

    public static boolean isValid(int[] arr, int start, int end, int num) {
        int max = Integer.MIN_VALUE;
        int min = Integer.MAX_VALUE;
        for (int i = start; i <= end; i++) {
            max = Math.max(max, arr[i]);
            min = Math.min(min, arr[i]);
        }
        return max - min <= num;
    }
}

```



分析时间复杂度： o(N3) n的3次方



**最优解思路：**

- 如果有一个子数组 L~R 已经符合要求（并且其最大值为 max，最小值为 min），则其中内部的子数组一定也符合要求，因为内部空间（即范围小于 L ~ R）中的最大值只会 <= max，并且最小值只会 >=min，所有相减的结果肯定小于等于 num。

- 同理如果已经不达标 则往两边扩也肯定不达标（因为扩大返回只会导致 max 值变大，min 值变小）。

- 总计规律： 就是L一直往右走 不回退 R也跟着往右扩大范围

首先数组左边固定在 0 位置，然后右边界一直扩充，但是同时维护一个窗口内最大值更新结构和窗口内最小值更新结构，使得每次扩充值之后都可以比较当前数组是否满足最大值 - 最小值 <= max 的情况，比如当到 X 下标的时候是满足的，但是 X + 1 下标就不满足的时候，则表示以 0 开头的满足的子数组数目为 X +1 个（0 ~ 0,0 ~1，。。0 ~X）。

然后 L 缩一个位置，到 1 的位置，然后更新现有的最大和最小窗口更新结构（因为更新结构中 0 下标的值需要弹出），然后 R 继续向右走进行判断，直到不可以了就可以得到所有以 1 开头的子数组个数。

其他类型，以此类推。



代码：

```java
package com.gjxaiou.advanced.class02;

import java.util.LinkedList;

/**
 * @Author GJXAIOU
 * @Date 2020/1/1 20:59
 */
public class AllLessNumSubArray {
    // 暴力解法:O(N^3)
    public static int getNum1(int[] arr, int num) {
        int res = 0;
        // 双层 for 循环穷尽所有的子数组可能性。
        for (int start = 0; start < arr.length; start++) {
            for (int end = start; end < arr.length; end++) {
                if (isValid(arr, start, end, num)) {
                    res++;
                }
            }
        }
        return res;
    }

    public static boolean isValid(int[] arr, int start, int end, int num) {
        int max = Integer.MIN_VALUE;
        int min = Integer.MAX_VALUE;
        for (int i = start; i <= end; i++) {
            max = Math.max(max, arr[i]);
            min = Math.min(min, arr[i]);
        }
        return max - min <= num;
    }

    /**
     * 使用双向最大最小值更新结构，时间复杂度为 O（N）
     */
    public static int getNum(int[] arr, int num) {
        if (arr == null || arr.length == 0) {
            return 0;
        }
        // 分别准备最大值和最小值更新结构
        LinkedList<Integer> qmax = new LinkedList<Integer>();
        LinkedList<Integer> qmin = new LinkedList<Integer>();
        int L = 0;
        int R = 0;
        int res = 0;
        while (L < arr.length) {
            while (R < arr.length) {
                while (!qmin.isEmpty() && arr[qmin.peekLast()] >= arr[R]) {
                    qmin.pollLast();
                }
                qmin.addLast(R);
                while (!qmax.isEmpty() && arr[qmax.peekLast()] <= arr[R]) {
                    qmax.pollLast();
                }
                qmax.addLast(R);
                // 不达标
                if (arr[qmax.getFirst()] - arr[qmin.getFirst()] > num) {
                    break;
                }
                R++;
            }
            if (qmin.peekFirst() == L) {
                qmin.pollFirst();
            }
            if (qmax.peekFirst() == L) {
                qmax.pollFirst();
            }
            res += R - L;
            // 换一个开头
            L++;
        }
        return res;
    }
}

```





## 五、单调栈

问题描述：给定一个数组 请确定每个元素左右距离最近的比它大的数字

![img](AlgorithmMediumDay02.resource/20180624215316588.png)

**常规想法：** 到某一个元素时，通过两个for 分别获取其左边比它大的和右边比他大的数字 时间复杂度为 $$O(n^2)$$



**最优解思路（单调栈）**，栈中实际放置下标即可。O(N)

- 首先一个按照从大到小顺序排序的栈结构 ，若在压栈过程中发现要压栈的元素和栈顶的元素相比要大，则弹出当前栈顶元素，并从开始弹出处记录，要压栈的元素就是其右边离栈顶元素最近比它大的数，之后继续弹出的下一个即为栈顶元素左边距离最近的一个元素。

![img](AlgorithmMediumDay02.resource/20180624215839593.png)

注意： 到数组末尾时 但是栈中依然有元素 则此时元素弹出 右为null 而左边为栈中的下一元素

记得 观察 这个元素弹出的驱动是啥？  之前的是因为右边要压栈的比栈顶元素要大 所以可以弹出并记录信息

![img](AlgorithmMediumDay02.resource/20180625215803883.png)



**特殊情况：**若出现相等元素情况，则将下标放在一起，等到出现比它们大的数字时再都依次弹出即可。

![image-20200101220719195](AlgorithmMediumDay02.resource/image-20200101220719195.png)





### （一） 应用一：构造数组的maxtree

定义二叉树的节点如下：

```
public class Node{
    public int value;
    public Node left;
    public Node right;
    public Node(int data){
        this.value=data;
    }
}
```



一个数组的MaxTree定义：

- 数组必须没有重复元素
- MaxTree是一棵二叉树，数组的每一个值对应一个二叉树节点
- 包括MaxTree树在内且在其中的每一棵子树上，值最大的节点都是树的头

给定一个没有重复元素的数组arr，写出生成这个数组的MaxTree的函数，要求如果数组长度为N，则时间负责度为O(N)、额外空间负责度为O(N)。

#### 实现思路

对每一个元素，从左边和右边各选择第一个比这个元素大的值，选择值较小的元素作为父节点。
  在【生成窗口最大数组】里面，已经掌握了，在O(N)时间复杂度里面，找到每个元素位置最近的比元素大的元素，同个这个套路，就可以构造一棵MaxTree了。

#### 证明

1 构造的不是森林
2 是一棵二叉树

证明：1  
  对于每一个树节点，都能往上找到一个节点，直到找到最大节点为止，这样所有树节点都有共同的父节点，这样构造出来的就是一棵树。

证明：2
  使用反证法解决，如果是一棵二叉树，那么对于每个作为父节点的元素，能够在元素的一边找到两个或两个以上的元素。存在如：[p, b1, x, b2]这样的结构，p是父节点、b1、b2为子节点, x为其他节点。

- 按照题目，可以设定：
      p > b1, p > b2
- 当b1 > b2：
      b2不会选择p作为父节点，可能选择b1作为父节点.
- 当b1 < b2：
      当x < b2时，b1不会选择p作为父节点，选择b2作为父节点.
      当x > b2时，b2不会选择p作为父节点，选择x作为父节点.

![img](https://img-blog.csdn.net/20180625221545761?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1b2R1bzE4dXA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

​         ![img](https://img-blog.csdn.net/201806252217091?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1b2R1bzE4dXA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



**注意**：树不要求一定要平衡

思路1 ： 按照大根堆思路建立 O(N)

思路2： 单调栈：O（N）

按照单调栈的思路找到每个元素左右最近比它大的元素---分以下几种情况进行讨论：

 1 若一个元素左右均是null 则它是全局最大的 直接作为根节点

 2 若一个元素左或者右 只存在一个 则只具有唯一的父节点

 3  若一个元素左右两个元素均存在 则选择其中最小的那个作为父节点

![img](https://img-blog.csdn.net/20180625222059635?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1b2R1bzE4dXA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

注意： 一定只能构成一棵树，不会成为多叉树或者森林：

因为数组中没有重复值，所有最大值肯定作为整棵树的头结点，而任何一个节点都会找到一个比它大的，然后放置在其下面，所以每个节点都有归属并且均以最大值为头结点 -》最终形成一棵树。

证明不是多叉树：即是证明每个结点下面最多只有两个子节点，即是证明每个结点的单侧只有一个数会挂在该结点的底下。

![image-20200103145926092](AlgorithmMediumDay02.resource/image-20200103145926092.png)



代码 ==核对一下==

```java
package com.gjxaiou.advanced.class01;

import java.util.HashMap;
import java.util.Stack;

/**
 * @Author GJXAIOU
 * @Date 2020/1/3 15:05
 */
public class MaxTree {
    public static class Node {
        public int value;
        public Node left;
        public Node right;

        public Node(int data) {
            this.value = data;
        }
    }

    public static Node getMaxTree(int[] arr) {
        Node[] nArr = new Node[arr.length];
        for (int i = 0; i < arr.length; i++) {
            nArr[i] = new Node(arr[i]);
        }
        HashMap<Node, Node> lBigMap = new HashMap<Node, Node>();
        HashMap<Node, Node> rBigMap = new HashMap<Node, Node>();
        Stack<Node> stack = new Stack<Node>();
        for (int i = arr.length - 1; i >= 0; i--) {
            Node curNode = nArr[i];
            while ((!stack.isEmpty()) && stack.peek().value < curNode.value) {
                popStackSetMap(stack, lBigMap);
            }
            stack.push(curNode);
        }
        while (!stack.isEmpty()) {
            popStackSetMap(stack, lBigMap);
        }

        for (int i = nArr.length - 1; i != -1; i--) {
            Node curNode = nArr[i];
            while ((!stack.isEmpty()) && stack.peek().value < curNode.value) {
                popStackSetMap(stack, rBigMap);
            }
            stack.push(curNode);
        }
        while (!stack.isEmpty()) {
            popStackSetMap(stack, rBigMap);
        }

        Node head = null;
        for (int i = 0; i != nArr.length; i++) {
            Node curNode = nArr[i];
            Node left = lBigMap.get(curNode);
            Node right = rBigMap.get(curNode);
            if (left == null && right == null) {
                head = curNode;
            } else if (left == null) {
                if (right.left == null) {
                    right.left = curNode;
                } else {
                    right.right = curNode;
                }
            } else if (right == null) {
                if (left.left == null) {
                    left.left = curNode;
                } else {
                    left.right = curNode;
                }
            } else {
                Node parent = left.value < left.value ? left : right;
                if (parent.left == null) {
                    parent.left = curNode;
                } else {
                    parent.right = curNode;
                }
            }
        }
        return head;
    }

    public static void popStackSetMap(Stack<Node> stack, HashMap<Node, Node> map) {
        Node popNode = stack.pop();
        if (stack.isEmpty()) {
            map.put(popNode, null);
        } else {
            map.put(popNode, stack.peek());
        }
    }

    public static void printPreOrder(Node head) {
        if (head == null) {
            return;
        }
        System.out.print(head.value + " ");
        printPreOrder(head.left);
        printPreOrder(head.right);
    }

    public static void main(String[] args) {
        int[] uniqueArr ={3, 4, 5, 1, 2};
        Node head = getMaxTree(uniqueArr);
        printPreOrder(head);
        System.out.println();
        printPreOrder(head);
    }
}


```



代码：

![img](https://img-blog.csdn.net/20180628215634109?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1b2R1bzE4dXA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

   ![img](https://img-blog.csdn.net/20180628215700907?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1b2R1bzE4dXA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![img](https://img-blog.csdn.net/20180628215813854?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1b2R1bzE4dXA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![img](https://img-blog.csdn.net/20180628215841863?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1b2R1bzE4dXA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



### （二）求最大子矩阵的大小： 

![img](https://img-blog.csdn.net/20180625223145661?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1b2R1bzE4dXA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)





**类似的，下图的直方图** （值可以相等），以每个矩阵的中心高度为杠---然后分别向左右去扩展，到比它小的时候就停止 ---并记录能够达成的最大格子数目

![img](https://img-blog.csdn.net/20180625223536951?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1b2R1bzE4dXA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**解法:**   此题用单调栈，但是是找到左边和右边离他最近的比它小的数---从小到大的顺序  

若压栈元素<当前栈顶元素 则弹出栈顶元素 ，并且结算值，例如放入  1 -> 3 时候会弹出 0 -》4 则表示 0 位置数的距离其最近的比它小的数在 1 位置，同时 0-》 4 下面没有元素，表示最左边可以扩充到 -1 位置，因为 -1 和 1 位置都不可达，所以其真正能扩充的只有它自己。 

如果栈下面没有元素 则一定能够到达左边界 右边界就是那个压栈元素所在值  

若到最后栈里有元素 但是没有主动让元素弹出的右端元素 则可以最右到达右边界 左边就是下面的元素（两边不可达）

![img](AlgorithmMediumDay01.resource/20180625224236941.png)

**转换到原始问题中** ：

数组为 10 11 带入上面代码求出，以第 0 行为底的所有长方形中，含有1 最大的长方形；

则 对于1 行  2 1 2 2（表示以该行开始往上看有多少个连续的 1）带入上面代码求出以 第 1 行为底的。。。。

对于行 3 2 2 0 （注意此时为0 因为最上面是0 并没构成连续的1）

目的就是找到 每一行打底的最大的1的长方形

![img](AlgorithmMediumDay01.resource/201806252302410.png)



分析时间复杂度：因为每行运算都是 O（m），然后一共 N 行，所以是 O（n*m ) 就是遍历一遍矩阵

![img](AlgorithmMediumDay01.resource/2018062523051320.png)



代码

```java
package com.gjxaiou.advanced.class01;

import java.util.Stack;

/**
 * @Author GJXAIOU
 * @Date 2020/1/3 15:59
 */
public class MaximalRectangle {
    // 原问题（就是 0 1 矩阵）
    public static int maxRecSize(int[][] map) {
        if (map == null || map.length == 0 || map[0].length == 0) {
            return 0;
        }
        int maxArea = 0;
        int[] height = new int[map[0].length];
        for (int i = 0; i < map.length; i++) {
            for (int j = 0; j < map[0].length; j++) {
                // 形成辅助数组：就是 [10 11][2 1 2 2]等等
                height[j] = map[i][j] == 0 ? 0 : height[j] + 1;
            }
            maxArea = Math.max(maxRecFromBottom(height), maxArea);
        }
        return maxArea;
    }

    // 最基本方法，即一个数组代表直方图的话在其中找到最大矩形
    public static int maxRecFromBottom(int[] height) {
        if (height == null || height.length == 0) {
            return 0;
        }
        int maxArea = 0;
        // 准备一个单调栈，栈中放置下标
        Stack<Integer> stack = new Stack<Integer>();
        for (int i = 0; i < height.length; i++) {
            // 当栈不为空，且当前数小于等于栈顶
            while (!stack.isEmpty() && height[i] <= height[stack.peek()]) {
                // 弹出栈顶
                int j = stack.pop();
                // k 为左边界 （即弹出的数的下面是什么）
                int k = stack.isEmpty() ? -1 : stack.peek();
                // i 为当前数，就是右边界，自己在 k 位置上
                int curArea = (i - k - 1) * height[j];
                maxArea = Math.max(maxArea, curArea);
            }
            stack.push(i);
        }
        // 遍历完成之后，栈中剩余元素进行结算
        while (!stack.isEmpty()) {
            int j = stack.pop();
            int k = stack.isEmpty() ? -1 : stack.peek();
            int curArea = (height.length - k - 1) * height[j];
            maxArea = Math.max(maxArea, curArea);
        }
        return maxArea;
    }

    public static void main(String[] args) {
        int[][] map = {{1, 0, 1, 1}, {1, 1, 1, 1}, {1, 1, 1, 0}};
        int maxArea = maxRecSize(map);
        System.out.println("maxArea = " + maxArea);
    }
}

```



### （三）回形山

使用一个数组表示环形山，然后每座山上放烽火，相邻山之间的烽火可以看见；因为任意两座山之间可以通过顺时针或者逆时针的方式互相到达，如果两种方式中一条经过的数字都不大于两座山值的较小的一个，则两座山相互可见（如 3,4 之间可以看见），返回能返回相互看见的山峰有多少对。

![image-20200103164426898](AlgorithmMediumDay02.resource/image-20200103164426898.png)

如果数组中所有值都不相等：

1 座山峰规定为 0 对，2 座山峰规定为 1 对，如果大于 2 座，则 i 座山峰共 （ 2 * i - 3 ）对。

找发规定为：小的找大的，

![image-20200103171117637](AlgorithmMediumDay02.resource/image-20200103171117637.png)



但是实际问题中是可能含有最大值的，因此首先找到数组中的最大值，如果有多个取第一个，从该值开始进行环形遍历（例如：3  3 5  4  5，遍历顺序为：5 4  5 3  3）

![image-20200103173145387](AlgorithmMediumDay02.resource/image-20200103173145387.png)

针对遍历结束之后单调栈中剩余的元素，倒数第 3 及其以上的仍然适用于原来的公式，其余倒数第 2 和倒数第 1 单独计算。

![image-20200103174347520](AlgorithmMediumDay02.resource/image-20200103174347520.png)

```java
package com.gjxaiou.advanced.class01;

import java.awt.*;
import java.util.Scanner;
import java.util.Stack;

/**
 * @Author GJXAIOU
 * @Date 2020/1/3 17:44
 */
public class MountainsAndFlame {
    public static void main(String[] args) {
        // 输入两部分值：数组长度和数组具体的内容
        Scanner in = new Scanner(System.in);
        while (in.hasNextInt()) {
            int size = in.nextInt();
            int[] arr = new int[size];
            for (int i = 0; i < size; i++) {
                arr[i] = in.nextInt();
            }
            System.out.println(communications(arr));
        }
        in.close();
    }

    // 在环形数组中 i 位置的下一位，没到底则 +1，到底就为 0
    public static int nextIndex(int size, int i) {
        return i < (size - 1) ? (i + 1) : 0;
    }

    public static long getInternalSum(int n) {
        return n == 1L ? 0L : (long) n * (long) (n - 1) / 2L;
    }

    public static class Pair {
        public int value;
        public int times;

        public Pair(int value) {
            this.value = value;
            this.times = 1;
        }
    }

    // arr 为环形数组
    public static long communications(int[] arr) {
        if (arr == null || arr.length < 2) {
            return 0;
        }
        int size = arr.length;
        // 找到最大值位置（第一个）
        int maxIndex = 0;
        for (int i = 0; i < size; i++) {
            maxIndex = arr[maxIndex] < arr[i] ? i : maxIndex;
        }
        // value 为最大值
        int value = arr[maxIndex];
        int index = nextIndex(size, maxIndex);
        long res = 0L;
        // 首先将最大值扔入栈中
        Stack<Pair> stack = new Stack<Pair>();
        stack.push(new Pair(value));
        // 因为是从 maxIndex 的下一个位置开始遍历的，所有如果值再次相等说明遍历结束
        while (index != maxIndex) {
            // 数组中的当前值
            value = arr[index];
            while (!stack.isEmpty() && stack.peek().value < value) {
                int times = stack.pop().times;
                // res += getInernalSum(times) +times;
                // res += stack.isEmpty() ? 0 : times;
                // 下面结果为上面两句合并
                res += getInternalSum(times) + 2 * times;
            }
            // 如果当前值使得栈顶弹出，然后栈顶和弹出之后露出的栈顶值相同，则原来栈顶数目 +1
            if (!stack.isEmpty() && stack.peek().value == value) {
                stack.peek().times++;
                // 如果和弹出之后露出的栈顶值不相等，则将当前值放入栈中
            } else {
                stack.push(new Pair(value));
            }
            index = nextIndex(size, index);
        }
        // 遍历结束之后剩余栈中元素进行结算
        while (!stack.isEmpty()) {
            int times = stack.pop().times;
            res += getInternalSum(times);
            if (!stack.isEmpty()) {
                res += times;
                if (stack.size() > 1) {
                    res += times;
                } else {
                    res += stack.peek().times > 1 ? times : 0;
                }
            }
        }
        return res;
    }
}

```





