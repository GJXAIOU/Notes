[TOC]

# 线程基本类 Thread 类与 Runnable 接口

在进入多线程学习之前，首先分析一下 Thread 类、Object 类中有关线程的基本方法。

> JDK 版本为：1.8.0_221
>
> Object 类：java.lang.Object
>
> Thread 类：java.lang.Thread
>
> Runnable 接口：java.lang.Runnable
>
> 文中涉及的翻译均为个人理解翻译，偏向于口语化，若想深究推荐阅读源码英文说明。
>
> 慢工出细活，不要急~

## 一、Thread 类

### （一）Thread  类和 Runnable 接口两者关系

从 UML 图中可以明显看出，Thread 类是继承自 Runnable 接口。从源码层面也能得到这一结论：

```java
public class Thread implements Runnable {
    // XXXX
}
```

![image-20210328193247582](一、线程基本类.resource/image-20210328193247582.png)

### （二）Thread 类的 JavaDoc 文档阅读

首先完整的阅读分析一下 Thread 类的 JavaDoc 文档。【】内为自己注释文字，非原文翻译。

> 一个 Thread 是程序的一个执行线程，Java 虚拟机允许应用程序同时并发运行多个执行线程。
>
> **每个线程都有一个优先级，优先级较高的线程优先于优先级较低的线程执行**。每个线程也可以/不可以被标记为守护进程。当在某个线程中运行的代码创建一个新的 Thread 对象时，**新线程的优先级最初设置为等于创建线程的优先级**【即如果在一个线程中创建另一个线程，则被创建线程初始优先级和创建它的线程优先级相同】，**并且仅当创建线程是守护进程时，被创建的线程才是守护进程线程**。
>
> 当 Java 虚拟机启动时，通常只有一个非守护进程线程（它通常调用某些指定类的名为 main 的方法）【所以 main 方法是执行在线程上的】。Java 虚拟机会一直执行线程，直到发生以下任一情况：
>
> - Runtime 类的 exit() 方法被调用，并且类安全管理器允许退出操作发生；
>
> - 所有非守护进程的线程都已死亡，消亡原因可能是调用 run 方法返回了，或者抛出了超过 run 方法范围的异常。
>
> 有两种方法可以创建一个新的执行线程，**一种是将类声明为 Thread 类的子类。这个子类应该覆盖重写Thread 类的 run 方法**。然后可以分配并启动子类的实例。例如，计算大于指定值的素数的线程可以如下编写：
>
> ```java
> class PrimeThread extends Thread {
>    long minPrime;
>    PrimeThread(long minPrime) {
>        this.minPrime = minPrime;
>    }
>    public void run() {
>        // compute primes larger than minPrime
>        ...
>    }
> }
> ```
>
> 然后通过如下代码可以创建一个线程然后开始运行：
>
> ```java
> PrimeThread p = new PrimeThread(143);
> p.start();
> ```
>
> **创建线程的另一种方式是声明一个实现  Runnable 接口的类。然后，该类实现 run 方法。**然后可以分配类的实例【即可以创建该类的实例】，**在创建 Thread 时将该实例对象作为参数传递**，然后启动。其他样式中的相同示例如下所示：
>
> ```java
> class PrimeRun implements Runnable {
>     long minPrime;
>     PrimeRun(long minPrime) {
>         this.minPrime = minPrime;
>     }
> 
>     public void run() {
>         // compute primes larger than minPrime
>         ......
>     }
> }
> ```
>
> 然后创建线程对象，启动
>
> ```java
> PrimeRun p = new PrimeRun(143);
> new Thread(p).start();
> ```
>
> **每个线程都有一个用于标识的名称。多个线程可能具有相同的名称。如果在创建线程时未指定名称，则会为其生成新名称。**

**Thread 类的 JavaDoc 文档总结**：

- 每个线程都有一个优先级，高优先级的线程优于低优先级的线程执行，**新创建线程的优先级等同于创建该线程的线程优先级。**

- 只有创建线程为守护线程，其创建的线程才是守护线程。

    > 当进程中不存在非守护线程时，守护线程自动销毁（如垃圾回收线程）。

- 创建线程有两种方式：
    - 继承 Thread 类，然后重写 run 方法；
    - 实现 Runnable 接口，然后实现 run 方法；
    
- 每个线程在创建时都可以设置名称，如未指定则会自动设置。并且多个线程可能具有相同的名称。

2.**然后在 Thread 类的代码中同样含有设置了线程优先级代码，代码如下：**

```java
/**
* The minimum priority that a thread can have.
* 一个线程可以使用的最小优先级
*/
public final static int MIN_PRIORITY = 1;

/**
* The default priority that is assigned to a thread.
* 分配给一个线程的默认优先级
*/
public final static int NORM_PRIORITY = 5;

/**
* The maximum priority that a thread can have.
* 一个线程可以使用的最大优先级
*/
public final static int MAX_PRIORITY = 10;
```

