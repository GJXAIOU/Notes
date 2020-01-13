# AlgorithmMediumDay01



## 一、KMP

### （一）基本知识

- 子序列：可以连续可以不连续；
- 子数组（子串）：必须是连续的；
- 但是题目中子序列和子数组可能不区分；



### （二）作用

- 解决原始问题：给定两个字符串 str 和 match，长度分别为 N 和 M，实现一个算法，如果字符串 str 中含有子串 match，则返回 match 在 str 中的开始位置，不含有则返回 -1。

  **举例**

  str=”acbc”, match=”bc”，返回 2。

  str=”acbc”, mathc=”bcc”, 返回 -1。

  **要求**

  如果 match 的长度大于 str 的长度（M > N），str 必然不会含有 match，则直接返回 -1，但是如果 N >= M，要求算法复杂度为 O(N)。



### （三）常规解法

每次都是从头开始一个个匹配，但是时间复杂度太高，最差的情况下是：$$O(N * M)$$

**大致思路**：从左到右遍历 str 的每一个字符，然后看如果以当前字符作为第一个字符出发是否匹配出 match。比如str="aaaaaaaaaaaaaaaaab"，match="aaaab"。从 str[0] 出发， 开始匹配，匹配到 str[4]='a' 时发现和 match[4]='b' 不一样，所以匹配失败，说明从 str[0] 出发是不行的。从 str[1] 出发，开始匹配，匹配到 str[5]='a' 时发现和 match[4]='b' 不一样， 所以匹配失败，说明从 str[1] 出发是不行的。从str[2...12]出发，都会一直失败。从 str[13] 出 发，开始匹配，匹配到 str[17]=='b'，时发现和 match[4]='b'—样，match己经全部匹配完，说 明匹配成功，返回13。普通解法的时间复杂度较高，从每个字符串发时，匹配的代价都可能是O(M)，那么一共有N个字符，所以整体的时间复杂度为 O（N*M）。普通解法的时间复杂度这么高，是因为每次遍历到一个字符时，检查工作相当于从无开始，之前的遍历检查 不能优化当前的遍历检查。

**总结**： 我们发现每次匹配前面的并没有给后面一些指导信息



### （四）KMP算法思路

- **首先根据 match 生成最大前后缀匹配长度数组 ：**

  首先生成 match 字符串的 nextArr 数组，这个数组的长度与 match 字符串的长度一 样，nextArr[i] 的含义是在 match[i] 之前的字符串 match[0..i-1]中，心须以 match[i-1] 结尾的后缀子串（不能包含match[0]）与必须以 match[0] 开头的前缀子串（不能包含match[i - 1]）最大匹配长度是多少。这个长度就是 nextArr[i] 的值。比如，match="aaaab"字符串，nextArr[4] 的值该是多少呢？`match[4]=="b"`，所以它之前的字符串为 "aaaa"，根据定义这个字符串的后缀子串和前缀子串最大匹配为 "aaa"。也就是当后缀子串等于 match[1...3] = "aaa"，前缀子串等于 `match[0..2]=="aaa"` 时，这时前缀和后缀不仅相等，而且是所有前缀和后缀的可能性中

注：数组中值表示一个字符之前的所有字符串的最长前缀和最长后缀的匹配长度（前缀不包括数组中最后一个字符，后缀不包含第一个字符）

示例：对于数组： a b c a b c d，则元素 d  的最长XXXX为 3

前缀： a       ab         abc             abca          abcab   不能包含最后一个，至此截止，所以最长为 3.    

后缀： c        bc         abc             cabc          bcabc

值：  不等   不等    相等，为3     不等           不等



**2  从str[i]字符出发---匹配到j位置上发现开始出现不匹配：**

假设从 str[i] 字符出发时候，匹配到 j  位置的字符发现与 match 中的字符不一致，也就是说， str[i] 与 match[0] 一样，并且从这个位置开始一直可以匹配，即 str[i..j-1] 与 match[0..j-i-1] 一样，知道发现 str[j] != match[j -i]，匹配停止，如图所示。

![image-20200102144804517](AlgorithmMediumDay01.resource/image-20200102144804517.png)



**3 下一次 match 数组往右滑动， 滑动大小为 match 当前字符的最大前后缀匹配长度，再让 str[j] 与 match[k] 进行下一次匹配~**

那么下一次的匹配检查不再像普通解法那样退回到str[i+1]重新开始与 match[0] 的匹配过程，而是直接让 str[j] 与 match[k] 进行匹配检查，如图所示.在图中，在 str 中要匹配的位置仍是 j，而不进行退回。对 match来说，相当于向右滑动，让 match[k] 滑动到与 str[j] 同一个位置上，然后进行后续的匹配检查。普通解法 str 要退回到 i+1 位置，然后让 str[i+1] 与 match[0] 进行匹配，而我们的解法在匹配的过程中一直进行这样的滑动匹配的过程，直到在 str 的某一个位置把 match 完全匹配完，就说明str 中有 match。如果 match 滑到最后也没匹配出来，就说明str 中没有 match。

![img](AlgorithmMediumDay01.resource/20180617171300922.png)



**4 注意为什么加快，主要是str[i]--c前那段不用考虑，因为肯定不能匹配~**

**若有匹配 则出现更大的前后缀匹配长度 ，与最初nextArr[] 定义违背。**

![img](AlgorithmMediumDay01.resource/20180617171438808.png)

**关于nextArr数组的求解：**

