# AlgorithmSummary



## 一、数组

### （一）对一组数据进行排序需要考虑的点

* 有没有可能包含有大量重复的元素?

* 是否大部分数据距离它正确的位置很近?是否近乎有序

* 是否数据的取值范围非常有限?比如对学生成绩排序

* 是否需要稳定排序?

* 是否是使用链表存储的?

* 数据的大小是否可以装载在内存里?



### （二）数组中常见问题

- 排序：选择排序、插入排序、归并排序、快速排序；

- 查找：二分查找法；

- 数据结构：栈、堆、队列； 



### （三）正确的程序

- 明确变量定义

- 循环不变量

- 小数据量调试

- 大数据量测试



## 二、递归

### 注：递归过程总结

递归很好用，可以高度套路：

1.  分析问题的解决需要哪些步骤（这里是遍历每个结点，确认每个结点为根节点的子树是否为平衡二叉树）
2.  确定递归：父问题是否和子问题相同
3.  子过程要收集哪些信息
4.  本次递归如何利用子过程返回的信息得到本过程要返回的信息
5.  `base case`



## 三、动态规划

**暴力递归修改为动态规划**

- 步骤一：写出尝试版本
- 步骤二：分析可变参数，哪几个参数可以代表返回值状态，可变参数几维即 DP 为几维表
- 步骤三：在 DP 表中找到需要的最终状态，然后标明；
- 步骤四：回到代码的 baseCase 中，在 DP 中设置到完全不依赖的值；
- 步骤五：看一个普遍位置需要哪些位置信息；
- 步骤六：逆过程回去就是 DP 填表的顺序；

## 注：目录说明

- AlgorithmEasy01：基本排序算法

# AlgorithmMediumDay08

[TOC]

## 一、两个有序数组间相加和的 TOP K 问题

【题目】
给定两个**有序数组** arr1 和 arr2，再给定一个整数 k，返回分别来自 arr1 和 arr2 的两个数相加和最大的前 k 个，两个数必须分别来自两个数组。
【举例】
`arr1=[1,2,3,4,5]，arr2=[3,5,7,9,11]，k=4。`
返回数组[16,15,14,14]。
【要求】
时间复杂度达到O(klogk)。

【思路】

使用大根堆结构。假设 arr1 的长度是 M，arr2 的长度是 N。因为是已经排序的数组，arr1 中最后一个数加上 arr2 中最后一个数一定就是最大的相加和。将这个数压入大根堆中。然后从大根堆中弹出一个堆顶，此时这个堆顶一定是 (M-1, N-1) 位置的和，表示获得一个最大相加和。然后，将两个相邻位置的和再放入堆中，即位置 (M-1,N-2) 和 (M-2, N-1)，因为除 (M-1, N-1) 位置的和外，最大的相加和一定在位置(M-1,N-2)和(m-2, N-1)中产生。重新调整大根堆，然后继续弹出，继续将弹出元素的两个相邻位置添加到堆中，直到弹出的元素达到K个。

