# Java异常处理和最佳实践（含案例分析）

如何处理Java异常？作者查看了一些异常处理的规范，对 Java 异常处理机制有更深入的了解，并将自己的学习内容记录下来，希望对有同样困惑的同学提供一些帮助。

## 一、概述

最近在代码 CR 的时候发现一些值得注意的问题，特别是在对 Java 异常处理的时候，比如有的同学对每个方法都进行 try-catch，在进行 IO 操作时忘记在 finally 块中关闭连接资源等等问题。回想自己对 java 的异常处理也不是特别清楚，看了一些异常处理的规范，并没有进行系统的学习，所以为了对 Java 异常处理机制有更深入的了解，我查阅了一些资料将自己的学习内容记录下来，希望对有同样困惑的同学提供一些帮助。

在 Java 中处理异常并不是一个简单的事情，不仅仅初学者很难理解，即使一些有经验的开发者也需要花费很多时间来思考如何处理异常，包括需要处理哪些异常，怎样处理等等。

在写本文之前，通过查阅相关资料了解如何处理 Java 异常，首先查看了阿里巴巴 Java 开发规范，其中有 15 条关于异常处理的说明，这些说明告诉了我们应该怎么做，但是并没有详细说明为什么这样做，比如为什么推荐使用 try-with-resources 关闭资源 ，为什么 finally 块中不能有 return 语句，这些问题当我们从字节码层面分析时，就可以非常深刻的理解它的本质。

通过本文的的学习，你将有如下收获：

- 了解 Java 异常的分类，什么是检查异常，什么是非检查异常
- 从字节码层面理解 Java 的异常处理机制，为什么 finally 块中的代码总是会执行
- 了解 Java 异常处理的不规范案例
- 了解 Java 异常处理的最佳实践
- 了解项目中的异常处理，什么时候抛出异常，什么时候捕获异常

## 二、java 异常处理机制

### （一）java 异常分类

![图片](Java%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E5%92%8C%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%EF%BC%88%E5%90%AB%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90%EF%BC%89%20.resource/640.jpeg)

**总结：**

- Thorwable类（表示可抛出）是所有异常和错误的超类，两个直接子类为 Error 和 Exception，分别表示错误和异常。
- 其中异常类 Exception 又分为运行时异常(RuntimeException)和非运行时异常， 这两种异常有很大的区别，也称之为非检查异常（Unchecked Exception）和检查异常（Checked Exception），其中 Error 类及其子类也是非检查异常。

#### 1.检查异常和非检查异常

- **检查异常：也称为「编译时异常」**，编译器在编译期间检查的那些异常。由于编译器「检查」这些异常以确保它们得到处理，因此称为「检查异常」。如果抛出检查异常，那么编译器会报错，需要开发人员手动处理该异常，要么捕获，要么重新抛出。**除了 RuntimeException 之外，所有直接继承 Exception 的异常都是检查异常。**
- **非检查异常：也称为「运行时异常」**，编译器不会检查运行时异常，在抛出运行时异常时编译器不会报错，当运行程序的时候才可能抛出该异常。**Error 及其子类和RuntimeException 及其子类都是非检查异常。**

说明：检查异常和非检查异常是针对编译器而言的，是编译器来检查该异常是否强制开发人员处理该异常：

- 检查异常导致异常在方法调用链上显式传递，而且一旦底层接口的检查异常声明发生变化，会导致整个调用链代码更改。
- 使用非检查异常不会影响方法签名，而且调用方可以自由决定何时何地捕获和处理异常

**建议使用非检查异常让代码更加简洁，而且更容易保持接口的稳定性。**

#### 2.检查异常举例

在代码中使用 throw 关键字手动抛出一个检查异常，编译器提示错误，如下图所示：

![图片](Java%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E5%92%8C%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%EF%BC%88%E5%90%AB%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90%EF%BC%89%20.resource/640-20230320151016592.png)

通过编译器提示，有两种方式处理检查异常，要么将异常添加到方法签名上，要么捕获异常：

![图片](Java%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E5%92%8C%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%EF%BC%88%E5%90%AB%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90%EF%BC%89%20.resource/640-20230320151016655-9296216.png)

**方式一：**将异常添加到方法签名上，通过 **throws** 关键字抛出异常，由调用该方法的方法处理该异常：

![图片](Java%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E5%92%8C%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%EF%BC%88%E5%90%AB%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90%EF%BC%89%20.resource/640-20230320151016582.png)

**方式二：**使用 try-catch 捕获异常，在 catch 代码块中处理该异常，下面的代码是将检查异常包装在非检查异常中重新抛出，这样编译器就不会提示错误了，关于如何处理异常后面会详细介绍：

![图片](Java%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E5%92%8C%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%EF%BC%88%E5%90%AB%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90%EF%BC%89%20.resource/640-20230320151016773.png)

#### 3.非检查异常举例

所有继承 RuntimeException 的异常都是非检查异常，直接抛出非检查异常编译器不会提示错误：

![图片](Java%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E5%92%8C%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%EF%BC%88%E5%90%AB%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90%EF%BC%89%20.resource/640-20230320151016784.png)

#### 4.自定义检查异常

自定义检查异常只需要**继承 Exception** 即可，如下代码所示：

![图片](Java%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E5%92%8C%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%EF%BC%88%E5%90%AB%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90%EF%BC%89%20.resource/640-20230320151016694.png)

自定义检查异常的处理方式前面已经介绍，这里不再赘述。