3.**被创建线程的优先级和创建该线程的线程优先级相同**

```java
package com.gjxaiou;

public class Demo extends Thread {
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName());
        System.out.println(Thread.currentThread().getPriority());
    }

    public static void main(String[] args) {
        System.out.println("main 方法中的线程 " + Thread.currentThread().getName());
        Thread.currentThread().setPriority(3);
        Demo demo = new Demo();
        demo.start();
    }
}
```

### （三）Thread 类的构造方法分析

Thread 类包括 8 个重载的构造方法，然后都调用 init 方法来实现线程的创建和初始化，示例如：

```java
public Thread() {
    init(null, null, "Thread-" + nextThreadNum(), 0);
}
public Thread(Runnable target) {
    init(null, target, "Thread-" + nextThreadNum(), 0);
}
```

其中 init 方法参数含义为：

```java
/**
  * @param g ：线程组
  * @param target ：被执行的目标
  * @param name ：线程名称
  * @param stackSize ：新线程期待栈的大小，或者值为 0， 0 表示该参数会被忽略。
  * @param acc ：访问控制
  */
private void init(ThreadGroup g, Runnable target, String name,
                  long stackSize, AccessControlContext acc) {
}
```

针对上述谈到的线程自动命名，在上述的 Thread 类的构造方法中，线程被命名为：`Thread- + nextThreadNum`，其中 nextThreadNum 为 Thread 类中的一个同步方法，源代码如下：

```java
/*
 * For autonumbering anonymous threads.
 * 用于自动为匿名线程编号
 */
private static int threadInitNumber;
private static synchronized int nextThreadNum() {
    return threadInitNumber++;
}
```

该方法主要功能就是将变量 threadInitNumber 加一，使用 synchronized 修饰该方法使得同一个时刻只能有一个线程进入该方法。而该变量使用 static 方法修饰，是为了保证所有的 Thread 类实例都能看到并修改的是同一个变量。其中 synchronized 使用后续会详细分析。

### （四）Thread 类的 start 方法分析

**1.首先阅读 JavaDoc 文档**

>**该方法会导致这个线程开始执行，Java 虚拟机会调用这个线程的 run() 方法。**
>
>执行的结果就是两个线程同时执行：当前线程（调用 start 方法返回的线程）和其他线程（执行其 run 方法的线程）。
>
>【因为调用 start 方法肯定是通过某个线程的对象来调用 start 方法，这是一个线程。 =》调用 start 方法的线程】
>
>【当调用 start 方法之后，JVM 又会调用该线程的 run 方法，这是在另一个线程中执行。=》调用 run 方法的线程】
>
>启动一个线程多次是不合法的，特别的，如果一个线程执行完成之后不会再被重启。

然后针对该方法进行源码分析：

```java
/*
 * Java 线程状态，已初始化以显示线程「尚未启动」
 */
// 这里 volatile 主要保证该变量的可见性。
private volatile int threadStatus = 0; 

public synchronized void start() {
    /**
     * 对于 VM（虚拟机）创建/设置的 main 方法或者 system 组线程，不会调用该方法，将来添加到该方法的任何新功能可能也必须添加到 VM 中。
     * 状态值 0 对应状态 「NEW」
     */
    // 如果该线程已经被启动了（变量值不为 0），则直接抛出 IllegalThreadStateException
    if (threadStatus != 0)
        throw new IllegalThreadStateException();

    /*
     *  通知线程组该线程即将启动，以便可以将其添加到组的线程列表中，并且可以减少组的未启动计数。
     */
    group.add(this);

    // started 表示线程的运行状态
    boolean started = false;
    try {
        // start0() 为 Native 方法，因此线程本质上是由操作系统创建执行的。源代码见：https://github.com/GJXAIOU/jdk/blob/master/src/hotspot/share/runtime/thread.cpp
        start0();
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {
            /* do nothing. If start0 threw a Throwable then
                  it will be passed up the call stack */
        }
    }
}
```

### （五）Thread 类的 run  方法分析

**首先分析 run 方法的 JavaDoc 文档**：

>如果该线程是由一个单独的 Runnable 对象来构建，将会调用 Runnable 对象中的 run 方法，构造该方法不做任何事并且方法。**Thread 类的子类应该重写这个方法。**

源码分析为：

```java
@Override
public void run() {
    // target 就是一个 Runnable 对象
    if (target != null) {
        // 调用传入的 Runnable 对象的 run 方法
        target.run();
    }
}
```

**执行 start() 方法的顺序并不代表执行 run() 的顺序**

```java
package com.gjxaiou;

public class Demo2  extends Thread{
	private int  i;
	Demo2(int i){
		this.i = i;
	}
	@Override
	public void run() {
		System.out.println(i);
	}

	public static void main(String[] args) {
		new Demo2(1).start();
		new Demo2(3).start();
		new Demo2(2).start();
	}
}
// output：
1
3
2
```

