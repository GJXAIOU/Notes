# AlgorithmEasyDay02

[TOC]



## 一、荷兰国旗类问题（数组划分）

### （一）问题一：按给定值分割数组
给定一个数组arr，和一个数num，请把小于等于num的数放在数组的左边，大于num的数放在数组的右边。
要求额外空间复杂度O(1)，时间复杂度O(N)；
![示例](AlgorithmEasyDay02.resource/%E7%A4%BA%E4%BE%8B.png)

其中 x 坐标为 Left - 1，<=x 位置上放置的都是 <= num 值的数，然后依次向右遍历，如果该数大于 num，则 x 不动，直接判断下一个数，如果该数小于等于 num，则该数和 x + 1 位置上的数互换， x 向右移动一位，即 x + 1，然后继续判断下一个数；以此类推；

```java
package nowcoder.easy.day01;  
  
import java.util.Arrays;  
  
/**  
 * @author GJXAIOU  
 * @create 2019-10-07-15:03  
 * <p>  
 * 分割数组  
  * 注：该算法会调整数组中元素顺序，所以不能直接和对数器逐个值比较  
  */  
public class SplitArray {  
    /**  
 * @param sourceArray 输入的原数组  
  * @param left 需要判断的数组左区间下标：默认为 0  
 * @param right 需要判断的数组左区间下标：默认为 sourceArray.length - 1  
 * @param tagMum 目标值  
  */  
  public static void sort(int[] sourceArray, int left, int right, int tagMum) {  
        int less = left - 1;  
        while (left <= right) {  
            if (sourceArray[left] <= tagMum) {  
                // 因为 less 的下一位就是大于 tagMum 的数  
  // 如果没有这个判断，则如果数组最后一位则不会处理  
  if (left == right) {  
                    swap(sourceArray, left, ++less);  
                }  
                swap(sourceArray, left++, ++less);  
            } else {  
                left++;  
            }  
        }  
        return;  
    }  
  
    public static void swap(int[] sourceArray, int left, int right) {  
        if (left == right) {  
            return;  
        }  
        sourceArray[left] = sourceArray[left] ^ sourceArray[right];  
        sourceArray[right] = sourceArray[left] ^ sourceArray[right];  
        sourceArray[left] = sourceArray[left] ^ sourceArray[right];  
    }  
  
    // 测试方法  
  public static void main(String[] args) {  
        int[] sourceArray = {1, 2, 8, -2, 4, 3, 4, 2, 8, 12, 3, 9, 8, 10, 9, 5, -1, 4, 9, 2};  
        int[] sourceArray2 = copyArray(sourceArray);  
        int left = 0;  
        int right = sourceArray.length - 1;  
        int tagMum = 6;  
  
        System.out.println("原数组为：");  
        for (int i : sourceArray) {  
            System.out.print(i + " ");  
        }  
  
        sort(sourceArray, left, right, tagMum);  
  
        System.out.println("\n分割之后数组为：");  
        for (int i = 0; i < sourceArray.length; i++) {  
            System.out.print(sourceArray[i] + " ");  
        }  
  
        System.out.println("\n---对数器输出结果-----");  
        compare(sourceArray2, 6);  
        for (int i = 0; i < sourceArray2.length; i++) {  
            System.out.print(sourceArray2[i] + " ");  
        }  
  
        boolean equal = isEqual(sourceArray, sourceArray2, tagMum);  
        System.out.println("\n" + equal);  
    }  
  
    // ---------对数器---------------------  
 // 思路：遍历数组中每一个元素，小于等于 tagNum 放置于一个数组，大于 tagNum 放置于一个数组，最后将两个数组进行合并，可以保证和数组原顺序一致  
  public static void compare(int[] sourceArray, int tagNum) {  
        int[] lessArr = new int[sourceArray.length];  
        int[] moreArr = new int[sourceArray.length];  
        int less = 0;  
        int more = 0;  
        for (int i = 0; i < sourceArray.length; i++) {  
            if (sourceArray[i] <= tagNum) {  
                lessArr[less++] = sourceArray[i];  
            } else {  
                moreArr[more++] = sourceArray[i];  
            }  
        }  
        for (int i = 0; i < more; i++) {  
            lessArr[less++] = moreArr[i];  
        }  
        for (int i = 0; i < lessArr.length; i++) {  
            sourceArray[i] = lessArr[i];  
        }  
        return;  
    }  
  
  
    public static int[] copyArray(int[] sourceArray) {  
        int[] res = new int[sourceArray.length];  
        if (sourceArray == null || sourceArray.length == 0) {  
            return res;  
        }  
        for (int i = 0; i < sourceArray.length; i++) {  
            res[i] = sourceArray[i];  
        }  
        return res;  
    }  
  
    // 依次比较两个数组在相同位置上元素和 tagNum 的关系  
  public static boolean isEqual(int[] arr1, int[] arr2, int tagNum) {  
        boolean[] booleanArr1 = new boolean[arr1.length];  
        boolean[] booleanArr2 = new boolean[arr2.length];  
        for (int i = 0; i < arr1.length; i++) {  
            booleanArr1[i] = (arr1[i] <= tagNum);  
            booleanArr2[i] = (arr2[i] <= tagNum);  
        }  
        return Arrays.equals(booleanArr1, booleanArr2);  
    }  
}
```
程序运行结果为：
```java
原数组为：
1 2 8 -2 4 3 4 2 8 12 3 9 8 10 9 5 -1 4 9 2 
分割之后数组为：
1 2 -2 4 3 4 2 3 5 -1 4 2 9 10 9 8 12 8 9 8 
---对数器输出结果-----
1 2 -2 4 3 4 2 3 5 -1 4 2 8 8 12 9 8 10 9 9 
true
```

