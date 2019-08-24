---
tags: 
- IO流
- JavaAPI
---
@toc

## 一、 IO流
分为：输入流和输出流    
字节流和字符流
- 字节流：
  - InputStream 
      - FileInputStream
  - OutputStream
      - FileOutputStream
    
- 字符流：
  - Reader
      - FileReader
  - Writer  
      - FileWriter

- 注意：
  - 1.使用缓冲效率更高，原因是解决了内存访问硬盘的次数过多导致的时间上的浪费，通常缓冲流使用的缓冲空间一般都是4KB或者8KB，为了迎合硬盘读取的特征，正是一个扇区的大小；
  - 2.FileWriter 不是直接把数据写入到磁盘，而是在内存中建立了一个缓冲区，用于保存用户想要写入到硬盘的数据，有三种情况才会真正的写入数据到硬盘：
    1> 缓冲区满了
    2> 调用flush，清空缓冲区
    3> FileWriter输出管道关闭
  - 3.字节流和字符流选择，
    字节流基本上可以满足所有的文件内容传输需求
    字符流，个人建议 只用来处理记事本可以打开的可视化文件


## 二、缓冲

昨天学习字符流和字节流的时候，发现如果使用了缓冲，时间效率更高
  	
Sun提供了Java自己的缓冲机制：字节缓冲流和字符缓冲流

---| InputStream 输入字节流的基类/超类 抽象类
------| FileInputStream 文件操作的字节输入流
------| BufferedInputStream 缓冲输入字节流，在缓冲字符流对象中，底层维护了一个8kb缓冲字节数组

- 构造方法：
	BufferedInputStream(InputStream in);
	BufferedInputStream(InputStream in, int size);	
构造方法中都有一个参数是InputStream, 要求传入的是InputStream的子类对象（多态），第二个构造方法中多了一个参数是int size ，这个size表示设置缓冲区的大 小， 默认缓冲数组的大小是一个8kb的字节数组
	构造方法中的InputStream是给缓冲流提供读写能力的！！！
	
- 【记住】
		**缓冲流是没有读写能力的，需要对应的字节流或者字符流来提供**
		
- 使用流程：
  - 1.找到目标文件
  - 2.建立管道
  	- a) 首先创建当前文件的InputStream的子类对象，FileInputStream
  	-  b) 使用InputStream的子类对象，作为BufferedInputStream构造方法参数，创建缓冲流对象
  - 3.读取数据
  - 4.关闭资源
### （一）输入字节缓冲
```java
package com.qfedu.a_buffer;

import java.io.BufferedInputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;


public class StreamInputBuffered {
	public static void main(String[] args) throws IOException {
		readTest1();
	}
	
	public static void readTest1() throws IOException {
		//1. 找到文件
		File file = new File("C:/aaa/1.txt");
		
		//判断他是否是一个普通文件，是否存在
		if (!file.exists() || !file.isFile()) {
			throw new FileNotFoundException();
		}
		
		//2. 建立管道
		//创建FileInputStream提供读写能力
		FileInputStream fis = new FileInputStream(file);
		
		//利用FileInputStream对象，创建对应的BufferedInputStream
		BufferedInputStream bs = new BufferedInputStream(fis);
		
		//这种方式将上面所有语句合成这一句话，前提是文件必须存在；
		//BufferedInputStream bs2 = new BufferedInputStream(
		//		new FileInputStream(new File("C:/aaa/1.txt")));
		
		//3. 读取数据（准备一个缓冲数组）
		int length = -1;
		byte[] buffer = new byte[512];
		
		while ((length = bs.read(buffer)) != -1) {
			System.out.println(new String(buffer, 0, length));
		}
		
		
		//4. 关闭资源
		bs.close();
		//在BufferedInputStream的close方法中，该方法会自动关闭创建缓冲流时使用的输入字节流对象：FileInputStream对象
		//fis.close();//不需要关闭了
	}
}	
```