**执行 start() 方法和执行 run() 方法区别**

```java
package com.gjxaiou.thread;

public class MyTest1 {
	public static void main(String[] args) {
		MyThread myThread = new MyThread();
		myThread.setName("新创建的线程 Thread - 0 ");
		myThread.start();
		// myThread.run();
	}
}

class MyThread extends Thread {
	String threadName;

	MyThread() {
		System.out.println("执行构造方法线程：" + Thread.currentThread().getName());
	}

	@Override
	public void run() {
		System.out.println("执行 run 方法线程： " + Thread.currentThread().getName());
	}

	public String getThreadName() {
		return this.threadName;
	}

	public void setThreadName(String threadName) {
		this.threadName = threadName;
	}
}
```

main 方法中执行 `myThread.start()` 和 `myThread.run()` 的结果分别为：

```java
// 执行 myThread.start() 方法时候输出结果。启动新的线程执行 run() 方法，同时执行 run() 方法时机不确定。
执行构造方法线程：main
执行 run 方法线程： 新创建的线程 Thread - 0 

// 执行 myThread.run() 方法输出结果。不启动新的线程，立即执行 run() 方法。
执行构造方法线程：main
执行 run 方法线程： main    
```

### （六）Thread 类的 isAlive()  方法

用于判断当前线程是否存活（即处于活动状态），活动状态指线程已经启动并且尚未终止的状态。

示例程序：

```java
package com.gjxaiou.thread;

public class MyTest2 {
    public static void main(String[] args) throws InterruptedException {
        MyThread1 myThread1 = new MyThread1();
        System.out.println("begin：" + myThread1.isAlive());
        myThread1.start();
        // Thread.sleep(2000);
        System.out.println("end：" + myThread1.isAlive());
    }
}

class MyThread1 extends Thread {
    @Override
    public void run() {
        System.out.println("run 方法：" + this.isAlive());
    }
}
```

执行结果：

```java
// 情况一：注释掉 sleep() 时输出的结果
begin：false
end：true
run 方法：true
    
// 情况二：添加 sleep() 时输出的结果
begin：false
run 方法：true
end：false    
```

情况一：线程启动之后， main 线程继续执行，则继续输出 end：true，当前因为 start() 方法还在调用 run 方法，所以线程是存活状态，所以 end 值为 true。

情况二：当主线程创建新的线程并且启动之后，`Thread.sleep()` 会让主线程睡眠，但是新的线程继续执行，等主线程醒来之后新创建的线程已经执行完毕，所以 end:false。

### （七）Thread 类的 sleep 方法详解

**首先分析 JavaDoc 文档**

> 会导致当前正在执行的线程进入休眠状态（临时的停止执行一段特定时间的毫秒数），它会受到系统定时器和调度器的精度的影响。**线程并不会失去任何锁的所有权**。

```java
/* @param  millis： 睡眠的毫秒数
 * @throws  IllegalArgumentException：如果 millis 为负数就抛这个异常
 * @throws  InterruptedException：如果任何其它线程中断了当前线程， 当抛出这个异常中，当前线程的中断状态就被清空了。
  */
public static native void sleep(long millis) throws InterruptedException;

// 这是纳秒级别的重载方法
public static void sleep(long millis, int nanos){
  // XXXXXX 
}
```

注意：这里「正在执行的线程」是指 `this.currentThread()` 返回的线程，同时如果调用 sleep() 方法所在类是  Thread 类，则 `Thread.sleep()` 和 `this.sleep()` 效果等价，如果不在 Thread 类，必须使用 `Thread.sleep()`。



### （八）StackTraceElement[]   getStackTrace() 方法与 static void dumpStack() 方法

`getStackTrace()` 方法返回一个表示该线程堆栈跟踪元素数组。如果该线程尚未启动或者已经终止，则返回一个零长度的数组。否则返回数组的第一个元素代表堆栈顶（数组中最新的方法调用），最后一个元素表示堆栈底（数组中最旧的方法调用）。

`dumpStack()` 方法是将当前线程的堆栈跟踪信息输出至标准错误流，该方法仅用于调试。

```java
package com.gjxaiou.thread;

public class Mytest4 {
	public void a() {
		b();
	}

	public void b() {
		c();
	}

	public void c() {
		d();
	}

	public void d() {
		StackTraceElement[] stackTrace = Thread.currentThread().getStackTrace();
		if (stackTrace != null) {
			for (StackTraceElement stackTraceElement : stackTrace) {
				System.out.println(stackTraceElement.getClassName() + " : " + stackTraceElement.getMethodName() + " " +
						": " + stackTraceElement.getLineNumber());
			}
		}

		System.out.println("当前线程的堆栈跟踪信息为：");
		Thread.dumpStack();
	}

	public static void main(String[] args) {
		Mytest4 mytest4 = new Mytest4();
		mytest4.a();
	}
}
```