```java
package nowcoder.advanced.advanced_class_08;

import java.util.Arrays;
import java.util.HashSet;

public class Code_01_TopKSumCrossTwoArrays {

    public static class HeapNode {
        public int row;
        public int col;
        public int value;

        public HeapNode(int row, int col, int value) {
            this.row = row;
            this.col = col;
            this.value = value;
        }
    }

    public static int[] topKSum(int[] a1, int[] a2, int topK) {
        if (a1 == null || a2 == null || topK < 1) {
            return null;
        }
        topK = Math.min(topK, a1.length * a2.length);
        HeapNode[] heap = new HeapNode[topK + 1];
        int heapSize = 0;
        int headR = a1.length - 1;
        int headC = a2.length - 1;
        int uR = -1;
        int uC = -1;
        int lR = -1;
        int lC = -1;
        heapInsert(heap, heapSize++, headR, headC, a1[headR] + a2[headC]);
        HashSet<String> positionSet = new HashSet<String>();
        int[] res = new int[topK];
        int resIndex = 0;
        while (resIndex != topK) {
            HeapNode head = popHead(heap, heapSize--);
            res[resIndex++] = head.value;
            headR = head.row;
            headC = head.col;
            uR = headR - 1;
            uC = headC;
            if (headR != 0 && !isContains(uR, uC, positionSet)) {
                heapInsert(heap, heapSize++, uR, uC, a1[uR] + a2[uC]);
                addPositionToSet(uR, uC, positionSet);
            }
            lR = headR;
            lC = headC - 1;
            if (headC != 0 && !isContains(lR, lC, positionSet)) {
                heapInsert(heap, heapSize++, lR, lC, a1[lR] + a2[lC]);
                addPositionToSet(lR, lC, positionSet);
            }
        }
        return res;
    }

    public static HeapNode popHead(HeapNode[] heap, int heapSize) {
        HeapNode res = heap[0];
        swap(heap, 0, heapSize - 1);
        heap[--heapSize] = null;
        heapify(heap, 0, heapSize);
        return res;
    }

    public static void heapify(HeapNode[] heap, int index, int heapSize) {
        int left = index * 2 + 1;
        int right = index * 2 + 2;
        int largest = index;
        while (left < heapSize) {
            if (heap[left].value > heap[index].value) {
                largest = left;
            }
            if (right < heapSize && heap[right].value > heap[largest].value) {
                largest = right;
            }
            if (largest != index) {
                swap(heap, largest, index);
            } else {
                break;
            }
            index = largest;
            left = index * 2 + 1;
            right = index * 2 + 2;
        }
    }

    public static void heapInsert(HeapNode[] heap, int index, int row, int col,
                                  int value) {
        heap[index] = new HeapNode(row, col, value);
        int parent = (index - 1) / 2;
        while (index != 0) {
            if (heap[index].value > heap[parent].value) {
                swap(heap, parent, index);
                index = parent;
                parent = (index - 1) / 2;
            } else {
                break;
            }
        }
    }

    public static void swap(HeapNode[] heap, int index1, int index2) {
        HeapNode tmp = heap[index1];
        heap[index1] = heap[index2];
        heap[index2] = tmp;
    }

    public static boolean isContains(int row, int col, HashSet<String> set) {
        return set.contains(String.valueOf(row + "_" + col));
    }

    public static void addPositionToSet(int row, int col, HashSet<String> set) {
        set.add(String.valueOf(row + "_" + col));
    }

    // For test, this method is inefficient but absolutely right
    public static int[] topKSumTest(int[] arr1, int[] arr2, int topK) {
        int[] all = new int[arr1.length * arr2.length];
        int index = 0;
        for (int i = 0; i != arr1.length; i++) {
            for (int j = 0; j != arr2.length; j++) {
                all[index++] = arr1[i] + arr2[j];
            }
        }
        Arrays.sort(all);
        int[] res = new int[Math.min(topK, all.length)];
        index = all.length - 1;
        for (int i = 0; i != res.length; i++) {
            res[i] = all[index--];
        }
        return res;
    }

    public static int[] generateRandomSortArray(int len) {
        int[] res = new int[len];
        for (int i = 0; i != res.length; i++) {
            res[i] = (int) (Math.random() * 50000) + 1;
        }
        Arrays.sort(res);
        return res;
    }

    public static void printArray(int[] arr) {
        for (int i = 0; i != arr.length; i++) {
            System.out.print(arr[i] + " ");
        }
        System.out.println();
    }

    public static boolean isEqual(int[] arr1, int[] arr2) {
        if (arr1 == null || arr2 == null || arr1.length != arr2.length) {
            return false;
        }
        for (int i = 0; i != arr1.length; i++) {
            if (arr1[i] != arr2[i]) {
                return false;
            }
        }
        return true;
    }

    public static void main(String[] args) {
        int a1Len = 5000;
        int a2Len = 4000;
        int k = 2000;
        int[] arr1 = generateRandomSortArray(a1Len);
        int[] arr2 = generateRandomSortArray(a2Len);
        long start = System.currentTimeMillis();
        int[] res = topKSum(arr1, arr2, k);
        long end = System.currentTimeMillis();
        System.out.println(end - start + " ms");

        start = System.currentTimeMillis();
        int[] absolutelyRight = topKSumTest(arr1, arr2, k);
        end = System.currentTimeMillis();
        System.out.println(end - start + " ms");

        System.out.println(isEqual(res, absolutelyRight));

    }

}

```

程序运行结果：

```java
5 ms
843 ms
true
```





## 二、子数组的最大累加和问题

【题目】
给定一个数组arr，返回子数组的最大累加和。
例如，arr=[1,-2,3,5,-2,6,-1]，所有的子数组中，[3,5,-2,6]可以累加出最大的和12，所以返回12。
【要求】
如果arr长度为N，要求时间复杂度为O(N)，额外空间复杂度为O(1)。



```java
package nowcoder.advanced.advanced_class_08;

public class Code_02_MaxSubMatrixSum {

    public static int maxSum(int[][] m) {
        if (m == null || m.length == 0 || m[0].length == 0) {
            return 0;
        }
        int max = Integer.MIN_VALUE;
        int cur = 0;
        int[] s = null;
        for (int i = 0; i != m.length; i++) {
            s = new int[m[0].length];
            for (int j = i; j != m.length; j++) {
                cur = 0;
                for (int k = 0; k != s.length; k++) {
                    s[k] += m[j][k];
                    cur += s[k];
                    max = Math.max(max, cur);
                    cur = cur < 0 ? 0 : cur;
                }
            }
        }
        return max;
    }

    public static void main(String[] args) {
        int[][] matrix = {{-90, 48, 78}, {64, -40, 64}, {-81, -7, 66}};
        System.out.println(maxSum(matrix));

    }
}

```

程序运行结果：

```java
209
```





## 三、边界都是1的最大正方形大小

