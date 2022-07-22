# 加餐一：用一篇文章带你了解专栏中用到的所有Java语法

尽管说设计模式跟编程语言没有直接关系，但是，我们也无法完全脱离代码来讲设计模式。

我本人熟悉的是 Java 语言，所以专栏中的代码示例我都是用 Java 语言来写的。考虑到有些同学并不熟悉 Java 语言，我今天用一篇文章介绍一下专栏中用到的 Java 语法。

# Hello World

我们先来看一下，Java 语言的 Hello World 代码如何编写。

在 Java 中，所有的代码都必须写在类里面，所以，我们定义一个 HelloWorld 类。main() 函数是程序执行的入口。main() 函数中调用了 Java 开发包 JDK 提供的打印函数 System.out.println() 来打印 hello world 字符串。除此之外，Java 中有两种代码注释方式，第一种是“// 注释…”双斜杠，表示后面的字符串都是注释，第二种是“/注释…/”， 表示中间的内容都是注释。

```java
/*hello world 程序 */
public class HelloWorld {
    public static void main(String []args) {
        System.out.println("Hello World"); // 打印 Hello World
    }
}
```

# 基本数据类型

Java 语言中的基本数据类型跟其他语言类似，主要有下面几种：

- 整型类型：byte（字节）、short（短整型）、int（整型）、long（长整型） 
- 浮点类型：float（单精度浮点）、double（双精度浮点）
- 字符型：char
- 布尔型：boolean

如下，我们来定义一个基本类型变量：

`int a = 6;`

除此之外，为了方便我们使用，**Java 还提供了一些封装这些基本数据类型的类，这些类实现了一些常用的功能函数**，可以直接拿来使用。常用的有下面几个类：

- Integer：对应封装了基本类型 int； 
- Long：对应封装了基本类型 long； 
- Float：对应封装了基本类型 float； 
- Double：对应封装了基本类型 double； 
- Boolean：对应封装了基本类型 boolean； 
- String：对应封装了字符串类型 char\[\]。

如下，我们来定义一个 Integer 对象：

`Integer oa = new Integer(6);`

# 数组

Java 中，我们使用 \[\] 来定义一个数组，如下所示：

```java
int a[] = new int[10] // 定义一个长度为 10 的 int 类型数组
```

在 Java 中，我们通过如下方式访问数组中的元素：

```java
a[1] = 3; // 将下标是 1 的数组元素赋值为 3
System.out.println(a[2]); // 打印下标是 2 的数组元素值
```



# 流程控制

流程控制语句跟其他语言类似，主要有下面几种。

if-else 语句，代码示例如下所示：

```java
// 用法一
int a;
if (a > 1) {
    // 执行代码块
} else {
    // 执行代码块
} 

// 用法二
int a;
if (a > 1) {
    // 执行代码块
} else if (a == 1) {
    // 执行代码块
} else {
    // 执行代码块
}
```

switch-case 语句，代码示例如下所示：

```java
int a;
switch (a) {
    case 1:
        // 执行代码块
        break;
    case 2:
        // 执行代码块
        break;
    default:
        // 默认执行代码
}
```

for、while 循环，代码示例如下所示：

```java
for (int i = 0; i < 10; ++i) {
    // 循环执行 10 次此代码块
}

int i = 0;
while (i < 10) {
    // 循环执行 10 次此代码块
}
```

continue、break、return，代码示例如下所示：

```java
for (int i = 0; i < 10; ++i) {
    if (i == 4) {
        continue; // 跳过本次循环，不会打印出 4 这个值
    }
    System.out.println(i);
} 

for (int i = 0; i < 10; ++i) {
    if (i == 4) {
        break; // 提前终止循环，只会打印 0、1、2、3
    }
    System.out.println(i);
} 

public void func(int a) {
    if (a == 1) {
        return; // 结束一个函数，从此处返回
    }
    System.out.println(a);
}
```

# 类、对象

Java 语言使用关键词 class 来定义一个类，类中包含成员变量（也叫作属性）和方法（也叫作函数），其中有一种特殊的函数叫作构造函数，其命名比较固定，跟类名相同。除此之外，Java 语言通过 new 关键词来创建一个类的对象，并且可以通过构造函数，初始化一些成员变量的值。代码示例如下所示：

