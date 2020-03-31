# AlgorithmEasyDay07

[TOC]

## 一、前缀树

字典树又称为前缀树或Trie树，是处理字符串常见的数据结构。假设组成所有单词的字符仅是“a”~”z”，请实现字典树结构，并包含以下四个主要功能：

- void insert(String word)：添加word，可重复添加。
- void delete(String word)：删除word，如果word添加过多次，仅删除一次。
- boolean search(String word)：查询word是否在字典树中。
- int prefixNumber(String pre)：返回以字符串pre为前缀的单词数量。

思考：

字典树的介绍。字典树是一种树形结构，优点是利用字符串的公共前缀来节约存储空间。

**字典树的基本性质如下：**

- 根节点没有字符路径。除根节点外，每一个节点都被一个字符路径找到。
- 从根节点到某一节点，将路径上经过的字符连接起来，为扫过的对应字符串。
- 每个节点向下所有的字符路径上的字符都不同。

**在字典树上搜索添加过的单词的步骤为：**

1. 从根节点开始搜索。
2. 取得要查找单词的第一个字母，并根据该字母选择对应的字符路径向下继续搜索。
3. 字符路径指向的第二层节点上，根据第二个字母选择对应的字符路径向下继续搜索。
4. 一直向下搜索，如果单词搜索完后，找到的最后一个节点是一个终止节点，说明字典树中含有这个单词，如果找到的最后一个节点不是一个终止节点，说明单词不是字典树中添加过的单词。如果单词没搜索完，但是已经没有后续的节点，也说明单词不是字典树中添加过的单词

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

将字符串不断的加入前缀树中，**默认刚开始树的结构只有头结点**。然后在加入新字符串时候，判断当前有没有通向该结点的路径，如果没有就新建否则就复用。

![生成前缀树](AlgorithmEasyDay07.resource/%E7%94%9F%E6%88%90%E5%89%8D%E7%BC%80%E6%A0%91.png)

**示例问题**：

- 一个字符串类型的数组 arr1，另一个字符串类型的数组 arr2，arr2 中有哪些字符是在 arr1 中出现的？请打印。

- arr2 中有哪些字符是作为 arr1 中某个字符串前缀出现的？请打印。

- arr2 中有哪些字符是作为 arr1 中某个字符串前缀出现的？请打印 arr2 中出现次数最大的前缀。



代码

