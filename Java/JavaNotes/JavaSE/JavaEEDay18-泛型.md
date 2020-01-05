---
tags : 
- java基础

flag: yellow
---

@toc

# JavaDay18 泛型

## 一、复习
### 1.ArrayList
   
 ArrayList底层维护的是一个Object类型的数组，使用无参构造方法，创建一个ArrayList集合对象.默认的元素个数为10
- 特征：
   -  查询快，增删慢        
   - 查询快：
            使用的是数组的下标访问方式，可以直达目标位置        
   - 增删慢：
      -  增加: 有可能会触发数组的扩容机制，会创建一个新的数组，新的数组元素个数大约是原数组的1.5倍。然后会有一个数组拷贝的过程，这个过程是将原数组里面的每一个元素挨个复制到新数组中，这个操作消耗大量的时间和空间;
      - 增加的原理是：`newCapacity = oldCapacity + (oldCapacity >> 1); `          
      - 删除:  是从Object数组中删除是一个元素，删除之后，后面元素会向前移动，移动的过程是一个复制的过程，这个操作，比较浪费时间;
        
####  Vector 
线程安全，效率较低的ArrayList JDK1.0         

### 2.LinkedList:
  底层维护的是一个链表 《数据结构》
  增删快，查找慢
        
    
###   3.HashSet
 
底层维护的是一个哈希表，存储效率极高
        
- 存储原理：
    调用存入对象的hashCode方法，获取到该对象的hashCode的值，通过【移位 计算】,计算该对象应该放到哈希表的哪一个位置       
  - 情况1:
            该位置没有任何元素，直接添加
        
  - 情况2：
      该位置存在其他元素，这时候会调用对象的equals方法，和在该位置保存的其他元素进行一一比较，如果所有的比较结果都为false，表示不相同，可以添加。如果出现任何一个true，表示和该位置其他的元素相同，不能添加
    
###   4.TreeSet
存入TreeSet的元素必须有自然顺序或者是存在比较方式

自定义类对象，想要放入到TreeSet集合中，有两种方式：
- 该自定义类【遵从】Comparable接口，实现compareTo(Object o) 方法
- 给TreeSet提供自定义比较器，创建自定义比较器需要【遵从】Comparator接口实现compare(Object o1, Object o2);

---

## 二、泛型

### （一）引言
【问题】：
- 在 ArrayList 中可以放入任意类型的数据，但实际操作中数据类型不一致会导致更多的错误；
- 例如下列代码中，就算明知取出的数据时 string 类型，但是还是得通过强制类型转换才能得到 string 类型数据；
```java
ArrayList list = new ArrayList();
list.add(new Demo1); //Demo1为该类名称
list.add("hello");
String str = (String)(list.get(1)); //必须通过强转才能获得 
```

【期望】：
集合中的数据类型能够统一，即涉及到数据类型一致化问题；
【解决方式】：
泛型（JDK1.5 之后）

### （二）使用
泛型作用：
- 解决集合中数据类型不一致的问题，要求保存什么数据，只能保存什么数据，否则报错，即将异常提前；
- 从集合中取数据，保存的是什么数据，取出也是什么数据，不需要强制类型转换

标准格式：
`ArrayList<String> arrayList = new ArrayList<String>();`
代码示例：
```java
package study;
import java.util.ArrayList;

public class FanXin_Use {
	public static void main(String[] args) {
		//这里<String>即为泛型，要求这个ArrayList集合中有且只能保存String类型的数据
		ArrayList<String> arrayList = new ArrayList<String>();
		
		arrayList.add("hello");
		arrayList.add("world");
		//无法保存其他数据类型
		//arrayList.add(1);
		
		String string = arrayList.get(1);
		System.out.println(string);
	}
}

```

### （三）泛型在函数中的使用
【需求】
定义一个方法，可以接受任意数据类型，而且要求返回的数据类型，就是你传入的数据类型；例如：传入 String 类型，返回 string 类型，传入 Demo1 类型，返回 Demo1 类型；

