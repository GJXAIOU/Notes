## AlgorithmEasyDay08

[TOC]

## 一、暴力递归

- 把问题转化为规模缩小了的同类问题的子问题；
- 有明确的不需要继续进行递归的条件（base case），因为不能一直递归；
- 有当得到了子问题的结果之后的决策过程；
- 不记录每一个子问题的解；



### （一）问题一：求 n!

求 $$n!$$ 的结果

**解析**：

该问题可以分解为子问题：$$n * (n - 1)!$$，同样可以一层层分解，逐层进行依赖，**但是其实每一层都是不知道具体的计算方法**；

**一般方法**：知道这个问题的计算方式： $$ 1 * 2 * 3 * 4 * ... * n$$，这种方式就是上面计算方式的逆过程。

**代码**

```java
package com.gjxaiou.easy.class_08;

public class Code_01_Factorial {
    // 递归版本
    public static long getFactorial1(int n) {
        if (n == 1) {
            return 1L;
        }
        return (long) n * getFactorial1(n - 1);
    }

    // 非递归版本
    public static long getFactorial2(int n) {
        long result = 1L;
        for (int i = 1; i <= n; i++) {
            result *= i;
        }
        return result;
    }

    public static void main(String[] args) {
        int n = 5;
        System.out.println(getFactorial1(n));
        System.out.println(getFactorial2(n));
    }
}

```

程序输出结果：

```java
120
120
```



### （二）汉诺塔问题

给定一个数n，表示n层汉诺塔问题，请打印最优步数的所有过程
进阶：给定一个汉诺塔的状况用数组arr表示（arr中只有1，2，3三种数字），请返回这是汉诺塔最优步数的第几步？
举例：
arr = {3,2,1}
arr长度为3，表示这是一个3层汉诺塔问题；
arr[0] == 3表示上面的汉诺塔在右边；
arr[1] == 2表示中间的汉诺塔在中间；
arr[2] == 1表示底下的汉诺塔在左间；
这种状况是3层汉诺塔最优步数的第2步，所以返回2。

**该问题的递归不能改为动态规划**，因为其每一步都需要打印结果。

**问题**：打印 N 层汉诺塔从最左边移动到最右边的全部过程

**汉诺塔原则**：只能小的压大的，不能大的压小的；

![汉诺塔示例](AlgorithmEasyDay08.resource/%E6%B1%89%E8%AF%BA%E5%A1%94%E7%A4%BA%E4%BE%8B.png)

步骤一：将 1 ~ n - 1 移动到 help 

步骤二：见 n 移动到 to

步骤单：将 1 ~ n -1 移动到 to

**一共需要的步骤**： $$2^n - 1$$



代码：

```java
package com.gjxaiou.easy.class_08;

public class Code_02_Hanoi {

    public static void hanoi(int n) {
        if (n > 0) {
            func(n, n, "left", "mid", "right");
        }
    }

    public static void func(int rest, int down, String from, String help, String to) {
        if (rest == 1) {
            System.out.println("move " + down + " from " + from + " to " + to);
        } else {
            func(rest - 1, down - 1, from, to, help);
            func(1, down, from, help, to);
            func(rest - 1, down - 1, help, from, to);
        }
    }

    // 递归版本，只有这一个函数
    // N 表示现在为 1~N 问题，同时 N 个都是停留在 From 上面
    public static void process(int N, String from, String to, String help) {
        if (N == 1) {
            System.out.println("move 1 from " + from + " to " + to);
        } else {
            // 从 from 做到 help 上，可以借助 to
            process(N - 1, from, help, to);
            // 挪动 N
            System.out.println("move " + N + " from " + from + " to " + to);
            // 挪回来
            process(N - 1, help, to, from);
        }
    }

    public static void moveLeftToRight(int N) {
        if (N == 1) {
            System.out.println("move 1 from left to right");
        }
        moveLeftToMid(N - 1);
        System.out.println("move " + N + "from left to right");
        moveMidToRight(N - 1);
    }

    public static void moveRightToLeft(int N) {

    }

    public static void moveLeftToMid(int N) {
        if (N == 1) {
            System.out.println("move 1 from left to mid");
        }
        moveLeftToRight(N - 1);
        System.out.println("move " + N + "from left to mid");
        moveRightToMid(N - 1);
    }

    public static void moveMidToLeft(int N) {

    }

    public static void moveRightToMid(int N) {

    }

    public static void moveMidToRight(int N) {
        if (N == 1) {
            System.out.println("move 1 from mid to right");
        }
        moveMidToLeft(N - 1);
        System.out.println("move " + N + "from mid to right");
        moveLeftToRight(N - 1);
    }

    public static void main(String[] args) {
        int n = 3;
        hanoi(n);

        process(3, "左", "右", "中");
    }
}

```