### （二）问题二：荷兰国旗问题
给定一个数组arr，和一个数num，请把小于num的数放在数组的左边，等于num的数放在数组的中间，大于num的数放在数组的右边。
要求额外空间复杂度O(1)，时间复杂度O(N)
![示例2](AlgorithmEasyDay02.resource/%E7%A4%BA%E4%BE%8B2.png)

x 坐标为 L - 1，y 坐标为 R + 1，两边分别表示小于 num 和大于 num 的值，当前位置坐标为 cur，然后依次向右遍历，如果该数小于 num，则该数和小于区域最右边下标（x）的下一个坐标元素交换，小于区域向右扩充（即 x + 1），如果该数等于 num ,则 cur 指向下一个元素，如果大于 num，则该数和大于区域最左边区域的前一个坐标元素交换，大于区域向左扩充一个（即 y - 1），然后这里**交换回来的数还需要按照上面的标准进行判断**，直到 cur 和 又边界相遇停止；

```java
package nowcoder.easy.day01;  
  
/**  
 * @author GJXAIOU  
 * 荷兰国旗问题  
  */  
public class NetherlandsFlag {  
    /**  
 * @param sourceArray：要分割的数组  
  * @param left ：数组左边界  
  * @param right：数组右边界  
  * @param tagNum：用于分割的参考数字  
  * @return  
  */  
  public static int[] partition(int[] sourceArray, int left, int right, int tagNum) {  
        // less 表示小于 tagNum 区域最右边数，more 是大于 tagNum 区域最左边数  
  int less = left - 1;  
        int more = right + 1;  
        while (left < more) {  
            if (sourceArray[left] < tagNum) {  
                swap(sourceArray, ++less, left++);  
            } else if (sourceArray[left] > tagNum) {  
                // 从大于区域换过来的值还需要再次判断  
  swap(sourceArray, --more, left);  
            } else {  
                left++;  
            }  
        }  
        return new int[]{less + 1, more - 1};  
    }  
  
    public static void swap(int[] sourceArray, int left, int right) {  
        int tmp = sourceArray[left];  
        sourceArray[left] = sourceArray[right];  
        sourceArray[right] = tmp;  
    }  
  
  
    // for test  
  public static int[] generateArray() {  
        int[] arr = new int[20];  
        for (int i = 0; i < arr.length; i++) {  
            arr[i] = (int) (Math.random() * 10);  
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
  
    public static void main(String[] args) {  
        int[] test = generateArray();  
        System.out.println("测试数据为：");  
        printArray(test);  
  
        int[] res = partition(test, 0, test.length - 1, 5);  
        System.out.println("划分之后数据为：值为原地修改");  
        printArray(test);  
  
        System.out.println("等于区域的开始下标为：");  
        System.out.println(res[0]);  
        System.out.println("等于区域的结束下标为：");  
        System.out.println(res[1]);  
    }  
}
```
随机输入结果为：
```java
测试数据为：
7 4 4 6 3 6 1 4 7 2 8 3 4 1 2 1 5 3 0 3 
划分之后数据为：值为原地修改
3 4 4 0 3 3 1 4 2 1 3 4 1 2 5 8 7 6 6 7 
等于区域的开始下标为：
14
等于区域的结束下标为：
14
```