```java
package com.gjxaiou.easy.day07;

public class TrieTree {
    /**
     * 步骤一：生成前缀树
     */
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

    // 创建节点过程
    public static class Trie {
        private TrieNode root;

        // 创建头结点
        public Trie() {
            root = new TrieNode();
        }

        // 插入
        public void insert(String word) {
            if (word == null) {
                return;
            }
            char[] chs = word.toCharArray();
            TrieNode node = root;
            int index = 0;
            // 遍历整个字符串
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

        // 以 pre 为前缀个数
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

一块金条切成两半，是需要花费和长度数值一样的铜板的。比如长度为 20 的金条，不管切成长度多大的两半，都要花费 20 个铜板。一群人想整分整块金条，怎么分最省铜板？ 例如,给定数组{10, 20, 30}，代表一共三个人，整块金条长度为 10+20+30=60。金条要分成 10, 20, 30 三个部分。 如果， 先把长度 60 的金条分成 10 和 50，花费60，再把长度 50 的金条分成 20 和 30， 花费 50，一共花费 110 铜板。 但是如果， 先把长度 60 的金条分成 30 和30，花费 60 再把长度 30 金条分成 10 和 20，花费 30， 一共花费 90 铜板。 输入一个数组，返回分割的最小代价。

**解答思路**

==本质上该题为 哈夫曼编码 问题==，也是比较经典的贪心问题。

**举个栗子**

最终要的结果为：{1,2,6,4,3,7,1,8}

**实现过程**：首先将所有元素形成**小根堆**，每次弹出最小的两个元素计算其和，然后将和再次放入堆中，以此反复直到只剩下一个元素，则所有的和为总代价。

**步骤：**

首先形成小根堆：

步一：拿出两个元素 1,1，和为 2，现在堆中元素为：2,6,4,3,7,8,2；

步二：拿出两个元素 2,2，和为 4，现在堆中元素为：6,4,3,7,8,4；

步三：拿出两个元素 3,4，和为 7，现在堆中元素为：6,7,8,4,7；

步四：拿出两个元素 4,6，和为 10，现在堆中元素为：7,8,7,10；

步五：拿出两个元素 7,7，和为 10，现在堆中元素为：8,10,14；

步六：拿出两个元素 8,10，和为 18，现在堆中元素为：18,14；

步六：拿出两个元素 18,14，和为 32，现在堆中元素为：32；

所有最终总代价为：2 + 4 + 7 + 10 + 14 + 18 + 32  = 87

#### 1. Java  中使用优先级队列实现小根堆和大根堆的方式

- 方式一：小根堆直接使用优先级队列，大根堆直接重写比较器

- 方式二：首先全部重写比较器，然后创建实例的时候加入构造器

    ```java
    /**
         * 方式一：小根堆直接使用优先级队列，大根堆直接重写比较器
         */
    // 形成小根堆
    PriorityQueue<Integer> minHeap1 = new PriorityQueue<Integer>();
    
    // 形成大根堆
    PriorityQueue<Integer> maxHeap1 = new PriorityQueue<>(new Comparator<Integer>() {
        @Override
        public int compare(Integer o1, Integer o2) {
            return o2 - o1;
        }
    });
    
    // 方式二：首先全部重写比较器，然后创建实例的时候加入构造器
    public static class MinheapComparator implements Comparator<Integer> {
        @Override
        public int compare(Integer o1, Integer o2) {
            return o1 - o2;
        }
    }
    
    public static class MaxheapComparator implements Comparator<Integer> {
        @Override
        public int compare(Integer o1, Integer o2) {
            return o2 - o1;
        }
    }
    
    PriorityQueue<Integer> minHeap2 = new PriorityQueue<>(new MinheapComparator());
    PriorityQueue<Integer> maxHeap2 = new PriorityQueue<>(new MaxheapComparator());
    ```

    

代码为：

```java
package com.gjxaiou.easy.day07;

import java.util.Comparator;
import java.util.PriorityQueue;

public class LessMoney {

    public static int lessMoney(int[] arr) {
        // Java 中小根堆可以采用优先级队列实现，如果实现大根堆重写比较器即可
        PriorityQueue<Integer> priorityQueue = new PriorityQueue<>();
        
        // 将数组中元素全部塞入优先级队列
        for (int i = 0; i < arr.length; i++) {
            priorityQueue.add(arr[i]);
        }

        int sum = 0;
        int cur = 0;
        // 弹出两个最小的，计算其和并放入。
        while (priorityQueue.size() > 1) {
            cur = priorityQueue.poll() + priorityQueue.poll();
            sum += cur;
            priorityQueue.add(cur);
        }
        return sum;
    }