输出结果为：

```java
java.lang.Thread : getStackTrace : 1559
com.gjxaiou.thread.Mytest4 : d : 17
com.gjxaiou.thread.Mytest4 : c : 13
com.gjxaiou.thread.Mytest4 : b : 9
com.gjxaiou.thread.Mytest4 : a : 5
com.gjxaiou.thread.Mytest4 : main : 31
当前线程的堆栈跟踪信息为：
java.lang.Exception: Stack trace
	at java.lang.Thread.dumpStack(Thread.java:1336)
	at com.gjxaiou.thread.Mytest4.d(Mytest4.java:26)
	at com.gjxaiou.thread.Mytest4.c(Mytest4.java:13)
	at com.gjxaiou.thread.Mytest4.b(Mytest4.java:9)
	at com.gjxaiou.thread.Mytest4.a(Mytest4.java:5)
	at com.gjxaiou.thread.Mytest4.main(Mytest4.java:31)
```

### （九）static Map<Thread，StackTraceElement[]> getAllStackTraces() 方法

返回所有**活动线程**的堆栈跟踪的一个映射，key 为线程，value 值为 StackTraceElement 数组（即对应 Thread 的堆栈转储）。

在调用该方法同时，线程可能也在执行。每个线程的堆栈跟踪仅代表一个快照，并且每个堆栈跟踪都可以在不同时间获得。没有线程的堆栈跟踪信息则返回空数组。

```java
package com.gjxaiou.thread;

import java.util.Iterator;
import java.util.Map;

public class MyTest5 {
	public static void main(String[] args) {
		Test test = new Test();
		test.a();
	}
}

class Test {
	public void a() {
		b();
	}

	public void b() {
		c();
	}

	public void c() {
		d();
	}

	public void d() {
		Map<Thread, StackTraceElement[]> allStackTraces =
				Thread.currentThread().getAllStackTraces();
		if (allStackTraces != null && allStackTraces.size() != 0) {
			Iterator<Thread> iterator = allStackTraces.keySet().iterator();
			while (iterator.hasNext()) {
				Thread next = iterator.next();
				StackTraceElement[] stackTraceElements = allStackTraces.get(next);
				System.out.println("每个线程的基本信息：" + "线程名称：" + next.getName() + " 线程状态：" + next.getState());
				if (stackTraceElements.length != 0) {
					System.out.println("输出 stackTraceElement[] 数组具体信息：");
					for (StackTraceElement stackTraceElement : stackTraceElements) {
						System.out.println(stackTraceElement.getClassName() + "   " + stackTraceElement.getMethodName() + "  " + stackTraceElement.getLineNumber());
					}
				} else {
					System.out.println("stackTraceElement[] 为空，因为线程" + next.getName() + " 中的 " +
							"StackTraceElement " +
							"数组长度为 0");
				}
			}
		}
	}
}
```

输出结果为：

```java
每个线程的基本信息：线程名称：Attach Listener 线程状态：RUNNABLE
stackTraceElement[] 为空，因为线程Attach Listener 中的 StackTraceElement 数组长度为 0
每个线程的基本信息：线程名称：Monitor Ctrl-Break 线程状态：RUNNABLE
输出 stackTraceElement[] 数组具体信息：
java.net.PlainSocketImpl   <init>  97
java.net.SocksSocketImpl   <init>  55
java.net.Socket   setImpl  503
java.net.Socket   <init>  424
java.net.Socket   <init>  211
com.intellij.rt.execution.application.AppMainV2$1   run  43
每个线程的基本信息：线程名称：Signal Dispatcher 线程状态：RUNNABLE
stackTraceElement[] 为空，因为线程Signal Dispatcher 中的 StackTraceElement 数组长度为 0
每个线程的基本信息：线程名称：Finalizer 线程状态：WAITING
输出 stackTraceElement[] 数组具体信息：
java.lang.Object   wait  -2
java.lang.ref.ReferenceQueue   remove  144
java.lang.ref.ReferenceQueue   remove  165
java.lang.ref.Finalizer$FinalizerThread   run  216
每个线程的基本信息：线程名称：Reference Handler 线程状态：WAITING
输出 stackTraceElement[] 数组具体信息：
java.lang.Object   wait  -2
java.lang.Object   wait  502
java.lang.ref.Reference   tryHandlePending  191
java.lang.ref.Reference$ReferenceHandler   run  153
每个线程的基本信息：线程名称：main 线程状态：RUNNABLE
输出 stackTraceElement[] 数组具体信息：
java.lang.Thread   dumpThreads  -2
java.lang.Thread   getAllStackTraces  1610
com.gjxaiou.thread.Test   d  25
com.gjxaiou.thread.Test   c  21
com.gjxaiou.thread.Test   b  18
com.gjxaiou.thread.Test   a  15
com.gjxaiou.thread.MyTest5   main  9
```