#### 5.自定义非检查异常

自定义非检查异常只需要**继承 RuntimeException** 即可，如下代码所示：

![图片](Java%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E5%92%8C%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%EF%BC%88%E5%90%AB%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90%EF%BC%89%20.resource/640-20230320151016780-9297282.png)

### （二）从字节码层面分析异常处理

前面已经简单介绍了一下Java 的异常体系，以及如何自定义异常，下面我将从字节码层面分析异常处理机制，通过字节码的分析你将对 try-catch-finally 有更加深入的认识。

#### 1.try-catch-finally的本质

首先查阅 jvm 官方文档，有如下的描述说明：

![图片](Java%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E5%92%8C%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%EF%BC%88%E5%90%AB%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90%EF%BC%89%20.resource/640-20230320151016655.png)

从官方文档的描述我们可以知道，图片中的字节码是在 JDK 1.6 （class 文件的版本号为50，表示java编译器的版本为jdk 1.6）及之前的编译器生成的，因为有 jsr 和 ret 指令可以使用。然而在 idea 中通过 **jclasslib 插件**查看 try-catch-finally 的字节码文件并没有 jsr/ret 指令，通过查阅资料，有如下说明：

**jsr / ret 机制最初用于实现finally块，但是他们认为节省代码大小并不值得额外的复杂性，因此逐渐被淘汰了。Sun JDK 1.6之后的javac就不生成jsr/ret指令了，那finally块要如何实现？**

**javac采用的办法是把finally块的内容复制到原本每个jsr指令所在的地方，这样就不需要jsr/ret了，代价则是字节码大小会膨胀，但是降低了字节码的复杂性，因为减少了两个字节码指令（jsr/ret）。**

##### **案例一：try-catch 字节码分析**

在 JDK 1.8 中 try-catch 的字节码如下所示：

![图片](Java%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E5%92%8C%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%EF%BC%88%E5%90%AB%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90%EF%BC%89%20.resource/640-20230320151016823.png)

这里需要说明一下 athrow 指令的作用：

![图片](Java%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E5%92%8C%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%EF%BC%88%E5%90%AB%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90%EF%BC%89%20.resource/640-20230320151016753.png)

异常表

![图片](Java%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E5%92%8C%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%EF%BC%88%E5%90%AB%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90%EF%BC%89%20.resource/640-20230320151016582-9296216.png)

**athrow指令：**在Java程序中显示抛出异常的操作（throw语句)都是由 athrow指令来实现的，athrow 指令抛出的Objectref 必须是类型引用，并且必须作为 Throwable 类或 Throwable 子类的实例对象。它从操作数堆栈中弹出，然后通过在当前方法的异常表中搜索与 objectref 类匹配的第一个异常处理程序：

- 如果在异常表中找到与 objectref 匹配的异常处理程序，PC 寄存器被重置到用于处理此异常的代码的位置，然后会清除当前帧的操作数堆栈，objectref 被推回操作数堆栈，执行继续。
- 如果在当前框架中没有找到匹配的异常处理程序，则弹出该栈帧，该异常会重新抛给上层调用的方法。如果当前帧表示同步方法的调用，那么在调用该方法时输入或重新输入的监视器将退出，就好像执行了监视退出指令(monitorexit)一样。
- 如果在所有栈帧弹出前仍然没有找到合适的异常处理程序，这个线程将终止。

**异常表：**异常表中用来记录程序计数器的位置和异常类型。如上图所示，表示的意思是：如果在 8 到 16 （不包括16）之间的指令抛出的异常匹配 MyCheckedException 类型的异常，那么程序跳转到16 的位置继续执行。

**分析上图中的字节码：**第一个 athrow 指令抛出 MyCheckedException 异常到操作数栈顶，然后去到异常表中查找是否有对应的类型，异常表中有 MyCheckedException ，然后跳转到 16 继续执行代码。第二个 athrow 指令抛出 RuntimeException 异常，然后在异常表中没有找到匹配的类型，当前方法强制结束并弹出当前栈帧，该异常重新抛给调用者，任然没有找到匹配的处理器，该线程被终止。

##### **案例二：try-catch-finally 字节码分析**

在刚刚的代码基础之上添加 finally 代码块，然后分析字节码如下：

![图片](Java%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E5%92%8C%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%EF%BC%88%E5%90%AB%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90%EF%BC%89%20.resource/640.png)

异常表的信息如下：

![图片](Java%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E5%92%8C%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%EF%BC%88%E5%90%AB%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90%EF%BC%89%20.resource/640-20230320151016648.png)

添加 finally 代码块后，在异常表中新增了一条记录，捕获类型为 any，这里解释一下这条记录的含义：

在 8 到 27（不包括27） 之间的指令执行过程中，抛出或者返回任何类型的结果都会跳转到 26 继续执行。

从上图的字节码中可以看到，字节码索引为 26 后到结束的指令都是 finally 块中的代码，再解释一下finally块的字节码指令的含义，从 25 开始介绍，finally 块的代码是从 26 开始的：

