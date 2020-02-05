# AlgorithmMediumDay01

[TOC]

## 一、不用比较符返回较大数

给定两个数a和b，如何不用比较运算符，返回较大的数。

```java
package nowcoder.advanced.day01;

public class Code_01_GetMax {

	public static int flip(int n) {
		return n ^ 1;
	}

	public static int sign(int n) {
		return flip((n >> 31) & 1);
	}

	public static int getMax1(int a, int b) {
		int c = a - b;
		int scA = sign(c);
		int scB = flip(scA);
		return a * scA + b * scB;
	}

	public static int getMax2(int a, int b) {
		int c = a - b;
		int sa = sign(a);
		int sb = sign(b);
		int sc = sign(c);
		int difSab = sa ^ sb;
		int sameSab = flip(difSab);
		int returnA = difSab * sa + sameSab * sc;
		int returnB = flip(returnA);
		return a * returnA + b * returnB;
	}

	public static void main(String[] args) {
		int a = -16;
		int b = 1;
		System.out.println(getMax1(a, b));
		System.out.println(getMax2(a, b));
		a = 2147483647;
		b = -2147480000;
		System.out.println(getMax1(a, b)); // wrong answer because of overflow
		System.out.println(getMax2(a, b));

	}

}

```



## 二、二叉树和为 Sum 的最长路径长度

给定一棵二叉树的头节点head，和一个整数sum，二叉树每个节点上都有数字，我们规定路径必须是从上往下的，求二叉树上累加和为sum的最长路径长度。

```java
package nowcoder.advanced.day01;

import java.util.HashMap;

public class Code_03_LongestPathSum {

	public static class Node {
		public int value;
		public Node left;
		public Node right;

		public Node(int data) {
			this.value = data;
		}
	}

	public static int getMaxLength(Node head, int sum) {
		HashMap<Integer, Integer> sumMap = new HashMap<Integer, Integer>();
		sumMap.put(0, 0); // important
		return preOrder(head, sum, 0, 1, 0, sumMap);
	}

	public static int preOrder(Node head, int sum, int preSum, int level,
			int maxLen, HashMap<Integer, Integer> sumMap) {
		if (head == null) {
			return maxLen;
		}
		int curSum = preSum + head.value;
		if (!sumMap.containsKey(curSum)) {
			sumMap.put(curSum, level);
		}
		if (sumMap.containsKey(curSum - sum)) {
			maxLen = Math.max(level - sumMap.get(curSum - sum), maxLen);
		}
		maxLen = preOrder(head.left, sum, curSum, level + 1, maxLen, sumMap);
		maxLen = preOrder(head.right, sum, curSum, level + 1, maxLen, sumMap);
		if (level == sumMap.get(curSum)) {
			sumMap.remove(curSum);
		}
		return maxLen;
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
		Node head = new Node(-3);
		head.left = new Node(3);
		head.right = new Node(-9);
		head.left.left = new Node(1);
		head.left.right = new Node(0);
		head.left.right.left = new Node(1);
		head.left.right.right = new Node(6);
		head.right.left = new Node(2);
		head.right.right = new Node(1);
		printTree(head);
		System.out.println(getMaxLength(head, 6));
		System.out.println(getMaxLength(head, -9));

	}

}

```



## 一、KMP

### （一）基本知识

- 子序列：可以连续可以不连续；
- 子数组（子串）：必须是连续的；
- 但是题目中子序列和子数组可能不区分；



### （二）作用

- 解决原始问题：给定两个字符串 str 和 match，长度分别为 N 和 M，实现一个算法，如果字符串 str 中含有子串 match，则返回 match 在 str 中的开始位置，不含有则返回 -1。

  **举例**

  str=”acbc”, match=”bc”，返回 2。

  str=”acbc”, mathc=”bcc”, 返回 -1。

  **要求**

  如果 match 的长度大于 str 的长度（M > N），str 必然不会含有 match，则直接返回 -1，但是如果 N >= M，要求算法复杂度为 O(N)。



### （三）常规解法

每次都是从头开始一个个匹配，但是时间复杂度太高，最差的情况下是：$$O(N * M)$$

**大致思路**：从左到右遍历 str 的每一个字符，然后看如果以当前字符作为第一个字符出发是否匹配出 match。比如str="aaaaaaaaaaaaaaaaab"，match="aaaab"。从 str[0] 出发， 开始匹配，匹配到 str[4]='a' 时发现和 match[4]='b' 不一样，所以匹配失败，说明从 str[0] 出发是不行的。从 str[1] 出发，开始匹配，匹配到 str[5]='a' 时发现和 match[4]='b' 不一样， 所以匹配失败，说明从 str[1] 出发是不行的。从str[2...12]出发，都会一直失败。从 str[13] 出 发，开始匹配，匹配到 str[17]=='b'，时发现和 match[4]='b'—样，match己经全部匹配完，说 明匹配成功，返回13。普通解法的时间复杂度较高，从每个字符串发时，匹配的代价都可能是O(M)，那么一共有N个字符，所以整体的时间复杂度为 O（N*M）。普通解法的时间复杂度这么高，是因为每次遍历到一个字符时，检查工作相当于从无开始，之前的遍历检查 不能优化当前的遍历检查。

**总结**： 我们发现每次匹配前面的并没有给后面一些指导信息



### （四）KMP算法思路

- **首先根据 match 生成最大前后缀匹配长度数组 ：**

  首先生成 match 字符串的 nextArr 数组，这个数组的长度与 match 字符串的长度一 样，nextArr[i] 的含义是在 match[i] 之前的字符串 match[0..i-1]中，心须以 match[i-1] 结尾的后缀子串（不能包含match[0]）与必须以 match[0] 开头的前缀子串（不能包含match[i - 1]）最大匹配长度是多少。这个长度就是 nextArr[i] 的值。比如，match="aaaab"字符串，nextArr[4] 的值该是多少呢？`match[4]=="b"`，所以它之前的字符串为 "aaaa"，根据定义这个字符串的后缀子串和前缀子串最大匹配为 "aaa"。也就是当后缀子串等于 match[1...3] = "aaa"，前缀子串等于 `match[0..2]=="aaa"` 时，这时前缀和后缀不仅相等，而且是所有前缀和后缀的可能性中

注：数组中值表示一个字符之前的所有字符串的最长前缀和最长后缀的匹配长度（前缀不包括数组中最后一个字符，后缀不包含第一个字符）

示例：对于数组： a b c a b c d，则元素 d  的最长XXXX为 3

前缀： a       ab         abc             abca          abcab   不能包含最后一个，至此截止，所以最长为 3.    

后缀： c        bc         abc             cabc          bcabc

值：  不等   不等    相等，为3     不等           不等



**2  从str[i]字符出发---匹配到j位置上发现开始出现不匹配：**

假设从 str[i] 字符出发时候，匹配到 j  位置的字符发现与 match 中的字符不一致，也就是说， str[i] 与 match[0] 一样，并且从这个位置开始一直可以匹配，即 str[i..j-1] 与 match[0..j-i-1] 一样，知道发现 str[j] != match[j -i]，匹配停止，如图所示。

![image-20200102144804517](AlgorithmMediumDay01.resource/image-20200102144804517.png)



**3 下一次 match 数组往右滑动， 滑动大小为 match 当前字符的最大前后缀匹配长度，再让 str[j] 与 match[k] 进行下一次匹配~**

