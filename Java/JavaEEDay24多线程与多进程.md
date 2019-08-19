---
tags: 
- 进程
- 线程
---

# JavaDay24 多线程与多进程

@toc

代码示例：
```java
package DemoDay24;

import org.junit.jupiter.api.Test;

/**使用线程实现同时视频和语音
 * @author GJXAIOU
 * @create 2019-07-24-20:54
 */

class VideoThread extends Thread{
    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println("视频中。。。。。");
        }
       
    }
}
class AudioThread extends Thread{
    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println("语音中。。。。。");
        }
        
    }
}

public class Demo1 {
    
    //方法一：使用main函数进行调试
//    public static void main(String[] args) {
//        VideoThread videoThread = new VideoThread();
//        AudioThread audioThread = new AudioThread();
//
//        videoThread.start();
//        audioThread.start();
//    }

    //方法二：使用JUnit中@test进行调试
    @Test
    public void test(){
        VideoThread videoThread = new VideoThread();
        AudioThread audioThread = new AudioThread();

        videoThread.start();
        audioThread.start();
    }

}
```
程序运行结果：
每次运行结果会不同，进程会别抢占；
```java
语音中。。。。。
语音中。。。。。
语音中。。。。。
语音中。。。。。
语音中。。。。。
视频中。。。。。
视频中。。。。。
视频中。。。。。
视频中。。。。。
视频中。。。。。
```

## 一、线程中的常用方法

方法名  | 含义  | 说明
---|---|---
Thread(String name);  |初始化线程的名字|属于线程的一个有参数的构造方法
setName(String name); |修改线程的名字
getName(); | 获取线程的名字
sleep(); |static静态方法，通过Thread类名调用，这里需要处理一些异常，要求当前线程睡觉多少毫秒;|【哪一个线程执行了sleep方法，哪一个线程就睡觉】。
currentThead(); |static静态方法，返回当前的线程对象;|【哪一个线程执行了currentThread方法，就返回哪一个线程对象】。
getPriority(); |返回当前线程的优先级 |CPU执行的优先级，不是绝对的，仅仅是提升概率。
setPriority(int newPriority); |设置线程的优先级。

- 【注意】
  		线程的优先级范围是从1 ~ 10， 10最高，1最低
  		这里的优先级只是提高了当前线程拥有CPU执行权的概率，并不能完全保证当前线程能够一定会占用更多的CPU时间片。线程的默认优先级为5。
  
  Thread[main,5,main]
  Thread[Thread-0,5,main]
   Thread[线程名， 优先级， 线程组名]
   
- 线程中常见方法的测试：
==run()方法中不能抛出异常，只能使用 try-catch==
```java
package DemoDay24;

/**
 * @author GJXAIOU
 * @create 2019-07-24-21:23
 */
public class Demo2 extends Thread {

    public Demo2(String name) {
        super(name);//调用父类Thread的有参构造方法
    }

    
    @Override
    public void run() {
        //这里是Demo2线程对象的线程代码
        System.out.println("28:" + Thread.currentThread());

        for (int i = 0; i < 5; i++) {
            System.out.println("自定义线程");
		/*
		 在其他方法中， 使用sleep方法，可以抛出，可以捕获，
		 但是在run方法为什么只有捕获没有抛出？因为这是一个语法规则：
		 在Java中，重写父类的方法，要求和父类的方法声明一模一样，
		 在Thread类中，run方法没有抛出异常，所以在子类中，你也不能抛出异常，要和父类一致
		 */
            try {
                sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        //这里是main线程
        Demo2 d = new Demo2("狗蛋");

        d.setName("狗娃");
        d.setPriority(10);
        d.start(); //开启自定义线程，执行自定义线程中的run方法里面的功能
        System.out.println("39:" + Thread.currentThread());

        for (int i = 0; i < 5; i++) {
            System.out.println("这里是main线程");
            sleep(100);
        }
    }
}


```
程序运行结果：
```java
39:Thread[main,5,main]
这里是main线程
28:Thread[狗娃,10,main]
自定义线程
这里是main线程
自定义线程
自定义线程
这里是main线程
这里是main线程
自定义线程
自定义线程
这里是main线程

```
下面的 InterfaceA 和 testA 是为了测试什么时候是不能使用抛出异常，只能使用 try-catch
```java
package com.qfedu.a_thread;

interface A {
	public void testA();
}

public class Demo2 extends Thread implements A{
	public Demo2(String name) {
		super(name);//调用父类Thread的有参构造方法
	}
	
	@Override
	public void testA() {
		//这里也无法抛出异常，两种处理方法，第一种，捕获异常，
		//第二种，在接口中声明方法部分，声明该异常
		try {
			Thread.sleep(100);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}	
	}
//--------------------------------------------------	
//下面代码省略
}

```