```java
25 athrow  // 匹配到异常表中的异常 any，清空操作数栈，将 RuntimeExcepion 的引用添加到操作数栈顶，然后跳转到 26 继续执行
26 astore_2  // 将栈顶的引用保存到局部变量表索引为 2 的位置
27 getstatic #2 <java/lang/System.out : Ljava/io/PrintStream;> // 获取类的静态字段引用放在操作数栈顶
30 ldc #9 <执行finally 代码>//将字符串的放在操作数栈顶
32 invokevirtual #4 <java/io/PrintStream.println : (Ljava/lang/String;)V>// 调用方法
35 aload_2// 将局部变量表索引为 2 到引用放到操作数栈顶，这里就是前面抛出的RuntimeExcepion 的引用
36 athrow// 在异常表中没有找到对应的异常处理程序，弹出该栈帧，该异常会重新抛给上层调用的方法
```

##### 案例三：finally 块中的代码为什么总是会执行

![图片](Java%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E5%92%8C%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%EF%BC%88%E5%90%AB%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90%EF%BC%89%20.resource/640-20230320151016811.png)

![图片](Java%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E5%92%8C%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%EF%BC%88%E5%90%AB%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90%EF%BC%89%20.resource/640-20230320151016678.png)

简单分析一下上面代码的字节码指令：字节码指令 2 到 8 会抛出 ArithmeticException 异常，该异常是 Exception 的子类，正好匹配异常表中的第一行记录，然后跳转到 13 继续执行，也就是执行 catch 块中的代码，然后执行 finally 块中的代码，最后通过 goto 31 跳转到 finally 块之外执行后续的代码。

如果 try 块中没有抛出异常，则执行完 try 块中的代码然后继续执行 finally 块中的代码，因为编译器在编译的时候将 finally 块中的代码添加到了 try 块代码后面，执行完 finally 的代码后通过 goto 31 跳转到 finally 块之外执行后续的代码 。

编译器会将 finally 块中的代码放在 try 块和 catch 块的末尾，所以 finally 块中的代码总是会执行。

通过上面的分析，你应该可以知道 finally 块的代码为什么总是会执行了，如果还是有不明白的地方欢迎留言讨论。

##### 案例四：finally 块中使用 return 字节码分析

```java
public int getInt() {
    int i = 0;
    try {
        i = 1;
        return i;
    } finally {
        i = 2;
        return i;
    }
}

public int getInt2() {
    int i = 0;
    try {
        i = 1;
        return i;
    } finally {
        i = 2;
    }
}
```

先分析一下 getInt() 方法的字节码：

![图片](Java%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E5%92%8C%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%EF%BC%88%E5%90%AB%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90%EF%BC%89%20.resource/640-20230320151016631.png)

局部变量表：

![图片](Java%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E5%92%8C%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%EF%BC%88%E5%90%AB%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90%EF%BC%89%20.resource/640-20230320151016727.png)

异常表：

![图片](Java%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E5%92%8C%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%EF%BC%88%E5%90%AB%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90%EF%BC%89%20.resource/640-20230320151016734.png)

**总结：**从上面的字节码中我们可以看出，如果finally 块中有 return 关键字，那么 try 块以及 catch 块中的 return 都将会失效，所以在开发的过程中**不应该在 finally 块中写 return 语句。**

先分析一下 getInt2() 方法的字节码：

![图片](Java%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E5%92%8C%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%EF%BC%88%E5%90%AB%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90%EF%BC%89%20.resource/640-20230320151016744.png)

异常表：

![图片](Java%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E5%92%8C%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%EF%BC%88%E5%90%AB%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90%EF%BC%89%20.resource/640-20230320151016632.png)

从上图字节码的分析，我们可以知道，虽然执行了finally块中的代码，但是返回的值还是 1，这是因为在执行finally代码块之前，将原来局部变量表索引为 1 的值 1 保存到了局部变量表索引为 2 的位置，最后返回到是局部变量表索引为 2 的值，也就是原来的 1。

总结：如果在 finally 块中没有 return 语句，那么无论在 finally 代码块中是否修改返回值，返回值都不会改变，仍然是执行 finally 代码块之前的值。

#### 2.try-with-resources 的本质

下面通过一个打包文件的代码来演示说明一下 try-with-resources 的本质：

```java
/**
     * 打包多个文件为 zip 格式
     *
     * @param fileList 文件列表
     */
    public static void zipFile(List<File> fileList) {
        // 文件的压缩包路径
        String zipPath = OUT + "/打包附件.zip";
        // 获取文件压缩包输出流
        try (OutputStream outputStream = new FileOutputStream(zipPath);
             CheckedOutputStream checkedOutputStream = new CheckedOutputStream(outputStream, new Adler32());
             ZipOutputStream zipOut = new ZipOutputStream(checkedOutputStream)) {
            for (File file : fileList) {
                // 获取文件输入流
                InputStream fileIn = new FileInputStream(file);
                // 使用 common.io中的IOUtils获取文件字节数组
                byte[] bytes = IOUtils.toByteArray(fileIn);
                // 写入数据并刷新
                zipOut.putNextEntry(new ZipEntry(file.getName()));
                zipOut.write(bytes, 0, bytes.length);
                zipOut.flush();
            }
        } catch (FileNotFoundException e) {
            System.out.println("文件未找到");
        } catch (IOException e) {
            System.out.println("读取文件异常");
        }
    }
```

可以看到在 **try()** 的括号中定义需要关闭的资源，实际上这是Java的一种语法糖，查看编译后的代码就知道编译器为我们做了什么，下面是反编译后的代码：

