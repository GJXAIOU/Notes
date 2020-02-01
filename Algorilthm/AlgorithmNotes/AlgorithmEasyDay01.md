---
tags: [时间复杂度，空间复杂度，冒泡排序，选择排序，插入排序，二分查找，递归]
note: 文章主要包含 时间复杂度，空间复杂度，冒泡排序，选择排序，插入排序，二分查找，递归 的内容；
style: summer
custom: GJXAIOU
---

# AlgorithmEasyDay01

## 一、时间复杂度和空间复杂度

### （一）时间复杂度

- 常数时间的操作：一个操作和样本数据量没有关系，每次都是在固定的时间内完成，叫做常数操作；
- 操作数量表达式中，只要高阶项，不要低阶项，去掉高阶项的系数，比较先比较次幂，然后比较系数以及其他项；
-  可以将 O(1) 看做每一次操作的时间；

**以一个示例比较：**（示例一）

一个有序数组A，另一个无序数组B，请打印B中的所有不在A中的数，A数
组长度为N，B数组长度为M。

- **算法流程1**：对于数组B中的每一个数，都在A中通过遍历的方式找一下；
相当于：B 中每一个数都要在 A 中遍历一遍，则需要操作 N 遍，而 B 中 M 个数都需要按照上面操作一遍，共操作 M * N 遍，因此时间复杂度为：$O(M * N)$；
- **算法流程2**：对于数组B中的每一个数，都在A中通过**二分的方式**找一下；
因为 A 中数组是有序的，因此可以进行二分查找，因此整体时间复杂度为：$O(M * \log_{2}^{N})$；
- **算法流程3**：先把数组B排序，然后用类似外排的方式打印所有不在A中出现的数；
因为可以是会用快速排序对数组 B 进行排序，因此时间复杂度为：$M * \log_{2}^{M}$，==**外排思想：**==  数组 A 开头放置下标 a，数组 B 开头放置下标 b，比较两个下标指向的值，如果 b <= a  指向的值，则 b 向右移动，否则 a 向右移动，其中若 b 指向的值  < a 指向的值，则 b 向右移动同时打印 b 指向的数，若等于则向右移动不打印；
因此整体外排时间复杂度最差为 $O(M + N)$，因此整个流程时间复杂度为：$O(M * log_{2}^{M}) + O(M + N)$

**总结：** 流程一：$O(M * N)$，流程二：$O(M * \log_{2}^{N})$，流程三：$O(M * log_{2}^{M}) + O(M + N)$；当 A 数组较短的时候，流程二较好，当 B 数组较短的时候，流程三较好（因为流程三需要对 B 进行排序）；


二分查找： **二分查找前提是该数组必须有序**，每次进行查找都是将数组划分一半，N 样本的数组一共可以划分 $\log_{2}^{N}$次，因此时间复杂度为$O(\log_{2}^{N})$

```java
package sort.nowcoder.easy.day01;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.HashMap;
import java.util.List;

public class GetAllNotIncluded {

	public static List<Integer> GetAllNotIncluded(int[] A, int[] B) {
		List<Integer> res = new ArrayList<>();
		for (int i = 0; i < B.length; i++) {
			int l = 0;
			int r = A.length - 1;
			boolean contains = false;
			while (l <= r) {
				int mid = l + ((r - l) >> 1);
				if (A[mid] == B[i]) {
					contains = true;
					break;
				}
				if (A[mid] > B[i]) {
					r = mid - 1;
				} else {
					l = mid + 1;
				}
			}
			if (!contains) {
				res.add(B[i]);
			}
		}
		return res;
	}
}
```




### （二）空间复杂度
就是在操作的过程中需要的**额外的空间**，如果仅仅需要有限个变量：$O(1)$，如果**需要原来数组的长度或者和样本数有关**，则为：$O(N)$；


## 二、排序算法

### （一）冒泡排序
两两比较，将较大的一个一个向上冒，**每次只能排好一个数**；

时间复杂度：$O(N^{2})$
空间复杂度：$O(1)$

