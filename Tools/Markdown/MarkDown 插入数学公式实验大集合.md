# MarkDown 插入数学公式实验大集合



## 1\. 基础认识

笔者认为所谓插入数学公式其实就是引入一种规则，然后通过`模板？`渲染成公式，不知道这个理解对不对，不对望指正。其实你以前可能就看到过有的博客本该出现公式的时候不显示，点击后会链接到一个 new tab 然后显示一张公式的图片，有时却出现一大堆的代码。这里就是通过这段代码解析成公式然后显示的。

这里我们选取 MathJax 引擎。 引入脚本，把下面代码插入 MD 文件里面，如果你怕这份在线文件源别人访问不到的话，可以把这个下下来自己做一个源，这样比较稳定缺点是要自己手动更新源。

```
<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>
复制代码
```

好了到这里就可以插入公式了，如果你懂 LaTeX 的话那看一两个例子就知道了，不懂也没关系，自己写一写代码就知道了，可以找一个可以预览 MD 的工具一直尝试。

### 1.1 插入方式

> 这里分两种，一种是行间插入，另一种是另取一行

#### 1.1.1 行间插入

```
\\(a + b\\)
复制代码
```

这里是行间插入公式 a + b : \(a + b\)，特点就是通过`(` 和 `)` 包含公式，然后为了模板引擎能够区分该 `(` 不是普通文本的 `(` 而是公式的 `(`，通过 `\\` 转义一下。这样应该就很好理解这个语法构成了。注意这里方式不唯一，这是笔者喜欢的方式，其他的使用方式自行搜索。下面的介绍同样是这样。

