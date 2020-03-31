## AlgorithmEasyDay08

[TOC]

## 一、暴力递归

- 把问题转化为规模缩小了的同类问题的子问题；
- 有明确的不需要继续进行递归的条件（base case），因为不能一直递归；
- 有当得到了子问题的结果之后的决策过程；
- 不记录每一个子问题的解；



### （一）问题一：求 n!

【问题】求 $$n!$$ 的结果

【分析】该问题可以分解为子问题：$$n * (n - 1)!$$，同样可以一层层分解，逐层进行依赖，**但是其实每一层都是不知道具体的计算方法**；

#### 方法一：从上到下进行递归

- 第一要素：明确函数功能

    这个函数就是计算输入为 n 的阶乘值，所以返回值为 int，参数为 n

    ```java
    package com.gjxaiou.easy.day08;
    
    public class Factorial {
        public static long getFactorial(int n) {
        
        }
    }
    ```

- 第二要素：寻找递归结束条件

    当 n = 1 的时候，可以明确的得到函数值

    ```java
    package com.gjxaiou.easy.day08;
    
    public class Factorial {
        public static long getFactorial(int n) {
            // 递归结束条件
        	if(n == 1){
        		return 1;
        	}
        }
    }
    ```

- 第三元素：找出函数等价关系式

    找函数等级关系式肯定**需要不断缩小参数范围**，由题意分析可知，f(n) = n * f(n - 1);

    ```java
    package com.gjxaiou.easy.day08;
    
    public class Factorial {
        public static long getFactorial(int n) {
            // 递归结束条件
            if(n == 1){
                return 1;
            }
            return n * getFactorial(n - 1);
        }
    }
    ```

- 最后检查一下递归结束条件是否有遗漏，是否可以结束递归，这里是可以的，当然这里忽略讨论 n = 0 的情况，如果加上改情况只需要将递归结束条件更改为：

    ```java
    if(n == 0 || n == 1){
    	return 1;
    }
    ```



#### 方法二：从下到上递推

这个问题的计算方式： $$ 1 * 2 * 3 * 4 * ... * n$$，这种方式就是上面计算方式的逆过程。

```java
package com.gjxaiou.easy.day08;

public class Factorial {
 
    // 非递归版本
    public static long getFactorial2(int n) {
        long result = 1L;
        for (int i = 1; i <= n; i++) {
            result *= i;
        }
        return result;
    }
}
```



### （二）汉诺塔问题

【问题】给定一个数 n，表示 n 层汉诺塔问题。汉诺塔问题为：从左到右有A、B、C三根柱子，其中 A 柱子上面有从小叠到大的 n 个圆盘，现要求将 A 柱子上的圆盘移到 C 柱子上去，期间只有一个原则：一次只能移到一个盘子且大盘子不能在小盘子上面，求移动的步骤和移动的次数，打印 N 层汉诺塔从最左边移动到最右边的全部过程。

【进阶问题】给定一个汉诺塔的状况用数组 arr 表示（arr中只有1，2，3三种数字），请返回这是汉诺塔最优步数的第几步？

【举个栗子】

> arr = {3,2,1}
> arr 长度为 3，表示这是一个 3 层汉诺塔问题；
> arr[0] == 3表示上面的汉诺塔在右边；
> arr[1] == 2表示中间的汉诺塔在中间；
> arr[2] == 1表示底下的汉诺塔在左间；
> 这种状况是3层汉诺塔最优步数的第2步，所以返回2。

**该问题的递归不能改为动态规划**，因为其每一步都需要打印结果。

**汉诺塔原则**：只能小的压大的，不能大的压小的；

![汉诺塔示例](AlgorithmEasyDay08.resource/%E6%B1%89%E8%AF%BA%E5%A1%94%E7%A4%BA%E4%BE%8B.png)

> 注：图示表示将 from 柱上的  ③②① 移动到 to 柱上同样保持  ③②① ，其它序号为移动步骤。

步骤一：将 1 ~ n - 1 移动到 help 

步骤二：见 n 移动到 to

步骤三：将 1 ~ n -1 移动到 to

**一共需要的步骤**： $$2^n - 1$$

