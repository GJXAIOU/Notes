


# JavaDay27 反射
## 一、反射总结
当一个Java文件编译之后，编译成一个 `.class` 文件，也就是字节码文件，当这个字节码文件【加载】到内存的方法区/代码区，JVM 会根据【加载】的字节码文件内容，创建一个 Class 的类对象。这个 Class 类对象里面包含了当前字节码文件里面的所有内容。


这个 Class 对象中包含了对应字节码文件的所有成员变量 `Field` ，所有成员方法 `Method`，构造方法 `Constructor`。

- 获取 Class 对象的三种方式：
  
    - **Class.forName("完整的类名字符串");**
            完整类名字符串是指 包名.类名
    - 类名.class;
    - 指定类对象.getClass();
    
- Constructor, Method, Field 
    是 Java 反射包 `java.lang.reflect` 里面的类 
    
    - Constructor 是构造方法类
    - Method 是成员方法类
    - Field 是成员变量类
    
    

### Constructor 常用方法：

返回值 | 方法名| 作用
---|---|----
Constructor[] | `getConstructors();` |获取所有 public 修饰的构造方法
  Constructor[] |`getDeclaredConstructors();`| 获取当前类里面所有的构造方法，包含用 private 修饰的构造方法 
Constructor |`getConstructor(Object... paramterTyeps);`|根据所需参数不同，获取指定的构造方法对象
Constructor| `getDeclaredConstructor(Object... paramterTyeps);` |根据所需参数不同，获取指定的构造方法对象，包括私有化的方法
 Object | `newInstance(Object... initargs);` |给予确定的参数，通过反射调用构造方法，这里的参数列表是一个不定参数列表

### Method 常用方法 

返回值 | 方法名| 作用
---|---|----
Method[]  |`getMethods();` |获取当前类里面所有的 public 修饰的成员方法，这里或显示父类继承而来的 public 方法
Method[] | `getDeclaredMethods();` |获取当前类里面的所有方法，包括 private 修饰的方法，但是会过滤父类继承而来的方法
Method  | `getMethod(String methodName, Object... args);` |根据方法的名字和对应的参数列表，获取指定方法
Method | `getDeclaredMethod(String methodName, Object... args);` |根据方法的名字和对应的参数列表，获取指定方法，可以获取private修饰的方法
 无    | `invoke(Object obj, Object... args);` |执行成员方法的函数，第一个参数是执行该方法的类对象，第二个参数是执行该方法需要的参数列表

### Field 常用方法

返回值 | 方法名| 作用
---|---|----
 Field[]  |`getFields();` | 获取所有的用 public 修饰的成员变量 
 Field[]| `getDeclaredFields();` | 获取所用成员变量，包括用 private 修饰的成员变量 
Field |`getField(String fieldName);`|根据成员变量的名字获取对应的成员变量
Field |`getDeclaredField(String fieldName);`|根据成员变量的名字获取包括 private 修饰在内的成员变量
无   | `set(Object obj, Object value);` |设置成员变量的数值，第一个参数是调用该成员变量的对象，第二个参数是赋予数值

### 暴力反射赋予权限的函数

`setAccessible(boolean)`

##  二、获取 Class 类 对象的三种方式   

1.首先新建一个 Person 类，作为示例
```java
package com.qfedu.a_reflect;

public class Person {
	private int id;
	private String name;
	public int test;
	public static int testStatic = 10;
	
	private Person() {}
	
	public Person(int id, String name) {
		this.id = id;
		this.name = name;
	}
	
	public int getId() {
		return id;
	}

	public void setId(int id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public static void eat() {
		System.out.println("黄焖鸡米饭~~~");
	}
	
	public void sleep(int num) {
		System.out.println(name + "每天睡" + num + "个小时");
	}
	
	public void game() {
		System.out.println("大吉大利，今晚吃鸡~~~");
	}
	
	private void testPrivate() {
		System.out.println("这是一个Private修饰的私有化方法");
	}

	@Override
	public String toString() {
		return "Person [id=" + id + ", name=" + name + "]";
	}
}

```

2.通过反射获取类对象
```java
package com.qfedu.a_reflect;

public class GetClassObject {
	public static void main(String[] args) throws ClassNotFoundException {
		//如果想要为所欲为，首先获取到Class类对象
		
		/*方式1：Class.forName("完整的类名字符串");
		完整类名是包括    包名.类名
		*/
		Class cls1 = Class.forName("com.qfedu.a_reflect.Person");
		System.out.println(cls1);
		
		//方式2：类名.class
		Class cls2 = Person.class;
		System.out.println(cls2);
		
		//方式3：通过对象获取到对应的Class类对象
		Class cls3 = new Person(1, "逗比").getClass();
		System.out.println(cls3);
		
		System.out.println(cls1 == cls2);
		System.out.println(cls2 == cls3);
		System.out.println(cls1 == cls3);
	}
}
```


