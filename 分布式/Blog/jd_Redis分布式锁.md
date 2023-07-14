## 1、为什么要有分布式锁？

JUC 提供的锁机制，可以保证在同一个 JVM 进程中同一时刻只有一个线程执行操作逻辑；

多服务多节点的情况下，就意味着有多个JVM进程，要做到这样，就需要有一个中间人；

分布式锁就是用来保证在同一时刻，仅有一个JVM进程中的一个线程在执行操作逻辑；

换句话说，JUC的锁和分布式锁都是一种保护系统资源的措施。尽可能将并发带来的不确定性转换为同步的确定性；

## 2、分布式锁特性（五大特性 非常重要）

 **特性1：互斥性**。在任意时刻，只有一个客户端能持有锁。

 **特性2： 不会发生死锁**。即使有一个客户端在持有锁的期间崩溃而没有主动解锁，也能保证后续其他客户端能加锁。

 **特性3： 解铃还须系铃人。**加锁和解锁必须是同一个客户端（线程），客户端自己不能把别人加的锁给解了。

 **特性4：可重入性**。同一个现线程已经获取到锁，可再次获取到锁。

 **特性5： 具有容错性**。只要大部分的分布式锁节点正常运行，客户端就可以加锁和解锁。

### **2-1 常见分布式锁的三种实现方式**

1. 数据库锁；2. 基于ZooKeeper的分布式锁；3. 基于Redis的分布式锁。

### **2-2 本文我们主要聊 redis实现**[**分布式**](https://so.csdn.net/so/search?q=分布式&spm=1001.2101.3001.7020)**锁：**

一个 setnx 就行了？value没意义？还有人认为 incr 也可以？再加个超时时间就行了？

## **3、分布式锁特性2之不会发生死锁**

很多线程去上锁，谁锁成功谁就有权利执行操作逻辑，其他线程要么直接走抢锁失败的逻辑，要么自旋尝试抢锁；

•比方说 A线程竞争到了锁，开始执行操作逻辑（代码逻辑演示中，使用 Jedis客户端为例）；

