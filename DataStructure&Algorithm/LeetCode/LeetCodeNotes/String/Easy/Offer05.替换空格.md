---
layout:     post
title:      Offer05：替换空格
subtitle:   String.easy
date:       2020-02-24
author:     GJXAIOU
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - 字符串
	- 完成
---

# Offer05：替换空格

## 一、题目


请实现一个函数，把字符串 `s` 中的每个空格替换成"%20"。

**示例 1：**

```
输入：s = "We are happy."
输出："We%20are%20happy."
```

**限制：**

```
0 <= s 的长度 <= 10000
```



## 二、解答

### 方法一：推荐

- 初始化一个 StringBuilder ，记为 res ；
- 遍历字符串 s 中的每个字符 c ：
    - 当 c 为空格时：向 res 后添加字符串 "%20"；
    - 当 c 不为空格时：向 res 后添加字符 c ；
- 将 res 转化为 String 类型并返回。

```java
package string.easy;

/**
 * @Author GJXAIOU
 * @Date 2020/2/24 13:56
 */
public class Offer05 {
    public String replaceSpace2(String s) {
        StringBuilder res = new StringBuilder();
        for (Character c : s.toCharArray()) {
            if (c == ' ') {
                res.append("%20");
            } else {
                res.append(c);
            }
        }
        return res.toString();
    }
}
```

- 复杂度分析：
    - 时间复杂度 O(N)： 遍历使用 O(N)，每轮添加（修改）字符操作使用 O(1) ；
    - 空间复杂度 O(N)： Java 新建的 StringBuilder 都使用了线性大小的额外空间。



### 方法二：字符数组

==由于每次替换从 1 个字符变成 3 个字符==，使用字符数组可方便地进行替换。建立字符数组地长度为 s 的长度的 3 倍，这样可保证字符数组可以容纳所有替换后的字符。

- 获得 s 的长度 length
- 创建字符数组 array，其长度为 length * 3，可以先遍历一遍，根据空格的数量来增加长度
- 初始化 size 为 0，size 表示替换后的字符串的长度
- 从左到右遍历字符串 s
    - 获得 s 的当前字符 c
    - 如果字符 c 是空格，则令 array[size] = '%'，array[size + 1] = '2'，array[size + 2] = '0'，并将 - size 的值加 3
    - 如果字符 c 不是空格，则令 array[size] = c，并将 size 的值加 1
- 遍历结束之后，size 的值等于替换后的字符串的长度，从 array 的前 size 个字符创建新字符串，并返回新字符串

```java
package string.easy;

/**
 * @Author GJXAIOU
 * @Date 2020/2/24 13:56
 */
public class Offer05 {

    public String replaceSpace(String s) {
        if (s.length() == 0) {
            return s;
        }
        int count = 0;
        for (int i = 0; i < s.length(); i++) {
            if (s.charAt(i) == ' ') {
                count++;
            }
        }
        // 空格占 1 个字符，%20 占 3 个字符，所以有一个空格结果中需要加两个位置
        char[] resArray = new char[s.length() + count * 2];
        int j = 0;
        for (int i = 0; i < s.length(); i++) {
            char temp = s.charAt(i);
            if (temp == ' ') {
                resArray[j++] = '%';
                resArray[j++] = '2';
                resArray[j++] = '0';
            } else {
                resArray[j++] = temp;
            }

        }
        StringBuilder res = new StringBuilder();
        for (int i = 0; i < resArray.length; i++) {
            res.append(resArray[i]);
        }

        return res.toString();
    }

}

```

**复杂性分析**

- 时间复杂度：O(n)。遍历字符串 `s` 一遍。
- 空间复杂度：O(n)。额外创建字符数组，长度为 `s` 的长度的 3 倍。