那么下一次的匹配检查不再像普通解法那样退回到str[i+1]重新开始与 match[0] 的匹配过程，而是直接让 str[j] 与 match[k] 进行匹配检查，如图所示.在图中，在 str 中要匹配的位置仍是 j，而不进行退回。对 match来说，相当于向右滑动，让 match[k] 滑动到与 str[j] 同一个位置上，然后进行后续的匹配检查。普通解法 str 要退回到 i+1 位置，然后让 str[i+1] 与 match[0] 进行匹配，而我们的解法在匹配的过程中一直进行这样的滑动匹配的过程，直到在 str 的某一个位置把 match 完全匹配完，就说明str 中有 match。如果 match 滑到最后也没匹配出来，就说明str 中没有 match。

![img](AlgorithmMediumDay01.resource/20180617171300922.png)



**4 注意为什么加快，主要是str[i]--c前那段不用考虑，因为肯定不能匹配~**

**若有匹配 则出现更大的前后缀匹配长度 ，与最初nextArr[] 定义违背。**

![img](AlgorithmMediumDay01.resource/20180617171438808.png)

**关于nextArr数组的求解：**

![img](https://img-blog.csdn.net/20180617211712204?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1b2R1bzE4dXA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![img](https://img-blog.csdn.net/20180617214652560?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1b2R1bzE4dXA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

4.如果向前跳到最左位置（即 match[0] 的位置），此时 nextArr[0] == -1，说明字符 A 之前的字符串不存在前缀和后缀匹配的情况，则令 nextArr[i] = 0。用这种不断向前跳的方式可以算出正确的 nextArr[i] 值的原因还是因为每跳到一个位置 cn，nextArr[cn] 的意义就表示它之前字符串的最大匹配长度。求解 nextArr 数组的具体过程请参照如下代码中的 getNextArray 方法。



**讨论计算复杂度：**

getNextArray 方法中的 while 循环就是求解 nextArr 数组的过程，现在证明这个循环发生的次数不会超过 2M 这个数量。先来看 pos 量，一个为（pos-cn）的量，对 pos 量来说，从 2 开始又必然不会大于 match 的长度，即 pos < M，对（pos-cn）量来说，pos 最大为 M - 1， cn 最小为 0，所以 （pos - cn）<= M。

循环的第一个逻辑分支会让 pos 的值增加，（pos - cn）的值不变。循环的第二个逻辑分支为 cn 向左跳的过程，所以会让 cn 减小， pos 值在这个分支中不变，所以 (pos - cn)的值会增加，循环的第三个逻辑分支会让 pos 的值增加，（pos- cn) 的值也会增加，如下表所示：

|                      | Pos  | pos-cn |
| -------------------- | ---- | ------ |
| 循环的第一个逻辑分支 | 增加 | 不变   |
| 循环的第二个逻辑分支 | 不变 | 增加 |
| 循环的第三个逻辑分支 | 增加 | 增加 |

因为 pos+ （pos - cn）< 2M，和上表的关系，所以循环发生的总体次数小于 pos 量 和 （pos - cn）量的增加次数，也必然小于 2M，证明完毕。

所以整个 KMP 算法的复杂度为 O(M)(求解 nextArr 数组的过程) + O(N)(匹配的过程)，因为有 N>= M，所以时间复杂度为 O（N）。

代码

```java
package nowcoder.advanced.day01;

/**
 * @author GJXAIOU
 */
public class KMP {

    public static int getIndexOf(String s, String m) {
        if (s == null || m == null || m.length() < 1 || s.length() < m.length()) {
            return -1;
        }
        char[] ss = s.toCharArray();
        char[] ms = m.toCharArray();
        int si = 0;
        int mi = 0;
        int[] next = getNextArray(ms);
        while (si < ss.length && mi < ms.length) {
            if (ss[si] == ms[mi]) {
                si++;
                mi++;
                // 数组中值等于 -1 ，说明是第一个元素，说明当前 str1  中值连 str2 第一个字母都匹配不上，则直接从 str1 的下一个开始进行匹配
            } else if (next[mi] == -1) {
                si++;
            } else {
                mi = next[mi];
            }
        }
        return mi == ms.length ? si - mi : -1;
    }

    // 求解 next 数组方法
    public static int[] getNextArray(char[] ms) {
        if (ms.length == 1) {
            return new int[]{-1};
        }
        int[] next = new int[ms.length];
        next[0] = -1;
        next[1] = 0;
        // 需要求值的位置
        int pos = 2;
        int cn = 0;
        while (pos < next.length) {
            if (ms[pos - 1] == ms[cn]) {
                next[pos++] = ++cn;
            } else if (cn > 0) {
                cn = next[cn];
            } else {
                next[pos++] = 0;
            }
        }
        return next;
    }

    public static void main(String[] args) {
        String str = "abcabcababaccc";
        String match = "ababa";
        System.out.println(getIndexOf(str, match));

    }

}

```
程序运行结果为：`6`


### （五）相关题目

- （京东）给定一个字符串 如何加最短的字符（只能在原始串的后面进行添加）使其构成一个长的字符串且包含两个原始字符串，并且开始位置还得不一样

  示例：abcabc ---->abcabcabc 最少增加 3 个

**思路：**其实就是最大前后缀长度数组

**步骤**：求原字符串每一个的 next 数组，得到原字符串最大长度**后一个**的 next 数组值。该值表明原字符串的最大可复用长度（next 数组的意义），然后用原字符串长度减去最大可复用长度，得到应该添加字符的长度，即从原字符串倒数取得，添加即可。

总结： 在 KMP 中 nextArr 数组基础上多求一位终止位，将不是的补上即可

```java
package nowcoder.advanced.day01;

/**
 * @author GJXAIOU
 */
public class ShortestHaveTwice {

    public static String answer(String str) {
        if (str == null || str.length() == 0) {
            return "";
        }
        char[] chas = str.toCharArray();
        if (chas.length == 1) {
            return str + str;
        }
        if (chas.length == 2) {
            return chas[0] == chas[1] ? (str + String.valueOf(chas[0])) : (str + str);
        }
        int endNext = endNextLength(chas);
        return str + str.substring(endNext);
    }

    public static int endNextLength(char[] chas) {
        int[] next = new int[chas.length + 1];
        next[0] = -1;
        next[1] = 0;
        int pos = 2;
        int cn = 0;
        while (pos < next.length) {
            if (chas[pos - 1] == chas[cn]) {
                next[pos++] = ++cn;
            } else if (cn > 0) {
                cn = next[cn];
            } else {
                next[pos++] = 0;
            }
        }
        return next[next.length - 1];
    }

    public static void main(String[] args) {
        String test1 = "a";
        System.out.println(answer(test1));

        String test2 = "aa";
        System.out.println(answer(test2));

        String test3 = "ab";
        System.out.println(answer(test3));

        String test4 = "abcdabcd";
        System.out.println(answer(test4));

        String test5 = "abracadabra";
        System.out.println(answer(test5));

    }

}

```
程序运行结果为
```java
aa
aaa
abab
abcdabcdabcd
abracadabracadabra
```


- 给定两个树（值可以相同或者不同） 判断数 1 的某棵子树是否包含树 2（结构和值完全相同），  是则返回true

子树就是从某个头结点开始下面所有子节点都要，下图第一个图就是可以 true，另一个图就是 false。

<img src="AlgorithmMediumDay01.resource/image-20191231154124355.png" alt="image-20191231154124355" style="zoom:50%;" />