---

## 二、线程的生命周期

![线程的生命周期]($resource/%E7%BA%BF%E7%A8%8B%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)

---

## 三、线程的共享资源问题

以动物园买票为示例：
一共有 50张票 3个窗口同时售卖
这里隐含多线程，这里可以把3个窗口看做是3个线程

- 第一次问题:
发现票每一张都被买了三次
  - 原因：
因为Ticket是在每一个线程中run方法里面的一个局部变量，这个局部变量是每一个线程对象都拥有的,这里Ticket就是不在是一个共享资源
  - 处理方式：
把Ticket变成成员变量

- 第二次问题：
发现貌似每一张票还都是卖了50次，而且这里还优化了售卖的算法
  - 原因：
这里Ticket变成了一个成员变量，在每一个线程对象中，都拥有这个Ticket成员变量，每一个成员变量是一个独立的个体，不是共享资源
  - 处理方式:
用static修饰ticket成员变量，变成一个存放在数据共享区的一个静态成员变量

- 第三次问题：
发现会出现几张票是买了多次的
  - 原因：
因为窗口1在卖票的时候，还没有运行到ticket--这条语句的时候，下一个窗口2开始执行卖票算法
这里窗口2卖的票是窗口1还没有ticket--的票
![线程安全问题]($resource/%E7%BA%BF%E7%A8%8B%E5%AE%89%E5%85%A8%E9%97%AE%E9%A2%98.png)
处理方式：
上锁，锁门

- **Java中的线程同步机制**：
方式1：
同步代码块：

```java
synchronized (锁对象) {
  //需要同步的代码;
}
```
同步代码块的注意事项：
1. 锁对象，可以是任意的一个对象， 但是必须是同一个对象！！！不能在这里使用new 来创建匿名对象
2. sleep() 不会释放锁对象，不会开锁。例如： 厕所有人关门睡着了
3. 使用synchronized 同步代码块的时候，必须是真正意义上存在共享资源的线程问题，才会使用
而且通常情况下，用synchronized锁住的代码越少越好，提高代码执行效率
```java
package com.qfedu.a_thread;


class SaleTicket extends Thread {
  private static int ticket = 50;

	public SaleTicket(String name) {
		super(name);
	}

	@Override
	public void run() {

		while (true) {
			synchronized ("你好") { 
			//可以使用"你好 "的任何确定的对象 
			//不能使用new Demo3()创建不同对象，即对应不同的锁 
				if (ticket > 0) {
					System.out.println(Thread.currentThread().getName()+
					":卖出来第" + ticket+ "张票");
					try {
						sleep(500); //睡眠也不会释放锁对象
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				} else {
					System.out.println("卖完了");
					break;
				}

				ticket--;
			}
		}
	}
}

public class Demo3 {
	public static void main(String[] args) {
		SaleTicket s1 = new SaleTicket("窗口1");
		SaleTicket s2 = new SaleTicket("窗口2");
		SaleTicket s3 = new SaleTicket("窗口3");

		s2.start();
		s1.start();
		s3.start();

	}
}
```
程序运行结果：
```java
窗口1:卖出来第50张票
窗口1:卖出来第49张票
。。。。。
窗口3:卖出来第28张票
窗口3:卖出来第27张票
窗口3:卖出来第26张票
窗口3:卖出来第2张票
窗口3:卖出来第1张票
卖完了
卖完了
卖完了
```

