# OtherSummary

## 一、HashMap

- `map.get(key)` 不能使用 `++`运算符，可以使用 `+1` 代替；

- 遍历 Key 和 Value 的方式：

    ```java
    HashMap<Integer, Integer>() map  = new HashMap<>();  
    // 遍历 key
    for (Integer key : map.keySet()){
    }
    // 遍历 value
    for(Integer value : map.values()){
    }
    ```

- 已经含有 Key 则对应 value + 1，否则放入

    ```java
    HashMap<Integer, Integer> map = new HashMap<>();
    // 这里的 defaultValue 设置为 0，根据题意设置即可
    map.put(key, map.getOrDefault(key, 0) + 1);
    ```

- 遍历整个 hash 表方式

    ```java
    for (Entry<String, String> entry : map.entrySet()) {
        String key = entry.getKey();
        String value = entry.getValue();
        System.out.println(key + "," + value);
    }
    ```

- 根据 value 值查找键值对

    ```java
    HashMap<String,Integer> map = new HashMap<>();
    map.put("A", "1");
    map.put("B", "2");
    map.put("C", "3");
    map.put("D", "1");
    map.put("E", "2");
    map.put("F", "3");
    map.put("G", "1");
    map.put("H", "2");
    map.put("I", "3");
    
    List<String> removeKeys = new ArrayList<String>();
    for (Entry<String, String> entry : map.entrySet()) {
        if (!entry.getValue().equals("1")) {
            removeKeys.add(entry.getKey());
        }
    }
    for (String removeKey : removeKeys) {
        map.remove(removeKey);
    }
    for (Entry<String, String> entry : map.entrySet()) {
        String key = entry.getKey();
        String value = entry.getValue();
        System.out.println(key + "," + value);
    }
    ```

- ==HashMap  赋初值==

    上面方式是在方法体内部，如果在方法外：

    ```java
    public class LeetCode17 {
        HashMap<Character, String> valueMap = new HashMap<Character, String>() {{
            put('2', "abc");
            put('3', "def");
            put('4', "ghi");
            put('5', "jkl");
            put('6', "mno");
            put('7', "pqrs");
            put('8', "tuv");
            put('9', "wxyz");
        }};
    }
    ```

    


## 二、零碎知识点

- 在 Java 中

    *   数组相关，用 `length`
    *   集合相关，用 `size()`
    *   字符串相关，用 `length()`

## 三、代码中常见特殊情况

- 只有一个元素
- 为空
- 只有一个元素且该元素为 0；



## 三、求中点

首先假设我们的变量都是 int 值。二分查找中我们需要根据 start 和 end 求中点，正常情况下加起来除以 2 即可。

- 方法一：`int mid = (start + end) / 2`
    - 缺点：当 start 和 end 均为 int 类型的最大值即 `Integer.MAX_VALUE = 2147483647`，则相加会造成溢出。
- 方法二：`(start + end) / 2 = (start + end + start - start) / 2 = start + (end - start) / 2`
    - 解决的一个方案就是利用数学上的技巧，我们可以加一个 start 再减一个 start 将公式变形。

- 方法三：`int mid = (start + end) >>> 1`
    - 这是 JDK 源码中采用的方式，通过移位实现除以 2；
    - 右移 `>>` 为有符号右移，右移之后最高为保持原来的最高位，而右移 `>>>` 右移之后最高位补 0；所以这里其实利用到了整数的补码形式，最高位其实是符号位，所以当 start + end 溢出的时候，其实本质上只是符号位收到了进位，而 `>>>` 这个右移不仅可以把符号位右移，同时最高位只是补零，不会对数字的大小造成影响。

## 四、二分查找

