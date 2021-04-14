# Lock

[TOC]



### Lock 锁机制深入详解

jdk 1.5 之前对于对象同步只能使用 synchronized 关键字，之后引入了 锁，可以实现对象同步。

java.util.concurrency.locks.Lock.java 的 JavaDoc

> Lock 实现提供了比使用 synchronized 方法和语句更广泛的锁定操作。它们允许更灵活的结构，可能具有完全不同的属性，并且可能支持多个关联的 Condition 对象（也在 locks 中，后面讲解）。
>
> **锁是一种工具，用于控制多个线程对共享资源的访问**。通常情况下，锁提供对共享资源的独占访问：一次只有一个线程可以获取锁，**对共享资源的所有访问都要求首先获取锁**。但是，有些锁可能允许并发访问共享资源，例如 ReadWriteLock 的读锁。
>
> > 注：通常情况下， Lock 是一种排他性的锁，即同一时刻只能有一个线程拥有这把锁，然后访问该锁控制的资源，如果有其它线程想访问该共享资源则只能等待持有该锁的线程执行完或者抛出异常从而释放该锁，然后去争抢这把锁。 当时这种方式对于共享资源划分力度不够，例如通常资源读的次数大于写次数，当多个线程对同一个资源都是读取，本质上都是不需要上锁的，所以通过 ReadWriteLock，读和写线程分别获取读锁和写锁。具体后续再分析啦。
>
> 使用 synchronized 方法或语句可以访问与每个对象关联的隐式监视锁，但会强制以块结构的方式获取和释放所有锁：当获取多个锁时，它们必须按相反的顺序释放，所有锁都必须在获得它们的相同词法（作用域）范围内释放。
>
> 虽然 synchronized 方法和语句的作用域机制使使用监视器锁编程更加容易，并有助于避免许多涉及锁的常见编程错误，但有时需要以更灵活的方式使用锁。例如，一些遍历并发访问数据结构的算法需要使用“head - over - head”或“链锁定”：先获取节点 A 的锁，然后获取节点 B，然后释放 A 和获取 C，然后释放 B 和获取D，依此类推。 **Lock 接口的实现允许在不同的作用域中获取和释放一个锁，并允许以任何顺序获取和释放多个锁**，从而允许使用此类技术。
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
> 当锁定和解锁发生在不同的作用域中时，必须注意确保在锁定期间执行的所有代码都受到 try finally 或 try catch 的保护，以确保在必要时释放锁定。
>
> Lock 实现通过提供一个非阻塞的获取锁的尝试 `tryLock()`，一个获取可以中断的锁的尝试lockInterruptibly，以及试图获取可以超时的锁 `tryLock（long，TimeUnit)`。
>
> Lock 类还可以提供与隐式监视锁完全不同的行为和语义，例如保证排序、不可重入使用或死锁检测。如果一个实现提供了这样的专门语义，那么该实现必须记录这些语义。
>
> 注意 Lock 实例只是普通对象，它们本身可以用作  synchronized 语句中的目标。**获取 lock 实例的监视器锁与调用该实例的任何 lock 方法都没有指定的关系**。建议您不要以这种方式使用 Lock 实例，除非在它们自己的实现中。
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