```
public static void zipFile(List<File> fileList) {
        String zipPath = "./打包附件.zip";

        try {
            OutputStream outputStream = new FileOutputStream(zipPath);
            Throwable var3 = null;

            try {
                CheckedOutputStream checkedOutputStream = new CheckedOutputStream(outputStream, new Adler32());
                Throwable var5 = null;

                try {
                    ZipOutputStream zipOut = new ZipOutputStream(checkedOutputStream);
                    Throwable var7 = null;

                    try {
                        Iterator var8 = fileList.iterator();

                        while(var8.hasNext()) {
                            File file = (File)var8.next();
                            InputStream fileIn = new FileInputStream(file);
                            byte[] bytes = IOUtils.toByteArray(fileIn);
                            zipOut.putNextEntry(new ZipEntry(file.getName()));
                            zipOut.write(bytes, 0, bytes.length);
                            zipOut.flush();
                        }
                    } catch (Throwable var60) {
                        var7 = var60;
                        throw var60;
                    } finally {
                        if (zipOut != null) {
                            if (var7 != null) {
                                try {
                                    zipOut.close();
                                } catch (Throwable var59) {
                                    var7.addSuppressed(var59);
                                }
                            } else {
                                zipOut.close();
                            }
                        }

                    }
                } catch (Throwable var62) {
                    var5 = var62;
                    throw var62;
                } finally {
                    if (checkedOutputStream != null) {
                        if (var5 != null) {
                            try {
                                checkedOutputStream.close();
                            } catch (Throwable var58) {
                                var5.addSuppressed(var58);
                            }
                        } else {
                            checkedOutputStream.close();
                        }
                    }

                }
            } catch (Throwable var64) {
                var3 = var64;
                throw var64;
            } finally {
                if (outputStream != null) {
                    if (var3 != null) {
                        try {
                            outputStream.close();
                        } catch (Throwable var57) {
                            var3.addSuppressed(var57);
                        }
                    } else {
                        outputStream.close();
                    }
                }

            }
        } catch (FileNotFoundException var66) {
            System.out.println("文件未找到");
        } catch (IOException var67) {
            System.out.println("读取文件异常");
        }

    }
```

JDK1.7开始，java引入了 try-with-resources 声明，将 try-catch-finally 简化为 try-catch，在编译时会进行转化为 try-catch-finally 语句，我们就不需要在 finally 块中手动关闭资源。

try-with-resources 声明包含三部分：**try(声明需要关闭的资源)、try 块、catch 块**。它要求在 try-with-resources 声明中定义的变量实现了 AutoCloseable 接口，这样在系统可以自动调用它们的close方法，从而替代了finally中关闭资源的功能，编译器为我们生成的异常处理过程如下：

- try 块没有发生异常时，自动调用 close 方法，
- try 块发生异常，然后自动调用 close 方法，如果 close 也发生异常，catch 块只会捕捉 try 块抛出的异常，close 方法的异常会在catch 中通过调用 Throwable.addSuppressed 来压制异常，但是你可以在catch块中，用 Throwable.getSuppressed 方法来获取到压制异常的数组。

## 三、java 异常处理不规范案例

异常处理分为三个阶段：捕获->传递->处理。try……catch的作用是捕获异常，throw的作用将异常传递给合适的处理程序。捕获、传递、处理，三个阶段，任何一个阶段处理不当，都会影响到整个系统。下面分别介绍一下常见的异常处理不规范案例。

### （一）捕获

- 捕获异常的时候不区分异常类型
- 捕获异常不完全，比如该捕获的异常类型没有捕获到

```
try{
    ……
} catch (Exception e){ // 不应对所有类型的异常统一捕获，应该抽象出业务异常和系统异常，分别捕获
    ……
}
```

### 传递

- 异常信息丢失
- 异常信息转译错误，比如在抛出异常的时候将业务异常包装成了系统异常
- 吃掉异常
- 不必要的异常包装
- 检查异常传递过程中不适用非检查检异常包装，造成代码被throws污染

```
try{
    ……
} catch (BIZException e){ 
    throw new BIZException(e); // 重复包装同样类型的异常信息 
} catch (Biz1Exception e){ 
    throw new BIZException(e.getMessage()); // 没有抛出异常栈信息，正确的做法是throw new BIZException(e); 
} catch (Biz2Exception e){
    throw new Exception(e); // 不能使用低抽象级别的异常去包装高抽象级别的异常，这样在传递过程中丢失了异常类型信息
} catch (Biz3Exception e){
    throw new Exception(……); // 异常转译错误，将业务异常直接转译成了系统异常
} catch (Biz4Exception e){
    …… // 不抛出也不记Log，直接吃掉异常
} catch (Exception e){
    throw e;
}
```

### 处理

- 重复处理
- 处理方式不统一
- 处理位置分散

```
try{
    try{
        try{
            ……
        } catch (Biz1Exception e){
            log.error(e);  // 重复的LOG记录
            throw new e;
        }
        
        try{
            ……
        } catch (Biz2Exception e){
            ……  // 同样是业务异常，既在内层处理，又在外层处理
        }
    } catch (BizException e){
        log.error(e); // 重复的LOG记录
        throw e;
    }
} catch (Exception e){
    // 通吃所有类型的异常
    log.error(e.getMessage(),e);
}
```

## 四、java 异常处理规范案例

### （一）阿里巴巴Java异常处理规约

![图片](Java%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E5%92%8C%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%EF%BC%88%E5%90%AB%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90%EF%BC%89%20.resource/640-20230320151016852.png)

![图片](Java%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E5%92%8C%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%EF%BC%88%E5%90%AB%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90%EF%BC%89%20.resource/640-20230320151016935.png)

