# Java中static作用及用法详解

[原文地址链接:](https://blog.csdn.net/fengyuzhengfan/article/details/38082999)
@toc

## 1.1概述：

   static是静态修饰符，什么叫静态修饰符呢？大家都知道，在程序中任何变量或者代码都是在编译时由系统自动分配内存来存储的，而所谓**静态就是指在编译后所分配的内存会一直存在，直到程序退出内存才会释放这个空间**，也就是只要程序在运行，那么这块内存就会一直存在。这样做有什么意义呢？在Java程序里面，所有的东西都是对象，而**对象的抽象就是类**，对于一个类而言，如果要使用他的成员，那么普通情况下必须先实例化对象后，通过对象的引用才能够访问这些成员，但是**用static修饰的成员可以通过类名加“.”进行直接访问。**

static表示“全局”或者“静态”的意思，用来修饰成员变量和成员方法，也可以形成静态static代码块，但是Java语言中没有全局变量的概念。

 **被static修饰的成员变量和成员方法独立于该类的任何对象**。也就是说，它不依赖类特定的实例，被类的所有实例共享。只要这个类被加载，Java虚拟机就能根据类名在运行时数据区的方法区内定找到他们。因此，static对象可以在它的任何对象创建之前访问，无需引用任何对象。

用public修饰的static成员变量和成员方法本质是全局变量和全局方法，当声明它类的对象时，不生成static变量的副本，而是**类的所有实例共享同一个static变量**。

static变量前可以**有private修饰**，表示这个变量可以在类的静态代码块中，或者类的其他静态成员方法中使用（当然也可以在非静态成员方法中使用--废话），但是**不能在其他类中通过类名来直接引用**，这一点很重要。实际上你需要搞明白，**private是访问权限限定，static表示不要实例化就可以使用**，这样就容易理解多了。static前面加上其它访问权限关键字的效果也以此类推。

static修饰的成员变量和成员方法习惯上称为静态变量和静态方法，可以直接通过类名来访问，访问语法为：
```
类名.静态方法名(参数列表...) 
类名.静态变量名
```
用static修饰的代码块表示静态代码块，当Java虚拟机（JVM）加载类时，就会执行该代码块（用处非常大，呵呵）。


## 1.2 static变量

按照是否静态,对类成员变量进行分类可分两种：一种是被static修饰的变量，叫静态变量或类变量；另一种是没有被static修饰的变量，叫实例变量。两者的区别是：

- 对于静态变量在内存中只有一个拷贝（节省内存），JVM只为静态分配一次内存，在**加载类的过程中完成静态变量的内存分配**，可用类名直接访问（方便），当然也**可以通过对象来访问（但是这是不推荐的）**。

- 对于实例变量，每创建一个实例，就会为实例变量分配一次内存，实例变量可以在内存中有多个拷贝，互不影响（灵活）。 

## 1.3static方法

静态方法可以直接通过类名调用，任何的实例也都可以调用，因此**静态方法中不能用this和super关键字**，**不能直接访问所属类的实例变量和实例方法(就是不带static的成员变量和成员成员方法)**，只能访问所属类的静态成员变量和成员方法。因为实例成员与特定的对象关联！这个需要去理解，想明白其中的道理，不是记忆！！！因为static方法独立于任何实例，因此static方法必须被实现，而不能是抽象的abstract。 

## 1.4static代码块

static代码块也叫静态代码块，是**在类中独立于类成员的static语句块**，可以有多个，位置可以随便放，它不在任何的方法体内，JVM加载类时会执行这些静态的代码块，如果static代码块有多个，JVM将按照它们在类中出现的先后顺序依次执行它们，**每个代码块只会被执行一次**。例如： 

```java
public class Test5 { 
        private static int a; 
        private int b;
        static { 
                Test5.a = 3; //因为变量a使用static修饰，因此是静态变量，可以直接使用类名进行调用；
                System.out.println(a); 
                Test5 t = new Test5(); 
                t.f(); 
                t.b = 1000; 
                System.out.println(t.b); 
        }
        static { 
                Test5.a = 4; 
                System.out.println(a); 
        }
        public static void main(String[] args) { 
                // TODO 自动生成方法存根 
        }
        static { 
                Test5.a = 5; 
                System.out.println(a); 
        } 
        public void f() { 
                System.out.println("hhahhahah"); 
        } 
}
```

运行结果：
```
3
hhahhahah
1000
4
5
```
利用静态代码块可以对一些static变量进行赋值，最后再看一眼这些例子，都一个static的main方法，这样JVM在运行main方法的时候可以直接调用而不用创建实例。 



1. **static和final一块用表示什么**
static final用来修饰成员变量和成员方法，可简单理解为“全局常量”！
- 对于变量，表示一旦给值就**不可修改**，并且通过类名可以访问。
- 对于方法，表示**不可覆盖**，并且可以通过类名直接访问。       

**特别要注意一个问题：**

对于被static和final修饰过的实例常量，实例本身不能再改变了，但对于一些容器类型（比如，ArrayList、HashMap）的实例变量，不可以改变容器变量本身，但**可以修改容器中存放的对象**，这一点在编程中用到很多。看个例子：

```java
public class TestStaticFinal {
        private static final String strStaticFinalVar ="aaa";
        private static String strStaticVar =null;
        private final String strFinalVar = null;
        private static final int intStaticFinalVar = 0;
        private static final Integer integerStaticFinalVar =new Integer(8);
        private static final ArrayList<String>alStaticFinalVar = new ArrayList<String>();
        private void test() {
                System.out.println("-------------值处理前----------\r\n");
                System.out.println("strStaticFinalVar=" +strStaticFinalVar + "\r\n");
                System.out.println("strStaticVar=" +strStaticVar + "\r\n");
                System.out.println("strFinalVar=" +strFinalVar + "\r\n");
                System.out.println("intStaticFinalVar=" +intStaticFinalVar + "\r\n");
                System.out.println("integerStaticFinalVar=" +integerStaticFinalVar + "\r\n");
                System.out.println("alStaticFinalVar=" +alStaticFinalVar + "\r\n");
                //strStaticFinalVar="哈哈哈哈";        //错误，final表示终态,不可以改变变量本身.
                strStaticVar = "哈哈哈哈";               //正确，static表示类变量,值可以改变.
                //strFinalVar="呵呵呵呵";                    //错误, final表示终态，在定义的时候就要初值（哪怕给个null），一旦给定后就不可再更改。
                //intStaticFinalVar=2;                        //错误, final表示终态，在定义的时候就要初值（哪怕给个null），一旦给定后就不可再更改。
                //integerStaticFinalVar=new Integer(8);            //错误, final表示终态，在定义的时候就要初值（哪怕给个null），一旦给定后就不可再更改。
                alStaticFinalVar.add("aaa");       //正确，容器变量本身没有变化，但存放内容发生了变化。这个规则是非常常用的，有很多用途。
                alStaticFinalVar.add("bbb");       //正确，容器变量本身没有变化，但存放内容发生了变化。这个规则是非常常用的，有很多用途。
                System.out.println("-------------值处理后----------\r\n");
                System.out.println("strStaticFinalVar=" +strStaticFinalVar + "\r\n");
                System.out.println("strStaticVar=" +strStaticVar + "\r\n");
                System.out.println("strFinalVar=" +strFinalVar + "\r\n");
                System.out.println("intStaticFinalVar=" +intStaticFinalVar + "\r\n");
                System.out.println("integerStaticFinalVar=" +integerStaticFinalVar + "\r\n");
                System.out.println("alStaticFinalVar=" +alStaticFinalVar + "\r\n");
        }
        public static void main(String args[]) {
                new TestStaticFinal().test();
        }
}
```

运行结果如下：
```java
-------------值处理前----------

strStaticFinalVar=aaa

strStaticVar=null

strFinalVar=null

intStaticFinalVar=0

integerStaticFinalVar=8

alStaticFinalVar=[]

-------------值处理后----------

strStaticFinalVar=aaa

strStaticVar=哈哈哈哈

strFinalVar=null

intStaticFinalVar=0

integerStaticFinalVar=8

alStaticFinalVar=[aaa, bbb]
Process finished with exit code 0
```

看了上面这个例子，就清楚很多了，但必须明白：通过static final修饰的容器类型变量中所“装”的对象是可改变的。这和一般基本类型和类型变量差别很大的地方。

## 1.5 Java static块和static方法的使用区别

- static 块:如果有些代码必须在项目启动的时候就执行,就需要使用静态代码块,这种代码是**主动执行的**；
- static 方法: 需要在项目启动的时候就初始化但是不执行,在不创建对象的情况下,可以供其他程序调用,而在调用的时候才执行，这需要使用静态方法,这种代码是被动执行的。 静态方法在类加载的时候 就已经加载 可以用类名直接调用。

**静态代码块和静态方法的区别是：**
- 静态代码块是自动执行的;
- 静态方法是被调用的时候才执行的.

• 静态方法：如果我们在程序编写的时候需要一个不实例化对象就可以调用的方法，我们就可以使用静态方法，具体实现是在方法前面加上static，如下：
```java
public static void method(){
}
```


**在使用静态方法的时候需要注意一下几个方面：**

在静态方法里只能直接调用同类中其他的静态成员（包括变量和方法），而不能直接访问类中的非静态成员。这是因为，对于非静态的方法和变量，需要先创建类的实例对象后才可使用，而静态方法在使用前不用创建任何对象。（备注：**静态变量是属于整个类的变量而不是属于某个对象的**）

静态方法不能以任何方式引用this和super关键字，因为静态方法在使用前不用创建任何实例对象，当静态方法调用时，this所引用的对象根本没有产生。

静态程序块：当一个类需要在被载入时就执行一段程序，这样可以使用静态程序块。

## 1.6总结

有时你希望定义一个类成员，使它的使用完全独立于该类的任何对象。通常情况下，类成员必须通过它的类的对象访问，但是可以创建这样一个成员，它能够被它自己使用，而不必引用特定的实例。在成员的声明前面加上关键字static(静态的)就能创建这样的成员。如果一个成员被声明为static，它就能够在它的类的任何对象创建之前被访问，而不必引用任何对象。你可以将方法和变量都声明为static。static 成员的最常见的例子是main( ) 。因为在程序开始执行时必须调用main() ，所以它被声明为static。

声明为static的变量实质上就是全局变量。当声明一个对象时，并不产生static变量的拷贝，而是该类所有的实例变量共用同一个static变量。声明为static的方法有以下几条限制：

- 它们仅能调用其他的static方法。
- 它们只能访问static数据。
- 它们不能以任何方式引用this或super（关键字super 与继承有关，在下一章中描述）。

如果你需要通过计算来初始化你的static变量，你可以声明一个static块，Static 块仅在该类被加载时执行一次。