```java
package com.gjxaiou.easy.day08;

public class Hanoi {

    // 方式一：
    public static void hanoi(int n) {
        if (n > 0) {
            func(n, n, "left", "mid", "right");
        }
    }

   public static void func(int rest, int down, String from, String help, String to) {
        if (rest == 1) {
            System.out.println("move " + down + " from " + from + " to " + to);
        } else {
            // 将 1 ~ n - 1 移动到 help 上
            func(rest - 1, down - 1, from, to, help);
            // 将 n 移动到
            func(1, down, from, help, to);
            func(rest - 1, down - 1, help, from, to);
        }
    }

    // 方式二：递归版本，只有这一个函数
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

    public static void main(String[] args) {
        int n = 5;
        hanoi(n);

        process(5, "左", "右", "中");
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

**问题**：打印一个字符串的全部子序列，包括空字符串，**注意**：不是子串（在原字符串中可以找到）。

![子序列示例](AlgorithmEasyDay08.resource/%E5%AD%90%E5%BA%8F%E5%88%97%E7%A4%BA%E4%BE%8B.png)

代码：

```java
package com.gjxaiou.easy.day08;

import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class PrintAllSubsquences {
    // 递归版本一：
    // res 为上一级决策之后的字符串值
    public static void printAllSub(String inputString, int i, String res) {
        char[] str = inputString.toCharArray();
        // 如果到最后一个元素就不要递归了
        if (i == str.length) {
            System.out.println(res);
            return;
        }
        // 有两种决策，一种当前为空，就是将上一步决策接着往下扔，另一种就是加上当前字符串然后往下扔
        printAllSub(inputString, i + 1, res);
        printAllSub(inputString, i + 1, res + String.valueOf(str[i]));
    }


    // 方式二：
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

    public static void main(String[] args) {
        String test = "abc";
        // 测试递归版本
        printAllSub(test, 0, "");
        System.out.println("----------------------");
        printAllSubsquence(test);
        System.out.println("----------------");
    }
}

```

程序运行结果

```java
c
b
bc
a
ac
ab
abc
----------------------
abc
ab 
ac
a  
bc
b 
c    
```



### （四）全排列

**【问题】**：打印一个字符串的全部排列

**【进阶问题】**：打印一个字符串的全部排列，要求不要出现重复的排列



**原问题分析**

把需要全排列的字符串分为两部分看待：

- 字符串的第一个字符；

- 第一个字符后面的所有字符；

求所有可能出现在第一个位置的字符；将第一个字符和后面的字符一次交换；

固定第一个字符，对第一个字符后面的所有字符求全排列。第一个字符后面的所有字符又可以分为两部分；

- 第一元素：确定函数作用

    函数打印出字符串的全部排列，输入值类型为字符型数组，和当前字符长度。输出值为 void；

    ```java
    package com.gjxaiou.easy.day08;
    
    import java.util.HashSet;
    
    /**
     * 打印一个字符串的全排列（去重）
     */
    public class PrintAllPermutations {
    
        // 方案一：全排列（不去重）
        public static void printAllPermutations1(String str) {
            char[] charArray = str.toCharArray();
            process1(charArray, 0);
        }
    
        public static void process1(char[] charArray, int local) {
            
        }
    }
    ```

- 第二元素：确定递归结束条件

    这里的递归结束条件比较明显，就是排列的字符串长度达到原始字符串长度就停止。

    ```java
    package com.gjxaiou.easy.day08;
    
    import java.util.HashSet;
    
    /**
     * 打印一个字符串的全排列（去重）
     */
    public class PrintAllPermutations {
    
        // 方案一：全排列（不去重）
        public static void printAllPermutations1(String str) {
            char[] charArray = str.toCharArray();
            process1(charArray, 0);
        }
    
        public static void process1(char[] charArray, int local) {
            if (local == charArray.length) {
                System.out.println(String.valueOf(charArray));
            }
         
        }
    }
    ```

- 第三元素：寻找函数等价关系式

    等价关系式很明显，整个字符串的交换可以设定为第一个字符和后面所有字符进行交换，然后固定第一个字符，将后面 n - 1 个字符全排列，最后在交换一次即可

    ```java
    package com.gjxaiou.easy.day08;
    
    import java.util.HashSet;
    
    /**
     * 打印一个字符串的全排列（去重）
     */
    public class PrintAllPermutations {
    
        // 方案一：全排列（不去重）
        public static void printAllPermutations1(String str) {
            char[] charArray = str.toCharArray();
            process1(charArray, 0);
        }
    
        public static void process1(char[] charArray, int local) {
            if (local == charArray.length) {
                System.out.println(String.valueOf(charArray));
            }
            for (int j = local; j < charArray.length; j++) {
                // 将第一个字符与后面的字符交换
                swap(charArray, local, j);
                // 对后面的所有字符进行全排列
                process1(charArray, local + 1);
                // 再将原来交换的字符交换回来，以便第一个字符在与其他字符交换
                swap(charArray, local, j);
            }
        }
        
        public static void swap(char[] chs, int i, int j) {
            char tmp = chs[i];
            chs[i] = chs[j];
            chs[j] = tmp;
        }
    }
    ```

    

**进阶问题**：使用一个 HashSet 进行去重即可

```java
package com.gjxaiou.easy.day08;