```java
import java.util.Arrays;

public class BubbleSort {

    public static void bubbleSort(int[] sourceArray) {
        if (sourceArray == null || sourceArray.length < 2) {
            return;
        }
        // end 刚开始在 length-1,但是得大于零，每排完一圈就确定一个最大值，然后值减一；
        for (int end = sourceArray.length - 1; end > 0; end--) {
            for (int start = 0; start < end; start++) {
                if (sourceArray[start] > sourceArray[start + 1]) {
                    swap(sourceArray, start, start + 1);
                }
            }
        }
    }

    public static void swap(int[] sourceArray, int left, int right) {
        sourceArray[left] = sourceArray[left] ^ sourceArray[right];
        sourceArray[right] = sourceArray[left] ^ sourceArray[right];
        sourceArray[left] = sourceArray[left] ^ sourceArray[right];
    }
}    
```



### （二）选择排序
首先从 1 ~ N-1 上选择最小的数和 0 位置上互换，然后才能够 2 ~ N-1 上选择最小的数和 1 位置上互换.....，**每次只能排好一个数**；

时间复杂度：$O(N^{2})$
空间复杂度：$O(1)$

```java
package sort.nowcoder;

import java.util.Arrays;

public class SelectionSort {

    public static void selectionSort(int[] sourceArray) {
        if (sourceArray == null || sourceArray.length < 2) {
            return;
        }
        for (int start = 0; start < sourceArray.length - 1; start++) {
            int minIndex = start;
            // 从 i + 1 位置到最后的最小的数下标
            for (int cur = start + 1; cur < sourceArray.length; cur++) {
                minIndex = sourceArray[cur] < sourceArray[minIndex] ? cur : minIndex;
            }
            swap(sourceArray, start, minIndex);
        }
    }

    public static void swap(int[] sourceArray, int left, int right) {
        int tmp = sourceArray[left];
        sourceArray[left] = sourceArray[right];
        sourceArray[right] = tmp;
    }
}
```

**注意：**
上面的交换程序可以改为：
```java
 public static void swap(int[] sourceArray, int left, int right) {
        if (left == right){
            return;
        }
        sourceArray[left] = sourceArray[left] ^ sourceArray[right];
        sourceArray[right] = sourceArray[left] ^ sourceArray[right];
        sourceArray[left] = sourceArray[left] ^ sourceArray[right];
    }
```
因为如果之前 minIndex 没有改变，则 start 与 minIndex 是相等的，在进行swap()的时候，`arr[left] = arr[left] ^ arr[right];`相当于对两个相同的数做异或运算，结果是0。swap函数出现错误。可以在swap函数加上`if(left==right)return;`来解决。

交换数组中的两个数，一个数自己与自己异或结果为0；一个数与0异或，结果还是自己。



### （三）插入排序
首先默认第一个数（0）是排好序的，然后拿第二个数和第一个数比较，如果比第一个数小，就互换，反之不动，这样 0~1 位置上是排好序的；然后拿第三个数和第二个数比较，比他小就互换，如果更换之后再次和第一个数比较，看是否需要互换，最终 0~2 位置上是排好序的.......

最好的情况是原来就有序：$O(N)$，最差的情况是原来就是倒序的：$O(N^{2})$；因此最终的时间复杂度为：$O(N^{2})$，空间复杂度：$O(1)$；
```java
import java.util.Arrays;

public class InsertionSort {

    public static void insertionSort(int[] sourceArray) {
        if (sourceArray == null || sourceArray.length < 2) {
            return;
        }
        for (int end = 1; end < sourceArray.length; end++) {
            for (int cur = end - 1; cur >= 0 && sourceArray[cur] > sourceArray[cur + 1]; cur--) {
                swap(sourceArray, cur, cur + 1);
            }
        }
    }

    public static void swap(int[] sourceArray, int left, int right) {
        sourceArray[left] = sourceArray[left] ^ sourceArray[right];
        sourceArray[right] = sourceArray[left] ^ sourceArray[right];
        sourceArray[left] = sourceArray[left] ^ sourceArray[right];
    }
}
```



### （四）归并排序
归并排序（MERGE-SORT）是利用**归并**的思想实现的排序方法，该算法采用经典的**分治**（divide-and-conquer）策略（分治法将问题**分**(divide)成一些小的问题然后递归求解，而**治(conquer)**的阶段则将分的阶段得到的各答案"修补"在一起，即分而治之)。

流程：首先将数组对半分为两个部分，然后分别对左右两边进行排序；最后对整体进行外排；
重点关注：最后整体的外排

