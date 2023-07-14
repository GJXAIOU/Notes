[TOC]

## 线程同步方法分析

### 一、Object 类中的关于线程的方法

主要包括 wait() 、notify() 、notifyAll() 方法

```java

package java.lang;

public class Object {

    public final native void notify();

    public final native void notifyAll();
   
    public final void wait() throws InterruptedException {
        wait(0);
    }
   
    public final native void wait(long timeout) throws InterruptedException;
   
    public final void wait(long timeout, int nanos) throws InterruptedException {
       // 纳秒时间处理省略
        wait(timeout);
    }
}    
```

### 二、wait 方法详解

进入 wait 的条件判断，如果是单线程可以使用 if，但是多线程必须使用 while。

wait() 方法有三个重载版本，本质上最终都会调用含有时间参数的版本。

#### （一）无参数 wait 方法分析

**JavaDoc 文档如下**：

> 会使当前线程等待，直到另一个线程调用此对象的 `java.lang.Object.notify()` 方法或`java.lang.Object.notifyAll()` 方法。（因此 wait 和 notify或者 notifyall 方法总是成对出现的，wait 会使当前线程出现等待，直到另一个线程调用了当前这个对象的 notify 或者 notifyAll 方法才会使当前线程被唤醒）
>
> 换句话说，这个方法的行为就像它只是执行调用 wait（0）(因为 wait() 里面的参数为超时时间，如果不写或者是 wait(0) 就是一直等待)。
>
> **当前线程必须拥有此对象的监视器（通常就是指锁）**。（当这个线程调用了 wait 方法之后）线程**释放**此监视器的所有权并等待，直到另一个线程通过调用 `notify` 方法或 `notifyAll` 方法通知等待此对象监视器唤醒的线程。然后线程还需等待，直到它可以重新获得监视器的所有权并继续执行。

上面总结：

  - 如果想调用 wait 方法，当前线程必须**拥有**这个对象的锁；
  - 一旦调用 wait 方法之后，调用该方法的线程就会**释放**被他所调用的对象的锁，然后进入等待状态。
   - 一直等待到另外的线程去通知在这个对象的锁上面等待的所有的线程（因为有可能一个对象，有多个线程都调用了这个对象的 wait 方法，这样多个线程都会陷入等待的状态）

> 在单参数版本中，中断和虚假唤醒是可能发生的，此方法应始终在循环中使用：
>
> ```java
> synchronized (obj) {
> while (<条件尚未满足>)
>   obj.wait();
> ...//Perform action appropriate to condition
> } 
> ```
>
> **此方法只能由作为此对象监视器所有者的线程调用**（即只能被拥有该对象锁的线程调用，否则抛出 IllegalMonitorStateException）。请参阅 notify方法，以了解线程成为监视器所有者的方式（线程如何获取对象的锁见 notify 方法）。

```java
public final void wait() throws InterruptedException {
    wait(0);
}
```

----

【总结】

**当前线程必须持有对象的锁才能调用其 wait 方法，而且一旦调用 wait 方法之后就会将调用了 wait 对象的锁释放掉。**

上面分析的是不带参数的 wait 方法，该方法实际实际上是调用了带参数的 wait 方法（见下），传入的是参数 0，接下来分析带参数的 wait 方法

```java
  public final void wait() throws InterruptedException {
        wait(0);
    }
```

#### （二）带参数 wait 方法分析

等待一段时间看是否有线程对锁进行 notify() 通知唤醒，如果超过这个线程线程会自动唤醒，但是继续向下执行的前提是再次持有锁，如果没有获取锁则会一直等待，直到持有锁为止。

-----

> **使当前线程等待，直到另一个线程调用此对象的 notify() 方法或 notifyAll() 方法，或者指定的时间已过。**
> 当前线程必须拥有此对象的监视器（锁）。
>
> 此方法会导致当前线程（称为 T 线程）将其自身放置在此对象的等待集合中，然后放弃此对象上的任何和所有同步声明（即是释放锁）。线程 T 出于线程调度目的被禁用，并处于休眠状态，直到发生以下四种情况之一：
>
> - 其他一些线程调用了此对象的 notify 方法，而线程 T 恰好被任意选择为要唤醒的线程（从等待集合中选择一个）。
> - 其他一些线程为此对象调用 notifyAll 方法。
> - 其他一些线程中断了线程 T 。
> - 指定的时间或多或少已经过去。但是，如果参数 timeout 为零，则不考虑实时性，线程只需等待通知。
>
> 然后从该对象的等待集合删除线程 T （因为已经被唤醒了），并重新启用线程调度。然后，它以通常的方式与其他线程去竞争该对象上的同步权；一旦它获得了对象的控制权（可以对这个对象同步了），它对对象的所有同步声明都将恢复到原来的状态，也就是说，恢复到调用 wait 方法时的所处的状态。然后线程 T 从 wait方法的调用中返回。因此，从 wait 方法返回时，对象和线程 T 的同步状态与调用 wait 方法时完全相同。
>
> 线程也可以在不被通知、中断或超时的情况下唤醒，即所谓的“虚假唤醒”。虽然这种情况在实践中很少发生，但应用程序必须通过测试本应导致线程被唤醒的条件，并在条件不满足时继续等待来防范这种情况。换句话说，等待应该总是以循环的形式出现，如下所示
>
> ```java
> // 对 obj 对象同步和上锁
> synchronized (obj) {
>     while (<condition does not hold>)
>     // 当另一个线程调用 obj 的 notify 方法的时候，正好当前线程就是被唤醒的线程的话，就会从这里唤醒然后执行一系列操作，然后再次判断
>        obj.wait(timeout);
>        ... // Perform action appropriate to condition
> }
> ```
>
> 如果当前线程在等待之前或等待期间被任何线程中断，则抛出一个 InterruptedException。在还原此对象的锁定状态（如上所述）之前，不会引发此异常。
>
> 注意，wait 方法在将当前线程放入此对象的等待集合中时，只解锁此对象；在线程等待期间，当前线程可能同步的任何其他对象都将保持锁定状态（因为一个线程在执行的时候可能同时调用几个对象的 wait 方法，但是某个时刻通过 notify 方法唤醒线程之后，但是其他对象还保持锁定）。
>
> 此方法只能由作为此对象监视器所有者的线程调用。查看 notify 方法，了解线程成为监视器所有者的方式

