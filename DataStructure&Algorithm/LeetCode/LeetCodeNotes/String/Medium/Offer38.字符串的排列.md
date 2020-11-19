---
layout:     post
title:      Offer38. 字符串的排列
subtitle:   String.medium
date:       2020-05-06
author:     GJXAIOU
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - 字符串
	- 回溯算法
	- 完成
---

# [Offer38. 字符串的排列](https://leetcode-cn.com/problems/zi-fu-chuan-de-pai-lie-lcof/)

## 一、题目

输入一个字符串，打印出该字符串中字符的所有排列。

你可以以任意顺序返回这个字符串数组，但里面不能有重复元素。



**示例:**

```
输入：s = "abc"
输出：["abc","acb","bac","bca","cab","cba"]
```

 

**限制：**

```
1 <= s 的长度 <= 8
```

## 二、解答

**该题和 47.全排列II 一样**。

```java
package string.medium;

import java.lang.reflect.Array;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

/**
 * @author GJXAIOU
 * @create 2020/05/06 14:14
 */
public class Offer38 {
    List<String> res = new ArrayList<>();

    public String[] permutation(String s) {
        if (s.length() == 0) {
            return res.toArray(new String[res.size()]);
        }
        char[] sArray = s.toCharArray();
        Arrays.sort(sArray);
        StringBuilder path = new StringBuilder();
        boolean[] used = new boolean[sArray.length];
        backtrack(sArray, path, used);
        return res.toArray(new String[res.size()]);
    }

    public void backtrack(char[] sArray, StringBuilder path, boolean[] used) {
        // 结束条件
        if (path.length() == sArray.length) {
            res.add(path.toString());
            return;
        }
        // 选择列表
        for (int i = 0; i < sArray.length; i++) {
            //从给定的数中除去，用过的数，就是当前的选择列表
            if (!used[i]) {
                // 剪枝
                if ((i >= 1) && (sArray[i - 1] == sArray[i]) && !used[i - 1]) {
                    continue;
                }// 做出选择
                path.append(sArray[i]);
                used[i] = true;
                // 进入下一层
                backtrack(sArray, path, used);
                // 撤销选择
                path.deleteCharAt(path.length() - 1);
                used[i] = false;
            }
        }
    }
}
```