PS: 这里掘金使用的是 `$a + b$` : ![a + b](https://juejin.im/equation?tex=a%20%2B%20b)，如果对您的阅读产生印象，请看最后说明，这里就不做一一更改了。谢谢。

#### 1.1.2 另取一行

```
$$a + b$$
复制代码
```

这里是另取一行

![a + b](https://juejin.im/equation?tex=a%20%2B%20b)

特点就是通过`$$`包含公式。

笔者认为第二种方式更好，以下没看 JS 源码纯属猜测：行间的需要考虑到当前行的行高并对公式进行处理，而另取一行就更简单一些，可能解析起来更快。最最最最最最主要是看起来漂亮 ^_^ 不太要考虑空间不够换行。

### 1.2 基本类型的插入

> 这里对 [@houkai ：LATEX数学公式基本语法](http://www.cnblogs.com/houkai/p/3399646.html) 的思路稍加修改，然后进行介绍。

#### 1.2.1 上、下标

先看结果再总结语法吧。

```
$$x_1$$

$$x_1^2$$

$$x^2_1$$

$$x_{22}^{(n)}$$

$${}^*x^*$$

$$x_{balabala}^{bala}$$
复制代码
```

![x_1](https://juejin.im/equation?tex=x_1)

![x_1^2](https://juejin.im/equation?tex=x_1%5E2)

![x^2_1](https://juejin.im/equation?tex=x%5E2_1)

![x_{22}^{(n)}](https://juejin.im/equation?tex=x_%7B22%7D%5E%7B(n)%7D)

![{}^*x^*](https://juejin.im/equation?tex=%7B%7D%5E*x%5E*)

![x_{balabala}^{bala}](https://juejin.im/equation?tex=x_%7Bbalabala%7D%5E%7Bbala%7D)

可以看到 `x` 元素的上标通过 `^` 符号后接的内容体现，下表通过 `_` 符号后接的内容体现，多于一位是要加 `{}` 包裹的。 笔者习惯先下标后上标的写法，和我的书写习惯一致：`x_{balabala}^{bala}`，不管你使用哪一种风格，最好自己注意统一，不要混用。

#### 1.2.2 分式

```
$$\frac{x+y}{2}$$

$$\frac{1}{1+\frac{1}{2}}$$
复制代码
```

![\frac{x+y}{2}](https://juejin.im/equation?tex=%5Cfrac%7Bx%2By%7D%7B2%7D)

![\frac{1}{1+\frac{1}{2}}](https://juejin.im/equation?tex=%5Cfrac%7B1%7D%7B1%2B%5Cfrac%7B1%7D%7B2%7D%7D)

这里就出现了一个 `frac{}{}` 函数的东西，同样，为了区分这是函数不是几个字母，通过 `\frac` 转义，于是 `frac` 被解析成函数，然后第一个 `{}` 里面的被解析成分子，第二个 `{}` 被解析成分母。这里可以试试分数的行间解析![\frac{1}{1+\frac{1}{2}}](https://juejin.im/equation?tex=%5Cfrac%7B1%7D%7B1%2B%5Cfrac%7B1%7D%7B2%7D%7D)。我要看行间填充效果我要看行间填充效果我要看行间填充效果我要看行间填充效果我要看行间填充效果我要看行间填充效果我要看行间填充效果我要看行间填充效果我要看行间填充效果我要看行间填充效果我要看行间填充效果我要看行间填充效果。

#### 1.2.3 根式

```
$$\sqrt{2}<\sqrt[3]{3}$$

$$\sqrt{1+\sqrt[p]{1+a^2}}$$

$$\sqrt{1+\sqrt[^p\!]{1+a^2}}$$
复制代码
```

![\sqrt{2}<\sqrt[3]{3}](https://juejin.im/equation?tex=%5Csqrt%7B2%7D%3C%5Csqrt%5B3%5D%7B3%7D)

![\sqrt{1+\sqrt[p]{1+a^2}}](https://juejin.im/equation?tex=%5Csqrt%7B1%2B%5Csqrt%5Bp%5D%7B1%2Ba%5E2%7D%7D)

![\sqrt{1+\sqrt[^p]{1+a^2}}](https://juejin.im/equation?tex=%5Csqrt%7B1%2B%5Csqrt%5B%5Ep%5D%7B1%2Ba%5E2%7D%7D)

读到这里你已经了解了函数的概念，那么这历久很简单了，语法就是 `sqrt[]{}` 。`[]` 中代表是几次根式，`{}` 代表根号下的表达式。第二和第三个的区别在于为了美观微调位置 ^_^。

#### 1.2.4 求和、积分

```
$$\sum_{k=1}^{n}\frac{1}{k}$$

$\sum_{k=1}^n\frac{1}{k}$

$$\int_a^b f(x)dx$$

$\int_a^b f(x)dx$
复制代码
```

![\sum_{k=1}^{n}\frac{1}{k}](https://juejin.im/equation?tex=%5Csum_%7Bk%3D1%7D%5E%7Bn%7D%5Cfrac%7B1%7D%7Bk%7D)

![\sum_{k=1}^n\frac{1}{k}](https://juejin.im/equation?tex=%5Csum_%7Bk%3D1%7D%5En%5Cfrac%7B1%7D%7Bk%7D)

![\int_{a}^b f(x)dx](https://juejin.im/equation?tex=%5Cint_%7Ba%7D%5Eb%20f(x)dx)

![\int_a^b f(x)dx](https://juejin.im/equation?tex=%5Cint_a%5Eb%20f(x)dx)

这里很容易看出求和函数表达式 `sum_{起点}^{终点}表达式`，积分函数表达式 `int_下限^上限 被积函数d被积量`。还有一个有趣的是行间的公式都被压缩了。

#### 1.2.5 空格

```
紧贴 $a\!b$
没有空格 $ab$
小空格 a\,b
中等空格 a\;b
大空格 a\ b
quad空格 $a\quad b$
两个quad空格 $a\qquad b$
复制代码
```

![a\!b](https://juejin.im/equation?tex=a%5C!b)

![ab](https://juejin.im/equation?tex=ab)

![a\,b](https://juejin.im/equation?tex=a%5C%2Cb)

![a\;b](https://juejin.im/equation?tex=a%5C%3Bb)

![a\ b](https://juejin.im/equation?tex=a%5C%20b)

![a\quad b](https://juejin.im/equation?tex=a%5Cquad%20b)

![a\qquad b](https://juejin.im/equation?tex=a%5Cqquad%20b)

这个直接看上面的文字，介绍很清楚，主要指微调距离，使得公式更加漂亮。请比较下面的积分公式：

```
$$\int_a^b f(x)\mathrm{d}x$$

$$\int_a^b f(x)\,\mathrm{d}x$$
复制代码
```

![\int_a^b f(x)\mathrm{d}x](https://juejin.im/equation?tex=%5Cint_a%5Eb%20f(x)%5Cmathrm%7Bd%7Dx)

![\int_a^b f(x)\,\mathrm{d}x](https://juejin.im/equation?tex=%5Cint_a%5Eb%20f(x)%5C%2C%5Cmathrm%7Bd%7Dx)

#### 1.2.6 公式界定符

```
\\( ( \\)
\\( ) \\)
\\( [ \\)
\\( ] \\)
\\( \\{ \\)
\\( \\} \\)
\\( | \\)
\\( \\| \\)

掘金：
$ ( $
$ ) $
$ [ $
$ ] $
$ \{ $
$ \} $
$ | $
$ \| $
复制代码
```

主要符号有 ![(](https://juejin.im/equation?tex=() ![)](https://juejin.im/equation?tex=)) ![[](https://juejin.im/equation?tex=%5B) ![]](https://juejin.im/equation?tex=%5D) ![\{](https://juejin.im/equation?tex=%5C%7B) ![\}](https://juejin.im/equation?tex=%5C%7D) ![|](https://juejin.im/equation?tex=%7C) ![\|](https://juejin.im/equation?tex=%5C%7C) 那么如何使用呢？ 通过 `\left` 和 `\right` 后面跟界定符来对同时进行界定。

```
$$\left(\sum_{k=\frac{1}{2}}^{N^2}\frac{1}{k}\right)$$
复制代码
```

![\left(\sum_{k=\frac{1}{2}}^{N^2}\frac{1}{k}\right)](https://juejin.im/equation?tex=%5Cleft(%5Csum_%7Bk%3D%5Cfrac%7B1%7D%7B2%7D%7D%5E%7BN%5E2%7D%5Cfrac%7B1%7D%7Bk%7D%5Cright))

#### 1.2.7 矩阵

```
$$\begin{matrix}1 & 2\\\\3 &4\end{matrix}$$

$$\begin{pmatrix}1 & 2\\\\3 &4\end{pmatrix}$$

$$\begin{bmatrix}1 & 2\\\\3 &4\end{bmatrix}$$

$$\begin{Bmatrix}1 & 2\\\\3 &4\end{Bmatrix}$$

$$\begin{vmatrix}1 & 2\\\\3 &4\end{vmatrix}$$

$$\left|\begin{matrix}1 & 2\\\\3 &4\end{matrix}\right|$$

$$\begin{Vmatrix}1 & 2\\\\3 &4\end{Vmatrix}$$
复制代码
```

![\begin{matrix}1 & 2\\\\3 &4\end{matrix}](https://juejin.im/equation?tex=%5Cbegin%7Bmatrix%7D1%20%26%202%5C%5C%5C%5C3%20%264%5Cend%7Bmatrix%7D)

![\begin{pmatrix}1 & 2\\\\3 &4\end{pmatrix}](https://juejin.im/equation?tex=%5Cbegin%7Bpmatrix%7D1%20%26%202%5C%5C%5C%5C3%20%264%5Cend%7Bpmatrix%7D)

![\begin{bmatrix}1 & 2\\\\3 &4\end{bmatrix}](https://juejin.im/equation?tex=%5Cbegin%7Bbmatrix%7D1%20%26%202%5C%5C%5C%5C3%20%264%5Cend%7Bbmatrix%7D)

![\begin{Bmatrix}1 & 2\\\\3 &4\end{Bmatrix}](https://juejin.im/equation?tex=%5Cbegin%7BBmatrix%7D1%20%26%202%5C%5C%5C%5C3%20%264%5Cend%7BBmatrix%7D)

![\begin{vmatrix}1 & 2\\\\3 &4\end{vmatrix}](https://juejin.im/equation?tex=%5Cbegin%7Bvmatrix%7D1%20%26%202%5C%5C%5C%5C3%20%264%5Cend%7Bvmatrix%7D)

![\left|\begin{matrix}1 & 2\\\\3 &4\end{matrix}\right|](https://juejin.im/equation?tex=%5Cleft%7C%5Cbegin%7Bmatrix%7D1%20%26%202%5C%5C%5C%5C3%20%264%5Cend%7Bmatrix%7D%5Cright%7C)

![\begin{Vmatrix}1 & 2\\\\3 &4\end{Vmatrix}](https://juejin.im/equation?tex=%5Cbegin%7BVmatrix%7D1%20%26%202%5C%5C%5C%5C3%20%264%5Cend%7BVmatrix%7D)

类似于 left right，这里是 begin 和 end。而且里面有具体的矩阵语法，`&` 区分行间元素，`\\\\` 代表换行。可以理解为 HTML 的标签之类的。

#### 1.2.8 排版数组

```
\mathbf{X} =
\left( \begin{array}{ccc}
x\_{11} & x\_{12} & \ldots \\\\
x\_{21} & x\_{22} & \ldots \\\\
\vdots & \vdots & \ddots
\end{array} \right)
复制代码
```

![\mathbf{X} =
\left( \begin{array}{ccc}
x\_{11} & x\_{12} & \ldots \\\\
x\_{21} & x\_{22} & \ldots \\\\
\vdots & \vdots & \ddots
\end{array} \right)](https://juejin.im/equation?tex=%5Cmathbf%7BX%7D%20%3D%0A%5Cleft(%20%5Cbegin%7Barray%7D%7Bccc%7D%0Ax%5C_%7B11%7D%20%26%20x%5C_%7B12%7D%20%26%20%5Cldots%20%5C%5C%5C%5C%0Ax%5C_%7B21%7D%20%26%20x%5C_%7B22%7D%20%26%20%5Cldots%20%5C%5C%5C%5C%0A%5Cvdots%20%26%20%5Cvdots%20%26%20%5Cddots%0A%5Cend%7Barray%7D%20%5Cright))

## 2\. 常用公式举例

> 持续更新……

### 2.1 多行公式

> 主要是各种方程的表达

#### 2.1.1 长公式

```
$$
\begin{multline}
x = a+b+c+{} \\\\
d+e+f+g
\end{multline}
$$

$$
\begin{aligned}
x ={}& a+b+c+{} \\\\
&d+e+f+g
\end{aligned}
$$
复制代码
```

不对齐

![\left| \begin{multline}
x = a+b+c+{} \\\\
d+e+f+g
\end{multline} \right|](https://juejin.im/equation?tex=%5Cleft%7C%20%5Cbegin%7Bmultline%7D%0Ax%20%3D%20a%2Bb%2Bc%2B%7B%7D%20%5C%5C%5C%5C%0Ad%2Be%2Bf%2Bg%0A%5Cend%7Bmultline%7D%20%5Cright%7C)

对齐

![\left| \begin{aligned}
x ={}& a+b+c+{} \\\\
&d+e+f+g
\end{aligned} \right|](https://juejin.im/equation?tex=%5Cleft%7C%20%5Cbegin%7Baligned%7D%0Ax%20%3D%7B%7D%26%20a%2Bb%2Bc%2B%7B%7D%20%5C%5C%5C%5C%0A%26d%2Be%2Bf%2Bg%0A%5Cend%7Baligned%7D%20%5Cright%7C)

#### 2.1.2 公式组

```
$$
\begin{gather}
a = b+c+d \\\\
x = y+z
\end{gather}
$$

$$
\begin{align}
a &= b+c+d \\\\
x &= y+z
\end{align}
$$
复制代码
```

![\begin{gather}
a = b+c+d \\\\
x = y+z
\end{gather}](https://juejin.im/equation?tex=%5Cbegin%7Bgather%7D%0Aa%20%3D%20b%2Bc%2Bd%20%5C%5C%5C%5C%0Ax%20%3D%20y%2Bz%0A%5Cend%7Bgather%7D)

![\begin{align}
a &= b+c+d \\\\
x &= y+z
\end{align}](https://juejin.im/equation?tex=%5Cbegin%7Balign%7D%0Aa%20%26%3D%20b%2Bc%2Bd%20%5C%5C%5C%5C%0Ax%20%26%3D%20y%2Bz%0A%5Cend%7Balign%7D)

#### 2.1.3 分段函数

```
$$
y=\begin{cases}
-x,\quad x\leq 0 \\\\
x,\quad x>0
\end{cases}
$$
复制代码
```

![y=\begin{cases}
-x,\quad x\leq 0 \\\\
x,\quad x>0
\end{cases}](https://juejin.im/equation?tex=y%3D%5Cbegin%7Bcases%7D%0A-x%2C%5Cquad%20x%5Cleq%200%20%5C%5C%5C%5C%0Ax%2C%5Cquad%20x%3E0%0A%5Cend%7Bcases%7D)

里面用到了 \(\leq\) 符号，下一章会介绍常用数学符号。

### 2.2 数组的其他使用

#### 2.2.1 划线

```
$$
\left(\begin{array}{|c|c|}
1 & 2 \\\\
\\hline
3 & 4
\end{array}\right)
$$
复制代码
```

![\left( \begin{array}{|c|c|}
1 & \ldots \\\\
\hline
\vdots & \ddots 
\end{array} \right)](https://juejin.im/equation?tex=%5Cleft(%20%5Cbegin%7Barray%7D%7B%7Cc%7Cc%7C%7D%0A1%20%26%20%5Cldots%20%5C%5C%5C%5C%0A%5Chline%0A%5Cvdots%20%26%20%5Cddots%20%0A%5Cend%7Barray%7D%20%5Cright))

#### 2.2.2 制表

```
$$
\begin{array}{|c|c|}
\hline
{1111111111} & 2 \\\\
\hline
3 & 4 \\\\
\hline
\end{array}
$$
复制代码
```

![\begin{array}{|c|c|}
\hline
{1111111111} & 2 \\\\
\hline
{balabala} & 你好啊 \\\\
\hline
\end{array}](https://juejin.im/equation?tex=%5Cbegin%7Barray%7D%7B%7Cc%7Cc%7C%7D%0A%5Chline%0A%7B1111111111%7D%20%26%202%20%5C%5C%5C%5C%0A%5Chline%0A%7Bbalabala%7D%20%26%20%E4%BD%A0%E5%A5%BD%E5%95%8A%20%5C%5C%5C%5C%0A%5Chline%0A%5Cend%7Barray%7D)

可以看到，其实其他很多东西都可以很灵活的表达出来。碰到其他有趣的我会继续写出来的。

## 3\. 常用数学符号

> 这里提供一个[文档下载](http://files.cnblogs.com/houkai/LATEX%E6%95%B0%E5%AD%A6%E7%AC%A6%E5%8F%B7%E8%A1%A8.rar)，如果上面的链接失效，也可以到我的 [GitHub 下载 pdf 版](https://github.com/mk43/BlogResource/blob/master/LaTex/LATEX%E6%95%B0%E5%AD%A6%E7%AC%A6%E5%8F%B7%E8%A1%A8.pdf)。下面举几个例子。

### 3.1 希腊字母

```
$$
\begin{array}{|c|c|c|c|c|c|c|c|}
\hline
{\alpha} & {\backslash alpha} & {\theta} & {\backslash theta} & {o} & {o} & {\upsilon} & {\backslash upsilon} \\\\
\hline
{\beta} & {\backslash beta} & {\vartheta} & {\backslash vartheta} & {\pi} & {\backslash pi} & {\phi} & {\backslash phi} \\\\
\hline
{\gamma} & {\backslash gamma} & {\iota} & {\backslash iota} & {\varpi} & {\backslash varpi} & {\varphi} & {\backslash varphi} \\\\
\hline
{\delta} & {\backslash delta} & {\kappa} & {\backslash kappa} & {\rho} & {\backslash rho} & {\chi} & {\backslash chi} \\\\
\hline
{\epsilon} & {\backslash epsilon} & {\lambda} & {\backslash lambda} & {\varrho} & {\backslash varrho} & {\psi} & {\backslash psi} \\\\
\hline
{\varepsilon} & {\backslash varepsilon} & {\mu} & {\backslash mu} & {\sigma} & {\backslash sigma} & {\omega} & {\backslash omega} \\\\
\hline
{\zeta} & {\backslash zeta} & {\nu} & {\backslash nu} & {\varsigma} & {\backslash varsigma} & {} & {} \\\\
\hline
{\eta} & {\backslash eta} & {\xi} & {\backslash xi} & {\tau} & {\backslash tau} & {} & {} \\\\
\hline
{\Gamma} & {\backslash Gamma} & {\Lambda} & {\backslash Lambda} & {\Sigma} & {\backslash Sigma} & {\Psi} & {\backslash Psi} \\\\
\hline
{\Delta} & {\backslash Delta} & {\Xi} & {\backslash Xi} & {\Upsilon} & {\backslash Upsilon} & {\Omega} & {\backslash Omega} \\\\
\hline
{\Omega} & {\backslash Omega} & {\Pi} & {\backslash Pi} & {\Phi} & {\backslash Phi} & {} & {} \\\\
\hline
\end{array}
$$
复制代码
```

![\begin{array}{|c|c|c|c|c|c|c|c|}
\hline
{\alpha} & {\backslash alpha} & {\theta} & {\backslash theta} & {o} & {o} & {\upsilon} & {\backslash upsilon} \\\\
\hline
{\beta} & {\backslash beta} & {\vartheta} & {\backslash vartheta} & {\pi} & {\backslash pi} & {\phi} & {\backslash phi} \\\\
\hline
{\gamma} & {\backslash gamma} & {\iota} & {\backslash iota} & {\varpi} & {\backslash varpi} & {\varphi} & {\backslash varphi} \\\\
\hline
{\delta} & {\backslash delta} & {\kappa} & {\backslash kappa} & {\rho} & {\backslash rho} & {\chi} & {\backslash chi} \\\\
\hline
{\epsilon} & {\backslash epsilon} & {\lambda} & {\backslash lambda} & {\varrho} & {\backslash varrho} & {\psi} & {\backslash psi} \\\\
\hline
{\varepsilon} & {\backslash varepsilon} & {\mu} & {\backslash mu} & {\sigma} & {\backslash sigma} & {\omega} & {\backslash omega} \\\\
\hline
{\zeta} & {\backslash zeta} & {\nu} & {\backslash nu} & {\varsigma} & {\backslash varsigma} & {} & {} \\\\
\hline
{\eta} & {\backslash eta} & {\xi} & {\backslash xi} & {\tau} & {\backslash tau} & {} & {} \\\\
\hline
{\Gamma} & {\backslash Gamma} & {\Lambda} & {\backslash Lambda} & {\Sigma} & {\backslash Sigma} & {\Psi} & {\backslash Psi} \\\\
\hline
{\Delta} & {\backslash Delta} & {\Xi} & {\backslash Xi} & {\Upsilon} & {\backslash Upsilon} & {\Omega} & {\backslash Omega} \\\\
\hline
{\Omega} & {\backslash Omega} & {\Pi} & {\backslash Pi} & {\Phi} & {\backslash Phi} & {} & {} \\\\
\hline
\end{array}](https://juejin.im/equation?tex=%5Cbegin%7Barray%7D%7B%7Cc%7Cc%7Cc%7Cc%7Cc%7Cc%7Cc%7Cc%7C%7D%0A%5Chline%0A%7B%5Calpha%7D%20%26%20%7B%5Cbackslash%20alpha%7D%20%26%20%7B%5Ctheta%7D%20%26%20%7B%5Cbackslash%20theta%7D%20%26%20%7Bo%7D%20%26%20%7Bo%7D%20%26%20%7B%5Cupsilon%7D%20%26%20%7B%5Cbackslash%20upsilon%7D%20%5C%5C%5C%5C%0A%5Chline%0A%7B%5Cbeta%7D%20%26%20%7B%5Cbackslash%20beta%7D%20%26%20%7B%5Cvartheta%7D%20%26%20%7B%5Cbackslash%20vartheta%7D%20%26%20%7B%5Cpi%7D%20%26%20%7B%5Cbackslash%20pi%7D%20%26%20%7B%5Cphi%7D%20%26%20%7B%5Cbackslash%20phi%7D%20%5C%5C%5C%5C%0A%5Chline%0A%7B%5Cgamma%7D%20%26%20%7B%5Cbackslash%20gamma%7D%20%26%20%7B%5Ciota%7D%20%26%20%7B%5Cbackslash%20iota%7D%20%26%20%7B%5Cvarpi%7D%20%26%20%7B%5Cbackslash%20varpi%7D%20%26%20%7B%5Cvarphi%7D%20%26%20%7B%5Cbackslash%20varphi%7D%20%5C%5C%5C%5C%0A%5Chline%0A%7B%5Cdelta%7D%20%26%20%7B%5Cbackslash%20delta%7D%20%26%20%7B%5Ckappa%7D%20%26%20%7B%5Cbackslash%20kappa%7D%20%26%20%7B%5Crho%7D%20%26%20%7B%5Cbackslash%20rho%7D%20%26%20%7B%5Cchi%7D%20%26%20%7B%5Cbackslash%20chi%7D%20%5C%5C%5C%5C%0A%5Chline%0A%7B%5Cepsilon%7D%20%26%20%7B%5Cbackslash%20epsilon%7D%20%26%20%7B%5Clambda%7D%20%26%20%7B%5Cbackslash%20lambda%7D%20%26%20%7B%5Cvarrho%7D%20%26%20%7B%5Cbackslash%20varrho%7D%20%26%20%7B%5Cpsi%7D%20%26%20%7B%5Cbackslash%20psi%7D%20%5C%5C%5C%5C%0A%5Chline%0A%7B%5Cvarepsilon%7D%20%26%20%7B%5Cbackslash%20varepsilon%7D%20%26%20%7B%5Cmu%7D%20%26%20%7B%5Cbackslash%20mu%7D%20%26%20%7B%5Csigma%7D%20%26%20%7B%5Cbackslash%20sigma%7D%20%26%20%7B%5Comega%7D%20%26%20%7B%5Cbackslash%20omega%7D%20%5C%5C%5C%5C%0A%5Chline%0A%7B%5Czeta%7D%20%26%20%7B%5Cbackslash%20zeta%7D%20%26%20%7B%5Cnu%7D%20%26%20%7B%5Cbackslash%20nu%7D%20%26%20%7B%5Cvarsigma%7D%20%26%20%7B%5Cbackslash%20varsigma%7D%20%26%20%7B%7D%20%26%20%7B%7D%20%5C%5C%5C%5C%0A%5Chline%0A%7B%5Ceta%7D%20%26%20%7B%5Cbackslash%20eta%7D%20%26%20%7B%5Cxi%7D%20%26%20%7B%5Cbackslash%20xi%7D%20%26%20%7B%5Ctau%7D%20%26%20%7B%5Cbackslash%20tau%7D%20%26%20%7B%7D%20%26%20%7B%7D%20%5C%5C%5C%5C%0A%5Chline%0A%7B%5CGamma%7D%20%26%20%7B%5Cbackslash%20Gamma%7D%20%26%20%7B%5CLambda%7D%20%26%20%7B%5Cbackslash%20Lambda%7D%20%26%20%7B%5CSigma%7D%20%26%20%7B%5Cbackslash%20Sigma%7D%20%26%20%7B%5CPsi%7D%20%26%20%7B%5Cbackslash%20Psi%7D%20%5C%5C%5C%5C%0A%5Chline%0A%7B%5CDelta%7D%20%26%20%7B%5Cbackslash%20Delta%7D%20%26%20%7B%5CXi%7D%20%26%20%7B%5Cbackslash%20Xi%7D%20%26%20%7B%5CUpsilon%7D%20%26%20%7B%5Cbackslash%20Upsilon%7D%20%26%20%7B%5COmega%7D%20%26%20%7B%5Cbackslash%20Omega%7D%20%5C%5C%5C%5C%0A%5Chline%0A%7B%5COmega%7D%20%26%20%7B%5Cbackslash%20Omega%7D%20%26%20%7B%5CPi%7D%20%26%20%7B%5Cbackslash%20Pi%7D%20%26%20%7B%5CPhi%7D%20%26%20%7B%5Cbackslash%20Phi%7D%20%26%20%7B%7D%20%26%20%7B%7D%20%5C%5C%5C%5C%0A%5Chline%0A%5Cend%7Barray%7D)

写太累了😂😂😂。。。其他的详见 [PDF](https://github.com/mk43/BlogResource/blob/master/LaTex/LATEX%E6%95%B0%E5%AD%A6%E7%AC%A6%E5%8F%B7%E8%A1%A8.pdf)。

## 4\. 总结

> 通过这样梳理一下基本的公式都能插入了，而且也会如何查资料。对于自己日后学习 LaTeX 写论文有很大帮助。以下建议带有很强的主观性，仅供参考。

*   公式一律使用另取一行，并且上下都空一行
*   一个公式一个语句，不要写在一个 `$$***$$` 里，保证`独立性`，一个公式错误不影响另一个公式。
*   风格统一，不要混用。比如上下标的写法：`x_{balabala}^{bala}`
*   行间字母可以使用 `\\(a\\)` 代替 `a` ，养成自己的写作风格。

最后：我的 [Blog](http://fitzeng.org/) 和 [GitHub](https://github.com/mk43)，感谢阅读。

## 5\. 参考资料

> 十分感谢以下作者的无私分享。

1.  [Markdown中插入数学公式的方法](http://blog.csdn.net/xiahouzuoxin/article/details/26478179)
2.  [LATEX数学公式基本语法](http://www.cnblogs.com/houkai/p/3399646.html)
3.  [一份其实很短的 LaTeX 入门文档](https://liam0205.me/2014/09/08/latex-introduction/)

说明：掘金好像有些转义不出来，如果想看效果可以[参考这篇](http://fitzeng.org/2018/01/23/LaTexFormula/)自己多试试。