## 三、Constructor 常用方法

通过 Class 类对象获取当前类的构造方法并且调用
```java
package com.qfedu.a_reflect;
import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationTargetException;

public class GetClassConstructor {
	public static void main(String[] args) throws 
	ClassNotFoundException, NoSuchMethodException, SecurityException, InstantiationException, IllegalAccessException, IllegalArgumentException, InvocationTargetException {
/*
 【加载】指定类的字节码文件，获取对应的Class对象
	下面的语句做了两件事情：
	1. 让JVM根据类名，加载Person.java对应的字节码文件Person.class 到内存的代码区
	2. 把加载到内存代码区Person.class 字节码文件生成一个Class类对象返回
*/
		Class cls = Class.forName("com.qfedu.a_reflect.Person");
		
		
	//Constructor 这是构造方法的类
	//第一种：可以通过Class获取所有的【非私有化构造方法】，方法是 getConstuctors();
		Constructor[] constructors = cls.getConstructors();
		
		for (Constructor constructor : constructors) {
			System.out.println(constructor);
		}
		System.out.println("---------------------------");


	//第二种：暴力反射（获取所有构造方法，不管是不是私有化）
		Constructor[] allConstructors = cls.getDeclaredConstructors();
		for (Constructor constructor : allConstructors) {
			System.out.println(constructor);
		}
		
	//第三种方法：
		/*
		 根据参数获取具体的构造方法 
		 getConstructor(Class... parameterTypes);
		 上面括号里面是不定参数列表，表示参数的类型要求是Class类型，但是数量不限制
		 在确定数据类型的情况下，可以直接通过数据类型.class获取对应Class数据类型
		 例如：
		 	int.class String.class 这里会自动包装为Integer.class和String.class
		 */
		System.out.println("----------------------------------");
		Constructor aConstructor = cls.getConstructor(int.class, String.class);
		System.out.println(aConstructor);
		
		

	//☆☆☆☆根据Constructor对象创建类对象，即调用构造函数

		/*
		 构造方法Constructor对象获取完毕，怎么利用Constructor对象创建一个Person类对象
		 newInstance(Object... initargs) 
		 也是一个不定参的方法，需要的参数都是Object类型的，参数个数不确定
		 */
		Person p = (Person) aConstructor.newInstance(1, "海洋");//默认产生是Object对象，因此需要强转
		System.out.println(p.getId() + ":" + p.getName());
		p.sleep(5);
		p.game();
		p.eat(); //如何通过反射机制，调用static修饰的成员方法，并且不报警告
		

		/*
		 通过暴力反射，借助于指定的参数，获取private修饰的无参构造方法 
		 */
		System.out.println("---------------------------------------");
		Constructor privateConstructor = cls.getDeclaredConstructor(null);
		System.out.println(privateConstructor);
		
		//这里需要通过setAccessible(boolean ) 给予操作Private修饰方法的权限
		privateConstructor.setAccessible(true);
		Person p2 = (Person) privateConstructor.newInstance(null);
		p2.setId(2);
		p2.setName("刘德华");
		System.out.println(p2);
		
		/*
		 单例和反射的共存：
		  	在实际开发中，如果一个类是一个单例类，那么一般不会有程序猿通过反射的方式来使用这个类里面
		  	私有化的构造方法，这违背了单例的原则
		 */
	}
}

```

**至此：上面除了 Person 类以及暴力反射之外所有代码的整理精简版如下**
```java
/**
 * @author GJXAIOU
 * @create 2019-08-10-13:56
 */
public class ReflectPractice {
    public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
        // 1.获取class类对象
        Class person = Class.forName("entity.Person");

        // 2.获取具体的构造方法
        // 获取无参构造方法
        Constructor constructor = person.getConstructor();
        // 获取带有参数的构造方法
        Constructor constructor1 = person.getConstructor(int.class,String.class);
        // 获取全部构造方法（非私有的）
        Constructor[] constructors = person.getConstructors();
        for (Constructor constructor2 : constructors) {
            System.out.println(constructor);
        }

        // 3.使用构造方法构建类对象,默认是Object类型，需要强转
        Person person1 = (Person) constructor.newInstance();
        Person person2 = (Person) constructor1.newInstance(2, "张三");

        // 4.使用成员变量和成员方法
        System.out.println(person1.getName());
        System.out.println(person2.getName());
    }
}
```

补充：单例
```java
package com.qfedu.a_reflect;

public class SingleDemo {
	
	int id;
	
	//为了判定数据，定义一个静态私有化成员变量，数据类型为该类的变量，保存上次创建的数据的类对象的首地址
	private static SingleDemo s = null;
	
	//1.私有化构造方法
	private SingleDemo(int id) {  
		this.id = id;
	}
	
	//2.提供一个类外可以通过类名直接调用的，返回值为当前类对象类型的，参数为对应需要的构造方法参数，
	//方法名通常为getInstance，作为类外获取方式
	public static SingleDemo getInstance(int id) {
		//但我们调用这个公开的，静态方法修饰的获取类对象的这种方法时，对这个保存地址的变量进行判定
		if (null == s) {
			s = new SingleDemo(id);
		}	
		return s; //返回上次创建对象
	}
}

```