----

```java
/*
 * @param      timeout   // 超时时间（最长等待时间），如果为 0 表示一直等待。
 * @throws  IllegalArgumentException      如果超时时间 timeout 为负数。
 * @throws  IllegalMonitorStateException  如果当前线程不是对象 monitor 的拥有者。
 * @throws  InterruptedException 如果任何线程在当前线程等待通知之前或者期间中断了当前线程，会抛出该                                     异常，当抛出该异常之后，当前线程的中断状态将被清楚。
 */
public final native void wait(long timeout) throws InterruptedException;
```

另一个 wait 方法和上面 wait 一样，只不多加了一个设置纳秒的参数，可以实现更加精确的时间，可以从下面代码中看出最后还是调用了上面的一个参数的 wait 方法；

```java
// 这里的 JavaDoc 和上面相似，只是多了 nanos 说明以及使用方式
public final void wait(long timeout, int nanos) throws InterruptedException {
    if (timeout < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (nanos < 0 || nanos > 999999) {
        throw new IllegalArgumentException(
            "nanosecond timeout value out of range");
    }

    if (nanos >= 500000 || (nanos != 0 && timeout == 0)) {
        timeout++;
    }

    wait(timeout);
}
```



#### （三）测试

**测试1：线程必须用于对象的锁才能调用 wait 方法，如果直接调用会报错**

==当前的线程一定要持有调用 wait() 方法的对象（这里是 object 对象）的锁才可以==

```java
package com.gjxaiou;

/**
 * @Author GJXAIOU
 * @Date 2020/2/14 11:23
 */
public class MyTest1 {
    public static void main(String[] args) throws InterruptedException {

        Object object = new Object();
        // 测试：线程必须用于对象的锁才能调用 wait 方法，如果直接调用会报错
        // Exception in thread "main" java.lang.IllegalMonitorStateException
        // 解决方法：可以将调用 wait 方法放入 synchronized 同步代码块，因为进入代码块中就相当于获取到对象的锁了
        // object.wait();
        
        // 正确的使用方式如下：
        synchronized (object) {
            // 进入代码块相当于已经获取到 object 对象的锁
            object.wait();
        }
    }
}
```

对编译之后的 MyTest1.class 进行反编译之后得到（下面仅仅为 main 方法中反编译结果，省略了如构造方法等）

```java
 public static void main(java.lang.String[]) throws java.lang.InterruptedException;
    Code:
       0: new           #2                  // class java/lang/Object
       3: dup
       4: invokespecial #1                  // Method java/lang/Object."<init>":()V
       7: astore_1
       8: aload_1
       9: dup
      10: astore_2
      11: monitorenter // 注：当执行 synchronized 代码块，一旦进入对应的字节码指令为 monitorenter
      12: aload_1
      13: invokevirtual #3                  // Method java/lang/Object.wait:()V
      16: aload_2
      17: monitorexit // 注：从 synchronized 代码块正常或非正常退出都对应着 monitorexit 指令
      18: goto          26
      21: astore_3
      22: aload_2
      23: monitorexit // 这是异常的退出对应的 monitoexit 指令
      24: aload_3
      25: athrow
      26: return
```

**小结**

在调用 wait 方法时候，线程必须要持有被调用对象的锁，当调用 wait 方法之后，线程就会释放掉该对象的锁（monitor）。

### 三、Thread 类的 sleep 方法详解

> 会导致当前正在执行的线程进入休眠状态（临时的停止执行一段特定时间的毫秒数），它会受到系统定时器和调度器的精度的影响。**线程并不会失去任何锁的所有权**。

```java
/* @param  millis  //睡眠的时间长度
 * @throws  IllegalArgumentException // 如果 millis 值为负数
 * @throws  InterruptedException //  如果任何线程中断了当前线程，会抛出该异常，当抛出该异常之                                              后，当前线程的中断状态将被清楚。
 */
public static native void sleep(long millis) throws InterruptedException;
```

sleep() 方法的另一个版本和 wait() 方法类型，仅仅是加了一个纳秒的参数。

【总结】

**在调用 Thread 类的 sleep 方法时候，线程是不会释放掉对象的锁的。**调用时候不需要获取对象中锁，同时因为是静态方法，所以直接使用 `Thread.sleep();`即可。



### 四、notify 与 notifyAll 方法详解

#### （一） notify 方法分析

---

> **唤醒等待此对象锁的单个线程**。如果有任何（多个）线程正在等待这个对象（的锁），则选择其中一个等待的线程被唤醒。选择是**任意**的，由实现者自行决定【依赖于 JVM 的实现】。线程通过调用这个对象 wait 方法中的一个（因为有多个 wait）来等待对象的监视器。
>
> 在当前线程放弃对该对象的锁定之前，唤醒的线程将无法继续（执行）。唤醒的线程将以常规的方式与任何其他线程竞争，这些线程竞争在此对象上的同步；例如，唤醒的线程在成为下一个锁定此对象的线程时没有可靠的特权或劣势（和其它线程地位相同）。
>
> 此方法只能由作为此对象监视器所有者的线程调用（即调用了 notify 方法的线程一定是持有当前对象锁的线程）。线程通过以下三种方式之一成为对象锁的持有者：
>
> - 通过执行该对象的同步实例方法（即被标记为 Synchronized 的实例方法）。
> - 通过执行在对象上同步的{@code synchronized}语句的主体（即执行 synchronized 的语句块）。
> - 对于 class 类型的对象，通过执行该类的同步静态方法(即被标记为 Synchronized 的静态方法)
>     在某一时刻只有一个线程可以拥有该对象的锁的。

