# Spring AOP(上）

AOP应该是写过写的东西，但最近项目中又接触到，拿来复盘下。

我默认大家已经掌握知乎专栏《动态代理》的内容，所以这里不会再解释什么是动态代理。

主要内容：

- AOP的一些概念
- 为什么需要切点表达式

老实说，没学SpringBoot之前，AOP难度要大很多。因为配置略微繁琐，而培训班又讲得太复杂了，一下子抛出很多概念，诸如连接点、切入点、织入等，其实就单纯使用AOP来说，这些概念根本没必要提。

个人认为，要掌握AOP只需要了解以下4个概念：

- 通知方法
- 目标对象
- 切面
- 切点表达式

Spring AOP底层也是动态代理，所以我们按动态代理的思路去理解AOP：

![img](https://pic1.zhimg.com/80/v2-5bc843c1dd1327dd08ce9254ba2a0e74_720w.jpg)

- 通知方法：打印日志
- 目标对象：target

调用代理对象proxyCalculator.add()就会在调用target.add()的前后加上日志打印。

你会发现，单纯的动态代理并没有所谓“切面”和“切点表达式”的概念，这两个概念是Spring AOP对动态代理的扩展。那么，为什么Spring AOP要引入“切面”和“切入点表达式”呢？我们应该从动态代理上找原因。

一般来说，大部分人动态代理是这样写的：

```java
public class ProxyTest {
	public static void main(String[] args) throws Throwable {
		CalculatorImpl target = new CalculatorImpl();
		Calculator calculatorProxy = (Calculator) getProxy(target);
		calculatorProxy.add(1, 2);
		calculatorProxy.subtract(2, 1);
	}

	private static Object getProxy(final Object target) throws Exception {
		Object proxy = Proxy.newProxyInstance(
				target.getClass().getClassLoader(),/*类加载器*/
				target.getClass().getInterfaces(),/*让代理对象和目标对象实现相同接口*/
				new InvocationHandler(){/*代理对象的方法最终都会被JVM导向它的invoke方法*/
					public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
						System.out.println(method.getName() + "方法开始执行...");
						Object result = method.invoke(target, args);
						System.out.println(result);
						System.out.println(method.getName() + "方法执行结束...");
						return result;
					}
				}
		);
		return proxy;
	}
}
```

![img](https://pic4.zhimg.com/80/v2-7d64329d3aa5c5b42ae7d8dcbf69b72f_720w.jpg)

上面代码其实类似于下面这张图：

![img](https://pic3.zhimg.com/80/v2-f931e1e7a8903a95e81e8c4b5a7653e2_720w.jpg)

如果要实现不同的增强逻辑，只需写多个不同的getProxy()，然后需要代理时只要传入target就能得到增强。

动态代理在复用性上已经远远优于静态代理，但上面的代码还有改善空间。

主要在于ProxyTest的定位有问题，模糊的定位导致我们无法对ProxyTest做进一步优化。比如我问你，ProxyTest属于什么呢？Controller、Service、Dao还是Util？

而Spring则明确地把ProxyTest这样的类称为**切面类。**

**所谓的切面，其实就是把所有通知方法集中起来，都放在一个类中**，这个类叫做切面类。

![img](https://pic2.zhimg.com/80/v2-c1b0348ceee9a5309d3788eb4d78473d_720w.jpg)

有了明确的定位后，Spring并没有止步于此，毕竟单纯使用动态代理也可以编写“切面类”。那Spring AOP做了什么改进呢？

我们再来看看之前ProxyTest：

```java
@RestController
class Controller {
 
    @Autowired
    private UserService userService;
    
    @GetMapping("/test")
    public void get() {
       切面类 qiemian = new 切面类();
       Proxy proxyUserService = qiemian.getProxy(userService)
       proxyUserService.addUser();
    }
    
}
```

由于ProxyTest现在的定位是“管理所有的通知方法”，所以如果一个对象希望得到增强，必然要通过ProxyTest得到代理对象。

但有个问题！

如果我们希望对Service做日志记录，就需要在Controller中调用getProxy()得到Service的代理对象，**是显式地硬编码在Controller中的。**

坏处在于：

- 由于是硬编码，如果后期想要撤销日志打印的功能，需要手动一条条改
- 不够优雅

Spring希望消除这种硬编码的代理风格，改为**无感知**的动态代理，让程序员可以像使用普通对象一样使用代理对象（@Autowired注入时干脆就是代理对象）。

**那就必须制定目标对象和通知方法之间的对应规则。**

嗯，什么？目标对象和通知方法之间有对应规则吗？不是直接传入的吗？

是啊，正因为以前使用动态代理都是直接传入，所以久而久之很多人觉得它们之间没有对应关系。

我问你，一个系统中所有的对象都需要代理吗？显然不是吧。那么为什么有些对象有代理功能，有些对象没有呢？

因为你编码时手动把需要代理的对象传入getProxy()然后得到了代理对象。此时**对应的规则在你心中**，你在编写代码时根据自己的意愿指定哪些对象需要代理。

**但如果你希望Spring帮你完成自动代理，就必须把心中的规则具象化为一段代码，一段Spring能读懂的代码，这样它才可能按你的规则完成代理。**

**这套给Spring看的规则就是切点表达式。**

![img](https://pic3.zhimg.com/80/v2-ebdf528602f37b77767198f6243f2bf6_720w.jpg)

![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='1500' height='880'></svg>)

来看看Spring AOP有没有解决硬编码问题。

如果要想取消代理，纯动态代理方式需要跑到Controller中删除getProxy()方法调用：

![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='982' height='644'></svg>)

而Spring AOP是通过切点表达式控制，只需要注释切点表达式就阻止了这一切，**就像天上有无数风筝，但线被我攥在手里：**

![img](data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' width='1004' height='638'></svg>)

总而言之，切面类是为了统一管理通知方法，切点表达式定制哪些对象需要代理，以及如何代理（用哪个通知方法增强）。

具体的切点表达式规则以及AOP的实际案例，下一篇再写。我觉得概念最重要，AOP写起来倒是很简单。

- 

- 

- # 