**思路**： 把一棵树序列化（可以先序、中序、后续）为字符串（字符数组）  。如果 str2 是 str1的子串 则T2也是T1的子树。
```java
package nowcoder.advanced.day01;

/**
 * 判断树 1 的某棵子树是否包含树 2
 *
 * @author GJXAIOU
 */
public class T1SubtreeEqualsT2 {

    public static class Node {
        public int value;
        public Node left;
        public Node right;

        public Node(int data) {
            this.value = data;
        }
    }

    public static boolean isSubtree(Node t1, Node t2) {
        String t1Str = serialByPre(t1);
        String t2Str = serialByPre(t2);
        return getIndexOf(t1Str, t2Str) != -1;
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

    // KMP
    public static int getIndexOf(String s, String m) {
        if (s == null || m == null || m.length() < 1 || s.length() < m.length()) {
            return -1;
        }
        char[] ss = s.toCharArray();
        char[] ms = m.toCharArray();
        int[] nextArr = getNextArray(ms);
        int index = 0;
        int mi = 0;
        while (index < ss.length && mi < ms.length) {
            if (ss[index] == ms[mi]) {
                index++;
                mi++;
            } else if (nextArr[mi] == -1) {
                index++;
            } else {
                mi = nextArr[mi];
            }
        }
        return mi == ms.length ? index - mi : -1;
    }

    public static int[] getNextArray(char[] ms) {
        if (ms.length == 1) {
            return new int[]{-1};
        }
        int[] nextArr = new int[ms.length];
        nextArr[0] = -1;
        nextArr[1] = 0;
        int pos = 2;
        int cn = 0;
        while (pos < nextArr.length) {
            if (ms[pos - 1] == ms[cn]) {
                nextArr[pos++] = ++cn;
            } else if (cn > 0) {
                cn = nextArr[cn];
            } else {
                nextArr[pos++] = 0;
            }
        }
        return nextArr;
    }

    public static void main(String[] args) {
        Node t1 = new Node(1);
        t1.left = new Node(2);
        t1.right = new Node(3);
        t1.left.left = new Node(4);
        t1.left.right = new Node(5);
        t1.right.left = new Node(6);
        t1.right.right = new Node(7);
        t1.left.left.right = new Node(8);
        t1.left.right.left = new Node(9);

        Node t2 = new Node(2);
        t2.left = new Node(4);
        t2.left.right = new Node(8);
        t2.right = new Node(5);
        t2.right.left = new Node(9);

        System.out.println(isSubtree(t1, t2));
    }
}

```


- 判断一个大字符串是否由一个小字符串重复得到，就是某个字符串是否为某个小字符串 * n 得到；

**思路**：转换为 一个前缀和后缀相差整数倍的问题



## 二、Manacher算法：

### 原始问题

Manacher算法是由题目“求字符串中最长回文子串的长度”而来。比如`abcdcb`的最长回文子串为`bcdcb`，其长度为5。

我们可以遍历字符串中的每个字符，当遍历到某个字符时就比较一下其左边相邻的字符和其右边相邻的字符是否相同，如果相同则继续比较其右边的右边和其左边的左边是否相同，如果相同则继续比较……，我们暂且称这个过程为向外“扩”。当“扩”不动时，经过的所有字符组成的子串就是以当前遍历字符为中心的最长回文子串。

我们每次遍历都能得到一个最长回文子串的长度，使用一个全局变量保存最大的那个，遍历完后就能得到此题的解。但分析这种方法的时间复杂度：当来到第一个字符时，只能扩其本身即1个；来到第二个字符时，最多扩两个；……；来到字符串中间那个字符时，最多扩`(n-1)/2+1`个；因此时间复杂度为`1+2+……+(n-1)/2+1`即`O(N^2)`。但Manacher算法却能做到`O(N)`。

Manacher算法中定义了如下几个概念：

- 回文半径：串中某个字符最多能向外扩的字符个数称为该字符的回文半径。比如`abcdcb`中字符`d`，能扩一个`c`，还能再扩一个`b`，再扩就到字符串右边界了，再算上字符本身，字符`d`的回文半径是3。
- 回文半径数组`pArr`：长度和字符串长度一样，保存串中每个字符的回文半径。比如`charArr="abcdcb"`，其中`charArr[0]='a'`一个都扩不了，但算上其本身有`pArr[0]=1`；而`charArr[3]='d'`最多扩2个，算上其本身有`pArr[3]=3`。
- 最右回文右边界`R`：遍历过程中，“扩”这一操作扩到的最右的字符的下标。比如`charArr=“abcdcb”`，当遍历到`a`时，只能扩`a`本身，向外扩不动，所以`R=0`；当遍历到`b`时，也只能扩`b`本身，所以更新`R=1`；但当遍历到`d`时，能向外扩两个字符到`charArr[5]=b`，所以`R`更新为5。
- 最右回文右边界对应的回文中心`C`：`C`与`R`是对应的、同时更新的。比如`abcdcb`遍历到`d`时，`R=5`，`C`就是`charArr[3]='d'`的下标`3`。

处理回文子串长度为偶数的问题：上面拿`abcdcb`来举例，其中`bcdcb`属于一个回文子串，但如果回文子串长度为偶数呢？像`cabbac`，按照上面定义的“扩”的逻辑岂不是每个字符的回文半径都是0，但事实上`cabbac`的最长回文子串的长度是6。因为我们上面“扩”的逻辑默认是将回文子串当做奇数长度的串来看的，因此我们在使用Manacher算法之前还需要将字符串处理一下，这里有一个小技巧，那就是将字符串的首尾和每个字符之间加上一个特殊符号，这样就能将输入的串统一转为奇数长度的串了。比如`abba`处理过后为`#a#b#b#a`，这样的话就有`charArr[4]='#'`的回文半径为4，也即原串的最大回文子串长度为4。相应代码如下：

```java
public static char[] manacherString(String str) {  
    char[] charArr = str.toCharArray();  
    char[] res = new char[str.length() * 2 + 1];  
    int index = 0;  
    for (int i = 0; i != res.length; i++) {  
        res[i] = (i & 1) == 0 ? '#' : charArr[index++];  
    }  
    return res;  
}
```

接下来分析，BFPRT算法是如何利用遍历过程中计算的`pArr`、`R`、`C`来为后续字符的回文半径的求解加速的。

首先，情况1是，遍历到的字符下标`cur`在`R`的右边（起初另`R=-1`），这种情况下该字符的最大回文半径`pArr[cur]`的求解无法加速，只能一步步向外扩来求解。