----

```java
/*
 * @throws  IllegalMonitorStateException  // 如果当前线程不是对象 monitor 的拥有者
 */
public final native void notify();
```

#### （二）notifyAll 方法分析

>**会唤醒此对象监视器上等待的所有线程**。线程通过调用其中一个 wait 方法来等待对象的监视器（锁）。
>
>在当前线程放弃对该对象的锁定之前，被唤醒的线程将无法继续。唤醒的线程将以通常的方式与任何其他线程竞争，这些线程可能正在积极竞争以在此对象上同步；例如，唤醒的线程在成为下一个锁定此对象的线程时没有可靠的特权或劣势。
>
>**此方法只能由作为此对象监视器所有者的线程调用**。请参阅 notify 方法，以了解线程成为监视器所有者的方式。

```java
/**
     * @throws  IllegalMonitorStateException  if the current thread is not
     *               the owner of this object's monitor.
     * @see        java.lang.Object#notify()
     * @see        java.lang.Object#wait()
     */
public final native void notifyAll();
```



### 五、wait 和 notify 、notifyAll 方法与线程同步系统总结

- 当调用 wait 方法时，首先需要确保**调用的 wait 方法的线程已经持有了对象的锁**；这里的对象是调用 wait 的对象，因为 wait 是定义在 Object 类中，所以任何一个对象都会有一个 wait 方法。
- 当调用 wait 方法后，该线程就会释放掉这个对象的锁，然后进入到等待状态（或者成为等待集合：wait set)；
- 当线程调用了 wait 之后进入到等待状态时候，它就可以等待其他线程调用相同对象的 notify 或 notifyAll 方法来使得自己被唤醒；
- 一旦这个线程被其他线程唤醒之后，该线程就会与其他线程一同开始**竞争**这个对象的锁（公平竞争）；**只有当该线程获取到了这个对象的锁之后**，代码才会继续向下执行，没有获取到则继续等待。
- 调用 wait 方法的代码片段需要放在一个 synchronized 块或者是 synchronized 方法中，这样才可以确保线程在调用 wait 方法前已经获取到了对象的锁。
- 当调用对象的 notify 方法时，它会随机唤醒该对象等待集合（wait set) 中的任意一个线程，当某个线程被唤醒之后，他就会与其他线程一同竞争对象的锁。
- 当调用对象的 notifyAll 方法时候，他就会唤醒该对象等待集合（wait set) 中的所有线程，这些线程被唤醒之后，又会开始竞争对象的锁；
- 在某一个时刻，只有唯一一个线程可以拥有对象的锁。

wait/notify 实现了多个线程之间的通信。不采用该方案，多个线程之间也可以通信，其本质上是因为多个线程共同访问同一个变量，两个线程完全是主动式地操作同一个共享变量，但是该方式一方面划分读取时间，同时读取到的值不能确定是不是想要的。

如通过 `sleep() 和 while(true)`死循环实现多个线程间通信。

```java
package com.gjxaiou.object;


import java.util.ArrayList;
import java.util.List;
import java.util.stream.IntStream;

public class MyTest1 {
	public static void main(String[] args) {
		MyList list = new MyList();
		new MyThread1(list).start();
		new MyThread2(list).start();
	}

}

class MyThread1 extends Thread {
	public MyList list;

	MyThread1(MyList list) {
		this.list = list;
	}

	@Override
	public void run() {
		try {
			for (int i = 0; i < 10; i++) {
				list.add();
				System.out.println("添加了 " + (i + 1) + " 个元素");
				Thread.sleep(1000);
			}
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}

class MyThread2 extends Thread {
	public MyList list;

	MyThread2(MyList list) {
		this.list = list;
	}

	@Override
	public void run() {
		try {
			while (true) {
                // Thread.sleep(1500);
				if (list.getSize() == 5) {
					System.out.println("到达 5 个元素了，线程 MyThread2 退出");
					throw new InterruptedException();
				}
			}
		} catch (InterruptedException e) {
			e.printStackTrace();
		}

	}
}

class MyList {
	public volatile List list = new ArrayList();

	public void add() {
		list.add("GJXAIOU");
	}

	public int getSize() {
		return list.size();
	}
}
```

执行结果：

```java
添加了 1 个元素
添加了 2 个元素
添加了 3 个元素
添加了 4 个元素
添加了 5 个元素
到达 5 个元素了，线程 MyThread2 退出
java.lang.InterruptedException
	at com.gjxaiou.object.MyThread2.run(MyTest1.java:50)
添加了 6 个元素
添加了 7 个元素
添加了 8 个元素
添加了 9 个元素
添加了 10 个元素
```

该种方式的两个线程之间的通信，缺点是 MyThread2 需要通过 while 循环轮询机制来检测某一个条件，浪费 CPU 资源，但是如果加上注释中的 sleep，会可能得不到想要数据无法退出线程。

**使用 wait/notify 机制实现 `list.size()` 等于 5 时的线程销毁**