## 二、快速排序
### （一）经典快排
首先以数组最后一个数值为基准，将小于等于该数值的全部放在数组前半部分，大于该数值的全部放在数组的后半部分，然后前半部分和后半部分分别以该部分最后一个元素为基准重复以上步骤；

**改进**：使用荷兰国旗思想
首先还是选取数组最后一个值为基准，但是遍历判断的时候最后一个值不再进行判断，即只有 L ~ R - 1 的值与基准进行比较，同时区域划分为三个区域：小于基准、等于基准、大于基准；然后将小于基准和大于基准部分进行再次取最后一个数，同上进行比较.....最后全部比较结束之后，将最后一个基准值放在大于该基准的范围的前一个位置；


特点：
- 经典快排和数据的状态有关：
  - 当小于最后一个数值的元素远大于大于最后一个数组的元素个数时候，或者反之情况，时间复杂度都是：$O({N}^{2})$
  - 如果数据状态较好，即大于和小于差不多的情况下，时间复杂度为：$T(N) = 2T(\frac{N}{2}) + O(N) = O(N * log_{2}^{N})$

经典快排的空间复杂度为：$O(N)$

### （二）随机快速排序
通过随机选一个数和最后一个数进行互换，使得每次划分标准都在改变；
根据随机性，随机快速排序的时间复杂度是：$O(N * \log_{2}^{N})$，同时需要空间复杂度为：$O(\log_{2}^{N})$，这里的额外空间主要用于记录每次划分区域的断点；

```java
package nowcoder.easy.day01;  
  
/**  
 * @author GJXAIOU  
 * @create 2019-10-04-20:08  
 * <p>  
 * 随机快排  
  */  
  
import java.util.Arrays;  
  
public class QuickSort {  
    /**  
 * 首先调用该方法，可以设置排序的区间，默认为 0 ~ length-1；  
  *  
 * @param sourceArray：需要排序的数组  
  */  
  public static void quickSort(int[] sourceArray) {  
        if (sourceArray == null || sourceArray.length < 2) {  
            return;  
        }  
        quickSort(sourceArray, 0, sourceArray.length - 1);  
    }  
  
    /**  
 * @param sourceArray：需要排序的数组  
  * @param left：排序数组左边界，一般为：0  
 * @param right：排序数组右边界，一般为：length - 1;  
 * less：小于参照元素区域的最右边边界：less = p[0] - 1;  
 * more：大于参照元素区域的最左边边界：more = p[1] + 1;  
 * p[0]：等于参照元素区域的最左边边界；  
  * p[1]：等于参数元素区域的最右边边界；  
  * 小于参照元素区域：[Left ~ less];  
 * 等于参照元素区域：[p[0] ~ p[1]]；  
  * 大于参照元素区域：[more ~ right]；  
  */  
  public static void quickSort(int[] sourceArray, int left, int right) {  
        if (left < right) {  
            swap(sourceArray, left + (int) (Math.random() * (right - left + 1)), right);  
            // p 数组中： p[0] 表示等于区域的左边界，p[1] 表示等于区域的右边界，  
  // 左边区域：L ~ p[0] - 1;右边区域： p[1] + 1 ~ R;  int[] p = partition(sourceArray, left, right);  
            quickSort(sourceArray, left, p[0] - 1);  
            quickSort(sourceArray, p[1] + 1, right);  
        }  
    }  
  
    public static int[] partition(int[] sourceArray, int left, int right) {  
        int less = left - 1;  
        int more = right;  
        while (left < more) {  
            // 以数组最后一个元素为标准，将整个数组划分为 小于、等于、大于 三个部分  
  if (sourceArray[left] < sourceArray[right]) {  
                swap(sourceArray, ++less, left++);  
            } else if (sourceArray[left] > sourceArray[right]) {  
                swap(sourceArray, --more, left);  
            } else {  
                left++;  
            }  
        }  
        swap(sourceArray, more, right);  
        return new int[]{less + 1, more};  
    }  
  
    public static void swap(int[] sourceArray, int left, int right) {  
        int tmp = sourceArray[left];  
        sourceArray[left] = sourceArray[right];  
        sourceArray[right] = tmp;  
    }  
  
  
    public static void main(String[] args) {  
        int[] arr = {43, -31, 10, -38, -42, -2, 22, 29, 30, 15, -60, -50, -13, 26, 3, 22, 27, 24, 18, 18, 42, -40, 22, 8, 33, -52, -70, -55, 31, 42, 82, 19, -8, 8, 41, -35, 59, 65, -23, 3, -34, 65};  
        System.out.println("原数组为：");  
        for (int i = 0; i < arr.length; i++) {  
            System.out.print(arr[i] + "  ");  
        }  
  
        quickSort(arr);  
        System.out.println("\n排序后数组为：");  
        for (int i = 0; i < arr.length; i++) {  
            System.out.print(arr[i] + "  ");  
        }  
    }  
  
}
```
程序运行结果：
```java
原数组为：
43  -31  10  -38  -42  -2  22  29  30  15  -60  -50  -13  26  3  22  27  24  18  18  42  -40  22  8  33  -52  -70  -55  31  42  82  19  -8  8  41  -35  59  65  -23  3  -34  65  
排序后数组为：
-70  -60  -55  -52  -50  -42  -40  -38  -35  -34  -31  -23  -13  -8  -2  3  3  8  8  10  15  18  18  19  22  22  22  24  26  27  29  30  31  33  41  42  42  43  59  65  65  82 
```