![img](https://img-blog.csdn.net/20180617211712204?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1b2R1bzE4dXA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![img](https://img-blog.csdn.net/20180617214652560?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1b2R1bzE4dXA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



![img](https://img-blog.csdn.net/20180617215337314?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1b2R1bzE4dXA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)





#### 讨论计算复杂度：

![img](AlgorithmMediumDay01.resource/20180617225822654.png)



代码

```java
package nowcoder.advanced.class01;

public class KMP {
    public static int getIndexOf(String s, String m) {
        if (s == null || m == null || m.length() < 1 || s.length() < m.length()) {
            return -1;
        }
        char[] ss = s.toCharArray();
        char[] ms = m.toCharArray();
        int si = 0;
        int mi = 0;
        int[] next = getNextArray(ms);
        while (si < ss.length && mi < ms.length) {
            if (ss[si] == ms[mi]) {
                si++;
                mi++;
                // 数组中值等于 -1 ，说明是第一个元素，说明当前 str1  中值连 str2 第一个字母都匹配不上，则直接从 str1 的下一个开始进行匹配
            } else if (next[mi] == -1) {
                si++;
            } else {
                mi = next[mi];
            }
        }
        return mi == ms.length ? si - mi : -1;
    }


    // 求解 next 数组方法
    public static int[] getNextArray(char[] str2) {
        if (str2.length == 1) {
            return new int[]{-1};
        }
        int[] next = new int[str2.length];
        next[0] = -1;
        next[1] = 0;
        // 需要求值的位置
        int pos = 2;
        int cn = 0;
        while (pos < next.length) {
            if (str2[pos - 1] == str2[cn]) {
                next[pos++] = ++cn;
            } else if (cn > 0) {
                cn = next[cn];
            } else {
                next[pos++] = 0;
            }
        }
        return next;
    }

    public static void main(String[] args) {
        int indexOf = getIndexOf("abcabdad", "bd");
        System.out.println(indexOf);
    }
}



```



### （五）相关题目

- （京东）给定一个字符串 如何加最短的字符（只能在原始串的后面进行添加）使其构成一个长的字符串且包含两个原始字符串，并且开始位置还得不一样

  示例：abcabc ---->abcabcabc 最少增加 3 个

**思路：**其实就是最大前后缀长度数组

**步骤**：求原字符串每一个的 next 数组，得到原字符串最大长度**后一个**的 next 数组值。该值表明原字符串的最大可复用长度（next 数组的意义），然后用原字符串长度减去最大可复用长度，得到应该添加字符的长度，即从原字符串倒数取得，添加即可。

总结： 在 KMP 中 nextArr 数组基础上多求一位终止位，将不是的补上即可

- 给定两个树（值可以相同或者不同） 判断数 1 的某棵子树是否包含树 2（结构和值完全相同），  是则返回true

子树就是从某个头结点开始下面所有子节点都要，下图第一个图就是可以 true，另一个图就是 false。

<img src="AlgorithmMediumDay01.resource/image-20191231154124355.png" alt="image-20191231154124355" style="zoom:50%;" />



**思路**： 把一棵树序列化（可以先序、中序、后续）为字符串（字符数组）  。如果 str2 是 str1的子串 则T2也是T1的子树。



- 判断一个大字符串是否由一个小字符串重复得到，就是某个字符串是否为某个小字符串 * n 得到；

**思路**：转换为 一个前缀和后缀相差整数倍的问题



## 二、Manacher算法：

### 原始问题

Manacher算法是由题目“求字符串中最长回文子串的长度”而来。比如`abcdcb`的最长回文子串为`bcdcb`，其长度为5。

我们可以遍历字符串中的每个字符，当遍历到某个字符时就比较一下其左边相邻的字符和其右边相邻的字符是否相同，如果相同则继续比较其右边的右边和其左边的左边是否相同，如果相同则继续比较……，我们暂且称这个过程为向外“扩”。当“扩”不动时，经过的所有字符组成的子串就是以当前遍历字符为中心的最长回文子串。

我们每次遍历都能得到一个最长回文子串的长度，使用一个全局变量保存最大的那个，遍历完后就能得到此题的解。但分析这种方法的时间复杂度：当来到第一个字符时，只能扩其本身即1个；来到第二个字符时，最多扩两个；……；来到字符串中间那个字符时，最多扩`(n-1)/2+1`个；因此时间复杂度为`1+2+……+(n-1)/2+1`即`O(N^2)`。但Manacher算法却能做到`O(N)`。

Manacher算法中定义了如下几个概念：

- 回文半径：串中某个字符最多能向外扩的字符个数称为该字符的回文半径。比如`abcdcb`中字符`d`，能扩一个`c`，还能再扩一个`b`，再扩就到字符串右边界了，再算上字符本身，字符`d`的回文半径是3。
- 回文半径数组`pArr`：长度和字符串长度一样，保存串中每个字符的回文半径。比如`charArr="abcdcb"`，其中`charArr[0]='a'`一个都扩不了，但算上其本身有`pArr[0]=1`；而`charArr[3]='d'`最多扩2个，算上其本身有`pArr[3]=3`。
- 最右回文右边界`R`：遍历过程中，“扩”这一操作扩到的最右的字符的下标。比如`charArr=“abcdcb”`，当遍历到`a`时，只能扩`a`本身，向外扩不动，所以`R=0`；当遍历到`b`时，也只能扩`b`本身，所以更新`R=1`；但当遍历到`d`时，能向外扩两个字符到`charArr[5]=b`，所以`R`更新为5。
- 最右回文右边界对应的回文中心`C`：`C`与`R`是对应的、同时更新的。比如`abcdcb`遍历到`d`时，`R=5`，`C`就是`charArr[3]='d'`的下标`3`。

处理回文子串长度为偶数的问题：上面拿`abcdcb`来举例，其中`bcdcb`属于一个回文子串，但如果回文子串长度为偶数呢？像`cabbac`，按照上面定义的“扩”的逻辑岂不是每个字符的回文半径都是0，但事实上`cabbac`的最长回文子串的长度是6。因为我们上面“扩”的逻辑默认是将回文子串当做奇数长度的串来看的，因此我们在使用Manacher算法之前还需要将字符串处理一下，这里有一个小技巧，那就是将字符串的首尾和每个字符之间加上一个特殊符号，这样就能将输入的串统一转为奇数长度的串了。比如`abba`处理过后为`#a#b#b#a`，这样的话就有`charArr[4]='#'`的回文半径为4，也即原串的最大回文子串长度为4。相应代码如下：

```
public static char[] manacherString(String str){
  char[] source = str.toCharArray();
  char chs[] = new char[str.length() * 2 + 1];
  for (int i = 0; i < chs.length; i++) {
    chs[i] = i % 2 == 0 ? '#' : source[i / 2];
  }
  return chs;
}
复制代码
```

接下来分析，BFPRT算法是如何利用遍历过程中计算的`pArr`、`R`、`C`来为后续字符的回文半径的求解加速的。

首先，情况1是，遍历到的字符下标`cur`在`R`的右边（起初另`R=-1`），这种情况下该字符的最大回文半径`pArr[cur]`的求解无法加速，只能一步步向外扩来求解。



![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e8308cb40c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



情况2是，遍历到的字符下标`cur`在`R`的左边，这时`pArr[cur]`的求解过程可以利用之前遍历的字符回文半径信息来加速。分别做`cur`、`R`关于`C`的对称点`cur'`和`L`：

- 如果从`cur'`向外扩的最大范围的左边界没有超过`L`，那么`pArr[cur]=pArr[cur']`。

    

    ![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e83279de43?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

    

    证明如下：

    

    ![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e83231d1ba?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

    

    由于之前遍历过`cur'`位置上的字符，所以该位置上能扩的步数我们是有记录的（`pArr[cur']`），也就是说`cur'+pArr[cur']`处的字符`y'`是不等于`cur'-pArr[cur']`处的字符`x'`的。根据`R`和`C`的定义，整个`L`到`R`范围的字符是关于`C`对称的，也就是说`cur`能扩出的最大回文子串和`cur'`能扩出的最大回文子串相同，因此可以直接得出`pArr[cur]=pArr[cur']`。

- 如果从`cur'`向外扩的最大范围的左边界超过了`L`，那么`pArr[cur]=R-cur+1`。

    

    ![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e83297b708?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

    

    证明如下：

    

    ![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e8328b74da?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

    

    `R`右边一个字符`x`，`x`关于`cur`对称的字符`y`，`x,y`关于`C`对称的字符`x',y'`。根据`C,R`的定义有`x!=x'`；由于`x',y'`在以`cur'`为中心的回文子串内且关于`cur'`对称，所以有`x'=y'`，可推出`x!=y'`；又`y,y'`关于`C`对称，且在`L,R`内，所以有`y=y'`。综上所述，有`x!=y`，因此`cur`的回文半径为`R-cur+1`。

- 以`cur'`为中心向外扩的最大范围的左边界正好是`L`，那么`pArr[cur] >= （R-cur+1）`

    

    ![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e8329df8aa?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

    

    这种情况下，`cur'`能扩的范围是`cur'-L`，因此对应有`cur`能扩的范围是`R-cur`。但`cur`能否扩的更大则取决于`x`和`y`是否相等。而我们所能得到的前提条件只有`x!=x'`、`y=y'`、`x'!=y'`，无法推导出`x,y`的关系，只知道`cur`的回文半径最小为`R-cur+1`（算上其本身），需要继续尝试向外扩以求解`pArr[cur]`。

综上所述，`pArr[cur]`的计算有四种情况：暴力扩、等于`pArr[cur']`、等于`R-cur+1`、从`R-cur+1`继续向外扩。使用此算法求解原始问题的过程就是遍历串中的每个字符，每个字符都尝试向外扩到最大并更新`R`（只增不减），每次`R`增加的量就是此次能扩的字符个数，而`R`到达串尾时问题的解就能确定了，因此时间复杂度就是每次扩操作检查的次数总和，也就是`R`的变化范围（`-1~2N`，因为处理串时向串中添加了`N+1`个`#`字符），即`O(1+2N)=O(N)`。

整体代码如下：

```
public static int maxPalindromeLength(String str) {
  char charArr[] = manacherString(str);
  int pArr[] = new int[charArr.length];
  int R = -1, C = -1;
  int max = Integer.MIN_VALUE;
  for (int i = 0; i < charArr.length; i++) {
    pArr[i] = i > R ? 1 : Math.min(pArr[C * 2 - i], R - i);
    while (i + pArr[i] < charArr.length && i - pArr[i] > -1) {
      if (charArr[i + pArr[i]] == charArr[i - pArr[i]]) {
        pArr[i]++;
      } else {
        break;
      }
    }
    if (R < i + pArr[i]) {
      R = i + pArr[i]-1;
      C = i;
    }
    max = Math.max(max, pArr[i]);
  }
  return max-1;
}

public static void main(String[] args) {
  System.out.println(maxPalindromeLength("zxabcdcbayq"));
}
复制代码
```

上述代码将四种情况的分支处理浓缩到了`7~14`行。其中第`7`行是确定加速信息：如果当前遍历字符在`R`右边，先算上其本身有`pArr[i]=1`，后面检查如果能扩再直接`pArr[i]++`即可；否则，当前字符的`pArr[i]`要么是`pArr[i']`（`i`关于`C`对称的下标`i'`的推导公式为`2*C-i`），要么是`R-i+1`，要么是`>=R-i+1`，可以先将`pArr[i]`的值置为这三种情况中最小的那一个，后面再检查如果能扩再直接`pArr[i]++`即可。

最后得到的`max`是处理之后的串（`length=2N+1`）的最长回文子串的半径，`max-1`刚好为原串中最长回文子串的长度。

### 进阶问题

给你一个字符串，要求添加尽可能少的字符使其成为一个回文字符串。

> 思路：当`R`第一次到达串尾时，做`R`关于`C`的对称点`L`，将`L`之前的字符串逆序就是结果。

--------------------------------

-----------------------------------------------------------------------------------------

### （一）Manacher 算法应用

给定一个字符串 str，返回 str 中最长回文子串的长度。

常规思路： 

- 总是从中间那个字符出发，向两边开始扩展，观察回文，然后每个字符都得进行一遍

   但是奇数位数字完全可以  偶数位数字不可以，例如 1221 对应的每位扩充结果为：1 1 1 1 ，但是结果为 4.会出错

-  经过规律发现 无论是哪种情况 总是 `最大值/2` 即为最大回文串长度

前提是要处理字符串形式，加入特殊字符占位 #（其实什么字符都行）进行填充（在开头结尾和每个中间加入）

 这种方法是暴力法，时间复杂度为$$O(n^2)$$-----而Manacher算法可以降到O(n)

<img src="AlgorithmMediumDay01.resource/20180619213235664.png" alt="img" style="zoom:50%;" />

**概念汇总**

- 回文直径：从某个值开始往两边扩充，能扩充的最长值；

- 回文半径：见图

![image-20191231161002607](AlgorithmMediumDay01.resource/image-20191231161002607.png)

- 回文半径数组：还是从头到最后挨个求每个位置的回文半径，然后存入一个数组中

- 最右回文半径：如图所示，刚开始默认在 -1 位置，然后当到 0 位置，回文半径只能到 0 位置，所以最右的回文半径为 0  位置，当到 1 位置时候，回文半径扩充到 2 位置，则最右回文半径扩充到 2 位置，当移动到 3 位置时候，最右回文半径为 6

![image-20191231163310036](AlgorithmMediumDay01.resource/image-20191231163310036.png)

- 最右回文半径的中心：上面最后一个取得最右边界时候中心为 3，后面 4,5,6 的最右边界可能也是该位置，但是仅仅记录第一个使得最右回文边界到的中心，即 3。



### （二）Manacher 算法详解

针对 i 位置的回文串求法

- 如果 i 位置当前不在最右回文半径中，只能采用上面的方式暴力破：O(N )

    ![image-20191231164036523](AlgorithmMediumDay01.resource/image-20191231164036523.png)

- i 位置在回文右边界内：则设当前回文中心为 C，则 i 位置肯定是在回文中心 C 和回文右边界之间，做 i 位置关于中心 C 的对称点 i’，则针对 i’ 的所在的回文区域与整个回文区域关系有不同情况

    - i' 所在的回文区域在整个回文区域之间：O(1)

        ![image-20191231165149527](AlgorithmMediumDay01.resource/image-20191231165149527.png)

**则 i 回文长度大小和 i’ 回文长度相同**。证明：使用对称和区间内小回文，整个大区间大回文的结论可以证明 i 对应的区域是回文，并且是最大的。

- ​		i' 所在的回文区域在整个回文区域相交：O(1)

    ![image-20191231170010115](AlgorithmMediumDay01.resource/image-20191231170010115.png)

**则 i 回文长度为 i 到 R 长度**。

- i' 所在的回文区域与整个回文区域之间边界重叠

    ![image-20200101132409088](AlgorithmMediumDay01.resource/image-20200101132409088.png)

这种情况下，只能保证 i 到 R  区间的数据不需要进行暴力破，相当于可以从 R + 1 位置开始判断。



代码：

```java
package nowcoder.advanced.class01;

/**
 * @Author GJXAIOU
 * @Date 2020/1/1 14:22
 */
public class Manacher {
    public static char[] manacherString(String str) {
        char[] charArr = str.toCharArray();
        // 存放加入特殊字符的数组
        char[] res = new char[str.length() * 2 + 1];
        int index = 0;
        for (int i = 0; i != res.length; i++) {
            // (i & 1) == 0 用于判断 i 的奇偶性，因为如果 i 为奇数，则其二进制最后一位比为 1，则相当于 XXX1 & 0001，结果肯定为 1。
            res[i] = (i & 1) == 0 ? '#' : charArr[index++];
        }
        return res;
    }

    public static int maxLcpsLength(String str) {
        if (str == null || str.length() == 0) {
            return 0;
        }
        char[] charArr = manacherString(str);
        // 回文半径数组
        int[] pArr = new int[charArr.length];
        // index  为对称中心 C
        int index = -1;
        int pR = -1;
        int max = Integer.MIN_VALUE;
        for (int i = 0; i != charArr.length; i++) {
            // 2 * index - 1 就是对应 i' 位置，R> i,表示 i 在回文右边界里面，则最起码有一个不用验的区域，
            pArr[i] = pR > i ? Math.min(pArr[2 * index - i], pR - i) : 1;
            // 四种情况都让扩一下，其中 1 和 4 会成功，但是 2 ，3 会失败则回文右边界不改变；可以自己写成 if-else 问题。
            while (i + pArr[i] < charArr.length && i - pArr[i] > -1) {
                if (charArr[i + pArr[i]] == charArr[i - pArr[i]]) {
                    pArr[i]++;
                } else {
                    break;
                }
            }
            if (i + pArr[i] > pR) {
                pR = i + pArr[i];
                index = i;
            }
            max = Math.max(max, pArr[i]);
        }
        return max - 1;
    }

    public static void main(String[] args) {
        int length = maxLcpsLength("123abccbadbccba4w2");
        System.out.println(length);
    }
}

```



### （三）题目拓展

一个字符串，只能向后面添加字符，怎么将整个串都变成回文串，要求添加字符最短。

示例： ‘abc12321’  应添加  ‘cba’  变成  ’abc12321cba‘

即求，在包含最后一个字符情况下，最长回文串多少，前面不是的部分，逆序添加即可。

用manacher求得回文边界，发现边界与字符串最后位置，停止，求得回文中心C与有边界R，找到回文字符串，用原字符串减去该部分，在逆序就是结果。

```java
package nowcoder.advanced.class01;

/**
 * @Author GJXAIOU
 * @Date 2020/1/1 14:11
 */
public class LastAddString {
    public static char[] manacherString(String str) {
        char[] charArr = str.toCharArray();
        char[] res = new char[str.length() * 2 + 1];
        int index = 0;
        for (int i = 0; i != res.length; i++) {
            res[i] = (i & 1) == 0 ? '#' : charArr[index++];
        }
        return res;
    }

    public static String shortestEnd(String str) {
        if (str == null || str.length() == 0) {
            return null;
        }
        char[] charArr = manacherString(str);
        int[] pArr = new int[charArr.length];
        int index = -1;
        int pR = -1;
        int maxContainsEnd = -1;
        for (int i = 0; i != charArr.length; i++) {
            pArr[i] = pR > i ? Math.min(pArr[2 * index - i], pR - i) : 1;
            while (i + pArr[i] < charArr.length && i - pArr[i] > -1) {
                if (charArr[i + pArr[i]] == charArr[i - pArr[i]]) {
                    pArr[i]++;
                } else {
                    break;
                }
            }
            if (i + pArr[i] > pR) {
                pR = i + pArr[i];
                index = i;
            }
            if (pR == charArr.length) {
                maxContainsEnd = pArr[i];
                break;
            }
        }
        char[] res = new char[str.length() - maxContainsEnd + 1];
        for (int i = 0; i < res.length; i++) {
            res[res.length - 1 - i] = charArr[i * 2 + 1];
        }
        return String.valueOf(res);
    }

    public static void main(String[] args) {
        String yuanlai = LastAddString.shortestEnd("abcd123321");
        System.out.println(yuanlai);
    }
}
```



## 三、BFPRT算法：

题目：给你一个整型数组，返回其中第K小的数。

这道题可以利用荷兰国旗改进的`partition`和随机快排的思想：随机选出一个数，将数组以该数作比较划分为`<,=,>`三个部分，则`=`部分的数是数组中第几小的数不难得知，接着对`<`（如果第K小的数在`<`部分）或`>`（如果第K小的数在`>`部分）部分的数递归该过程，直到`=`部分的数正好是整个数组中第K小的数。这种做法不难求得时间复杂度的数学期望为`O(NlogN)`（以2为底）。但这毕竟是数学期望，在实际工程中的表现可能会有偏差，而BFPRT算法能够做到时间复杂度就是`O(NlogN)`。

BFPRT算法首先将数组按5个元素一组划分成`N/5`个小部分（最后不足5个元素自成一个部分），再这些小部分的内部进行排序，然后将每个小部分的中位数取出来再排序得到中位数：



![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e84ccac4b5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



BFPRT求解此题的步骤和开头所说的步骤大体类似，但是“随机选出一个的作为比较的那个数”这一步替换为上图所示最终选出来的那个数。

`O(NlogN)`的证明，为什么每一轮`partition`中的随机选数改为BFPRT定义的选数逻辑之后，此题的时间复杂度就彻底变为`O(NlogN)`了呢？下面分析一下这个算法的步骤：

BFPRT算法，接收一个数组和一个K值，返回数组中的一个数

1. 数组被划分为了`N/5`个小部分，每个部分的5个数排序需要`O(1)`，所有部分排完需要`O(N/5)=O(N)`
2. 取出每个小部分的中位数，一共有`N/5`个，递归调用BFPRT算法得到这些数中第`(N/5)/2`小的数（即这些数的中位数），记为`pivot`
3. 以`pivot`作为比较，将整个数组划分为`pivot`三个区域
4. 判断第K小的数在哪个区域，如果在`=`区域则直接返回`pivot`，如果在`<`或`>`区域，则将这个区域的数递归调用BFPRT算法
5. `base case`：在某次递归调用BFPRT算法时发现这个区域只有一个数，那么这个数就是我们要找的数。

代码示例：

```
public static int getMinKthNum(int[] arr, int K) {
  if (arr == null || K > arr.length) {
    return Integer.MIN_VALUE;
  }
  int[] copyArr = Arrays.copyOf(arr, arr.length);
  return bfprt(copyArr, 0, arr.length - 1, K - 1);
}

public static int bfprt(int[] arr, int begin, int end, int i) {
  if (begin == end) {
    return arr[begin];
  }
  int pivot = medianOfMedians(arr, begin, end);
  int[] pivotRange = partition(arr, begin, end, pivot);
  if (i >= pivotRange[0] && i <= pivotRange[1]) {
    return arr[i];
  } else if (i < pivotRange[0]) {
    return bfprt(arr, begin, pivotRange[0] - 1, i);
  } else {
    return bfprt(arr, pivotRange[1] + 1, end, i);
  }
}

public static int medianOfMedians(int[] arr, int begin, int end) {
  int num = end - begin + 1;
  int offset = num % 5 == 0 ? 0 : 1;
  int[] medians = new int[num / 5 + offset];
  for (int i = 0; i < medians.length; i++) {
    int beginI = begin + i * 5;
    int endI = beginI + 4;
    medians[i] = getMedian(arr, beginI, Math.min(endI, end));
  }
  return bfprt(medians, 0, medians.length - 1, medians.length / 2);
}

public static int getMedian(int[] arr, int begin, int end) {
  insertionSort(arr, begin, end);
  int sum = end + begin;
  int mid = (sum / 2) + (sum % 2);
  return arr[mid];
}

public static void insertionSort(int[] arr, int begin, int end) {
  if (begin >= end) {
    return;
  }
  for (int i = begin + 1; i <= end; i++) {
    for (int j = i; j > begin; j--) {
      if (arr[j] < arr[j - 1]) {
        swap(arr, j, j - 1);
      } else {
        break;
      }
    }
  }
}

public static int[] partition(int[] arr, int begin, int end, int pivot) {
  int L = begin - 1;
  int R = end + 1;
  int cur = begin;
  while (cur != R) {
    if (arr[cur] > pivot) {
      swap(arr, cur, --R);
    } else if (arr[cur] < pivot) {
      swap(arr, cur++, ++L);
    } else {
      cur++;
    }
  }
  return new int[]{L + 1, R - 1};
}

public static void swap(int[] arr, int i, int j) {
  int tmp = arr[i];
  arr[i] = arr[j];
  arr[j] = tmp;
}

public static void main(String[] args) {
  int[] arr = {6, 9, 1, 3, 1, 2, 2, 5, 6, 1, 3, 5, 9, 7, 2, 5, 6, 1, 9};
  System.out.println(getMinKthNum(arr,13));
}
复制代码
```

时间复杂度为`O(NlogN)`（底数为2）的证明，分析`bfprt`的执行步骤（假设`bfprt`的时间复杂度为`T(N)`）：

1. 首先数组5个5个一小组并内部排序，对5个数排序为`O(1)`，所有小组排好序为`O(N/5)=O(N)`
2. 由步骤1的每个小组抽出中位数组成一个中位数小组，共有`N/5`个数，递归调用`bfprt`求出这`N/5`个数中第`(N/5)/2`小的数（即中位数）为`T(N/5)`，记为`pivot`
3. 对步骤2求出的`pivot`作为比较将数组分为小于、等于、大于三个区域，由于`pivot`是中位数小组中的中位数，所以中位数小组中有`N/5/2=N/10`个数比`pivot`小，这`N/10`个数分别又是步骤1中某小组的中位数，可推导出至少有`3N/10`个数比`pivot`小，也即最多有`7N/10`个数比`pivot`大。也就是说，大于区域（或小于）最大包含`7N/10`个数、最少包含`3N/10`个数，那么如果第`i`大的数不在等于区域时，无论是递归`bfprt`处理小于区域还是大于区域，最坏情况下子过程的规模最大为`7N/10`，即`T(7N/10)`

综上所述，`bfprt`的`T(N)`存在推导公式：`T(N/5)+T(7N/10)+O(N)`。根据 **基础篇** 中所介绍的Master公式可以求得`bfprt`的时间复杂度就是`O(NlogN)`（以2为底）。

## morris遍历二叉树

> 关于二叉树先序、中序、后序遍历的递归和非递归版本在【直通BAT算法（基础篇）】中有讲到，但这6种遍历算法的时间复杂度都需要`O(H)`（其中`H`为树高）的额外空间复杂度，因为二叉树遍历过程中只能向下查找孩子节点而无法回溯父结点，因此这些算法借助栈来保存要回溯的父节点（递归的实质是系统帮我们压栈），并且栈要保证至少能容纳下`H`个元素（比如遍历到叶子结点时回溯父节点，要保证其所有父节点在栈中）。而morris遍历则能做到时间复杂度仍为`O(N)`的情况下额外空间复杂度只需`O(1)`。

### 遍历规则

首先在介绍morris遍历之前，我们先把先序、中序、后序定义的规则抛之脑后，比如先序遍历在拿到一棵树之后先遍历头结点然后是左子树最后是右子树，并且在遍历过程中对于子树的遍历仍是这样。

忘掉这些遍历规则之后，我们来看一下morris遍历定义的标准：

1. 定义一个遍历指针`cur`，该指针首先指向头结点

2. 判断

    ```
    cur
    ```

    的左子树是否存在

    - 如果`cur`的左孩子为空，说明`cur`的左子树不存在，那么`cur`右移来到`cur.right`

    - 如果

        ```
        cur
        ```

        的左孩子不为空，说明

        ```
        cur
        ```

        的左子树存在，找出该左子树的最右结点，记为

        ```
        mostRight
        ```

        - 如果，`mostRight`的右孩子为空，那就让其指向`cur`（`mostRight.right=cur`），并左移`cur`（`cur=cur.left`）
        - 如果`mostRight`的右孩子不空，那么让`cur`右移（`cur=cur.right`），并将`mostRight`的右孩子置空

3. 经过步骤2之后，如果`cur`不为空，那么继续对`cur`进行步骤2，否则遍历结束。

下图所示举例演示morris遍历的整个过程：



![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e86eb7cac7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



### 先序、中序序列

遍历完成后对`cur`进过的节点序列稍作处理就很容易得到该二叉树的先序、中序序列：



![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e8a7c5bf81?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



示例代码：

```
public static class Node {
    int data;
    Node left;
    Node right;
    public Node(int data) {
        this.data = data;
    }
}

public static void preOrderByMorris(Node root) {
    if (root == null) {
        return;
    }
    Node cur = root;
    while (cur != null) {
        if (cur.left == null) {
            System.out.print(cur.data+" ");
            cur = cur.right;
        } else {
            Node mostRight = cur.left;
            while (mostRight.right != null && mostRight.right != cur) {
                mostRight = mostRight.right;
            }
            if (mostRight.right == null) {
                System.out.print(cur.data+" ");
                mostRight.right = cur;
                cur = cur.left;
            } else {
                cur = cur.right;
                mostRight.right = null;
            }
        }
    }
    System.out.println();
}

public static void mediumOrderByMorris(Node root) {
    if (root == null) {
        return;
    }
    Node cur = root;
    while (cur != null) {
        if (cur.left == null) {
            System.out.print(cur.data+" ");
            cur = cur.right;
        } else {
            Node mostRight = cur.left;
            while (mostRight.right != null && mostRight.right != cur) {
                mostRight = mostRight.right;
            }
            if (mostRight.right == null) {
                mostRight.right = cur;
                cur = cur.left;
            } else {
                System.out.print(cur.data+" ");
                cur = cur.right;
                mostRight.right = null;
            }
        }
    }
    System.out.println();
}

public static void main(String[] args) {
    Node root = new Node(1);
    root.left = new Node(2);
    root.right = new Node(3);
    root.left.left = new Node(4);
    root.left.right = new Node(5);
    root.right.left = new Node(6);
    root.right.right = new Node(7);
    preOrderByMorris(root);
    mediumOrderByMorris(root);

}
复制代码
```

这里值得注意的是：**morris遍历会来到一个左孩子不为空的结点两次**，而其它结点只会经过一次。因此使用morris遍历打印先序序列时，如果来到的结点无左孩子，那么直接打印即可（这种结点只会经过一次），否则如果来到的结点的左子树的最右结点的右孩子为空才打印（这是第一次来到该结点的时机），这样也就忽略了`cur`经过的结点序列中第二次出现的结点；而使用morris遍历打印中序序列时，如果来到的结点无左孩子，那么直接打印（这种结点只会经过一次，左中右，没了左，直接打印中），否则如果来到的结点的左子树的最右结点不为空时才打印（这是第二次来到该结点的时机），这样也就忽略了`cur`经过的结点序列中第一次出现的重复结点。

### 后序序列

使用morris遍历得到二叉树的后序序列就没那么容易了，因为对于树种的非叶结点，morris遍历最多会经过它两次，而我们后序遍历实在第三次来到该结点时打印该结点的。因此要想得到后序序列，仅仅改变在morris遍历时打印结点的时机是无法做到的。

但其实，在morris遍历过程中，如果在每次遇到第二次经过的结点时，将该结点的左子树的右边界上的结点从下到上打印，最后再将整个树的右边界从下到上打印，最终就是这个数的后序序列：



![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e8ace72a83?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)





![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e8ad102b9d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)





![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e8ae36d836?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)





![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e8c2a5f049?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



其中无非就是在morris遍历中在第二次经过的结点的时机执行一下打印操作。而从下到上打印一棵树的右边界，可以将该右边界上的结点看做以`right`指针为后继指针的链表，将其反转`reverse`然后打印，最后恢复成原始结构即可。示例代码如下（其中容易犯错的地方是`18`行和`19`行的代码不能调换）：

```
public static void posOrderByMorris(Node root) {
    if (root == null) {
        return;
    }
    Node cur = root;
    while (cur != null) {
        if (cur.left == null) {
            cur = cur.right;
        } else {
            Node mostRight = cur.left;
            while (mostRight.right != null && mostRight.right != cur) {
                mostRight = mostRight.right;
            }
            if (mostRight.right == null) {
                mostRight.right = cur;
                cur = cur.left;
            } else {
                mostRight.right = null;
                printRightEdge(cur.left);
                cur = cur.right;
            }
        }
    }
    printRightEdge(root);
}

private static void printRightEdge(Node root) {
    if (root == null) {
        return;
    }
    //reverse the right edge
    Node cur = root;
    Node pre = null;
    while (cur != null) {
        Node next = cur.right;
        cur.right = pre;
        pre = cur;
        cur = next;
    }
    //print 
    cur = pre;
    while (cur != null) {
        System.out.print(cur.data + " ");
        cur = cur.right;
    }
    //recover
    cur = pre;
    pre = null;
    while (cur != null) {
        Node next = cur.right;
        cur.right = pre;
        pre = cur;
        cur = next;
    }
}

public static void main(String[] args) {
    Node root = new Node(1);
    root.left = new Node(2);
    root.right = new Node(3);
    root.left.left = new Node(4);
    root.left.right = new Node(5);
    root.right.left = new Node(6);
    root.right.right = new Node(7);
    posOrderByMorris(root);
}
复制代码
```

### 时间复杂度分析

因为morris遍历中，只有左孩子非空的结点才会经过两次而其它结点只会经过一次，也就是说遍历的次数小于`2N`，因此使用morris遍历得到先序、中序序列的时间复杂度自然也是`O(1)`；但产生后序序列的时间复杂度还要算上`printRightEdge`的时间复杂度，但是你会发现整个遍历的过程中，所有的`printRightEdge`加起来也只是遍历并打印了`N`个结点：



![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e8c715ba38?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



因此时间复杂度仍然为`O(N)`。

> morris遍历结点的顺序不是先序、中序、后序，而是按照自己的一套标准来决定接下来要遍历哪个结点。
>
> morris遍历的独特之处就是充分利用了叶子结点的无效引用（引用指向的是空，但该引用变量仍然占内存），从而实现了`O(1)`的时间复杂度。


作者：Anwen
链接：https://juejin.im/post/5c6b9d4c6fb9a04a05403cbe
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

-------------

------------------

### （一）BFPRT 算法详解与应用：

 **这个算法其实就是用来解决 ： 求解最小/最大的 k 个数问题，不是第 K 个数。**

- 最暴力的方法是：遍历然后查找：$$O(N*log^N)$$

- 常规的做法 就是 平均时间复杂度为 O(n) 的 partition 函数的应用，划分结果为小于某个数的放左边，等于某个数的放中间，大于某个数的放右边，然后看等于的左右边界，例如第一次划分结果左右边界为：500 ~ 600 位，查找的是第 300 小的数，则将小于区域按照上面方式再次使用 partition 函数进行划分，如果等于区域命中就结束过程，依次类推。  
    - 缺点：是概率式的，无法避免最差的情况（因为每次选取的划分值是随机的，很难保证大于和小于该值的区间长度差不多）。因此最终期望为：$$O(N)$$

- BFPRT算法优点是确定性的，严格的 $$O(N)$$。



大概思路：

 将整个数组以5个数字一组划分 ,最后不满 5 个的单独成组，分别进行组内排序（组间不排序） ，因为只是 5 个数排序，所有时间复杂度为O(1)， 同时因为整个数组分为N/5组，所以则总共时间复杂度为O(N)。

3) 把每个组内的中位数拿出来构成一个新数组，若为偶数则拿其上中位数即可，数组长度为 N/5

4）递归调用该bfprt函数，传入刚才的中位数数组 和 中位数数组长度的一半 （继续求中位数）【BFPRT 函数格式为： int BFPRT（arr,k) 输入数组 arr 和要求的第 k 小的值】

5）将4）返回的值作为原始数组划分值，一直以该值作为 paritition 函数的划分值。



例子：

![BFPRT 示例](AlgorithmMediumDay01.resource/BFPRT%20%E7%A4%BA%E4%BE%8B.png)

每一步的时间复杂度为：假设原始数据量为 N，则

- 步骤一：O(1)

- 步骤二：O(N)

- 步骤三：O(N)

- 步骤四：因为是递归：T（N/5），最终获取划分值 P

- 步骤五：在 partition 算法中如果不是值正好在相等区域的情况下，则不是在小于就是在大于区间上，同时两个区间后续只有一个需要再次判断，另一个区间直接舍弃。

    判断最差的情况：左侧最差情况，就是找到最多能有多少数是小于划分值 P（即）最少多少数的确定大于划分值 P 的，所以至少共 3N/10 大于划分值。所以左边比它小的最多有7N/10。右边类似。
    
    所以最终该步骤左右时间复杂度为：$$T(\frac{7N}{10})$$ ，因为只算一个，所以最终仍然是：$$T(\frac{7N}{10})$$ 。
    
    所以整体时间复杂度为：$$T(N) = T(\frac{N}{5}) + T(\frac{7N}{10}) + O(N)$$，结果为 O(N)，证明见算法导论。



优点 :我们选择的基准一定将数组划分为 一定规模的子部分

![img](AlgorithmMediumDay01.resource/20180620233656220.png)

由上图我们可以看到： 最中间的中位数--- 右边比它大的**至少**有N/10 * 3 个（每个组又有两个大于该组中位数的数字）

​                                右边比它小的最多有N/10 * 7

​                             ---- 同理左边也是~

如下例子：  7为中位数 ----比它大的至少有 8 9 12 13 14 这几个数

![img](AlgorithmMediumDay01.resource/20180620233956888.png)







重点掌握思路 可以面试的时候告诉面试官~~



**代码：** ==还是有问题==

```java
package nowcoder.advanced.class01;

import java.sql.SQLOutput;

/**
 * @Author GJXAIOU
 * @Date 2020/1/1 16:13
 */
public class BFPRT {
    // 使用堆的方式来求解该题，时间复杂度为：O(N * logN)
    public static int[] getMinKNumsByHeap(int[] arr, int k) {
        if (k < 1 || k > arr.length) {
            return arr;
        }
        int[] kHeap = new int[k];
        for (int i = 0; i != k; i++) {
            heapInsert(kHeap, arr[i], i);
        }
        for (int i = k; i != arr.length; i++) {
            if (arr[i] < kHeap[0]) {
                kHeap[0] = arr[i];
                heapify(kHeap, 0, k);
            }
        }
        return kHeap;
    }

    public static void heapInsert(int[] arr, int value, int index) {
        arr[index] = value;
        while (index != 0) {
            int parent = (index - 1) / 2;
            if (arr[parent] < arr[index]) {
                swap(arr, parent, index);
                index = parent;
            } else {
                break;
            }
        }
    }

    public static void heapify(int[] arr, int index, int heapSize) {
        int left = index * 2 + 1;
        int right = index * 2 + 2;
        int largest = index;
        while (left < heapSize) {
            if (arr[left] > arr[index]) {
                largest = left;
            }
            if (right < heapSize && arr[right] > arr[largest]) {
                largest = right;
            }
            if (largest != index) {
                swap(arr, largest, index);
            } else {
                break;
            }
            index = largest;
            left = index * 2 + 1;
            right = index * 2 + 2;
        }
    }

    // ------------------分割线---------------------

    public static int[] getMinKNumsByBFPRT(int[] arr, int k) {
        if (k < 1 || k > arr.length) {
            return arr;
        }
        int minKth = getMinKthByBFPRT(arr, k);
        int[] res = new int[k];
        int index = 0;
        for (int i = 0; i != arr.length; i++) {
            if (arr[i] < minKth) {
                res[index++] = arr[i];
            }
        }
        for (; index != res.length; index++) {
            res[index] = minKth;
        }
        return res;
    }

    public static int getMinKthByBFPRT(int[] arr, int K) {
        int[] copyArr = copyArray(arr);
        return bfprt(copyArr, 0, copyArr.length - 1, K - 1);
    }

    public static int[] copyArray(int[] arr) {
        int[] res = new int[arr.length];
        for (int i = 0; i != res.length; i++) {
            res[i] = arr[i];
        }
        return res;
    }

    // 在 begin 到 end 范围内求第 i 小的数
    public static int bfprt(int[] arr, int begin, int end, int i) {
        if (begin == end) {
            return arr[begin];
        }
        // 划分值
        int pivot = medianOfMedians(arr, begin, end);
        int[] pivotRange = partition(arr, begin, end, pivot);
        if (i >= pivotRange[0] && i <= pivotRange[1]) {
            return arr[i];
        } else if (i < pivotRange[0]) {
            return bfprt(arr, begin, pivotRange[0] - 1, i);
        } else {
            return bfprt(arr, pivotRange[1] + 1, end, i);
        }
    }

    public static int medianOfMedians(int[] arr, int begin, int end) {
        int num = end - begin + 1;
        int offset = num % 5 == 0 ? 0 : 1;
        int[] mArr = new int[num / 5 + offset];
        for (int i = 0; i < mArr.length; i++) {
            int beginI = begin + i * 5;
            int endI = beginI + 4;
            mArr[i] = getMedian(arr, beginI, Math.min(end, endI));
        }
        return bfprt(mArr, 0, mArr.length - 1, mArr.length / 2);
    }

    public static int[] partition(int[] arr, int begin, int end, int pivotValue) {
        int small = begin - 1;
        int cur = begin;
        int big = end + 1;
        while (cur != big) {
            if (arr[cur] < pivotValue) {
                swap(arr, ++small, cur++);
            } else if (arr[cur] > pivotValue) {
                swap(arr, cur, --big);
            } else {
                cur++;
            }
        }
        int[] range = new int[2];
        // 等于区域最左边
        range[0] = small + 1;
        // 等于区域最右边
        range[1] = big - 1;
        return range;
    }

    public static int getMedian(int[] arr, int begin, int end) {
        insertionSort(arr, begin, end);
        int sum = end + begin;
        int mid = (sum / 2) + (sum % 2);
        return arr[mid];
    }

    public static void insertionSort(int[] arr, int begin, int end) {
        for (int i = begin + 1; i != end + 1; i++) {
            for (int j = i; j != begin; j--) {
                if (arr[j - 1] > arr[j]) {
                    swap(arr, j - 1, j);
                } else {
                    break;
                }
            }
        }
    }

    public static void swap(int[] arr, int begin, int end) {
        arr[begin] = arr[begin] ^ arr[end];
        arr[end] = arr[begin] ^ arr[end];
        arr[begin] = arr[begin] ^ arr[end];
    }
    public static void printArray(int[] array){
        for (int i = 0; i < array.length; i++) {
            System.out.print(array[i] + " ");
        }
        System.out.println();
    }

    public static void main(String[] args) {
        int[] demo ={6,9,1,3,1,2,2,5,6,1,3,5,9,7,2,5,6,1,9};
        printArray(getMinKNumsByHeap(demo, 10));
        printArray(getMinKNumsByBFPRT(demo, 10));
    }
}

```

注意： 此时的partition和常规的partition函数不太一样 多了一个我们定义的基准划分值

​      且以此基准值划分 比它小的在左边 比它大的在右边 和它相等的在中间并将相等的左右边界存放在一个数组中

====第二课==



## 窗口： 

## 窗口最大值更新结构

### 最大值更新结构



![img](https://user-gold-cdn.xitu.io/2019/2/19/169045e8fb6ccd04?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



当向此结构放数据时会检查一下结构中的已有数据，从时间戳最大的开始检查，如果检查过程中发现该数据小于即将放入的数据则将其弹出并检查下一个，直到即将放入的数据小于正在检查的数据或者结构中的数据都被弹出了为止，再将要放入的数据放入结构中并盖上时间戳。如此每次从该结构取数据时，都会返回结构中时间戳最小的数据，也是目前为止进入过此结构的所有数据中最大的那一个。

此结构可以使用一个双端队列来实现，一端只用来放数据（放数据之前的检查过程可能会弹出其他数据），另一端用来获取目前为止出现过的最大值。

示例如下：

```
package top.zhenganwen.structure;

import java.util.LinkedList;

public class MaxValueWindow {

  private LinkedList<Integer> queue;
  public MaxValueWindow() {
    this.queue = new LinkedList();
  }

  //更新窗口最大值
  public void add(int i){
    while (!queue.isEmpty() && queue.getLast() <= i) {
      queue.pollLast();
    }
    queue.add(i);
  }

  //获取窗口最大值
  public int getMax() {
    if (!queue.isEmpty()) {
      return queue.peek();
    }
    return Integer.MIN_VALUE;
  }

  //使窗口最大值过期
  public void expireMaxValue() {
    if (!queue.isEmpty()) {
      queue.poll();
    }
  }

  public static void main(String[] args) {
    MaxValueWindow window = new MaxValueWindow();
    window.add(6);
    window.add(4);
    window.add(9);
    window.add(8);
    System.out.println(window.getMax());//9
    window.expireMaxValue();
    System.out.println(window.getMax());//8
  }
}
复制代码
```

### 例题

#### 窗口移动

给你一个长度为`N`的整型数组和大小为`W`的窗口，用一个长度为`N-W+1`的数组记录窗口从数组由左向右移动过程中窗口内最大值。

对于数组`[1,2,3,4,5,6,7]`和窗口大小为`3`，窗口由左向右移动时有：

- `[1,2,3],4,5,6,7`，窗口起始下标为0时，框住的数是`1,2,3`，最大值是3
- `1,[2,3,4],5,6,7`，最大值是4
- `1,2,[3,4,5],6,7`，最大值是5
- ……

因此所求数组是`[3,4,5,6,7]`。

> 思路：前面介绍的窗口最大值更新结构的特性是，先前放入的数如果还存在于结构中，那么该数一定比后放入的数都大。此题窗口移动的过程就是从窗口中减一个数和增一个数的过程。拿`[1,2,3],4`到`1,[2,3,4]`这一过程分析：首先`[1,2,3],4`状态下的窗口应该只有一个值`3`（因为先加了1，加2之前弹出了1，加3之前弹出了2）；转变为`1,[2,3,4]`的过程就是向窗口先减一个数`1`再加一个数`4`的过程，因为窗口中不含`1`所以直接加一个数`4`（弹出窗口中的`3`，加一个数`4`）。

代码示例：

```
public static void add(int arr[], int index, LinkedList<Integer> queue) {
  if (queue == null) {
    return;
  }
  while (!queue.isEmpty() && arr[queue.getLast()] < arr[index]) {
    queue.pollLast();
  }
  queue.add(index);
}

public static void expireIndex(int index, LinkedList<Integer> queue) {
  if (queue == null) {
    return;
  }
  if (!queue.isEmpty() && queue.peek() == index) {
    queue.pollFirst();
  }
}

public static int[] maxValues(int[] arr, int w) {
  int[] res = new int[arr.length - w + 1];
  LinkedList<Integer> queue = new LinkedList();
  for (int i = 0; i < w; i++) {
    add(arr, i, queue);
  }
  for (int i = 0; i < res.length; i++) {
    res[i] = queue.peek();
    if (i + w <= arr.length - 1) {
      expireIndex(i, queue);
      add(arr, i + w, queue);
    }
  }
  for (int i = 0; i < res.length; i++) {
    res[i] = arr[res[i]];
  }
  return res;
}

public static void main(String[] args) {
  int[] arr = {3, 2, 1, 5, 6, 2, 7, 8, 10, 6};
  System.out.println(Arrays.toString(maxValues(arr,3)));//[3, 5, 6, 6, 7, 8, 10, 10]
}
复制代码
```

这里需要的注意的是，针对这道题将窗口最大值更新结构的`add`和`expire`方法做了改进（结构中存的是值对应的下标）。例如`[2,1,2],-1->2,[1,2,-1]`，应当翻译为`[2,1,2],-1`状态下的窗口最大值为2下标上的数`2`，变为`2,[1,2,-1]`时应当翻译为下标为0的数从窗口过期了，而不应该是数据`2`从窗口过期了（这样会误删窗口中下标为2的最大值2）。

#### 求达标的子数组个数

给你一个整型数组，判断其所有子数组中最大值和最小值的差值不超过`num`（如果满足则称该数组达标）的个数。（子数组指原数组中任意个连续下标上的元素组成的数组）

暴力解：遍历每个元素，再遍历以当前元素为首的所有子数组，再遍历子数组找到其中的最大值和最小值以判断其是否达标。很显然这种方法的时间复杂度为`o(N^3)`，但如果使用最大值更新结构，则能实现`O(N)`级别的解。

如果使用`L`和`R`两个指针指向数组的两个下标，且`L`在`R`的左边。当`L~R`这一子数组达标时，可以推导出以`L`开头的长度不超过`R-L+1`的所有子数组都达标；当`L~R`这一子数组不达标时，无论`L`向左扩多少个位置或者`R`向右扩多少个位置，`L~R`还是不达标。

`O(N)`的解对应的算法是：`L`和`R`都从0开始，`R`先向右移动，`R`每右移一个位置就使用最大值更新结构和最小值更新结构记录一下`L~R`之间的最大值和最小值的下标，当`R`移动到如果再右移一个位置`L~R`就不达标了时停止，这时以当前`L`开头的长度不超过`R-L+1`的子数组都达标；然后`L`右移一个位置，同时更新一下最大值、最小值更新结构（`L-1`下标过期了），再右移`R`至`R`如果右移一个位置`L~R`就不达标了停止（每右移`R`一次也更新最大、小值更新结构）……；直到`L`到达数组尾元素为止。将每次`R`停止时，`R-L+1`的数量累加起来就是`O(N)`的解，因为`L`和`R`都只向右移动，并且每次`R`停止时，以`L`开头的达标子串的数量直接通过`R-L+1`计算，所以时间复杂度就是将数组遍历了一遍即`O(N)`。

示例代码：

```
public static int getComplianceChildArr(int arr[], int num) {
  //最大值、最小值更新结构
  LinkedList<Integer> maxq = new LinkedList();
  LinkedList<Integer> minq = new LinkedList<>();
  int L = 0;
  int R = 0;
  maxq.add(0);
  minq.add(0);
  int res = 0;
  while (L < arr.length) {
    while (R < arr.length - 1) {
      while (!maxq.isEmpty() && arr[maxq.getLast()] <= arr[R + 1]) {
        maxq.pollLast();
      }
      maxq.add(R + 1);
      while (!minq.isEmpty() && arr[minq.getLast()] >= arr[R + 1]) {
        minq.pollLast();
      }
      minq.add(R + 1);
      if (arr[maxq.peekFirst()] - arr[minq.peekFirst()] > num) {
        break;
      }
      R++;
    }
    res += (R - L + 1);
    if (maxq.peekFirst() == L) {
      maxq.pollFirst();
    }
    if (minq.peekFirst() == L) {
      minq.pollFirst();
    }
    L++;
  }
  return res;
}

public static void main(String[] args) {
  int[] arr = {1, 2, 3, 5};
  System.out.println(getComplianceChildArr(arr, 3));//9
}
```

作者：Anwen
链接：https://juejin.im/post/5c6b9d4c6fb9a04a05403cbe
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

---

-----



介绍窗口以及窗口内最大值或者最小值的更新结构（单调双向队列）**

**什么是窗口：**

就是一个数组------有L和R指针，默认两个指针均位于数组的最左边即下标为 -1 的位置， 当有数字进入时R向右移动  当有数字删除时则L向右移动 且L 和R 不会回退且 L 不能到 R 右边~



思路： **双端队列（链表）**：双端队列中既需要放置数据又需要放置位置下标，本质上存放下标就行，对应值从数组中就能获取。

**可以从头 尾入 可以从尾 头出**

**规则： L< R  且 L,R 永远不回退~**

![增加元素过程](AlgorithmMediumDay01.resource/%E5%A2%9E%E5%8A%A0%E5%85%83%E7%B4%A0%E8%BF%87%E7%A8%8B.jpg)



**分析逻辑为：** 如果想实现窗口内最大值的更新结构： 使得双端队列保证从大到小的顺序 

当放入元素时候 ：

​           （R增加） 头部始终存放的是当前最大的元素--- 如果即将进入双端队列的元素（因为 R 增加而从数组中取出放入双端队列中的元素）比上一个进入双端队列的元素小，则从尾部进入，新进入的元素直接连接在后面，否则 原来双端队列的尾部一直弹出（包含相等情况，因为晚过期） 直到为即将要放入的元素找到合适的位置，或者整个队列为空，然后放入新加入的元素。（见上面示例）

当窗口减数时候：

​            （L增加） ---L向右移---（index下标一定得保留）则需要检查当前头部元素index是否过期 若过期则需要从头部进行弹出

![img](AlgorithmMediumDay01.resource/20180621232650862.png)

**时间复杂度**：因为从到到位滑过，每个数只会进队列一次，出队列一次，在队列中删除的数是不找回的，因此时间复杂度为：$$O(N)$$



#### **具体应用：**

#### **生成窗口最大值数组：** 

**题目**

有一个整型数组 arr 和一个大小为 w 的窗口从数组的最左边滑到最右边，窗口每次向右滑一个位置。

例如，数组为 [4,3,5,4,3,3,6,7]，窗口大小为 3 时候：

[4 3 5] 4 3 3 6 7   窗口中最大值为：5

4 [3 5 4] 3 3 6 7   窗口中最大值为：5

4 3 [5 4 3] 3 6 7   窗口中最大值为：5

4 3 5 [4 3 3] 6 7   窗口中最大值为：4

4 3 5 4 [3 3 6] 7   窗口中最大值为：6

4 3 5 4 3 [3 6 7]   窗口中最大值为：7

如果数组长度为 n，窗口大小为 w，则一共产生 n - w + 1 个窗口的最大值。

请实现一个函数：

输入：整型数组 arr，窗口大小为 w。

输出：一个长度为 n - w + 1 的数组 res，res[i]表示每一种窗口状态下的最大值。

上面的结果应该返回{5,5,5,4,6,7}

**代码：**

```java
package nowcoder.advanced.class02;

import java.util.LinkedList;

/**
 * @Author GJXAIOU
 * @Date 2020/1/1 20:29
 */
public class SlidingWindowMaxArray {
    public static int[] getMaxWindow(int[] arr, int w) {
        if (arr == null || w < 1 || arr.length < w) {
            return null;
        }
        // LinkedList 就是一个标准的双向链表
        LinkedList<Integer> qmax = new LinkedList<Integer>();
        // 生成的结果数组
        int[] res = new int[arr.length - w + 1];
        int index = 0;
        for (int i = 0; i < arr.length; i++) {
            //更新双端队列，如果双端队列不为空，并且尾结点(存的是下标)对应数组中的值是否小于等于当前值
            while (!qmax.isEmpty() && arr[qmax.peekLast()] <= arr[i]) {
                qmax.pollLast();
            }
            // 上面一直弹出，直到不符合然后加上当前值。
            qmax.addLast(i);
            // 上面加法是通用的，但是减法是针对该题定制的
            // 当过期的时候（当窗口形成之后再扩充才算过期），窗口形成过程中不会过期
            if (qmax.peekFirst() == i - w) {
                qmax.pollFirst();
            }
            //判断下标过期
            if (i >= w - 1) {
                res[index++] = arr[qmax.peekFirst()];
            }
        }
        return res;
    }

    public static void printArray(int[] array) {
        for (int i = 0; i < array.length; i++) {
            System.out.print(array[i] + " ");
        }
        System.out.println();
    }

    public static void main(String[] args) {
        int[] arr = {4, 3, 5, 4, 3, 3, 6, 7};
        printArray(getMaxWindow(arr, 3));
    }
}
```



## 最大值-最小值<=num的子数组数量:

**题目**：给定数组 arr 和整数 num，共返回有多少个子数组满足如下情况：`max(arr[i...j]) - min(arr[i...j]) <= num`，其中 `max(arr[i...j])` 表示子数组 `arr[i...j]` 中最大值，`min(arr[i...j])` 表示子数组 `arr[i...j]` 中的最小值。

**要求**：如果数组长度为 N，请实现时间复杂度为 O（N）的解法。



- 子数组（必须连续）一共有：$$N^2$$ 个（0~1,0~2，。。。0~n；1~1，。。。）



#### 最简单思路： 暴力求解 两个for 依次遍历 

```java
package nowcoder.advanced.class02;

/**
 * @Author GJXAIOU
 * @Date 2020/1/1 20:59
 */
public class AllLessNumSubArray {
    // 暴力解法:O(N^3)
    public static int getNum1(int[] arr, int num) {
        int res = 0;
        // 双层 for 循环穷尽所有的子数组可能性。
        for (int start = 0; start < arr.length; start++) {
            for (int end = start; end < arr.length; end++) {
                if (isValid(arr, start, end, num)) {
                    res++;
                }
            }
        }
        return res;
    }

    public static boolean isValid(int[] arr, int start, int end, int num) {
        int max = Integer.MIN_VALUE;
        int min = Integer.MAX_VALUE;
        for (int i = start; i <= end; i++) {
            max = Math.max(max, arr[i]);
            min = Math.min(min, arr[i]);
        }
        return max - min <= num;
    }
}

```



分析时间复杂度： o(N3) n的3次方



#### 最优解思路：

- 如果有一个子数组 L~R 已经符合要求（并且其最大值为 max，最小值为 min），则其中内部的子数组一定也符合要求，因为内部空间（即范围小于 L ~ R）中的最大值只会 <= max，并且最小值只会 >=min，所有相减的结果肯定小于等于 num。

- 同理如果已经不达标 则往两边扩也肯定不达标（因为扩大返回只会导致 max 值变大，min 值变小）。

- 总计规律： 就是L一直往右走 不回退 R也跟着往右扩大范围

首先数组左边固定在 0 位置，然后右边界一直扩充，但是同时维护一个窗口内最大值更新结构和窗口内最小值更新结构，使得每次扩充值之后都可以比较当前数组是否满足最大值 - 最小值 <= max 的情况，比如当到 X 下标的时候是满足的，但是 X + 1 下标就不满足的时候，则表示以 0 开头的满足的子数组数目为 X +1 个（0 ~ 0,0 ~1，。。0 ~X）。

然后 L 缩一个位置，到 1 的位置，然后更新现有的最大和最小窗口更新结构（因为更新结构中 0 下标的值需要弹出），然后 R 继续向右走进行判断，直到不可以了就可以得到所有以 1 开头的子数组个数。

其他类型，以此类推。



代码：

```java
package nowcoder.advanced.class02;

import java.util.LinkedList;

/**
 * @Author GJXAIOU
 * @Date 2020/1/1 20:59
 */
public class AllLessNumSubArray {
    // 暴力解法:O(N^3)
    public static int getNum1(int[] arr, int num) {
        int res = 0;
        // 双层 for 循环穷尽所有的子数组可能性。
        for (int start = 0; start < arr.length; start++) {
            for (int end = start; end < arr.length; end++) {
                if (isValid(arr, start, end, num)) {
                    res++;
                }
            }
        }
        return res;
    }

    public static boolean isValid(int[] arr, int start, int end, int num) {
        int max = Integer.MIN_VALUE;
        int min = Integer.MAX_VALUE;
        for (int i = start; i <= end; i++) {
            max = Math.max(max, arr[i]);
            min = Math.min(min, arr[i]);
        }
        return max - min <= num;
    }

    /**
     * 使用双向最大最小值更新结构，时间复杂度为 O（N）
     */
    public static int getNum(int[] arr, int num) {
        if (arr == null || arr.length == 0) {
            return 0;
        }
        // 分别准备最大值和最小值更新结构
        LinkedList<Integer> qmax = new LinkedList<Integer>();
        LinkedList<Integer> qmin = new LinkedList<Integer>();
        int L = 0;
        int R = 0;
        int res = 0;
        while (L < arr.length) {
            while (R < arr.length) {
                while (!qmin.isEmpty() && arr[qmin.peekLast()] >= arr[R]) {
                    qmin.pollLast();
                }
                qmin.addLast(R);
                while (!qmax.isEmpty() && arr[qmax.peekLast()] <= arr[R]) {
                    qmax.pollLast();
                }
                qmax.addLast(R);
                // 不达标
                if (arr[qmax.getFirst()] - arr[qmin.getFirst()] > num) {
                    break;
                }
                R++;
            }
            if (qmin.peekFirst() == L) {
                qmin.pollFirst();
            }
            if (qmax.peekFirst() == L) {
                qmax.pollFirst();
            }
            res += R - L;
            // 换一个开头
            L++;
        }
        return res;
    }
}

```





# 单调栈：

问题描述：给定一个数组 请确定每个元素左右距离最近的比它大的数字

![img](AlgorithmMediumDay01.resource/20180624215316588.png)

**常规想法：** 到某一个元素时，通过两个for 分别获取其左边比它大的和右边比他大的数字 时间复杂度为 $$O(n^2)$$



**最优解思路（单调栈）**，栈中实际放置下标即可。O(N)

- 首先一个按照从大到小顺序排序的栈结构 ，若在压栈过程中发现要压栈的元素和栈顶的元素相比要大，则弹出当前栈顶元素，并从开始弹出处记录，要压栈的元素就是其右边离栈顶元素最近比它大的数，之后继续弹出的下一个即为栈顶元素左边距离最近的一个元素。

![img](AlgorithmMediumDay01.resource/20180624215839593.png)

注意： 到数组末尾时 但是栈中依然有元素 则此时元素弹出 右为null 而左边为栈中的下一元素

记得 观察 这个元素弹出的驱动是啥？  之前的是因为右边要压栈的比栈顶元素要大 所以可以弹出并记录信息

![img](AlgorithmMediumDay01.resource/20180625215803883.png)



**特殊情况：**若出现相等元素情况，则将下标放在一起，等到出现比它们大的数字时再都依次弹出即可。

![image-20200101220719195](AlgorithmMediumDay01.resource/image-20200101220719195.png)



## 具体应用：

## **1  构造数组的maxtree:**

定义二叉树的节点如下：

```
public class Node{
    public int value;
    public Node left;
    public Node right;
    public Node(int data){
        this.value=data;
    }
}
```



一个数组的MaxTree定义：

- 数组必须没有重复元素
- MaxTree是一棵二叉树，数组的每一个值对应一个二叉树节点
- 包括MaxTree树在内且在其中的每一棵子树上，值最大的节点都是树的头

给定一个没有重复元素的数组arr，写出生成这个数组的MaxTree的函数，要求如果数组长度为N，则时间负责度为O(N)、额外空间负责度为O(N)。

## 实现思路

对每一个元素，从左边和右边各选择第一个比这个元素大的值，选择值较小的元素作为父节点。
  在【生成窗口最大数组】里面，已经掌握了，在O(N)时间复杂度里面，找到每个元素位置最近的比元素大的元素，同个这个套路，就可以构造一棵MaxTree了。

## 证明

1 构造的不是森林
2 是一棵二叉树

证明：1  
  对于每一个树节点，都能往上找到一个节点，直到找到最大节点为止，这样所有树节点都有共同的父节点，这样构造出来的就是一棵树。

证明：2
  使用反证法解决，如果是一棵二叉树，那么对于每个作为父节点的元素，能够在元素的一边找到两个或两个以上的元素。存在如：[p, b1, x, b2]这样的结构，p是父节点、b1、b2为子节点, x为其他节点。

- 按照题目，可以设定：
      p > b1, p > b2
- 当b1 > b2：
      b2不会选择p作为父节点，可能选择b1作为父节点.
- 当b1 < b2：
      当x < b2时，b1不会选择p作为父节点，选择b2作为父节点.
      当x > b2时，b2不会选择p作为父节点，选择x作为父节点.

![img](https://img-blog.csdn.net/20180625221545761?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1b2R1bzE4dXA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

​         ![img](https://img-blog.csdn.net/201806252217091?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1b2R1bzE4dXA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



**注意**：树不要求一定要平衡

思路1 ： 按照大根堆思路建立 O(N)

思路2： 单调栈：O（N）

按照单调栈的思路找到每个元素左右最近比它大的元素---分以下几种情况进行讨论：

 1 若一个元素左右均是null 则它是全局最大的 直接作为根节点

 2 若一个元素左或者右 只存在一个 则只具有唯一的父节点

 3  若一个元素左右两个元素均存在 则选择其中最小的那个作为父节点

![img](https://img-blog.csdn.net/20180625222059635?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1b2R1bzE4dXA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

注意： 一定只能构成一棵树，不会成为多叉树或者森林：

因为数组中没有重复值，所有最大值肯定作为整棵树的头结点，而任何一个节点都会找到一个比它大的，然后放置在其下面，所以每个节点都有归属并且均以最大值为头结点 -》最终形成一棵树。

证明不是多叉树：即是证明每个结点下面最多只有两个子节点，即是证明每个结点的单侧只有一个数会挂在该结点的底下。

![image-20200103145926092](AlgorithmMediumDay01.resource/image-20200103145926092.png)



代码 ==核对一下==

```java
package nowcoder.advanced.class01;

import java.util.HashMap;
import java.util.Stack;

/**
 * @Author GJXAIOU
 * @Date 2020/1/3 15:05
 */
public class MaxTree {
    public static class Node {
        public int value;
        public Node left;
        public Node right;

        public Node(int data) {
            this.value = data;
        }
    }

    public static Node getMaxTree(int[] arr) {
        Node[] nArr = new Node[arr.length];
        for (int i = 0; i < arr.length; i++) {
            nArr[i] = new Node(arr[i]);
        }
        HashMap<Node, Node> lBigMap = new HashMap<Node, Node>();
        HashMap<Node, Node> rBigMap = new HashMap<Node, Node>();
        Stack<Node> stack = new Stack<Node>();
        for (int i = arr.length - 1; i >= 0; i--) {
            Node curNode = nArr[i];
            while ((!stack.isEmpty()) && stack.peek().value < curNode.value) {
                popStackSetMap(stack, lBigMap);
            }
            stack.push(curNode);
        }
        while (!stack.isEmpty()) {
            popStackSetMap(stack, lBigMap);
        }

        for (int i = nArr.length - 1; i != -1; i--) {
            Node curNode = nArr[i];
            while ((!stack.isEmpty()) && stack.peek().value < curNode.value) {
                popStackSetMap(stack, rBigMap);
            }
            stack.push(curNode);
        }
        while (!stack.isEmpty()) {
            popStackSetMap(stack, rBigMap);
        }

        Node head = null;
        for (int i = 0; i != nArr.length; i++) {
            Node curNode = nArr[i];
            Node left = lBigMap.get(curNode);
            Node right = rBigMap.get(curNode);
            if (left == null && right == null) {
                head = curNode;
            } else if (left == null) {
                if (right.left == null) {
                    right.left = curNode;
                } else {
                    right.right = curNode;
                }
            } else if (right == null) {
                if (left.left == null) {
                    left.left = curNode;
                } else {
                    left.right = curNode;
                }
            } else {
                Node parent = left.value < left.value ? left : right;
                if (parent.left == null) {
                    parent.left = curNode;
                } else {
                    parent.right = curNode;
                }
            }
        }
        return head;
    }

    public static void popStackSetMap(Stack<Node> stack, HashMap<Node, Node> map) {
        Node popNode = stack.pop();
        if (stack.isEmpty()) {
            map.put(popNode, null);
        } else {
            map.put(popNode, stack.peek());
        }
    }

    public static void printPreOrder(Node head) {
        if (head == null) {
            return;
        }
        System.out.print(head.value + " ");
        printPreOrder(head.left);
        printPreOrder(head.right);
    }

    public static void main(String[] args) {
        int[] uniqueArr ={3, 4, 5, 1, 2};
        Node head = getMaxTree(uniqueArr);
        printPreOrder(head);
        System.out.println();
        printPreOrder(head);
    }
}


```



代码：

![img](https://img-blog.csdn.net/20180628215634109?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1b2R1bzE4dXA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

   ![img](https://img-blog.csdn.net/20180628215700907?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1b2R1bzE4dXA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![img](https://img-blog.csdn.net/20180628215813854?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1b2R1bzE4dXA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![img](https://img-blog.csdn.net/20180628215841863?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1b2R1bzE4dXA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



## 2  求最大子矩阵的大小： 

![img](https://img-blog.csdn.net/20180625223145661?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1b2R1bzE4dXA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)





**类似的，下图的直方图** （值可以相等），以每个矩阵的中心高度为杠---然后分别向左右去扩展，到比它小的时候就停止 ---并记录能够达成的最大格子数目

![img](https://img-blog.csdn.net/20180625223536951?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R1b2R1bzE4dXA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**解法:**   此题用单调栈，但是是找到左边和右边离他最近的比它小的数---从小到大的顺序  

若压栈元素<当前栈顶元素 则弹出栈顶元素 ，并且结算值，例如放入  1 -> 3 时候会弹出 0 -》4 则表示 0 位置数的距离其最近的比它小的数在 1 位置，同时 0-》 4 下面没有元素，表示最左边可以扩充到 -1 位置，因为 -1 和 1 位置都不可达，所以其真正能扩充的只有它自己。 

如果栈下面没有元素 则一定能够到达左边界 右边界就是那个压栈元素所在值  

若到最后栈里有元素 但是没有主动让元素弹出的右端元素 则可以最右到达右边界 左边就是下面的元素（两边不可达）

![img](AlgorithmMediumDay01.resource/20180625224236941.png)

**转换到原始问题中** ：

数组为 10 11 带入上面代码求出，以第 0 行为底的所有长方形中，含有1 最大的长方形；

则 对于1 行  2 1 2 2（表示以该行开始往上看有多少个连续的 1）带入上面代码求出以 第 1 行为底的。。。。

对于行 3 2 2 0 （注意此时为0 因为最上面是0 并没构成连续的1）

目的就是找到 每一行打底的最大的1的长方形

![img](AlgorithmMediumDay01.resource/201806252302410.png)



分析时间复杂度：因为每行运算都是 O（m），然后一共 N 行，所以是 O（n*m ) 就是遍历一遍矩阵

![img](AlgorithmMediumDay01.resource/2018062523051320.png)



代码

```java
package nowcoder.advanced.class01;

import java.util.Stack;

/**
 * @Author GJXAIOU
 * @Date 2020/1/3 15:59
 */
public class MaximalRectangle {
    // 原问题（就是 0 1 矩阵）
    public static int maxRecSize(int[][] map) {
        if (map == null || map.length == 0 || map[0].length == 0) {
            return 0;
        }
        int maxArea = 0;
        int[] height = new int[map[0].length];
        for (int i = 0; i < map.length; i++) {
            for (int j = 0; j < map[0].length; j++) {
                // 形成辅助数组：就是 [10 11][2 1 2 2]等等
                height[j] = map[i][j] == 0 ? 0 : height[j] + 1;
            }
            maxArea = Math.max(maxRecFromBottom(height), maxArea);
        }
        return maxArea;
    }

    // 最基本方法，即一个数组代表直方图的话在其中找到最大矩形
    public static int maxRecFromBottom(int[] height) {
        if (height == null || height.length == 0) {
            return 0;
        }
        int maxArea = 0;
        // 准备一个单调栈，栈中放置下标
        Stack<Integer> stack = new Stack<Integer>();
        for (int i = 0; i < height.length; i++) {
            // 当栈不为空，且当前数小于等于栈顶
            while (!stack.isEmpty() && height[i] <= height[stack.peek()]) {
                // 弹出栈顶
                int j = stack.pop();
                // k 为左边界 （即弹出的数的下面是什么）
                int k = stack.isEmpty() ? -1 : stack.peek();
                // i 为当前数，就是右边界，自己在 k 位置上
                int curArea = (i - k - 1) * height[j];
                maxArea = Math.max(maxArea, curArea);
            }
            stack.push(i);
        }
        // 遍历完成之后，栈中剩余元素进行结算
        while (!stack.isEmpty()) {
            int j = stack.pop();
            int k = stack.isEmpty() ? -1 : stack.peek();
            int curArea = (height.length - k - 1) * height[j];
            maxArea = Math.max(maxArea, curArea);
        }
        return maxArea;
    }

    public static void main(String[] args) {
        int[][] map = {{1, 0, 1, 1}, {1, 1, 1, 1}, {1, 1, 1, 0}};
        int maxArea = maxRecSize(map);
        System.out.println("maxArea = " + maxArea);
    }
}
```



### 回形山

使用一个数组表示环形山，然后每座山上放烽火，相邻山之间的烽火可以看见；因为任意两座山之间可以通过顺时针或者逆时针的方式互相到达，如果两种方式中一条经过的数字都不大于两座山值的较小的一个，则两座山相互可见（如 3,4 之间可以看见），返回能返回相互看见的山峰有多少对。

![image-20200103164426898](AlgorithmMediumDay01.resource/image-20200103164426898.png)

如果数组中所有值都不相等：

1 座山峰规定为 0 对，2 座山峰规定为 1 对，如果大于 2 座，则 i 座山峰共 （ 2 * i - 3 ）对。

找发规定为：小的找大的，

![image-20200103171117637](AlgorithmMediumDay01.resource/image-20200103171117637.png)



但是实际问题中是可能含有最大值的，因此首先找到数组中的最大值，如果有多个取第一个，从该值开始进行环形遍历（例如：3  3 5  4  5，遍历顺序为：5 4  5 3  3）

![image-20200103173145387](AlgorithmMediumDay01.resource/image-20200103173145387.png)

针对遍历结束之后单调栈中剩余的元素，倒数第 3 及其以上的仍然适用于原来的公式，其余倒数第 2 和倒数第 1 单独计算。

![image-20200103174347520](AlgorithmMediumDay01.resource/image-20200103174347520.png)

```java
package nowcoder.advanced.class01;

import java.awt.*;
import java.util.Scanner;
import java.util.Stack;

/**
 * @Author GJXAIOU
 * @Date 2020/1/3 17:44
 */
public class MountainsAndFlame {
    public static void main(String[] args) {
        // 输入两部分值：数组长度和数组具体的内容
        Scanner in = new Scanner(System.in);
        while (in.hasNextInt()) {
            int size = in.nextInt();
            int[] arr = new int[size];
            for (int i = 0; i < size; i++) {
                arr[i] = in.nextInt();
            }
            System.out.println(communications(arr));
        }
        in.close();
    }

    // 在环形数组中 i 位置的下一位，没到底则 +1，到底就为 0
    public static int nextIndex(int size, int i) {
        return i < (size - 1) ? (i + 1) : 0;
    }

    public static long getInternalSum(int n) {
        return n == 1L ? 0L : (long) n * (long) (n - 1) / 2L;
    }

    public static class Pair {
        public int value;
        public int times;

        public Pair(int value) {
            this.value = value;
            this.times = 1;
        }
    }

    // arr 为环形数组
    public static long communications(int[] arr) {
        if (arr == null || arr.length < 2) {
            return 0;
        }
        int size = arr.length;
        // 找到最大值位置（第一个）
        int maxIndex = 0;
        for (int i = 0; i < size; i++) {
            maxIndex = arr[maxIndex] < arr[i] ? i : maxIndex;
        }
        // value 为最大值
        int value = arr[maxIndex];
        int index = nextIndex(size, maxIndex);
        long res = 0L;
        // 首先将最大值扔入栈中
        Stack<Pair> stack = new Stack<Pair>();
        stack.push(new Pair(value));
        // 因为是从 maxIndex 的下一个位置开始遍历的，所有如果值再次相等说明遍历结束
        while (index != maxIndex) {
            // 数组中的当前值
            value = arr[index];
            while (!stack.isEmpty() && stack.peek().value < value) {
                int times = stack.pop().times;
                // res += getInernalSum(times) +times;
                // res += stack.isEmpty() ? 0 : times;
                // 下面结果为上面两句合并
                res += getInternalSum(times) + 2 * times;
            }
            // 如果当前值使得栈顶弹出，然后栈顶和弹出之后露出的栈顶值相同，则原来栈顶数目 +1
            if (!stack.isEmpty() && stack.peek().value == value) {
                stack.peek().times++;
                // 如果和弹出之后露出的栈顶值不相等，则将当前值放入栈中
            } else {
                stack.push(new Pair(value));
            }
            index = nextIndex(size, index);
        }
        // 遍历结束之后剩余栈中元素进行结算
        while (!stack.isEmpty()) {
            int times = stack.pop().times;
            res += getInternalSum(times);
            if (!stack.isEmpty()) {
                res += times;
                if (stack.size() > 1) {
                    res += times;
                } else {
                    res += stack.peek().times > 1 ? times : 0;
                }
            }
        }
        return res;
    }
}

```