```java
package com.gjxaiou.object;

import java.util.ArrayList;
import java.util.List;

public class MyTest2 {

	public static void main(String[] args) {
		try {
			Object o = new Object();
			new MyThread3(o).start();
			Thread.sleep(50);
			new MyThread4(o).start();
		} catch (Exception e) {
			e.printStackTrace();
		}

	}
}


class MyThread3 extends Thread {
	public Object lock;

	MyThread3(Object lock) {
		this.lock = lock;
	}

	@Override
	public void run() {
		try {
			synchronized (lock) {
				if (MyList2.getSize() != 5) {
					System.out.println("wait begin" + System.currentTimeMillis());
					lock.wait();
					System.out.println("wait end" + System.currentTimeMillis());
				}
			}
		} catch (Exception e) {
			e.printStackTrace();
		}

	}
}

class MyThread4 extends Thread {
	private Object lock;

	MyThread4(Object lock) {
		this.lock = lock;
	}

	public void run() {
		try {
			synchronized (lock) {
				for (int i = 0; i < 10; i++) {
					MyList2.add();
					if (MyList2.getSize() == 5) {
						lock.notify();
						System.out.println("发出了唤醒通知");
					}
					System.out.println("添加了 " + (i + 1) + "个元素");
					Thread.sleep(1000);
				}
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

}


class MyList2 {
	private static List list = new ArrayList();

	public static void add() {
		list.add("GJXAIOU");
	}

	public static int getSize() {
		return list.size();
	}
}
```

程序输出结果：

```java
wait begin1622564798907
添加了 1个元素
添加了 2个元素
添加了 3个元素
添加了 4个元素
发出了唤醒通知
添加了 5个元素
添加了 6个元素
添加了 7个元素
添加了 8个元素
添加了 9个元素
添加了 10个元素
wait end1622564809053
```

**说明**：

- wait end 最后输出，表明 notify() 方法执行后并不立刻释放锁，即必须执行完 notify() 方法所在的同步 synchronized 代码块后才释放锁。
- 任意一个 Object 对象都可以作为锁，内部都有 `wait()` 和 `notify()` 方法；
- 通过调用 `wait()` 方法可以使得临界区类的线程进入等待状态，同时释放被同步对象的锁，notify 可以唤醒一个因调用了 wait 操作而处于 wait 状态的线程，使其进入就绪状态，被重新唤醒的线程会视图重新获取临界区的控制权（锁），并继续执行临界区内 wait 之后的代码。 	

### 六、wait 和 notify 方法案例剖析和详解

编写一个多线程程序，实现目标：

- 存在一个对象，该对象（即对象对应的类中）存在一个 int 类型的成员变量 counter，该成员变量的初始值为 0；
- 创建两个线程，其中一个线程对该对象的成员变量 counter 增1，另一个线程对该对象的成员变量减 1；
- 输出该对象成员变量 counter 每次变化后的值；最终输出的结果应该为：1010101010…



**分析**

- 需要两个线程，因为操作不同，所以需要两个线程类；

首先构造 MyObject 类

```java
package com.gjxaiou;

/**
 * @Author GJXAIOU
 * @Date 2020/2/15 10:51
 */
public class MyObject {
    private int counter;

    // 分别实现递增、递减方法，因为要调用 wait 方法，所以需要使用 synchronized
    public synchronized void increase() {
        // 进入 increase 方法之后，说明该线程已经拿到该对象的锁，如果 counter 不为 0，则该线程等待
        while (counter != 0) {
            try {
                // 调用 wait 使得释放掉该对象的锁，使得递减的线程拿到锁
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        // 当另一个线程执行完递减之后，调用 notify 方法让递增线程唤醒了
        counter++;
        System.out.println(counter);
        // 通知其它线程起来（因为只有两个线程，所以这里就是指递减的线程）
        notify();
    }

    // 下面的解释同上
    public synchronized void decrease() {
        while (counter == 0) {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        counter--;
        System.out.println(counter);
        notify();
    }
}
```

然后分别实现递增和递减两个线程类

```java
package com.gjxaiou;

/**
 * @Author GJXAIOU
 * @Date 2020/2/15 11:18
 */
public class IncreaseThread extends Thread {
    // 因为要操作 MyObject 的对象
    private MyObject myObject;

    public IncreaseThread(MyObject myObject) {
        this.myObject = myObject;
    }

    // 重写 run 方法
    @Override
    public void run() {
        for (int i = 0; i < 30; i++) {
            // 每次先让线程随机休眠 0 ~ 1 秒
            try {
                Thread.sleep((long) (Math.random() * 1000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            // 每次调用都会尝试获取该对象的锁，如果获取到就进入了方法体内部，然后判断 counter 不等于 0 的话就会释放掉这把锁，并且进入等待状态。直到另一个线程使用 notify  进行唤醒继续往下执行。
            myObject.increase();
        }
    }
}

```

```java
package com.gjxaiou;

/**
 * @Author GJXAIOU
 * @Date 2020/2/15 11:24
 */
public class DecreaseThread extends Thread {
    private MyObject myObject;

    public DecreaseThread(MyObject myObject) {
        this.myObject = myObject;
    }

    // 重写 run 方法
    @Override
    public void run() {
        for (int i = 0; i < 30; i++) {
            // 每次先让线程休眠 0 ~ 1 秒
            try {
                Thread.sleep((long) (Math.random() * 1000));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            myObject.decrease();
        }
    }
}

```

最后在 main 方法中创建分别创建一个递增和递减线程，然后执行程序

```java
package com.gjxaiou;

/**
 * @Author GJXAIOU
 * @Date 2020/2/15 11:26
 */
public class Client {
    public static void main(String[] args) {
        // 只能创建一个对象，因为锁是加在对象上，如果两个线程操作两个对象就没有意义了MyObject myObject = new MyObject();
        MyObject myObject = new MyObject();
        Thread increaseThread = new IncreaseThread(myObject);
        Thread decreaseThread = new DecreaseThread(myObject);
        increaseThread.start();
        decreaseThread.start();
    }
}
```