```
public static void doSomething() {
    // RedisLock是封装好的一个类
    RedisLock redisLock = new RedisLock(jedis); // 创建jedis实例的代码省略，不是重点
    try {
        redisLock.lock(); // 上锁
        
        // 处理业务
        System.out.println(Thread.currentThread().getName() + " 线程处理业务逻辑中...");
        Thread.sleep(2000);
        System.out.println(Thread.currentThread().getName() + " 线程处理业务逻辑完毕");
        
        redisLock.unlock(); // 释放锁
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

•正常情况下，A 线程执行完操作逻辑后，应该将锁释放。如果说执行过程中抛出异常，程序不再继续走正常的释放锁流程，没有释放锁怎么办？所以我们想到：

•**释放锁的流程一定要在 finally{} 块中执行，当然，上锁的流程一定要在 finally{} 对应的 try{} 块中，否则 finally{} 就没用了，如下：**

```
public static void doSomething() {
    RedisLock redisLock = new RedisLock(jedis); // 创建jedis实例的代码省略，不是重点
    try {
        redisLock.lock(); // 上锁，必须在 try{}中
        
        // 处理业务
        System.out.println(Thread.currentThread().getName() + " 线程处理业务逻辑中...");
        Thread.sleep(2000);
        System.out.println(Thread.currentThread().getName() + " 线程处理业务逻辑完毕");
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        redisLock.unlock(); // 在finally{} 中释放锁
    }
}
```

**写法注意： redisLock.lock();** **上分布式锁，必须在** **try{}中。**

在JAVA多线程中 lock.lock(); 单机多线程加锁操作需要在try{}之前。

### 3-1 redisLock.unlock() 放在 finally{} 块中就行了吗？还需要设置超时时间

如果在执行 try{} 中逻辑的时候，程序出现了 System.exit(0); 或者 finally{} 中执行异常，比方说连接不上 redis-server了；或者还未执行到 finally{}的时候，JVM进程挂掉了，服务宕机；这些情况都会导致没有成功释放锁，别的线程一直拿不到锁，怎么办？如果我的系统因为一个节点影响，别的节点也都无法正常提供服务了，那我的系统也太弱了。所以我们想到必须要将风险降低，可以给锁设置一个超时时间，比方说 1秒，即便发生了上边的情况，那我的锁也会在 1秒之后自动释放，其他线程就可以获取到锁，接班干活了；

```
public static final String lock_key = "zjt-lock";
 
     public void lock() {		
		while (!tryLock()) {
			try {
				Thread.sleep(50); // 在while中自旋，如果说读者想设置一些自旋次数，等待最大时长等自己去扩展，不是此处的重点
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
		
		System.out.println("线程：" + threadName + "，占锁成功！★★★");
	 }
 
 	 private boolean tryLock() {
		SetParams setParams = new SetParams();
		setParams.ex(1); // 超时时间1s
		setParams.nx();  // nx
		String response = jedis.set(lock_key, "", setParams); // 转换为redis命令就是：set zjt-key "" ex 1 nx
		return "OK".equals(response);
	 }
```

注意，上锁的时候，设置key和设置超时时间这两个操作要是**原子性**的，要么都执行，要么都不执行。

Redis原生支持：

```
// http://redis.io/commands/set.html
SET key value [EX seconds] [PX milliseconds] [NX|XX]
```

不要在代码里边分两次调用：

```
set k v
exipre k time
```

### 3-2 锁的超时时间该怎么计算？

刚才假设的超时时间 1s是怎么计算的？这个时间该设多少合适呢？

锁中的业务逻辑的执行时间，一般是我们在测试环境进行多次测试，然后在压测环境多轮压测之后，比方说计算出平均的执行时间是 200ms，锁的超时时间放大3-5倍，比如这里我们设置为 1s，为啥要放大，因为如果锁的操作逻辑中有网络 IO操作，线上的网络不会总一帆风顺，我们要给网络抖动留有缓冲时间。另外，如果你设置 10s，果真发生了宕机，那意味着这 10s中间，你的这个分布式锁的服务全部节点都是不可用的，这个和你的业务以及系统的可用性有挂钩，要衡量，要慎重（后边3-13会再详细聊）。那如果一个节点宕机之后可以通知 redis-server释放锁吗？注意，我是宕机，不可控力，断电了兄弟，通知不了的。

回头一想，如果我是优雅停机呢，我不是 kill -9，也不是断电，这样似乎可以去做一些编码去释放锁，你可以参考下 JVM的钩子、**Dubbo的优雅停机**、或者 linux进程级通信技术来做这件事情。当然也可以手动停服务后，手动删除掉 redis中的锁。

## 4、分布式锁特性3:解铃还须系铃人

如果说 A线程在执行操作逻辑的过程中，别的线程直接进行了释放锁的操作，是不是就出问题了？

什么？别的线程没有获得锁却直接执行了释放锁？？现在是 A线程上的锁，那肯定只能 A线程释放锁呀！别的线程释放锁算怎么回事？联想 ReentrantLock中的 isHeldByCurrentThread()方法，所以我们想到，必须在锁上加个标记，只有上锁的线程 A线程知道，相当于是一个密语，也就是说释放锁的时候，首先先把密语和锁上的标记进行匹配，如果匹配不上，就没有权利释放锁；

```
   private boolean tryLock() {
		SetParams setParams = new SetParams();
		setParams.ex(1); // 超时时间1s
		setParams.nx();  // nx
		String response = jedis.set(lock_key, "", setParams); // 转换为redis命令就是：set zjt_key "" ex 1 nx
		return "OK".equals(response);
	}
  
    // 别的线程直接调用释放锁操作，分布式锁崩溃！
 	public void unlock() {
		jedis.del(encode(lock_key));
		System.out.println("线程：" + threadName + " 释放锁成功！☆☆☆");
	}
 
 	private byte[] encode(String param) {
		return param.getBytes();
	}
```

### 4-1 这个密语value（约定）设置成什么呢？

很多同学说设置成一个 UUID就行了，上锁之前，在该线程代码中生成一个 UUID，将这个作为秘钥，存在锁键的 value中，释放锁的时候，用这个进行校验，因为只有上锁的线程知道这个秘钥，别的线程是不知道的。这个可行吗，当然可行。

```
   String releaseLock_lua = "if redis.call(\"get\",KEYS[1]) == ARGV[1] \n" + 
				"then\n" + 
				"    return redis.call(\"del\", KEYS[1])\n" + 
				"else\n" + 
				"    return 0\n" + 
				"end";
    
    private boolean tryLock(String uuid) {
		SetParams setParams = new SetParams();
		setParams.ex(1); // 超时时间1s
		setParams.nx();  // nx
		String response = jedis.set(lock_key, uuid, setParams); // 转换为redis命令就是：set zjt-key "" ex 1 nx
		return "OK".equals(response);
	}
 
 	public void unlock(String uuid) {
		
		List<byte[]> keys = Arrays.asList(encode(lock_key));
		List<byte[]> args = Arrays.asList(encode(uuid));
           
           // 使用lua脚本，保证原子性
		long eval = (Long) jedis.eval(encode(releaseLock_lua), keys, args);
		if (eval == 1) {
			System.out.println("线程：" + threadName + " 释放锁成功！☆☆☆");
		} else {
			System.out.println("线程：" + threadName + " 释放锁失败！该线程未持有锁！！！");
		}
		
	}
 
 	private byte[] encode(String param) {
		return param.getBytes();
	}
```

为什么使用 lua脚本？因为保证原子性

因为是两个操作，如果分两步那就是：

```
get k // 进行秘钥 value的比对
del k // 比对成功后，删除k
```

如果第一步比对成功后，第二步还没来得及执行的时候，锁到期，然后紧接着别的线程获取到锁，里边的 uuid已经变了，也就是说持有锁的线程已经不是该线程了，此时再执行第二步的删除锁操作，肯定是错误的了。

## **5.分布式锁特性4之可重入性**

作为一把锁，我们在使用 synchronized、ReentrantLock的时候是不是有可重入性？

那咱们这把分布式锁该如何实现可重入呢？如果 A线程的锁方法逻辑中调用了 x()方法，x()方法中也需要获取这把锁，按照这个逻辑，x()方法中的锁应该重入进去即可，那是不是需要将刚才生成的这个 UUID秘钥传递给 x()方法？怎么传递？用参数传递就会侵入业务代码

### 5-1 不侵入业务代码实现可重入：Thread-Id

我们主要是想给上锁的 A线程设置一个只有它自己知道的秘钥，把思路时钟往回拨，想想：

线程本身的 id（Thread.currentThread().getId()）是不是就是一个唯一标识呢？我们把秘钥 value设置为线程的 id不就行了。

```
   String releaseLock_lua = "if redis.call(\"get\",KEYS[1]) == ARGV[1] \n" + 
				"then\n" + 
				"    return redis.call(\"del\", KEYS[1])\n" + 
				"else\n" + 
				"    return 0\n" + 
				"end";
    String addLockLife_lua = "if redis.call(\"exists\", KEYS[1]) == 1\n" + 
				"then\n" + 
				"    return redis.call(\"expire\", KEYS[1], ARGV[1])\n" + 
				"else\n" + 
				"    return 0\n" + 
				"end";
    	
     public void lock() {
             // 判断是否可重入
		if (isHeldByCurrentThread()) {
			return;
		}
		
		while (!tryLock()) {
			try {
				Thread.sleep(50); // 自旋
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
		
		System.out.println("线程：" + threadName + "，占锁成功！★★★");
	}
 
   // 是否是当前线程占有锁，同时将超时时间重新设置，这个很重要，同样也是原子操作
 	private boolean isHeldByCurrentThread() {
		
		List<byte[]> keys = Arrays.asList(encode(lock_key));
		List<byte[]> args = Arrays.asList(encode(String.valueOf(threadId)), encode(String.valueOf(1)));
		
		long eval = (Long) jedis.eval(encode(addLockLife_lua), keys, args);
		return eval == 1;
	}
    
    private boolean tryLock(String uuid) {
		SetParams setParams = new SetParams();
		setParams.ex(1); // 超时时间1s
		setParams.nx();  // nx
		String response = jedis.set(lock_key, String.valueOf(threadId), setParams); // 转换为redis命令就是：set zjt-key xxx ex 1 nx
		return "OK".equals(response);
	}
 
 	public void unlock(String uuid) {
		
		List<byte[]> keys = Arrays.asList(encode(lock_key));
		List<byte[]> args = Arrays.asList(encode(String.valueOf(threadId)));
           
        // 使用lua脚本，保证原子性
		long eval = (Long) jedis.eval(encode(releaseLock_lua), keys, args);
		if (eval == 1) {
			System.out.println("线程：" + threadName + " 释放锁成功！☆☆☆");
		} else {
			System.out.println("线程：" + threadName + " 释放锁失败！该线程未持有锁！！！");
		}
		
	}
 
 	private byte[] encode(String param) {
		return param.getBytes();
	}
```

### 5-2 Thread-Id 真能行吗？不行。

想想，我们说一个 **Thread的id是唯一**的，是在同一个 JVM进程中，是在一个操作系统中，也就是在一个机器中。而现实是，我们的部署是集群部署，多个实例节点，那意味着会存在这样一种情况，S1机器上的线程上锁成功，此时锁中秘钥 value是线程id=1，如果说同一时间 S2机器中，正好线程id=1的线程尝试获得这把锁，比对秘钥发现成功，结果也重入了这把锁，也开始执行逻辑，此时，我们的分布式锁崩溃！怎么解决？我们只需要在每个节点中维护不同的标识即可，怎么维护呢？应用启动的时候，使用 **UUID生成一个唯一标识 APP_ID，放在内存中（或者使用zookeeper去分配机器id等等）**。此时，我们的**秘钥 value这样存即可：APP_ID+ThreadId**

```
   // static变量，final修饰，加载在内存中，JVM进程生命周期中不变
   private static final String APP_ID = UUID.randomUUID().toString();

    String releaseLock_lua = "if redis.call(\"get\",KEYS[1]) == ARGV[1] \n" + 
				"then\n" + 
				"    return redis.call(\"del\", KEYS[1])\n" + 
				"else\n" + 
				"    return 0\n" + 
				"end";
    String addLockLife_lua = "if redis.call(\"exists\", KEYS[1]) == 1\n" + 
				"then\n" + 
				"    return redis.call(\"expire\", KEYS[1], ARGV[1])\n" + 
				"else\n" + 
				"    return 0\n" + 
				"end";
    	
     public void lock() {
             // 判断是否可重入
		if (isHeldByCurrentThread()) {
			return;
		}
		
		while (!tryLock()) {
			try {
				Thread.sleep(50); // 自旋
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
		
		System.out.println("线程：" + threadName + "，占锁成功！★★★");
	}
 
    // 是否是当前线程占有锁，同时将超时时间重新设置，这个很重要，同样也是原子操作
 	private boolean isHeldByCurrentThread() {
		
		List<byte[]> keys = Arrays.asList(encode(lock_key));
		List<byte[]> args = Arrays.asList(encode(APP_ID + String.valueOf(threadId)), encode(String.valueOf(1)));
		
		long eval = (Long) jedis.eval(encode(addLockLife_lua), keys, args);
		return eval == 1;
	}
    
    private boolean tryLock(String uuid) {
		SetParams setParams = new SetParams();
		setParams.ex(1); // 超时时间1s
		setParams.nx();  // nx
		String response = jedis.set(lock_key, APP_ID + String.valueOf(threadId), setParams); // 转换为redis命令就是：set zjt-key xxx ex 1 nx
		return "OK".equals(response);
	}
 
 	public void unlock(String uuid) {
		
		List<byte[]> keys = Arrays.asList(encode(lock_key));
		List<byte[]> args = Arrays.asList(encode(APP_ID + String.valueOf(threadId)));
           
           // 使用lua脚本，保证原子性
		long eval = (Long) jedis.eval(encode(releaseLock_lua), keys, args);
		if (eval == 1) {
			System.out.println("线程：" + threadName + " 释放锁成功！☆☆☆");
		} else {
			System.out.println("线程：" + threadName + " 释放锁失败！该线程未持有锁！！！");
		}
		
	}
 
 	private byte[] encode(String param) {
		return param.getBytes();
	}
```

### 5-3 APP_ID（实例唯一标识） + ThreadId 还是 UUID 好呢？

继续听我说，如果 A线程执行逻辑中间开启了一个子线程执行任务，这个子线程任务中也需要重入这把锁，因为子线程获取到的线程 id不一样，导致重入失败。那意味着需要将这个秘钥继续传递给子线程，JUC中 InheritableThreadLocal 派上用场，但是感觉怪怪的，因为线程间传递的是父线程的 id。

微服务中多服务间调用的话可以借用系统自身有的 traceId作为秘钥即可。比如**sgm中的traceId**  **或者** **利用RPC框架的隐式传参**

「至于选择哪种 value的方式，根据实际的系统设计 + 业务场景，选择最合适的即可，没有最好，只有最合适。」

### 5-4、锁重入的超时时间怎么设置？

注意，我们上边的主要注意力在怎么重入进去，而我们这是分布式锁，要考虑的事情还有很多，重入进去后，超时时间随便设吗？

比方说 A线程在锁方法中调用了 x()方法，而 x()方法中也有获取锁的逻辑，如果 A线程获取锁后，执行过程中，到 x()方法时，这把锁是要重入进去的，但是请注意，这把锁的超时时间如果小于第一次上锁的时间，比方说 A线程设置的超时时间是 1s，在 100ms的时候执行到 x()方法中，而 x()方法中设置的超时时间是 100ms，那么意味着 100ms之后锁就释放了，而这个时候我的 A线程的主方法还没有执行完呢！却被重入锁设置的时间搞坏了！这个怎么搞？

如果说我在内存中设置一个这把锁设置过的最大的超时时间，重入的时候判断下传进来的时间，我重入时 expire的时候始终设置成最大的时间，而不是由重入锁随意降低锁时间导致上一步的主锁出现问题

放在内存中行吗？我们上边举例中，调用的 x()方法是在一个 JVM中，如果是调用远程的一个 RPC服务呢（像这种调用的话就需要将秘钥value通过 RpcContext传递过去了）到另一个节点的服务中进行锁重入，这个时间依然是要用当前设置过锁的最大时间的，所以这个**最大的时间要存在 redis中而非 JVM内存中**

经过这一步的分析，我们的重入 lua脚本就修改为这样了：

```
	ADD_LOCK_LIFE("if redis.call(\"get\", KEYS[1]) == ARGV[1]\n" + 	// 判断是否是锁持有者
				"then\n" + 
				"    local thisLockMaxTimeKeepKey=KEYS[1] .. \":maxTime\"\n" +  // 记录锁最大时间的key是：锁名字:maxTime
				"    local nowTime=tonumber(ARGV[2])\n" +  // 当前传参进来的time
				"    local maxTime=redis.call(\"incr\", thisLockMaxTimeKeepKey)\n" + // 取出当前锁设置的最大的超时时间，如果这个保持时间的key不存在返回的是字符串nil，这里为了lua脚本的易读性，用incr操作，这样读出来的都是number类型的操作
				"    local bigerTime=maxTime\n" + // 临时变量bigerTime=maxTime
				"    if nowTime>maxTime-1\n" +    // 如果传参进来的时间>记录的最大时间
				"    then\n" + 
				"        bigerTime=nowTime\n" + // 则更新bigerTime
				"        redis.call(\"set\", thisLockMaxTimeKeepKey, tostring(bigerTime))\n" + // 设置超时时间为最大的time，是最安全的
				"    else \n" + 
				"        redis.call(\"decr\", thisLockMaxTimeKeepKey)\n" + // 当前传参time<maxTime，将刚才那次incr减回来
				"    end\n" + 
				"    return redis.call(\"expire\", KEYS[1], tostring(bigerTime))\n" + // 重新设置超时时间为当前锁过的最大的time
				"else\n" + 
				"    return 0\n" + 
				"end"),
```

其实，还有另外一种方案比较简单，就是锁的超时时间=第一次上锁的时间+后面所有重入锁的时间。也就是（expire = 主ttl + 重入exipre），这种方案是放大的思想，一放大就又有上边提到过的一个问题：expire太大怎么办，参考上边。 

### 5-5、重入锁的方法中直接执行 unlock？考虑重入次数

A线程执行一共需要500ms，执行中需要调用 x()方法，x()方法中有一个重入锁，执行用了 50ms，然后执行完后，x()方法的 finally{} 块中将锁进行释放。

为啥能释放掉？因为秘钥我有，匹配成功了我就直接释放了。

这当然是有问题的，所以我们要通过**锁重入次数**来进行释放锁时候的判断，也就是说上锁的时候需要多维护一个 key来保存当前锁的重入次数，如果执行释放锁时，先进行重入次数 -1，-1后如果是0，可以直接 del，如果>0，说明还有重入的锁在，不能直接 del。

### 5-6 考虑如何存储锁的属性（锁的key  重入次数key  最大超时时间key）？

目前为止，算上上一步中设置最大超时时间的key，加上这一步重入次数的key，加上锁本身的key，已经有3个key，需要注意的事情是，这三个key的超时时间是都要设置的！为什么？假如说重入次数的 key没有设置超时时间，服务A节点中在一个JVM中重入了5次后，调用一次 RPC服务，RPC服务中同样重入锁，此时，锁重入次数是 6，这个时候A服务宕机，就意味着无论怎样，这把锁不可能释放了，这个分布式锁提供的完整能力，全线不可用了！

所以，这几个 key是要设置超时时间的！怎么设置？我上一个锁要维护这么多 key的超时时间？太复杂了吧，多则乱，则容易出问题。怎么办？我们想一下，是不是最大超时时间的 key和重入次数的 key，都**附属于锁**，它们都是**锁的属性**，如果锁不在了，谈它们就毫无意义，这个时候用什么存储呢？**redis的 hash数据结构**，就可以做，key是锁，里边的 hashKey分别是锁的属性， hashValue是属性值，超时时间只设置锁本身 key就可以了。这个时候，我们的锁的数据结构就要改变一下了。

![img](jd_Redis%E5%88%86%E5%B8%83%E5%BC%8F%E9%94%81.resource/2022-10-26-14-48x2frUK2BL9PuRBd.png)



## **6、如何解决过期时间确定和业务执行时长不确定性的问题：看门狗机制**

3-2中设置超时时间那里，我们预估锁方法执行时间是 200ms，我们放大 5倍后，设置超时时间是 1s（**过期时间确定**）。假想一下，如果生产环境中，锁方法中的 IO操作，极端情况下超时严重，比方说 IO就消耗了 2s（**业务执行时长不确定**），那就意味着，在这次 IO还没有结束的时候，我这把锁已经到期释放掉了，就意味着别的线程趁虚而入，分布式锁崩溃！

我们要做的是一把分布式锁，想要的目的是同一时刻只有一个线程持有锁，作为服务而言，这个锁现在不管是被哪个线程上锁成功了，我服务应该保证这个线程执行的安全性，怎么办？**锁续命（看门狗机制）**。什么意思，一旦这把锁出现了上锁操作，就意味着这把锁开始投入使用，这时我的服务中需要有一个 **daemon线程定时去守护我的锁的安全性**，怎么守护？比如说锁超时时间设置的是 1s，那么我这个定时任务是每隔 300ms去 redis服务端做一次检查，如果我还持有，你就给我续命，就像 session会话的活跃机制一样。看个例子，我上锁时候超时时间设置的是 1s，实际方法执行时间是 3s，这中间我的定时线程每隔 300ms就会去把这把锁的超时时间重新设置为 1s，每隔 300ms一次，成功将锁续命成功。

```
public class RedisLockIdleThreadPool {
    private String threadAddLife_lua = "if redis.call(\"exists\", KEYS[1]) == 1\n" + 
				"then\n" + 
				"    return redis.call(\"expire\", KEYS[1], ARGV[1])\n" + 
				"else\n" + 
				"    return 0\n" + 
				"end";
 
	private volatile ScheduledExecutorService scheduledThreadPool;
	
	public RedisLockIdleThreadPool() {
		
		if (scheduledThreadPool == null) {
			synchronized (this) {
				if (scheduledThreadPool == null) {
					scheduledThreadPool = Executors.newSingleThreadScheduledExecutor(); // 我这样创建线程池是为了代码的易读性，大家务必使用ThreadPoolExecutor去创建
					
					scheduledThreadPool.scheduleAtFixedRate(() -> {
						addLife();
					}, 0, 300, TimeUnit.MILLISECONDS);
				}
			}
		}
	}
	
	private void addLife() {
            // ... 省略jedis的初始化过程
            
		List<byte[]> keys = Arrays.asList(RedisLock.lock_key.getBytes());
		List<byte[]> args = Arrays.asList(String.valueOf(1).getBytes());
		
		jedis.eval(threadAddLife_lua.getBytes(), keys, args);
	}
	
}
```

这就行吗？还不行！

为啥？想一下，如果每个服务中都像这样去续命锁，假如说A服务还在执行过程中的时候，还没有执行完，就是说还没有手动释放锁的时候，宕机，此时 redis中锁还在有效期。服务B 也一直在续命这把锁，此时这把锁一直在续命，但是 B的这个续命一直续的是 A当时设的锁，这不是扯吗？我自己在不断续命，导致我的服务上一直获取不到锁，实际上 A已经宕机了呀！该释放了，不应该去续命了，这不是我服务 B该干的活！

续命的前提是，得判断是不是当前进程持有的锁，也就是我们的 APP_ID，如果不是就不进行续命。

续命锁的 lua脚本发生改变，如下：

```
	THREAD_ADD_LIFE("local v=redis.call(\"get\", KEYS[1]) \n" + 	// get key
				"if v==false \n" +  // 如果不存在key，读出结果v是false
				"then \n" + 		// 不存在不处理
				"else \n" + 
				"    local match = string.find(v, ARGV[1]) \n" + // 存在，判断是否能和APP_ID匹配，匹配不上时match是nil
				"    if match==\"nil\" \n" + 
				"    then \n" + 
				"    else  \n" + 
				"        return redis.call(\"expire\", KEYS[1], ARGV[2]) \n" + // 匹配上了返回的是索引位置，如果匹配上了意味着就是当前进程占有的锁，就延长时间
				"    end \n" + 
				"end")
```

### 6-1 锁在我手里，我挂了，这...  没救。只能等待锁超时释放

即便设置了一个很合理的 expire，比如 10s，但是线上如果真出现了A节点刚拿到锁就宕机了，那其他节点也只能干等10s，之后才能拿到锁。主要还是业务能不能接受。而如果是 To C的业务中，大部分场景无法接受的，因为可能会导致用户流失。所以我们需要另外**一个监控服务**，定时去**监控 redis中锁的获得者的健康状态**，如果获取者超过n次无法通信，由监控服务负责将锁摘除掉，让别的线程继续去获取到锁去干活。

## **7 文章参考标注**

分布式锁的特性以及看门狗机制的应用借鉴自技术好文：

Redis分布式锁正确打开方式 http://jnews.jd.com/circle-info-detail?postId=210273 。