【题目】
给定一个NN的矩阵matrix，在这个矩阵中，只有0和1两种值，返回边框全是1的最大正方形的边长长度。
例如：
0 1 1 1 1
0 1 0 0 1
0 1 0 0 1
0 1 1 1 1
0 1 0 1 1
其中，边框全是1的最大正方形的大小为4*4，所以返回4。

```java
package nowcoder.advanced.advanced_class_08;

public class Code_03_MaxOneBorderSize {

    public static void setBorderMap(int[][] m, int[][] right, int[][] down) {
        int r = m.length;
        int c = m[0].length;
        if (m[r - 1][c - 1] == 1) {
            right[r - 1][c - 1] = 1;
            down[r - 1][c - 1] = 1;
        }
        for (int i = r - 2; i != -1; i--) {
            if (m[i][c - 1] == 1) {
                right[i][c - 1] = 1;
                down[i][c - 1] = down[i + 1][c - 1] + 1;
            }
        }
        for (int i = c - 2; i != -1; i--) {
            if (m[r - 1][i] == 1) {
                right[r - 1][i] = right[r - 1][i + 1] + 1;
                down[r - 1][i] = 1;
            }
        }
        for (int i = r - 2; i != -1; i--) {
            for (int j = c - 2; j != -1; j--) {
                if (m[i][j] == 1) {
                    right[i][j] = right[i][j + 1] + 1;
                    down[i][j] = down[i + 1][j] + 1;
                }
            }
        }
    }

    public static int getMaxSize(int[][] m) {
        int[][] right = new int[m.length][m[0].length];
        int[][] down = new int[m.length][m[0].length];
        setBorderMap(m, right, down);
        for (int size = Math.min(m.length, m[0].length); size != 0; size--) {
            if (hasSizeOfBorder(size, right, down)) {
                return size;
            }
        }
        return 0;
    }

    public static boolean hasSizeOfBorder(int size, int[][] right, int[][] down) {
        for (int i = 0; i != right.length - size + 1; i++) {
            for (int j = 0; j != right[0].length - size + 1; j++) {
                if (right[i][j] >= size && down[i][j] >= size
                        && right[i + size - 1][j] >= size
                        && down[i][j + size - 1] >= size) {
                    return true;
                }
            }
        }
        return false;
    }

    public static int[][] generateRandom01Matrix(int rowSize, int colSize) {
        int[][] res = new int[rowSize][colSize];
        for (int i = 0; i != rowSize; i++) {
            for (int j = 0; j != colSize; j++) {
                res[i][j] = (int) (Math.random() * 2);
            }
        }
        return res;
    }

    public static void printMatrix(int[][] matrix) {
        for (int i = 0; i != matrix.length; i++) {
            for (int j = 0; j != matrix[0].length; j++) {
                System.out.print(matrix[i][j] + " ");
            }
            System.out.println();
        }
    }

    public static void main(String[] args) {
        int[][] matrix = generateRandom01Matrix(7, 8);
        printMatrix(matrix);
        System.out.println(getMaxSize(matrix));
    }
}

```

程序运行结果：

```java
0 0 1 1 1 0 0 0 
0 0 1 0 1 1 1 1 
0 0 1 0 1 0 1 1 
1 1 0 1 1 1 1 0 
0 0 0 0 0 1 0 1 
1 0 0 0 1 1 0 1 
1 1 1 0 1 0 1 1 
3
```



## 四、斐波那契系列问题的递归和动态规划

【题目】
给定整数N，返回斐波那契数列的第N项。
【补充题目1】
给定整数N，代表台阶数，一次可以跨2个或者1个台阶，返回有多少种走法。
【举例】
N=3，可以三次都跨1个台阶；也可以先跨2个台阶，再跨1个台阶；还可以先跨1个台阶，再跨2个台阶。所以有三种走法，返回3。
【补充题目2】
假设农场中成熟的母牛每年只会生1头小母牛，并且永远不会死。第一年农场有1只成熟的母牛，从第二年开始，母牛开始生小母牛。每只小母牛3年之后成熟又可以生小母牛。给定整数N，求出N年后牛的数量。
【举例】
N=6，第1年1头成熟母牛记为a；第2年a生了新的小母牛，记为b，总牛数为2；第3年a生了新的小母牛，记为c，总牛数为3；第4年a生了新的小母牛，记为d，总牛数为4。第5年b成熟了，a和b分别生了新的小母牛，总牛数为6；第6年c也成熟了，a、b和c分别生了新的小母牛，总牛数为9，返回9。
【要求】
对以上所有的问题，请实现时间复杂度O(logN)的解法。





程序运行结果：

```java

```





## 五、找到字符串的最长无重复字符子串

【题目】
给定一个字符串str，返回str的最长无重复字符子串的长度。
【举例】
str="abcd"，返回4
str="aabcb"，最长无重复字符子串为"abc"，返回3。
【要求】
如果str的长度为N，请实现时间复杂度为O(N)的方法。



```java

```

程序运行结果：

```java
bltpirbrifkownqrttwu
9
brifkownq
```



## 六、认识完美洗牌问题