程序运行结果：（这里选取前十结果）

```java
1
0
1
0
1
0
1
0
1
0
```

如果创建 4 个线程对象，它们分别来自两个类的实例（IncreaseThread 和 DecreaseThread），然后分别启动这 4 个线程。

```java
package com.gjxaiou;

public class Client {
    public static void main(String[] args) {
    
        MyObject myObject = new MyObject();
        Thread increaseThread = new IncreaseThread(myObject);
        Thread increaseThread2 = new IncreaseThread(myObject);
        Thread decreaseThread = new DecreaseThread(myObject);
        Thread decreaseThread2 = new DecreaseThread(myObject);
        increaseThread.start();
        increaseThread2.start();
        decreaseThread.start();
        decreaseThread2.start();
    }
}
```

执行结果就是随机值了。同时程序执行完之后，JVM 并没有退出。【前提是 increase 和 decrease 方法中使用是 if 额不是 while】

原来只有两个线程对象，一个增加一个减少。假如在某个时刻，用于增加的线程进入了  increase 方法，如果此时恰巧 counter = 0，则会执行 counter++，然后输出 1，接着调用 notify() 方法，notify 只会唤醒唯一一个即用于减少的线程（执行 decrease() 方法的线程对象），该线程对象可能正处于  wait() 方法位置，然后因为收到  notify 通知，则该线程被唤醒（因为只有 2 个线程，所以一个线程调用 notify 会唤醒另一个）。然后减少线程起来之后 counter—，输出 0，然后又调用了 notify 。减少的线程调用 notify 唤醒的肯定是调用增加方法的线程，该线程可能处于 wait() 状态，或者还没有拿到锁而没有进入 increase 方法。

针对四个线程对象：

例如开始是起始状态，counter = 0。两个用于增加的线程中的一个进入了 increase 方法中，则会依次执行 counter++，然后输出，并且调用 notify 方法，唤醒在这个 object 对象上调用 wait 方法的线程，如果此时没有线程在 wait，则第一个线程正常执行完该方法之后就退出了。然后用于减少的线程对象中的一个执行了 decrease 方法，然后依次执行counter–，然后输出，然后执行  notify 方法。如果第三个线程执行的是减少的方法，然后因为 counter = 0，则进入 wait 进行等待。因为 wait  会释放掉这个对象的锁，则另外三个线程（2个用于增加，一个用户减少）都有可能执行  increase 和 decrease 方法，如当减少线程进入 decrease 方法，由进行了减一操作。

进入 wait 之后被 notify 唤醒之后也要再次争抢获取对象的锁，获取到才能继续执行。没抢到就一直在 wait 进行等待。

修改 if 为 while，即当线程从 wait 被唤醒之后需要再次判断  counter 的值，因为此时 counter 值可能被其它线程修改，使得其不再是之前进入 wait 时候的值了，符合了才能继续执行。



### 七、生产者和消费者模式

生产者和消费者的关系有一对一、一对多、多对一、多对多。但是都是基于 wait/notify 的。

#### 一对一操作值

```java
package com.gjxaiou.object.productAndConsumer;

import java.lang.String;

public class One2OneValue {

	public static void main(String[] args) {
		String lock = "";
		Product product = new Product(lock);
		Consumer consumer = new Consumer(lock);
		new ThreadProduct(product).start();
		new ThreadConsumer(consumer).start();
	}

}

// 存储值的对象
class ObjectValue {
	public static String value = "";
}

// 生产者和消费者
class Product {
	private String lock;

	public Product(String lock) {
		this.lock = lock;
	}

	public void setValue() {
		try {
			synchronized (lock) {
				if (!ObjectValue.value.equals("")) {
					System.out.println("product 进入等待 waiting 状态," + Thread.currentThread().getName());
					lock.wait();
				}
				System.out.println("product 进入 Runnable 状态，" + Thread.currentThread().getName());
				ObjectValue.value = System.currentTimeMillis() + "--" + System.nanoTime();
				System.out.println("product 设置值为： " + ObjectValue.value);
				lock.notify();
			}
		} catch (Exception e) {
			e.printStackTrace();
		}

	}
}

class Consumer {
	private String lock;

	public Consumer(String lock) {
		this.lock = lock;
	}

	public void getValue() {
		try {
			synchronized (lock) {
				if (ObjectValue.value.equals("")) {
					System.out.println("consumer 进入等待 waiting 状态," + Thread.currentThread().getName());
					lock.wait();
				}
				System.out.println("consumer 进入 Runnable 状态，" + Thread.currentThread().getName());
				System.out.println("consumer 获取到值为： " + ObjectValue.value);
				ObjectValue.value = "";
				lock.notify();
			}
		} catch (Exception e) {
			e.printStackTrace();
		}

	}
}

// 生成者和消费者线程
class ThreadProduct extends Thread {
	private Product product;

	ThreadProduct(Product product) {
		this.product = product;
	}

	@Override
	public void run() {
		while (true) {
			product.setValue();
		}
	}
}

class ThreadConsumer extends Thread {
	private Consumer consumer;

	ThreadConsumer(Consumer consumer) {
		this.consumer = consumer;
	}

	@Override
	public void run() {
		while (true) {
			consumer.getValue();
		}
	}
}
```

运行结果为：