import java.util.HashSet;

/**
 * 打印一个字符串的全排列（去重）
 */
public class PrintAllPermutations {
    // 方法二：全排列去重
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
                swap(chs, i, j);
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

**推导步骤**

- 第一因素：确定函数作用

    函数作用就是返回 N 年之后母牛的数量

    ```java
    package com.gjxaiou.easy.day08;
    
    public class Cow {
    	public static int cowNumber1(int n) {
    
    
        }
    }
    
    ```

- 第二因素：确定递归结束条件

    第一年肯定是知道的，一共是 1 头

    ```java
    package com.gjxaiou.easy.day08;
    
    public class Cow {
    	public static int cowNumber1(int n) {
    		if(n == 1){
                return 1;
            }
        }
    }
    
    ```

- 第三因素：寻找函数等价关系式

    由题意知道 $$F(n) = F(n - 1) + F(n - 3)$$

    ```java
    package com.gjxaiou.easy.day08;
    
    public class Cow {
    	public static int cowNumber1(int n) {
    		if(n == 1){
                return 1;
            }
            return cowNumber1(n - 1) + cowNumber1(n - 3);
        }
    }
    
    ```

    最后检查递归结束条件，发现 n == 1 时候可以结束，n == 2 时候 n - 3 为 -1 结束不了，n == 3 时候 n - 3 == 0 结束不了，n == 4 时候可以结束。所以缺少条件，最终代码为：