### （二） 输出字节缓冲
StreamOutputBuffered.java
能用缓冲就不用字节流
```java
package com.qfedu.a_buffer;

import java.io.BufferedInputStream;
import java.io.BufferedOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;

public class StreamOutputBuffered {
	public static void main(String[] args) throws IOException {
		copyFile();
		//WriteTest();
	}
	
	public static void copyFile() throws IOException {
		//1. 找到源文件
		File srcFile = new File("C:\\Users\\刘晓磊\\Desktop\\颈椎操.avi");
		
		if (!srcFile.exists() || !srcFile.isFile()) {
			throw new FileNotFoundException();
		}
		
		//2. 确定目标文件
		File dstFile = new File("C:\\Users\\刘晓磊\\Desktop\\颈椎操2.avi");
		
		//3. 建立输入输出管道
		FileInputStream fis = new FileInputStream(srcFile);
		FileOutputStream fos = new FileOutputStream(dstFile);
		
		//4. 提供对应的缓冲流
		BufferedInputStream bis = new BufferedInputStream(fis);
		BufferedOutputStream bos = new BufferedOutputStream(fos);
		
		//5. 读取数据拷贝
		int length = -1;
		byte[] buffer = new byte[1024 * 8];
		
		while ((length = bis.read(buffer)) != -1) {
			bos.write(buffer, 0, length);
		}
		
		//6. 关闭资源
		bos.close();
		bis.close();
		
	}
	
	public static void WriteTest() throws IOException {
		//1. 确定要操作的文件
		File file = new File("C:/aaa/5.txt");
		
		//2. 建立管道
		//创建FileOutputStream对象，提供读写能力
		FileOutputStream fos = new FileOutputStream(file);
		
		//创建BufferedOutputStream对象，用FileOutputStream作为参数
		BufferedOutputStream bs = new BufferedOutputStream(fos);
		
		//3. 写入数据
		String str = "今天JD iPad Pro又便宜了一百~~~";
		
		bs.write(str.getBytes());
		
		//4. 关闭资源
		bs.close();
		//fos FileOutputStream不用单独关闭，缓冲流对象的close会关闭输出字节流
	}
}

```

### （三）输入字符缓冲

```java
package com.qfedu.a_buffer;

import java.io.BufferedReader;
import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;

/*
 	BufferedReader 缓冲区是一个char类型数组，数组元素个数为8192， 占用空间大小是16KB
 */

public class ReaderBuffered {
	public static void main(String[] args) throws IOException {
		//1. 找到文件
		File file = new File("C:\\Users\\刘晓磊\\Desktop\\稻香.lrc");
		
		//判断文件是否存在，是否是普通文件
		if (!file.exists() || !file.isFile()) {
			throw new FileNotFoundException();
		}
		
		//2. 建立管道 
		//所有的缓冲流都是没有读取能力的，需要提供对应的字符流来提供读写能力
		FileReader fr = new FileReader(file);
		BufferedReader br = new BufferedReader(fr);
		
		//3. 读取文件内容
		String str = null;
		
		while ((str = br.readLine()) != null) { //按行读取
			System.out.println(str);
		}
		
		//4. 关闭资源
		br.close();
		
	}
}

```

### （四）输出字符缓冲
```java
package com.qfedu.a_buffer;

import java.io.BufferedWriter;
import java.io.File;
import java.io.FileWriter;
import java.io.IOException;

public class WriterBuffered {
	public static void main(String[] args) throws IOException {
		//1. 确定目标文件
		File file = new File("C:/aaa/6.txt");
		
		//2. 建立管道
		FileWriter fw = new FileWriter(file);
		BufferedWriter bw = new BufferedWriter(fw);
		
		//3. 写入数据
		bw.write("感谢你给我的光荣");
		bw.write("&&&&&&&&&&&");
		bw.newLine(); //换行！！！
		bw.write("*****************");
		
		//4. 关闭资源
		bw.close();
	}
}

```

