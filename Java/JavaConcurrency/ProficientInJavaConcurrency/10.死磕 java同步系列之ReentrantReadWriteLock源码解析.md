🖕欢迎关注我的公众号“彤哥读源码”，查看更多源码系列文章, 与彤哥一起畅游源码的海洋。 

（手机横屏看源码更方便）

---

## 问题

（1）读写锁是什么？

（2）读写锁具有哪些特性？

（3）ReentrantReadWriteLock是怎么实现读写锁的？

（4）如何使用ReentrantReadWriteLock实现高效安全的TreeMap？

## 简介

读写锁是一种特殊的锁，它把对共享资源的访问分为读访问和写访问，多个线程可以同时对共享资源进行读访问，但是同一时间只能有一个线程对共享资源进行写访问，使用读写锁可以极大地提高并发量。

## 特性

读写锁具有以下特性：

|是否互斥|读|写|
|:---:|:---:|:---:|
|读|否|是| 
|写|是|是|

可以看到，读写锁除了读读不互斥，读写、写读、写写都是互斥的。

那么，ReentrantReadWriteLock是怎么实现读写锁的呢？

## 类结构

在看源码之前，我们还是先来看一下ReentrantReadWriteLock这个类的主要结构。

![ReentrantReadWriteLock](https://gitee.com/alan-tang-tt/yuan/raw/master/死磕%20java同步系列/resource/ReentrantReadWriteLock.png)

ReentrantReadWriteLock中的类分成三个部分：

（1）ReentrantReadWriteLock本身实现了ReadWriteLock接口，这个接口只提供了两个方法`readLock()`和`writeLock（）`；

（2）同步器，包含一个继承了AQS的Sync内部类，以及其两个子类FairSync和NonfairSync；

（3）ReadLock和WriteLock两个内部类实现了Lock接口，它们具有锁的一些特性。

## 源码分析

### 主要属性

```java
// 读锁
private final ReentrantReadWriteLock.ReadLock readerLock;
// 写锁
private final ReentrantReadWriteLock.WriteLock writerLock;
// 同步器
final Sync sync;
```

维护了读锁、写锁和同步器。

### 主要构造方法

```java
// 默认构造方法
public ReentrantReadWriteLock() {
    this(false);
}
// 是否使用公平锁的构造方法
public ReentrantReadWriteLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
    readerLock = new ReadLock(this);
    writerLock = new WriteLock(this);
}
```

它提供了两个构造方法，默认构造方法使用的是非公平锁模式，在构造方法中初始化了读锁和写锁。

### 获取读锁和写锁的方法

```java
public ReentrantReadWriteLock.WriteLock writeLock() { return writerLock; }
public ReentrantReadWriteLock.ReadLock  readLock()  { return readerLock; }
```

属性中的读锁和写锁是私有属性，通过这两个方法暴露出去。

下面我们主要分析读锁和写锁的加锁、解锁方法，且都是基于非公平模式的。

### ReadLock.lock()

```java
// ReentrantReadWriteLock.ReadLock.lock()
public void lock() {
    sync.acquireShared(1);
}
// AbstractQueuedSynchronizer.acquireShared()
public final void acquireShared(int arg) {
    // 尝试获取共享锁（返回1表示成功，返回-1表示失败）
    if (tryAcquireShared(arg) < 0)
        // 失败了就可能要排队
        doAcquireShared(arg);
}
// ReentrantReadWriteLock.Sync.tryAcquireShared()
protected final int tryAcquireShared(int unused) {
    Thread current = Thread.currentThread();
    // 状态变量的值
    // 在读写锁模式下，高16位存储的是共享锁（读锁）被获取的次数，低16位存储的是互斥锁（写锁）被获取的次数
    int c = getState();
    // 互斥锁的次数
    // 如果其它线程获得了写锁，直接返回-1
    if (exclusiveCount(c) != 0 &&
        getExclusiveOwnerThread() != current)
        return -1;
    // 读锁被获取的次数
    int r = sharedCount(c);
    
    // 下面说明此时还没有写锁，尝试去更新state的值获取读锁
    // 读者是否需要排队（是否是公平模式）
    if (!readerShouldBlock() &&
        r < MAX_COUNT &&
        compareAndSetState(c, c + SHARED_UNIT)) {
        // 获取读锁成功
        if (r == 0) {
            // 如果之前还没有线程获取读锁
            // 记录第一个读者为当前线程
            firstReader = current;
            // 第一个读者重入的次数为1
            firstReaderHoldCount = 1;
        } else if (firstReader == current) {
            // 如果有线程获取了读锁且是当前线程是第一个读者
            // 则把其重入次数加1
            firstReaderHoldCount++;
        } else {
            // 如果有线程获取了读锁且当前线程不是第一个读者
            // 则从缓存中获取重入次数保存器
            HoldCounter rh = cachedHoldCounter;
            // 如果缓存不属性当前线程
            // 再从ThreadLocal中获取
            // readHolds本身是一个ThreadLocal，里面存储的是HoldCounter
            if (rh == null || rh.tid != getThreadId(current))
                // get()的时候会初始化rh
                cachedHoldCounter = rh = readHolds.get();
            else if (rh.count == 0)
                // 如果rh的次数为0，把它放到ThreadLocal中去
                readHolds.set(rh);
            // 重入的次数加1（初始次数为0）
            rh.count++;
        }
        // 获取读锁成功，返回1
        return 1;
    }
    // 通过这个方法再去尝试获取读锁（如果之前其它线程获取了写锁，一样返回-1表示失败）
    return fullTryAcquireShared(current);
}
// AbstractQueuedSynchronizer.doAcquireShared()
private void doAcquireShared(int arg) {
    // 进入AQS的队列中
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            // 当前节点的前一个节点
            final Node p = node.predecessor();
            // 如果前一个节点是头节点（说明是第一个排队的节点）
            if (p == head) {
                // 再次尝试获取读锁
                int r = tryAcquireShared(arg);
                // 如果成功了
                if (r >= 0) {
                    // 头节点后移并传播
                    // 传播即唤醒后面连续的读节点
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;
                    return;
                }
            }
            // 没获取到读锁，阻塞并等待被唤醒
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
// AbstractQueuedSynchronizer.setHeadAndPropagate()
private void setHeadAndPropagate(Node node, int propagate) {
    // h为旧的头节点
    Node h = head;
    // 设置当前节点为新头节点
    setHead(node);
    
    // 如果旧的头节点或新的头节点为空或者其等待状态小于0（表示状态为SIGNAL/PROPAGATE）
    if (propagate > 0 || h == null || h.waitStatus < 0 ||
        (h = head) == null || h.waitStatus < 0) {
        // 需要传播
        // 取下一个节点
        Node s = node.next;
        // 如果下一个节点为空，或者是需要获取读锁的节点
        if (s == null || s.isShared())
            // 唤醒下一个节点
            doReleaseShared();
    }
}
// AbstractQueuedSynchronizer.doReleaseShared()
// 这个方法只会唤醒一个节点
private void doReleaseShared() {
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            // 如果头节点状态为SIGNAL，说明要唤醒下一个节点
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                // 唤醒下一个节点
                unparkSuccessor(h);
            }
            else if (ws == 0 &&
                     // 把头节点的状态改为PROPAGATE成功才会跳到下面的if
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        // 如果唤醒后head没变，则跳出循环
        if (h == head)                   // loop if head changed
            break;
    }
}
```

看完【[死磕 java同步系列之ReentrantLock源码解析（一）——公平锁、非公平锁](https://mp.weixin.qq.com/s/52Ib23kbmqqkWAZtlZF-zA)】的分析再看这章的内容应该会比较简单，中间一样的方法我们这里直接跳过了。

我们来看看大致的逻辑：

（1）先尝试获取读锁；

（2）如果成功了直接结束；

（3）如果失败了，进入doAcquireShared()方法；

（4）doAcquireShared()方法中首先会生成一个新节点并进入AQS队列中；

（5）如果头节点正好是当前节点的上一个节点，再次尝试获取锁；

（6）如果成功了，则设置头节点为新节点，并传播；

（7）传播即唤醒下一个读节点（如果下一个节点是读节点的话）；

（8）如果头节点不是当前节点的上一个节点或者（5）失败，则阻塞当前线程等待被唤醒；

（9）唤醒之后继续走（5）的逻辑；

在整个逻辑中是在哪里连续唤醒读节点的呢？

答案是在doAcquireShared()方法中，在这里一个节点A获取了读锁后，会唤醒下一个读节点B，这时候B也会获取读锁，然后B继续唤醒C，依次往复，也就是说这里的节点是一个唤醒一个这样的形式，而不是一个节点获取了读锁后一次性唤醒后面所有的读节点。

![ReentrantReadWriteLock1](https://gitee.com/alan-tang-tt/yuan/raw/master/死磕%20java同步系列/resource/ReentrantReadWriteLock1.png)

### ReadLock.unlock()

```java
// java.util.concurrent.locks.ReentrantReadWriteLock.ReadLock.unlock
public void unlock() {
    sync.releaseShared(1);
}
// java.util.concurrent.locks.AbstractQueuedSynchronizer.releaseShared
public final boolean releaseShared(int arg) {
    // 如果尝试释放成功了，就唤醒下一个节点
    if (tryReleaseShared(arg)) {
        // 这个方法实际是唤醒下一个节点
        doReleaseShared();
        return true;
    }
    return false;
}
// java.util.concurrent.locks.ReentrantReadWriteLock.Sync.tryReleaseShared
protected final boolean tryReleaseShared(int unused) {
    Thread current = Thread.currentThread();
    if (firstReader == current) {
        // 如果第一个读者（读线程）是当前线程
        // 就把它重入的次数减1
        // 如果减到0了就把第一个读者置为空
        if (firstReaderHoldCount == 1)
            firstReader = null;
        else
            firstReaderHoldCount--;
    } else {
        // 如果第一个读者不是当前线程
        // 一样地，把它重入的次数减1
        HoldCounter rh = cachedHoldCounter;
        if (rh == null || rh.tid != getThreadId(current))
            rh = readHolds.get();
        int count = rh.count;
        if (count <= 1) {
            readHolds.remove();
            if (count <= 0)
                throw unmatchedUnlockException();
        }
        --rh.count;
    }
    for (;;) {
        // 共享锁获取的次数减1
        // 如果减为0了说明完全释放了，才返回true
        int c = getState();
        int nextc = c - SHARED_UNIT;
        if (compareAndSetState(c, nextc))
            return nextc == 0;
    }
}
// java.util.concurrent.locks.AbstractQueuedSynchronizer.doReleaseShared
// 行为跟方法名有点不符，实际是唤醒下一个节点
private void doReleaseShared() {
    for (;;) {
        Node h = head;
        if (h != null && h != tail) {
            int ws = h.waitStatus;
            // 如果头节点状态为SIGNAL，说明要唤醒下一个节点
            if (ws == Node.SIGNAL) {
                if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                    continue;            // loop to recheck cases
                // 唤醒下一个节点
                unparkSuccessor(h);
            }
            else if (ws == 0 &&
                     // 把头节点的状态改为PROPAGATE成功才会跳到下面的if
                     !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                continue;                // loop on failed CAS
        }
        // 如果唤醒后head没变，则跳出循环
        if (h == head)                   // loop if head changed
            break;
    }
}
```

解锁的大致流程如下：

（1）将当前线程重入的次数减1；

（2）将共享锁总共被获取的次数减1；

（3）如果共享锁获取的次数减为0了，说明共享锁完全释放了，那就唤醒下一个节点；

如下图，ABC三个节点各获取了一次共享锁，三者释放的顺序分别为ACB，那么最后B释放共享锁的时候tryReleaseShared()才会返回true，进而才会唤醒下一个节点D。

![ReentrantReadWriteLock2](https://gitee.com/alan-tang-tt/yuan/raw/master/死磕%20java同步系列/resource/ReentrantReadWriteLock2.png)

### WriteLock.lock()

```java
// java.util.concurrent.locks.ReentrantReadWriteLock.WriteLock.lock()
public void lock() {
    sync.acquire(1);
}
// java.util.concurrent.locks.AbstractQueuedSynchronizer.acquire()
public final void acquire(int arg) {
    // 先尝试获取锁
    // 如果失败，则会进入队列中排队，后面的逻辑跟ReentrantLock一模一样了
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
// java.util.concurrent.locks.ReentrantReadWriteLock.Sync.tryAcquire()
protected final boolean tryAcquire(int acquires) {
    Thread current = Thread.currentThread();
    // 状态变量state的值
    int c = getState();
    // 互斥锁被获取的次数
    int w = exclusiveCount(c);
    if (c != 0) {
        // 如果c!=0且w==0，说明共享锁被获取的次数不为0
        // 这句话整个的意思就是
        // 如果共享锁被获取的次数不为0，或者被其它线程获取了互斥锁（写锁）
        // 那么就返回false，获取写锁失败
        if (w == 0 || current != getExclusiveOwnerThread())
            return false;
        // 溢出检测
        if (w + exclusiveCount(acquires) > MAX_COUNT)
            throw new Error("Maximum lock count exceeded");
        // 到这里说明当前线程已经获取过写锁，这里是重入了，直接把state加1即可
        setState(c + acquires);
        // 获取写锁成功
        return true;
    }
    // 如果c等于0，就尝试更新state的值（非公平模式writerShouldBlock()返回false）
    // 如果失败了，说明获取写锁失败，返回false
    // 如果成功了，说明获取写锁成功，把自己设置为占有者，并返回true
    if (writerShouldBlock() ||
        !compareAndSetState(c, c + acquires))
        return false;
    setExclusiveOwnerThread(current);
    return true;
}
// 获取写锁失败了后面的逻辑跟ReentrantLock是一致的，进入队列排队，这里就不列源码了
```

写锁获取的过程大致如下：

（1）尝试获取锁；

（2）如果有读者占有着读锁，尝试获取写锁失败；

（3）如果有其它线程占有着写锁，尝试获取写锁失败；

（4）如果是当前线程占有着写锁，尝试获取写锁成功，state值加1；

（5）如果没有线程占有着锁（state==0），当前线程尝试更新state的值，成功了表示尝试获取锁成功，否则失败；

（6）尝试获取锁失败以后，进入队列排队，等待被唤醒；

（7）后续逻辑跟ReentrantLock是一致；

### WriteLock.unlock()

```java
// java.util.concurrent.locks.ReentrantReadWriteLock.WriteLock.unlock()
public void unlock() {
    sync.release(1);
}
//java.util.concurrent.locks.AbstractQueuedSynchronizer.release()
public final boolean release(int arg) {
    // 如果尝试释放锁成功（完全释放锁）
    // 就尝试唤醒下一个节点
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
// java.util.concurrent.locks.ReentrantReadWriteLock.Sync.tryRelease()
protected final boolean tryRelease(int releases) {
    // 如果写锁不是当前线程占有着，抛出异常
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    // 状态变量的值减1
    int nextc = getState() - releases;
    // 是否完全释放锁
    boolean free = exclusiveCount(nextc) == 0;
    if (free)
        setExclusiveOwnerThread(null);
    // 设置状态变量的值
    setState(nextc);
    // 如果完全释放了写锁，返回true
    return free;
}
```

写锁释放的过程大致为：

（1）先尝试释放锁，即状态变量state的值减1；

（2）如果减为0了，说明完全释放了锁；

（3）完全释放了锁才唤醒下一个等待的节点；

## 总结

（1）ReentrantReadWriteLock采用读写锁的思想，能提高并发的吞吐量；

（2）读锁使用的是共享锁，多个读锁可以一起获取锁，互相不会影响，即读读不互斥；

（3）读写、写读和写写是会互斥的，前者占有着锁，后者需要进入AQS队列中排队；

（4）多个连续的读线程是一个接着一个被唤醒的，而不是一次性唤醒所有读线程；

（5）只有多个读锁都完全释放了才会唤醒下一个写线程；

（6）只有写锁完全释放了才会唤醒下一个等待者，这个等待者有可能是读线程，也可能是写线程；

## 彩蛋

（1）如果同一个线程先获取读锁，再获取写锁会怎样？

![ReentrantReadWriteLock3](https://gitee.com/alan-tang-tt/yuan/raw/master/死磕%20java同步系列/resource/ReentrantReadWriteLock3.png)

分析上图中的代码，在tryAcquire()方法中，如果读锁被获取的次数不为0（c != 0 && w == 0），返回false，返回之后外层方法会让当前线程阻塞。

可以通过下面的方法验证：

```java
readLock.lock();
writeLock.lock();
writeLock.unlock();
readLock.unlock();
```

运行程序后会发现代码停止在`writeLock.lock();`，当然，你也可以打个断点跟踪进去看看。

（2）如果同一个线程先获取写锁，再获取读锁会怎样？

![ReentrantReadWriteLock4](https://gitee.com/alan-tang-tt/yuan/raw/master/死磕%20java同步系列/resource/ReentrantReadWriteLock4.png)

分析上面的代码，在tryAcquireShared()方法中，第一个红框处并不会返回，因为不满足`getExclusiveOwnerThread() != current`；第二个红框处如果原子更新成功就说明获取了读锁，然后就会执行第三个红框处的代码把其重入次数更改为1。

可以通过下面的方法验证：

```java
writeLock.lock();
readLock.lock();
readLock.unlock();
writeLock.unlock();
```

你可以打个断点跟踪一下看看。

（3）死锁了么？

通过上面的两个例子，我们可以感受到同一个线程先读后写和先写后读是完全不一样的，为什么不一样呢？

先读后写，一个线程占有读锁后，其它线程还是可以占有读锁的，这时候如果在其它线程占有读锁之前让自己占有了写锁，其它线程又不能占有读锁了，这段程序会非常难实现，逻辑也很奇怪，所以，设计成只要一个线程占有了读锁，其它线程包括它自己都不能再获取写锁。

先写后读，一个线程占有写锁后，其它线程是不能占有任何锁的，这时候，即使自己占有一个读锁，对程序的逻辑也不会有任何影响，所以，一个线程占有写锁后是可以再占有读锁的，只是这个时候其它线程依然无法获取读锁。

如果你仔细思考上面的逻辑，你会发现一个线程先占有读锁后占有写锁，会有一个很大的问题——锁无法被释放也无法被获取了。这个线程先占有了读锁，然后自己再占有写锁的时候会阻塞，然后它就自己把自己搞死了，进而把其它线程也搞死了，它无法释放锁，其它线程也无法获得锁了。

这是死锁吗？似乎不是，死锁的定义是线程A占有着线程B需要的资源，线程B占有着线程A需要的资源，两个线程相互等待对方释放资源，经典的死锁例子如下：

```java
Object a = new Object();
Object b = new Object();

new Thread(()->{
    synchronized (a) {
        LockSupport.parkNanos(1000000);
        synchronized (b) {

        }
    }
}).start();

new Thread(()->{
    synchronized (b) {
        synchronized (a) {

        }
    }
}).start();
```

简单的死锁用jstack是可以看到的：

```java
"Thread-1":
        at com.coolcoding.code.synchronize.ReentrantReadWriteLockTest.lambda$main$1(ReentrantReadWriteLockTest.java:40)
        - waiting to lock <0x000000076baa9068> (a java.lang.Object)
        - locked <0x000000076baa9078> (a java.lang.Object)
        at com.coolcoding.code.synchronize.ReentrantReadWriteLockTest$$Lambda$2/1831932724.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)
"Thread-0":
        at com.coolcoding.code.synchronize.ReentrantReadWriteLockTest.lambda$main$0(ReentrantReadWriteLockTest.java:32)
        - waiting to lock <0x000000076baa9078> (a java.lang.Object)
        - locked <0x000000076baa9068> (a java.lang.Object)
        at com.coolcoding.code.synchronize.ReentrantReadWriteLockTest$$Lambda$1/1096979270.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)

Found 1 deadlock.
```

（4）如何使用ReentrantReadWriteLock实现一个高效安全的TreeMap？

```java
class SafeTreeMap {
    private final Map<String, Object> m = new TreeMap<String, Object>();
    private final ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
    private final Lock readLock = lock.readLock();
    private final Lock writeLock = lock.writeLock();

    public Object get(String key) {
        readLock.lock();
        try {
            return m.get(key);
        } finally {
            readLock.unlock();
        }
    }

    public Object put(String key, Object value) {
        writeLock.lock();
        try {
            return m.put(key, value);
        } finally {
            writeLock.unlock();
        }
    }
}
```

## 推荐阅读

1. [死磕 java同步系列之ReentrantLock VS synchronized](https://mp.weixin.qq.com/s/o8ZFXDoKhj237SsrqGeJPQ)

2. [死磕 java同步系列之ReentrantLock源码解析（二）——条件锁](https://mp.weixin.qq.com/s/iipAVWynBUZazhSvBwMB5g)

3. [死磕 java同步系列之ReentrantLock源码解析（一）——公平锁、非公平锁](https://mp.weixin.qq.com/s/52Ib23kbmqqkWAZtlZF-zA)

4. [死磕 java同步系列之AQS起篇](https://mp.weixin.qq.com/s/nAqgec8GscULz6DkkYFINg)

5. [死磕 java同步系列之自己动手写一个锁Lock](https://mp.weixin.qq.com/s/1RU5jh7UcXGtKlae8tusVA)

6. [死磕 java魔法类之Unsafe解析](https://mp.weixin.qq.com/s/0s-u-MysppIaIHVrshp9fA)

7. [死磕 java同步系列之JMM（Java Memory Model）](https://mp.weixin.qq.com/s/jownTN--npu3o8B4c3sbeA)

8. [死磕 java同步系列之volatile解析](https://mp.weixin.qq.com/s/TROZ4BhcDImwHvhAl_I_6w)

9. [死磕 java同步系列之synchronized解析](https://mp.weixin.qq.com/s/RT7VreIh9PU03HhE3WSLjg)

欢迎关注我的公众号“彤哥读源码”，查看更多源码系列文章, 与彤哥一起畅游源码的海洋。

![qrcode](https://gitee.com/alan-tang-tt/yuan/raw/master/死磕%20java集合系列/resource/qrcode_ss.jpg)