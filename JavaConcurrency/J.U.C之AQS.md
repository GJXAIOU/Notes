<!-- GFM-TOC -->
* [J.U.C之AQS](#J.U.C之AQS)
    * [介绍](#介绍)
    * [设计原理](#设计原理)
    * [具体实现思路](#具体实现思路)
    * [同步组件](#同步组件)
        * [CountDownLatch](#CountDownLatch)
        * [Semaphore](#Semaphore)
        * [CyclicBarrier](#CyclicBarrier)
    * [JUC下的锁](#JUC下的锁)
        * [ReentrantLock](#ReentrantLock)
        * [ReentrantReadWriteLock](#ReentrantReadWriteLock)    
        * [StampedLock](#StampedLock)
        * [Condition](#Condition)
    * [总结](#总结)
<!-- GFM-TOC -->

# J.U.C之AQS
## 介绍
Java从1.5之后引入了JUC包，这个包大大提升了并发执行的性能，而AQS就JUC的核心。
AQS使用了**一个int成员变量表示同步状态，通过内置的FIFO队列来完成资源获
取线程的排队工作**，这个队列可以用来构建锁或者其他基础框架。


AQS：AbstractQueuedSynchronizer，即队列同步器。
它是**构建锁或者其他同步组件**的基础框架（如ReentrantLock、ReentrantReadWriteLock、Semaphore等），JUC并发包的作者（Doug Lea）期望它能够成为实现大部分同步需求的基础。

同步器的主要使用方式是**继承**，子类通过继承同步器并实现它的抽象方法来**管理同步状
态**，在抽象方法的实现过程中免不了要对同步状态进行更改，这时就需要使用同步器提供的3
个方法来进行操作，因为它们**能够保证状态的改变是安全的**。
子类推荐被定义为自定义同步组件的静态内部类，同步器自身没有实现任何同步接口，它**仅仅是定义了若干同步状态获取和释放的方法来
供自定义同步组件使用**，同步器既可以支持独占式地获取同步状态，也可以支持共享式地获
取同步状态，这样就可以方便实现不同类型的同步组件（ReentrantLock、
ReentrantReadWriteLock和CountDownLatch等）。
>同步器提供的3个操作状态的方法：getState()、setState(int newState)和compareAndSetState(int expect,int update)。


同步器是实现锁（也可以是任意同步组件）的关键，在锁的实现中聚合同步器，利用同步
器实现锁的语义。可以这样理解二者之间的关系：**锁是面向使用者的，它定义了使用者与锁交
互的接口（比如可以允许两个线程并行访问），隐藏了实现细节；同步器面向的是锁的实现者，
它简化了锁的实现方式，屏蔽了同步状态管理、线程的排队、等待与唤醒等底层操作**。锁和同
步器很好地隔离了使用者和实现者所需关注的领域。

其底层的数据结构可以看做一个队列，如下图所示：

![](pics/aqs/AQS_01.png)

- Sync queue：双向链表，同步队列，head节点主要负责后面的调度。
- Condition queue：单向链表，不是必须的的，也可以有多个。
## 设计原理
* 使用Node实现FIFO队列，可以用于构建锁或者其他同步装置的基础框架

* 利用了一个int类型标示状态，AQS中有一个state的成员变量，基于AQS有一个同步组件（ReentrantLock），在这个组件里面，state表示获取锁的线程数（0没有线程获取锁，1有线程获取锁，大于1表示重入锁的数量）。

* 使用方法是继承，基于模板方法。

* 子类通过继承并通过实现它的方法管理其状态{acquire和release}的方法操作状态

* 可以实现排它锁和共享锁的模式（独占、共享），它是所有子类中要么使用或实现了独占功能的API，要么使用了共享锁的功能，**而不会同时使用两套API。**
## 具体实现思路
1. 首先 AQS内部维护了一个CLH队列，来管理锁

线程尝试获取锁，如果获取失败，则将等待信息等包装成一个Node结点，加入到同步队列Sync queue里

2. 接着会不断重新尝试获取锁（当前结点为head的直接后继才会尝试），如果获取失败，则会阻塞自己，直到被唤醒

3. 当持有锁的线程释放锁的时候，会唤醒队列中的后继线程。

基于以上思路，JDK中实现了我们常用的AQS的子类。

[补充内容](https://github.com/CL0610/Java-concurrency/blob/master/08.%E5%88%9D%E8%AF%86Lock%E4%B8%8EAbstractQueuedSynchronizer(AQS)/%E5%88%9D%E8%AF%86Lock%E4%B8%8EAbstractQueuedSynchronizer(AQS).md)

## 同步组件
### CountDownLatch
用来控制一个线程等待多个线程。

维护了一个计数器 cnt，每次调用 countDown() 方法会让计数器的值减 1，减到 0 的时候，
那些因为调用 await() 方法而在等待的线程就会被唤醒。

<div align="center"><img src="pics//aqs//AQS_02.png" width="600"></div>

#### 使用场景
* 场景1：
 
程序执行需要等待某个条件完成后，才能进行后面的操作。比如父任务等待所有子任务都完成的时候，
再继续往下进行。

```java
public class CountDownLatchEaample {
    //线程数量
    private static int threadCount=10;

    public static void main(String[] args) throws InterruptedException {
        ExecutorService executorService= Executors.newCachedThreadPool();

        final CountDownLatch countDownLatch=new CountDownLatch(threadCount);


        for (int i = 0; i < threadCount; i++) {
            final int threadNum=i;
            executorService.execute(()->{
                try {
                    test(threadNum);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    countDownLatch.countDown();
                }

            });
        }
        countDownLatch.await();
        //上面的所有线程都执行完了,再执行主线程
        System.out.println("Finished!");
        executorService.shutdown();
    }

    private static void test(int threadNum) throws InterruptedException {
        Thread.sleep(100);
        System.out.println("run: "+threadNum);
        Thread.sleep(100);
    }
}
/**
 输出结果： 
 run: 0
 run: 2
 run: 4
 run: 3
 run: 1
 run: 6
 run: 5
 run: 9
 run: 7
 run: 8
 Finished!
 */
```

* 场景2：

指定执行时间的情况，超过这个任务就不继续等待了，完成多少算多少。
```java
public class CountDownLatchEaample2 {
    //线程数量
    private static int threadCount=10;

    public static void main(String[] args) throws InterruptedException {
        ExecutorService executorService= Executors.newCachedThreadPool();

        final CountDownLatch countDownLatch=new CountDownLatch(threadCount);


        for (int i = 0; i < threadCount; i++) {
            final int threadNum=i;
            executorService.execute(()->{
                try {
                    test(threadNum);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    countDownLatch.countDown();
                }

            });
        }
        countDownLatch.await(10, TimeUnit.MILLISECONDS);
        //上面线程如果在10 内未完成，则有可能会执行主线程
        System.out.println("Finished!");
        //不会立马关闭线程池，而是等待当前线程全部执行完再关闭线程池。
        executorService.shutdown();
    }

    private static void test(int threadNum) throws InterruptedException {
        Thread.sleep(10);
        System.out.println("run: "+threadNum);
    }
}
/**
 输出结果：
 run: 0
 run: 1
 run: 2
 run: 3
 run: 4
 Finished!
 run: 7
 run: 8
 run: 6
 run: 9
 run: 5
 */
```
### Semaphore

Semaphore:信号量，用来控制并发线程的个数，与操作系统中的信号量的概念类似。

其中有`acquire()`方法，用来获取资源，`release()`方法用来释放资源。Semaphore维护了当前访问的个数，通过提供**同步机制**来控制同时访问的个数。

<div align="center"><img src="pics//aqs//AQS_03.png" width="600"></div>


#### 使用场景
* 场景1： 

仅能提供有限访问的资源：比如数据库的连接数最大只有20，而上层的并发数远远大于20，这时候如果不作限制，
可能会由于无法获取连接而导致并发异常，这时候可以使用Semaphore来进行控制，当信号量设置为1的时候，就和单线程很相似了。

```java
//每次获取一个许可
public class SemaphoreExample {
    private static int clientCount = 3;
    private static int totalRequestCount = 10;

    public static void main(String[] args) {
        ExecutorService executorService= Executors.newCachedThreadPool();

        final Semaphore semaphore=new Semaphore(clientCount);

        for(int i=0;i<totalRequestCount;i++){
            final int threadNum=i;
            executorService.execute(new Runnable() {
                @Override
                public void run() {
                    try{
                        //获取一个许可
                       semaphore.acquire();
                       test(threadNum);
                    }catch (InterruptedException e) {
                        e.printStackTrace();
                    }finally {
                        semaphore.release();
                        //释放一个许可
                    }
                }
            });
        }
        executorService.shutdown();
    }

    private static void test(int threadNum) throws InterruptedException {
        System.out.println("run:"+threadNum);
        Thread.sleep(1000);
    }
}
/**
 输出结果：
 每隔一秒输出3个
 run:0
 run:2
 run:1

 run:4
 run:3
 run:5

 run:6
 run:7
 run:8

 run:9
 * */
```
* 场景2：

每次获取多个许可
```java
//每次获取多个许可
public class SemaphoreExample2 {
    private static int clientCount = 3;
    private static int totalRequestCount = 10;

    public static void main(String[] args) {
        ExecutorService executorService= Executors.newCachedThreadPool();

        final Semaphore semaphore=new Semaphore(clientCount);

        for(int i=0;i<totalRequestCount;i++){
            final int threadNum=i;
            executorService.execute(new Runnable() {
                @Override
                public void run() {
                    try{
                        //获取多个许可
                       semaphore.acquire(3);
                       //并发数是3,一次性获取3个许可，同1s内无其他许可释放，相当于单线程了
                       test(threadNum);
                    }catch (InterruptedException e) {
                        e.printStackTrace();
                    }finally {
                        semaphore.release(3);
                        //释放多个许可
                    }
                }
            });
        }
        executorService.shutdown();
    }

    private static void test(int threadNum) throws InterruptedException {
        System.out.println("run:"+threadNum);
        Thread.sleep(1000);
    }
}
/**
 输出结果：
 每隔一秒输出1个
 run:0

 run:2

 run:1

 run:4

 run:3

 run:5
 
 run:6

 run:7

 run:8

 run:9
 * */
```
* 场景3：

尝试获取许可
```java
//尝试获取一个许可
public class SemaphoreExample3 {
    private static int clientCount = 3;
    private static int totalRequestCount = 10;

    public static void main(String[] args) {
        ExecutorService executorService= Executors.newCachedThreadPool();

        final Semaphore semaphore=new Semaphore(clientCount);

        for(int i=0;i<totalRequestCount;i++){
            final int threadNum=i;
            executorService.execute(new Runnable() {
                @Override
                public void run() {
                    try{
                        if(semaphore.tryAcquire()){
                            ////尝试获取一个许可
                            test(threadNum);
                            semaphore.release();
                        }
                    }catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            });
        }
        executorService.shutdown();
    }

    private static void test(int threadNum) throws InterruptedException {
        System.out.println("run:"+threadNum);
        Thread.sleep(1000);
    }
}
/**
 输出结果：
 run:0
 run:1
 run:2
 * */
```

### CyclicBarrier

用来控制多个线程互相等待，只有当多个线程都到达时，这些线程才会继续执行。

和 CountdownLatch 相似，都是通过维护计数器来实现的。
线程执行 await() 方法之后计数器会减 1，并进行等待，直到计数器为 0，所有调用 await() 方法而在等待的线程才能继续执行。

CyclicBarrier 有两个构造函数，其中 parties 指示计数器的初始值，barrierAction 在所有线程都到达屏障的时候会执行一次。

```java
public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    this.parties = parties;
    this.count = parties;
    this.barrierCommand = barrierAction;
}

public CyclicBarrier(int parties) {
    this(parties, null);
}
```
<div align="center"><img src="pics//aqs//AQS_04.png" width="600"></div>


* CyclicBarrier与CountDownLatch区别：

1、CyclicBarrier可以重复使用（使用reset方法），所以它才被叫做循环屏障；CountDownLatch只能用一次

2、CountDownLatch主要用于实现一个或n个线程需要等待其他线程完成某项操作之后，才能继续往下执行，描述的是一个或n个线程等待其他线程的关系；
CyclicBarrier是多个线程相互等待，知道满足条件以后再一起往下执行，描述的是多个线程相互等待的场景。

#### 使用场景

多线程计算数据，最后合并计算结果的应用场景。

比如用Excel保存了用户的银行流水，
每一页保存了一个用户近一年的每一笔银行流水，现在需要统计用户的日均银行流水，
这时候我们就可以用多线程处理每一页里的银行流水，都执行完以后，得到每一个页的日均银行流水，
之后通过CyclicBarrier的action，利用这些线程的计算结果，计算出整个excel的日均流水。

```java
public class CyclicBarrierExample {
    private static int threadCount = 10;

    public static void main(String[] args) throws InterruptedException {
        CyclicBarrier cyclicBarrier=new CyclicBarrier(5);

        ExecutorService executorService= Executors.newCachedThreadPool();


        for (int i = 0; i < threadCount; i++) {
            final int threadNum=i;
            executorService.execute(()->{
                try {
                    System.out.println("before..."+threadNum);
                    cyclicBarrier.await();
                    System.out.println("after..."+threadNum);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            });
        }
        executorService.shutdown();
    }
}
/**
 * 输出结果：
 before...1
 before...0
 before...2
 before...3
 before...4
 before...5
 after...4
 after...1
 after...0
 after...2
 after...3

 before...6
 before...7
 before...8
 before...9
 after...9
 after...5
 after...6
 after...7
 after...8
 */
```
## JUC下的锁
Java一共分为两类锁:

* 一类是由synchornized修饰的锁

* 一类是JUC里提供的锁，核心就是ReentrantLock

AQS锁分为独占锁和共享锁两种：

* 独占锁：锁在一个时间点只能被一个线程占有。
根据锁的获取机制，又分为“公平锁”和“非公平锁”。
等待队列中按照FIFO的原则获取锁，等待时间越长的线程越先获取到锁，这就是公平的获取锁，即公平锁。
而非公平锁，线程获取的锁的时候，无视等待队列直接获取锁。ReentrantLock和ReentrantReadWriteLock.Writelock是独占锁。

* 共享锁：同一个时候能够被多个线程获取的锁，能被共享的锁。
JUC包中ReentrantReadWriteLock.ReadLock，CyclicBarrier，CountDownLatch和Semaphore都是共享锁。

### ReentrantLock
ReentrantLock,它实现是一种自旋锁，通过循环调用CAS操作来实现加锁，性能较好的原因是在于**避免进入进程的内核态
的阻塞状态**。
想尽办法避免进入内核态的阻塞状态是我们设计锁的关键。

* synchronized与ReentrantLock的区别
| \ | synchronized | ReentrantLock |
| :---: | :---: | :---: |
| 可重入性 | 可重入  | 可重入 |
| 锁的实现 | JVM实现，很难操作源码，得到实现 | JDK实现 |
| 性能 | 在引入轻量级锁（偏向锁，自旋锁）后性能大大提升，建议都可以选择的时候选择synchornized | -  |
| 功能区别 | 方便简洁，由编译器负责加锁和释放锁  | 	手工操作  |
| 粒度 | 细粒度，可灵活控制  |  |
| 可否指定公平锁 | 只能是非公平锁 | 可以 |
| 可否放弃锁 | 不可以 | 可以 |

* ReentrantLock独有的功能：

1、 可以指定为公平锁（先等待的线程先获得锁）或非公平锁；

```java
/**
      默认实现了非公平锁
     * Creates an instance of {@code ReentrantLock}.
     * This is equivalent to using {@code ReentrantLock(false)}.
     */
    public ReentrantLock() {
        sync = new NonfairSync();
    }
```

2、提供了一个Condition（条件）类，可以分组唤醒需要唤醒的线程；

3、提供了能够中断等待锁的线程机制，`lock.lockInterruptibly()`

使用synchronized计数：
```java
public class CountExample {

    //请求总数
    public static int clientTotal=5000;

    //同时并发执行的线程数
    public static int threadTotal=200;

    public static volatile int count=0;

    public static void main(String[] args) throws InterruptedException {
        //创建线程池
        ExecutorService executorService= Executors.newCachedThreadPool();
        //定义信号量，闭锁
        final Semaphore semaphore=new Semaphore(threadTotal);

        final CountDownLatch countDownLatch=new CountDownLatch(clientTotal);
        //模拟并发
        for (int i = 0; i < clientTotal; i++) {
            executorService.execute(()->{
                try {
                    semaphore.acquire();
                    add();
                    semaphore.release();
                }catch (Exception e){
                    e.printStackTrace();
                }
                countDownLatch.countDown();

            });
        }
        //确保线程全部执行结束，阻塞进程，并保证
        countDownLatch.await();
        executorService.shutdown();
        System.out.println("count:{"+count+"}");
    }

    private static synchronized void add(){
        count++;
    }
}
//输出结果：count:{5000}
```

使用ReentrantLock计数：
```java
public class CountExample2 {

    //请求总数
    public static int clientTotal=5000;

    //同时并发执行的线程数
    public static int threadTotal=200;

    public static volatile int count=0;

    private final static ReentrantLock lock=new ReentrantLock();

    public static void main(String[] args) throws InterruptedException {
        //创建线程池
        ExecutorService executorService= Executors.newCachedThreadPool();
        //定义信号量，闭锁
        final Semaphore semaphore=new Semaphore(threadTotal);

        final CountDownLatch countDownLatch=new CountDownLatch(clientTotal);
        //模拟并发
        for (int i = 0; i < clientTotal; i++) {
            executorService.execute(()->{
                try {
                    semaphore.acquire();
                    add();
                    semaphore.release();
                }catch (Exception e){
                    e.printStackTrace();
                }
                countDownLatch.countDown();

            });
        }
        //确保线程全部执行结束，阻塞进程，并保证
        countDownLatch.await();
        executorService.shutdown();
        System.out.println("count:{"+count+"}");
    }

    private static void add(){
        lock.lock();
        try {
            count++;
        }finally {
            lock.unlock();
        }
    }
}
//count:{5000}
```

### ReentrantReadWriteLock
ReentrantLock是一个排他锁，同一时间只允许一个线程访问，
而ReentrantReadWriteLock允许多个读线程同时访问，但不允许写线程和读线程、写线程和写线程同时访问。相对于排他锁，提高了并发性。

在实际应用中，大部分情况下对共享数据（如缓存）的访问都是读操作远多于写操作，
这时ReentrantReadWriteLock能够提供比排他锁更好的并发性和吞吐量。

读写锁内部维护了两个锁，一个用于读操作，一个用于写操作。
所有 ReadWriteLock实现都必须保证 writeLock操作的内存同步效果也要保持与相关 readLock的联系。
也就是说，**成功获取读锁的线程会看到写入锁之前版本所做的所有更新**。

- 分装
```java
public class LockExample {
    private final Map<String,Data> map=new TreeMap();

    private final static ReentrantReadWriteLock lock=new ReentrantReadWriteLock();

    private final static Lock readLock=lock.readLock();

    private final static Lock writeLock=lock.writeLock();

    public  Data get(String key){
        readLock.lock();
        try {
            return map.get(key);
        }finally {
            readLock.unlock();
        }

    }

    public Set<String> getAllKeys(){
        readLock.lock();
        try {
            return map.keySet();
        }finally {
            readLock.unlock();
        }

    }

    public Data put(String key,Data value){
        writeLock.lock();
        try{
            return map.put(key,value);
        }finally {
            writeLock.unlock();
        }
    }
    
    class Data{
    
    }
}
```

### StampedLock
它控制锁有三种模式：写、读和**乐观读**

状态由版本和模式两个部分组成，锁获取方法是一个数字，作为票据（Stamped）。
它用相应的锁的状态来表示和控制当前的访问，数字0表示没有写锁被授权访问。

StampedLock首先调用tryOptimisticRead方法,此时会获得一个“印戳”。然后读取值并检查票据（Stamped），是否仍然有效(例如其他线程已经获得了一个读锁)。
如果有效,就可以使用这个值。
如果无效,就会获得一个读锁(它会阻塞所有的写锁)

在读锁上分为悲观读和乐观读：

乐观读：如果读的操作很多，写操作很少的情况下，我们可以乐观的认为，读写同时发生的几率很小，因此不悲观的使用完全的读取锁定，
程序可以在查看相关的状态之后，判断有没有写操作的变更，再采取相应的措施，这一小小的改进，可以大大提升执行效率。

```java
public class CountExample3 {

    //请求总数
    public static int clientTotal=5000;

    //同时并发执行的线程数
    public static int threadTotal=200;

    public static volatile int count=0;

    private final static StampedLock lock=new StampedLock();

    public static void main(String[] args) throws InterruptedException {
        //创建线程池
        ExecutorService executorService= Executors.newCachedThreadPool();
        //定义信号量，闭锁
        final Semaphore semaphore=new Semaphore(threadTotal);

        final CountDownLatch countDownLatch=new CountDownLatch(clientTotal);
        //模拟并发
        for (int i = 0; i < clientTotal; i++) {
            executorService.execute(()->{
                try {
                    semaphore.acquire();
                    add();
                    semaphore.release();
                }catch (Exception e){
                    e.printStackTrace();
                }
                countDownLatch.countDown();

            });
        }
        //确保线程全部执行结束，阻塞进程，并保证
        countDownLatch.await();
        executorService.shutdown();
        System.out.println("count:{"+count+"}");
    }

    private static void add(){
        long stamp=lock.writeLock();
        try {
            count++;
        }finally {
            lock.unlock(stamp);
        }
    }
}
//count:{5000}
```
## Condition
```java
public class LockExample2 {
    public static void main(String[] args) {
        ReentrantLock reentrantLock = new ReentrantLock();
        Condition condition = reentrantLock.newCondition();

        new Thread(() -> {
            try {
                reentrantLock.lock();
                System.out.println("wait signal");//1
                condition.await();
                //从AQS队列中移除锁，加入了condition的等待队列中，并释放当前锁，当其他线程调用signal()会重新请求锁。
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("get signal");//4
            reentrantLock.unlock();
        }).start();

        new Thread(() -> {
            reentrantLock.lock();
            System.out.println("get lock"); //2
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            condition.signal(); //当其他线程调用signal()会重新请求锁。
            System.out.println("send signal ~ ");
            reentrantLock.unlock();
        }).start();
    }
}
```
## 总结

* synchronized：JVM实现，不但可以通过一些监控工具监控，而且在出现未知异常的时候JVM也会自动帮我们释放锁，
不会造成死锁现象;

* ReentrantLock、ReentrantRead/WriteLock、StempedLock 他们都是对象层面的锁定，
**要想保证锁一定被释放**，要放到finally里面，才会更安全一些；
StampedLock对性能有很大的改进，特别是在读线程越来越多的情况下，

使用:

1、在只有少量竞争者的时候，synchronized是一个很好的锁的实现

2、竞争者不少，但是增长的趋势是可以预估的，ReentrantLock是一个很好的锁的实现（适合自己的才是最好的，不是越高级越好）






