# Condition

[TOC]



## 一、condition 详解及相比于传统线程并发模式的改进

传统上，我们可以通过 synchronized 关键字 ＋ wait + notify/notifyAll 来实现多个线程之间的协调与通信，整个过程都是由 JVM 来帮助我们实现的，开发者无需(也无法)了解底层的实现细节。

从 JDK5 开始，并发包提供了 lock + Condition(主要是 await 与 signal/signalAll )来实现多个线程之间的协调与通信，整个过程都是由开发者来控制的，而且相比于传统方式更加灵活，功能也更加强大。

**Thread.sleep 与 await(或是 Object#wait() 方法)的区别**

 **sleep 方法本质上不会释放锁，而 await 会释放锁**，并且在 signal 后，还需要重新获得锁才能继续执行(该行为与 Object 的 wait 方法完全一致)。

**Object 类监视器方法和 Condition 对比**

| 对比项                                                   | Object 监视器方法         | Condition                                                    |
| -------------------------------------------------------- | ------------------------- | ------------------------------------------------------------ |
| 前置条件                                                 | 获取对象的监视器锁        | 调用 Lock.lock() 获取锁调用 Lock.newCondition() 获取 Condition 对象 |
| 调用方法                                                 | 直接调用如：object.wait() | 直接调用如：condition.await()                                |
| **等待队列个数**                                         | 一个                      | 多个                                                         |
| 当前线程释放锁并进入等待队列                             | 支持                      | 支持                                                         |
| 当前线程释放锁并进入等待队列，在等待状态中不**响应中断** | 不支持                    | 支持                                                         |
| 当前线程释放锁并进入超时等待状态                         | 支持                      | 支持                                                         |
| **当前线程释放锁并进入等待状态到将来的某个时间**         | 不支持                    | 支持：即 awaitUntil()                                        |
| 唤醒等待队列中的一个线程                                 | 支持                      | 支持                                                         |
| 唤醒等待队列中的全部线程                                 | 支持                      | 支持                                                         |

## 二、Condition 的 JavaDoc 分析

**Condition 相当于将 Object 类中的 monitor 方法（wait/notify/notifyAll）方法插入到不同对象中，以实现==让每个对象有多个等待集合== wait-sets 的效果**。通过任意一个 Lock 实现将他们组合起来，其中使用 Lock 替换掉 synchronized 修饰方法/代码块的使用，使用 Condition 替换掉 Object 中的那些监视器 monitor 方法。

> 传统方式是调用 wait 之后，所有线程都会进入该对象的唯一的一个等待集合中，但是 condition 可以将某几个线程放在一个等待集合中，另几个放在另一个等待集合。

Condition（也称之为条件队列/条件变量）提供了一种方式让线程挂起执行（类似于 wait）直到被另一个线程通知，这里的另一个线程满足某些状态条件变成了 true。由于在不同线程中发生这种共享状态信息的访问，因此其需要被保护，所以一个 lock 的某种形式需要关联到 condition 上。等待一个 condition 提供的关键属性是它以原子方式释放关联的锁并挂起当前线程，就像 Object 的 wait 方法。

**Condition 实例本质上绑定到 lock 实例上。使用 `Lock.newCondition()`可以获取特定 Lock 实例的 Condition 实例。**线程调用 Condition 中方法前需要首先获取到 Condition 对象关联的锁。

> 因此一个 lock 对象和 condition 对象是一对多的关系，因为每次通过 newCondition() 方法就获得一个 condition 对象，该方法在 Lock 接口的最后。

Condition 实现可以提供不同于 Object 监视方法的行为和语义，例如**保证通知的顺序，或者在执行通知时不需要持有锁。**

**Condition 实例只是普通对象，它们本身可以用作 synchronized 语句中的目标**，并且可以调用它们自己的监视器 `Object#wait()` 和  `Object#notify()` 方法。获取 Condition 实例的监视锁或使用其监视方法与获取与该 Condition}关联的 lock 或使用其 await 和 signal 方法没有指定的关系。**但是建议不要以这种方式使用 Condition实例**，除了它们自己的实现之外。

> 即 condition 也是一个普通的对象，其自身也是有 monitor 等，所以我们获取 condition 的 monitor 锁和使用 Condition 中的 wait/signal 方法没有任何关系。



**实施注意事项**