泛型的使用需要：
  占位符，即一个大写字母，只是一个占位符，没有实际含义，不同地方定义的占位符没有联系；

泛型在函数中使用的格式：
```
修饰符 <声明的自定义泛型占位符> 返回值类型(可以使用自定义泛型) 函数名(形式参数列表（可以使用泛型）){
    函数体（函数体中，所有使用到自定义泛型的地方，数据类型都被传入数据类型替换）
}
```

代码示例：
```java
package study;

public class FanXin_Fun {
	public static void main(String[] args) {
		String string = getType("hello"); //这里传入的“hello”是字符串类型，因此调用的时候，所有的E相当于就是字符串类型；
		FanXin_Fun fanXin_Fun = getType(new FanXin_Fun());
		int num = getType(5);
	}
	//<E>是自定义泛型的占位符，表示在该函数中可以使用占位符E，而E的具体数据类型由传入的参数控制，这样的操作可以让函数多元化，更加简单；
	public static <E> E getType(E e) {
		return e;
	}
}
```
上面代码中 num 是 Integer 类型，是一个包装类

**包装类：**
Java 是完全面向对象的语言，在 Java 中万物皆对象，如果是要保存类对象，那么八大基本数据类型就无法使用，所以，Java 提供了一个包装机制，包装基本数据类型，让其变成类对象；称为自动封箱；
基本数据类型 | 封装之后
---| ---
short|Short
int|Integer
byte|Byte
long|Long
double|Double
float|Float
boolean|Boolean
char|Character
如果使用包装类直接赋值给普通的基本数据类型，该操作称为 拆箱；


### （四）在类内使用泛型和匿名内部类
格式：
```
class 类名<自定义泛型的占位符> {
    //在这里所用到的泛型和用户创建对象时候声明的是一致的；
}
```

注意事项：
- 一个类声明的自定义泛型，如果在创建该类对象的时候，确定了泛型的具体数据类型，那么整个类内所有用到该泛型占位符的非静态成员方法，使用的数据类型都是创建时候确定的类型；
- 如果创建使用了自定义泛型类对象，但是没有确定泛型的具体类型，那么编译器会把这个泛型认为是 Object 类型；
- 类中声明的自定义泛型，不能在类中的静态方法使用，如果想让静态方法使用泛型，则需要自己声明、自己使用，类似于方法中使用泛型；
- 4.建议：如果在代码中出现了多个使用不同泛型的地方，使用不同名字的占位符，一般常用的占位符为：T 和 E

代码示例：
```java
package study;

import java.util.Comparator;

//异常类
class InvalidArrayException extends Exception{
	public InvalidArrayException(String message) {
		super(message);
	}
}

class InvalidComparatorException extends Exception{
	public 	InvalidComparatorException (String message) {
		super(message);
	}
}


//泛型类
class ArrayTools<A>{
	/**
	 * 利用泛型，来满足不同数据类型的排序算法，可以在创建类对象时约束
	 * @param array A类型，泛型的数组，可以是任意类型
	 * @param com <? super A> 是A类型的比较器或者其父类的比较器
	 * @throws InvalidArrayException 数组无效异常
	 * @throws InvalidComparatorException 比较器无效异常
	 */
	public void selectSortUsingCompare(A[ ] array , Comparator<? super A> com) //这里的?super A表示可以传入A及其父类的数据类型
		throws InvalidArrayException,InvalidComparatorException{
			//参数合法性判断
		if (null == array || array.length == 0) {
			throw new InvalidArrayException("数组无效");
		}else if (null == com) {
			throw new InvalidComparatorException("比较器无效");			
		}
		
		for (int i = 0; i < array.length - 1; i++) {
			int index = i;
			
			for (int j = i; j < array.length; j++) {
				if (com.compare(array[index], array[j]) > 0) {
					index = j;
				}
			}
			
			if (index != i) {
				A temp = array[index];
				array[index] = array[i];
				array[i] = temp;
			}
		}
		
	}	
	
	public void printArray(A[]  array) {
		for (A a : array) {
			System.out.println(a);
		}
	}
	
	public static <T> void test(T a) {  //因为静态方法比类加载早，所以如果想要使用泛型，则需要自己声明 ，类里面的泛型与之无关
		System.out.println(a);
	}
}

public class FanXin_class {
	public static void main(String[] args) throws InvalidArrayException,InvalidComparatorException{
		Integer [] array = {1,3,4,3,6,2,8,1};
		ArrayTools<Integer> tools = new ArrayTools<Integer>();
		
		tools.selectSortUsingCompare(array, new Comparator<Integer>() {

			@Override
			public int compare(Integer arg0, Integer arg1) {
				return arg0 - arg1;
			}
		
		});
		
		tools.printArray(array);
	}
}


```