```java
public class Dog { // 定义了一个 Dog 类
    private int age; // 属性或者成员变量
    private int weight;
    public Dog(int age, int weight) { // 构造函数
        this.age = age;
        this.weight = weight;
    } 
    public int getAge() { // 函数或者方法
        return age;
    } 
    public int getWeigt() {
        return weight;
    }
    public void run() {
        // ...
    }
} 
Dog dog1 = new Dog(2, 10);// 通过 new 关键词创建了一个 Dog 对象 dog1
int age = dog1.getAge();// 调用 dog1 的 getAge() 方法
dog1.run();// 调用 dog1 的 run() 方法   
```

# 权限修饰符

在前面的代码示例中，我们多次用到 private、public，它们跟 protected 一起，构成了Java 语言的三个权限修饰符。权限修饰符可以修饰函数、成员变量。

private 修饰的函数或者成员变量，只能在类内部使用。protected 修饰的函数或者成员变量，可以在类及其子类内使用。public 修饰的函数或者成员变量，可以被任意访问。

除此之外，权限修饰符还可以修饰类，不过，专栏中所有的类定义都是 public 访问权限的，所以，我们可以不用去了解三个修饰符修饰类的区别。

对于权限修饰符的理解，我们可以参看下面的代码示例：

```java
public class Dog {// public 修饰类
    private int age; // private 修饰属性，只能在类内部使用
    private int weight;
    public Dog(int age, int weight) {
        this.age = age;
        this.weight = weight;
    } 
    public int getAge() { //public 修饰的方法，任意代码都是可以调用
        return age;
    } 
    public void run() {
        // ...
    }
}
```

# 继承

Java 语言使用 extends 关键字来实现继承。被继承的类叫作父类，继承类叫作子类。子类继承父类的所有非 private 属性和方法。具体的代码示例如下所示：

```java
public class Animal { // 父类
    protected int age;
    protected int weight;
    public Animal(int age, int weight) {
        this.age = age;
        this.weight = weight;
    } 
    public int getAge() { // 函数或者方法
        return age;
    }
    public int getWeigt() {
        return weight;
    } 
    public void run() {
        // ...
    }
} 

public class Dog extends Animal { // 子类
    public Dog(int age, int weight) { // 构造函数
        super(age, weight); // 调用父类的构造函数
    } 
    public void wangwang() {
        //...
    }
} 

public class Cat extends Animal { // 子类
    public Cat(int age, int weight) { // 构造函数
        super(age, weight); // 调用父类的构造函数
    }
    public void miaomiao() {
        //...
    }
} 
// 使用举例
Dog dog = new Dog(2, 8);
dog.run();
dog.wangwang();
Cat cat = new Cat(1, 3);
cat.run();
cat.miaomiao();
```

# 接口

Java 语言通过 interface 关键字来定义接口。接口中只能声明方法，不能包含实现，也不能定义属性。类通过 implements 关键字来实现接口中定义的方法。在专栏的第 8 讲中， 我们会详细讲解接口，所以，这里我只简单介绍一下语法。具体的代码示例如下所示：

```java
public interface Runnable {
    void run();
} 

public class Dog implements Runnable {
    private int age; // 属性或者成员变量
    private int weight;
    public Dog(int age, int weight) { // 构造函数
        this.age = age;
        this.weight = weight;
    } 
    public int getAge() { // 函数或者方法
        return age;
    }
    public int getWeigt() {
        return weight;
    } 
    @Override
    public void run() { // 实现接口中定义的 run() 方法
        // ...
    }
}
```

# 容器

Java 提供了一些现成的容器。容器可以理解为一些工具类，底层封装了各种数据结构。比如 ArrayList 底层就是数组，LinkedList 底层就是链表，HashMap 底层就是散列表等。这些容器我们可以拿来直接使用，不用从零开始开发，大大提高了编码的效率。具体的代码示例如下所示：

```java
public class DemoA {
    private ArrayList<User> users;
    public void addUser(User user) {
        users.add(user);
    }
}
```

# 异常处理

Java 提供了异常这种出错处理机制。我们可以指直接使用 JDK 提供的现成的异常类，也可以自定义异常。在 Java 中，我们通过关键字 throw 来抛出一个异常，通过 throws 声明函数抛出异常，通过 try-catch-finally 语句来捕获异常。代码示例如下所示：