![](AlgorithmEasyDay01.resource/1024555-20161218163120151-452283750.png)

![](AlgorithmEasyDay01.resource/1024555-20161218194508761-468169540.png)

```java
package nowcoder.easy.day01;

import java.util.Arrays;

/**
 * 归并排序
 * 将整个数组分为两个部分，然后分别排序之后再使用外排进行合并
 */
public class MergeSort {

    public static void mergeSort(int[] sourceArray) {
        if (sourceArray == null || sourceArray.length < 2) {
            return;
        }
        mergeSort(sourceArray, 0, sourceArray.length - 1);
    }

    public static void mergeSort(int[] sourceArray, int left, int right) {
        if (left == right) {
            return;
        }
        // 求数组中间点，将数组划分为两部分
        int mid = left + ((right - left) >> 1);
        mergeSort(sourceArray, left, mid);
        mergeSort(sourceArray, mid + 1, right);
        merge(sourceArray, left, mid, right);
    }

    public static void merge(int[] sourceArray, int left, int mid, int right) {
        // 准备一个和原数组等长的辅助数组；
        int[] help = new int[right - left + 1];
        int i = 0;
        int startLeft = left;
        int startRight = mid + 1;
        while (startLeft <= mid && startRight <= right) {
            help[i++] = sourceArray[startLeft] < sourceArray[startRight] ? sourceArray[startLeft++] : sourceArray[startRight++];
        }
        // 上面的 while 循环会将一个数组中元素全部挪到 help 数组中，而另个数组还会剩余最后几个元素
        // 将剩余的一个数组中剩余的元素全部移到 help 数组中，这两个 while 只会执行一个
        while (startLeft <= mid) {
            help[i++] = sourceArray[startLeft++];
        }
        while (startRight <= right) {
            help[i++] = sourceArray[startRight++];
        }
        for (i = 0; i < help.length; i++) {
            sourceArray[left + i] = help[i];
        }
    }
}
```

**示例：** 原来数组元素为：3，6，4，5，2，8；首先划分为：Left:[3,6,4]和 right:[5,2,8]；然后将他们分别排序为：Left:[3,4,6] 和 Right:[2,5,8]，然后左右各取指针 a,b，同时准备一个辅助数组，长度和原数组长度相同；然后比较 a,b 指向元素的大小，哪一个小哪一个就放进数组，同时下标 + 1，然后再次比较，直到某一方全部放入数组，则另一方剩余的全部放入数组，最后将该数组拷贝回原来数组；
因为一共 N 的样本，分为两个一样的部分，每部分的时间复杂度为 ：$T(\frac{\mathrm{N}}{2})$，每一部分进行比较等操作为：$O(N)$，因此时间复杂度为：$T(N) = 2* T({\frac{N}{2}}) + O(N)$，根据 master 公式结果为：$O(N * \log_{2}^{N})$ 
空间复杂度：$O(N)$



## 三、==对数器==

- 首先有一个你想要验证是否正确的方法 A；
- 其次需要一个已知绝对正确但是可能时间复杂度不好的方法 B；（也可以不完全正确）
- 实现一个随机样本产生器；
- 实现两个方法 A 和 B 对比的方法；
- 将方法 A 和方法 B 通过很多次验证来判断 A 是否正确；
- 如果某一个样本输出两个方法的结果不一致，可以打印出该样本，然后根据该样本分析是哪个方法出错了；
- 当进行大量样本量对比之后测试仍然正确，可以确认方法 A 是正确的；

例如验证冒泡排序是否正确：