### （十）interrupt、interrupted 和 isInterrupted 方法

首先后面两个是用于判断线程是否为停止状态。

```java
// 用于判断 currentThread() 是否已经中断
public static boolean interrupted() {
    return currentThread().isInterrupted(true);
}

// 用于判断 this 关键字所在类的对象是否已经中断
public boolean isInterrupted() {
    return isInterrupted(false);
}
private native boolean isInterrupted(boolean ClearInterrupted);
```

其中 `isInterrupted()` 用于判断当前线程是否已经中断，当前线程即运行 `this.interrupted()` 方法的线程。

```java
package com.gjxaiou.thread;

public class MyTest7 {
	public static void main(String[] args) throws InterruptedException {
		MyThread7 myThread7 = new MyThread7();
		myThread7.start();
		Thread.sleep(2);
        // 在 myThread7 对象上调用该方法来停止 myThread7 对象所代表的线程
		myThread7.interrupt();
        // 判断 myThread7 对象所代表的线程是否停止
		System.out.println(myThread7.interrupted());
		System.out.println(myThread7.interrupted());
	}
}

class MyThread7 extends Thread {
	@Override
	public void run() {
		int i = 0;
		while (i++ < 1000) {
			System.out.println("自定义线程执行");
		}
	}
}
```

执行结果为：（部分中间结果）

```java
自定义线程执行
自定义线程执行
自定义线程执行
false
false
自定义线程执行
自定义线程执行
```

但是 `interrupted()` 方法是**判断当前线程是否已经中断**，当前线程为 main，但是 main 线程并没有暂停，所以输出两个 false。

同时上面的 `myThread.interrupted()` 是用来判断 currentThread() 是否被中断，等价于 `Thread.interrupted()`，因为在 Thread 类中调用静态 static 方法时，大多数是针对 currentThread() 线程进行操作。

**暂停 main 线程**：

```java
package com.gjxaiou.thread;

public class MyTest8 {
	public static void main(String[] args) {
		Thread.currentThread().interrupt();
		System.out.println(Thread.interrupted());
		System.out.println(Thread.interrupted());
	}
}
```

输出结果为：

```java
true
false
```

第二次返回 false 的原因：因为 `interrupted()` 方法不仅可以判断当前线程是否已经中断，同时还会清除中断状态。即连续调用两次这个方法，因为第一次调用已经清除了其中断状态之后，第二次返回了 false。

**isInterrupted() 方法非 static 方法，作用于调用这个方法的对象**：

将 MyTest7 中的输出语句替换为：

```java
System.out.println(myThread7.isInterrupted());
System.out.println(myThread7.isInterrupted());
```

输出结果为：

```java
自定义线程执行
自定义线程执行
true
true
自定义线程执行
自定义线程执行
```

结果可知：isInterrupted() 没有清除状态标志，所以输出两个都是 true。

**总结**：

- `this.interrupted()`：检测当前线程是否已经是中断状态，执行后具有清除状态标志值为 false 的功能。
- `this.isInterrupted()`：检测线程 Thread 对象是否已经是中断状态，不清除状态标志。

#### 使用 interrupt() 方法中断线程

正确的方式是，如果检测到中断了则抛出异常，然后通过捕捉异常来实现中断。

当然可以结合 `return` 实现同样的效果，即将 `run()` 方法中的 `throw new InterruptedException();` 替换为 `return`，然后 `run()` 方法和 `main()` 方法中的 `try-catch` 均可删除。

但是当 run 中存在多种操作需要多次判断 `this.interrupted()`时，通过抛出和捕捉异常，可以将日志等统一在 catch 处输出，但结合 return 则需要在每个 return 返回前都要打印日志。

```java
package com.gjxaiou.thread;

public class MyTest9 {
	public static void main(String[] args) {

		try {
			MyThread9 myThread9 = new MyThread9();
			myThread9.start();
			Thread.sleep(20);
			myThread9.interrupt();
		} catch (InterruptedException e) {
			System.out.println("main catch");
			e.printStackTrace();
		}
		System.out.println("happy end");
	}
}

class MyThread9 extends Thread {
	@Override
	public void run() {
		System.out.println("开始执行 run.");
		try {
			for (int i = 0; i < 30000; i++) {
				if (this.interrupted()) {
					System.out.println("线程已经停止，要退出啦");
					throw new InterruptedException();
				}
				System.out.println("i 的值" + i);
			}
			System.out.println("不会执行 for 循环外面");
		} catch (InterruptedException e) {
			System.out.println("进入 run 的 catch 方法");
			e.printStackTrace();
		}
	}
}
```

执行结果为：

```java
开始执行 run.
i 的值0
i 的值1
i 的值2
// 省略。。。。
i 的值3366
i 的值3367
i 的值3368
线程已经停止，要退出啦
happy end
进入 run 的 catch 方法
java.lang.InterruptedException
	at com.gjxaiou.thread.MyThread9.run(MyTest9.java:27)

Process finished with exit code 0
```