**在等待 Condition 时，通常允许发生虚假唤醒，作为对底层平台语义的让步。**

这对大多数应用程序几乎没有实际影响，因为 **Condition 应该始终在循环中等待**，测试正在等待的状态成分。实现可以自由地消除虚假唤醒的可能性，

但建议应用程序程序员始终假设它们可以发生，因此始终在循环中等待。条件等待的三种形式（可中断、不可中断和定时）在某些平台上的易实现性和性能特征可能有所不同。尤其是，可能很难提供这些特性并维护特定的语义，例如排序保证。此外，中断线程的实际挂起的能力可能并不总是能够在所有平台上实现。

因此，实现不需要为所有三种形式的等待定义完全相同的保证或语义，也不需要支持中断线程的实际挂起。

一个实现需要清楚地记录每个等待方法提供的语义和保证，当一个实现确实支持线程挂起的中断时，它必须遵守这个接口中定义的中断语义。

由于中断通常意味着取消，并且对中断的检查通常不经常发生，所以**实现可能会倾向于响应中断而不是正常的方法返回**。即使可以显示中断发生在另一个可能已解除线程阻塞的操作之后，也是如此。实现应该记录这种行为。

**常用概念**：

- 等待队列是一个单向 FIFO 队列，队列每个节点都包含了一个线程引用，该线程是在 Condition 对象上等待的线程；该等待队列和 AQS 中的同步队列，都是采用 AQS#Node 静态内部类；
- 一个 ConditionObject 拥有首节点(fisrtWaiter)和尾节点(lastWaiter)；【ConditionObject 类是 Condition 接口在 AQS 中内部实现类】
- 如果一个线程调用了 Condition.await() 方法，那么该线程将会释放锁（从同步队列中移除），构造成节点加入等待队列，等待被唤醒；
- 如果一个线程调用了 Condition.signal() 方法，那么该线程将会被唤醒（从等待队列中移除），构造成节点加入同步队列，尝试重新获取同步状态；



## 三、主要方法分析

![image-20210423212436526](五、Condition.resource/image-20210423212436526-1620313601896.png)

### （一）await() 方法

**该方法会导致当前线程处于等待状态，直到被调用 signal 或者该线程被中断了。**
**调用 await 之后，与此 Condition 关联的锁被原子释放，当前线程无法进行线程调度，并且处于休眠状态**，直到发生以下四种情况中的一种：

- 另一个线程调用当前 Condition 的 signal 方法，而当前线程恰好被选为要唤醒的线程； 
- 另一个线程为此 Condition 调用 signalAll 方法；   
- 另一个线程中断了当前线程，并中断线程支持暂停；


- 发生「虚假唤醒」。

在该方法返回之前（继续往下执行），当前线程必须重新获取与该 Condition 相关联的锁。当线程返回时保证其持有此锁。    

如果当前线程：在进入此方法时设置了中断状态；或者在等待时被中断了，并且支持中断线程挂起，则抛出 InterruptedException ，并且当前线程的中断状态被清除。在第一种情况下，没有规定是否在锁定之前进行中断测试释放。
**实施注意事项**

调用此方法时，假定当前线程持有与此 Condition 关联的锁。这取决于执行情况，以确定是否是这种情况，如果不是，如何应对。通常，会抛出异常（例如  IllegalMonitorStateException），实现必须记录该事实。
与响应信号的正常方法返回相比，实现更倾向于响应中断。在这种情况下，实现必须确保信号被重定向到另一个等待线程（如果有）。



### （二）awaitUninterruptibly()

相比上面的 await() 方法，该方法等待不可中断。

### （三）long awaitNanos(long nanosTimeout) throws InterruptedException;

该方法同样使当前线程等待，直到调用 signal 或中断，或者经过指定的等待时间。同时相比 `await()` 方法线程**被唤醒的情况增加一种：指定的等待时间已经过去了**

该方法会返回一个近似的纳秒时间，该时间是剩余的响应时间（如设定 500 纳秒，但是 200 纳秒之后就响应了，则返回值为 300）。如果返回值小于或者等于 0 则表示超时了。

###  （四）boolean await(long time, TimeUnit unit) throws InterruptedException;

同上，只是上面时间只能设定为纳秒而已。

###    （五）void signal();

该方法会唤醒**其中一个**等待线程。