## 三、堆排序

### （一）堆
- **堆是一个完全二叉树**，可以采用数组进行实现；
- 对于完全二叉树，结点 i 的左孩子序号为：2i + 1；右孩子序号为：2i + 2；父结点的序号为：$\frac{i - 1}{2}$。

- 分类：
	- 大根堆：每棵树（包括任意一棵子树）的最大值都是其头部（父结点）；
	- 小根堆：每棵树（包括任意一棵子树）的最小值都是其头部（父结点）；

**完全二叉树**：对一棵具有 n 个节点的二叉树按照层序进行编号，如果编号 i （1<=i<=n）的结点与同样深度的满二叉树中编号为 i 的结点位置完全相同；
**满二叉树**：所有的分支结点都存在左子树和右子树，且所有的叶子都在同一层；

### （二）数组转换为大根堆
以数组 [2,1,3,6,0,4]为例：
首先取出第一元素 2 作为头结点，然后取出第二个元素 1，该元素比 2 小，放在左孩子位置，数组元素为：[2,1]；然后取出第三个元素 3，计算该元素的父结点：$\frac{2 - 1}{2} = 0$，则与 0 位置元素 2 进行比较，发现 3 比较大，则在数组中将 3 和 2 两个元素互换，则数组变为：[3,1,2]；然后取出第四个元素 6，计算父结点下标为：$\frac{3 - 1}{2} = 1$，则与 1 位置的元素 1 进行比较，发现 6 比较大，则将元素 1 和元素 6 互换，得到[3,6,2,1]，然后再比较元素 6 和其父结点：$\frac{1 - 1}{2} = 0$，比较得出 6 比 3 大，然后再换，最后得到数组为：[6,3,2,1]，剩下元素依次类推......

**每加入一个节点其最多比较的次数和已经形成的二叉树高度有关**（因为每次只和其父结点比较），因此最多时间复杂度为：$O(log_{2}^{N})$，所有整个转换过程时间复杂度为：$log_{2}^{1} + log_{2}^{2} + ..... + log_{2}^{N} = O(N)$


**题目**
吐泡泡：一个 XXX 会不停的吐出数字，求任意时刻的已经吐出的所有元素的中位数；

**解答：**
这里需要同时使用大根堆和小根堆，大根堆中存放着较小的 $\frac{N}{2}$个元素，小根堆中存放较大的  $\frac{N}{2}$个元素；