### （五） 接口中使用泛型

在接口中定义泛型：
格式 :
```java
interface 接口名<自定义泛型的占位符> {
  //成员变量  缺省属性：public static final 定义时必须初始化
  //成员方法  缺省属性：abstract
}
```


两种【遵从】带有自定义泛型的接口方式：
1. 更加自由，需要使用的泛型类型，在创建对象时确定，类似ArrayList
2. 适合原本这个类就没有使用泛型的情况，例如：一个类遵从Comparable接口 实现 compareTo方法，这里可以在【遵从】时，确定Comparable需要的泛型具体数据类型，减少没有必要的强制类型转换

代码示例：
```java
interface A<T> {
	public void testA(T t); //这个方法中使用了定义接口时声明的自定义泛型
}

//PlanA
//一个类【遵从】接口，而且类中声明的自定义泛型和接口泛型一致，没有确定泛型的具体类型，由调用者来确定
class TestClass1<T> implements A<T> {

	@Override
	public void testA(T t) {
		System.out.println(t.getClass() + "类型！！！");
	}	
}

//PlanB
//一个类【遵从】接口，但是接口的泛型已经被确定的数据类型替代
class TestClass2 implements A<String> {

	@Override
	public void testA(String t) {
		System.out.println("String类型的方法");
	}
	
}

public class Demo5 {
	public static void main(String[] args) {
		TestClass1<Integer> test = new TestClass1<Integer>();
		
		test.testA(5);
		
		TestClass2 test2 = new TestClass2();
		
		test2.testA("233333");
	}
}


```

### （六）泛型的上下限

- 泛型的上下限
	<? super T>
	<? extends T>
	
	T 泛型的占位符
	? 通配符（表示一个字符）
	super: 调用父类方法的关键字
	extends: 继承的关键字
	
- 需求：定义一个函数接受任意类型数值的集合，但是这个数据必须是数值类型
	数值类型：Number 已知子类：Integer Short Long Double Float (包装类)
	
	要求传入的对象是Number类对象或者其子类对象
		Integer extends Number; //Integer 是 Number 的子类
		Float extends Number;
		Double extends Number;
		
	? extends Number;//这就是泛型的上限：
	<? extends E> //通用类型
			
- 需求：定义一个函数能够传入一个任意类型的集合，但是要求集合里面保存的数据必须是Number类对象，或者其父类对象
		<? super E>泛型的下限
		例如：<? super Number>
			能够保存的数据是Number类对象本身或者其父类对象
  
```java
package com.qfedu.a_generticity;

import java.util.ArrayList;
import java.util.Collection;

public class Demo6 {
	public static void main(String[] args) {
		ArrayList<Number> list1 = new ArrayList<Number>();
		ArrayList<Double> list2 = new ArrayList<Double>();
		ArrayList<String> list3 = new ArrayList<String>();
		ArrayList<Object> list4 = new ArrayList<Object>();
		
		//传入的实际参数是一个ArrayList对象，里面保存的数据是Number类型
		test1(list1);
		test1(list2);
		//test1(list3); 不行，因为ArrayList<String> 里面保存的String类型不是Number的子类
		
		test2(list1);
		test2(list4);
	}
	
	/**
	 * 
	 * @param c Collection<? extends Number> 要求传入的是一个Collection集合接口的实现类
	 * 而且要求该实现类里面保存的数据是Number类对象本身或者其子类对象
	 */
	public static void test1(Collection<? extends Number> c) {
		System.out.println(c.toString());
	}
	
	public static void test2(Collection<? super Number> c) {
		System.out.println("泛型的下限！！！");
	}
}
```



