# Lock

[TOC]

## Lock 锁机制深入详解

jdk 1.5 之前对于对象同步只能使用 synchronized 关键字，之后引入了 Lock 接口（java.util.concurrency.locks.Lock.java），可以实现对象同步。

**首先针对 Lock 类的 JavaDoc 进行分析**：

> Lock 实现提供了比使用 synchronized 方法和语句更广泛的锁定操作。它们允许更灵活的结构，可能具有完全不同的属性，并且可能**支持多个关联的 Condition 对象**（也在 locks 中，后面讲解）。
>
> **锁是一种工具，用于控制多个线程对共享资源的访问**。通常情况下，锁提供对共享资源的独占访问：一次只有一个线程可以获取锁，**对共享资源的所有访问都要求首先获取锁**。但是，有些锁可能允许并发访问共享资源，例如 ReadWriteLock 的读锁。
>
> > 注：通常情况下， Lock 是一种排他性的锁，即同一时刻只能有一个线程拥有这把锁，然后访问该锁控制的资源，如果有其它线程想访问该共享资源则只能等待持有该锁的线程执行完或者抛出异常从而释放该锁，然后去争抢这把锁。 当时这种方式对于共享资源划分力度不够，例如通常资源读的次数大于写次数，当多个线程对同一个资源都是读取，本质上都是不需要上锁的，所以通过 ReadWriteLock，读和写线程分别获取读锁和写锁。具体后续再分析。
>
> 使用 synchronized 方法或语句可以访问与每个对象关联的隐式监视锁，但会强制以块结构的方式获取和释放所有锁：当获取多个锁时，它们必须按相反的顺序释放，所有锁都必须在获得它们的相同词法（作用域）范围内释放。
>
> 虽然 synchronized 方法和语句的作用域机制使使用监视器锁编程更加容易，并有助于避免许多涉及锁的常见编程错误，但**有时需要以更灵活的方式使用锁**。例如，一些遍历并发访问数据结构的算法需要使用「head - over - head」或「链锁定」：先获取节点 A 的锁，然后获取节点 B，然后释放 A 和获取 C，然后释放 B 和获取 D，依此类推。 **Lock 接口的实现允许在不同的作用域中获取和释放一个锁，并允许以任何顺序获取和释放多个锁**，从而允许使用此类技术。
>
> 随着这种灵活性的增加，也带来了额外的责任。缺少块结构锁定将删除 synchronized 方法和语句所发生的锁的自动释放。在大多数情况下，应使用以下用法：
>
> ```java
> Lock l = ...;
> l.lock();
> // 即在 try 中访问锁保护的资源，并且在 finally 中释放锁
> try {
>    // access the resource protected by this lock
> } finally {
>    l.unlock();
> }
> ```
>
> **当锁定和解锁发生在不同的作用域中时，必须注意确保在锁定期间执行的所有代码都受到 try finally 或 try catch 的保护，以确保在必要时释放锁定。**
>
> **Lock 实现通过提供一个非阻塞的获取锁的尝试 `tryLock()`，一个获取可以中断的锁的尝试lockInterruptibly，以及试图获取可以超时的锁 `tryLock（long，TimeUnit)`**。
>
> Lock 类还可以提供与隐式监视锁完全不同的行为和语义，例如保证排序、不可重入使用或死锁检测。如果一个实现提供了这样的专门语义，那么该实现必须记录这些语义。
>
> 注意 **Lock 实例只是普通对象**，它们本身可以用作  synchronized 语句中的目标。**获取 lock 实例的监视器锁与调用该实例的任何 lock 方法都没有指定的关系**。建议您不要以这种方式使用 Lock 实例，除非在它们自己的实现中。
>
> 除非另有说明，否则为任何参数传递 null 值将导致引发 NullPointerException。
>
> <h4>内存同步</h4>
>
> 所有的 Lock实现都必须执行内置监视器锁提供的相同内存同步语义，如 [Java 语言规范（17.4内存模型）]( http://docs.oracle.com/javase/specs/jls/se7/html/jls-17.html )
>
> 成功的 lock 操作与成功的<em>lock</em>动作具有相同的内存同步效果。
>
> 成功的 unlock 操作与成功的<em>unlock</em>动作具有相同的内存同步效果。
>
> 不成功的锁定和解锁操作以及可重入的锁定/解锁动作不需要任何内存同步效果。
>
> <h4>实现上的注意事项</h4>
>
> 锁获取的三种形式（可中断、不可中断和定时）在性能特征、顺序保证或其他实现质量方面可能有所不同。此外，在给定的 lock 类中可能无法中断正在进行的获取锁的能力。因此，实现不需要为所有三种锁获取形式定义完全相同的保证或语义，也不需要支持正在进行的锁获取的中断。需要一个实现来清楚地记录每个锁定方法提供的语义和保证。它还必须遵守此接口中定义的中断语义，只要支持锁获取中断：要么完全中断，要么仅中断方法入口。
>
> 由于中断通常意味着取消，并且对中断的检查通常是不经常的，所以实现可能倾向于响应中断，而不是普通的方法返回。即使可以显示在另一个操作可能已解除阻止线程之后发生的中断，这也是正确的。实现应该记录此行为。

**JavaDoc 的总结**：

- Lock 支持关联多个 Condition
- 需要自行以反加锁的顺序释放锁；
- Lock 接口的实现允许在不同的作用域中获取和释放一个锁，并允许以任何顺序获取和释放多个锁；
- 当锁定和解锁发生在不同的作用域中时，必须注意确保在锁定期间执行的所有代码都受到 try finally 或 try catch 的保护，以确保在必要时释放锁定。
- Lock 实现通过提供一个非阻塞的获取锁的尝试 `tryLock()`，一个获取可以中断的锁的尝试lockInterruptibly，以及试图获取可以超时的锁 `tryLock（long，TimeUnit)`

```java

public interface Lock {

    /**
     *  获取锁
     */
    void lock();

    /**
    获取锁（可中断）
    
    获取锁，除非当前线程被 interrupted。
	获取锁（如果可用）并立即返回。
	如果锁不可用，则当前线程将被禁用以进行线程调度，并处于休眠状态，直到发生以下两种情况之一：
	- 这个锁是由当前线程获取的；
	- 或者某些其他线程中断当前线程（的休眠状态），并且支持锁获取的中断。

	如果当前线程：
	- 在进入此方法时设置了中断状态；
	- 或者在获取锁时被 interrupted，并且支持中断获取锁，则抛出 InterruptedException，并清除当前线程的中断状态。

	实施注意事项

	在某些实现中中断锁获取的能力可能是不可能的，并且如果可能的话可能是一个昂贵的操作。程序员应该意识到情况可能是这样的。在这种情况下，实现应该记录下来。

	实现有利于响应中断，而不是普通的方法返回。

	{@code Lock}实现可能能够检测到锁的错误使用，例如可能导致死锁的调用，并且在这种情况下可能抛出（未检查的）异常。环境和异常类型必须由{@code Lock}实现记录。
     
     * @throws InterruptedException if the current thread is
     *         interrupted while acquiring the lock (and interruption
     *         of lock acquisition is supported)
     */
    void lockInterruptibly() throws InterruptedException;

    /**
    尝试获取锁，如果获取到返回 true，否则返回 false。
    
    只有在调用时锁是空闲的情况下才获取锁。
	获取锁（如果可用），并立即返回值 true。如果锁不可用，则此方法将立即返回值 false。
    该方法的典型使用示例：

     // 定义一个 lock 对象
      Lock lock = ...;
      if (lock.tryLock()) {
        try {
          // manipulate protected state
        } finally {
          lock.unlock();
        }
      } else {
        // perform alternative actions
      }}
     
     此用法确保在获取锁时将其解锁，并且在未获取锁时不会尝试解锁。因为if 之后在 finally 中解锁了。
     */
    boolean tryLock();

    /**
    尝试获取锁，如果没有获取到锁则等待一段时间，在等待时间之内还没有获取到锁则返回 false。
    
    如果锁在给定的等待时间内空闲并且当前线程未被终中断，则获取该锁。
    如果锁可用（获取到），则此方法立即返回值 true。
    如果锁不可用（没有获取到），则当前线程将被禁用以进行线程调度，并处于休眠状态，直到发生以下三种情况之一：
	- 锁由当前线程获取；
	- 另一个线程中断当前线程，并且支持锁获取中断；
	- 指定的等待时间已过

	如果获取了锁，则返回值 true。

	如果当前线程：在进入此方法时设置了中断状态；或在获取锁时是中断的，并且支持中断获取锁，然后 InterruptedException 被抛出，当前线程的中断状态被清除。

	如果指定的等待时间已过，则值返回 false

	如果时间小于或等于零，则该方法根本不会等待。

    实施考虑1
	在某些实现中中断锁获取的能力可能是不可能的，并且如果可能的话可能是一个昂贵的操作。
	程序员应该意识到情况可能是这样的。在这种情况下，实现应该记录下来。
	实现可以支持响应中断，而不是普通的方法返回，或者报告超时。
	Lock实现可能能够检测到锁的错误使用，例如可能导致死锁的调用，并且在这种情况下可能抛出（未检查的）异常。
	环境和异常类型必须由{@code Lock}实现记录。  
     */
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;


    /**
     释放锁
     */
    void unlock();

    /**
    返回一个 Condition 实例，该实例绑定到调用了 newCondition() 方法的 Lock 实例对象上。
	在等待 condition 之前，锁必须由当前线程持有。
    对 Condition 的 await（）方法的调用将会在等待之前自动释放锁，并在等待返回之前重新获取锁
	
	实施注意事项
    {@link Condition}实例的确切操作取决于{@code Lock}实现，并且必须由该实现记录。
    如果当前 Lock 实现不支持 conditions  则抛出 UnsupportedOperationException。
     */
    Condition newCondition();
}
```

## ReentrantLock

是 Lock  接口下面最重要的实现，re 即再次，entrant 是进入，因此 ReentrantLock 为可重入锁。重入锁，是指一个线程获取锁之后再尝试获取锁时会自动获取锁。

![image-20210421193815709](四、Lock 锁.resource/image-20210421193815709.png)

### 主要内部类

ReentrantLock 中主要定义了三个内部类：Sync、NonfairSync、FairSync。

```java
abstract static class Sync extends AbstractQueuedSynchronizer {}

static final class NonfairSync extends Sync {}

static final class FairSync extends Sync {}
```

（1）抽象类Sync实现了AQS的部分方法；

（2）NonfairSync实现了Sync，主要用于非公平锁的获取；

（3）FairSync实现了Sync，主要用于公平锁的获取。

在这里我们先不急着看每个类具体的代码，等下面学习具体的功能点的时候再把所有方法串起来。

### 主要属性

```java
private final Sync sync;
```

主要属性就一个sync，它在构造方法中初始化，决定使用公平锁还是非公平锁的方式获取锁。

### 主要构造方法

```java
// 默认构造方法
public ReentrantLock() {
    sync = new NonfairSync();
}
// 自己可选择使用公平锁还是非公平锁
public ReentrantLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
}
```

（1）默认构造方法使用的是非公平锁；

（2）第二个构造方法可以自己决定使用公平锁还是非公平锁；

上面我们分析了ReentrantLock的主要结构，下面我们跟着几个主要方法来看源码。

### lock()方法

彤哥贴心地在每个方法的注释都加上方法的来源。

#### 公平锁

这里我们假设ReentrantLock的实例是通过以下方式获得的：

```java
ReentrantLock reentrantLock = new ReentrantLock(true);
```

下面的是加锁的主要逻辑：

```java
// ReentrantLock.lock()
public void lock() {
    // 调用的sync属性的lock()方法
    // 这里的sync是公平锁，所以是FairSync的实例
    sync.lock();
}
// ReentrantLock.FairSync.lock()
final void lock() {
    // 调用AQS的acquire()方法获取锁
    // 注意，这里传的值为1
    acquire(1);
}
// AbstractQueuedSynchronizer.acquire()
public final void acquire(int arg) {
    // 尝试获取锁
    // 如果失败了，就排队
    if (!tryAcquire(arg) &&
        // 注意addWaiter()这里传入的节点模式为独占模式
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
// ReentrantLock.FairSync.tryAcquire()
protected final boolean tryAcquire(int acquires) {
    // 当前线程
    final Thread current = Thread.currentThread();
    // 查看当前状态变量的值
    int c = getState();
    // 如果状态变量的值为0，说明暂时还没有人占有锁
    if (c == 0) {
        // 如果没有其它线程在排队，那么当前线程尝试更新state的值为1
        // 如果成功了，则说明当前线程获取了锁
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            // 当前线程获取了锁，把自己设置到exclusiveOwnerThread变量中
            // exclusiveOwnerThread是AQS的父类AbstractOwnableSynchronizer中提供的变量
            setExclusiveOwnerThread(current);
            // 返回true说明成功获取了锁
            return true;
        }
    }
    // 如果当前线程本身就占有着锁，现在又尝试获取锁
    // 那么，直接让它获取锁并返回true
    else if (current == getExclusiveOwnerThread()) {
        // 状态变量state的值加1
        int nextc = c + acquires;
        // 如果溢出了，则报错
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        // 设置到state中
        // 这里不需要CAS更新state
        // 因为当前线程占有着锁，其它线程只会CAS把state从0更新成1，是不会成功的
        // 所以不存在竞争，自然不需要使用CAS来更新
        setState(nextc);
        // 当线程获取锁成功
        return true;
    }
    // 当前线程尝试获取锁失败
    return false;
}
// AbstractQueuedSynchronizer.addWaiter()
// 调用这个方法，说明上面尝试获取锁失败了
private Node addWaiter(Node mode) {
    // 新建一个节点
    Node node = new Node(Thread.currentThread(), mode);
    // 这里先尝试把新节点加到尾节点后面
    // 如果成功了就返回新节点
    // 如果没成功再调用enq()方法不断尝试
    Node pred = tail;
    // 如果尾节点不为空
    if (pred != null) {
        // 设置新节点的前置节点为现在的尾节点
        node.prev = pred;
        // CAS更新尾节点为新节点
        if (compareAndSetTail(pred, node)) {
            // 如果成功了，把旧尾节点的下一个节点指向新节点
            pred.next = node;
            // 并返回新节点
            return node;
        }
    }
    // 如果上面尝试入队新节点没成功，调用enq()处理
    enq(node);
    return node;
}
// AbstractQueuedSynchronizer.enq()
private Node enq(final Node node) {
    // 自旋，不断尝试
    for (;;) {
        Node t = tail;
        // 如果尾节点为空，说明还未初始化
        if (t == null) { // Must initialize
            // 初始化头节点和尾节点
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            // 如果尾节点不为空
            // 设置新节点的前一个节点为现在的尾节点
            node.prev = t;
            // CAS更新尾节点为新节点
            if (compareAndSetTail(t, node)) {
                // 成功了，则设置旧尾节点的下一个节点为新节点
                t.next = node;
                // 并返回旧尾节点
                return t;
            }
        }
    }
}
// AbstractQueuedSynchronizer.acquireQueued()
// 调用上面的addWaiter()方法使得新节点已经成功入队了
// 这个方法是尝试让当前节点来获取锁的
final boolean acquireQueued(final Node node, int arg) {
    // 失败标记
    boolean failed = true;
    try {
        // 中断标记
        boolean interrupted = false;
        // 自旋
        for (;;) {
            // 当前节点的前一个节点
            final Node p = node.predecessor();
            // 如果当前节点的前一个节点为head节点，则说明轮到自己获取锁了
            // 调用ReentrantLock.FairSync.tryAcquire()方法再次尝试获取锁
            if (p == head && tryAcquire(arg)) {
                // 尝试获取锁成功
                // 这里同时只会有一个线程在执行，所以不需要用CAS更新
                // 把当前节点设置为新的头节点
                setHead(node);
                // 并把上一个节点从链表中删除
                p.next = null; // help GC
                // 未失败
                failed = false;
                return interrupted;
            }
            // 是否需要阻塞
            if (shouldParkAfterFailedAcquire(p, node) &&
                // 真正阻塞的方法
                parkAndCheckInterrupt())
                // 如果中断了
                interrupted = true;
        }
    } finally {
        // 如果失败了
        if (failed)
            // 取消获取锁
            cancelAcquire(node);
    }
}
// AbstractQueuedSynchronizer.shouldParkAfterFailedAcquire()
// 这个方法是在上面的for()循环里面调用的
// 第一次调用会把前一个节点的等待状态设置为SIGNAL，并返回false
// 第二次调用才会返回true
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    // 上一个节点的等待状态
    // 注意Node的waitStatus字段我们在上面创建Node的时候并没有指定
    // 也就是说使用的是默认值0
    // 这里把各种等待状态再贴出来
    //static final int CANCELLED =  1;
    //static final int SIGNAL    = -1;
    //static final int CONDITION = -2;
    //static final int PROPAGATE = -3;
    int ws = pred.waitStatus;
    // 如果等待状态为SIGNAL(等待唤醒)，直接返回true
    if (ws == Node.SIGNAL)
        return true;
    // 如果前一个节点的状态大于0，也就是已取消状态
    if (ws > 0) {
        // 把前面所有取消状态的节点都从链表中删除
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        pred.next = node;
    } else {
        // 如果前一个节点的状态小于等于0，则把其状态设置为等待唤醒
        // 这里可以简单地理解为把初始状态0设置为SIGNAL
        // CONDITION是条件锁的时候使用的
        // PROPAGATE是共享锁使用的
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
// AbstractQueuedSynchronizer.parkAndCheckInterrupt()
private final boolean parkAndCheckInterrupt() {
    // 阻塞当前线程
    // 底层调用的是Unsafe的park()方法
    LockSupport.park(this);
    // 返回是否已中断
    return Thread.interrupted();
}
```

看过之前彤哥写的【[死磕 java同步系列之自己动手写一个锁Lock](https://mp.weixin.qq.com/s/1RU5jh7UcXGtKlae8tusVA)】的同学看今天这个加锁过程应该思路会比较清晰。

下面我们看一下主要方法的调用关系，可以跟着我的 → 层级在脑海中大概过一遍每个方法的主要代码：

```java
ReentrantLock#lock()
->ReentrantLock.FairSync#lock() // 公平模式获取锁
  ->AbstractQueuedSynchronizer#acquire() // AQS的获取锁方法
    ->ReentrantLock.FairSync#tryAcquire() // 尝试获取锁
    ->AbstractQueuedSynchronizer#addWaiter()  // 添加到队列
	  ->AbstractQueuedSynchronizer#enq()  // 入队
    ->AbstractQueuedSynchronizer#acquireQueued() // 里面有个for()循环，唤醒后再次尝试获取锁
      ->AbstractQueuedSynchronizer#shouldParkAfterFailedAcquire() // 检查是否要阻塞
      ->AbstractQueuedSynchronizer#parkAndCheckInterrupt()  // 真正阻塞的地方
```

获取锁的主要过程大致如下：

（1）尝试获取锁，如果获取到了就直接返回了；

（2）尝试获取锁失败，再调用addWaiter()构建新节点并把新节点入队；

（3）然后调用acquireQueued()再次尝试获取锁，如果成功了，直接返回；

（4）如果再次失败，再调用shouldParkAfterFailedAcquire()将节点的等待状态置为等待唤醒（SIGNAL）；

（5）调用parkAndCheckInterrupt()阻塞当前线程；

（6）如果被唤醒了，会继续在acquireQueued()的for()循环再次尝试获取锁，如果成功了就返回；

（7）如果不成功，再次阻塞，重复（3）（4）（5）直到成功获取到锁。

以上就是整个公平锁获取锁的过程，下面我们看看非公平锁是怎么获取锁的。

#### 非公平锁

```java
// ReentrantLock.lock()
public void lock() {
    sync.lock();
}
// ReentrantLock.NonfairSync.lock()
// 这个方法在公平锁模式下是直接调用的acquire(1);
final void lock() {
    // 直接尝试CAS更新状态变量
    if (compareAndSetState(0, 1))
        // 如果更新成功，说明获取到锁，把当前线程设为独占线程
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
// ReentrantLock.NonfairSync.tryAcquire()
protected final boolean tryAcquire(int acquires) {
    // 调用父类的方法
    return nonfairTryAcquire(acquires);
}
// ReentrantLock.Sync.nonfairTryAcquire()
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        // 如果状态变量的值为0，再次尝试CAS更新状态变量的值
        // 相对于公平锁模式少了!hasQueuedPredecessors()条件
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

相对于公平锁，非公平锁加锁的过程主要有两点不同：

（1）一开始就尝试CAS更新状态变量state的值，如果成功了就获取到锁了；

（2）在tryAcquire()的时候没有检查是否前面有排队的线程，直接上去获取锁才不管别人有没有排队呢；

总的来说，相对于公平锁，非公平锁在一开始就多了两次直接尝试获取锁的过程。

### lockInterruptibly()方法

支持线程中断，它与lock()方法的主要区别在于lockInterruptibly()获取锁的时候如果线程中断了，会抛出一个异常，而lock()不会管线程是否中断都会一直尝试获取锁，获取锁之后把自己标记为已中断，继续执行自己的逻辑，后面也会正常释放锁。

题外话：

线程中断，只是在线程上打一个中断标志，并不会对运行中的线程有什么影响，具体需要根据这个中断标志干些什么，用户自己去决定。

比如，如果用户在调用lock()获取锁后，发现线程中断了，就直接返回了，而导致没有释放锁，这也是允许的，但是会导致这个锁一直得不到释放，就出现了死锁。

```java
lock.lock();

if (Thread.currentThread().interrupted()) {
    return ;
}

lock.unlock();
```

当然，这里只是举个例子，实际使用肯定是要把lock.lock()后面的代码都放在try...finally...里面的以保证锁始终会释放，这里主要是为了说明线程中断只是一个标志，至于要做什么完全由用户自己决定。

### tryLock()方法

尝试获取一次锁，成功了就返回true，没成功就返回false，不会继续尝试。

```java
// ReentrantLock.tryLock()
public boolean tryLock() {
    // 直接调用Sync的nonfairTryAcquire()方法
    return sync.nonfairTryAcquire(1);
}
// ReentrantLock.Sync.nonfairTryAcquire()
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

tryLock()方法比较简单，直接以非公平的模式去尝试获取一次锁，获取到了或者锁本来就是当前线程占有着就返回true，否则返回false。

### tryLock(long time, TimeUnit unit)方法

尝试获取锁，并等待一段时间，如果在这段时间内都没有获取到锁，就返回false。

```java
// ReentrantLock.tryLock()
public boolean tryLock(long timeout, TimeUnit unit)
        throws InterruptedException {
    // 调用AQS中的方法
    return sync.tryAcquireNanos(1, unit.toNanos(timeout));
}
// AbstractQueuedSynchronizer.tryAcquireNanos()
public final boolean tryAcquireNanos(int arg, long nanosTimeout)
        throws InterruptedException {
    // 如果线程中断了，抛出异常
    if (Thread.interrupted())
        throw new InterruptedException();
    // 先尝试获取一次锁
    return tryAcquire(arg) ||
        doAcquireNanos(arg, nanosTimeout);
}
// AbstractQueuedSynchronizer.doAcquireNanos()
private boolean doAcquireNanos(int arg, long nanosTimeout)
        throws InterruptedException {
    // 如果时间已经到期了，直接返回false
    if (nanosTimeout <= 0L)
        return false;
    // 到期时间
    final long deadline = System.nanoTime() + nanosTimeout;
    final Node node = addWaiter(Node.EXCLUSIVE);
    boolean failed = true;
    try {
        for (;;) {
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return true;
            }
            nanosTimeout = deadline - System.nanoTime();
            // 如果到期了，就直接返回false
            if (nanosTimeout <= 0L)
                return false;
            // spinForTimeoutThreshold = 1000L;
            // 只有到期时间大于1000纳秒，才阻塞
            // 小于等于1000纳秒，直接自旋解决就得了
            if (shouldParkAfterFailedAcquire(p, node) &&
                nanosTimeout > spinForTimeoutThreshold)
                // 阻塞一段时间
                LockSupport.parkNanos(this, nanosTimeout);
            if (Thread.interrupted())
                throw new InterruptedException();
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

tryLock(long time, TimeUnit unit)方法在阻塞的时候加上阻塞时间，并且会随时检查是否到期，只要到期了没获取到锁就返回false。

### unlock()方法

释放锁。

```java
// java.util.concurrent.locks.ReentrantLock.unlock()
public void unlock() {
    sync.release(1);
}
// java.util.concurrent.locks.AbstractQueuedSynchronizer.release
public final boolean release(int arg) {
    // 调用AQS实现类的tryRelease()方法释放锁
    if (tryRelease(arg)) {
        Node h = head;
        // 如果头节点不为空，且等待状态不是0，就唤醒下一个节点
        // 还记得waitStatus吗？
        // 在每个节点阻塞之前会把其上一个节点的等待状态设为SIGNAL（-1）
        // 所以，SIGNAL的准确理解应该是唤醒下一个等待的线程
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
// java.util.concurrent.locks.ReentrantLock.Sync.tryRelease
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    // 如果当前线程不是占有着锁的线程，抛出异常
    if (Thread.currentThread() != getExclusiveOwnerThread())
        throw new IllegalMonitorStateException();
    boolean free = false;
    // 如果状态变量的值为0了，说明完全释放了锁
    // 这也就是为什么重入锁调用了多少次lock()就要调用多少次unlock()的原因
    // 如果不这样做，会导致锁不会完全释放，别的线程永远无法获取到锁
    if (c == 0) {
        free = true;
        // 清空占有线程
        setExclusiveOwnerThread(null);
    }
    // 设置状态变量的值
    setState(c);
    return free;
}
private void unparkSuccessor(Node node) {
    // 注意，这里的node是头节点
    
    // 如果头节点的等待状态小于0，就把它设置为0
    int ws = node.waitStatus;
    if (ws < 0)
        compareAndSetWaitStatus(node, ws, 0);

    // 头节点的下一个节点
    Node s = node.next;
    // 如果下一个节点为空，或者其等待状态大于0（实际为已取消）
    if (s == null || s.waitStatus > 0) {
        s = null;
        // 从尾节点向前遍历取到队列最前面的那个状态不是已取消状态的节点
        for (Node t = tail; t != null && t != node; t = t.prev)
            if (t.waitStatus <= 0)
                s = t;
    }
    // 如果下一个节点不为空，则唤醒它
    if (s != null)
        LockSupport.unpark(s.thread);
}
```

释放锁的过程大致为：

（1）将state的值减1；

（2）如果state减到了0，说明已经完全释放锁了，唤醒下一个等待着的节点；


**未完待续，下一章我们继续学习ReentrantLock中关于条件锁的部分**

## 彩蛋

为什么ReentrantLock默认采用的是非公平模式？

答：因为非公平模式效率比较高。

为什么非公平模式效率比较高？

答：因为非公平模式会在一开始就尝试两次获取锁，如果当时正好state的值为0，它就会成功获取到锁，少了排队导致的阻塞/唤醒过程，并且减少了线程频繁的切换带来的性能损耗。

非公平模式有什么弊端？

答：非公平模式有可能会导致一开始排队的线程一直获取不到锁，导致线程饿死。



## 第二部分：条件锁

（1）条件锁是什么？

（2）条件锁适用于什么场景？

（3）条件锁的await()是在其它线程signal()的时候唤醒的吗？

## 简介

条件锁，是指在获取锁之后发现当前业务场景自己无法处理，而需要等待某个条件的出现才可以继续处理时使用的一种锁。

比如，在阻塞队列中，当队列中没有元素的时候是无法弹出一个元素的，这时候就需要阻塞在条件notEmpty上，等待其它线程往里面放入一个元素后，唤醒这个条件notEmpty，当前线程才可以继续去做“弹出一个元素”的行为。

注意，这里的条件，必须是**在获取锁之后去等待**，对应到ReentrantLock的条件锁，就是获取锁之后才能调用condition.await()方法。

在java中，条件锁的实现都在AQS的ConditionObject类中，ConditionObject实现了Condition接口，下面我们通过一个例子来进入到条件锁的学习中。

## 使用示例

```java
public class ReentrantLockTest {
    public static void main(String[] args) throws InterruptedException {
        // 声明一个重入锁
        ReentrantLock lock = new ReentrantLock();
        // 声明一个条件锁
        Condition condition = lock.newCondition();

        new Thread(()->{
            try {
                lock.lock();  // 1
                try {
                    System.out.println("before await");  // 2
                    // 等待条件
                    condition.await();  // 3
                    System.out.println("after await");  // 10
                } finally {
                    lock.unlock();  // 11
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
        
        // 这里睡1000ms是为了让上面的线程先获取到锁
        Thread.sleep(1000);
        lock.lock();  // 4
        try {
            // 这里睡2000ms代表这个线程执行业务需要的时间
            Thread.sleep(2000);  // 5
            System.out.println("before signal");  // 6
            // 通知条件已成立
            condition.signal();  // 7
            System.out.println("after signal");  // 8
        } finally {
            lock.unlock();  // 9
        }
    }
}
```

上面的代码很简单，一个线程等待条件，另一个线程通知条件已成立，后面的数字代表代码实际运行的顺序，如果你能把这个顺序看懂基本条件锁掌握得差不多了。

## 源码分析

### ConditionObject的主要属性

```java
public class ConditionObject implements Condition, java.io.Serializable {
    /** First node of condition queue. */
    private transient Node firstWaiter;
    /** Last node of condition queue. */
    private transient Node lastWaiter;
}
```

可以看到条件锁中也维护了一个队列，为了和AQS的队列区分，我这里称为条件队列，firstWaiter是队列的头节点，lastWaiter是队列的尾节点，它们是干什么的呢？接着看。

### lock.newCondition()方法

新建一个条件锁。

```java
// ReentrantLock.newCondition()
public Condition newCondition() {
    return sync.newCondition();
}
// ReentrantLock.Sync.newCondition()
final ConditionObject newCondition() {
    return new ConditionObject();
}
// AbstractQueuedSynchronizer.ConditionObject.ConditionObject()
public ConditionObject() { }
```

新建一个条件锁最后就是调用的AQS中的ConditionObject类来实例化条件锁。

### condition.await()方法

condition.await()方法，表明现在要等待条件的出现。

```java
// AbstractQueuedSynchronizer.ConditionObject.await()
public final void await() throws InterruptedException {
    // 如果线程中断了，抛出异常
    if (Thread.interrupted())
        throw new InterruptedException();
    // 添加节点到Condition的队列中，并返回该节点
    Node node = addConditionWaiter();
    // 完全释放当前线程获取的锁
    // 因为锁是可重入的，所以这里要把获取的锁全部释放
    int savedState = fullyRelease(node);
    int interruptMode = 0;
    // 是否在同步队列中
    while (!isOnSyncQueue(node)) {
        // 阻塞当前线程
        LockSupport.park(this);
        
        // 上面部分是调用await()时释放自己占有的锁，并阻塞自己等待条件的出现
        // *************************分界线*************************  //
        // 下面部分是条件已经出现，尝试去获取锁
        
        if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
            break;
    }
    
    // 尝试获取锁，注意第二个参数，这是上一章分析过的方法
    // 如果没获取到会再次阻塞（这个方法这里就不贴出来了，有兴趣的翻翻上一章的内容）
    if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
        interruptMode = REINTERRUPT;
    // 清除取消的节点
    if (node.nextWaiter != null) // clean up if cancelled
        unlinkCancelledWaiters();
    // 线程中断相关
    if (interruptMode != 0)
        reportInterruptAfterWait(interruptMode);
}
// AbstractQueuedSynchronizer.ConditionObject.addConditionWaiter
private Node addConditionWaiter() {
    Node t = lastWaiter;
    // 如果条件队列的尾节点已取消，从头节点开始清除所有已取消的节点
    if (t != null && t.waitStatus != Node.CONDITION) {
        unlinkCancelledWaiters();
        // 重新获取尾节点
        t = lastWaiter;
    }
    // 新建一个节点，它的等待状态是CONDITION
    Node node = new Node(Thread.currentThread(), Node.CONDITION);
    // 如果尾节点为空，则把新节点赋值给头节点（相当于初始化队列）
    // 否则把新节点赋值给尾节点的nextWaiter指针
    if (t == null)
        firstWaiter = node;
    else
        t.nextWaiter = node;
    // 尾节点指向新节点
    lastWaiter = node;
    // 返回新节点
    return node;
}
// AbstractQueuedSynchronizer.fullyRelease
final int fullyRelease(Node node) {
    boolean failed = true;
    try {
        // 获取状态变量的值，重复获取锁，这个值会一直累加
        // 所以这个值也代表着获取锁的次数
        int savedState = getState();
        // 一次性释放所有获得的锁
        if (release(savedState)) {
            failed = false;
            // 返回获取锁的次数
            return savedState;
        } else {
            throw new IllegalMonitorStateException();
        }
    } finally {
        if (failed)
            node.waitStatus = Node.CANCELLED;
    }
}
// AbstractQueuedSynchronizer.isOnSyncQueue
final boolean isOnSyncQueue(Node node) {
    // 如果等待状态是CONDITION，或者前一个指针为空，返回false
    // 说明还没有移到AQS的队列中
    if (node.waitStatus == Node.CONDITION || node.prev == null)
        return false;
    // 如果next指针有值，说明已经移到AQS的队列中了
    if (node.next != null) // If has successor, it must be on queue
        return true;
    // 从AQS的尾节点开始往前寻找看是否可以找到当前节点，找到了也说明已经在AQS的队列中了
    return findNodeFromTail(node);
}
```

这里有几个难理解的点：

（1）Condition的队列和AQS的队列不完全一样；

    AQS的队列头节点是不存在任何值的，是一个虚节点；
    
    Condition的队列头节点是存储着实实在在的元素值的，是真实节点。

（2）各种等待状态（waitStatus）的变化；

    首先，在条件队列中，新建节点的初始等待状态是CONDITION（-2）；
    
    其次，移到AQS的队列中时等待状态会更改为0（AQS队列节点的初始等待状态为0）；
    
    然后，在AQS的队列中如果需要阻塞，会把它上一个节点的等待状态设置为SIGNAL（-1）；
    
    最后，不管在Condition队列还是AQS队列中，已取消的节点的等待状态都会设置为CANCELLED（1）；
    
    另外，后面我们在共享锁的时候还会讲到另外一种等待状态叫PROPAGATE（-3）。

（3）相似的名称；

    AQS中下一个节点是next，上一个节点是prev；
    
    Condition中下一个节点是nextWaiter，没有上一个节点。

如果弄明白了这几个点，看懂上面的代码还是轻松加愉快的，如果没弄明白，彤哥这里指出来了，希望您回头再看看上面的代码。

下面总结一下await()方法的大致流程：

（1）新建一个节点加入到条件队列中去；

（2）完全释放当前线程占有的锁；

（3）阻塞当前线程，并等待条件的出现；

（4）条件已出现（此时节点已经移到AQS的队列中），尝试获取锁；

也就是说await()方法内部其实是`先释放锁->等待条件->再次获取锁`的过程。

### condition.signal()方法

condition.signal()方法通知条件已经出现。

```java
// AbstractQueuedSynchronizer.ConditionObject.signal
public final void signal() {
    // 如果不是当前线程占有着锁，调用这个方法抛出异常
    // 说明signal()也要在获取锁之后执行
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    // 条件队列的头节点
    Node first = firstWaiter;
    // 如果有等待条件的节点，则通知它条件已成立
    if (first != null)
        doSignal(first);
}
// AbstractQueuedSynchronizer.ConditionObject.doSignal
private void doSignal(Node first) {
    do {
        // 移到条件队列的头节点往后一位
        if ( (firstWaiter = first.nextWaiter) == null)
            lastWaiter = null;
        // 相当于把头节点从队列中出队
        first.nextWaiter = null;
        // 转移节点到AQS队列中
    } while (!transferForSignal(first) &&
             (first = firstWaiter) != null);
}
// AbstractQueuedSynchronizer.transferForSignal
final boolean transferForSignal(Node node) {
    // 把节点的状态更改为0，也就是说即将移到AQS队列中
    // 如果失败了，说明节点已经被改成取消状态了
    // 返回false，通过上面的循环可知会寻找下一个可用节点
    if (!compareAndSetWaitStatus(node, Node.CONDITION, 0))
        return false;

    // 调用AQS的入队方法把节点移到AQS的队列中
    // 注意，这里enq()的返回值是node的上一个节点，也就是旧尾节点
    Node p = enq(node);
    // 上一个节点的等待状态
    int ws = p.waitStatus;
    // 如果上一个节点已取消了，或者更新状态为SIGNAL失败（也是说明上一个节点已经取消了）
    // 则直接唤醒当前节点对应的线程
    if (ws > 0 || !compareAndSetWaitStatus(p, ws, Node.SIGNAL))
        LockSupport.unpark(node.thread);
    // 如果更新上一个节点的等待状态为SIGNAL成功了
    // 则返回true，这时上面的循环不成立了，退出循环，也就是只通知了一个节点
    // 此时当前节点还是阻塞状态
    // 也就是说调用signal()的时候并不会真正唤醒一个节点
    // 只是把节点从条件队列移到AQS队列中
    return true;
}
```

signal()方法的大致流程为：

（1）从条件队列的头节点开始寻找一个非取消状态的节点；

（2）把它从条件队列移到AQS队列；

（3）且只移动一个节点；

注意，这里调用signal()方法后并不会真正唤醒一个节点，那么，唤醒一个节点是在啥时候呢？

还记得开头例子吗？倒回去再好好看看，signal()方法后，最终会执行lock.unlock()方法，此时才会真正唤醒一个节点，唤醒的这个节点如果曾经是条件节点的话又会继续执行await()方法“分界线”下面的代码。

结束了，仔细体会下^^

如果非要用一个图来表示的话，我想下面这个图可以大致表示一下（这里是用时序图画的，但是实际并不能算作一个真正的时序图哈，了解就好）：

![ReentrantLock](https://gitee.com/alan-tang-tt/yuan/raw/master/死磕%20java同步系列/resource/condition.png)

## 总结

（1）重入锁是指可重复获取的锁，即一个线程获取锁之后再尝试获取锁时会自动获取锁；

（2）在ReentrantLock中重入锁是通过不断累加state变量的值实现的；

（3）ReentrantLock的释放要跟获取匹配，即获取了几次也要释放几次；

（4）ReentrantLock默认是非公平模式，因为非公平模式效率更高；

（5）条件锁是指为了等待某个条件出现而使用的一种锁；

（6）条件锁比较经典的使用场景就是队列为空时阻塞在条件notEmpty上；

（7）ReentrantLock中的条件锁是通过AQS的ConditionObject内部类实现的；

（8）await()和signal()方法都必须在获取锁之后释放锁之前使用；

（9）await()方法会新建一个节点放到条件队列中，接着完全释放锁，然后阻塞当前线程并等待条件的出现；

（10）signal()方法会寻找条件队列中第一个可用节点移到AQS队列中；

（11）在调用signal()方法的线程调用unlock()方法才真正唤醒阻塞在条件上的节点（此时节点已经在AQS队列中）；

（12）之后该节点会再次尝试获取锁，后面的逻辑与lock()的逻辑基本一致了。

## 彩蛋

为什么java有自带的关键字synchronized了还需要实现一个ReentrantLock呢？

首先，它们都是可重入锁；

其次，它们都默认是非公平模式；

然后，...，呃，我们下一章继续深入探讨 ReentrantLock VS synchronized。

### 示例

当第一个线程执行 Method1 ，首先获取锁之后进行输出，但是因为 lock.unlock(); 被注释了，所以锁并没有释放，这样当第二个线程尝试执行 method2 的时候，在 lock.lock() 时候获取不到锁，所以第二个线程进入睡眠状态，等待着获取这把锁，在等待的过程中，当第一个线程等待 2s 之后进入了下一个循环之中，然后又执行了 `myTest1.myMethod1()`，即执行了 myMethod1 方法，然后即执行其内部的 lock.lock()；因为该对象的锁已经被该线程持有了，因此该线程再次尝试获取该对象的锁的时候，因为是可重入锁所以是可以获取的。

```java
package com.gjxaiou.reentrantLock;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @Author GJXAIOU
 * @Date 2021/3/19 15:38
 */
public class MyTest1 {

    // 先定义锁的对象实例
    public Lock lock = new ReentrantLock();

    public void myMethod1(){
        try {
            // 首先尝试获取锁
            lock.lock();
            System.out.println("myMethod1 invoked");
        } finally {
            // 如果注释该行代码，则只执行 method1，并且输出完之后 JVM 不退出
            lock.unlock();
        }
    }

    public void myMethod2(){
        try {
            // 首先尝试获取锁
            lock.lock();
            System.out.println("myMethod2 invoked");
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) {
         MyTest1 myTest1 = new MyTest1();
        // 分别构建两个线程对象，分别访问上面两个方法

        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                myTest1.myMethod1();
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                myTest1.myMethod2();
                try {
                    Thread.sleep(300);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        t1.start();
        t2.start();

    }
}
```

执行结果为：每 2 秒输出一次 `myMethod1 invoked`，但是输出完之后 JVM 并没有退出。

```java
myMethod1 invoked
myMethod1 invoked
myMethod1 invoked
myMethod1 invoked
myMethod1 invoked
myMethod1 invoked
myMethod1 invoked
myMethod1 invoked
myMethod1 invoked
myMethod1 invoked
```

如果换成 tryLock，可以正常执行完，这也是比 syhchronized 的优势。

```java
package com.gjxaiou.reentrantLock;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @Author GJXAIOU
 * @Date 2021/3/19 16:49
 */
public class MyTest2 {

    public Lock lock = new ReentrantLock();

    public void myMethod1() {
        try {
            lock.lock();
            System.out.println("myMethod1 invoked");
        } finally {
            // 取消对锁的释放
        }
    }

    public void myMethod2() {
        boolean result = false;
        try {
            result = lock.tryLock(800, TimeUnit.MILLISECONDS);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        if (result) {
            System.out.println("get the lock");
        } else {
            System.out.println("can't get the lock");
        }
    }


    public static void main(String[] args) {
        MyTest2 myTest2 = new MyTest2();

        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                myTest2.myMethod1();

                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                myTest2.myMethod2();
                try {
                    Thread.sleep(800);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        });

        t1.start();
        t2.start();

    }
}
```

输出结果为：正常结束

```java
myMethod1 invoked
can't get the lock
myMethod1 invoked
can't get the lock
myMethod1 invoked
can't get the lock
can't get the lock
myMethod1 invoked
can't get the lock
myMethod1 invoked
can't get the lock
myMethod1 invoked
can't get the lock
myMethod1 invoked
can't get the lock
can't get the lock
myMethod1 invoked
can't get the lock
myMethod1 invoked
myMethod1 invoked

Process finished with exit code 0
```

测试三：将线程中睡眠动作放入方法中，因为现在是每次方法执行完才睡眠，不会对 lock 产生影响。

```java
package com.gjxaiou.reentrantLock;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * @Author GJXAIOU
 * @Date 2021/3/19 16:59
 */
public class MyTest3 {

    public Lock lock = new ReentrantLock();

    public void myMethod1() {
        try {
            lock.lock();
            System.out.println("myMethod1 invoked");
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();

        } finally {
            lock.unlock();
        }
    }

    public void myMethod2() {
        boolean result = false;
        try {
            result = lock.tryLock(800, TimeUnit.MILLISECONDS);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        if (result) {
            System.out.println("get the lock");
        } else {
            System.out.println("can't get the lock");
        }
    }

    public static void main(String[] args) {
        MyTest3 myTest3 = new MyTest3();
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                myTest3.myMethod1();
            }
        });

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                myTest3.myMethod2();
            }
        });

        t1.start();
        t2.start();

    }

}

```

当第一个线程执行 myMethod1 时候，通过 lock.lock() 获取锁之后进入睡眠状态，则第二个线程执行 myMethod2 的 lock.tryLock() 时可能获取到也可能获取不到。

如果第一个线程刚好执行 unlock() 之后，还没有获取到新的锁之后，第二个线程就可以获取到锁。如果第一个线程还处于睡眠状态，则该线程还没有执行完该方法体中的代码，则这个锁还没有释放，则第二个线程获取不到锁。

输出结果

```java
myMethod1 invoked
can't get the lock
can't get the lock
myMethod1 invoked
can't get the lock
can't get the lock
get the lock
get the lock
get the lock
get the lock
get the lock
get the lock
```

#### Lock 和 synchronized 的区别

- 锁的获取方式∶前者是通过程序代码的方式由开发者手工获取（通过 lock()/tryLock() 等方法），后者是通过JVM来获取（无需开发者干预) 

- 具体实现方式: 前者是通过Java代码的方式来实现，后者是通过JVM底层来实现(无需开发者关注) 

- 锁的释放方式: 前者务必通过unlock()方法在finally块中手工释放，后者是通过JVM来释放（无需开发者关注) 

- 锁的具体类型: 前者提供了多种，如公平锁、非公平锁，后者与前者均提供了可重入锁 

常用的实现类：ReentrantLock（可重入锁，可以重新获得的锁）

## 问题

（1）ReentrantLock有哪些优点？

（2）ReentrantLock有哪些缺点？

（3）ReentrantLock是否可以完全替代synchronized？

## 简介

synchronized是Java原生提供的用于在多线程环境中保证同步的关键字，底层是通过修改对象头中的MarkWord来实现的。

ReentrantLock是Java语言层面提供的用于在多线程环境中保证同步的类，底层是通过原子更新状态变量state来实现的。

既然有了synchronized的关键字来保证同步了，为什么还要实现一个ReentrantLock类呢？它们之间有什么异同呢？

## ReentrantLock VS synchronized

直接上表格：（手机横屏查看更方便）

| 功能                     | ReentrantLock                                                | synchronized                                          |
| ------------------------ | ------------------------------------------------------------ | ----------------------------------------------------- |
| 可重入                   | 支持                                                         | 支持                                                  |
| 非公平                   | 支持（默认）                                                 | 支持                                                  |
| 加锁/解锁方式            | 需要手动加锁、解锁，一般使用try..finally..保证锁能够释放     | 手动加锁，无需刻意解锁                                |
| 按key锁                  | 不支持，比如按用户id加锁                                     | 支持，synchronized加锁时需要传入一个对象              |
| 公平锁                   | 支持，new ReentrantLock(true)                                | 不支持                                                |
| 中断                     | 支持，lockInterruptibly()                                    | 不支持                                                |
| 尝试加锁                 | 支持，tryLock()                                              | 不支持                                                |
| 超时锁                   | 支持，tryLock(timeout, unit)                                 | 不支持                                                |
| 获取当前线程获取锁的次数 | 支持，getHoldCount()                                         | 不支持                                                |
| 获取等待的线程           | 支持，getWaitingThreads()                                    | 不支持                                                |
| 检测是否被当前线程占有   | 支持，isHeldByCurrentThread()                                | 不支持                                                |
| 检测是否被任意线程占有   | 支持，isLocked()                                             | 不支持                                                |
| 条件锁                   | 可支持多个条件，condition.await()，condition.signal()，condition.signalAll() | 只支持一个，obj.wait()，obj.notify()，obj.notifyAll() |

## 对比测试

在测试之前，我们先预想一下结果，随着线程数的不断增加，ReentrantLock（fair）、ReentrantLock（unfair）、synchronized三者的效率怎样呢？

我猜测应该是ReentrantLock（unfair）> synchronized > ReentrantLock（fair）。

到底是不是这样呢？

直接上测试代码：（为了全面对比，彤哥这里把AtomicInteger和LongAdder也拿来一起对比了）

```java
public class ReentrantLockVsSynchronizedTest {
    public static AtomicInteger a = new AtomicInteger(0);
    public static LongAdder b = new LongAdder();
    public static int c = 0;
    public static int d = 0;
    public static int e = 0;

    public static final ReentrantLock fairLock = new ReentrantLock(true);
    public static final ReentrantLock unfairLock = new ReentrantLock();


    public static void main(String[] args) throws InterruptedException {
        System.out.println("-------------------------------------");
        testAll(1, 100000);
        System.out.println("-------------------------------------");
        testAll(2, 100000);
        System.out.println("-------------------------------------");
        testAll(4, 100000);
        System.out.println("-------------------------------------");
        testAll(6, 100000);
        System.out.println("-------------------------------------");
        testAll(8, 100000);
        System.out.println("-------------------------------------");
        testAll(10, 100000);
        System.out.println("-------------------------------------");
        testAll(50, 100000);
        System.out.println("-------------------------------------");
        testAll(100, 100000);
        System.out.println("-------------------------------------");
        testAll(200, 100000);
        System.out.println("-------------------------------------");
        testAll(500, 100000);
        System.out.println("-------------------------------------");
//        testAll(1000, 1000000);
        System.out.println("-------------------------------------");
        testAll(500, 10000);
        System.out.println("-------------------------------------");
        testAll(500, 1000);
        System.out.println("-------------------------------------");
        testAll(500, 100);
        System.out.println("-------------------------------------");
        testAll(500, 10);
        System.out.println("-------------------------------------");
        testAll(500, 1);
        System.out.println("-------------------------------------");
    }

    public static void testAll(int threadCount, int loopCount) throws InterruptedException {
        testAtomicInteger(threadCount, loopCount);
        testLongAdder(threadCount, loopCount);
        testSynchronized(threadCount, loopCount);
        testReentrantLockUnfair(threadCount, loopCount);
//        testReentrantLockFair(threadCount, loopCount);
    }

    public static void testAtomicInteger(int threadCount, int loopCount) throws InterruptedException {
        long start = System.currentTimeMillis();

        CountDownLatch countDownLatch = new CountDownLatch(threadCount);
        for (int i = 0; i < threadCount; i++) {
            new Thread(() -> {
                for (int j = 0; j < loopCount; j++) {
                    a.incrementAndGet();
                }
                countDownLatch.countDown();
            }).start();
        }

        countDownLatch.await();

        System.out.println("testAtomicInteger: result=" + a.get() + ", threadCount=" + threadCount + ", loopCount=" + loopCount + ", elapse=" + (System.currentTimeMillis() - start));
    }

    public static void testLongAdder(int threadCount, int loopCount) throws InterruptedException {
        long start = System.currentTimeMillis();

        CountDownLatch countDownLatch = new CountDownLatch(threadCount);
        for (int i = 0; i < threadCount; i++) {
            new Thread(() -> {
                for (int j = 0; j < loopCount; j++) {
                    b.increment();
                }
                countDownLatch.countDown();
            }).start();
        }

        countDownLatch.await();

        System.out.println("testLongAdder: result=" + b.sum() + ", threadCount=" + threadCount + ", loopCount=" + loopCount + ", elapse=" + (System.currentTimeMillis() - start));
    }

    public static void testReentrantLockFair(int threadCount, int loopCount) throws InterruptedException {
        long start = System.currentTimeMillis();

        CountDownLatch countDownLatch = new CountDownLatch(threadCount);
        for (int i = 0; i < threadCount; i++) {
            new Thread(() -> {
                for (int j = 0; j < loopCount; j++) {
                    fairLock.lock();
                    // 消除try的性能影响
//                    try {
                        c++;
//                    } finally {
                        fairLock.unlock();
//                    }
                }
                countDownLatch.countDown();
            }).start();
        }

        countDownLatch.await();

        System.out.println("testReentrantLockFair: result=" + c + ", threadCount=" + threadCount + ", loopCount=" + loopCount + ", elapse=" + (System.currentTimeMillis() - start));
    }

    public static void testReentrantLockUnfair(int threadCount, int loopCount) throws InterruptedException {
        long start = System.currentTimeMillis();

        CountDownLatch countDownLatch = new CountDownLatch(threadCount);
        for (int i = 0; i < threadCount; i++) {
            new Thread(() -> {
                for (int j = 0; j < loopCount; j++) {
                    unfairLock.lock();
                    // 消除try的性能影响
//                    try {
                        d++;
//                    } finally {
                        unfairLock.unlock();
//                    }
                }
                countDownLatch.countDown();
            }).start();
        }

        countDownLatch.await();

        System.out.println("testReentrantLockUnfair: result=" + d + ", threadCount=" + threadCount + ", loopCount=" + loopCount + ", elapse=" + (System.currentTimeMillis() - start));
    }

    public static void testSynchronized(int threadCount, int loopCount) throws InterruptedException {
        long start = System.currentTimeMillis();

        CountDownLatch countDownLatch = new CountDownLatch(threadCount);
        for (int i = 0; i < threadCount; i++) {
            new Thread(() -> {
                for (int j = 0; j < loopCount; j++) {
                    synchronized (ReentrantLockVsSynchronizedTest.class) {
                        e++;
                    }
                }
                countDownLatch.countDown();
            }).start();
        }

        countDownLatch.await();

        System.out.println("testSynchronized: result=" + e + ", threadCount=" + threadCount + ", loopCount=" + loopCount + ", elapse=" + (System.currentTimeMillis() - start));
    }

}
```

运行这段代码，你会发现结果大大出乎意料，真的是不测不知道，一测吓一跳，运行后发现以下规律：

`随着线程数的不断增加，synchronized的效率竟然比ReentrantLock非公平模式要高！`

彤哥的电脑上大概是高3倍左右，我的运行环境是4核8G，java版本是8，请大家一定要在自己电脑上运行一下，并且最好能给我反馈一下。

彤哥又使用Java7及以下的版本运行了，发现在Java7及以下版本中synchronized的效率确实比ReentrantLock的效率低一些。

## 总结

（1）synchronized是Java原生关键字锁；

（2）ReentrantLock是Java语言层面提供的锁；

（3）ReentrantLock的功能非常丰富，解决了很多synchronized的局限性；

（4）至于在非公平模式下，ReentrantLock与synchronized的效率孰高孰低，彤哥给出的结论是随着Java版本的不断升级，synchronized的效率只会越来越高；

## 彩蛋

既然ReentrantLock的功能更丰富，而且效率也不低，我们是不是可以放弃使用synchronized了呢？

答：我认为不是。因为synchronized是Java原生支持的，随着Java版本的不断升级，Java团队也是在不断优化synchronized，所以我认为在功能相同的前提下，最好还是使用原生的synchronized关键字来加锁，这样我们就能获得Java版本升级带来的免费的性能提升的空间。

另外，在Java8的ConcurrentHashMap中已经把ReentrantLock换成了synchronized来分段加锁了，这也是Java版本不断升级带来的免费的synchronized的性能提升。