```java

public interface Lock {

    /**
     * Acquires the lock.
     *
     * <p>If the lock is not available then the current thread becomes
     * disabled for thread scheduling purposes and lies dormant until the
     * lock has been acquired.
     *
     * <p><b>Implementation Considerations</b>
     *
     * <p>A {@code Lock} implementation may be able to detect erroneous use
     * of the lock, such as an invocation that would cause deadlock, and
     * may throw an (unchecked) exception in such circumstances.  The
     * circumstances and the exception type must be documented by that
     * {@code Lock} implementation.
     */
    void lock();

    /**
    获取锁，除非当前线程是{@linkplain thread#interrupt interrupted}。
获取锁（如果可用）并立即返回。
如果锁不可用，则当前线程将被禁用以进行线程调度，并处于休眠状态，直到发生以下两种情况之一：
- 这个锁是由当前线程获取的；
- 或者某些其他线程中断当前线程（的休眠状态），并且支持锁获取的中断。

如果当前线程：
- 在进入此方法时设置了中断状态；
- 或者在获取锁时设置了{@linkplain thread#interrupt interrupted}，并且支持中断获取锁
则抛出{@link InterruptedException}，并清除当前线程的中断状态。

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
    只有在调用时锁是空闲的情况下才获取锁。

获取锁（如果可用），并立即返回值{@code true}。如果锁不可用，则此方法将立即返回值{@code false}。
     * Acquires the lock only if it is free at the time of invocation.
     *
     * <p>Acquires the lock if it is available and returns immediately
     * with the value {@code true}.
     * If the lock is not available then this method will return
     * immediately with the value {@code false}.
     *
     * <p>A typical usage idiom for this method would be:
     *  <pre> {@code
     // 定义一个 lock 对象
     * Lock lock = ...;
     * if (lock.tryLock()) {
     *   try {
     *     // manipulate protected state
     *   } finally {
     *     lock.unlock();
     *   }
     * } else {
     *   // perform alternative actions
     * }}</pre>
     *
     // 此用法确保在获取锁时将其解锁，并且在未获取锁时不会尝试解锁。因为if 之后在 finally 中解锁了。
     * This usage ensures that the lock is unlocked if it was acquired, and
     * doesn't try to unlock if the lock was not acquired.
     *
     * @return {@code true} if the lock was acquired and
     *         {@code false} otherwise
     */
    boolean tryLock();

    /**
    如果锁在给定的等待时间内空闲并且当前线程未被终中断，则获取该锁。

如果锁可用（获取到），则此方法立即返回值 true。

如果锁不可用（没有获取到），则当前线程将被禁用以进行线程调度，并处于休眠状态，直到发生以下三种情况之一：
- 锁由当前线程获取；
- 另一个线程中断当前线程，并且支持锁获取中断；
- 指定的等待时间已过

如果获取了锁，则返回值{@code true}。

如果当前线程：在进入此方法时设置了中断状态；或在获取锁时是中断的，并且支持中断获取锁，然后 InterruptedException 被抛出，当前线程的中断状态被清除。



如果指定的等待时间已过，则值返回 false

如果时间小于或等于零，则该方法根本不会等待。

### 实施考虑1

在某些实现中中断锁获取的能力可能是不可能的，并且如果可能的话可能是一个昂贵的操作。

程序员应该意识到情况可能是这样的。在这种情况下，实现应该记录下来。

实现可以支持响应中断，而不是普通的方法返回，或者报告超时。

{@code Lock}实现可能能够检测到锁的错误使用，例如可能导致死锁的调用，并且在这种情况下可能抛出（未检查的）异常。

环境和异常类型必须由{@code Lock}实现记录。
     * Acquires the lock if it is free within the given waiting time and the
     * current thread has not been {@linkplain Thread#interrupt interrupted}.
     *
     * <p>If the lock is available this method returns immediately
     * with the value {@code true}.
     * If the lock is not available then
     * the current thread becomes disabled for thread scheduling
     * purposes and lies dormant until one of three things happens:
     * <ul>
     * <li>The lock is acquired by the current thread; or
     * <li>Some other thread {@linkplain Thread#interrupt interrupts} the
     * current thread, and interruption of lock acquisition is supported; or
     * <li>The specified waiting time elapses
     * </ul>
     *
     * <p>If the lock is acquired then the value {@code true} is returned.
     *
     * <p>If the current thread:
     * <ul>
     * <li>has its interrupted status set on entry to this method; or
     * <li>is {@linkplain Thread#interrupt interrupted} while acquiring
     * the lock, and interruption of lock acquisition is supported,
     * </ul>
     * then {@link InterruptedException} is thrown and the current thread's
     * interrupted status is cleared.
     *
     * <p>If the specified waiting time elapses then the value {@code false}
     * is returned.
     * If the time is
     * less than or equal to zero, the method will not wait at all.
     *
     * <p><b>Implementation Considerations</b>
     *
     * <p>The ability to interrupt a lock acquisition in some implementations
     * may not be possible, and if possible may
     * be an expensive operation.
     * The programmer should be aware that this may be the case. An
     * implementation should document when this is the case.
     *
     * <p>An implementation can favor responding to an interrupt over normal
     * method return, or reporting a timeout.
     *
     * <p>A {@code Lock} implementation may be able to detect
     * erroneous use of the lock, such as an invocation that would cause
     * deadlock, and may throw an (unchecked) exception in such circumstances.
     * The circumstances and the exception type must be documented by that
     * {@code Lock} implementation.
     *
     * @param time the maximum time to wait for the lock
     * @param unit the time unit of the {@code time} argument
     * @return {@code true} if the lock was acquired and {@code false}
     *         if the waiting time elapsed before the lock was acquired
     *
     * @throws InterruptedException if the current thread is interrupted
     *         while acquiring the lock (and interruption of lock
     *         acquisition is supported)
     */
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;

    /**
     * Releases the lock.
     *
     * <p><b>Implementation Considerations</b>
     *
     * <p>A {@code Lock} implementation will usually impose
     * restrictions on which thread can release a lock (typically only the
     * holder of the lock can release it) and may throw
     * an (unchecked) exception if the restriction is violated.
     * Any restrictions and the exception
     * type must be documented by that {@code Lock} implementation.
     */
    void unlock();

    /**
    返回一个 Condition 实例，该实例绑定到调用了 newCondition() 方法的 Lock 实例对象上。
	在等待条件之前，锁必须由当前线程持有。
    对 Condition 的 await（）方法的调用将会在等待之前自动释放锁，并在等待返回之前重新获取锁
	
	实施注意事项
    {@link Condition}实例的确切操作取决于{@code Lock}实现，并且必须由该实现记录。
     * Returns a new {@link Condition} instance that is bound to this
     * {@code Lock} instance.
     *
     * <p>Before waiting on the condition the lock must be held by the
     * current thread.
     * A call to {@link Condition#await()} will atomically release the lock
     * before waiting and re-acquire the lock before the wait returns.
     *
     * <p><b>Implementation Considerations</b>
     *
     * <p>The exact operation of the {@link Condition} instance depends on
     * the {@code Lock} implementation and must be documented by that
     * implementation.
     *
     * @return A new {@link Condition} instance for this {@code Lock} instance
     * @throws UnsupportedOperationException if this {@code Lock}
     *         implementation does not support conditions
     */
    Condition newCondition();
}

```





### 对 Lock 接口下最重要的实现  ReentrantLock 的研究

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