    public static void main(String[] args) {
        // solution
        int[] arr = {10, 20, 30};
        System.out.println("形成 10，20,30 最少需要成本为：" + lessMoney(arr));
    }
}
```

结果为：

```java
形成 10，20,30 最少需要成本为：90
```



### （二）问题二：做项目完成最大收益

**题目**：

- 输入参数1：正数数组 costs ：`costs[i]` 表示 i 号项目的花费
- 参数2：正数数组 profits ：`profits[i]`表示 i 号项目在扣除花费之后还能挣到的钱(利润)，
- 参数3：正数 k ：k 表示你不能并行、**只能串行的最多做 k 个项目**
- 参数4：正数 m ： m 表示你初始的资金 。

说明：你每做完一个项目，马上获得的收益，可以支持你去做下 一个 项目。 

输出： 你最后获得的最大钱数。

**解答：**

首先每个项目包括两个属性：花费成本 cost 和利润 profile，然后将所有的项目放入数组中，示例如图所示：

![数组中项目结构](AlgorithmEasyDay07.resource/%E6%95%B0%E7%BB%84%E4%B8%AD%E9%A1%B9%E7%9B%AE%E7%BB%93%E6%9E%84.png)

然后将数组中所有的项目**按照花费成本的大小**形成一个**小根堆**，接着挨着从小根堆中弹出头部，只要花费成本小于初始资金（m）就**都弹出**，（弹出来的表示在当前资金下可以做的任务），然后将弹出的元素按照收益的高低放入大根堆中（则收益越高越优先做），每次只做大根堆的堆头部的任务，做完任务之后初始资金发生变化，再次从小根堆中弹出符合的元素进入大根堆，以此类推。直到大根堆中没有元素或者达到了最大 K 次就结束。



**示例**：以下任务的初始资金 m 为：7

cost：    5   10    6   20

profile：3    2     4    9

首先将所有的项目按照成本放入小根堆，然后通过比较 cost 和初始资金，则首先放入大根堆中是： 6,4 和 5,3，先做堆顶项目6,4，做完之后收益变为：11，则任务 10,2 可以做了，现在大根堆中为：5，3；10，2，先做 5,3 任务，收益变为 14，没有新的任务可以加入大根堆中，则继续做大根堆中任务：10,2，昨晚之后收益变为：16，还是没有新的项目可以加入大根堆，这时候大根堆没有任务可以完成了，则最终停止。

代码为：

```java
package com.gjxaiou.easy.day07;

import java.util.Comparator;
import java.util.PriorityQueue;

public class IPO {
    // 构造每个任务
    public static class Node {
        public int profit;
        public int cost;

        public Node(int profit, int cost) {
            this.profit = profit;
            this.cost = cost;
        }
    }

    // 下面两个用于实现小根堆和大根堆
    public static class MinCostComparator implements Comparator<Node> {
        @Override
        public int compare(Node o1, Node o2) {
            return o1.cost - o2.cost;
        }
    }

    public static class MaxProfitComparator implements Comparator<Node> {
        @Override
        public int compare(Node o1, Node o2) {
            return o2.profit - o1.profit;
        }
    }


