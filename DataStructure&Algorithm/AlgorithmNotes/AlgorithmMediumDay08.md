# AlgorithmMediumDay08

[TOC]

## 一、两个有序数组间相加和的 TOP K 问题

【题目】
给定两个**有序数组** `arr1` 和 `arr2`，再给定一个整数 `k`，返回分别来自 `arr1` 和 `arr2` 的两个数相加和最大的前 `k` 个，两个数必须分别来自两个数组。
【举例】
`arr1=[1,2,3,4,5]，arr2=[3,5,7,9,11]，k=4。`
返回数组[16,15,14,14]。
【要求】
时间复杂度达到O(klogk)。

【思路】**使用大根堆结构**

假设 arr1 的长度是 M，arr2 的长度是 N。因为是已经排序的数组，arr1 中最后一个数加上 arr2 中最后一个数一定就是最大的相加和。将这个数压入大根堆中。然后从大根堆中弹出一个堆顶，此时这个堆顶一定是 (M-1, N-1) 位置的和，表示获得一个最大相加和。然后，将两个相邻位置的和再放入堆中，即位置 (M-1,N-2) 和 (M-2, N-1)，因为除 (M-1, N-1) 位置的和外，最大的相加和一定在位置(M-1,N-2)和(m-2, N-1)中产生。重新调整大根堆，然后继续弹出，继续将弹出元素的两个相邻位置添加到堆中，直到弹出的元素达到K个。

```java
package com.gjxaiou.advanced.day08;

import java.util.Arrays;
import java.util.HashSet;

public class TopKSumCrossTwoArrays {

    public static class HeapNode {
        public int row;
        public int column;
        public int value;

        public HeapNode(int row, int col, int value) {
            this.row = row;
            this.column = col;
            this.value = value;
        }
    }

    public static int[] topKSum(int[] arr1, int[] arr2, int topK) {
        if (arr1 == null || arr2 == null || topK < 1) {
            return null;
        }
        topK = Math.min(topK, arr1.length * arr2.length);

        HeapNode[] heap = new HeapNode[topK + 1];
        int heapSize = 0;
        int headRight = arr1.length - 1;
        int headColumn = arr2.length - 1;
        int uR = -1;
        int uC = -1;
        int lR = -1;
        int lC = -1;
        heapInsert(heap, heapSize++, headRight, headColumn, arr1[headRight] + arr2[headColumn]);
        HashSet<String> positionSet = new HashSet<String>();
        int[] res = new int[topK];
        int resIndex = 0;
        while (resIndex != topK) {
            HeapNode head = popHead(heap, heapSize--);
            res[resIndex++] = head.value;
            headRight = head.row;
            headColumn = head.column;
            uR = headRight - 1;
            uC = headColumn;
            if (headRight != 0 && !isContains(uR, uC, positionSet)) {
                heapInsert(heap, heapSize++, uR, uC, arr1[uR] + arr2[uC]);
                addPositionToSet(uR, uC, positionSet);
            }
            lR = headRight;
            lC = headColumn - 1;
            if (headColumn != 0 && !isContains(lR, lC, positionSet)) {
                heapInsert(heap, heapSize++, lR, lC, arr1[lR] + arr2[lC]);
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


    ////////////////// 测试程序 //////////////////////////
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
给定一个数组 `arr`，返回子数组的最大累加和。
例如，`arr=[1,-2,3,5,-2,6,-1]`，所有的子数组中，`[3,5,-2,6]` 可以累加出最大的和 `12`，所以返回 `12`。
【要求】
如果 `arr` 长度为 `N`，要求时间复杂度为 O(N)，额外空间复杂度为O(1)。

```java
package com.gjxaiou.advanced.day08;