程序运行结果：

```java
move 1 from left to right
move 2 from left to mid
move 1 from right to mid
move 3 from left to right
move 1 from mid to left
move 2 from mid to right
move 1 from left to right
move 1 from 左 to 右
move 2 from 左 to 中
move 1 from 右 to 中
move 3 from 左 to 右
move 1 from 中 to 左
move 2 from 中 to 右
move 1 from 左 to 右
```



### （三）输出全部子序列

**问题**：打印一个字符串的全部子序列，包括空字符串，**注意**：不是子串。

![子序列示例](AlgorithmEasyDay08.resource/%E5%AD%90%E5%BA%8F%E5%88%97%E7%A4%BA%E4%BE%8B.png)

代码：

```java
package com.gjxaiou.easy.class_08;

import java.util.ArrayList;
import java.util.List;

public class Code_03_Print_All_Subsquences {

	public static void printAllSubsquence(String str) {
		char[] chs = str.toCharArray();
		process(chs, 0);
	}

	public static void process(char[] chs, int i) {
		if (i == chs.length) {
			System.out.println(String.valueOf(chs));
			return;
		}
		process(chs, i + 1);
		char tmp = chs[i];
		chs[i] = 0;
		process(chs, i + 1);
		chs[i] = tmp;
	}
	
	public static void function(String str) {
		char[] chs = str.toCharArray();
		process(chs, 0, new ArrayList<Character>());
	}
	
	public static void process(char[] chs, int i, List<Character> res) {
		if(i == chs.length) {
			printList(res);
		}
		List<Character> resKeep = copyList(res);
		resKeep.add(chs[i]);
		process(chs, i+1, resKeep);
		List<Character> resNoInclude = copyList(res);
		process(chs, i+1, resNoInclude);
	}
	
	public static void printList(List<Character> res) {
		// ...;
	}
	
	public static List<Character> copyList(List<Character> list){
		return null;
	}

	// 递归版本

	/**
	 *
	 * @param str
	 * @param i
	 * @param res 上一级决策之后的字符串值
	 */
	public static void printAllSub(char[] str, int i, String res){
		// 如果到最后一个元素就不要递归了
		if(i == str.length){
			System.out.println(res);
			return;
		}
		// 有两种决策，一种当前为空，就是将上一步决策接着往下扔，另一种就是加上当前字符串然后往下扔
		printAllSub(str, i + 1, res);
		printAllSub(str, i + 1, res + String.valueOf(str[i]));
	}

	public static void main(String[] args) {
		String test = "abc";
		printAllSubsquence(test);
		System.out.println("----------------");
		// 测试递归版本
		printAllSub(test.toCharArray(),0, "");
		System.out.println("----------------------");
	}
}

```

程序运行结果

```java
abc
ab 
a c
a  
 bc
 b 
  c
   
----------------

c
b
bc
a
ac
ab
abc
----------------------
```



### （四）全排列

**问题**：打印一个字符串的全部排列

**补充问题**

**问题**：打印一个字符串的全部排列，要求不要出现重复的排列

代码：