#### sleep 状态下停止线程

分为两种：

- 在 sleep 状态下执行 interrupt() 方法会抛出异常

    ```java
    package com.gjxaiou.thread;
    
    public class MyTest10 {
    	public static void main(String[] args) {
    		MyThread10 myThread10 = new MyThread10();
    		myThread10.start();
    		try {
    			// 主线程只 sleep 100ms，下面的 thread 则 sleep 20000
    			Thread.sleep(100);
    			myThread10.interrupt();
    		} catch (InterruptedException e) {
    			System.out.println("进入 main catch 中");
    			e.printStackTrace();
    		}
    		System.out.println("happy end");
    	}
    }
    
    class MyThread10 extends Thread {
    	@Override
    	public void run() {
    		System.out.println("run begin");
    		try {
    			Thread.sleep(20000);
    		} catch (InterruptedException e) {
    			System.out.println("sleep() 中被停止，进入 catch " + this.isInterrupted());
    			e.printStackTrace();
    		}
    	}
    }
    ```

    输出结果：

    ```java
    run begin
    happy end
    sleep() 中被停止，进入 catch false
    java.lang.InterruptedException: sleep interrupted
    	at java.lang.Thread.sleep(Native Method)
    	at com.gjxaiou.thread.MyThread10.run(MyTest10.java:23)
    ```

- 调用 interrupted() 方法给线程打了中断标记，再执行 sleep() 方法也会抛出异常。

    ```java
    package com.gjxaiou.thread;
    
    public class MyTest11 {
    	public static void main(String[] args) {
    		MyThread11 myThread11 = new MyThread11();
    		myThread11.start();
    		myThread11.interrupt();
    	}
    
    }
    
    class MyThread11 extends Thread {
    	@Override
    	public void run() {
    		System.out.println("run begin");
    		try {
    			Thread.sleep(2000);
    			System.out.println("run end");
    		} catch (InterruptedException e) {
    			System.out.println("先停止然后进入 sleep，进入了 run catch");
    			e.printStackTrace();
    		}
    	}
    }
    ```

    输出结果为：

    ```java
    run begin
    先停止然后进入 sleep，进入了 run catch
    java.lang.InterruptedException: sleep interrupted
    	at java.lang.Thread.sleep(Native Method)
    	at com.gjxaiou.thread.MyThread11.run(MyTest11.java:17)
    ```

### join 方法

join 方法作用：使得其所属的线程对象 X 正常执行 run() 方法里面的任务，而使当前线程 Z 进行无限期的等待，等待线程 X 销毁后再继续执行线程 Z 后面的代码，具有串行执行的效果。

- 其和 synchronized 区别为：join() 方法内部使用 wait() 方法进行等待，synchronized 使用锁作为同步。

- 使用 join() 方法过程中，如果当前线程对象被 interrupt() 方法中断，则当前线程出现异常。

    ```java
    package com.gjxaiou.thread;
    
    import java.lang.String;
    
    public class MyTest13 {
    	public static void main(String[] args) {
    		try {
    			ThreadB threadB = new ThreadB();
    			threadB.start();
    
    			Thread.sleep(500);
    			ThreadC threadC = new ThreadC(threadB);
    			threadC.start();
    		} catch (InterruptedException e) {
    			e.printStackTrace();
    		}
    	}
    
    }
    
    class ThreadA extends Thread {
    	@Override
    	public void run() {
    		for (int i = 0; i < Integer.MAX_VALUE; i++) {
    			String string = new String();
    			Math.random();
    		}
    	}
    }
    
    class ThreadB extends Thread {
    	@Override
    	public void run() {
    		try {
    			ThreadA threadA = new ThreadA();
    			threadA.start();
    			threadA.join();
    			System.out.println("线程 B 在 run end 处打印");
    		} catch (InterruptedException e) {
    			System.out.println("线程 B 在 catch 中打印了");
    			e.printStackTrace();
    		}
    	}
    }
    
    class ThreadC extends Thread {
    	private ThreadB threadB;
    
    	ThreadC(ThreadB threadB) {
    		this.threadB = threadB;
    	}
    
    	@Override
    	public void run() {
    		threadB.interrupt();
    	}
    }
    ```

    程序执行结果：

    ```java
    线程 B 在 catch 中打印了
    java.lang.InterruptedException
    	at java.lang.Object.wait(Native Method)
    	at java.lang.Thread.join(Thread.java:1252)
    	at java.lang.Thread.join(Thread.java:1326)
    	at com.gjxaiou.thread.ThreadB.run(MyTest13.java:37)
    ```

    #### join(long)

    x.join(long) 方法中的参数用于设定等待的时间，不管 X 线程是否执行完毕，时间到了并且重新获得了锁，则当前线程会继续向后执行。如果没有重新获得锁，则一直在尝试，直到获得锁为止。