## 三、Map
- 回顾：
  ---| Collection
  ------| List
  ----------| ArrayList      查询快 增删慢
  ----------| LinkedList    查询慢，增删快
  ----------| Vector         线程安全的ArrayList
  ------| Set
  ----------| HashSet       掌握它的存储原理
  ----------| TreeSet
  	----|比较器：
  		Comparable接口 实现compareTo方法
  		Comparator接口 实现compare方法
  					
生活中有关联，有关系的数据更多一点。例如：账号  密码；钥匙  锁


### Map

---| Map<K, V> 双列集合，这是一个接口
------| HashMap 实现类
------| TreeMap 

K：Key 键 !!! 唯一值!!! 不允许重复!!!
V：Value 值 一个键(Key)对应一个值(Value) 可以重复的

在Map<K, V> 双列集合中，保存的只能是一个键(Key)值(Value)对！！！


Map中要学习的方法：
- 增
  - put(K key, V value);    添加一个键(Key)值(Value)对
  - putAll(Map<? extends K, ? extends V> map);添加一个符合数据类型的Map双列集合
- 删
  - clear(); 清空所有的键(Key)值(Value)对
  - remove(Object key); 根据Key删除对应的键(Key)值(Value)对
- 改
  - put(K key, V value); 当键(Key)存在时，这个操作是重新修改值(Value)
- 查	 
  - size(); 获取键值对个数
  - get(Object key); 通过键(Key)找出对应的值(Value)
  - containsKey(Object key); 查看这个Key是否在Map中存在
  - containsValue(Object value); 查看这个Value是否在Map存在

  - keySet(); 返回所有键(Key)Set集合
  - values(); 返回所有值(Value)Collection集合
 
代码示例：
```java
package com.qfedu.b_map;

import java.util.Collection;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;


public class Demo1 {
	public static void main(String[] args) {
		Map<String, String> map = new HashMap<String, String>();
		
		//使用put(K k, V v)添加元素
		map.put("薛之谦", "高磊鑫");
		map.put("鹿晗", "关晓彤");
		map.put("宋仲基", "宋慧乔");
		map.put("余文乐", "王棠云");
		map.put("王宝强", "马蓉");
		
		System.out.println(map);
		
		Map<String, String> map2 = new HashMap<String, String>();
		
		map2.put("科比", "瓦妮莎");
		map2.put("TT", "卡戴珊");
		
		//添加另一个Map
		map.putAll(map2);
		
		System.out.println(map);
		
		//清空当前Map双列集合
		map2.clear();
		System.out.println(map2.isEmpty());
		
		//根据Key删除对应的键值对
		map.remove("TT");
		
		System.out.println(map);
		
		//当Key存在时，这个操作修改对应Value
		map.put("王宝强", null);
		System.out.println(map);
		
		
		System.out.println(map.size());
		
		System.out.println(map.containsKey("谢霆锋"));
		System.out.println(map.containsKey("薛之谦"));
		
		System.out.println(map.containsValue("高磊鑫"));
		System.out.println(map.containsValue("王菲"));
		
		System.out.println(map.get("科比"));
		System.out.println(map.get("TT"));
		
		Set<String> set = map.keySet();
		for (String string : set) {
			System.out.println(string);
		}
		System.out.println("-------------------------------");
		Collection<String> c = map.values();
		for (String string : c) {
			System.out.println(string);
		}
	
	}
}
```