如果有任何线程正在等待此条件，则会选择一个线程进行唤醒。然后该线程必须在从  await 返回之前重新获取锁。

### （六）void signalAll();

**唤醒所有的等待线程**

**任何在该 condition 上等待的线程都会被唤醒**，每个线程在从 await 方法上返回前必须重新获取锁

> 即全部被唤醒之后，只有一个线程能获取锁，所以只有一个线程能从 await 上返回，其他线程还是处于等待状态。因此线程被唤醒和可以继续执行不是一回事。

## 三、Condition 使用样例

有一个支持 put 和  take 方法的有界缓冲区。如果在空缓冲区上尝试 take，则线程将阻塞，直到某个项变为可用；如果在满缓冲区上尝试 put，则线程将阻塞，直到某个空间变为可用。希望保持 put 线程和 take 线程在单独的等待集中等待，这样当缓冲区中的项或空间可用时，我们就可以使用只通知单个线程的优化。 这可以通过使用两个 Condition 实例来实现。【来自 JavaDoc 文档】

> 实例代码来自 JavaDoc 文档，java.util.concurrent.ArrayBlockingQueue 类提供了这些功能，因此实际上可以直接使用类而不是使用下面代码。【见 ArrayBlockingQueue 类源码分析】

```java
package com.gjxaiou.condition;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @Author GJXAIOU
 * @Date 2021/3/19 20:15
 */
public class MyTest1 {

    final Lock lock = new ReentrantLock();
    // 调用同一个 lock 实例生成的两个 condition 对象。通常 Condition 都作为成员变量
    final Condition notFull = lock.newCondition();
    final Condition notEmpty = lock.newCondition();

    final Object[] items = new Object[100];
    int putptr, takeptr, count;

    public void put(Object x) throws InterruptedException {
        // 首先要获取锁
        lock.lock();
        try {
            while (count == items.length) {
           // 调用 await 进入等待状态，同时释放锁。同时放在 while 循环中，保证其它线程通过 signal
           // 方法唤醒该线程，则该线程需要和其他线程争抢说，争抢到了才能执行。
                notFull.await();
            }
            items[putptr] = x;
            if (++putptr == items.length) {
                putptr = 0;
            }
            ++count;
            // 通知另一个线程，可以取出了。
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }

    public Object take() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0) {
                notEmpty.await();
            }
            Object x = items[takeptr];
            if (++takeptr == items.length) {
                takeptr = 0;
            }
            --count;
            notFull.signal();
            return x;
        } finally {
            lock.unlock();
        }
    }
}
```

### Condition 主要方法测试和应用场景

该类实现的例子的描述：实现对容器的放置和取出的操作，在这个操作过程中保证在某一个时刻只能有一个线程在进行使用和操作。

整体的过程就是在模拟synchronized关键字和wait方法以及notify方法（只不过上面是从字节码层面来解决这个问题）

该类实现的例子的描述：实现对容器元素的放置和取出的操作，在多个线程操作过程中保证在某一个时刻只能有一个线程在进行使用和操作。使用condition来实现操作（这个concurrentHashMap，以及使用Collections工具类来实现集合实现并发安全的操作）。

put 方法不断往后放置元素，take 方法不断往后取元素，然后置为 NULL，如果满了从头开始。如果整个数组全部满了之后则 put 线程需要等待 take 线程取出一个元素，反之一样。

=> 如果 put 线程数和 get 线程数相同，则最终结果均为 Null，如果数目不一致：如  10 个 put, 8 个 take，则最后两个有元素，如果 8 个 put，10 个 take，则最终结果全部为 null，但是程序无法退出，因为第九个/十个 take 线程都陷入 await 的循环中了。因为 8 个唤醒线程已经用完了，没有线程使用 signal 来唤醒了。

