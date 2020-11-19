---
layout:     post
title:      Offer58I. 翻转单词顺序
subtitle:   String.easy
date:       2020-02-24
author:     GJXAIOU
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - 字符串
	- 完成
---

# Offer58I. 翻转单词顺序

## 一、题目

输入一个英文句子，翻转句子中单词的顺序，**但单词内字符的顺序不变**。为简单起见，标点符号和普通字母一样处理。例如输入字符串"I am a student. "，则输出"student. a am I"。

**示例 1：**

```
输入: "the sky is blue"
输出: "blue is sky the"
```

**示例 2：**

```
输入: "  hello world!  "
输出: "world! hello"
解释: 输入字符串可以在前面或者后面包含多余的空格，但是反转后的字符不能包括。
```

**示例 3：**

```
输入: "a good   example"
输出: "example good a"
解释: 如果两个单词间有多余的空格，将反转后单词间的空格减少到只含一个。
```

**说明：**

- 无空格字符构成一个单词。
- 输入字符串可以在前面或者后面包含多余的空格，但是反转后的字符不能包括。
- 如果两个单词间有多余的空格，将反转后单词间的空格减少到只含一个。

**注意：**本题与主站 151 题相同：https://leetcode-cn.com/problems/reverse-words-in-a-string/



## 二、解答

### 方法一：内置函数

==注意：可能有多个空格==

```java
package string.easy;

/**
 * @author GJXAIOU
 * @create 2020/04/13 20:03
 */
public class Offer58I {

    // 方法一：使用内置函数
    public String reverseWords(String s) {
        String[] sArray = s.trim().split(" ");
        StringBuilder res = new StringBuilder();
        for (int i = sArray.length - 1; i >= 1; i--) {
            // 可能有多个空格
            if (sArray[i].equals(" ")) {
                continue;
            }
            res.append(sArray[i]).append(" ");
        }
        // 最后一个不添加空格
        res.append(sArray[0]);
        // 去除最后的末尾空格
        return res.toString().trim();
    }

}
```

### 方法二：双指针

- 倒序遍历字符串 s ，记录单词左右索引边界 i , j ；
- 每确定一个单词的边界，则将其添加至单词列表 res ；
- 最终，将单词列表拼接为字符串，并返回即可。

![Offer58I](Offer58I.%E7%BF%BB%E8%BD%AC%E5%8D%95%E8%AF%8D%E9%A1%BA%E5%BA%8F.resource/Offer58I.gif)



复杂度分析：
时间复杂度 O(N) ： 其中 N 为字符串 s 的长度，线性遍历字符串。
空间复杂度 O(N) ： StringBuilder 中的字符串总长度 ≤N ，占用 O(N) 大小的额外空间。

```java
package string.easy;

import org.junit.jupiter.api.Test;

/**
 * @author GJXAIOU
 * @create 2020/04/13 20:03
 */
public class Offer58I {

    // 方法二：双指针
    public String reverseWords2(String s) {
        // 首先去除前后空格
        s = s.trim();
        int right = s.length() - 1;
        int left = right;
        StringBuilder res = new StringBuilder();
        while (left >= 0) {
            while (left >= 0 && s.charAt(left) != ' ') {
                left--; // 搜索首个空格
            }
            res.append(s.substring(left + 1, right + 1) + " "); // 添加单词
            while (left >= 0 && s.charAt(left) == ' ') {
                left--; // 跳过单词间空格
            }
            right = left; // right 指向下个单词的尾字符
        }
        return res.toString().trim();
    }
}
```