```java
consumer 进入 Runnable 状态，Thread-1
consumer 获取到值为： 1622887428535--1016397408781600
consumer 进入等待 waiting 状态,Thread-1
product 进入 Runnable 状态，Thread-0
product 设置值为： 1622887428535--1016397408805500
product 进入等待 waiting 状态,Thread-0
consumer 进入 Runnable 状态，Thread-1
consumer 获取到值为： 1622887428535--1016397408805500
consumer 进入等待 waiting 状态,Thread-1
product 进入 Runnable 状态，Thread-0
product 设置值为： 1622887428535--1016397408839100
product 进入等待 waiting 状态,Thread-0
consumer 进入 Runnable 状态，Thread-1
consumer 获取到值为： 1622887428535--1016397408839100
consumer 进入等待 waiting 状态,Thread-1
product 进入 Runnable 状态，Thread-0
product 设置值为： 1622887428535--1016397408865900
product 进入等待 waiting 状态,Thread-0
。。。。。
```

#### 多对多操作值【假死】

首先上面的 Product 和 Consumer 中的 wait() 条件判断应该将 if 修改为 while，然后 main 方法修改如下：

```java
public class More2MoreValue {
	public static void main(String[] args) throws InterruptedException {
		String lock = "";
		Product product1 = new Product(lock);
		Consumer consumer1 = new Consumer(lock);

		ThreadProduct[] threadProduct1s = new ThreadProduct[2];
		ThreadConsumer[] threadConsumer1s = new ThreadConsumer[2];

		for (int i = 0; i < 2; i++) {
			threadProducts[i]  = new ThreadProduct(product1);
			threadProducts[i].setName("生产者：" + (i + 1));

			threadConsumers[i]  = new ThreadConsumer(consumer1);
			threadConsumers[i].setName("消费者：" + (i + 1));

			threadProducts[i].start();
			threadConsumers[i].start();
		}

		Thread.sleep(5000);
		Thread[] threadArray = new Thread[Thread.currentThread().getThreadGroup().activeCount()];
		Thread.currentThread().getThreadGroup().enumerate(threadArray);

		for (int i = 0; i < threadArray.length; i++) {
			System.out.println(threadArray[i].getName() + "----" + threadArray[i].getState());
		}
	}
}
```

输出结果为：

```java
product 进入 Runnable 状态，生产者：1
product 设置值为： 1622890226099--1019194973749900
product 进入等待 waiting 状态,生产者：1
consumer 进入 Runnable 状态，消费者：1
consumer 获取到值为： 1622890226099--1019194973749900
consumer 进入等待 waiting 状态,消费者：1
product 进入 Runnable 状态，生产者：2
product 设置值为： 1622890226099--1019194974155800
product 进入等待 waiting 状态,生产者：2
consumer 进入 Runnable 状态，消费者：2
consumer 获取到值为： 1622890226099--1019194974155800
consumer 进入等待 waiting 状态,消费者：2
......
product 进入 Runnable 状态，生产者：1
product 设置值为： 1622890226124--1019194999256100
product 进入等待 waiting 状态,生产者：1
product 进入等待 waiting 状态,生产者：2
consumer 进入 Runnable 状态，消费者：1
consumer 获取到值为： 1622890226124--1019194999256100
consumer 进入等待 waiting 状态,消费者：1
consumer 进入等待 waiting 状态,消费者：2
main----RUNNABLE
Monitor Ctrl-Break----RUNNABLE
生产者：1----WAITING
消费者：1----WAITING
生产者：2----WAITING
消费者：2----WAITING    
```

当所有的线程都呈现 waiting 状态时候，不再执行任何任务，程序也没有退出，呈现假死状态。主要是因为 notify 唤醒可能是异类，也可能是同类，即「生产者」唤醒「生产者」、「生产者」唤醒「消费者」都可能。假死主要就是因为有可能连续的唤醒同类。

**假死解决方式**：不光唤醒同类，同时唤醒异类，即将所有的 `notify()` 修改为 `notifyAll()`。

#### 生产消费一对一：操作栈

生产者向堆栈 List 对象中放入数据，使用消费者从 List 堆栈中取出数据，List 最大容量为 1。

```java
package com.gjxaiou.object.productAndConsumer.operateList;


import java.util.ArrayList;
import java.util.List;

public class One2One {
	public static void main(String[] args) {
		MyStack myStack = new MyStack();
		Product product = new Product(myStack);
		Consumer consumer = new Consumer(myStack);
		new ProductThread(product).start();
		new ConsumerThread(consumer).start();
	}
}

// 操作值
class MyStack {
	private List list = new ArrayList();

	synchronized public void push() {
		try {
			if (list.size() == 1) {
				this.wait();
			}
			list.add(Math.random());
			this.notify();
			System.out.println("push 完成，list 大小为：" + list.size());
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	synchronized public String pop() {
		String result = "";
		try {
			if (list.size() == 0) {
				System.out.println("pop 中的 " + Thread.currentThread().getName() + " 线程是 wait 状态");
				this.wait();
			}
			result = list.get(0).toString();
			list.remove(0);
			this.notify();
			System.out.println("pop 完成，list 大小为：" + list.size());
		} catch (Exception e) {
			e.printStackTrace();
		}
		return result;
	}
}

// 生产者和消费者
class Product {
	private MyStack myStack;

	Product(MyStack myStack) {
		this.myStack = myStack;
	}

	public void pushService() {
		myStack.push();
	}
}

class Consumer {
	private MyStack myStack;

	Consumer(MyStack myStack) {
		this.myStack = myStack;
	}

	public void popService() {
		myStack.pop();
	}
}

// 生产者和消费者线程
class ProductThread extends Thread {
	private Product product;

	ProductThread(Product product) {
		this.product = product;
	}

	@Override
	public void run() {
		while (true) {
			product.pushService();
		}
	}
}

class ConsumerThread extends Thread {
	private Consumer consumer;

	ConsumerThread(Consumer consumer) {
		this.consumer = consumer;
	}

	@Override
	public void run() {
		while (true) {
			consumer.popService();
		}
	}
}
```

