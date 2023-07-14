# 第八章 J.U.C之其他组件

Future 接口测试

```java
package com.gjxaiou.concurrency.example.aqs;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

/**
 * @author GJXAIOU
 */
@Slf4j
public class FutureExample {

    static class MyCallable implements Callable<String> {

        @Override
        public String call() throws Exception {
            // 让线程 sleep 一会，相当于完成某件事情需要较长时间
            log.info("do something in callable");
            Thread.sleep(5000);
            return "Done";
        }
    }

    public static void main(String[] args) throws Exception {
        ExecutorService executorService = Executors.newCachedThreadPool();
        // 接收另外一个线程执行的结果
        Future<String> future = executorService.submit(new MyCallable());
        // 另一个线程在做其他事情
        log.info("do something in main");
        Thread.sleep(1000);
        // 获取之前任务返回值，如果值没有计算完，或阻塞等待，这里可以看到最终日志前面有 5s
        String result = future.get();
        log.info("result：{}", result);
    }
}

```



## FutureTask
在介绍 Callable 时我们知道它可以有返回值，返回值通过 `Future<V>` 进行封装。
```java
public class FutureExample {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executorService= Executors.newCachedThreadPool();
        // 直接在里面定义任务
        Future<Integer> future=executorService.submit(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                int result = 0;
                for (int i = 0; i < 100; i++) {
                    Thread.sleep(10);
                    result += i;
                }
                return result;
            }
        });

        Thread otherThread = new Thread(() -> {
            System.out.println("other task is running...");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        // 启动任务
        otherThread.start();
        // 获取任务结果
        Integer ret=(Integer) future.get();
        System.out.println(ret);
    }
}
//输出结果：
//other task is running...
//5050
```

FutureTask 实现了 RunnableFuture 接口，该接口继承自 Runnable 和 `Future<V>` 接口，
这使得 FutureTask 既可以当做**一个任务执行**，也可以有**返回值**。

```java
public class FutureTask<V> implements RunnableFuture<V>
```

```java
public interface RunnableFuture<V> extends Runnable, Future<V>
```

FutureTask 可用于异步获取执行结果或取消执行任务的场景。
当一个计算任务需要执行很长时间，那么就可以用 FutureTask 来封装这个任务，主线程在完成自己的任务之后再去获取结果。

```java
public class FutureTaskExample {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<Integer> futureTask = new FutureTask<Integer>(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                int result = 0;
                for (int i = 0; i < 100; i++) {
                    Thread.sleep(10);
                    result += i;
                }
                return result;
            }
        });

        Thread computeThread = new Thread(futureTask);
        computeThread.start();

        Thread otherThread = new Thread(() -> {
            System.out.println("other task is running...");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        otherThread.start();
        System.out.println(futureTask.get());
    }
}
//输出结果：
//other task is running...
//5050
```

## BlockingQueue

java.util.concurrent.BlockingQueue 接口有以下阻塞队列的实现：

-  **FIFO 队列** ：LinkedBlockingQueue、ArrayBlockingQueue（固定长度）
-  **优先级队列** ：PriorityBlockingQueue
-  DelayQueue：可指定按照什么进行排序（一般按照过期时间）
-  SynchronousQueue：同步队列，内部仅仅存放一个值。

提供了阻塞的 take() 和 put() 方法：
如果队列为空进行 take() 将阻塞，直到队列中有内容；如果队列为满进行 put() 将阻塞，直到队列有空闲位置（即有另一个线程进行了出队列操作）。

**使用 BlockingQueue 实现生产者消费者问题** 

```java
public class ProducerConsumer {

    private static BlockingQueue<String> queue = new ArrayBlockingQueue<>(5);

    private static class Producer extends Thread {
        @Override
        public void run() {
            try {
                queue.put("product");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.print("produce..");
        }
    }

    private static class Consumer extends Thread {

        @Override
        public void run() {
            try {
                String product = queue.take();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.print("consume..");
        }
    }
}
```

```java
public static void main(String[] args) {
    for (int i = 0; i < 2; i++) {
        Producer producer = new Producer();
        producer.start();
    }
    for (int i = 0; i < 5; i++) {
        Consumer consumer = new Consumer();
        consumer.start();
    }
    for (int i = 0; i < 3; i++) {
        Producer producer = new Producer();
        producer.start();
    }
}
/**
 * 输出结果：
 produce...
 produce...
 consume...
 consume...
 produce...
 consume...
 produce...
 consume...
 produce...
 consume...
 */
```

## Fork/Join 框架

主要用于并行计算中，和 MapReduce 原理类似，都是把大的计算任务拆分成多个小任务并行计算。

```java
public class ForkJoinExample extends RecursiveTask<Integer> {

    private final int threshold = 5;
    private int first;
    private int last;

    public ForkJoinExample(int first, int last) {
        this.first = first;
        this.last = last;
    }

    @Override
    protected Integer compute() {
        int result = 0;
        if (last - first <= threshold) {
            // 任务足够小则直接计算
            for (int i = first; i <= last; i++) {
                result += i;
            }
        } else {
            // 如果任务大于阈值就拆分成 2 个子任务
            int middle = first + (last - first) / 2;
            ForkJoinExample leftTask = new ForkJoinExample(first, middle);
            ForkJoinExample rightTask = new ForkJoinExample(middle + 1, last);
            // 执行子任务
            leftTask.fork();
            rightTask.fork();
            // 合并子任务
            result = leftTask.join() + rightTask.join();
        }
        return result;
    }
}
```

```java
public static void main(String[] args) throws ExecutionException, InterruptedException {
    // 生成一个任务，这里表示 1 + 2 + 。。。+ 100
    ForkJoinExample task = new ForkJoinExample(1, 100);
    ForkJoinPool forkJoinPool = new ForkJoinPool();
    // 执行一个任务
    Future result = forkJoinPool.submit(task);
    System.out.println(result.get());
}
//输出结果：5050
```

ForkJoin 使用 ForkJoinPool 来启动，它是一个特殊的线程池，线程数量取决于 CPU 核数。

```java
public class ForkJoinPool extends AbstractExecutorService
```

ForkJoinPool 实现了工作窃取算法来提高 CPU 的利用率。每个线程都维护了一个双端队列，用来存储需要执行的任务。
工作窃取算法允许空闲的线程从其它线程的双端队列中窃取一个任务来执行。
窃取的任务必须是**最晚的任务**，避免和队列所属线程发生竞争。
例如下图中，Thread2 从 Thread1 的队列中拿出最晚的 Task1 任务，Thread1 会拿出 Task2 来执行，这样就避免发生竞争。
但是如果队列中只有一个任务时还是会发生竞争。

<div align="center"> <img src="pics//JUCOthers/jucOthers.jpg" width="600"/> </div>