public class MaxSubMatrixSum {

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





## 三、边界都是 1 的最大正方形大小

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

```java
package nowcoder.advanced.advanced_class_08;

public class Code_04_FibonacciProblem {

	public static int f1(int n) {
		if (n < 1) {
			return 0;
		}
		if (n == 1 || n == 2) {
			return 1;
		}
		return f1(n - 1) + f1(n - 2);
	}

	public static int f2(int n) {
		if (n < 1) {
			return 0;
		}
		if (n == 1 || n == 2) {
			return 1;
		}
		int res = 1;
		int pre = 1;
		int tmp = 0;
		for (int i = 3; i <= n; i++) {
			tmp = res;
			res = res + pre;
			pre = tmp;
		}
		return res;
	}

	public static int f3(int n) {
		if (n < 1) {
			return 0;
		}
		if (n == 1 || n == 2) {
			return 1;
		}
		int[][] base = { { 1, 1 }, { 1, 0 } };
		int[][] res = matrixPower(base, n - 2);
		return res[0][0] + res[1][0];
	}

	public static int[][] matrixPower(int[][] m, int p) {
		int[][] res = new int[m.length][m[0].length];
		for (int i = 0; i < res.length; i++) {
			res[i][i] = 1;
		}
		int[][] tmp = m;
		for (; p != 0; p >>= 1) {
			if ((p & 1) != 0) {
				res = muliMatrix(res, tmp);
			}
			tmp = muliMatrix(tmp, tmp);
		}
		return res;
	}

	public static int[][] muliMatrix(int[][] m1, int[][] m2) {
		int[][] res = new int[m1.length][m2[0].length];
		for (int i = 0; i < m1.length; i++) {
			for (int j = 0; j < m2[0].length; j++) {
				for (int k = 0; k < m2.length; k++) {
					res[i][j] += m1[i][k] * m2[k][j];
				}
			}
		}
		return res;
	}

	public static int s1(int n) {
		if (n < 1) {
			return 0;
		}
		if (n == 1 || n == 2) {
			return n;
		}
		return s1(n - 1) + s1(n - 2);
	}

	public static int s2(int n) {
		if (n < 1) {
			return 0;
		}
		if (n == 1 || n == 2) {
			return n;
		}
		int res = 2;
		int pre = 1;
		int tmp = 0;
		for (int i = 3; i <= n; i++) {
			tmp = res;
			res = res + pre;
			pre = tmp;
		}
		return res;
	}

	public static int s3(int n) {
		if (n < 1) {
			return 0;
		}
		if (n == 1 || n == 2) {
			return n;
		}
		int[][] base = { { 1, 1 }, { 1, 0 } };
		int[][] res = matrixPower(base, n - 2);
		return 2 * res[0][0] + res[1][0];
	}

	public static int c1(int n) {
		if (n < 1) {
			return 0;
		}
		if (n == 1 || n == 2 || n == 3) {
			return n;
		}
		return c1(n - 1) + c1(n - 3);
	}

	public static int c2(int n) {
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

	public static int c3(int n) {
		if (n < 1) {
			return 0;
		}
		if (n == 1 || n == 2 || n == 3) {
			return n;
		}
		int[][] base = { { 1, 1, 0 }, { 0, 0, 1 }, { 1, 0, 0 } };
		int[][] res = matrixPower(base, n - 3);
		return 3 * res[0][0] + 2 * res[1][0] + res[2][0];
	}

	public static void main(String[] args) {
		int n = 20;
		System.out.println(f1(n));
		System.out.println(f2(n));
		System.out.println(f3(n));
		System.out.println("===");

		System.out.println(s1(n));
		System.out.println(s2(n));
		System.out.println(s3(n));
		System.out.println("===");

		System.out.println(c1(n));
		System.out.println(c2(n));
		System.out.println(c3(n));
		System.out.println("===");

	}

}

```

程序运行结果：

```java
6765
6765
6765
===
10946
10946
10946
===
1873
1873
1873
===
```







## 六、认识完美洗牌问题

```java
package nowcoder.advanced.advanced_class_08;

import java.util.Arrays;

public class Code_06_ShuffleProblem {

	// https://arxiv.org/pdf/0805.1598.pdf
	public static void shuffle(int[] arr) {
		if (arr != null && arr.length != 0 && (arr.length & 1) == 0) {
			shuffle(arr, 0, arr.length - 1);
		}
	}

	public static void shuffle(int[] arr, int l, int r) {
		while (r - l + 1 > 0) {
			int lenAndOne = r - l + 2;
			int bloom = 3;
			int k = 1;
			while (bloom <= lenAndOne / 3) {
				bloom *= 3;
				k++;
			}
			int m = (bloom - 1) / 2;
			int mid = (l + r) / 2;
			rotate(arr, l + m, mid, mid + m);
			cycles(arr, l - 1, bloom, k);
			l = l + bloom - 1;
		}
	}

	public static void cycles(int[] arr, int base, int bloom, int k) {
		for (int i = 0, trigger = 1; i < k; i++, trigger *= 3) {
			int next = (2 * trigger) % bloom;
			int cur = next;
			int record = arr[next + base];
			int tmp = 0;
			arr[next + base] = arr[trigger + base];
			while (cur != trigger) {
				next = (2 * cur) % bloom;
				tmp = arr[next + base];
				arr[next + base] = record;
				cur = next;
				record = tmp;
			}
		}
	}

	public static void rotate(int[] arr, int l, int m, int r) {
		reverse(arr, l, m);
		reverse(arr, m + 1, r);
		reverse(arr, l, r);
	}

	public static void reverse(int[] arr, int l, int r) {
		while (l < r) {
			int tmp = arr[l];
			arr[l++] = arr[r];
			arr[r--] = tmp;
		}
	}

	// for test
	public static void printArray(int[] arr) {
		for (int i = 0; i < arr.length; i++) {
			System.out.print(arr[i] + " ");
		}
		System.out.println();
	}

	// for test
	public static int[] generateArray() {
		int len = (int) (Math.random() * 10) * 2;
		int[] arr = new int[len];
		for (int i = 0; i < len; i++) {
			arr[i] = (int) (Math.random() * 100);
		}
		return arr;
	}

	// for test
	public static void shuffleTest(int[] arr) {
		int[] tarr = new int[arr.length];
		int bloom = arr.length + 1;
		for (int i = 1; i <= arr.length; i++) {
			tarr[((2 * i) % bloom) - 1] = arr[i - 1];
		}
		for (int i = 0; i < arr.length; i++) {
			arr[i] = tarr[i];
		}
	}

	public static boolean equalArrays(int[] arr1, int[] arr2) {
		if (arr1 == null || arr2 == null || arr1.length != arr2.length) {
			return false;
		}
		for (int i = 0; i < arr1.length; i++) {
			if (arr1[i] != arr2[i]) {
				return false;
			}
		}
		return true;
	}

	public static void main(String[] args) {
		for (int i = 0; i < 5000000; i++) {
			int[] arr = generateArray();
			int[] arr1 = Arrays.copyOfRange(arr, 0, arr.length);
			int[] arr2 = Arrays.copyOfRange(arr, 0, arr.length);
			shuffle(arr1);
			shuffleTest(arr2);
			if (!equalArrays(arr1, arr2)) {
				System.out.println("ooops!");
				printArray(arr);
				printArray(arr1);
				printArray(arr2);
				break;
			}
		}

	}

}

```