输出结果为：

```java

push 完成，list 大小为：1
pop 完成，list 大小为：0
pop 中的 Thread-1 线程是 wait 状态
push 完成，list 大小为：1
pop 完成，list 大小为：0
pop 中的 Thread-1 线程是 wait 状态
push 完成，list 大小为：1
pop 完成，list 大小为：0
pop 中的 Thread-1 线程是 wait 状态
push 完成，list 大小为：1
pop 完成，list 大小为：0
pop 中的 Thread-1 线程是 wait 状态
。。。。
```

一生产多消费：一个生产者向堆栈 List 对象中放入数据，多个消费者从 List 堆栈中取出数据，List 最大数量还是 1。或者多生产一消费，或者多生成多消费。

需要修改如下：

- wait() 条件判断由 if 修改为 while。

    使用 if 会出现条件发生改变时并没有得到及时响应，所以多个呈 wait 状态的线程被唤醒，继而执行 `list.remove(0)` 代码出现异常 `java.lang.IndexOutOfBoundsException`。

- 将 `notify()` 换成 `notifyAll()` 防止假死

多生产一消费 main 方法中示例如下，其他类似。

```java
package com.gjxaiou.object.productAndConsumer.operateList;


public class More2One {
	public static void main(String[] args) {
		MyStack myStack = new MyStack();
		Product product1 = new Product(myStack);
		Product product2 = new Product(myStack);
		Product product3 = new Product(myStack);

		new ProductThread(product1).start();
		new ProductThread(product2).start();
		new ProductThread(product3).start();

		Consumer consumer = new Consumer(myStack);
		new ConsumerThread(consumer).start();
	}
}
```

#### 连续生产多个、连续消耗多个

即实现「生产-生产-生产-消费-消费-生产。。」，多个生产与多个消费向一个 Box 容器中连续的放入和取出，容器的最大容量不超过 50。

```java
package com.gjxaiou.object.productAndConsumer.continueOperate;

import java.util.ArrayList;
import java.util.List;

public class More2More {
	public static void main(String[] args) throws InterruptedException {
		Box box = new Box();
		SetService setService = new SetService(box);
		for (int i = 0; i < 2; i++) {
			new SetValueThread(setService).start();
		}

		Thread.sleep(50);
		new SetCheckThread(setService).start();

		Thread.sleep(10000);
		GetService getService = new GetService(box);
		for (int i = 0; i < 10; i++) {
			new GetValueThread(getService).start();
		}
		Thread.sleep(50);
		new GetCheckThread(getService).start();
	}
}

// 操作的容器
class Box {
	private static List list = new ArrayList();

	synchronized public void add() {
		if (size() < 50) {
			list.add("gjxaiou");
			System.out.println(Thread.currentThread().getName() + " 执行 add() 方法，size 大小为： " + size());
		}
	}

	synchronized public int size() {
		return list.size();
	}

	synchronized public Object popFirst() {
		Object value = list.remove(0);
		System.out.println(Thread.currentThread().getName() + " 执行 popFirst() 方法，size 大小为： " + size());
		return value;
	}
}

// 生产者和消费者业务代码
class SetService {
	private Box box;

	public SetService(Box box) {
		this.box = box;
	}

	public void setMethod() {
		try {
			synchronized (this) {
				while (box.size() == 50) {
					System.out.println("●●●●●●●●●●●●");
					this.wait();
				}
			}
			Thread.sleep(300);
			box.add();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	public void checkBoxStatus() {
		try {
			while (true) {
				synchronized (this) {
					if (box.size() < 50) {
						this.notifyAll();
					}
				}
				System.out.println("set checkboxBox = " + box.size());
				Thread.sleep(1000);
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

}


class GetService {
	private Box box;

	GetService(Box box) {
		this.box = box;
	}

	public void getMethod() {
		try {
			synchronized (this) {
				while (box.size() == 0) {
					System.out.println("○○○○○○○○○○");
					this.wait();
				}
				box.popFirst();
			}
			Thread.sleep(300);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}

	public void checkBoxStatus() {
		try {
			while (true) {
				synchronized (this) {
					if (box.size() > 0) {
						this.notifyAll();
					}
				}
				System.out.println("get checkboxBox = " + box.size());
				Thread.sleep(1000);
			}
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}


// 生产者和消费者线程
class SetValueThread extends Thread {
	private SetService setService;

	SetValueThread(SetService setService) {
		this.setService = setService;
	}

	@Override
	public void run() {
		while (true) {
			setService.setMethod();
		}
	}
}

class GetValueThread extends Thread {
	private GetService getService;

	public GetValueThread(GetService getService) {
		this.getService = getService;
	}

	@Override
	public void run() {
		while (true) {
			getService.getMethod();
		}
	}
}


// 生产者和消费者容器大小测试线程
class SetCheckThread extends Thread {
	private SetService setService;

	SetCheckThread(SetService setService) {
		this.setService = setService;
	}

	@Override
	public void run() {
		setService.checkBoxStatus();
	}
}

class GetCheckThread extends Thread {
	private GetService getService;

	GetCheckThread(GetService getService) {
		this.getService = getService;
	}

	@Override
	public void run() {
		getService.checkBoxStatus();
	}
}
```

程序输出结果为：

