---
layout:     post
title:      Offer62. 圆圈中最后剩下的数字
subtitle:   Math.easy
date:       2020-04-12
author:     GJXAIOU
header-img: img/post-bg-leetcode.png
catalog: true
tags:
    - 数学
    - 约瑟夫环
	- 完成
---



# Offer62. 圆圈中最后剩下的数字

## 一、题目

`0,1,,n-1`这 n 个数字排成一个圆圈，从数字 0 开始，每次从这个圆圈里删除第 m 个数字。求出这个圆圈里剩下的最后一个数字。

例如，`0、1、2、3、4` 这 5 个数字组成一个圆圈，从数字 0 开始每次删除第 3 个数字，则删除的前 4 个数字依次是 `2、0、4、1`，因此最后剩下的数字是 3。

**示例 1：**

```
输入: n = 5, m = 3
输出: 3
```

**示例 2：**

```
输入: n = 10, m = 17
输出: 2
```

**限制：**

- `1 <= n <= 10^5`
- `1 <= m <= 10^6`

## 二、解答

### （一）方法一：见笔记

```java
package math.easy;

import java.util.LinkedList;
import java.util.List;

/**
 * @author GJXAIOU
 * @create 2020/04/12 15:17
 */
public class Offer62 {
    public int lastRemaining(int n, int m) {
        if (n == 0) {
            return 0;
        }
        List<Integer> inputValueList = new LinkedList<>();
        for (int i = 0; i < n; i++) {
            inputValueList.add(i);
        }

        int temp = n;
        temp = getLive(temp, m);
        return temp - 1;
    }

    public static int getLive(int i, int num) {
        if (i == 1) {
            return 1;
        }
        // 计算出新编号对应的旧编号，将该旧编号作为下一次计算的新编号
        return (getLive(i - 1, num) + num - 1) % i + 1;
    }
}

```



### （二）方法二

==只关心最终活着那个人的序号变化==

### 1 约瑟夫问题

这个问题实际上是约瑟夫问题，这个问题描述是

N个人围成一圈，第一个人从1开始报数，报M的将被杀掉，下一个人接着从1开始报。如此反复，最后剩下一个，求最后的胜利者。

### 2 问题转换

既然约塞夫问题就是用人来举例的，那我们也给每个人一个编号（索引值），每个人用字母代替

下面这个例子是 `N=8 m=3` 的例子

我们定义 `F(n,m)` 表示最后剩下那个人的索引号，因此我们只关系最后剩下来这个人的索引号的变化情况即可

![image-20200507204644755](Offer62.%E5%9C%86%E5%9C%88%E4%B8%AD%E6%9C%80%E5%90%8E%E5%89%A9%E4%B8%8B%E7%9A%84%E6%95%B0%E5%AD%97.resource/image-20200507204644755.png)

==从 8 个人开始，每次杀掉一个人，去掉被杀的人，然后把杀掉那个人之后的第一个人作为开头重新编号==

第一次 C 被杀掉，人数变成 7，D 作为开头，（最终活下来的 G 的编号从 6 变成 3 ）
第二次 F 被杀掉，人数变成 6，G 作为开头，（最终活下来的G的编号从 3 变成 0）
第三次 A 被杀掉，人数变成 5，B 作为开头，（最终活下来的G的编号从 0 变成 3）
==以此类推，当只剩一个人时，他的编号必定为0！（重点！）==

### 3 最终活着的人编号的反推

现在我们知道了 G 的索引号的变化过程，那么我们反推一下
从N = 7 到 N = 8 的过程

如何才能将 N = 7 的排列变回到 N = 8 呢？

==我们先把被杀掉的 C 补充到最后，然后右移 m 个人，发现溢出了，再把溢出的补充在最前面==

神奇了 经过这个操作就恢复了 N = 8 的排列了！

![image-20200507204707972](Offer62.%E5%9C%86%E5%9C%88%E4%B8%AD%E6%9C%80%E5%90%8E%E5%89%A9%E4%B8%8B%E7%9A%84%E6%95%B0%E5%AD%97.resource/image-20200507204707972.png)

因此我们可以推出递推公式 $f(8,3) = [f(7, 3) + 3] \% 8$
进行推广泛化，即 $f(n,m) = [f(n-1, m) + m] \% n$

### 4 递推公式的导出

再把 n=1 这个最初的情况加上，就得到递推公式

$f(n,m)=\left\{ \begin{aligned} &0 & & {n = 1}\\ &[f(n-1, m) + m] \% n & & {n > 1} \\ \end{aligned} \right.$
为了更好理解，这里是拿着约瑟夫环的结论进行举例解释，具体的数学证明请参考维基百科。

##### 5 代码

照着递推公式写即可，也可写成递归的形式

```java
class Solution {
    public int lastRemaining(int n, int m) {
        int pos = 0; // 最终活下来那个人的初始位置
        for(int i = 2; i <= n; i++){
            pos = (pos + m) % i;  // 每次循环右移
        }
        return pos;
    }
}
```