```java
public class UserNotFoundException extends Exception { // 自定义一个异常
    public UserNotFoundException() {
        super();
    } 
    public UserNotFoundException(String message) {
        super(message);
    } 
    public UserNotFoundException(String message, Throwable e) {
        super(message, e);
    }
} 

public class UserService {
    private UserRepository userRepo;
    public UserService(UseRepository userRepo) {
        this.userRepo = userRepo;
    } 
    public User getUserById(long userId) throws UserNotFoundException {
        User user = userRepo.findUserById(userId);
        if (user == null) { // throw 用来抛出异常
            throw new UserNotFoundException();// 代码从此处返回
        }
        return user;
    }
}

public class UserController {
    private UserService userService;
    public UserController(UserService userService) {
        this.userService = userService;
    } 
    public User getUserById(long userId) {
        User user = null;
        try { // 捕获异常
            user = userService.getUserById(userId);
        } catch (UserNotFoundException e) {
            System.out.println("User not found: " + userId);
        } finally { // 不管异常会不会发生，finally 包裹的语句块总会被执行
            System.out.println("I am always printed.");
        }
        return user;
    }
}
```

# package 包

Java 通过 pacakge 关键字来分门别类地组织类，通过 import 关键字来引入类或者 package。具体的代码示例如下所示：

```java
/*class DemoA*/
package com.xzg.cd; // 包名 com.xzg.cd
public class DemoA {
    //...
} 

/*class DemoB*/
package com.xzg.alg;
import java.util.HashMap; // Java 工具包 JDK 中的类
import java.util.Map;
import com.xzg.cd.DemoA;
public class DemoB {
    //...
}
```

# 总结

今天，我带你一块学习了专栏中用到的所有的 Java 基本语法。不过，我希望你不要纠结于专栏或者某某书籍到底是用什么编程语言来写的。语言层面的东西完全不会限制我的讲解和你的理解。这就像我们读小说一样，不管它是用英语写的，还是中文写的，故事都可以同样精彩。而且，多了解一些 Java 语法，对于你今后阅读 Java 语言编写的书籍或者文档，也很有帮助。

实际上，我之前在 Google 工作的时候，大家都不太在意自己熟悉的是哪种编程语言，很多同事都是“现学现卖”，什么项目适合用什么语言就现学什么语言。除此之外，Google 在招聘的时候，也不限定候选人一定要熟悉哪种编程语言，也很少问跟语言特性相关的问 题。因为他们觉得，编程语言只是一个工具，对于一个有一定学习能力的人，学习一门编程语言并不是件难事。

除此之外，对于专栏中的代码示例，你也可以用你熟悉语言重新实现一遍，我相信这也是件很有意义的事情，也更能加深你对内容的理解。

# 课堂讨论

不同的公司开发使用的编程语言可能不一样，比如阿里一般都是用 Java，今日头条用 Go、C++ 比较多。在招聘上，这些公司都倾向于招聘熟悉相应编程语言的同学，毕竟熟练掌握一门语言也是要花不少时间的，而且用熟悉的编程语言来开发，肯定会更得心应手，更不容易出错。今天课堂讨论的话题有两个：

1.  分享一下你学习一门编程语言的经历，从入门到熟练掌握，大约花了多久的时间？有什么好的学习编程语言的方法？

2.  在一个程序员的技术能力评价体系中，你觉得“熟练使用某种编程语言”所占的比重有多大？

## 精选留言

- Java用的时间最长，大概4-5年，不敢说自己“熟练”掌握了。最近反而觉得不懂的更多了。我没有抓入Java8不放，而是跟着Java的发展，开始学习Java11和Java13的新特性， 紧跟语言的变化，并学习虚拟机和并发编程的相关知识。
- 原谅我这篇文章三十秒就看完了，因为我是JAVA 1.用了多久我也不确定，但是学习方法是有的，首先看视频，资料，动手敲，晚上睡觉前在脑海里回顾一下学了什么，明天在动手敲一遍昨天学的内容，最后用自己的语言将其组织成一篇属于自己的文章。 2.熟练需要看成都，就比如很多人都说看源码感觉没用，看了就忘，也不知道能干嘛。我… 展开

- 从第一次接触Java，到得心应手，大概花了两年时间。这个周期让我理解了学习的非线性。大一开始学习C语言，学的似懂非懂，做了课程设计就放下了，发大二开始学Java，同样似懂非懂。大三开始接触Android开发，用到了Java，才发现自己Java知识不足，于是花时间重学了Java，过程中发现有些东西不理解，又穿插着把C需要的指针内存啃了几遍，大… 展开

- 大学课程中学习了C，工作中自学并使用JAVA，主要用于web和大数据开发，JAVA不仅仅是一门语言，还是一个技术体系，包括了后来的很多技术框架，学习JAVA语言如果有其他语言基础是很快的，精通后面的一些常用框架就需要一些设计模式的积累。所以还是学习能力最重要：学习，操练，总结，分享，这个路线是我认为很快捷的学习方法。最后学习的东西越多，越容易融会贯通，后来使用Python做推荐系统，我们几个JAVA开发人员，…展开

