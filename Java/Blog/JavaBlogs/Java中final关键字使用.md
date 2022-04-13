---
flag: red
tags: 
- final关键字
- 未看
---

# Java中final关键字使用
在Java中，final关键字可以用来修饰类、方法和变量（包括成员变量和局部变量）。final方法在编译阶段绑定，称为静态绑定(static binding)。

## 一、修饰类 

当一个类的整体定义为final时候，表明这个类不能被继承，比如java中的String类。但是注意的是final类中的所有成员方法都会被隐式地指定为final方法。这个很容易。

## 二、方法

当一个方法修饰为final的时候，意味着把该方法锁定，以防任何继承类修改它的含义。因此，如果只有明确禁止该方法在子类中被覆盖的情况下才将方法设置为final的。还有就是，类的private方法会隐式地被指定为final方法。代码如下：

```java
class WithFinals {
    private final void f() {
        System.out.println("WithFinals.f()");
    }

    private void g() {
        System.out.println("WithFinals.g()");
    }

    public final void h() {
        System.out.println("WithFinals.h()");
    }
}


public class OverridingPrivate extends WithFinals {
//    public void h() {
//        System.out.println("OverridingPrivate.h()");
//    }

//    @Override
    public void g() {
        System.out.println("OverridingPrivate.g()");
    }

//    @Override
    public void f() {
        System.out.println("OverridingPrivate.f()");
    }

}
```

如果把上面三处注释地方去掉，代码编译不通过，下面分析一下编译不通过的原因。第一个地方方法h()编译不通过，因为父类h() 方法修饰符为final不能通过编译。

第二个地方是g()上面的注解编译不通过，是因为父类发g()方法是修饰为private，这正如上面讲的类的private方法会隐式地被指定为final方法，但是跟final又不同，去掉@Override却编译通过，**这是因为“覆盖”只有在某个方法是基类的一部分才会出现。即，必须能将一个对象向上转型为它的基本类型并调用相同的方法**。如果方法为private，它就不是基类的一部分，它是隐藏于类中的程序代码，只不过有相同的名称。如果方法为public、protected或包访问权限方法的话，就不会产生在基类中出现的"仅具有相同的名称"。所以子类的g（）是一个新的方法，这其实与final关系不大；

第三个地方是f()上面的注解不能编译通过,去掉@Override也是正常编译通过的，所以这也能间接证明private 方法其实是否是final没有多大联系。

## 三、变量

对于一个final变量，如果是基本数据类型的变量，则其数值一旦在初始化之后便不能更改；如果是引用类型的变量，则在对其初始化之后便不能再让其指向另一个对象。引用变量被final修饰之后，虽然不能再指向其他对象，但是它指向的对象的内容是可变的。撸个代码：

```
class Value {
    int i;
    public Value(int i) {
        this.i = i;
    }
}


public class FinalData {
    private static Random rand = new Random(47);
    private String id;


    public FinalData(String id) {
        this.id = id;
    }

    private final int valueOne = 9;
    private static final int VALUE_TWO = 99;

    private final int i4 = rand.nextInt(20);
    static final int INT_5 = rand.nextInt(20);

    private Value v1 = new Value(11);
    private final Value v2 = new Value(22);
    private static final Value VAL_3 = new Value(33);


    @Override
    public String toString() {
        return id + ": " + "i4 = " + i4 + ", INT_5 = " + INT_5;
    }

    public static void main(String[] args) {
        FinalData fd1 = new FinalData("fd1");
        // fd1.valueOne++; // Error
        fd1.v2.i++;
        fd1.v1 = new Value(9);

        // fd1.v2 = new Value(0); // Error
        // fd1.VAL_3 = new Value(1); // Error
        System.out.println(fd1);
        System.out.println("Creating new FinalData");
        FinalData fd2 = new FinalData("fd2");
        System.out.println(fd1);
        System.out.println(fd2);
    }
}
```

一个个分析， 我们知道static强调唯一，而final强调常量，valueOne和VALUE_TWO 都是编译常量，区别就是VALUE_TWO在存储空间是唯一的，以及赋值阶段不同。

对于i4 和INT_5展示了将final数值定义为静态和非静态的区别。看一下上面代码编译后的情况，如下面的图：

![代码编译结果]($resource/%E4%BB%A3%E7%A0%81%E7%BC%96%E8%AF%91%E7%BB%93%E6%9E%9C.jpg)

图一