这里以：5 4 6 7 为例
首先将 5 放入大根堆，计算大根堆和小根堆的 Heapsize，差值为 1,不动， 然后因为 4 小于等于大根堆的堆顶，因此放入大根堆，再次计算 HeapSize，插值 > 1，然后将大根堆的堆顶放在小根堆，接着将大根堆剩余的调整为大根堆，然后元素 6 大于大根堆现在堆顶 4，因此放入小根堆；现在大根堆为 4，小根堆为 5,6；插值为 1，然后放入元素 7，同样大于大根堆堆顶，放入小根堆，然后插值> 1，将小根堆的堆顶放入大根堆末尾，小根堆重新排为：6,7；大根堆重新排为：5,4；中位数就是两个堆顶的平均值；


### （三）堆排序
首先将数组变成大根堆；
然后将堆中最后一个和堆顶进行交换，堆大小减一，则最后一个不动了，然后将剩下的前面进行 Heapify 调整；
然后再将堆的最后一个和堆顶进行交换，同上.....;

```java
package sort.nowcoder.easy.day01;

/**
 * @author GJXAIOU
 * @create 2019-10-04-20:08
 */
import java.util.Arrays;

public class HeapSort {

    public static void heapSort(int[] arr) {
        if (arr == null || arr.length < 2) {
            return;
        }
        // 首先将数组转化为大根堆；0 到  i 之间形成大根堆
        for (int i = 0; i < arr.length; i++) {
            heapInsert(arr, i);
        }

        // 不断将堆顶的元素和最后一个元素交换然后进行 heapify 过程
        int size = arr.length;
        swap(arr, 0, --size);
        while (size > 0) {
            heapify(arr, 0, size);
            swap(arr, 0, --size);
        }
    }

    public static void heapInsert(int[] arr, int index) {
        // 如果插入的新节点值大于其父结点大小
        while (arr[index] > arr[(index - 1) / 2]) {
            swap(arr, index, (index - 1) / 2);
            index = (index - 1) / 2;
        }
    }

    /**
     * size - 1 到 length - 1 位置上已经拍好
     * @param arr：要排序的数组
     * @param index：哪个节点位置上元素发生了变化，传入的初始值一直为0
     * @param size：还没有排好序的数组长度
     */
    public static void heapify(int[] arr, int index, int size) {
        // size 表示当前堆上节点数
        int left = index * 2 + 1;
        // 越界表示已经是叶子结点了
        while (left < size) {
            int largest = left + 1 < size && arr[left + 1] > arr[left] ? left + 1 : left;
            largest = arr[largest] > arr[index] ? largest : index;
            if (largest == index) {
                break;
            }
            swap(arr, largest, index);
            index = largest;
            left = index * 2 + 1;
        }
    }

    public static void swap(int[] arr, int left, int right) {
        int tmp = arr[left];
        arr[left] = arr[right];
        arr[right] = tmp;
    }
}    
```


**概念：**
- HeapInsert：将新的节点加入堆中同时按照堆的结构进行向上调整的过程；
-  HeapSize：堆的大小；对应数组中就是 0-i 位置;
- Heapify：假设堆中（数组中）某个值发生了变化，让其整体再次调整为大根堆（或小根堆）原来的样子的过程；
  首先找到变化的值的两个孩子，然后找到其中较大的一个与之交换，如果有交换，在新的位置上再次找现在的两个孩子进行比较，然后交换，一直到没有交换为止；
- 堆减小的过程：以大根堆为例：首先将堆顶元素和堆的最后一个元素位置互换，这样原来堆顶的元素就放在了数组的最后，然后将堆的 Heapsize - 1，这样最后一个元素就应为超过了现在的 HeapSize 而越界，从而失效；然后将剩余的 0 ~ i - 1 位置的按照 Heapify 重新进行调整（因为堆顶的元素值发生了变化）为原来的堆结构；


## 四、排序算法的稳定性