## 三、JavaAPI

### （一）String 类型
==和 equals 方法
```java
package com.qfedu.b_javaAPI;

public class Demo1 {
	public static void main(String[] args) {
		String str1 = "David";
		String str5 = "David";
		String str2 = new String(str1);
		String str3 = new String(str1);
		String str4 = new String(str2);
		
		System.out.println("str1 == str5 :" + (str1 == str5)); //true
		System.out.println("str1 == str2 :" + (str1 == str2)); //false
		System.out.println("str2 == str3 :" + (str2 == str3)); //false
		System.out.println("str2 == str4 :" + (str2 == str4)); //false
		System.out.println("str3 == str4 :" + (str3 == str4)); //false
		
		System.out.println("str1.equals(str2):" + str1.equals(str2)); //true
		System.out.println("str1.equals(str3):" + str1.equals(str3)); //true
		System.out.println("str1.equals(str4):" + str1.equals(str4)); //true
		System.out.println("str1.equals(str5):" + str1.equals(str5)); //true
	}
}
```

### （二）String 的方法
```java
package com.qfedu.b_javaAPI;

public class StringMethods {
	public static void main(String[] args) {
		
		// length()  求字符串元素个数，不是占用的空间
		System.out.println("123".length());  //3
		System.out.println("雷猴~".length()); //3
		
		// charAt(int index); 获取字符串中的一个字符
		System.out.println("123456789".charAt(5)); //6
		
		//indexOf(int ch); (String str); (char c)
		System.out.println("2345678987654345".indexOf('2')); // 0
		System.out.println("2345678987654345".indexOf("789")); //5
		
		//lastIndexOf()
		System.out.println("23456789876542345".lastIndexOf(2)); //-1 
		//这里找的是ASCII码为2的字符，因为前32个ASCII码值不可见
		System.out.println("23456789876542345".lastIndexOf(2)); //13
		System.out.println("23456789876542345".lastIndexOf("345")); //14
		
		//endWith(String str)
		System.out.println("1234567.txt".endsWith("tx")); //false
		
		//contains()
		System.out.println("321321321".contains("21")); //true
		
		//isEmpty()
		System.out.println("1".isEmpty()); //false
		System.out.println("".isEmpty()); //true
		
		//equalsIgnoreCase()
		System.out.println("abc".equals("ABC")); //false
		System.out.println("abc".equalsIgnoreCase("ABC")); //true
		
		//static String valueOf(char[] data);
		char[] arr = {'a','b','c','d','e','f','g'};
		System.out.println(String.valueOf(arr)); //abcdefg
		
		//toCharArray()
		char[] arr2 = "1234567890".toCharArray();
		
		for (char c : arr2) {
			System.out.println(c); 
		}
		//1
		//2
		//......
		
		//replace(char oldChar, char newChar);
		String str = "123456282";
		
		str = str.replace('2', '5');
		System.out.println(str);//"153456285" 因为有个遍历，因此是全部调换
		
		
		String lrc = "[00:00.00]侯高俊杰 - 稻香\r\n" + 
				"[00:07.74]\r\n" + 
				"[00:10.58]作词：周杰伦  作曲：周杰伦\r\n" + 
				"[00:16.05]\r\n" + 
				"[00:31.11]对这个世界如果你有太多的抱怨\r\n" + 
				"[00:34.65]跌倒了  就不敢继续往前走\r\n" + 
				"[00:37.48]为什么  人要这么的脆弱 堕落\r\n" + 
				"[00:41.61]请你打开电视看看\r\n" + 
				"[00:43.44]多少人为生命在努力勇敢的走下去\r\n" + 
				"[00:47.37]我们是不是该知足\r\n" + 
				"[00:49.88]珍惜一切 就算没有拥有";
		//切割字符串
		String[] array = lrc.split("\r\n");
		
		for (String string : array) {
			System.out.println(string);
		}
		
		//trim() 用户处理前端发送过来数据的多余空格
		String username = "    lxl";
		System.out.println(username); //     lxl
		username = username.trim();		
		System.out.println(username); //lxl
	}
}
```