## 四、Method 常用方法 
4.通过反射借助于Class类对象，获取这个类里面所有的成员方法,并且进行调用
```java
package com.qfedu.a_reflect;

import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

/**
 * 通过反射借助于Class类对象，获取这个类里面所有的成员方法
 * Method 就是成员方法类
 */
public class GetClassMethod {
	public static void main(String[] args) throws ClassNotFoundException, InstantiationException, IllegalAccessException, IllegalArgumentException, InvocationTargetException, NoSuchMethodException, SecurityException {
		//1.加载对应类的字节码文件，获取该类的Class类对象
		Class cls = Class.forName("com.qfedu.a_reflect.Person");
		
		//获取所有的公共的方法，这里也会获取一些额外Object里面公开的方法
		Method[] allPublicMethods = cls.getMethods();
		for (Method method : allPublicMethods) {
			System.out.println(method);
		}
		System.out.println("$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$");

		//暴力反射
		//能够获取Person里面的private方法， 并且能够过滤掉从父类继承而来的方法	
		Method[] allMethods = cls.getDeclaredMethods();
		for (Method method : allMethods) {
			System.out.println(method);
		}
		
		/*
		 通过反射机制，执行类中的成员方法 
		 invoke(Object obj, Object... args);
		 Object obj 这是底层调用该方法的类对象	
		 	the object the underlying method is invoked from
		 Object... args 不定参数，是执行该放的参数列表，是Object类型
		 	args is arguments used for method call
		 */
		//1.先利用反射，创建一个当前类的对象
		System.out.println("$$$$$$$$$$$$$$$$$$$$$$$$$$$$");
		Person p = (Person) cls.getConstructor(int.class, String.class).
				newInstance(1, "狗蛋");
		
		//获取一个指定的方法，需要的参数是方法的名字字符串和参数列表，
		Method aPublicMethod = cls.getMethod("sleep", int.class);
		System.out.println(aPublicMethod);
		
		aPublicMethod.invoke(p, 15);
		System.out.println("$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$");
		
		//获取一个静态方法
		Method aPublicStaticMethod = cls.getMethod("eat", null);
		aPublicStaticMethod.invoke(null, null);
		
		//利用暴力反射获取一个私有化的成员方法
		Method aPrivateMethod = cls.getDeclaredMethod("testPrivate", null);
		aPrivateMethod.setAccessible(true);
		aPrivateMethod.invoke(p, null);
		
	}
}
```

## 五、Field常用方法
5.通过反射获取Class类对象里面所有的成员变量
```java
package com.qfedu.a_reflect;

import java.lang.reflect.Field;
import java.lang.reflect.InvocationTargetException;

/**
 * 通过反射获取Class类对象里面所有的成员变量
 * Field 成员变量类
 *
 */

public class GetClassField {
	public static void main(String[] args) throws ClassNotFoundException, NoSuchFieldException, SecurityException, InstantiationException, IllegalAccessException, IllegalArgumentException, InvocationTargetException, NoSuchMethodException {
		//加载字节码文件，获取Class类对象
		Class cls = Class.forName("com.qfedu.a_reflect.Person");

		//获取所有用public修饰的成员变量
		Field[] allPublicFields = cls.getFields();

		for (Field field : allPublicFields) {
			System.out.println(field);
		}
		System.out.println("$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$");
		
		//暴力反射，获取私有化成员变量
		Field[] allFields = cls.getDeclaredFields();
		for (Field field : allFields) {
			System.out.println(field);
		}
		System.out.println("$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$");
		
		//获取一个公开的成员变量
		Field aPublicField = cls.getField("test");
		System.out.println(aPublicField);

		
		//set方法
		//set(Oject obj, Object value);
		//第一个参数: 要操作的是哪一个对象里面的成员变量
		//第二个参数: 需要设置的值
		
		//首先获得对象，然后调用set方法
		Person p = (Person) cls.getConstructor(int.class, String.class).
				newInstance(1, "狗蛋");
		aPublicField.set(p, 20);
		
		System.out.println(p.test);
		System.out.println("$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$");
		
		//静态的
		Field aStaticField = cls.getField("testStatic");
		System.out.println(aStaticField);
		aStaticField.set(null, 20);
		System.out.println(Person.testStatic);
		
		//暴力反射：私有化的
		System.out.println("$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$");
		Field aPrivateField = cls.getDeclaredField("id");
		System.out.println(aPrivateField);
		aPrivateField.setAccessible(true);
		aPrivateField.set(p, 10);
		System.out.println(p.getId());
	
	}
}
```