首先总结现有排序算法稳定性
算法名称 | 时间复杂度 | 算法种类 | 是否稳定 | 原因
---|---|---|---|---
冒泡排序|$O({N}^{2})$|基于比较|稳定 |在冒泡的时候，如果遇到前后相同的值两者不交换即可，只有前者比后者大才交换；
插入排序|$O({N}^{2})$|基于比较|稳定 |同样在比较的时候，相同的值不交换即可；
选择排序|$O({N}^{2})$|基于比较| 不稳定 |因为如果后面有小于前面的，就和前面的互换，如果有几个相同数，则相当于和最前面的数进行互换，这样顺序就乱了；
归并排序|$O(N * log_{2}^{N})$ |基于比较  |稳定 |以为归并之前是左右两个数组，左边数组在原数组中就是在左边，右边数组原来就是右边，这样只需要如果左右两个数组中有相同的数字，则只需要先拷贝左边数组值，然后拷贝右边数组中值即可；
快速排序 | $O(N * log_{2}^{N})$ |基于比较  |不稳定（也可以稳定） |因为 partition 过程就是交换，肯定是无序的；
堆排序 | $O(N * log_{2}^{N})$ |基于比较  |不稳定 | 因为在形成大根堆的时候，叶子结点与根节点进行交换的时候就会序号乱，例如：2,2,3；当放入 3 的时候，两个 2 的顺序就改变了；   
桶排序|$O(N)$|非基于比较 |稳定 |   
基数排序|$O(N)$|非基于比较 |稳定 |   
计数排序|$O(N)$|非基于比较 |稳定 |   


**排序问题补充：**
- 归并排序的空间复杂度可以变成 O(1)，可以采用 “归并排序 内部缓存法”实现，但是仅仅要求了解即可；
- 快速排序可以做到稳定性，采用“01 stable sort”；
- 荷兰国旗问题不可能稳定，因为明显存在交换；
问题：将一个数组的奇数放在数组左边，偶数放在数组右边，并且要求原始的相对次序不变，时间复杂度要求：O(N)，空间复杂度要求：O(1)；
**解析：**   因为每一个数不是奇数就是偶数，因此也是可以抽象为一个 0 1 问题，相当于把 0 类（例如 < 0.5 的，这里 0.5 是随便取，就是为了区分）的放在左边，把大于 0.5 的放在右边，即 1 类 ；且保证原来的相对顺序不变，抽象就是快排的 partition 过程保证稳定；因为 partition 过程就是将一个数组分为 <= 和 > 两个部分，也是 0  1 过程，如果上述满足就可以实现快排稳定；只能采用 01 stable sort 解决；
  

## 五、认识比较器
比较器作用：自己实现比较自己定义的对象的方法，然后通过将其传入**系统中有序的结构**就可以处理自己定义类型的比较；**就是自己只需要实现自定义比较规则**

例如：使用优先级队列（实质上就是堆）存放自定义对象，然后自定义比较器使得可以比较自定义的类型对象；

```java
package sort.nowcoder.easy.day01;

import java.util.Arrays;
import java.util.Comparator;
import java.util.PriorityQueue;

public class MyComparator {

	// 使用了内部类
	public static class Student {
		public String name;
		public int id;
		public int age;

		public Student(String name, int id, int age) {
			this.name = name;
			this.id = id;
			this.age = age;
		}
    }

	// 重载比较器
	public static class IdAscendingComparator implements Comparator<Student> {
		@Override
		public int compare(Student o1, Student o2) {
            // 返回值：负数：前面的放在前面，整数：后面的放在前面，0：两者相等；
			return o1.id - o2.id;
            /**
             * 上面的 return 等价于
             * if(o1.id < o2.id){
             *      return -1;
             * }else if(o1.id > o2.id){
             *     return 1;
             * }else{
             *     return 0;
             * }
             */
		}

	}

	// 按照 id 降序排列
	public static class IdDescendingComparator implements Comparator<Student> {
		@Override
		public int compare(Student o1, Student o2) {
			return o2.id - o1.id;
		}

	}

	public static class AgeAscendingComparator implements Comparator<Student> {
		@Override
		public int compare(Student o1, Student o2) {
			return o1.age - o2.age;
		}

	}

	public static class AgeDescendingComparator implements Comparator<Student> {
		@Override
		public int compare(Student o1, Student o2) {
			return o2.age - o1.age;
		}

	}

	public static void printStudents(Student[] students) {
		for (Student student : students) {
			System.out.println("Name : " + student.name + ", Id : " + student.id + ", Age : " + student.age);
		}
		System.out.println("===========================");
	}

	public static void main(String[] args) {
		Student student1 = new Student("A", 1, 23);
		Student student2 = new Student("B", 2, 21);
		Student student3 = new Student("C", 3, 22);

		Student[] students = new Student[] { student3, student2, student1 };
		printStudents(students);

		Arrays.sort(students, new IdAscendingComparator());
		printStudents(students);

		Arrays.sort(students, new IdDescendingComparator());
		printStudents(students);

		Arrays.sort(students, new AgeAscendingComparator());
		printStudents(students);

		Arrays.sort(students, new AgeDescendingComparator());
		printStudents(students);


        // 使用系统提供的堆：优先级队列进行排序： TreeMap 实现
        PriorityQueue<Student> heap =  new PriorityQueue<>(new IdAscendingComparator());
        // 添加自定义的类型
        heap.add(student1);
        heap.add(student2);
        heap.add(student3);

		System.out.println("==========优先级队列按照 Id 排序=================");
        while (!heap.isEmpty()){
            // 逐个弹出栈顶，内部实现就是 heapify
            Student student = heap.poll();
            System.out.println("Name : " + student.name + ", Id : " + student.id + ", Age : " + student.age);
        }
	}
}

```


