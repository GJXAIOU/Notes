---
tags : 
- java基础

flag: yellow
---
@toc

# JavaEEDay15-异常处理

## 一、异常

Java开发中遇到的异常/错误也是一步一步造成的；


### （一） Throwable
Java中所有错误或者异常的超类；
Throwable 分类： 
  - Error : 错误，无法处理，只能避免；
  - Exception: 异常, 可以处理, 分两种情况：运行时异常和编译时异常；
     - 吃完饭发现没带钱包：运行时异常
     - 准备去吃饭没带钱包：编译时异常


研究Exception和Error先从Throwable开始；

- **Throwable类**
  - ==构造方法==：
    - Throwable(); //无参的构造方法，创建一个【异常信息】为null的Throwable类对象；
    - Throwable(String message); //创建一个【异常信息】为message的Throwable类对象；
  - ==成员方法==：
    - getMessage();  //获取Throwable类对象里面保存的【异常信息】//返回值类型为 String
    - toMessage();  //返回一个异常的简短描述 //返回值类型为 String
    - printStackTrace(); //在**控制台**中显示详细的异常问题 //返回值类型为 void 

示例代码
```java
package a_JavaException;
public class YiChangLeiLianXi {
    public static void main(String[] args) {
        //首先创建一个异常类，
        //调用的无参的构造方法，创建了一个异常信息为null的Throwable类对象
        //调用有参的构造方法，创建一个异常信息为message的Throwable类对象
        
        //Throwable throwable = new Throwable(); //调用无参构造方法
        Throwable throwable2 = new Throwable("这里出错啦"); //调用有参构造方法
                
        System.out.println(throwable2.getMessage());
        System.out.println(throwable2.toString());
        //throwable2.printStackTrace();

        testThrowable(); //调用该异常位置；
    }

    public static void testThrowable(){
        Throwable throwable = new Throwable("这里有问题"); //创建该异常位置；
        throwable.printStackTrace();
    }
}
```
对异常类的测试
![对异常类的测试]($resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20190627092658.jpg)

这里的==两个错误分别对应了创建该异常的位置和调用该异常的位置==；



- **区分Error和Exception：**
看提示的末尾即可；
代码示例：
```java
public class Demo2 {
	public static void main(String[] args) {
		//Error:这里假设系统的内存空间为8GB 
		long[] arr = new long[1024 * 1024 * 1024];
		//Exception in thread "main"java.lang.OutOfMemoryError: 
		//根据末尾知道这是错误：程序使用的内存空间超过了当前硬件限制
		
		int[] arr1 = null; //数组名为arr1的引用类型变量保存的是null;
		arr1[0] = 7;
	//Exception in thread "main" java.lang.NullPointerException
	//异常，空指针异常，表示当前代码操作Null地址空间
	}
}
```


## 二、异常的处理

Java程序运行的时候，会监听异常，发生异常的时候，会创建Throwable类对象，处理异常；
### (一)第一种方式：捕获异常：
```
try{
  //可能出现异常的代码
} catch（异常类型 e）{
  //异常的处理方式
}
```


==**使用try - catch的注意事项**：==
- 在没有捕获异常的情况下，程序中一旦发生了异常，程序就会在发生异常的地方停下来，不会继续执行后面的代码；
- 一旦异常被try-catch捕获，那么JVM就会认为当前代码已经不存在异常，可以继续运行；
- 在 try-catch 语句块里面，如果出现了多种异常，这里可以加上多个 catch 语句块，对不同的异常做区别对待，做到异常的精准匹配；
- 在 try- catch 语句块中，如果存在大面积的异常，可以放在整个 catch 语句块中的最后一个，如果放在前面可能导致其他的 catch 异常无效；（因为 Exception 是其他异常的父类，包含所有异常）

