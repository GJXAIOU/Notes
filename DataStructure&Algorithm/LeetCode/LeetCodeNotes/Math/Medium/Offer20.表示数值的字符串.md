---
layout:     post
title:      Offer49. 丑数
subtitle:   Math.medium
date:       2020-02-09
author:     GJXAIOU
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - 数学
	- 有限状态机
	- 完成
---

# [Offer 20. 表示数值的字符串](https://leetcode-cn.com/problems/biao-shi-shu-zhi-de-zi-fu-chuan-lcof/)

## 一、题目

请实现一个函数用来判断字符串是否表示数值（包括整数和小数）。例如，字符串"+100"、"5e2"、"-123"、"3.1416"、"0123"都表示数值，但"12e"、"1a3.14"、"1.2.3"、"+-5"、"-1E-16"及"12e+5.4"都不是。

 

注意：本题与主站 65 题相同：https://leetcode-cn.com/problems/valid-number/



## 二、解答

**解题思路：**
本题使用有限状态自动机。根据字符类型和合法数值的特点，先定义状态，再画出状态转移图，最后编写代码即可。

**字符类型：**

空格 「 」、数字「 0—9 」 、正负号 「 +− 」 、小数点 「 . 」 、幂符号 「 e 」 。

**状态定义：**

按照字符串从左到右的顺序，定义以下 9 种状态。

- 开始的空格
- 幂符号前的正负号
- **小数点前的数字**
- **小数点、小数点后的数字**
- 当小数点前为空格时，小数点、小数点后的数字
- 幂符号
- 幂符号后的正负号
- **幂符号后的数字**
- **结尾的空格**

**结束状态：**

合法的**结束状态**有 2, 3, 7, 8 。

![image-20200623093900523](Offer20.%E8%A1%A8%E7%A4%BA%E6%95%B0%E5%80%BC%E7%9A%84%E5%AD%97%E7%AC%A6%E4%B8%B2.resource/image-20200623093900523.png)

算法流程：

- 初始化：
    - 状态转移表 states ： 设 states[i] ，其中 i 为所处状态，states[i] 使用哈希表存储可转移至的状态。键值对 (key, value) 含义：若输入 key ，则可从状态 i 转移至状态 value 。
    - 当前状态 p ： 起始状态初始化为 p=0 。

- 状态转移循环： 遍历字符串 s 的每个字符 c 。
    - 记录字符类型 t ： 分为三种情况。
        - 当 c 为正负号时，t = 's' ;
        - 当 c 为数字时，t = 'd' ;
        - 否则，t = c ，即用字符本身表示字符类型 ;
    - 终止条件： 若字符类型 t 不在哈希表 states[p] 中，说明无法转移至下一状态，因此直接返回 False 。
    - 状态转移： 状态 p 转移至 `states[p][t]` 。
- 返回值： 跳出循环后，若状态 $p \in {2, 3, 7, 8}$ ，说明结尾合法，返回 True ，否则返回 False 。

**复杂度分析：**
时间复杂度 O(N) ： 其中 N 为字符串 s 的长度，判断需遍历字符串，每轮状态转移的使用 O(1) 时间。
空间复杂度 O(1) ： states 和 p 使用常数大小的额外空间。

![image-20200623110126106](Offer20.%E8%A1%A8%E7%A4%BA%E6%95%B0%E5%80%BC%E7%9A%84%E5%AD%97%E7%AC%A6%E4%B8%B2.resource/image-20200623110126106.png)

​		![image-20200623110150434](Offer20.%E8%A1%A8%E7%A4%BA%E6%95%B0%E5%80%BC%E7%9A%84%E5%AD%97%E7%AC%A6%E4%B8%B2.resource/image-20200623110150434.png)

![image-20200623110203220](Offer20.%E8%A1%A8%E7%A4%BA%E6%95%B0%E5%80%BC%E7%9A%84%E5%AD%97%E7%AC%A6%E4%B8%B2.resource/image-20200623110203220.png)

![image-20200623110213641](Offer20.%E8%A1%A8%E7%A4%BA%E6%95%B0%E5%80%BC%E7%9A%84%E5%AD%97%E7%AC%A6%E4%B8%B2.resource/image-20200623110213641.png)