**join(long) 和 sleep(long) 区别**

`join(long)` 内部使用 `wait(long)` 实现，所以 `join(long)` 会释放当前线程的锁，而 `Thread.sleep(long)` 不会释放锁。

```java
package com.gjxaiou.thread;

import java.util.concurrent.SynchronousQueue;

public class MyTest14 {
	public static void main(String[] args) {
		try {
			Thread2 thread2 = new Thread2();
			new Thread1(thread2).start();

			Thread.sleep(1000);
			new Thread3(thread2).start();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}

class Thread1 extends Thread {
	private Thread2 thread2;

	public Thread1(Thread2 thread2) {
		this.thread2 = thread2;
	}

	@Override
	public void run() {
		try {
			synchronized (thread2) {
				thread2.start();
				// 这里要休眠 6s
				Thread.sleep(6000);
			}
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}

class Thread2 extends Thread {
	@Override
	public void run() {
		try {
			long beginTime = System.currentTimeMillis();
			System.out.println("thread2 begin run," + beginTime);
			Thread.sleep(5000);
			long endTime = System.currentTimeMillis();
			System.out.println("thread2 end run," + endTime + " -- total time = " + (endTime - beginTime)/1000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}

	synchronized public void thread2Service() {
		System.out.println("执行 thread2Service time = " + System.currentTimeMillis());
	}
}

class Thread3 extends Thread {
	private Thread2 thread2;

	Thread3(Thread2 thread2) {
		this.thread2 = thread2;
	}

	@Override
	public void run() {
		thread2.thread2Service();
	}
}
```

执行结果为：

```java
thread2 begin run,1622989218651
thread2 end run,1622989223657 -- total time = 5
执行 thread2Service time = 1622989224664
```

线程 Thread1 使用 `Thread.sleep(6000)` 方法一直持有 Thread2 对象的锁，时间为 6s，所以 Thread3 在 6s 后才能执行 Thread2 中的同步方法。

但是如果使用 join() 方法，在执行 join() 方法的一瞬间，线程所持有的锁就释放了。

## 二、Runnable 接口

**下面为 Runnable 接口的 JavaDoc 文档**

> `Runnable` 接口应该由其实例（要由线程执行的任何类）实现。该类必须定义一个没有参数的 run 方法。
>
> 此接口旨在为希望在活动时执行代码的对象提供通用协议。例如，Thread 类实现了 Runnable 接口。
>
> **处于活动状态只意味着线程已启动但尚未停止**。
>
> 此外，`Runnable` 提供了一种方法，使类在不子类化 `Thread` 类的情况下处于活动状态。实现 Runnable 接口的类可以通过实例化 Thread 实例并将其自身作为目标传入而运行，而无需子类化 Thread 类。在大多数情况下，**如果您只计划重写 run（）方法，而不打算重写其他 Thread 类的方法，则应使用  Runnable 接口**。这一点很重要，因为除非程序员打算修改或增强类的基本行为，否则不应该对类进行子类化（即不应该继承）。

```java
package java.lang;

/*
 * @FunctionalInterface 表示该接口为函数式接口
 */
@FunctionalInterface
public interface Runnable {
    // 当实现 Runnable 接口的对象被用于创建一个线程的时候，启动该线程的时候会导致该对象的 run 方法在独立执行的线程中被调用。
    public abstract void run();
}
```

> 函数式接口含义：当一个接口中有且只有一个抽象方法时候（可以有默认实现方法和静态方法以及重写 Object 类中的方法）可以使用 @FunctionalInterface，函数式接口的实例可以使用 Lambda 表达式、方法引用、构造方法引用等方式进行创建。

## 三、使用 Thread 类和 Runnable 接口创建线程分析

### （一）使用 Thread 类

首先创建一个类继承 Thread 类，然后重写其中的 run 方法，最终调用 start 方法启动该线程。当调用 start 方法启动该线程的时候，JVM 会将该线程放入就绪队列中等待被调度（不是调用 start 方法之后就一定马上执行的），当一个线程被调度的时候会执行该线程的 run 方法。run() 方法里面就是线程要执行的任务代码。

**示例代码**

```java
package com.gjxaiou;

public class CreateThread1 extends Thread {
    @Override
    public void run() {
        System.out.println("WeChat Subscription GJXAIOU welcome you....");
        System.out.println("执行 run 的线程为：" + Thread.currentThread().getName());
    }

    public static void main(String[] args) {
        CreateThread1 createThread1 = new CreateThread1();
        // 耗时比较大
        createThread1.start();
        // 耗时比较小
        System.out.println("当前线程为：" + Thread.currentThread().getName());
    }
}
```

执行结果为：

```java
当前线程为：main
WeChat Subscription GJXAIOU welcome you....
执行 run 的线程为：Thread-0
```