![图片](Java%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E5%92%8C%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%EF%BC%88%E5%90%AB%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90%EF%BC%89%20.resource/640-20230320151017016.png)

阿里巴巴Java开发规范中有15条异常处理的规约，其中下面两条使用的时候是比较困惑的，因为并没有告诉我们应该如何定义异常，如何抛出异常，如何处理异常：

- **【强制】捕获异常是为了处理它，不要捕获了却什么都不处理而抛弃之，如果不想处理它，请将该异常抛给它的调用者。最外层的业务使用者，必须处理异常，将其转化为用户可以理解的内容。**
- **【推荐】定义时区分unchecked / checked 异常，避免直接使用RuntimeException抛出，更不允许抛出Exception或者Throwable，应使用有业务含义的自定义异常。**

后面的章节我将根据自己的思考，说明如何定义异常，如何抛出异常，如何处理异常，接着往下看。

### （二）异常处理最佳实践

1、使用 try-with-resource 关闭资源。

2、抛出具体的异常而不是 Exception，并在注释中使用 @throw 进行说明。

3、捕获异常后使用描述性语言记录错误信息，如果是调用外部服务最好是包括入参和出参。

```
 logger.error("说明信息，异常信息：{}", e.getMessage(), e)
```

4、优先捕获具体异常。

5、不要捕获 Throwable 异常，除非特殊情况。

6、不要忽略异常，异常捕获一定需要处理。

7、不要同时记录和抛出异常，因为异常会打印多次，正确的处理方式要么抛出异常要么记录异常，如果抛出异常，不要原封不动的抛出，可以自定义异常抛出。

8、自定义异常不要丢弃原有异常，应该将原始异常传入自定义异常中。

```
throw MyException("my exception", e);
```

9、自定义异常尽量不要使用检查异常。

10、尽可能晚的捕获异常，如非必要，建议所有的异常都不要在下层捕获，而应该由最上层捕获并统一处理这些异常。。

11、为了避免重复输出异常日志，建议所有的异常日志都统一交由最上层输出。就算下层捕获到了某个异常，如非特殊情况，也不要将异常信息输出，应该交给最上层统一输出日志。

## 五、项目中的异常处理实践

### （一）如何自定义异常

在介绍如何自定义异常之前，有必要说明一下使用异常的好处，参考Java异常的官方文档，总结有如下好处：

- **能够将错误代码和正常代码分离**
- **能够在调用堆栈上传递异常**
- **能够将异常分组和区分**

在Java异常体系中定义了很多的异常，这些异常通常都是技术层面的异常，对于应用程序来说更多出现的是业务相关的异常，比如用户输入了一些不合法的参数，用户没有登录等，我们可以通过异常来对不同的业务问题进行分类，以便我们排查问题，所以需要自定义异常。那我们如何自定义异常呢？前面已经说了，在应用程序中尽量不要定义检查异常，应该定义非检查异常（运行时异常）。

在我看来，应用程序中定义的异常应该分为两类：

- 业务异常：用户能够看懂并且能够处理的异常，比如用户没有登录，提示用户登录即可。
- 系统异常：用户看不懂需要程序员处理的异常，比如网络连接超时，需要程序员排查相关问题。

下面是我设想的对于应用程序中的异常体系分类：

![图片](Java%E5%BC%82%E5%B8%B8%E5%A4%84%E7%90%86%E5%92%8C%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5%EF%BC%88%E5%90%AB%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90%EF%BC%89%20.resource/640-20230320151016809.jpeg)

在真实项目中，我们通常在遇到不符合预期的情况下，通过抛出异常来阻止程序继续运行，在抛出对应的异常时，需要在异常对象中描述抛出该异常的原因以及异常堆栈信息，以便提示用户和开发人员如何处理该异常。

一般来说，异常的定义我们可以参考Java的其他异常定义就可以了，比如异常中有哪些构造方法，方法中有哪些构造参数，但是这样的自定义异常只是通过异常的类名对异常进行了一个分类，对于异常的描述信息还是不够完善，因为异常的描述信息只是一个字符串。我觉得异常的描述信息还应该包含一个错误码（code）,异常中包含错误码的好处是什么呢？我能想到的就是和http请求中的状态码的优点差不多，还有一点就是能够方便提供翻译功能，对于不同的语言环境能够通过错误码找到对应语言的错误提示信息而不需要修改代码。

基于上述的说明，我认为应该这样来定义异常类，需要定义一个描述异常信息的枚举类，对于一些通用的异常信息可以在枚举中定义，如下所示：

```
/**
 * 异常信息枚举类
 *
 */
public enum ErrorCode {
    /**
     * 系统异常
     */
    SYSTEM_ERROR("A000", "系统异常"),
    /**
     * 业务异常
     */
    BIZ_ERROR("B000", "业务异常"),
    /**
     * 没有权限
     */
    NO_PERMISSION("B001", "没有权限"),

    ;
    /**
     * 错误码
     */
    private String code;
    /**
     * 错误信息
     */
    private String message;

    ErrorCode(String code, String message) {
        this.code = code;
        this.message = message;
    }

    /**
     * 获取错误码
     *
     * @return 错误码
     */
    public String getCode() {
        return code;
    }

    /**
     * 获取错误信息
     *
     * @return 错误信息
     */
    public String getMessage() {
        return message;
    }

    /**
     * 设置错误码
     *
     * @param code 错误码
     * @return 返回当前枚举
     */
    public ErrorCode setCode(String code) {
        this.code = code;
        return this;
    }

    /**
     * 设置错误信息
     *
     * @param message 错误信息
     * @return 返回当前枚举
     */
    public ErrorCode setMessage(String message) {
        this.message = message;
        return this;
    }

}
```