```java
package com.gjxaiou.condition;

import javax.lang.model.element.NestingKind;
import java.util.Arrays;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
import java.util.stream.IntStream;

/**
 * @Author GJXAIOU
 * @Date 2021/3/20 12:52
 */
public class MyTest2 {

    public static void main(String[] args) {
        // 创建线程
        BoundedContainer boundedContainer = new BoundedContainer();

        IntStream.range(0, 10).forEach(i -> new Thread(() -> {
            try {
                boundedContainer.take();
            } catch (InterruptedException exception) {
                exception.printStackTrace();
            }
        }).start());
        
        IntStream.range(0, 10).forEach(i -> new Thread(() -> {
            try {
                boundedContainer.put("hello");
            } catch (InterruptedException exception) {
                exception.printStackTrace();
            }
        }).start());
    }
}

class BoundedContainer {

    // 定义一个有界数组存放元素
    private String[] elements = new String[10];

    // 定义一个  lock 对象，因为针对该数组不能有任意两个数组同时进行操作，无论是放放还是放拿还是拿拿都不行
    private Lock lock = new ReentrantLock();

    // 因为针对两个动作：拿和放都需要一定的条件，需要需要两个 condition
    // 全空的时候 拿线程不能进行，全满的时候放置线程不能进行
    private Condition notEmptyCondition = lock.newCondition();
    private Condition notFullCondition = lock.newCondition();

    // 判断数组元素是不是满了，该值表示数组中已有元素数量
    private int elementCount;

    // 接下来放置元素的位置和接下来取元素位置
    private int putIndex;
    private int takeIndex;


    public void put(String element) throws InterruptedException {
        // 禁止任意两个线程同时执行 put 方法，所以首先要获取锁
        this.lock.lock();

        try {
            // 首先如果数组满了就需要等待
            while (this.elementCount == this.elements.length) {
                // 被 signal 唤醒之后会再次判断，如果为真则表示有位子放置了
                notFullCondition.await();
            }
            elements[putIndex] = element;
            if (++putIndex == this.elements.length) {
                putIndex = 0;
            }
            ++elementCount;
            System.out.println("put method" + Arrays.toString(elements));

            // 通知取元素可以取了
            notEmptyCondition.signal();

        } finally {
            lock.unlock();
        }
    }

    public String take() throws InterruptedException {
        lock.lock();

        try {
            while (this.elementCount == 0) {
                notEmptyCondition.await();
            }
            String res = elements[takeIndex];
            elements[takeIndex] = null;
            if (++takeIndex == this.elements.length) {
                takeIndex = 0;
            }
            --elementCount;
            System.out.println("take method" + Arrays.toString(elements));

            // 通知放置元素线程
            notFullCondition.signal();
            return res;
        } finally {
            lock.unlock();
        }
    }
}
```



### （1）概念认识

先引出简单的认识，其实对比同步队列来说，很好理解，实际上更加简单

- 

### （2）等待队列结构图

实际上，Condition的实现是在AQS中内部类ConditionObject实现Condition具体实现的：

