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

> **唤醒等待此对象锁的单个线程**。如果有任何（多个）线程正在等待这个对象（的锁），则选择其中一个等待的线程被唤醒。选择是**任意**的，由实现者自行决定。线程通过调用这个对象 wait 方法中的一个（因为有多个 wait）来等待对象的监视器。
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