程序示例：
```java
package a_JavaException;
public class Demo3 {
	public static void main(String[] args) {
		test(1, 0, null);
	}
	
	public static void test (int num1, int num2, int[] arr) {
		int ret = 0;
		try {
			arr[0] = 5;
			ret = num1/num2; 
			//可能的异常有：分母为0以及NullPointerException
		}catch (ArithmeticException e) {
				e.printStackTrace();
		}catch (NullPointerException e) {
				e.printStackTrace();			
		} catch (Exception e) { 
		//这个异常包含所有的异常,除上面以外的所有异常均有这条异常处理
		}
		System.out.println("执行该命令");
	}
}
```
程序执行结果：
```
java.lang.NullPointerException
执行该命令
	at a_JavaException.Demo3.test(Demo3.java:21)
	at a_JavaException.Demo3.main(Demo3.java:15)
```


### （二）第二种方法：抛出异常

- ==使用关键字 throw 和 throws==；
  - throw 在方法体中使用，用于抛出异常的关键字；
    代码格式： `throw new 异常类型(可以有参或者无参，参数为返回的语句提示);`
  - throws 在方法声明之后，表示当前抛出的异常有哪些（当方法体中含有 throw 的时候作为搭配使用）
    代码格式：`函数方法（参数）throws 异常类型`

- **注意事项**：
  - 如果在代码中存在语句 `throw new 异常类型()`，那么这条语句之后的代码不会再执行；
  - 如果代码中存在多种异常需要抛出，不能将异常写在同一个代码块中，否则只能处理一个异常，剩下的异常都会变成不能触及的代码；
  - 不同的异常要分情况抛出，需要使用 `if -else if` 结构，在不同的 if 中抛出不同的异常；
  - 如果代码中出现了多种异常，需要在函数之后声明，不同的异常使用逗号分开;

- 捕获异常和抛出异常注意：
    捕获在哪里都可以，但是抛出异常一旦涉及到用户层面的，一定要不能抛出，必须捕获；

**对于调用一个抛出异常的方法处理方式：**
- 处理当前异常；
- 继续抛出异常

```java
package a_JavaException;
public class Throw_Use {
	public static void main(String[] args) {
		//如果调用一个存在抛出异常的方法，通常有两种处理方法
		//第一种：处理当前异常
		try {
			testThrow(null, 1, 2);
		} catch (NullPointerException e) {
			e.printStackTrace();
		}catch (ArithmeticException e) {
			e.printStackTrace();
		}
	}	
	
	//第二种方案：继续向外抛出异常，就只需要在调用该方法的函数后面添加 throws 异常类型即可
		//	public static void main(String[] args) throws NullPointerException, ArithmeticException {
		//		testThrow(null, 1, 2);
		//	}
	
	public static void testThrow(int[] arr, int num1, int num2)
	throws NullPointerException, ArithmeticException{
		//之前使用的参数合法性判断
//		if (arr == null || arr.length == 0) {
//			System.out.println(" ");
//			return;
//		}
		
		if (arr == null || arr.length == 0) {
			throw new NullPointerException("测试一下Throw");
			//这里无论输入什么语句都不再执行
		}else if (num2 == 0) {
			throw new ArithmeticException();
		}					
	}	
}

```

### （三）自定义异常

==自定义异常==定义格式：
```
class 自定义异常类名 extends Exception{
  public 自定义异常类名(String message){ //构造方法必须实现
    super(message); //这里调用的是Exception中的message，最终调用的是Throwable中的getmessage方法；    
  }
}
```


代码示例：
```java
package a_JavaException;

public class ZiDingYiException  {
	public static void main(String[] args)throws NoGrilException{
		//以下的捕获或者抛出二选一；
		//进行捕获
		try {
			buyOneFreeOne(false);
		} catch (NoGrilException e) {
			e.printStackTrace();
		}
		
		//进行抛出
		buyOneFreeOne(true);  //这里是抛出，所以上面需要加throws
	}
		
	public static void buyOneFreeOne(boolean isLonely) throws NoGrilException {
		if (isLonely) {
			//直接创建自定义异常的匿名类对象，通过throw关键字跳出
			throw new NoGrilException("不用，谢谢");
		}
		System.out.println("阔以");
	}
	
}


class NoGrilException extends Exception {
	//String message 是当前异常的详细信息，用来传递给Exception的构造方法，保存该异常信息；
	public NoGrilException(String message) {
		super(message);
	}
}

```