### （一）死锁
- 出现死锁的原因：
  	1.存在两个或两个以上的共享资源
  	2.存在两个或者两个以上的线程使用这些共享资源
下面的代码会出现死锁；
```java
package com.qfedu.a_thread;
class DeadLock extends Thread {
	public DeadLock(String name) {
		super(name);
	}
	
	@Override
	public void run() {
		if (Thread.currentThread().getName().equals("小胖")) {
			synchronized ("电池") {
				System.out.println("小胖有电池，想要遥控器");
				
				synchronized ("遥控器") {
					System.out.println("小胖拿到了遥控器，打开了投影仪");
				}
			}
		} else if (Thread.currentThread().getName().equals("逗比")) {
			synchronized ("遥控器") {
				System.out.println("逗比有遥控器，想要电池");
				
				synchronized ("电池") {
					System.out.println("逗比拿到了电池，打开了投影仪");
				}
			}
		}		
	}
}

public class Demo4 {
	public static void main(String[] args) {
		DeadLock d1 = new DeadLock("小胖");
		DeadLock d2 = new DeadLock("逗比");
		
		d1.start();
		d2.start();		
	}
}
```
输出结果：
```java
小胖有电池，想要遥控器
逗比有遥控器，想要电池
```

### （二）自定义线程
Java语言是一种单继承，多实现【遵从】面向对象的语言
- 自定义线程的方式：
  - 方式1：
	1.自定义一个类，继承Thread类
	2.重写Thread里面的run方法，把线程的功能代码放入到run方法中
	3.创建自定义线程类对象
	4.开启线程，使用start方法
    - 弊端：
	因为Java是一个单继承的语言，一旦某一个类继承了Thread类，就无法再继承其他类，或者说一个类继承了其他类，也就没有办法继承Thread类

  - 方式2:  **强烈推荐**
	【遵从】Runnable接口实现自定线程类
	1.自定义一个类，【遵从】Runnable接口
	2.实现Runnable接口中唯一要求的方法 Run方法，把线程的功能代码写入到run方法中
	3.创建Thread类对象，并且把【遵从】Runnable接口的自定义类对象，作为参数传入到Thread构造方法中
	4.调用Thread类对象的start方法，开启线程

```java
package com.qfedu.a_thread;
import java.util.Arrays;
import java.util.Comparator;

class TestRunnable implements Runnable {

	//实现自定义线程类，遵从Runnable接口要求实现的Run方法，把线程代码写入到Runnable里面
	@Override
	public void run() {
		for (int i = 0 ; i < 10; i++) {
			System.out.println("当前线程为:" + Thread.currentThread());
		}
	}	
}

public class Demo5 {
	public static <T> void main(String[] args) {
		//创建Thread类对象，调用Thread构造方法中，需要传入Runnable接口实现类对象的方法~
		//方法一：
		Thread t1 = new Thread(new TestRunnable());//匿名对象

		//方法二：
		Thread t2 = new Thread(new Runnable() { //匿名内部类的匿名对象，不再需要定义上面的TestRunnable类 
			
			@Override
			public void run() {
				for (int i = 0; i < 10; i++) {
					System.out.println("匿名内部类的匿名对象，作为方法的参数，这里是作为线程对象的参数" + 
							Thread.currentThread());
				}
			}
		});
		
		t1.start();
		t2.start();
		
		/*
		target是在创建Thread类对象时候，传入的【遵从】Runnable接口的实现类，这个实现类中
		实现类【遵从】Runnable接口要求实现的run方法，在run方法中，就是定义的线程代码
		在Thread类中有一个成员变量
		//What will be run
		private Runnable target; 
		
		@Override
		public void run() {
			if (target != null) {
				target.run();
			}
		}
		*/
	}
}
```

##  三、守护线程(后台线程) 

例如：
软件的Log日志文件，软件的自动更新，软件的自动下载

特征：
如果整个程序再运行过程中，只剩下一个守护线程，那么这个守护线程也就没有意义了，会自动停止

