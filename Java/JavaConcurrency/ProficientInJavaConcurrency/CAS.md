### CAS(Compare And Swap)

[TOC]



#### 6.1 什么是CAS

- synchronized 关键字与 Lock 等锁机制都是悲观锁：无论做何种操作，首先都需要先上锁，接下来再去执行后续操作，从而确保了接下来的所有 操作都是由当前这个线程来执行的。

- 乐观锁：线程在操作之前不会做任何预先的处理，而是直接去执行；当在最后执行变量更新的时候， 当前线程需要有一种机制来确保当前被操作的变量是没有被其他线程修改的。CAS 是乐观锁的一种极为重要的实现方式。

 

比较与交换：这是一个不断循环的过程，一直到变量值被修改成功为止。CAS 本身是由硬件指令来提供支持的，换句话说，硬件中是通过一个原子指令来实现比较与交换的；因此，CAS 可以确保变量操作的原子性。

示例代码：计数器

```java
package com.gjxaiou.cas;

public class MyTest1 {
    private int count;

    public int getCount() {
        return count;
    }

    public void increaseCount() {
        // 这行在多线程运行时会出问题，通过字节码进行分析
        // 读取 =》 修改 =》 写入：这三个操作并非原子操作
        ++this.count;
    }
}
```

对应的字节码为：

```java
E:\Program\Project\NotesProject\JavaConcurrency\ProficientInJavaConcurrency\src\main\java\com\gjxaiou\cas>javap -v MyTest1
警告: 二进制文件MyTest1包含com.gjxaiou.cas.MyTest1
Classfile /E:/Program/Project/NotesProject/JavaConcurrency/ProficientInJavaConcurrency/src/main/java/com/gjxaiou/cas/MyTest1.class
  Last modified 2021-3-21; size 363 bytes
  MD5 checksum e802d2bde735dc5b83f98a2355d03dd1
  Compiled from "MyTest1.java"
public class com.gjxaiou.cas.MyTest1
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #4.#16         // java/lang/Object."<init>":()V
   #2 = Fieldref           #3.#17         // com/gjxaiou/cas/MyTest1.count:I
   #3 = Class              #18            // com/gjxaiou/cas/MyTest1
   #4 = Class              #19            // java/lang/Object
   #5 = Utf8               count
   #6 = Utf8               I
   #7 = Utf8               <init>
   #8 = Utf8               ()V
   #9 = Utf8               Code
  #10 = Utf8               LineNumberTable
  #11 = Utf8               getCount
  #12 = Utf8               ()I
  #13 = Utf8               increaseCount
  #14 = Utf8               SourceFile
  #15 = Utf8               MyTest1.java
  #16 = NameAndType        #7:#8          // "<init>":()V
  #17 = NameAndType        #5:#6          // count:I
  #18 = Utf8               com/gjxaiou/cas/MyTest1
  #19 = Utf8               java/lang/Object
{
  public com.gjxaiou.cas.MyTest1();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 4: 0

  public int getCount();
    descriptor: ()I
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: getfield      #2                  // Field count:I
         4: ireturn
      LineNumberTable:
        line 8: 0

  public void increaseCount();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=3, locals=1, args_size=1
         0: aload_0
         1: dup
         2: getfield      #2                 // Field count:I // 获取成员变量，然后放入栈顶
         5: iconst_1                          // 将数字 1 放到当前栈的栈顶
         6: iadd                              // 将当前栈栈顶和下面一个元素进行求和
         7: putfield      #2                  // Field count:I // 将求和之后结果放到栈顶
        10: return
      LineNumberTable:
        line 13: 0
        line 14: 10
}
SourceFile: "MyTest1.java"s
```

保证正确性，方式一将该读和写方法加上 synchronized 关键字。缺点就是：读方法不应该只能当线程访问，但是加上 synchronized 之后也只能单个线程访问。

```java
package com.gjxaiou.cas;

public class MyTest1 {
    private int count;

    public synchronized int getCount() {
        return count;
    }

    public synchronized void increaseCount() {
        // 这行在多线程运行时会出问题，通过字节码进行分析
        ++this.count;
    }
}
```

为什么在读取操作前面也需要加上 synchronized 关键字：

因为 synchronized 除了进行上锁进行资源隔离的作用外，还可以实现变量的可见性，通过在两个方法上都加入 synchronized 关键字，即在 `increaseCount` 方法中完成 ++this.count 并且写入的时候，根据 happen-before 原则，写入值会立刻刷新到高速缓存上。即如果不在 `getCount()` 方法上加上 synchronized 修饰则其他线程获取到的可能是旧的值。

CAS 代码示例：

```java
package com.gjxaiou.cas;

import java.util.concurrent.atomic.AtomicInteger;

public class MyTest2 {
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(5);

        System.out.println(atomicInteger.get());
        // 将值设置为 8 并且返回旧的值
        System.out.println(atomicInteger.getAndSet(8));
        System.out.println(atomicInteger.get());
        // 将值自增 1 然后返回旧值
        System.out.println(atomicInteger.getAndIncrement());
        System.out.println(atomicInteger.get());
    }
}
```

输出结果为：

```java
5
5
8
8
9
```

针对 atomicInteger 的源码分析为：

```java
// 其中主要分析 getAndSet 方法
public final int getAndSet(int newValue) {
    // 三个参数含义：当前被操作的 AtomicInteger 对象，操作的值在 AtomicInteger 对象中的内存偏移位置，将要被写入的新的值。
    return unsafe.getAndSetInt(this, valueOffset, newValue);
}

// OpenJDK 中的 unsafe 类中的 getAndSetInt 方法为
public final int getAndSetInt(Object o, long offset, int newValue) {
    int v;
    do {
        v = getIntVolatile(o, offset);
        // compareAndSwapInt 是 Native 方法，是由 C++ 实现的，参数为：要操作对象，即 AtomicInteger 的引用，映射到 C++
        // 就是其内存位置；要操作的变量在当前对象中的内存偏移位置；变量预期的值；即将要写入的新的值
    } while (!compareAndSwapInt(o, offset, v, newValue));
    return v;
}
```

[Unsafe 类分析](https://tech.meituan.com/2019/02/14/talk-about-java-magic-class-unsafe.html)





#### 6.2 CAS执行过程

对于CAS来说，其操作数主要涉及到如下三个:

- 需要被操作的内存值 v

- 需要进行比较的值 A

- 需要进行写入的值 B

只有当 V==A 的时候，CAS 才会通过原子操作的手段来将 v 的值更新为 B。

 

关于CAS的限制或是问题: .

- 循环开销问题：本质上通过 do-while 循环实现，并发量大的情况下会导致线程一直自旋 

- 只能保证一个变量的原子操作：可以通过 AtomicReference 来实现对多个变量的原子操作 ，即将多个变量放在同一个对象中进行原子操作。

- ABA 问题：采用时间戳或者版本号来做标记，从执行的结果是没有错但是语义出现了错误。可以看一下 TimeStampLock 源码。