    public static int findMaximizedCapital(int k, int m, int[] profits, int[] cost) {
        // 首先组装每个任务
        Node[] nodes = new Node[profits.length];
        for (int i = 0; i < profits.length; i++) {
            nodes[i] = new Node(profits[i], cost[i]);
        }

        PriorityQueue<Node> minCostQ = new PriorityQueue<>(new MinCostComparator());
        PriorityQueue<Node> maxProfitQ = new PriorityQueue<>(new MaxProfitComparator());
        // 首先将任务放入小根堆中
        for (int i = 0; i < nodes.length; i++) {
            minCostQ.add(nodes[i]);
        }
        // 开始做任务，一共最多做 K 次
        for (int i = 0; i < k; i++) {
            // 一直看小根堆堆顶，只要成本小于 m，则弹出该值并加入大根堆
            while (!minCostQ.isEmpty() && minCostQ.peek().cost <= m) {
                maxProfitQ.add(minCostQ.poll());
            }
            // 做不到 K 项目，即大根堆空了（资金不能做其他项目）也得停
            if (maxProfitQ.isEmpty()) {
                return m;
            }
            m += maxProfitQ.poll().profit;
        }
        return m;
    }
}
```

### （三）问题：在一个数据流中，随时可以获取到中位数

**题目描述：**

有一个源源不断地吐出整数的数据流，假设你有足够的空间来保存吐出的数。请设计一个名叫 MedianHolder 的结构，MedianHolder 可以随时取得之前吐出所有数的中位数。

**要求：**

- 如果 MedianHolder已经保存了吐出的 N 个数，那么任意时刻将一个新的数加入到 MedianHolder 的过程中，其时间复杂度为 O(logN)。
- 取得已经吐出的 N 个数整体的中位数的过程，时间复杂度为 O(1)。

**思考：**设计的 MedianHolder 中有两个堆，一个是大根堆，一个是小根堆。**大根堆中含有接收的所有数中较小的一半，并且按大根堆的方式组织起来，那么这个堆的堆顶就是较小一半的数中最大的那个**。小根堆中含有接收的所有数中较大的一半，并且按小根堆的方式组织起来，那么这个堆的堆顶就是较大一半的数中最小的那个。

例如，如果已经吐出的数为 6,1,3,0,9,8,7,2.

较小的一半为：0,1,2,3，那么 3 就是这一半的数组成的大根堆的堆顶

较大的一半为：6,7,8,9，那么 6 就是这一半的数组成的小根堆的堆顶

因为此时数的总个数为偶数，所以中位数就是两个堆顶相加，再除以 2.

**当加入一个数的时候**，如果此时新加入一个数10，那么这个数应该放进较大的一半里，所以此时较大的一半数为6,7,8,9,10，此时 6 依然是这一半的数组成的小根堆的堆顶，因为此时数的总个数为奇数，所以中位数应该是正好处在中间位置的数，而此时大根堆有 4 个数，小根堆有5个数，那么小根堆的堆顶 6 就是此时的中位数。如果此时又新加入一个数 11，那么这个数也应该放进较大的一半里，此时较大一半的数为：6,7,8,9,10,11.这个小根堆大小为6，而大根堆的大小为4，所以要进行如下调整：

- 如果大根堆的 size 比小根堆的 size 大 2，那么从大根堆里将堆顶元素弹出，并放入小根堆里

- 如果小根堆的size比大根堆的size大2，那么从小根堆里将堆顶弹出，并放入大根堆里。

经过这样的调整之后，大根堆和小根堆的size相同。

总结如下：

- 大根堆每时每刻都是较小的一半的数，堆顶为这一堆数的最大值

- 小根堆每时每刻都是较大的一半的数，堆顶为这一堆数的最小值
- 新加入的数根据与两个堆堆顶的大小关系，选择放进大根堆或者小根堆里

- 当任何一个堆的size比另一个size大2时，进行如上调整的过程。

这样随时都可以知道已经吐出的所有数处于中间位置的两个数是什么，取得中位数的操作时间复杂度为O(1)，同时根据堆的性质，向堆中加一个新的数，并且调整堆的代价为O（logN）。

代码：

```java
package com.gjxaiou.easy.day07;

import java.util.Arrays;
import java.util.Comparator;
import java.util.PriorityQueue;

public class MedianQuick {

    public static class MedianHolder {
        // 首先创建两个堆，一个大根堆、一个小根堆
        // 大根堆中放的是较小的一半数，所以堆顶是较小的一半数的最大一个
        private PriorityQueue<Integer> maxHeap =
                new PriorityQueue<Integer>(new MaxHeapComparator());
        // 小根堆中放的是较大的一半数，所以堆顶是较大的一半数中最小的一个
        private PriorityQueue<Integer> minHeap =
                new PriorityQueue<Integer>(new MinHeapComparator());

        // 如果大根堆和小根堆的 size 差距 >= 2，则调整堆
        private void modifyTwoHeapsSize() {
            if (maxHeap.size() == minHeap.size() + 2) {
                minHeap.add(this.maxHeap.poll());
            }
            if (minHeap.size() == maxHeap.size() + 2) {
                maxHeap.add(minHeap.poll());
            }
        }

        // 添加新数的时候，如果大根堆为空则加入大根堆
        public void addNumber(int num) {
            if (maxHeap.isEmpty()) {
                maxHeap.add(num);
                return;
            }
            // 如果该数小于等于大根堆顶，放入大根堆
            if (maxHeap.peek() >= num) {
                maxHeap.add(num);
            } else {
                // 如果该数大于大根堆的堆顶
                if (minHeap.isEmpty()) {
                    minHeap.add(num);
                    return;
                }
                // 当然如果该数小于小根堆的顶部，还是应该放入大根堆，否则放入小根堆
                if (minHeap.peek() > num) {
                    maxHeap.add(num);
                } else {
                    minHeap.add(num);
                }
            }
            // 放入之后判断是否需要调整堆大小
            modifyTwoHeapsSize();
        }