    ```java
    package com.gjxaiou.easy.day08;
    
    public class Cow {
    
    	public static int cowNumber1(int n) {
    		if (n < 1) {
    			return 0;
    		}
    		if (n == 1 || n == 2 || n == 3) {
    			return n;
    		}
    		return cowNumber1(n - 1) + cowNumber1(n - 3);
    	}
    }    
    ```



**方法二：从底向上递推**，因为这题可以明确的推出迭代公式，所以可以从底向上计算。

```java
package com.gjxaiou.easy.day08;

public class Cow {
    // 方法二：从下到上的递推
    public static int cowNumber2(int n) {
        if (n < 1) {
            return 0;
        }
        if (n == 1 || n == 2 || n == 3) {
            return n;
        }

        int preRes = 3;
        int prePre = 2;
        int prePrePre = 1;
        int temp1 = 0;
        int temp2 = 0;
        for (int i = 4; i <= n; i++) {
            temp1 = preRes;
            temp2 = prePre;
            // 当前值等于 f(n - 1) 即 preRes + f(n - 3) 即 prePrePre
            preRes += prePrePre;
            // f(n - 2) 值向右移动一个变成了 f(n - 1)，即 temp1 保存的 preRes 值。
            prePre = temp1;
            // f(n - 3) 值向右移动一个变成了 f(n - 2)，即 temp2 保存的 prePrePre 值。
            prePrePre = temp2;
        }
        return preRes;
    }
}

```

该代码的时间复杂度为： O(n)，将在进阶班优化为  $$O(log^{n})$$。

**进阶问题**
如果每只母牛只能活10年，求N年后，母牛的数量。



### （六）逆序栈

给你一个栈，请你逆序这个栈，不能申请额外的数据结构，只能使用递归函数。

**举个栗子**：一个栈依次压入1、2、3，将栈转置，使栈顶到栈底依次是1、2、3，只能用递归函数，不能借用额外的数据结构包括栈

**解决方案**：使用两个递归

```java
package com.gjxaiou.easy.day08;

import java.util.Stack;

public class ReverseStackUsingRecursive {
    /**
     * 每层递归取出栈底的元素并缓存到变量中，直到栈空；
     * 然后逆向将每层变量压入栈，最后实现原栈数据的逆序。
     *
     * @param stack
     */
    public static void reverse(Stack<Integer> stack) {
        if (stack.isEmpty()) {
            return;
        }
        // 依次返回1、2、3
        int i = getAndRemoveLastElement(stack);
        reverse(stack);
        // 依次压入3、2、1
        stack.push(i);
    }

    // 返回并且移除栈底元素（栈内元素：<栈底>1,2,3,4,5<栈顶>变为2,3,4,5<栈顶>）.
    public static int getAndRemoveLastElement(Stack<Integer> stack) {
        int result = stack.pop();
        if (stack.isEmpty()) {
            return result;
        } else {
            int last = getAndRemoveLastElement(stack);
            stack.push(result);
            // 第一轮时候返回栈底元素 1
            return last;
        }
    }

    public static void main(String[] args) {
        // Stack 继承 Vector，默认容量是 10(Vector 构造函数中进行了设置)
        Stack<Integer> test = new Stack<Integer>();
        test.push(1);
        test.push(2);
        test.push(3);
        test.push(4);
        test.push(5);
        reverse(test);
        while (!test.isEmpty()) {
            System.out.println(test.pop());
        }
    }
}
```

程序运行结果：

```java
1
2
3
4
5
```




## 二、动态规划

- 从暴力递归中来；来优化暴力递归

- 将每一个子问题的解记录下来，避免重复计算；
- 把暴力递归的过程抽象成了状态表达；
- 并且存在化简状态表达，使其更加简洁的可能；





### （一）矩阵路径最小和

【题目】给你一个二维数组，二维数组中的每个数都是正数，要求从左上角走到右下角，每一步只能向右或者向下。沿途经过的数字要累加起来。返回最小的路径和。

**首先使用暴力规划求解**

![image-20200221163654710](AlgorithmEasyDay08.resource/image-20200221163654710.png)


代码中的递归 =》包含大量的重复计算，将重复的这些结果计算完保留下来可以避免重复计算。

代码：
```java
package com.gjxaiou.easy.day08;

public class MinPath {

    public static int minPath1(int[][] matrix) {
        return process1(matrix, matrix.length - 1, matrix[0].length - 1);
    }

    // i 表示当前位置的行号，j 表示当前位置的列号；函数表示从 (i，j) 出发到达最右下角位置，最小路径和是多少（并且返回）
    public static int process1(int[][] matrix, int i, int j) {
        int res = matrix[i][j];
        if (i == 0 && j == 0) {
            return res;
        }
        if (i == 0 && j != 0) {
            return res + process1(matrix, i, j - 1);
        }
        if (i != 0 && j == 0) {
            return res + process1(matrix, i - 1, j);
        }
        return res + Math.min(process1(matrix, i, j - 1), process1(matrix, i - 1, j));
    }
    

    // 对应的递归版本（这里属于枚举了）
    public static int walk(int[][] matrix, int i, int j) {
        // 如果达到最右下角元素，则该元素达到最右下角距离为最右下角的值
        if (i == matrix.length - 1 && j == matrix[0].length - 1) {
            return matrix[i][j];
        }
        // 如果到达最底下一行，只能向右走，距离最右下角距离为右边元素距离最右下角元素的距离；
        if (i == matrix.length - 1) {
            return matrix[i][j] + walk(matrix, i, j + 1);
        }
        // 如果到达最右边一列
        if (j == matrix[0].length - 1) {
            return matrix[i][j] + walk(matrix, i + 1, j);
        }
        // 该点右边位置到最右下角的路径和
        int right = walk(matrix, i, j + 1);
        // 该点下边位置到最右下角的路径和
        int down = walk(matrix, i + 1, j);
        // 选择值比较小的那一个
        return matrix[i][j] + Math.min(right, down);
    }

    public static int minPath2(int[][] m) {
        if (m == null || m.length == 0 || m[0] == null || m[0].length == 0) {
            return 0;
        }
        int row = m.length;
        int col = m[0].length;
        int[][] dp = new int[row][col];
        dp[0][0] = m[0][0];
        for (int i = 1; i < row; i++) {
            dp[i][0] = dp[i - 1][0] + m[i][0];
        }
        for (int j = 1; j < col; j++) {
            dp[0][j] = dp[0][j - 1] + m[0][j];
        }
        for (int i = 1; i < row; i++) {
            for (int j = 1; j < col; j++) {
                dp[i][j] = Math.min(dp[i - 1][j], dp[i][j - 1]) + m[i][j];
            }
        }
        return dp[row - 1][col - 1];
    }

    // for test
    public static int[][] generateRandomMatrix(int rowSize, int colSize) {
        if (rowSize < 0 || colSize < 0) {
            return null;
        }
        int[][] result = new int[rowSize][colSize];
        for (int i = 0; i != result.length; i++) {
            for (int j = 0; j != result[0].length; j++) {
                result[i][j] = (int) (Math.random() * 10);
            }
        }
        return result;
    }

    public static void main(String[] args) {
        int[][] m = {{1, 3, 5, 9}, {8, 1, 3, 4}, {5, 0, 6, 1}, {8, 8, 4, 0}};
        System.out.println(minPath1(m));
        System.out.println(minPath2(m));

        m = generateRandomMatrix(6, 7);
        System.out.println(minPath1(m));
        System.out.println(minPath2(m));

        // 测试递归
        int walk = walk(m, 0, 0);
        System.out.println(walk);
    }
}

```

### （二）任意数字累加为 aim
给你一个数组arr，和一个整数aim。如果可以任意选择arr中的数字，能不能累加得到aim，返回true或者false，这里为了简化，数组中的所有元素都是正数，同样要得到的整数也是一个正数。

```java
package com.gjxaiou.easy.day08;

public class MoneyProblem {

    public static boolean money1(int[] arr, int aim) {
        return process1(arr, 0, 0, aim);
    }

    public static boolean process1(int[] arr, int i, int sum, int aim) {
        if (sum == aim) {
            return true;
        }
        // sum != aim
        if (i == arr.length) {
            return false;
        }
        return process1(arr, i + 1, sum, aim) || process1(arr, i + 1, sum + arr[i], aim);
    }

    public static boolean money2(int[] arr, int aim) {
        boolean[][] dp = new boolean[arr.length + 1][aim + 1];
        for (int i = 0; i < dp.length; i++) {
            dp[i][aim] = true;
        }
        for (int i = arr.length - 1; i >= 0; i--) {
            for (int j = aim - 1; j >= 0; j--) {
                dp[i][j] = dp[i + 1][j];
                if (j + arr[i] <= aim) {
                    dp[i][j] = dp[i][j] || dp[i + 1][j + arr[i]];
                }
            }
        }
        return dp[0][0];
    }

    // 递归版本
    public static boolean isSum(int[] arr, int i, int sum, int aim) {
        if (i == arr.length) {
            return sum == aim;
        }
        return isSum(arr, i + 1, sum, aim) || isSum(arr, i + 1, sum + arr[i], aim);
    }

    public static void main(String[] args) {
        int[] arr = {1, 4, 8};
        int aim = 12;
        System.out.println(money1(arr, aim));
        System.out.println(money2(arr, aim));

        // 验证递归
		isSum(arr,0,0,aim);
    }

}

```


### 暴力递归修改为动态规划
- 步骤一：写出尝试版本
- 步骤二：分析可变参数，哪几个参数可以代表返回值状态，可变参数几维即 DP 为几维表
- 步骤三：在 DP 表中找到需要的最终状态，然后标明；
- 步骤四：回到代码的 baseCase 中，在 DP 中设置到完全不依赖的值；
- 步骤五：看一个普遍位置需要哪些位置信息；
- 步骤六：逆过程回去就是 DP 填表的顺序；