```java
set checkboxBox = 0
Thread-1 执行 add() 方法，size 大小为： 1
Thread-0 执行 add() 方法，size 大小为： 2
Thread-0 执行 add() 方法，size 大小为： 3
Thread-1 执行 add() 方法，size 大小为： 4
Thread-0 执行 add() 方法，size 大小为： 5
Thread-1 执行 add() 方法，size 大小为： 6
set checkboxBox = 6
Thread-0 执行 add() 方法，size 大小为： 7
Thread-1 执行 add() 方法，size 大小为： 8
Thread-0 执行 add() 方法，size 大小为： 9
Thread-1 执行 add() 方法，size 大小为： 10
。。。。
Thread-1 执行 add() 方法，size 大小为： 45
Thread-0 执行 add() 方法，size 大小为： 46
set checkboxBox = 46
Thread-1 执行 add() 方法，size 大小为： 47
Thread-0 执行 add() 方法，size 大小为： 48
Thread-0 执行 add() 方法，size 大小为： 49
Thread-1 执行 add() 方法，size 大小为： 50
●●●●●●●●●●●●
●●●●●●●●●●●●
set checkboxBox = 50
set checkboxBox = 50
Thread-3 执行 popFirst() 方法，size 大小为： 49
Thread-6 执行 popFirst() 方法，size 大小为： 48
Thread-9 执行 popFirst() 方法，size 大小为： 47
Thread-5 执行 popFirst() 方法，size 大小为： 46
Thread-7 执行 popFirst() 方法，size 大小为： 45
Thread-4 执行 popFirst() 方法，size 大小为： 44
Thread-8 执行 popFirst() 方法，size 大小为： 43
Thread-11 执行 popFirst() 方法，size 大小为： 42
Thread-10 执行 popFirst() 方法，size 大小为： 41
Thread-12 执行 popFirst() 方法，size 大小为： 40
set checkboxBox = 40
get checkboxBox = 40
Thread-12 执行 popFirst() 方法，size 大小为： 39
Thread-8 执行 popFirst() 方法，size 大小为： 38
Thread-4 执行 popFirst() 方法，size 大小为： 37
Thread-3 执行 popFirst() 方法，size 大小为： 36
Thread-9 执行 popFirst() 方法，size 大小为： 35
Thread-10 执行 popFirst() 方法，size 大小为： 34
Thread-7 执行 popFirst() 方法，size 大小为： 33
Thread-5 执行 popFirst() 方法，size 大小为： 32
Thread-11 执行 popFirst() 方法，size 大小为： 31
Thread-6 执行 popFirst() 方法，size 大小为： 30
Thread-1 执行 add() 方法，size 大小为： 31
Thread-0 执行 add() 方法，size 大小为： 32
Thread-7 执行 popFirst() 方法，size 大小为： 31
Thread-10 执行 popFirst() 方法，size 大小为： 30
。。。。。
Thread-6 执行 popFirst() 方法，size 大小为： 2
Thread-9 执行 popFirst() 方法，size 大小为： 1
Thread-4 执行 popFirst() 方法，size 大小为： 0
○○○○○○○○○○
○○○○○○○○○○
Thread-1 执行 add() 方法，size 大小为： 1
Thread-0 执行 add() 方法，size 大小为： 2
Thread-7 执行 popFirst() 方法，size 大小为： 1
Thread-5 执行 popFirst() 方法，size 大小为： 0
○○○○○○○○○○
○○○○○○○○○○
○○○○○○○○○○
○○○○○○○○○○
○○○○○○○○○○
○○○○○○○○○○
Thread-1 执行 add() 方法，size 大小为： 1
Thread-0 执行 add() 方法，size 大小为： 2
set checkboxBox = 2
Thread-8 执行 popFirst() 方法，size 大小为： 1
get checkboxBox = 2
Thread-4 执行 popFirst() 方法，size 大小为： 0
○○○○○○○○○○
○○○○○○○○○○
○○○○○○○○○○
○○○○○○○○○○
○○○○○○○○○○
○○○○○○○○○○
○○○○○○○○○○
○○○○○○○○○○
Thread-0 执行 add() 方法，size 大小为： 1
Thread-1 执行 add() 方法，size 大小为： 2
Thread-4 执行 popFirst() 方法，size 大小为： 1
```

因为消费者数量大于生产者，所以多次输出 ○○○○○○○○，说明消费者执行了 wait() 等待。

#### 线程有序交叉执行

需求：4 个线程，一半执行 A 操作，一半执行 B 操作，两个操作轮流执行。

```java
package com.gjxaiou.object;

public class TurnBackup {

	public static void main(String[] args) {
		BackupService backupService = new BackupService();
		for (int i = 0; i < 4; i++) {
			new BackupBThread(backupService).start();
			new BackupAThread(backupService).start();
		}
	}
}

class BackupService {
    // 实现两个线程轮流交替执行
	volatile private boolean prevIsA = false;

	synchronized public void backupA() {
		try {
			while (prevIsA == true) {
				wait();
			}
			for (int i = 0; i < 2; i++) {
				System.out.println("backupA");
			}
			prevIsA = true;
			notifyAll();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}

	synchronized public void backupB() {
		try {
			while (prevIsA == false) {
				wait();
			}
			for (int i = 0; i < 2; i++) {
				System.out.println("backupB");
			}
			prevIsA = false;
			notifyAll();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}

}

// 线程工具类
class BackupAThread extends Thread {
	private BackupService backupService;

	BackupAThread(BackupService backupService) {
		this.backupService = backupService;

	}

	@Override
	public void run() {
		backupService.backupA();
	}
}

class BackupBThread extends Thread {
	private BackupService backupService;

	BackupBThread(BackupService backupService) {
		this.backupService = backupService;

	}

	@Override
	public void run() {
		backupService.backupB();
	}
}
```

程序输出结果：

```java
backupA
backupA
backupB
backupB
backupA
backupA
backupB
backupB
backupA
backupA
backupB
backupB
backupA
backupA
backupB
backupB
```