因为一个进程正在运行时至少会有一个线程正在运行，执行 main() 方法的线程是由 JVM 创建的，线程名称也为 main。

**执行流程**：

因为CreateThread1 类继承了 Thread 类并且重写了 run 方法，所以 Thread 类中的 run 方法被覆盖了。然后调用 start 方法之后启动该线程就会执行 run 方法中的内容。

**调用 start() 方法之后会启动一个线程，该线程启动之后会自动调用线程对象中的 run() 方法，run() 方法里面的代码就是线程对象要执行的任务，是线程执行任务的入口。**

因为 start()  方法执行了以下多个步骤，因此比较耗时：

- 通过 JVM 告诉操作系统创建 Thread。
- 操作系统开辟内存，并且使用系统 SDK  中的创建线程函数进行创建 Thread 线程对象。
- 操作系统对 Thread 对象进行调度，以确定执行时机。
- Thread 在操作系统中被成功执行。

main 线程执行 start() 方法时候不必等待四步都执行完毕，而是立即继续执行 start() 方法之后的代码，这四步和后面的代码将一同执行。但是，**线程执行的顺序具有随机性**，也可能在执行完成的 start() 方法的四步之后才输出下面的语句（极少情况，通常需要在 start() 方法调用处后面加上 Thread.sleep(XX) 才能看到）。

**但是，使用多线程时，代码的运行结果与代码的执行顺序或调用顺序无关**。

**线程调用随机性的展示**：

```java
package com.gjxaiou;

public class Demo2 extends Thread {
    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(Thread.currentThread().getName() + " 正在执行");
        }
    }

    public static void main(String[] args) {
        Demo2 demo2 = new Demo2();
        demo2.setName("myThread");
        demo2.start();

        for (int i = 0; i < 5; i++) {
            System.out.println(Thread.currentThread().getName() + " 正在执行");
        }
    }
}
```

输出结果：

```java
main 正在执行
myThread 正在执行
main 正在执行
myThread 正在执行
myThread 正在执行
main 正在执行
main 正在执行
main 正在执行
myThread 正在执行
myThread 正在执行
```



### （二）使用 Runnable 接口

首先创建一个类实现 Runnable 接口，因为该接口中只有一个 run 方法，因此要在该新类中实现该 run 方法。

**示例代码**：

```java
package com.gjxaiou;

public class CreateThread2 implements Runnable {
    @Override
    public void run() {
        System.out.println("WeChat Subscription GJXAIOU welcome you....");
        System.out.println("执行 run 的线程为：" + Thread.currentThread().getName());
    }

    public static void main(String[] args) {
        CreateThread2 createThread2 = new CreateThread2();
        Thread thread = new Thread(createThread2);
        thread.start();
        System.out.println("当前线程为：" + Thread.currentThread().getName());
    }
}
```

执行结果为：

```java
当前线程为：main
WeChat Subscription GJXAIOU welcome you....
执行 run 的线程为：Thread-0
```

**当然，上述在 `new Thread()` 中传入的是一个 Runnable 实例，但是因为 Thread 类是 Runnable 的实现类，如果我们自定义一个类继承 Thread 类，则自定义类也是 Runnable 的实现类，所以自定义类的实例也可以传入 `new Thread()` 中，只不过这样显得多此一举。**

**执行流程**：

因为 CreateThread2 类实现了 Runnable 接口，而上文提到的 Thread 类的 8 个重构的构造方法中的第二个就是在构造 Thread 实例时候传入一个 Runnable 实例。因为没有覆盖 Thread 类中的 run 方法，因此 Thread 类中的 run 方法会执行，而上述 Thread 类的 run 方法中会调用 target 的 run 方法，这里就是调用 createThread2 的 run 方法完成执行。同时 target 是在 init() 方法中进行赋值初始化的，而 init() 方法是在 Thread 类的构造方法中被调用的。

![image-20210331150311699](一、Thread 类与 Runnable 接口分析.resource/image-20210331150311699.png)

### （三）两者分析

 **Runnable 接口作用**：将 `run()` 向上抽取，做成抽象方法，让实现类去重写

因为通过 start() 方法开启线程之后，都会调用 Thread 的 run()，只是该 run() 可能来自 Thread 类的子类（针对方式一），或者是 Thread 类自身（针对方式二）。因此 Thread 类（及其子类）是线程运行的入口。

#### 继承 Thread 类和实现 Runnable 接口比较

使用实现 Runnable 接口的方式：将线程执行的内容放置到了 Runnable 实现类中，使得执行者（线程）和被执行者（资源）进行分离解耦。【如果 Runnable 使用匿名对象进行构建，则同样无法实现资源共享】。同时如果同时使用多个 new Thread，传入同一个 Runnable 实例，然后使用 start() 方法进行执行会出现线程安全问题。

使用继承 Thread 类的方式：线程和代运行的代码放置在同一个类中，使得每个子类都有一份独立的 run()，资源没有独立也就无法共享。