自定义系统异常类，其他类型的异常类似，只是异常的类名不同，如下代码所示：

```
/**
 * 系统异常类
 *
 */
public class SystemException extends RuntimeException {


    private static final long serialVersionUID = 8312907182931723379L;
  /**
     * 错误码
     */
    private String code;

 

    /**
     * 构造一个没有错误信息的 <code>SystemException</code>
     */
    public SystemException() {
        super();
    }


    /**
     * 使用指定的 Throwable 和 Throwable.toString() 作为异常信息来构造 SystemException
     *
     * @param cause 错误原因， 通过 Throwable.getCause() 方法可以获取传入的 cause信息
     */
    public SystemException(Throwable cause) {
        super(cause);
    }

    /**
     * 使用错误信息 message 构造 SystemException
     *
     * @param message 错误信息
     */
    public SystemException(String message) {
        super(message);
    }

    /**
     * 使用错误码和错误信息构造 SystemException
     *
     * @param code    错误码
     * @param message 错误信息
     */
    public SystemException(String code, String message) {
        super(message);
        this.code = code;
    }

    /**
     * 使用错误信息和 Throwable 构造 SystemException
     *
     * @param message 错误信息
     * @param cause   错误原因
     */
    public SystemException(String message, Throwable cause) {
        super(message, cause);
    }

    /**
     * @param code    错误码
     * @param message 错误信息
     * @param cause   错误原因
     */
    public SystemException(String code, String message, Throwable cause) {
        super(message, cause);
        this.code = code;
    }

    /**
     * @param errorCode ErrorCode
     */
    public SystemException(ErrorCode errorCode) {
        super(errorCode.getMessage());
        this.code = errorCode.getCode();
    }

    /**
     * @param errorCode ErrorCode
     * @param cause     错误原因
     */
    public SystemException(ErrorCode errorCode, Throwable cause) {
        super(errorCode.getMessage(), cause);
        this.code = errorCode.getCode();
    }

    /**
     * 获取错误码
     *
     * @return 错误码
     */
    public String getCode() {
        return code;
    }
}
```

上面定义的 SystemException 类中定义了很多的构造方法，我这里只是给出一个示例，所以保留了不传入错误码的构造方法，建议保留不使用错误码的构造方法，可以提高代码的灵活性，因为错误码的规范也是一个值得讨论的问题，关于如何定义错误码在阿里巴巴开发规范手册中有介绍，这里不再详细说明。

### （二）如何使用异常

前面介绍了如何自定义异常，接下来介绍一下如何使用异常，也就是什么时候抛出异常。**异常其实可以看作方法的返回结果**，当出现非预期的情况时，就可以通过抛出异常来阻止程序继续执行。比如期望用户有管理员权限才能删除某条记录，如果用户没有管理员权限，那么就可以抛出没有权限的异常阻止程序继续执行并提示用户需要管理员权限才能操作。

抛出异常使用 throw 关键字，如下所示：

```
throw new BizException(ErrorCode.NO_PERMISSION);
```

什么时候抛出业务异常，什么时候抛出系统异常？

**业务异常（bizException/bussessException）**： 用户操作业务时，提示出来的异常信息，这些信息能直接让用户可以继续下一步操作，或者换一个正确操作方式去使用，换句话就是用户可以自己能解决的。比如：“用户没有登录”，“没有权限操作”。

**系统异常（SystemException）**： 用户操作业务时，提示系统程序的异常信息，这类的异常信息时用户看不懂的，需要告警通知程序员排查对应的问题，如 NullPointerException，IndexOfException。另一个情况就是接口对接时，参数的校验时提示出来的信息，如：缺少ID，缺少必须的参数等，这类的信息对于客户来说也是看不懂的，也是解决不了的，所以我把这两类的错误应当统一归类于系统异常

关于应该抛出业务异常还是系统异常，一句话总结就是：该异常用户能否处理，如果用户能处理则抛出业务异常，如果用户不能处理需要程序员处理则抛出系统异常。

在调用第三方的 rpc 接口时，我们应该如何处理异常呢？首先我们需要知道 rpc 接口抛出异常还是返回的包含错误码的 Result 对象，关于 rpc 应该返回异常还是错误码有很多的讨论，关于这方面的内容可以查看相关文档，这个不是本文的重点，通过实际观察知道 rpc 的返回基本都是包含错误码的 Result 对象，所以这里以返回错误码的情况进行说明。首先需要明确 rpc 调用失败应该返回系统异常，所以我们可以定义一个继承 SystemException 的 rpc 异常 RpcException，代码如下所示：