编译之后i4到构造器中，而INT_5在static代码块中，可以看出i4在创建实列变量的时候被赋值，而INT_5在类加载过程准备阶段就赋值了(注意static且非final是在初始化阶段被赋值的)，还有一点valueOne并不在构造器中，这是因为valueOne是基本类型。另外在fd1和fd2中发现i4的值是唯一的（相对于对象而言），INT_5也是一样的（相对于类而言）。

v1到VAl_3中，不能因为v2是final的，就 认为v2的值不可以改变。从代码的第37行可以看出，v2的值是可以改变的，这是因为这是一个引用，但是无法将v2再次指向新的引用。这对数组具有同样的意义，数组不过是另一种引用。

## 四、进阶

### 1.空白final

空白final是指声明为final但又未给初值的变量。编译器确保空白的final在使用前必须初始化。代码如下

```
class Poppet {
    private int i;

    Poppet(int ii) {
        i = ii;
    }
}

public class BlankFinal {
    private final int i = 0;
    private final int j;
    private final Poppet p;

    public BlankFinal() {
        j = 1;
        p = new Poppet(1);
    }

}

```

i和p均没有值 ，这跟static不一样，**static非final会在类加载阶段的准备阶段会被赋予类型初始值，即 static修饰的 j在准备阶段为0，static修饰的 p为null，而空白final不会**。如果BlankFinal构造函数里的两行的代码注释掉会编译不通过。

### 2.final局部变量

废话不多说，撸一把代码

```
public class FinalTest {
    private int g1 = 127;
    private int g2 = 128;
    private int g3 = 32767;
    private int g4 = 32768;


    public static void main(String[] args) {

        int n1 = 2019;
        final int n2 = 2019;

        String s = "20190718";
        String s1 = n1 + "0718";
        String s2 = n2 + "0718";

        System.out.println(s == s1); //false
        System.out.println(s == s2); //true

    }
}

```

上面的类变量g1-g4先不要管，下面会解释其中的用意。运行结果发现第一个打印的是false，第二个是true，下面分析原因，先看一下编译之后的结果：
![程序编译结果]($resource/%E7%A8%8B%E5%BA%8F%E7%BC%96%E8%AF%91%E7%BB%93%E6%9E%9C.jpg)

 图二

图二中，发现很有意思的事情，n1编译之后为short类型了,n2编译之后为boolean类型了，这个其实算是编译器优化了，为每个常量选择合适的类型，这样可以减少虚拟机的内存空间，对于final局部常量来说对运行期是没有影响的（局部变量和字段（实类变量、类变量）是有区别，它在常量池是没有Fieldref符号引用，自然没有访问标志信息，因此将局部声明为final对运行期是没有影响的），非0的时候编译之后为true，0的时候为false；

接下来发现s与s2的是一样的，即在常量池中的位置是同一个位置，字符索引是同一个，说白了就是内存中同一个值，所以就很好的解释为什么s==s2是true。对于s1来说，其实是通过StringBuilder类把n1和“0718”相加的，直接来证明一下，如下图反编译之后的结果来看：

![](https://mmbiz.qpic.cn/mmbiz_png/IibUVnJ665Wq9501E22D8RBpFicNPacxSO47kbH5Q6al8GicGA7cu1AiakhQbjn2lynLyAHkgIfedicv1IdQDEKodzw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图三

图三中，24行中看出是通过StringBuilder的append进行相加的，然后通过StringBuilder的toString方法获取值，这就解释了s==s1为false的原因。

下面是扩展内容，接下来解释g1-g4的用途,先看一下g1-g4常量池的情况

![](https://mmbiz.qpic.cn/mmbiz_png/IibUVnJ665Wq9501E22D8RBpFicNPacxSOV6cR8iaJWcUjDkgMIymbWhKvbSK1vmkjbKQqvWZNwVRqzGRwUFLPmPQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图四

上面是编译后的部分截图，了解常量池的朋友从图中#5知道32768是在常量池的，但是127、128、32767不再常量池，那再哪里呢，再看反编译片段。

![](https://mmbiz.qpic.cn/mmbiz_png/IibUVnJ665Wq9501E22D8RBpFicNPacxSOjmpEQWVI7FicMTCxD1FsVFj6K91BPvSJrCKeGaEzib0ffoURAlAI88Xw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

图五

发现127是由bipush送至栈顶，而128和32767送至栈顶的sipush送至栈顶，

这两个命令是是什么意思？

bipush:将单字节常量（-128~127）推送至栈顶

sipush:将一个短整型常量值（-32768~32767）推送至栈顶

相当于虚拟机缓存了-32768~32767的数值，不需要在常量池定义了，类比于Integer类，Integer会缓存-128~127，超过了这个范围则从新生成Integer对象 。
