---
layout:     post
title:      Offer58-II 左旋转字符串
subtitle:   String.easy
date:       2020-02-24
author:     GJXAIOU
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - 字符串
	- 位运算
	- 完成
---

# Offer58-II 左旋转字符串

## 一、题目

字符串的左旋转操作是把字符串前面的若干个字符转移到字符串的尾部。请定义一个函数实现字符串左旋转操作的功能。比如，输入字符串"abcdefg"和数字2，该函数将返回左旋转两位得到的结果"cdefgab"。

- 示例 1：

> 输入: s = "abcdefg", k = 2
> 输出: "cdefgab"

- 示例 2：

> 输入: s = "lrloseumgh", k = 6
> 输出: "umghlrlose"

- 限制：

> 1 <= k < s.length <= 10000

来源：力扣（LeetCode）
链接：https://leetcode-cn.com/problems/zuo-xuan-zhuan-zi-fu-chuan-lcof
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

## 二、解答

```java
package string.easy;


/**
 * @Author GJXAIOU
 * @Date 2020/2/24 9:43
 */
public class Offer58II {
    // 方法一：前后直接替换
    public String reverseLeftWords(String s, int n) {
        // 注意：根据题目结果，如果 n 超过 s.length 则返回原字符串，下面这一块是适用于可以超过的情况。
        /**
         if (n == 0 || s.length() == 0) {
         return s;
         }
         int m = 0;
         if (n >= s.length()) {
         m = n % s.length();
         } else {
         m = n;
         }
         */
        if (s.length() > 0 && n > 0 && n < s.length()) {
            String front = s.substring(0, n);
            String after = s.substring(n);
            StringBuilder res = new StringBuilder();
            res.append(after).append(front);
            return res.toString();
        }
        return s;
    }

    // 方法二：模运算
    public String reverseLeftWords2(String s, int n) {
        String res = "";
        if (s.length() > 0 && n > 0 && n < s.length()) {
            for (int i = n; i < n + s.length(); i++) {
                res += s.charAt(i % s.length());
            }
        }
        return res;
    }

    // 方法三：分别旋转
    public String reverseLeftWords3(String str, int n) {
        if (str == null || str.length() == 0 || n % str.length() == 0) {
            return str;
        }
        n %= str.length();
        char[] inputString = str.toCharArray();
        reverse(inputString, 0, inputString.length - 1);
        reverse(inputString, 0, inputString.length - n - 1);
        reverse(inputString, inputString.length - n, inputString.length - 1);
        StringBuilder res = new StringBuilder();
        for (int i = 0; i < inputString.length; i++) {
            res.append(inputString[i]);
        }
        return res.toString();
    }

    public void reverse(char[] inputString, int begin, int end) {
        while (begin < end) {
            char temp = inputString[begin];
            inputString[begin] = inputString[end];
            inputString[end] = temp;
            begin++;
            end--;
        }
    }
        
}
```