```java
/**
 * rpc 异常类
 */
public class RpcException extends SystemException {


    private static final long serialVersionUID = -9152774952913597366L;

    /**
     * 构造一个没有错误信息的 <code>RpcException</code>
     */
    public RpcException() {
        super();
    }


    /**
     * 使用指定的 Throwable 和 Throwable.toString() 作为异常信息来构造 RpcException
     *
     * @param cause 错误原因， 通过 Throwable.getCause() 方法可以获取传入的 cause信息
     */
    public RpcException(Throwable cause) {
        super(cause);
    }

    /**
     * 使用错误信息 message 构造 RpcException
     *
     * @param message 错误信息
     */
    public RpcException(String message) {
        super(message);
    }

    /**
     * 使用错误码和错误信息构造 RpcException
     *
     * @param code    错误码
     * @param message 错误信息
     */
    public RpcException(String code, String message) {
        super(code, message);
    }

    /**
     * 使用错误信息和 Throwable 构造 RpcException
     *
     * @param message 错误信息
     * @param cause   错误原因
     */
    public RpcException(String message, Throwable cause) {
        super(message, cause);
    }

    /**
     * @param code    错误码
     * @param message 错误信息
     * @param cause   错误原因
     */
    public RpcException(String code, String message, Throwable cause) {
        super(code, message, cause);
    }

    /**
     * @param errorCode ErrorCode
     */
    public RpcException(ErrorCode errorCode) {
        super(errorCode);
    }

    /**
     * @param errorCode ErrorCode
     * @param cause     错误原因
     */
    public RpcException(ErrorCode errorCode, Throwable cause) {
        super(errorCode, cause);
    }

}
```

这个 RpcException 所有的构造方法都是调用的父类 SystemExcepion 的方法，所以这里不再赘述。定义好了异常后接下来是处理 rpc 调用的异常处理逻辑，调用 rpc 服务可能会发生 ConnectException 等网络异常，我们并不需要在调用的时候捕获异常，而是应该在最上层捕获并处理异常，调用 rpc 的处理demo代码如下：

```java
private Object callRpc() {
    Result<Object> rpc = rpcDemo.rpc();
    log.info("调用第三方rpc返回结果为：{}", rpc);
    if (Objects.isNull(rpc)) {
        return null;
    }
    if (!rpc.getSuccess()) {
        throw new RpcException(ErrorCode.RPC_ERROR.setMessage(rpc.getMessage()));
    }
    return rpc.getData();
}
```

### （三）如何处理异常

我们应该尽可能晚的捕获异常，如非必要，建议所有的异常都不要在下层捕获，而应该由最上层捕获并统一处理这些异常。前面的已经简单说明了一下如何处理异常，接下来将通过代码的方式讲解如何处理异常。

#### 1.rpc 接口全局异常处理

对于 rpc 接口，我们这里将 rpc 接口的返回结果封装到包含错误码的 Result 对象中，所以可以定义一个 aop 叫做 RpcGlobalExceptionAop，在 rpc 接口执行前后捕获异常，并将捕获的异常信息封装到 Result 对象中返回给调用者。

Result 对象的定义如下：

```java
/**
 * Result 结果类
 *
 */
public class Result<T> implements Serializable {

    private static final long serialVersionUID = -1525914055479353120L;
    /**
     * 错误码
     */
    private final String code;
    /**
     * 提示信息
     */
    private final String message;
    /**
     * 返回数据
     */
    private final T data;
    /**
     * 是否成功
     */
    private final Boolean success;

    /**
     * 构造方法
     *
     * @param code    错误码
     * @param message 提示信息
     * @param data    返回的数据
     * @param success 是否成功
     */
    public Result(String code, String message, T data, Boolean success) {
        this.code = code;
        this.message = message;
        this.data = data;
        this.success = success;
    }

    /**
     * 创建 Result 对象
     *
     * @param code    错误码
     * @param message 提示信息
     * @param data    返回的数据
     * @param success 是否成功
     */
    public static <T> Result<T> of(String code, String message, T data, Boolean success) {
        return new Result<>(code, message, data, success);
    }

    /**
     * 成功，没有返回数据
     *
     * @param <T> 范型参数
     * @return Result
     */
    public static <T> Result<T> success() {
        return of("00000", "成功", null, true);
    }

    /**
     * 成功，有返回数据
     *
     * @param data 返回数据
     * @param <T>  范型参数
     * @return Result
     */
    public static <T> Result<T> success(T data) {
        return of("00000", "成功", data, true);
    }

    /**
     * 失败，有错误信息
     *
     * @param message 错误信息
     * @param <T>     范型参数
     * @return Result
     */
    public static <T> Result<T> fail(String message) {
        return of("10000", message, null, false);
    }

    /**
     * 失败，有错误码和错误信息
     *
     * @param code    错误码
     * @param message 错误信息
     * @param <T>     范型参数
     * @return Result
     */
    public static <T> Result<T> fail(String code, String message) {
        return of(code, message, null, false);
    }


    /**
     * 获取错误码
     *
     * @return 错误码
     */
    public String getCode() {
        return code;
    }

    /**
     * 获取提示信息
     *
     * @return 提示信息
     */
    public String getMessage() {
        return message;
    }

    /**
     * 获取数据
     *
     * @return 返回的数据
     */
    public T getData() {
        return data;
    }

    /**
     * 获取是否成功
     *
     * @return 是否成功
     */
    public Boolean getSuccess() {
        return success;
    }
}
```

在编写 aop 代码之前需要先导入 spring-boot-starter-aop 依赖：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

RpcGlobalExceptionAop 代码如下：