![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e8308cb40c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



情况2是，遍历到的字符下标`cur`在`R`的左边，这时`pArr[cur]`的求解过程可以利用之前遍历的字符回文半径信息来加速。分别做`cur`、`R`关于`C`的对称点`cur'`和`L`：

- 如果从`cur'`向外扩的最大范围的左边界没有超过`L`，那么`pArr[cur]=pArr[cur']`。

    

    ![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e83279de43?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

    

    证明如下：

    

    ![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e83231d1ba?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

    

    由于之前遍历过`cur'`位置上的字符，所以该位置上能扩的步数我们是有记录的（`pArr[cur']`），也就是说`cur'+pArr[cur']`处的字符`y'`是不等于`cur'-pArr[cur']`处的字符`x'`的。根据`R`和`C`的定义，整个`L`到`R`范围的字符是关于`C`对称的，也就是说`cur`能扩出的最大回文子串和`cur'`能扩出的最大回文子串相同，因此可以直接得出`pArr[cur]=pArr[cur']`。

- 如果从`cur'`向外扩的最大范围的左边界超过了`L`，那么`pArr[cur]=R-cur+1`。

    

    ![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e83297b708?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

    

    证明如下：

    

    ![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e8328b74da?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

    

    `R`右边一个字符`x`，`x`关于`cur`对称的字符`y`，`x,y`关于`C`对称的字符`x',y'`。根据`C,R`的定义有`x!=x'`；由于`x',y'`在以`cur'`为中心的回文子串内且关于`cur'`对称，所以有`x'=y'`，可推出`x!=y'`；又`y,y'`关于`C`对称，且在`L,R`内，所以有`y=y'`。综上所述，有`x!=y`，因此`cur`的回文半径为`R-cur+1`。

- 以`cur'`为中心向外扩的最大范围的左边界正好是`L`，那么`pArr[cur] >= （R-cur+1）`

    

    ![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e8329df8aa?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

    

    这种情况下，`cur'`能扩的范围是`cur'-L`，因此对应有`cur`能扩的范围是`R-cur`。但`cur`能否扩的更大则取决于`x`和`y`是否相等。而我们所能得到的前提条件只有`x!=x'`、`y=y'`、`x'!=y'`，无法推导出`x,y`的关系，只知道`cur`的回文半径最小为`R-cur+1`（算上其本身），需要继续尝试向外扩以求解`pArr[cur]`。

综上所述，`pArr[cur]`的计算有四种情况：暴力扩、等于`pArr[cur']`、等于`R-cur+1`、从`R-cur+1`继续向外扩。使用此算法求解原始问题的过程就是遍历串中的每个字符，每个字符都尝试向外扩到最大并更新`R`（只增不减），每次`R`增加的量就是此次能扩的字符个数，而`R`到达串尾时问题的解就能确定了，因此时间复杂度就是每次扩操作检查的次数总和，也就是`R`的变化范围（`-1~2N`，因为处理串时向串中添加了`N+1`个`#`字符），即`O(1+2N)=O(N)`。

整体代码如下：

```java
package nowcoder.advanced.day01;

/**
 * @Author GJXAIOU
 * @Date 2020/1/1 14:22
 */
public class Manacher {
    public static char[] manacherString(String str) {
        char[] charArr = str.toCharArray();
        char[] res = new char[str.length() * 2 + 1];
        int index = 0;
        for (int i = 0; i != res.length; i++) {
            res[i] = (i & 1) == 0 ? '#' : charArr[index++];
        }
        return res;
    }

    public static int maxLcpsLength(String str) {
        if (str == null || str.length() == 0) {
            return 0;
        }
        char[] charArr = manacherString(str);
        // 回文半径数组
        int[] pArr = new int[charArr.length];
        // index  为对称中心 C
        int index = -1;
        int pR = -1;
        int max = Integer.MIN_VALUE;
        for (int i = 0; i != charArr.length; i++) {
            // 2 * index - 1 就是对应 i' 位置，R> i,表示 i 在回文右边界里面，则最起码有一个不用验的区域，
            pArr[i] = pR > i ? Math.min(pArr[2 * index - i], pR - i) : 1;
            // 四种情况都让扩一下，其中 1 和 4 会成功，但是 2 ，3 会失败则回文右边界不改变；可以自己写成 if-else 问题。
            while (i + pArr[i] < charArr.length && i - pArr[i] > -1) {
                if (charArr[i + pArr[i]] == charArr[i - pArr[i]]) {
                    pArr[i]++;
                } else {
                    break;
                }
            }
            if (i + pArr[i] > pR) {
                pR = i + pArr[i];
                index = i;
            }
            max = Math.max(max, pArr[i]);
        }
        return max - 1;
    }

    public static void main(String[] args) {
        int length = maxLcpsLength("123abccbadbccba4w2");
        System.out.println(length);
    }
}

```

上述代码将四种情况的分支处理浓缩到了`7~14`行。其中第`7`行是确定加速信息：如果当前遍历字符在`R`右边，先算上其本身有`pArr[i]=1`，后面检查如果能扩再直接`pArr[i]++`即可；否则，当前字符的`pArr[i]`要么是`pArr[i']`（`i`关于`C`对称的下标`i'`的推导公式为`2*C-i`），要么是`R-i+1`，要么是`>=R-i+1`，可以先将`pArr[i]`的值置为这三种情况中最小的那一个，后面再检查如果能扩再直接`pArr[i]++`即可。

最后得到的`max`是处理之后的串（`length=2N+1`）的最长回文子串的半径，`max-1`刚好为原串中最长回文子串的长度。

### 进阶问题

给你一个字符串，要求添加尽可能少的字符使其成为一个回文字符串。

> 思路：当`R`第一次到达串尾时，做`R`关于`C`的对称点`L`，将`L`之前的字符串逆序就是结果。

```java
package nowcoder.advanced.day01;

/**
 * 添加尽可能少的字符使其成为一个回文字符串
 *
 * @author GJXAIOU
 */
public class ShortestEnd {

    public static char[] manacherString(String str) {
        char[] charArr = str.toCharArray();
        char[] res = new char[str.length() * 2 + 1];
        int index = 0;
        for (int i = 0; i != res.length; i++) {
            res[i] = (i & 1) == 0 ? '#' : charArr[index++];
        }
        return res;
    }

    public static String shortestEnd(String str) {
        if (str == null || str.length() == 0) {
            return null;
        }
        char[] charArr = manacherString(str);
        int[] pArr = new int[charArr.length];
        int index = -1;
        int pR = -1;
        int maxContainsEnd = -1;
        for (int i = 0; i != charArr.length; i++) {
            pArr[i] = pR > i ? Math.min(pArr[2 * index - i], pR - i) : 1;
            while (i + pArr[i] < charArr.length && i - pArr[i] > -1) {
                if (charArr[i + pArr[i]] == charArr[i - pArr[i]])
                    pArr[i]++;
                else {
                    break;
                }
            }
            if (i + pArr[i] > pR) {
                pR = i + pArr[i];
                index = i;
            }
            if (pR == charArr.length) {
                maxContainsEnd = pArr[i];
                break;
            }
        }
        char[] res = new char[str.length() - maxContainsEnd + 1];
        for (int i = 0; i < res.length; i++) {
            res[res.length - 1 - i] = charArr[i * 2 + 1];
        }
        return String.valueOf(res);
    }

    public static void main(String[] args) {
        String str2 = "abcd1233212";
        System.out.println(shortestEnd(str2));
    }
}

```

### （一）Manacher 算法应用

给定一个字符串 str，返回 str 中最长回文子串的长度。

常规思路： 

- 总是从中间那个字符出发，向两边开始扩展，观察回文，然后每个字符都得进行一遍

   但是奇数位数字完全可以  偶数位数字不可以，例如 1221 对应的每位扩充结果为：1 1 1 1 ，但是结果为 4.会出错

-  经过规律发现 无论是哪种情况 总是 `最大值/2` 即为最大回文串长度

前提是要处理字符串形式，加入特殊字符占位 #（其实什么字符都行）进行填充（在开头结尾和每个中间加入）

 这种方法是暴力法，时间复杂度为$$O(n^2)$$-----而Manacher算法可以降到O(n)

<img src="AlgorithmMediumDay01.resource/20180619213235664.png" alt="img" style="zoom:50%;" />

**概念汇总**

- 回文直径：从某个值开始往两边扩充，能扩充的最长值；

- 回文半径：见图

![image-20191231161002607](AlgorithmMediumDay01.resource/image-20191231161002607.png)

- 回文半径数组：还是从头到最后挨个求每个位置的回文半径，然后存入一个数组中

- 最右回文半径：如图所示，刚开始默认在 -1 位置，然后当到 0 位置，回文半径只能到 0 位置，所以最右的回文半径为 0  位置，当到 1 位置时候，回文半径扩充到 2 位置，则最右回文半径扩充到 2 位置，当移动到 3 位置时候，最右回文半径为 6

![image-20191231163310036](AlgorithmMediumDay01.resource/image-20191231163310036.png)

- 最右回文半径的中心：上面最后一个取得最右边界时候中心为 3，后面 4,5,6 的最右边界可能也是该位置，但是仅仅记录第一个使得最右回文边界到的中心，即 3。



### （二）Manacher 算法详解

针对 i 位置的回文串求法

- 如果 i 位置当前不在最右回文半径中，只能采用上面的方式暴力破：O(N )

    ![image-20191231164036523](AlgorithmMediumDay01.resource/image-20191231164036523.png)

- i 位置在回文右边界内：则设当前回文中心为 C，则 i 位置肯定是在回文中心 C 和回文右边界之间，做 i 位置关于中心 C 的对称点 i’，则针对 i’ 的所在的回文区域与整个回文区域关系有不同情况

    - i' 所在的回文区域在整个回文区域之间：O(1)

        ![image-20191231165149527](AlgorithmMediumDay01.resource/image-20191231165149527.png)

**则 i 回文长度大小和 i’ 回文长度相同**。证明：使用对称和区间内小回文，整个大区间大回文的结论可以证明 i 对应的区域是回文，并且是最大的。

- ​		i' 所在的回文区域在整个回文区域相交：O(1)

    ![image-20191231170010115](AlgorithmMediumDay01.resource/image-20191231170010115.png)

**则 i 回文长度为 i 到 R 长度**。

- i' 所在的回文区域与整个回文区域之间边界重叠

    ![image-20200101132409088](AlgorithmMediumDay01.resource/image-20200101132409088.png)

这种情况下，只能保证 i 到 R  区间的数据不需要进行暴力破，相当于可以从 R + 1 位置开始判断。



代码：

```java
package nowcoder.advanced.day01;

/**
 * @Author GJXAIOU
 * @Date 2020/1/1 14:22
 */
public class Manacher {
    public static char[] manacherString(String str) {
        char[] charArr = str.toCharArray();
        char[] res = new char[str.length() * 2 + 1];
        int index = 0;
        for (int i = 0; i != res.length; i++) {
            res[i] = (i & 1) == 0 ? '#' : charArr[index++];
        }
        return res;
    }

    public static int maxLcpsLength(String str) {
        if (str == null || str.length() == 0) {
            return 0;
        }
        char[] charArr = manacherString(str);
        // 回文半径数组
        int[] pArr = new int[charArr.length];
        // index  为对称中心 C
        int index = -1;
        int pR = -1;
        int max = Integer.MIN_VALUE;
        for (int i = 0; i != charArr.length; i++) {
            // 2 * index - 1 就是对应 i' 位置，R> i,表示 i 在回文右边界里面，则最起码有一个不用验的区域，
            pArr[i] = pR > i ? Math.min(pArr[2 * index - i], pR - i) : 1;
            // 四种情况都让扩一下，其中 1 和 4 会成功，但是 2 ，3 会失败则回文右边界不改变；可以自己写成 if-else 问题。
            while (i + pArr[i] < charArr.length && i - pArr[i] > -1) {
                if (charArr[i + pArr[i]] == charArr[i - pArr[i]]) {
                    pArr[i]++;
                } else {
                    break;
                }
            }
            if (i + pArr[i] > pR) {
                pR = i + pArr[i];
                index = i;
            }
            max = Math.max(max, pArr[i]);
        }
        return max - 1;
    }

    public static void main(String[] args) {
        int length = maxLcpsLength("123abccbadbccba4w2");
        System.out.println(length);
    }
}

```



### （三）题目拓展

一个字符串，只能向后面添加字符，怎么将整个串都变成回文串，要求添加字符最短。

示例： ‘abc12321’  应添加  ‘cba’  变成  ’abc12321cba‘

即求，在包含最后一个字符情况下，最长回文串多少，前面不是的部分，逆序添加即可。

用manacher求得回文边界，发现边界与字符串最后位置，停止，求得回文中心C与有边界R，找到回文字符串，用原字符串减去该部分，在逆序就是结果。

```java
package nowcoder.advanced.day01;

/**
 * 添加尽可能少的字符使其成为一个回文字符串
 *
 * @author GJXAIOU
 * @Date 2020/1/1 14:11
 */

public class ShortestEnd {

    public static char[] manacherString(String str) {
        char[] charArr = str.toCharArray();
        char[] res = new char[str.length() * 2 + 1];
        int index = 0;
        for (int i = 0; i != res.length; i++) {
            res[i] = (i & 1) == 0 ? '#' : charArr[index++];
        }
        return res;
    }

    public static String shortestEnd(String str) {
        if (str == null || str.length() == 0) {
            return null;
        }
        char[] charArr = manacherString(str);
        int[] pArr = new int[charArr.length];
        int index = -1;
        int pR = -1;
        int maxContainsEnd = -1;
        for (int i = 0; i != charArr.length; i++) {
            pArr[i] = pR > i ? Math.min(pArr[2 * index - i], pR - i) : 1;
            while (i + pArr[i] < charArr.length && i - pArr[i] > -1) {
                if (charArr[i + pArr[i]] == charArr[i - pArr[i]])
                    pArr[i]++;
                else {
                    break;
                }
            }
            if (i + pArr[i] > pR) {
                pR = i + pArr[i];
                index = i;
            }
            if (pR == charArr.length) {
                maxContainsEnd = pArr[i];
                break;
            }
        }
        char[] res = new char[str.length() - maxContainsEnd + 1];
        for (int i = 0; i < res.length; i++) {
            res[res.length - 1 - i] = charArr[i * 2 + 1];
        }
        return String.valueOf(res);
    }

    public static void main(String[] args) {
        String str2 = "abcd1233212";
        System.out.println(shortestEnd(str2));
    }
}

```



## 三、BFPRT算法：

题目：给你一个整型数组，返回其中第K小的数。

这道题可以利用荷兰国旗改进的`partition`和随机快排的思想：随机选出一个数，将数组以该数作比较划分为`<,=,>`三个部分，则`=`部分的数是数组中第几小的数不难得知，接着对`<`（如果第K小的数在`<`部分）或`>`（如果第K小的数在`>`部分）部分的数递归该过程，直到`=`部分的数正好是整个数组中第K小的数。这种做法不难求得时间复杂度的数学期望为`O(NlogN)`（以2为底）。但这毕竟是数学期望，在实际工程中的表现可能会有偏差，而BFPRT算法能够做到时间复杂度就是`O(NlogN)`。

BFPRT算法首先将数组按5个元素一组划分成`N/5`个小部分（最后不足5个元素自成一个部分），再这些小部分的内部进行排序，然后将每个小部分的中位数取出来再排序得到中位数：



![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e84ccac4b5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



BFPRT求解此题的步骤和开头所说的步骤大体类似，但是“随机选出一个的作为比较的那个数”这一步替换为上图所示最终选出来的那个数。

`O(NlogN)`的证明，为什么每一轮`partition`中的随机选数改为BFPRT定义的选数逻辑之后，此题的时间复杂度就彻底变为`O(NlogN)`了呢？下面分析一下这个算法的步骤：

BFPRT算法，接收一个数组和一个K值，返回数组中的一个数

1. 数组被划分为了`N/5`个小部分，每个部分的5个数排序需要`O(1)`，所有部分排完需要`O(N/5)=O(N)`
2. 取出每个小部分的中位数，一共有`N/5`个，递归调用BFPRT算法得到这些数中第`(N/5)/2`小的数（即这些数的中位数），记为`pivot`
3. 以`pivot`作为比较，将整个数组划分为`pivot`三个区域
4. 判断第K小的数在哪个区域，如果在`=`区域则直接返回`pivot`，如果在`<`或`>`区域，则将这个区域的数递归调用BFPRT算法
5. `base case`：在某次递归调用BFPRT算法时发现这个区域只有一个数，那么这个数就是我们要找的数。

代码示例：

```
public static int getMinKthNum(int[] arr, int K) {
  if (arr == null || K > arr.length) {
    return Integer.MIN_VALUE;
  }
  int[] copyArr = Arrays.copyOf(arr, arr.length);
  return bfprt(copyArr, 0, arr.length - 1, K - 1);
}

public static int bfprt(int[] arr, int begin, int end, int i) {
  if (begin == end) {
    return arr[begin];
  }
  int pivot = medianOfMedians(arr, begin, end);
  int[] pivotRange = partition(arr, begin, end, pivot);
  if (i >= pivotRange[0] && i <= pivotRange[1]) {
    return arr[i];
  } else if (i < pivotRange[0]) {
    return bfprt(arr, begin, pivotRange[0] - 1, i);
  } else {
    return bfprt(arr, pivotRange[1] + 1, end, i);
  }
}

public static int medianOfMedians(int[] arr, int begin, int end) {
  int num = end - begin + 1;
  int offset = num % 5 == 0 ? 0 : 1;
  int[] medians = new int[num / 5 + offset];
  for (int i = 0; i < medians.length; i++) {
    int beginI = begin + i * 5;
    int endI = beginI + 4;
    medians[i] = getMedian(arr, beginI, Math.min(endI, end));
  }
  return bfprt(medians, 0, medians.length - 1, medians.length / 2);
}

public static int getMedian(int[] arr, int begin, int end) {
  insertionSort(arr, begin, end);
  int sum = end + begin;
  int mid = (sum / 2) + (sum % 2);
  return arr[mid];
}

public static void insertionSort(int[] arr, int begin, int end) {
  if (begin >= end) {
    return;
  }
  for (int i = begin + 1; i <= end; i++) {
    for (int j = i; j > begin; j--) {
      if (arr[j] < arr[j - 1]) {
        swap(arr, j, j - 1);
      } else {
        break;
      }
    }
  }
}

public static int[] partition(int[] arr, int begin, int end, int pivot) {
  int L = begin - 1;
  int R = end + 1;
  int cur = begin;
  while (cur != R) {
    if (arr[cur] > pivot) {
      swap(arr, cur, --R);
    } else if (arr[cur] < pivot) {
      swap(arr, cur++, ++L);
    } else {
      cur++;
    }
  }
  return new int[]{L + 1, R - 1};
}

public static void swap(int[] arr, int i, int j) {
  int tmp = arr[i];
  arr[i] = arr[j];
  arr[j] = tmp;
}

public static void main(String[] args) {
  int[] arr = {6, 9, 1, 3, 1, 2, 2, 5, 6, 1, 3, 5, 9, 7, 2, 5, 6, 1, 9};
  System.out.println(getMinKthNum(arr,13));
}
复制代码
```

时间复杂度为`O(NlogN)`（底数为2）的证明，分析`bfprt`的执行步骤（假设`bfprt`的时间复杂度为`T(N)`）：

1. 首先数组5个5个一小组并内部排序，对5个数排序为`O(1)`，所有小组排好序为`O(N/5)=O(N)`
2. 由步骤1的每个小组抽出中位数组成一个中位数小组，共有`N/5`个数，递归调用`bfprt`求出这`N/5`个数中第`(N/5)/2`小的数（即中位数）为`T(N/5)`，记为`pivot`
3. 对步骤2求出的`pivot`作为比较将数组分为小于、等于、大于三个区域，由于`pivot`是中位数小组中的中位数，所以中位数小组中有`N/5/2=N/10`个数比`pivot`小，这`N/10`个数分别又是步骤1中某小组的中位数，可推导出至少有`3N/10`个数比`pivot`小，也即最多有`7N/10`个数比`pivot`大。也就是说，大于区域（或小于）最大包含`7N/10`个数、最少包含`3N/10`个数，那么如果第`i`大的数不在等于区域时，无论是递归`bfprt`处理小于区域还是大于区域，最坏情况下子过程的规模最大为`7N/10`，即`T(7N/10)`

综上所述，`bfprt`的`T(N)`存在推导公式：`T(N/5)+T(7N/10)+O(N)`。根据 **基础篇** 中所介绍的Master公式可以求得`bfprt`的时间复杂度就是`O(NlogN)`（以2为底）。

## morris遍历二叉树

> 关于二叉树先序、中序、后序遍历的递归和非递归版本在【直通BAT算法（基础篇）】中有讲到，但这6种遍历算法的时间复杂度都需要`O(H)`（其中`H`为树高）的额外空间复杂度，因为二叉树遍历过程中只能向下查找孩子节点而无法回溯父结点，因此这些算法借助栈来保存要回溯的父节点（递归的实质是系统帮我们压栈），并且栈要保证至少能容纳下`H`个元素（比如遍历到叶子结点时回溯父节点，要保证其所有父节点在栈中）。而morris遍历则能做到时间复杂度仍为`O(N)`的情况下额外空间复杂度只需`O(1)`。

### 遍历规则

首先在介绍morris遍历之前，我们先把先序、中序、后序定义的规则抛之脑后，比如先序遍历在拿到一棵树之后先遍历头结点然后是左子树最后是右子树，并且在遍历过程中对于子树的遍历仍是这样。

忘掉这些遍历规则之后，我们来看一下morris遍历定义的标准：

1. 定义一个遍历指针`cur`，该指针首先指向头结点

2. 判断

    ```
    cur
    ```

    的左子树是否存在

    - 如果`cur`的左孩子为空，说明`cur`的左子树不存在，那么`cur`右移来到`cur.right`

    - 如果

        ```
        cur
        ```

        的左孩子不为空，说明

        ```
        cur
        ```

        的左子树存在，找出该左子树的最右结点，记为

        ```
        mostRight
        ```

        - 如果，`mostRight`的右孩子为空，那就让其指向`cur`（`mostRight.right=cur`），并左移`cur`（`cur=cur.left`）
        - 如果`mostRight`的右孩子不空，那么让`cur`右移（`cur=cur.right`），并将`mostRight`的右孩子置空

3. 经过步骤2之后，如果`cur`不为空，那么继续对`cur`进行步骤2，否则遍历结束。

下图所示举例演示morris遍历的整个过程：



![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e86eb7cac7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



### 先序、中序序列

遍历完成后对`cur`进过的节点序列稍作处理就很容易得到该二叉树的先序、中序序列：



![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e8a7c5bf81?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



示例代码：

```
public static class Node {
    int data;
    Node left;
    Node right;
    public Node(int data) {
        this.data = data;
    }
}

public static void preOrderByMorris(Node root) {
    if (root == null) {
        return;
    }
    Node cur = root;
    while (cur != null) {
        if (cur.left == null) {
            System.out.print(cur.data+" ");
            cur = cur.right;
        } else {
            Node mostRight = cur.left;
            while (mostRight.right != null && mostRight.right != cur) {
                mostRight = mostRight.right;
            }
            if (mostRight.right == null) {
                System.out.print(cur.data+" ");
                mostRight.right = cur;
                cur = cur.left;
            } else {
                cur = cur.right;
                mostRight.right = null;
            }
        }
    }
    System.out.println();
}

public static void mediumOrderByMorris(Node root) {
    if (root == null) {
        return;
    }
    Node cur = root;
    while (cur != null) {
        if (cur.left == null) {
            System.out.print(cur.data+" ");
            cur = cur.right;
        } else {
            Node mostRight = cur.left;
            while (mostRight.right != null && mostRight.right != cur) {
                mostRight = mostRight.right;
            }
            if (mostRight.right == null) {
                mostRight.right = cur;
                cur = cur.left;
            } else {
                System.out.print(cur.data+" ");
                cur = cur.right;
                mostRight.right = null;
            }
        }
    }
    System.out.println();
}

public static void main(String[] args) {
    Node root = new Node(1);
    root.left = new Node(2);
    root.right = new Node(3);
    root.left.left = new Node(4);
    root.left.right = new Node(5);
    root.right.left = new Node(6);
    root.right.right = new Node(7);
    preOrderByMorris(root);
    mediumOrderByMorris(root);

}
复制代码
```

这里值得注意的是：**morris遍历会来到一个左孩子不为空的结点两次**，而其它结点只会经过一次。因此使用morris遍历打印先序序列时，如果来到的结点无左孩子，那么直接打印即可（这种结点只会经过一次），否则如果来到的结点的左子树的最右结点的右孩子为空才打印（这是第一次来到该结点的时机），这样也就忽略了`cur`经过的结点序列中第二次出现的结点；而使用morris遍历打印中序序列时，如果来到的结点无左孩子，那么直接打印（这种结点只会经过一次，左中右，没了左，直接打印中），否则如果来到的结点的左子树的最右结点不为空时才打印（这是第二次来到该结点的时机），这样也就忽略了`cur`经过的结点序列中第一次出现的重复结点。

### 后序序列

使用morris遍历得到二叉树的后序序列就没那么容易了，因为对于树种的非叶结点，morris遍历最多会经过它两次，而我们后序遍历实在第三次来到该结点时打印该结点的。因此要想得到后序序列，仅仅改变在morris遍历时打印结点的时机是无法做到的。

但其实，在morris遍历过程中，如果在每次遇到第二次经过的结点时，将该结点的左子树的右边界上的结点从下到上打印，最后再将整个树的右边界从下到上打印，最终就是这个数的后序序列：



![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e8ace72a83?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)





![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e8ad102b9d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)





![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e8ae36d836?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)





![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e8c2a5f049?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



其中无非就是在morris遍历中在第二次经过的结点的时机执行一下打印操作。而从下到上打印一棵树的右边界，可以将该右边界上的结点看做以`right`指针为后继指针的链表，将其反转`reverse`然后打印，最后恢复成原始结构即可。示例代码如下（其中容易犯错的地方是`18`行和`19`行的代码不能调换）：

```
public static void posOrderByMorris(Node root) {
    if (root == null) {
        return;
    }
    Node cur = root;
    while (cur != null) {
        if (cur.left == null) {
            cur = cur.right;
        } else {
            Node mostRight = cur.left;
            while (mostRight.right != null && mostRight.right != cur) {
                mostRight = mostRight.right;
            }
            if (mostRight.right == null) {
                mostRight.right = cur;
                cur = cur.left;
            } else {
                mostRight.right = null;
                printRightEdge(cur.left);
                cur = cur.right;
            }
        }
    }
    printRightEdge(root);
}

private static void printRightEdge(Node root) {
    if (root == null) {
        return;
    }
    //reverse the right edge
    Node cur = root;
    Node pre = null;
    while (cur != null) {
        Node next = cur.right;
        cur.right = pre;
        pre = cur;
        cur = next;
    }
    //print 
    cur = pre;
    while (cur != null) {
        System.out.print(cur.data + " ");
        cur = cur.right;
    }
    //recover
    cur = pre;
    pre = null;
    while (cur != null) {
        Node next = cur.right;
        cur.right = pre;
        pre = cur;
        cur = next;
    }
}

public static void main(String[] args) {
    Node root = new Node(1);
    root.left = new Node(2);
    root.right = new Node(3);
    root.left.left = new Node(4);
    root.left.right = new Node(5);
    root.right.left = new Node(6);
    root.right.right = new Node(7);
    posOrderByMorris(root);
}
复制代码
```

### 时间复杂度分析

因为morris遍历中，只有左孩子非空的结点才会经过两次而其它结点只会经过一次，也就是说遍历的次数小于`2N`，因此使用morris遍历得到先序、中序序列的时间复杂度自然也是`O(1)`；但产生后序序列的时间复杂度还要算上`printRightEdge`的时间复杂度，但是你会发现整个遍历的过程中，所有的`printRightEdge`加起来也只是遍历并打印了`N`个结点：



![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e8c715ba38?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



因此时间复杂度仍然为`O(N)`。

> morris遍历结点的顺序不是先序、中序、后序，而是按照自己的一套标准来决定接下来要遍历哪个结点。
>
> morris遍历的独特之处就是充分利用了叶子结点的无效引用（引用指向的是空，但该引用变量仍然占内存），从而实现了`O(1)`的时间复杂度。



------------------

### （一）BFPRT 算法详解与应用：

 **这个算法其实就是用来解决 ： 求解最小/最大的 k 个数问题，不是第 K 个数。**

- 最暴力的方法是：遍历然后查找：$$O(N*log^N)$$

- 常规的做法 就是 平均时间复杂度为 O(n) 的 partition 函数的应用，划分结果为小于某个数的放左边，等于某个数的放中间，大于某个数的放右边，然后看等于的左右边界，例如第一次划分结果左右边界为：500 ~ 600 位，查找的是第 300 小的数，则将小于区域按照上面方式再次使用 partition 函数进行划分，如果等于区域命中就结束过程，依次类推。  
    - 缺点：是概率式的，无法避免最差的情况（因为每次选取的划分值是随机的，很难保证大于和小于该值的区间长度差不多）。因此最终期望为：$$O(N)$$

- BFPRT算法优点是确定性的，严格的 $$O(N)$$。



大概思路：

 将整个数组以5个数字一组划分 ,最后不满 5 个的单独成组，分别进行组内排序（组间不排序） ，因为只是 5 个数排序，所有时间复杂度为O(1)， 同时因为整个数组分为N/5组，所以则总共时间复杂度为O(N)。

3) 把每个组内的中位数拿出来构成一个新数组，若为偶数则拿其上中位数即可，数组长度为 N/5

4）递归调用该bfprt函数，传入刚才的中位数数组 和 中位数数组长度的一半 （继续求中位数）【BFPRT 函数格式为： int BFPRT（arr,k) 输入数组 arr 和要求的第 k 小的值】

5）将4）返回的值作为原始数组划分值，一直以该值作为 paritition 函数的划分值。



例子：

![BFPRT 示例](AlgorithmMediumDay01.resource/BFPRT%20%E7%A4%BA%E4%BE%8B.png)

每一步的时间复杂度为：假设原始数据量为 N，则

- 步骤一：O(1)

- 步骤二：O(N)

- 步骤三：O(N)

- 步骤四：因为是递归：T（N/5），最终获取划分值 P

- 步骤五：在 partition 算法中如果不是值正好在相等区域的情况下，则不是在小于就是在大于区间上，同时两个区间后续只有一个需要再次判断，另一个区间直接舍弃。

    判断最差的情况：左侧最差情况，就是找到最多能有多少数是小于划分值 P（即）最少多少数的确定大于划分值 P 的，所以至少共 3N/10 大于划分值。所以左边比它小的最多有7N/10。右边类似。
    
    所以最终该步骤左右时间复杂度为：$$T(\frac{7N}{10})$$ ，因为只算一个，所以最终仍然是：$$T(\frac{7N}{10})$$ 。
    
    所以整体时间复杂度为：$$T(N) = T(\frac{N}{5}) + T(\frac{7N}{10}) + O(N)$$，结果为 O(N)，证明见算法导论。



优点 :我们选择的基准一定将数组划分为 一定规模的子部分

![img](AlgorithmMediumDay01.resource/20180620233656220.png)

由上图我们可以看到： 最中间的中位数--- 右边比它大的**至少**有N/10 * 3 个（每个组又有两个大于该组中位数的数字）

​                                右边比它小的最多有N/10 * 7

​                             ---- 同理左边也是~

如下例子：  7为中位数 ----比它大的至少有 8 9 12 13 14 这几个数

![img](AlgorithmMediumDay01.resource/20180620233956888.png)







重点掌握思路 可以面试的时候告诉面试官~~



**代码：** ==还是有问题==

```java
package nowcoder.advanced.class01;

import java.sql.SQLOutput;

/**
 * @Author GJXAIOU
 * @Date 2020/1/1 16:13
 */
public class BFPRT {
    // 使用堆的方式来求解该题，时间复杂度为：O(N * logN)
    public static int[] getMinKNumsByHeap(int[] arr, int k) {
        if (k < 1 || k > arr.length) {
            return arr;
        }
        int[] kHeap = new int[k];
        for (int i = 0; i != k; i++) {
            heapInsert(kHeap, arr[i], i);
        }
        for (int i = k; i != arr.length; i++) {
            if (arr[i] < kHeap[0]) {
                kHeap[0] = arr[i];
                heapify(kHeap, 0, k);
            }
        }
        return kHeap;
    }

    public static void heapInsert(int[] arr, int value, int index) {
        arr[index] = value;
        while (index != 0) {
            int parent = (index - 1) / 2;
            if (arr[parent] < arr[index]) {
                swap(arr, parent, index);
                index = parent;
            } else {
                break;
            }
        }
    }

    public static void heapify(int[] arr, int index, int heapSize) {
        int left = index * 2 + 1;
        int right = index * 2 + 2;
        int largest = index;
        while (left < heapSize) {
            if (arr[left] > arr[index]) {
                largest = left;
            }
            if (right < heapSize && arr[right] > arr[largest]) {
                largest = right;
            }
            if (largest != index) {
                swap(arr, largest, index);
            } else {
                break;
            }
            index = largest;
            left = index * 2 + 1;
            right = index * 2 + 2;
        }
    }

    // ------------------分割线---------------------

    public static int[] getMinKNumsByBFPRT(int[] arr, int k) {
        if (k < 1 || k > arr.length) {
            return arr;
        }
        int minKth = getMinKthByBFPRT(arr, k);
        int[] res = new int[k];
        int index = 0;
        for (int i = 0; i != arr.length; i++) {
            if (arr[i] < minKth) {
                res[index++] = arr[i];
            }
        }
        for (; index != res.length; index++) {
            res[index] = minKth;
        }
        return res;
    }

    public static int getMinKthByBFPRT(int[] arr, int K) {
        int[] copyArr = copyArray(arr);
        return bfprt(copyArr, 0, copyArr.length - 1, K - 1);
    }

    public static int[] copyArray(int[] arr) {
        int[] res = new int[arr.length];
        for (int i = 0; i != res.length; i++) {
            res[i] = arr[i];
        }
        return res;
    }

    // 在 begin 到 end 范围内求第 i 小的数
    public static int bfprt(int[] arr, int begin, int end, int i) {
        if (begin == end) {
            return arr[begin];
        }
        // 划分值
        int pivot = medianOfMedians(arr, begin, end);
        int[] pivotRange = partition(arr, begin, end, pivot);
        if (i >= pivotRange[0] && i <= pivotRange[1]) {
            return arr[i];
        } else if (i < pivotRange[0]) {
            return bfprt(arr, begin, pivotRange[0] - 1, i);
        } else {
            return bfprt(arr, pivotRange[1] + 1, end, i);
        }
    }

    public static int medianOfMedians(int[] arr, int begin, int end) {
        int num = end - begin + 1;
        int offset = num % 5 == 0 ? 0 : 1;
        int[] mArr = new int[num / 5 + offset];
        for (int i = 0; i < mArr.length; i++) {
            int beginI = begin + i * 5;
            int endI = beginI + 4;
            mArr[i] = getMedian(arr, beginI, Math.min(end, endI));
        }
        return bfprt(mArr, 0, mArr.length - 1, mArr.length / 2);
    }

    public static int[] partition(int[] arr, int begin, int end, int pivotValue) {
        int small = begin - 1;
        int cur = begin;
        int big = end + 1;
        while (cur != big) {
            if (arr[cur] < pivotValue) {
                swap(arr, ++small, cur++);
            } else if (arr[cur] > pivotValue) {
                swap(arr, cur, --big);
            } else {
                cur++;
            }
        }
        int[] range = new int[2];
        // 等于区域最左边
        range[0] = small + 1;
        // 等于区域最右边
        range[1] = big - 1;
        return range;
    }

    public static int getMedian(int[] arr, int begin, int end) {
        insertionSort(arr, begin, end);
        int sum = end + begin;
        int mid = (sum / 2) + (sum % 2);
        return arr[mid];
    }

    public static void insertionSort(int[] arr, int begin, int end) {
        for (int i = begin + 1; i != end + 1; i++) {
            for (int j = i; j != begin; j--) {
                if (arr[j - 1] > arr[j]) {
                    swap(arr, j - 1, j);
                } else {
                    break;
                }
            }
        }
    }

    public static void swap(int[] arr, int begin, int end) {
        arr[begin] = arr[begin] ^ arr[end];
        arr[end] = arr[begin] ^ arr[end];
        arr[begin] = arr[begin] ^ arr[end];
    }
    public static void printArray(int[] array){
        for (int i = 0; i < array.length; i++) {
            System.out.print(array[i] + " ");
        }
        System.out.println();
    }

    public static void main(String[] args) {
        int[] demo ={6,9,1,3,1,2,2,5,6,1,3,5,9,7,2,5,6,1,9};
        printArray(getMinKNumsByHeap(demo, 10));
        printArray(getMinKNumsByBFPRT(demo, 10));
    }
}

```

注意： 此时的partition和常规的partition函数不太一样 多了一个我们定义的基准划分值

​      且以此基准值划分 比它小的在左边 比它大的在右边 和它相等的在中间并将相等的左右边界存放在一个数组中