## 六、非基于比较的排序
### （一）桶排序
桶排序仅仅是一种概念，整体思想是首先记录数据各状况出现的词频，然后根据词频进行还原从而达到排序目的；
它的具体实现有：计数排序、基数排序；


### （二）计数排序
**有多少个元素就需要多少个桶；**
示例：有一个元素值范围为：0 ~ N 的数组，将其排序；
步骤：首先准备一个长度为 N + 1 (数组最大值 + 1 )的辅助数组；辅助数组下标分别为：0 ~ N；
然后遍历原数组，有一个 X 值（大小位于 0~N 之间），就在辅助数组下标为 X 的对应元素值 + 1；一直遍历结束；
最后将辅助数组中各个下标对应的元素值还原，示例：辅助数组为：`[1,2,0,2]`就相当于有 1 个 0,2 个 1,0 个 3,2 个 4，因此结果为：`[0,1,1,4,4]`；

```java
package sort.nowcoder.easy.day01;

import java.util.Arrays;

/**
 * 这里是使用计数排序实现桶排序思想
 */
public class BucketSort {
	private static final int WITHOUT_SORT_LENGTH = 2;
	public static void bucketSort(int[] arr) {
		if (arr == null || arr.length < WITHOUT_SORT_LENGTH) {
			return;
		}
		// 首先找到要排序数组中的最大值
		int max = Integer.MIN_VALUE;
		for (int i = 0; i < arr.length; i++) {
			max = Math.max(max, arr[i]);
		}
		// 新建 max + 1 个桶，然后遍历原数组，数组中元素值为X，则 X号桶中值 + 1；
		int[] bucket = new int[max + 1];
		for (int i = 0; i < arr.length; i++) {
			bucket[arr[i]]++;
		}
		// 遍历将 bucket 中按照个数进行还原
		int i = 0;
		for (int j = 0; j < bucket.length; j++) {
			while (bucket[j]-- > 0) {
				arr[i++] = j;
			}
		}
	}
}	
```


### （三）基数排序

**补充示例：** 给定一个数组，求如果排序之后相邻两个元素的最大差值，要求时间复杂度为 O(N)，且不能用非基于比较的排序；
**解答：**
- 思想：借用桶的思想，但是不使用桶排序；
- 思路：
  - 准备桶，原数组中有 N 个元素，因此准备 N+1 个桶；
  - 遍历原数组，找到原数组中的最小值和最大值，分别放在第 0 号桶和第 N 号桶中；如果最大值等于最小值，直接返回 0 结束；
  - 将新数组（桶）的 0 ~ N 部分等分为 N+ 1 份，原数组中值属于哪一个部分就放在哪一个桶中；示例：如果原数组一共 9 个数，则准备 10 个桶，且第一个桶中放的是数组最小值 0（假定），最后一个桶放的是最大值 99（假定），则将 0 ~ 99 等分为 10 份，则原数组中出现 0 ~ 9 直接的数放在 0 号桶，出现 10 ~ 19 之间的数放在 1 号桶。。。；
  - 每个桶只保留三个值：一个 Boolean 值，用于判断该桶中是否有元素，一个 min，表示桶中的最小值，一个 max ，表示桶中的最大值；因此如果元素 X 进入 7 号桶，如果 7 号桶之前没有元素，则首先将 Boolean 值置为 true，然后 min = x，max = x；当又一个元素进入 7 号桶的时候，比较桶内元素的值，更新最大值和最小值，其他值扔掉；
  - 最后遍历所有的桶，如果遇到空桶，跳到下一个进行判断，如果是非空桶，找到其左边最近的非空桶，将后一个非空的 min - 前一个非空的 max，插值进行保存，然后比较所有的插值，取最大的就是最大插值，