![image-20200623110226385](Offer20.%E8%A1%A8%E7%A4%BA%E6%95%B0%E5%80%BC%E7%9A%84%E5%AD%97%E7%AC%A6%E4%B8%B2.resource/image-20200623110226385.png)



![image-20200623110240554](Offer20.%E8%A1%A8%E7%A4%BA%E6%95%B0%E5%80%BC%E7%9A%84%E5%AD%97%E7%AC%A6%E4%B8%B2.resource/image-20200623110240554.png)



![image-20200623110256325](Offer20.%E8%A1%A8%E7%A4%BA%E6%95%B0%E5%80%BC%E7%9A%84%E5%AD%97%E7%AC%A6%E4%B8%B2.resource/image-20200623110256325.png)



![image-20200623110311062](Offer20.%E8%A1%A8%E7%A4%BA%E6%95%B0%E5%80%BC%E7%9A%84%E5%AD%97%E7%AC%A6%E4%B8%B2.resource/image-20200623110311062.png)

![image-20200623110324170](Offer20.%E8%A1%A8%E7%A4%BA%E6%95%B0%E5%80%BC%E7%9A%84%E5%AD%97%E7%AC%A6%E4%B8%B2.resource/image-20200623110324170.png)

代码：
Java 的状态转移表 states 使用 Map[] 数组存储。

```java
import java.util.HashMap;
import java.util.Map;

class Solution {
    public boolean isNumber(String s) {
        Map[] states = {
                new HashMap<Character, Integer>() {{
                    put(' ', 0);
                    put('s', 1);
                    put('d', 2);
                    put('.', 4);
                }},
                // 0.
                new HashMap<Character, Integer>() {{
                    put('d', 2);
                    put('.', 4);
                }},                           // 1.
                new HashMap<Character, Integer>() {{
                    put('d', 2);
                    put('.', 3);
                    put('e', 5);
                    put(' ', 8);
                }}, // 2.
                new HashMap<Character, Integer>() {{
                    put('d', 3);
                    put('e', 5);
                    put(' ', 8);
                }},              // 3.
                new HashMap<Character, Integer>() {{
                    put('d', 3);
                }},                                        // 4.
                new HashMap<Character, Integer>() {{
                    put('s', 6);
                    put('d', 7);
                }},                           // 5.
                new HashMap<Character, Integer>() {{
                    put('d', 7);
                }},                                        // 6.
                new HashMap<Character, Integer>() {{
                    put('d', 7);
                    put(' ', 8);
                }},                           // 7.
                new HashMap<Character, Integer>() {{
                    put(' ', 8);
                }}                                         // 8.
        };
        int p = 0;
        char t;
        for (char c : s.toCharArray()) {
            if (c >= '0' && c <= '9') {
                t = 'd';
            } else if (c == '+' || c == '-') {
                t = 's';
            } else {
                t = c;
            }
            if (!states[p].containsKey(t)) {
                return false;
            }
            p = (int) states[p].get(t);
        }
        return p == 2 || p == 3 || p == 7 || p == 8;
    }
}

```



方法二：

```java

public class Solution {
    public boolean isNumeric(char[] str) {
        if (str == null || str.length == 0) {
            return false;
        }
        //标记是否遇到相应情况
        boolean numSeen = false;
        boolean dotSeen = false;
        boolean eSeen = false;
        for (int i = 0; i < str.length; i++) {
            if (str[i] >= '0' && str[i] <= '9') {
                numSeen = true;
            } else if (str[i] == '.') {
                //.之前不能出现.或者e
                if (dotSeen || eSeen) {
                    return false;
                }
                dotSeen = true;
            } else if (str[i] == 'e' || str[i] == 'E') {
                //e之前不能出现e，必须出现数
                if (eSeen || !numSeen) {
                    return false;
                }
                eSeen = true;
                numSeen = false;//重置numSeen，排除123e或者123e+的情况,确保e之后也出现数
            } else if (str[i] == '-' || str[i] == '+') {
                //+-出现在0位置或者e/E的后面第一个位置才是合法的
                if (i != 0 && str[i - 1] != 'e' && str[i - 1] != 'E') {
                    return false;
                }
            } else {//其他不合法字符
                return false;
            }
        }
        return numSeen;
    }
}
```