```java
/**
 * rpc 调用全局异常处理 aop 类
 *
 */
@Slf4j
@Aspect
@Component
public class RpcGlobalExceptionAop {
    /**
     * execution(* com.xyz.service ..*.*(..))：表示 rpc 接口实现类包中的所有方法
     */
    @Pointcut("execution(* com.xyz.service ..*.*(..))")
    public void pointcut() {}

    @Around(value = "pointcut()")
    public Object handleException(ProceedingJoinPoint joinPoint) {
        try {
            //如果对传入对参数有修改，那么需要调用joinPoint.proceed(Object[] args)
            //这里没有修改参数，则调用joinPoint.proceed()方法即可
            return joinPoint.proceed();
        } catch (BizException e) {
            // 对于业务异常，应该记录 warn 日志即可，避免无效告警
            log.warn("全局捕获业务异常", e);
            return Result.fail(e.getCode(), e.getMessage());
        } catch (RpcException e) {
            log.error("全局捕获第三方rpc调用异常", e);
            return Result.fail(e.getCode(), e.getMessage());
        } catch (SystemException e) {
            log.error("全局捕获系统异常", e);
            return Result.fail(e.getCode(), e.getMessage());
        } catch (Throwable e) {
            log.error("全局捕获未知异常", e);
            return Result.fail(e.getMessage());
        }
    }

}
```

aop 中 @Pointcut 的 execution 表达式配置说明：

```
execution(public * *(..)) 定义任意公共方法的执行
execution(* set*(..)) 定义任何一个以"set"开始的方法的执行
execution(* com.xyz.service.AccountService.*(..)) 定义AccountService 接口的任意方法的执行
execution(* com.xyz.service.*.*(..)) 定义在service包里的任意方法的执行
execution(* com.xyz.service ..*.*(..)) 定义在service包和所有子包里的任意类的任意方法的执行
execution(* com.test.spring.aop.pointcutexp…JoinPointObjP2.*(…)) 定义在pointcutexp包和所有子包里的JoinPointObjP2类的任意方法的执行
```

#### 2.http 接口全局异常处理

如果是 springboot 项目，http 接口的异常处理主要分为三类：

- 基于请求转发的方式处理异常；
- 基于异常处理器的方式处理异常；
- 基于过滤器的方式处理异常。

**基于请求转发的方式：**真正的全局异常处理。

实现方式有：

- BasicExceptionController

**基于异常处理器的方式：**不是真正的全局异常处理，因为它处理不了过滤器等抛出的异常。

实现方式有：

- @ExceptionHandler
- @ControllerAdvice+@ExceptionHandler
- SimpleMappingExceptionResolver
- HandlerExceptionResolver

**基于过滤器的方式****：**近似全局异常处理。它能处理过滤器及之后的环节抛出的异常。

实现方式有：

- Filter

关于 http 接口的全局异常处理，这里重点介绍**基于异常处理器的方式**，其余的方式建议查阅相关文档学习。

在介绍基于异常处理器的方式之前需要导入 spring-boot-starter-web 依赖即可，如下所示：

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

通过 @ControllerAdvice+@ExceptionHandler 实现**基于异常处理器的http接口全局异常处理：**

```
/**
* http 接口异常处理类
*/
@Slf4j
@RestControllerAdvice("org.example.controller")
public class HttpExceptionHandler {

    /**
     * 处理业务异常
     * @param request 请求参数
     * @param e 异常
     * @return Result
     */
    @ExceptionHandler(value = BizException.class)
    public Object bizExceptionHandler(HttpServletRequest request, BizException e) {
        log.warn("业务异常：" + e.getMessage() , e);
        return Result.fail(e.getCode(), e.getMessage());
    }

    /**
     * 处理系统异常
     * @param request 请求参数
     * @param e 异常
     * @return Result
     */
    @ExceptionHandler(value = SystemException.class)
    public Object systemExceptionHandler(HttpServletRequest request, SystemException e) {
        log.error("系统异常：" + e.getMessage() , e);
        return Result.fail(e.getCode(), e.getMessage());
    }

    /**
     * 处理未知异常
     * @param request 请求参数
     * @param e 异常
     * @return Result
     */
    @ExceptionHandler(value = Throwable.class)
    public Object unknownExceptionHandler(HttpServletRequest request, Throwable e) {
        log.error("未知异常：" + e.getMessage() , e);
        return Result.fail(e.getMessage());
    }

}
```

在 HttpExceptionHandler 类中，@RestControllerAdvice = @ControllerAdvice + @ResponseBody ，如果有其他的异常需要处理，只需要定义@ExceptionHandler注解的方法处理即可。

## 六、总结

读完本文应该了解Java异常处理机制，当一个异常被抛出时，JVM会在当前的方法里寻找一个匹配的处理，如果没有找到，这个方法会强制结束并弹出当前栈帧，并且异常会重新抛给上层调用的方法（在调用方法帧）。如果在所有帧弹出前仍然没有找到合适的异常处理，这个线程将终止。如果这个异常在最后一个非守护线程里抛出，将会导致JVM自己终止，比如这个线程是个main线程。

最后对本文的内容做一个简单的总结，Java语言的异常处理方式有两种，一种是 try-catch 捕获异常，另一种是通过 throw 抛出异常。在程序中可以抛出两种类型的异常，一种是检查异常，另一种是非检查异常，应该尽量抛出非检查异常，遇到检查异常应该捕获进行处理不要抛给上层。在异常处理的时候应该尽可能晚的处理异常，最好是定义一个全局异常处理器，在全局异常处理器中处理所有抛出的异常，并将异常信息封装到 Result 对象中返回给调用者。

**参考文档：**

http://javainsimpleway.com/exception-handling-best-practices/

https://www.infoq.com/presentations/effective-api-design/

https://docs.oracle.com/javase/tutorial/essential/exceptions/advantages.html

java 官方文档：https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-3.html#jvms-3.13