JVM的垃圾回收机制是守护线程。

这里当主线程停止，则下载也会自动停止；
```java
package com.qfedu.a_thread;

public class Demo6 extends Thread {
	
	public Demo6(String name) {
		super(name);
	}
	
	//模拟后台下载更新的线程
	@Override
	public void run() {
		for (int i = 0; i <= 100; i++) {
			System.out.println("软件更新下载中………………" + i + "%");
			
			if (i == 100) {
				System.out.println("软件更新下载完成，是否安装~~");
			}
			
			try {
				Thread.sleep(10);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
	
	public static void main(String[] args) throws InterruptedException {
		Demo6 d = new Demo6("后台线程");
		
		//设置当前线程为守护线程或者是后台线程
		d.setDaemon(true);//true为守护线程
		
		//System.out.println(d.isDaemon());
		d.start();
		for (int i = 0; i <= 50; i++) {
			Thread.sleep(10);
			System.out.println("主线程:" + i);
		}
	}
}
```

## 四、线程通讯
- 线程通讯:
	一个线程完成任务之后，通知另一个线程来完成该线程应该执行的任务
	
生产者和消费者问题:这里商品是两个线程直接的共享资源

wait(); 等待，如果一个线程执行了wait方法，那么这个线程就会进入临时阻塞状态，等待唤醒，这个唤醒必须其他线程调用notify() 方法唤醒;
notify(); 唤醒，唤醒线程池中进入【临时阻塞状态】的一个线程

- 注意事项：
1. wait()和notify()这两个方法都是Object类的方法
2. 在消费者生产者模式下，锁对象只能是商品；
```java
package com.qfedu.a_thread;

//两者之间的共享资源
class Product {
	String name; //商品的名字
	int price; //价格
	
	boolean flag = false; //产品是否生产成功，如果成功flag设置为true，消费者购买之后，设置为false
}

class Producer extends Thread {
	Product p;  //商品的类对象，是和消费者之间的共享资源
	
	public Producer(Product p) {
		this.p = p;
	}
	
	@Override
	public void run() {
		int count = 0;
		while (true) {
			synchronized (p) {
				try {
					sleep(100);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				if (!p.flag) { //p.flag == false
					
					//商品不存在，生产过程
					if (count % 2 == 0) {
						p.name = "红辣椒擀面皮";
						p.price = 5;
					} else {
						p.name = "唐风阁肉夹馍";
						p.price = 10;
					}
					
					count++;
					System.out.println("生产者生产了:" + p.name + ":" + p.price);
					p.flag = true;
					//生产结束，唤醒消费者
					p.notify();
				} else {
					//商品存在，要求消费者来购买，生产者进入临时阻塞
					try {
						p.wait();
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					} // try - catch
				}// if - else
			} // 同步代码块
		}// while (true)
	} //run()
}

class Customer extends Thread {
	Product p; //商品类对象，是和生产者之间的共享资源
	
	public Customer(Product p) {
		this.p = p;
	}
	
	@Override
	public void run() {
		while (true) {
			synchronized (p) {
				try {
					sleep(100);
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}
				if (p.flag) { //p.flag == true 
					//这里表示商品存在
					System.out.println("消费者购买了:" + p.name + ":" + p.price);
					
					p.flag = false; //表示消费者购买完毕，要求生产者生产
					//需要唤醒生产者
					p.notify(); //打开线程锁
				} else {
					try {
						//商品不存在，消费者进入临时阻塞
						p.wait();
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					} // try - catch
				}// if - else 
			} //同步代码块
		} // while (true)
	} //run()
}

public class Demo7 {
	public static void main(String[] args) {
		Product product = new Product();
		
		Producer p = new Producer(product);
		Customer c = new Customer(product);
		
		p.start();
		c.start();
	}
}

```

![可遇不可求的错误]($resource/%E5%8F%AF%E9%81%87%E4%B8%8D%E5%8F%AF%E6%B1%82%E7%9A%84%E9%94%99%E8%AF%AF.png)