        /**
         * 获取中位数，入口
         *
         * @return
         */
        public Integer getMedian() {
            int maxHeapSize = maxHeap.size();
            int minHeapSize = minHeap.size();
            if (maxHeapSize + minHeapSize == 0) {
                return null;
            }
            Integer maxHeapHead = maxHeap.peek();
            Integer minHeapHead = minHeap.peek();
            // 如果两个堆中元素数目和为偶数，则结果为两个堆顶和的一半
            if (((maxHeapSize + minHeapSize) & 1) == 0) {
                return (maxHeapHead + minHeapHead) / 2;
            }
            // 如果为奇数，则结果为较大的堆的堆顶
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

    ///////////// 测试 //////////////////
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
        System.out.println(err ? "Bad" : "OK");
    }
}
```



### （四）问题：拼接字符串获得最低字典序

**题目**：给定一个字符串类型的数组 strs，找到一种拼接方式，使得把所有字符串拼接起来之后形成的字符串具有最低的字典序。

**分析**：

**此题的比较策略即为贪心策略**，但是不一定其它也是两者一样的。

- 贪心方案一（错误）：按照每个字符串字典序排序然后进行合并，例 “b” “ba” 排完之后是 “b”,”ba”，合并是：”bba”，但是实际上最小的是 “bab”;

- 贪心方案二：若 “str1 str2“ 拼接结果 <= “str2 str1” 拼接结果，则 str1 放在前面。

**如何判断自定义比较器的正确性**

==自定义比较器正确的前提是：具有传递性并且结果值唯一，同时可以比较出所有值的大小。==

例如：甲乙比较：甲 < 乙；乙丙比较：乙 < 丙；甲丙比较：丙 < 甲，结果具有随机性，就是比较不出来结果；

应该具有传递性，例如：甲 < 乙，乙 < 丙，然后甲 < 丙；

==**字典序比较方案**==

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

做题的时候不要妄图验证贪心方案的正确性，直接使用小数据样本输入该贪心方案和对数器的结果比较，如果小样本验证是正确的则默认该方案就是正确的

代码：

```java
package com.gjxaiou.easy.day07;

import java.util.Arrays;
import java.util.Comparator;

public class LowestLexicography {

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

    ///////////////////// 测试程序 //////////////////////
    public static void main(String[] args) {
        String[] strs1 = {"jibw", "ji", "jp", "bw", "jibw"};
        System.out.println(lowestString(strs1));

        String[] strs2 = {"ba", "b"};
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

一些项目要占用一个会议室宣讲，会议室不能同时容纳两个项目的宣讲。 给你每一个项目开始的时间和结束的时间(给你一个数组，里面 是一个个具体的项目)，你来安排宣讲的日程，要求会议室进行 的宣讲的场次最多。返回这个最多的宣讲场次。

**解答**

下面是不同的贪心策略：

方式一（错误）：最早开始的最先安排；

方式二（错误）：哪个会议持续时间长；

方式三（正确）：哪个项目最早结束，然后淘汰因为安排这个项目而耽误的其他的项目；

代码：

```java
package com.gjxaiou.easy.day07;

import java.util.Arrays;
import java.util.Comparator;

public class BestArrange {

    // 构建项目结构
    public static class Program {
        public int start;
        public int end;

        public Program(int start, int end) {
            this.start = start;
            this.end = end;
        }
    }

    // 自定义比较器
    public static class ProgramComparator implements Comparator<Program> {
        @Override
        public int compare(Program o1, Program o2) {
            return o1.end - o2.end;
        }
    }

    public static int bestArrange(Program[] programs, int currentTime) {
        // 首先按照项目结束时间排序
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