下面问答的形式，探究几个最常用的二分查找场景：寻找一个数、寻找左侧边界、寻找右侧边界。第一个场景是最简单的算法形式，解决 [这道题](https://leetcode-cn.com/problems/binary-search/)，后两个场景就是本题。

### （一）二分查找框架

```java
int binarySearch(int[] nums, int target) {
    int left = 0, right = ...;

    while(...) {
        int mid = (left + right) >>> 1;
        if (nums[mid] == target) {
            ...
        } else if (nums[mid] < target) {
            left = ...;
        } else if (nums[mid] > target) {
            right = ...;
        }
    }
    return ...;
}
```

分析二分查找的一个技巧是：==不要出现 else，而是把所有情况用 else if 写清楚，这样可以清楚地展现所有细节==。当然理解后可自行简化。

其中 ... 标记的部分，就是可能出现细节问题的地方，当你见到一个二分查找的代码时，首先注意这几个地方。后文用实例分析这些地方能有什么样的变化。

### （二）寻找一个数（基本的二分搜索）

这个场景是最简单的，可能也是大家最熟悉的，即搜索一个数，如果存在，返回其索引，否则返回 -1。

```java
int binarySearch(int[] nums, int target) {
    int left = 0;
    int right = nums.length - 1; // 注意

    while (left <= right) {
        int mid = (left + right) >>> 1;
        if (nums[mid] == target) {
            return mid;
        } else if (nums[mid] < target) {
            left = mid + 1; // 注意
        } else if (nums[mid] > target) {
            right = mid - 1; // 注意
        }
    }
    return -1;
}
```

- 为什么 while 循环的条件中是 <=，而不是 < ？

答：因为初始化 right 的赋值是 `nums.length - 1`，即最后一个元素的索引，而不是 nums.length。

这二者可能出现在不同功能的二分查找中，区别是：前者相当于两端都闭区间 [left, right]，后者相当于左闭右开区间 [left, right)，因为索引大小为 nums.length 是越界的。

我们这个算法中使用的是前者 [left, right] 两端都闭的区间。这个区间其实就是每次进行搜索的区间，我们不妨称为「**搜索区间**」。

==什么时候应该停止搜索==呢？当然，找到了目标值的时候可以终止

```java
if (nums[mid] == target) {
    return mid;
}
```

==如果没找到，就需要 while 循环终止==，然后返回 -1。那 while 循环什么时候应该终止？搜索区间为空的时候应该终止，意味着你没得找了，就等于没找到嘛。

`while(left <= right)` 的终止条件是 `left == right + 1`，写成区间的形式就是 [right + 1, right]，或者带个具体的数字进去 [3, 2]，可见这时候搜索区间为空，因为没有数字既大于等于 3 又小于等于 2 的吧。所以这时候 while 循环终止是正确的，直接返回 -1 即可。

如果设定为：`while(left < right)` 的终止条件是 `left == right`，写成区间的形式就是 [left, right]，或者带个具体的数字进去 [2, 2]，这时候搜索区间非空，还有一个数 2，但此时 while 循环终止了。也就是说这区间 [2, 2] 被漏掉了，索引 2 没有被搜索，如果这时候直接返回 -1 就是错误的。

当然，如果你非要用 while(left < right) 也可以，我们已经知道了出错的原因，就打个补丁好了：

```java
//...
while(left < right) {
    // ...
}
return nums[left] == target ? left : -1;

```

- 为什么 `left = mid + 1`，`right = mid - 1`？我看有的代码是 `right = mid` 或者 `left = mid`，没有这些加加减减，到底怎么回事，怎么判断？

答：这也是二分查找的一个难点，不过只要你能理解前面的内容，就能够很容易判断。

刚才明确了「搜索区间」这个概念，而且本算法的搜索区间是两端都闭的，即 [left, right]。那么当我们发现索引 mid 不是要找的 target 时，如何确定下一步的搜索区间呢？

当然是 [left, mid - 1] 或者 [mid + 1, right] 对不对？因为 mid 已经搜索过，应该从搜索区间中去除。

- 此算法有什么缺陷？

答：至此，你应该已经掌握了该算法的所有细节，以及这样处理的原因。但是，这个算法存在局限性。

比如说给你有序数组 nums = [1,2,2,2,3]，target = 2，此算法返回的索引是 2，没错。但是如果我想得到 target 的左侧边界，即索引 1，或者我想得到 target 的右侧边界，即索引 3，这样的话此算法是无法处理的。

这样的需求很常见。你也许会说，找到一个 target，然后向左或向右线性搜索不行吗？可以，但是不好，因为这样难以保证二分查找对数级的复杂度了。

我们后续的算法就来讨论这两种二分查找的算法。

### （三）寻找左侧边界的二分搜索

直接看代码，其中的标记是需要注意的细节：

```java
int leftBound(int[] nums, int target) {
    if (nums.length == 0) {
        return -1;
    }
    int left = 0;
    int right = nums.length; // 注意

    while (left < right) { // 注意
        int mid = (left + right) >>> 1;
        if (nums[mid] == target) {
            right = mid;
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid; // 注意
        }
    }
    return left;
}
```

- 为什么 `while(left < right)` 而不是 `<=` ?

答：用相同的方法分析，因为 `right = nums.length` 而不是 `nums.length - 1` 。因此每次循环的「搜索区间」是 `[left, right)` 左闭右开。

`while(left < right)` 终止的条件是 `left == right`，此时搜索区间 `[left, left)` 为空，所以可以正确终止。

- 为什么没有返回 -1 的操作？如果 nums 中不存在 target 这个值，怎么办？

答：因为要一步一步来，先理解一下这个「左侧边界」有什么特殊含义：

<img src="OtherSummary.resource/0ee763a9e3b27dddf9c60ffe7e17db7160d2910d7bca591af8f3e3202d0f19ea-file_1560274288808.png" alt="binarySearch" style="zoom:67%;" />

对于这个数组，算法会返回 1。这个 1 的含义可以这样解读：nums 中小于 2 的元素有 1 个。

比如对于有序数组 nums = [2,3,5,7], target = 1，算法会返回 0，含义是：nums 中小于 1 的元素有 0 个。

再比如说 nums 不变，target = 8，算法会返回 4，含义是：nums 中小于 8 的元素有 4 个。

综上可以看出，函数的返回值（即 left 变量的值）取值区间是闭区间 [0, nums.length]，所以我们简单添加两行代码就能在正确的时候 return -1：

```java
while (left < right) {
    //...
}
// target 比所有数都大
if (left == nums.length) return -1;
// 类似之前算法的处理方式
return nums[left] == target ? left : -1;

```

- 为什么 `left = mid + 1，right = mid` ？和之前的算法不一样？

答：这个很好解释，因为我们的「搜索区间」是 [left, right) 左闭右开，所以当 nums[mid] 被检测之后，下一步的搜索区间应该去掉 mid 分割成两个区间，即 [left, mid) 或 [mid + 1, right)。

- 为什么该算法能够搜索左侧边界？

答：关键在于对于 nums[mid] == target 这种情况的处理：

```java
if (nums[mid] == target)
    right = mid;
```

可见，找到 target 时不要立即返回，而是缩小「搜索区间」的上界 right，在区间 [left, mid) 中继续搜索，即不断向左收缩，达到锁定左侧边界的目的。

- 为什么返回 left 而不是 right？

答：都是一样的，因为 while 终止的条件是 left == right。

### （四）寻找右侧边界的二分查找

寻找右侧边界和寻找左侧边界的代码差不多，只有两处不同，已标注：

```java
int right_bound(int[] nums, int target) {
    if (nums.length == 0) return -1;
    int left = 0, right = nums.length;
    
    while (left < right) {
        int mid = (left + right) / 2;
        if (nums[mid] == target) {
            left = mid + 1; // 注意
        } else if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid;
        }
    }
    return left - 1; // 注意
}
```

- 为什么这个算法能够找到右侧边界？

答：类似地，关键点还是这里：

```java
if (nums[mid] == target) {
    left = mid + 1;
```

当 `nums[mid] == target` 时，不要立即返回，而是增大「搜索区间」的下界 left，使得区间不断向右收缩，达到锁定右侧边界的目的。

- 为什么最后返回 left - 1 而不像左侧边界的函数，返回 left？而且我觉得这里既然是搜索右侧边界，应该返回 right 才对。

答：首先，while 循环的终止条件是 left == right，所以 left 和 right 是一样的，你非要体现右侧的特点，返回 right - 1 好了。

至于为什么要减一，这是搜索右侧边界的一个特殊点，关键在这个条件判断：

```java
if (nums[mid] == target) {
    left = mid + 1;
    // 这样想: mid = left - 1
```

<img src="OtherSummary.resource/dc975e6d3c8b9d0ee5453ce9253d5ee3b2b3ee6461a5183d3922d52724873709-file_1560274288798.png" alt="binarySearch" style="zoom:67%;" />

因为我们对 left 的更新必须是 left = mid + 1，就是说 while 循环结束时，nums[left] 一定不等于 target 了，而 nums[left-1] 可能是 target。

至于为什么 left 的更新必须是 left = mid + 1，同左侧边界搜索，就不再赘述。

- 为什么没有返回 −1 的操作？如果 nums 中不存在 target 这个值，怎么办？

答：类似之前的左侧边界搜索，因为 while 的终止条件是 left == right，就是说 left 的取值范围是 [0, nums.length]，所以可以添加两行代码，正确地返回 −1：

```java
while (left < right) {
    // ...
}
if (left == 0) return -1;
return nums[left-1] == target ? (left-1) : -1;

```

### （五）最后总结

来梳理一下这些细节差异的因果逻辑：

第一个，最基本的二分查找算法：

```python
因为我们初始化 right = nums.length - 1
所以决定了我们的「搜索区间」是 [left, right]
所以决定了 while (left <= right)
同时也决定了 left = mid+1 和 right = mid-1

因为我们只需找到一个 target 的索引即可
所以当 nums[mid] == target 时可以立即返回

```

第二个，寻找左侧边界的二分查找：

```python
因为我们初始化 right = nums.length
所以决定了我们的「搜索区间」是 [left, right)
所以决定了 while (left < right)
同时也决定了 left = mid + 1 和 right = mid

因为我们需找到 target 的最左侧索引
所以当 nums[mid] == target 时不要立即返回
而要收紧右侧边界以锁定左侧边界
```

第三个，寻找右侧边界的二分查找：

```python
因为我们初始化 right = nums.length
所以决定了我们的「搜索区间」是 [left, right)
所以决定了 while (left < right)
同时也决定了 left = mid + 1 和 right = mid

因为我们需找到 target 的最右侧索引
所以当 nums[mid] == target 时不要立即返回
而要收紧左侧边界以锁定右侧边界

又因为收紧左侧边界时必须 left = mid + 1
所以最后无论返回 left 还是 right，必须减一


```