　　　　　　　　[![img](https://img2020.cnblogs.com/blog/1352849/202012/1352849-20201223214958565-361316620.png)](https://img2020.cnblogs.com/blog/1352849/202012/1352849-20201223214958565-361316620.png)

等待队列的结构图如下图，相比较同步队列而言：

- **等待队列来说更加简单，是单向FIFO队列；**
- **Condition拥有首尾节点引用，新增节点直接nextWaiter指向即可，这个过程不需要CAS保证，因为调用Condition.await()方法肯定是获取了锁的线程，也就是说该过程是来保证线程安全的。**

　　　　　　[![img](https://img2020.cnblogs.com/blog/1352849/202012/1352849-20201223214720533-992570661.png)](https://img2020.cnblogs.com/blog/1352849/202012/1352849-20201223214720533-992570661.png)

实际上，**AQS同步器只拥有一个同步队列，但却有多个Condition等待队列**，如下图。

　　　　　　　　[![img](https://img2020.cnblogs.com/blog/1352849/202012/1352849-20201223220123984-923368951.png)](https://img2020.cnblogs.com/blog/1352849/202012/1352849-20201223220123984-923368951.png)　　　　　　　　　　　　　　

## 2、await()方法实现解析

当调用await()方法时，相当于同步队列的首节点（获取了锁的节点）移动到了Condition的等待队列中。

　　　[![img](https://img2020.cnblogs.com/blog/1352849/202012/1352849-20201223220817180-735315232.png)](https://img2020.cnblogs.com/blog/1352849/202012/1352849-20201223220817180-735315232.png)

更具体来说是**首先调用await()方法之前肯定是能获取到同步状态的线程，也就是同步队列中首节点，之后调用await()方法由将释放锁，进入等待队列**。

分析源码（重点部分都已经注释，结合图应该更好理解）：

**1）调用await()方法，通过addConditionWaiter()方法加入等待线程，然后释放全部同步状态**

**2）进入while循环，判断是否已经移动到同步队列中，如果已经被移动到同步队列中则说明线程已经被唤醒（signal）；**

**3）接下来尝试获取竞争同步状态，即调用acquireQueue方法**



```
   　　public final void await() throws InterruptedException {
            if (Thread.interrupted())
                throw new InterruptedException();
            // addConditionWaiter()方法当前线程加入等待队列
            Node node = addConditionWaiter();
            // 调用await()方法后释放锁（同步状态）
            int savedState = fullyRelease(node);
            int interruptMode = 0;
            // 检查node是否在同步队列中，不是的话说明已经获取到锁
            //  LockSupport.unpark唤醒线程后，从这里返回，此时已经在SyncQueue同步队列中，退出循环
            // 从这里也可以看出，也是经典的等待/通知模式
            while (!isOnSyncQueue(node)) {
                // 阻塞当前线程
                LockSupport.park(this);
                // 在调用signal前抛出中断异常，或者调用之后中断，都退出循环
                // THROW_IE if interrupted  before signalled, REINTERRUPT if after signalled
                if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                    break;
            }
            // 被唤醒后的线程重新尝试竞争获取同步状态
            if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
                interruptMode = REINTERRUPT;
            if (node.nextWaiter != null) // clean up if cancelled
                unlinkCancelledWaiters();
            if (interruptMode != 0)
                reportInterruptAfterWait(interruptMode);
        }
```

结构流程如下图：

[![img](https://img2020.cnblogs.com/blog/1352849/202012/1352849-20201223220913293-1701436389.png)](https://img2020.cnblogs.com/blog/1352849/202012/1352849-20201223220913293-1701436389.png)

## 3、signal()方法实现解析

调用signal方法将会唤醒等待队列中等待时间最长的节点（首节点），在唤醒节点之前，会将节点移动到同步队列中

　　　　 [![img](https://img2020.cnblogs.com/blog/1352849/202012/1352849-20201223221949802-1243441912.png)](https://img2020.cnblogs.com/blog/1352849/202012/1352849-20201223221949802-1243441912.png)

具体来说当前线程获取到了锁，接着获取等待队列的首节点，将其移动到同步队列中，并且唤醒节点中的线程。

分析源码（重点部分都已经注释，结合图应该更好理解）：

**1）进入signal()方法，调用doSignal(Node node)方法移动到同步队列中，并唤醒节点中线程**



```
　　public final void signal() {
            // 判断当前线程是否持有获得锁
            if (!isHeldExclusively())
                throw new IllegalMonitorStateException();
            Node first = firstWaiter;
            if (first != null)
                // 移动到同步队列中，并且唤醒节点中的线程
                doSignal(first);
        }
　　private void doSignal(Node first) {
            do {
                // 如果等待队列中只有一个节点（即首节点），则唤醒首节点后lastWaiter置空
                if ( (firstWaiter = first.nextWaiter) == null)
                    lastWaiter = null;
                // 否则获取等待队列中的首节点，即next域断开置空
                first.nextWaiter = null;
            } while (!transferForSignal(first) &&
                     (first = firstWaiter) != null);
        }
```

**2）doSignal(Node node)方法中调用transferForSignal(Node node)，通过调用enq(Node node)方法(这里其实就是同步队列的入队enq(Node node)方法)，等待队列中的头结点线程安全地移动到同步队列，当节点移动到同步队列后，当前线程将会被唤醒（LockSupport.unpark(node.thread)）。**



```
    final boolean transferForSignal(Node node) {
        /*
         * If cannot change waitStatus, the node has been cancelled.
         */
        // 如果没有正确设置等待状态为初始状态准备加入同步队列中，则返回，当前节点状态为Node.CONDITION
        if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
            return false;
        // 将等待队列中的头结点移动到同步队列中, 返回已经加入的当前node在同步队列中前节点
        Node p = enq(node);
        int ws = p.waitStatus;
        if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
            // 唤醒当前node线程，返回while(isOnSynQueue(Node node))处，退出循环
            LockSupport.unpark(node.thread);
        return true;
    }
```

流程结构如下图：　

[![img](https://img2020.cnblogs.com/blog/1352849/202012/1352849-20201223222355757-830289752.png)](https://img2020.cnblogs.com/blog/1352849/202012/1352849-20201223222355757-830289752.png)

 