- 原理：因为 0 号桶非空，N 号桶非空，但是只有 N 个数，因此**中间至少有一个桶是空的**，同时任何两个相邻的数可以来自于同一个桶，也可能来自于不同的桶；
  - 为什么要设置一个空桶：因为至少有一个桶为空，则距离空桶左右最近的两个非空桶：左非空 min .... 左非空 max 。。。空桶 。。。右非空 min....右非空 max，则右非空 min - 左非空 max 的插值一定大于桶内插值，因为其值至少是一个桶的长度，而同一个桶内元素之间的插值是不会大于桶长的， 为了证明：**最大的插值一定不会来自于同一个桶**。**但是空桶仅仅是用于否定最终答案不是在同一个桶中，但是不是答案一定就是在空桶的两边；**示例：非空：13,19；空；非空：30，39；非空：59,63；不是空桶左右俩个的插值最大；

```java
package sort.nowcoder.easy.day01;

import java.util.Arrays;

public class MaxGap {
	private static final int WITHOUT_SORT_LENGTH = 2;
	public static int maxGap(int[] nums) {
		if (nums == null || nums.length < WITHOUT_SORT_LENGTH) {
			return 0;
		}
		int len = nums.length;
		int min = Integer.MAX_VALUE;
		int max = Integer.MIN_VALUE;
		// 找到数组中的最大值和最小值
		for (int i = 0; i < len; i++) {
			min = Math.min(min, nums[i]);
			max = Math.max(max, nums[i]);
		}
		if (min == max) {
			return 0;
		}
		// 下面三个数组是描述 len + 1 个桶中每个桶的三个必备信息
		boolean[] hasNum = new boolean[len + 1];
		int[] maxs = new int[len + 1];
		int[] mins = new int[len + 1];
		int bid = 0;
		for (int i = 0; i < len; i++) {
		    // 确定该数去第几号桶
			bid = bucket(nums[i], len, min, max);
			// 该桶中的三个信息进行更新
			mins[bid] = hasNum[bid] ? Math.min(mins[bid], nums[i]) : nums[i];
			maxs[bid] = hasNum[bid] ? Math.max(maxs[bid], nums[i]) : nums[i];
			hasNum[bid] = true;
		}
		// 找到每一个非空桶和离他最近的非空桶的插值：用当前min - 前一个max；
		int res = 0;
		int lastMax = maxs[0];
		int i = 1;
		for (; i <= len; i++) {
			if (hasNum[i]) {
				res = Math.max(res, mins[i] - lastMax);
				lastMax = maxs[i];
			}
		}
		return res;
	}

	public static int bucket(long num, long len, long min, long max) {
		return (int) ((num - min) * len / (max - min));
	}
}
```


##  七、工程中的综合排序算法

- 首先会判断数组的长度（一般界限为 60）；
  - 如果数组长度较短，一般使用插入排序，虽然插入排序的时间复杂度为：$O({N}^{2})$ 但是因为数据量较小，因此 $O({N}^{2})$ 比 $log_{2}^{N}$不会差距很大，但是因为插入排序的常数项很低，因此整体的时间复杂度较低；
  - 如果数组长度较长
    - 首先判断数组中装的数据类型
      - 如果是基础数据类型：使用快排，因为其相同值没有区分，因此不必考虑稳定性；
      - 如果是自定义数据类型：使用归并排序，因为即使相同值也是有区别的，要保证稳定性；
    - 然后如果使用快排的话，因为快排使用分治划分的思想，因此在递归的时候如果划分多次之后数组长度减少到一定长度（例如 60），则直接使用插入排序；