```java
package nowcoder.easy.day01;  
  
/**  
 * @author GJXAIOU  
 * @create 2019-10-04-20:00  
 */  
import java.util.Arrays;  
  
public class BubbleSort {  
    ////////////////// 冒泡排序 /////////////////////////  public static void bubbleSort(int[] sourceArray) {  
        if (sourceArray == null || sourceArray.length < 2) {  
            return;  
        }  
        // end 刚开始在 length-1,但是得大于零，每排完一圈减一  
  for (int end = sourceArray.length - 1; end > 0; end--) {  
            for (int start = 0; start < end; start++) {  
                if (sourceArray[start] > sourceArray[start + 1]) {  
                    swap(sourceArray, start, start + 1);  
                }  
            }  
        }  
    }  
  
    public static void swap(int[] sourceArray, int left, int right) {  
        sourceArray[left] = sourceArray[left] ^ sourceArray[right];  
        sourceArray[right] = sourceArray[left] ^ sourceArray[right];  
        sourceArray[left] = sourceArray[left] ^ sourceArray[right];  
    }  
  
    ////////////////////// 使用对数器 //////////////////////// // 1.想要验证的方法，见上  
  // 2.准备一个绝对正确的方法：这里使用系统自带的排序方法  
  public static void comparator(int[] arr) {  
        Arrays.sort(arr);  
    }  
  
    // 3.实现一个随机样本产生器：这里随机生成一个任意长度，值为任意的数组  
  public static int[] generateRandomArray(int maxSize, int maxValue) {  
        // Math.random() 表示范围为： double [0,1) // (int)((maxSize + 1) * Math.random())：int [0,size]  
  int[] arr = new int[(int) ((maxSize + 1) * Math.random())];  
        for (int i = 0; i < arr.length; i++) {  
            arr[i] = (int) ((maxValue + 1) * Math.random()) - (int) (maxValue * Math.random());  
        }  
        return arr;  
    }  
  
    // 因为是原地排序，会改变原数组，所以复制一份两个算法使用  
  public static int[] copyArray(int[] arr) {  
        if (arr == null) {  
            return null;  
        }  
        int[] res = new int[arr.length];  
        for (int i = 0; i < arr.length; i++) {  
            res[i] = arr[i];  
        }  
        return res;  
    }  
  
    // 4.实现连个方法对比的方法  
  public static boolean isEqual(int[] arr1, int[] arr2) {  
        if ((arr1 == null && arr2 != null) || (arr1 != null && arr2 == null)) {  
            return false;  
        }  
        if (arr1 == null && arr2 == null) {  
            return true;  
        }  
        if (arr1.length != arr2.length) {  
            return false;  
        }  
        for (int i = 0; i < arr1.length; i++) {  
            if (arr1[i] != arr2[i]) {  
                return false;  
            }  
        }  
        return true;  
    }  
  
    // 可有可无  
  public static void printArray(int[] arr) {  
        if (arr == null) {  
            return;  
        }  
        for (int i = 0; i < arr.length; i++) {  
            System.out.print(arr[i] + " ");  
        }  
        System.out.println();  
    }  
      
    // 最后：进行大样本测试  
  public static void main(String[] args) {  
        int testTime = 500000;  
        // 长度为【0-100】  
  int maxSize = 100;  
        // 值为 【-100-100】之间  
  int maxValue = 100;  
        boolean succeed = true;  
        for (int i = 0; i < testTime; i++) {  
            int[] arr1 = generateRandomArray(maxSize, maxValue);  
            int[] arr2 = copyArray(arr1);  
            bubbleSort(arr1);  
            comparator(arr2);  
            if (!isEqual(arr1, arr2)) {  
                succeed = false;  
                break;  
            }  
        }  
        System.out.println(succeed ? "Nice!" : "Bad!");  
  
        int[] arr = generateRandomArray(maxSize, maxValue);  
        printArray(arr);  
        bubbleSort(arr);  
        printArray(arr);  
    }  
}
```





## 四、递归（Recursion）

==递归就是自己调自己==
==任何递归都可以改为非递归==
递归函数本质上就是系统在帮我们压栈，如果改为自己压栈就可以改为非递归；

**下面以一个示例解释递归过程：**





## 五、==master 公式==

一般的时间复杂度公式可以表示为：$T(N) = aT(\frac{N}{b}) + O({N}^{d}))$，其中 N 表示整个过程的样本量，$\frac{N}{b}$表示划分之后子过程的样本量，a 表示子过程共发生多少次，后面的：$O({N}^{d}))$ 表示除了子过程之外其它操作的时间复杂度，**这里的子过程只是划分一次之后的数量** ，并且划分的子问题应该规模相同；
如果一个过程的时间复杂度表示为：$T(N) = aT(\frac{N}{b}) + O({N}^{d}))$，则可以根据 master 公式化为以下结果：

- $\log_{b}^{a} < d$ ：复杂度为：$O({N}^{d})$；
- $\log_{b}^{a} = d$ ：复杂度为：$O({N}^{d} *{\log_{2}^{N}})$；
- $\log_{b}^{a} > d$ ：复杂度为：$O({N}^{log{b}^{a}})$；