```java
package com.gjxaiou.easy.class_08;

import java.util.HashSet;

public class Code_04_Print_All_Permutations {

    public static void printAllPermutations1(String str) {
        char[] chs = str.toCharArray();
        process1(chs, 0);
    }

    public static void process1(char[] chs, int i) {
        if (i == chs.length) {
            System.out.println(String.valueOf(chs));
        }
        for (int j = i; j < chs.length; j++) {
            swap(chs, i, j);
            process1(chs, i + 1);
            //swap(chs, i, j);
        }
    }

    public static void printAllPermutations2(String str) {
        char[] chs = str.toCharArray();
        process2(chs, 0);
    }

    public static void process2(char[] chs, int i) {
        if (i == chs.length) {
            System.out.println(String.valueOf(chs));
        }
        HashSet<Character> set = new HashSet<>();
        for (int j = i; j < chs.length; j++) {
            if (!set.contains(chs[j])) {
                set.add(chs[j]);
                swap(chs, i, j);
                process2(chs, i + 1);
                //swap(chs, i, j);
            }
        }
    }

    public static void swap(char[] chs, int i, int j) {
        char tmp = chs[i];
        chs[i] = chs[j];
        chs[j] = tmp;
    }

    public static void main(String[] args) {
        String test1 = "abc";
        printAllPermutations1(test1);
        System.out.println("======");
        printAllPermutations2(test1);
        System.out.println("======");

        String test2 = "acc";
        printAllPermutations1(test2);
        System.out.println("======");
        printAllPermutations2(test2);
        System.out.println("======");
    }

}

```

程序运行结果

```java
abc
acb
cab
cba
abc
acb
======
abc
acb
cab
cba
======
acc
acc
cac
cca
acc
acc
======
acc
cac
cca
======
```



### （五）母牛数量

母牛每年生一只母牛，新出生的母牛成长三年后也能每年生一只 母牛，假设不会死。求N年后，母牛的数量。

![母牛示例](AlgorithmEasyDay08.resource/%E6%AF%8D%E7%89%9B%E7%A4%BA%E4%BE%8B.png)

计算公式为：$$F(n) = F(n - 1) + F(n - 3)$$，
因为今年牛的数量 = 去年牛的数量 + 三年前牛对应生的牛的数量。

代码：== 待补充== ，该代码的时间复杂度为： O(n)，将在进阶班优化为  $$O(log^{n})$$。
```java
package com.gjxaiou.easy.day08;

public class Code_05_Cow {

	public static int cowNumber1(int n) {
		if (n < 1) {
			return 0;
		}
		if (n == 1 || n == 2 || n == 3) {
			return n;
		}
		return cowNumber1(n - 1) + cowNumber1(n - 3);
	}

	public static int cowNumber2(int n) {
		if (n < 1) {
			return 0;
		}
		if (n == 1 || n == 2 || n == 3) {
			return n;
		}
		int res = 3;
		int pre = 2;
		int prepre = 1;
		int tmp1 = 0;
		int tmp2 = 0;
		for (int i = 4; i <= n; i++) {
			tmp1 = res;
			tmp2 = pre;
			res = res + prepre;
			pre = tmp1;
			prepre = tmp2;
		}
		return res;
	}

	public static void main(String[] args) {
		int n = 20;
		System.out.println(cowNumber1(n));
		System.out.println(cowNumber2(n));
	}

}

```



**进阶问题**
如果每只母牛只能活10年，求N年后，母牛的数量。






## 二、动态规划

- 从暴力递归中来；来优化暴力递归
- 将每一个子问题的解记录下来，避免重复计算；
- 把暴力递归的过程抽象成了状态表达；
- 并且存在化简状态表达，使其更加简洁的可能；


**问题**
给你一个二维数组，二维数组中的每个数都是正数，要求从左上角走到右下角，每一步只能向右或者向下。沿途经过的数字要累加起来。返回最小的路径和。

**首先使用暴力规划求解**
==依赖关系见笔记==

代码中的递归 =》包含大量的重复计算，将重复的这些结果计算完保留下来可以避免重复计算。

代码：
```java

```

### 问题
给你一个数组arr，和一个整数aim。如果可以任意选择arr中的数字，能不能累加得到aim，返回true或者false，这里为了简化，数组中的所有元素都是正数，同样要得到的整数也是一个正数。






### 暴力递归修改为动态规划
- 步骤一：写出尝试版本
- 步骤二：分析可变参数，哪几个参数可以代表返回值状态，可变参数几维即 DP 为几维表
- 步骤三：在 DP 表中找到需要的最终状态，然后标明；
- 步骤四：回到代码的 baseCase 中，在 DP 中设置到完全不依赖的值；
- 步骤五：看一个普遍位置需要哪些位置信息；
- 步骤六：逆过程回去就是 DP 填表的顺序；