### （三）StringBuffer·
```java
package com.qfedu.b_javaAPI;

public class TestStringBuffer {
	public static void main(String[] args) {
		//调用无参构造方法，创建的一个默认字符个数为16的StringBuffer对象
		StringBuffer stringBuffer = new StringBuffer();
		
		stringBuffer.append("我的家在东北~~~");
		stringBuffer.append('松'); // \40
		
		stringBuffer.insert(10, "花江上啊~~~");
		
		String str = stringBuffer.substring(0, 6);
		
		stringBuffer.delete(0, 6);
		stringBuffer.deleteCharAt(0);
		
		stringBuffer.reverse();
		
		System.out.println(stringBuffer.toString());
		System.out.println(str);
		
		//StringBuilder 是线程不安全的， JDK1.5之后的新特征，但是效率
		//StringBuffer 是线程安全的，效率低
	}
}
```
### （四）System 里面的方法

```java
package com.qfedu.b_javaAPI;

import java.util.Properties;

public class TestSystem {
	public static void main(String[] args) {
		//属性  获取系统属性
		Properties ps = System.getProperties(); 
		
		//属性的展示方式
		ps.list(System.out);
		
		//使用属性是获取属性里面的内容
		String username = System.getProperty("user.name");
		System.out.println(username);
	}
}
```

### （五）Runtime
```java
package com.qfedu.b_javaAPI;

import java.io.IOException;

public class TsetRuntime {
	public static void main(String[] args) throws IOException, InterruptedException {
		//获取软件的运行环境
		Runtime run = Runtime.getRuntime();
		
		System.out.println("当前空余内存:" + run.freeMemory());
		System.out.println("JVM只能的总内存:" + run.totalMemory());
		System.out.println("JVM能够使用最大内存:" + run.maxMemory());
		
		Process notepad = run.exec("notepad"); //打开应用程序
		//Process myEclipse = run.exec("C:/MyEclipse Professional 2014/myeclipse.exe");
		
		Thread.sleep(10000);
		
		notepad.destroy();	
	}
}
```

### （六）Date
```java
package com.qfedu.b_javaAPI;

import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;

public class TestDate {
	public static void main(String[] args) {
		//Date 封装了系统的当中的时间类
		Date date = new Date();
		
		//Calender 日历类 可以直接获取年月日时分秒
		Calendar c = Calendar.getInstance();
		
		int year = c.get(Calendar.YEAR);
		int month = c.get(Calendar.MONTH);
		int day = c.get(Calendar.DAY_OF_MONTH);
		int dayOfWeek = c.get(Calendar.DAY_OF_WEEK);
		
		int hour = c.get(Calendar.HOUR_OF_DAY);
		int minute = c.get(Calendar.MINUTE);
		int second = c.get(Calendar.SECOND);
		
		System.out.println(year + ":" + (month + 1) + ":" + day + " " + dayOfWeek 
				+ " " + hour + ":" + minute + ":" + second );
		//格式化显示
		SimpleDateFormat sf = new SimpleDateFormat("yyyy年MM月dd日 E  HH:mm:ss");
		System.out.println(sf.format(date));
		
		
	}
}
```

### （七）Math
```java
package com.qfedu.b_javaAPI;

public class TestMath {
	public static void main(String[] args) {
		System.out.println(Math.PI);
		System.out.println(Math.ceil(3.14)); //向上取整 4
		System.out.println(Math.ceil(-3.14)); //向上取整 -3
		
		System.out.println(Math.floor(3.14)); //向下取整 3
		System.out.println(Math.floor(-3.14)); //向下取整 -4
		
		System.out.println(Math.round(15.5));
		System.out.println(Math.round(15.1));
		
		//0 ~ 1 之间的随机数
		System.out.println(Math.random());
	}
}
```