### 示例

**小和问题：**
在一个数组中，每一个数左边比当前数小的数累加起来，叫做这个数组的小和。求一个数组的小和。
```java
[1,3,4,2,5]
1左边比1小的数，没有；
3左边比3小的数，1；
4左边比4小的数，1、3；
2左边比2小的数，1；
5左边比5小的数，1、3、4、2；
所以小和为1+1+3+1+1+3+4+2=16
```

**求解：**
将该数组不断的切分[1,3,4][2,5]，然后再次切分：[1,3][4][2][5],最后切分为：[1][3][4][2][5];
这里以 1 为例，[1][3]中有一个 1（即 3 的左边有一个 1 比他小），然后[1,3]和[4]，有一个 1（即 4 左边有一个 1 比他小），然后 [1,3,4]和 [2,5]，有两个 1（即 2 和 5 左边有一个 1 比它小），最终以 1 为例的小和为：1 + 1 + 2*1 = 4;其他类似；
**过程**
- a. 将当前序列分为两个子序列，分别求其小和  
- b. 对a划分得到的两个子序列进行merge操作，得到合并过程产生的小和，再加上a得到的两个子序列的小和之和  
- c. 递归地执行a和b

merge操作采用二路归并排序的思想  
求一个数组的小和，可以转化为求每个元素在小和累加过程出现的次数，然后将当前元素与出现次数相乘，累加得到小和  
假设当前元素为a，a右边比a大的元素个数则为a在小和累加过程出现的次数

```java
package nowcoder.easy.day01;  
  
/**  
 * 小和问题  
  * 解决思路：  
  * 1.借助于归并排序，首先将原数组分为两部分  
  * 2.对划分的两个子序列进行 merge 操作，得到合并过程重产生的小和，再加上前面两个子序列的小和  
  * 3.递归的执行 1，2 步骤。  
  */  
public class SmallSum {  
  
   public static int smallSum(int[] sourceArray) {  
      if (sourceArray == null || sourceArray.length < 2) {  
         return 0;  
      }  
      return mergeSort(sourceArray, 0, sourceArray.length - 1);  
   }  
  
   public static int mergeSort(int[] sourceArray, int left, int right) {  
      if (left == right) {  
         return 0;  
      }  
      int mid = left + ((right - left) >> 1);  
      // 最终的小和数目为分开数组分别求得的小和加上合并之后的小和  
  return mergeSort(sourceArray, left, mid) + mergeSort(sourceArray, mid + 1, right) + merge(sourceArray, left, mid, right);  
   }  
  
   public static int merge(int[] arr, int left, int mid, int right) {  
      int[] help = new int[right - left + 1];  
      int i = 0;  
      int startLeft = left;  
      int startRight = mid + 1;  
      int res = 0;  
      while (startLeft <= mid && startRight <= right) {  
         // 因为两个序列都是按照从小到大排序好的，所以一旦 startLeft　＜　startRight　则相当于 arr[startLeft]　是　startRight + 1　～　right　之间所有元素的小和  
  res += arr[startLeft] < arr[startRight] ? (right - startRight + 1) * arr[startLeft] : 0;  
         help[i++] = arr[startLeft] < arr[startRight] ? arr[startLeft++] : arr[startRight++];  
      }  
      while (startLeft <= mid) {  
         help[i++] = arr[startLeft++];  
      }  
      while (startRight <= right) {  
         help[i++] = arr[startRight++];  
      }  
      for (i = 0; i < help.length; i++) {  
         arr[left + i] = help[i];  
      }  
      return res;  
   }  
  
   // --------对数器-----------  
 // 这里仅仅保留作为比较的绝对正确的方法，其它例如随机数产生器和比较器和上面一样  
  public static int comparator(int[] arr) {  
      if (arr == null || arr.length < 2) {  
         return 0;  
      }  
      int res = 0;  
      for (int i = 1; i < arr.length; i++) {  
         for (int j = 0; j < i; j++) {  
            res += arr[j] < arr[i] ? arr[j] : 0;  
         }  
      }  
      return res;  
   }  
    
}
```


**逆序对问题**
在一个数组中，左边的数如果比右边的数大，则折两个数构成一个逆序对，请打印所有逆序对。




