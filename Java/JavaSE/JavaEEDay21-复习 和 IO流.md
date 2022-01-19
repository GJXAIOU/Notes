# JavaDay21 IO 流

## 一、 复习
- 单例：
在整个程序运行的过程中有且只能有一个当前类对象
  - 实现步骤：
1.私有化构造方法
2.提供给类外获取类对象的方法，一个用static修饰的静态方法
    pubilc static 类对象类型 方法名(形式参数和类内的构造方法一致)
3.在类内定义一个static修饰的静态成员变量，用于保存之前创建过的类对象地址
4.在提供的静态方法中，进行判断，如果之前并没有这个对象，创建新对象方法， 并且保存地址；如果有，直接返回之前保存的对象地址
5.私有化保存对象地址的static成员变量
```java
public class Single {
    int num;
    private static Single s = null;
    
    private Single(int num) {
        this.num = num;
    }
    
    public static Single getInstance(int num) {
        if (null == s) {
            s = new Single(num);
        }           
        return s;
    }
}
```

## 二、 IO流

- IO（Input  Output）的参照物：当前运行程序：
  从硬盘中读取数据到内存中供程序使用： input
  从程序的内存中将数据保存到硬盘中:    output 
  
- 按照处理的数据单位来做划分：
  - **字节流**：
    完完全全按照二进制编码格式，一个字节一个字节获取
    
  - **字符流**：
    其实也是字节操作，但是会考虑当前系统的编码问题
    会将读取的字节数据根据当前使用的字符集进行翻译
  
 - 常见的字符集：
    GBK GB2312 BIG5 UTF8
 -  **字符流 = 字节流 + 解码**；

## 三、字节流

###  （一）代码中的体现：读取文件
- 字节流：
  输入字节流  input 从硬盘到内存的流向，读取数据操作
  InputStream：所有字节流的基类/父类   是一个抽象类
  FileInputStream：读取文件的输入字节流

- 字节流类：
  InputStream 与 FileInputStream
  OutputStream 与 FileOutputStream

- 操作步骤：
  - 1.找到文件

      是否是一个普通文件
      是否存在
      String字符串路径

  - 2.创建FileInputStream 输入管道      resource 资源

  - 3.读取文件

  - 4.关闭管道(资源)

读取示例 一：（通过循环得到所有字符）
```java
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
public class Demo1 {
	public static void main(String[] args) throws IOException {	
		readTest2();
	}

//借助于读取操作的特征，每一次read之后，文件指针都会向后移动，直到读取到文件末尾，返回-1结束
	//这里可以利用while循环来做循环读取操作
	public static void readTest2() throws IOException {
		long start = System.currentTimeMillis(); //获取当系统时间，毫秒
		//1. 找到文件
		File file = new File("C:/《红楼梦》.txt");
		
		//判断文件是否存在，是否是个普通文件
		if (!file.exists() || !file.isFile()) {
			throw new FileNotFoundException();
		}
		
		//2. 创建InputStream字节流输入管道
		FileInputStream fs = new FileInputStream(file);
		
		//3. 读取数据
		int content = -1;
		// fs.read()每次只能读取一个字节，不能读取所有的文件内容
		// int read() 返回值就是读取到的字节数据，如果文件读取到最后，没有内容了，返回-1
		while ((content = fs.read()) != -1) {
			System.out.println((char)content);
		}
		
		//4. 关闭资源
		fs.close(); //句柄
		long end = System.currentTimeMillis();
		System.out.println("耗时:" + (end - start));
	}
}
```
读取示例二：使用字节缓冲读取
```java
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
public class Demo1 {
	public static void main(String[] args) throws IOException {	
		readTest4();
	}

	public static void readTest4() throws IOException {
		long start = System.currentTimeMillis();
		//1. 找到文件
		File file = new File("C:/《红楼梦》.txt");
		
		//判断是否存在，是否是一个普通文件
		if (!file.exists() || !file.isFile()) {
		   throw new FileNotFoundException();
		}
		
		//2. 创建字节流输入管道
		FileInputStream fs = new FileInputStream(file);
		
		//3. 准备缓冲区，读取文件
		int length = -1;
		byte[] buffer = new byte[1024]; //字节缓冲区
		
		while ((length = fs.read(buffer)) != -1) {
			System.out.println(new String(buffer, 0, length));
		}
		
		//4. 关闭管道(资源)
		fs.close();
		
		long end = System.currentTimeMillis();
		System.out.println("耗时:" + (end - start));
 	}
 }
```

以上代码，以 10MB 的文件读取结果：
readTest2()：53363 毫秒
readTest4()：10448 毫秒

### （二）写入文件操作

Output 所有输出字节流的基类，父类，抽象类
FileOutputStream 文件输出字节流

- 写入文件的操作流程：
  	1.找到要操作的文件 (文件可以存在，可以不存在，但是要求操作的文件必须有后缀名)
  	2.创建输出管道
  	3.写入数据
  	4.关闭资源  	
  
- FileOutputStream注意事项:	
  	1.使用FileOutputStream写入数据到文件中，该文件可以存在，可以不存在，FileOutputStream是拥有创建文件的能力
  	2.默认情况下，FileOutputStream 采用的是覆盖写入数据的方式，如果想要使用追加写,使用 `FileOutputStream(File file, boolean append)`，添加追加写的模式	
  	3.小写字母a ascii码是97：`0110 0001`,若写入的数据是353
  `0000 0000 0000 0000 0000 0001 0110 0001` 写入的结果都是小写字母a  			
  	因为FileOutputStream调用write(int b)方法，**写入的数据其实是整个int数据内部二进制的低8位**，高24位对于write方法来说，是无效数据。因为一个汉字在GBK编码下占用的是2个字节，在UTF8编码下占用的是3个字节,因此不能使用 write 写入。只能使用 String [] buffer = “XX“；方法
```java
package com.qfedu.b_output;

import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;


public class Demo1 {
	public static void main(String[] args) throws IOException {
		
		writeTest1();
		writeTest2();
	}
	
	public static void writeTest2() throws IOException {
		//1. 确定要写入数据的文件路径
		File file = new File("C:/aaa/1212.txt");
		
		//2. 建立输出管道
        // 这时候才创建文件，后面没有true则是默认覆盖写入
		FileOutputStream fos = new FileOutputStream(file, true); 
		
		//3. 写入数据
		String str = "今天双12，可以去店里买东东，便宜~~~\r\n";
		//把字符串转换成byte[] 数组
		byte[] buffer = str.getBytes();
		fos.write(buffer);
		
		//4. 关闭资源
		fos.close();
	}
	
	
	public static void writeTest1() throws IOException {
		//1. 确定要写入文件的路径
		File file = new File("C:/aaa/你好.txt");
		
		//2. 建立管道
		FileOutputStream fos = new FileOutputStream(file, true);
		
		//3. 写入数据
		//覆盖写入，如果要使用追加写，需要使用FileOutputStream(File file, boolean append)
//		fos.write('H');
//		fos.write('e');
//		fos.write('l');
//		fos.write('l');
//		fos.write('o');
//		fos.write('W');
//		fos.write('o');
//		fos.write('r');
//		fos.write('l');
//		fos.write('d');
//      fos.write(97);
//      fos.write(353);
		fos.write('我');
		
		//4. 关闭资源
		fos.close();
	}
}

```


### （三）复制文件

```java
package com.qfedu.b_output;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;

public class CopyImage {
	public static void main(String[] args) throws IOException {
		//1. 找到源文件
		File srcFile = new File("C:\\Users\\刘晓磊\\Desktop\\1.jpg");
		
		//判断源文件是否存在，是否是一个普通文件
		if (!srcFile.exists() || !srcFile.isFile()) {
			throw new FileNotFoundException();
		}
		
		//2. 确定要拷贝的地址
		File dstFile = new File("C:\\Users\\刘晓磊\\Desktop\\2.jpg");
		
		//3. 打开input和output通道
		FileInputStream fis = new FileInputStream(srcFile);
		FileOutputStream fos = new FileOutputStream(dstFile);
		
		//4. 利用缓冲从硬盘上读取数据，把读取到的数据写入到文件中
		int length = -1; //读取到的字节个数
		byte[] buffer = new byte[1024 * 4];
		
		while ((length = fis.read(buffer)) != -1) {
			//fos.write(buffer); 会有可能导致复制之后的文件会变大，原因是从硬盘读取数据时最后一次有可能
			//无法填充满整个缓冲区，但是写入时没有判断缓冲区里面的数据是不是都是图片数据
			//造成复制之后的文件大小大于原数据文件
			fos.write(buffer, 0, length); //保证读多少写多少
		}
		
		//5. 关闭资源，先开后关，后开先关
		fos.close();
		fis.close();
	}
}
```


## 四、字符流

### （一）读取数据	
 字符流读取数据，输入字符流
Reader 输入字符流的基类/超类  抽象类
FileReader 读取文件的输入字符流
  	

- 使用方式：
    - 找到文件：判断文件是否存在，是否是一个普通文件
    - 建立FileReader读取通道
    - 读取数据
    - 关闭资源

```java
package com.qfedu.c_reader;

import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;

public class Demo1 {
	public static void main(String[] args) throws IOException {
		//readerTest2(); //1709
		readerTest1(); //3342
	}
	// 使用缓冲：
	public static void readerTest2() throws IOException {
		long start = System.currentTimeMillis();
		// 1. 找到文件：
		File file = new File("C:/《The Story of the Stone》.txt");

		// 判断文件是否存在，是否是一个普通文件
		if (!file.exists() || !file.isFile()) {
			throw new FileNotFoundException();
		}

		// 2. 建立FileReder 输入字符流管道
		FileReader fr = new FileReader(file);
		
		
		//3.建立缓冲区，利用缓冲区读取数据, 创建字符数组作为缓冲区
		int length = -1; //用于保存读取到的字符个数
		char[] buffer = new char[1024];
		
		while ((length = fr.read(buffer)) != -1) {
			System.out.println(new String(buffer, 0, length));
		}
		
		//4. 关闭资源
		fr.close();
		long end = System.currentTimeMillis();
		System.out.println("耗时:" + (end - start));
	}

	// low 慢
	public static void readerTest1() throws IOException {
		long start = System.currentTimeMillis();
		// 1. 找到文件：
		File file = new File("C:/《The Story of the Stone》.txt");

		// 判断文件是否存在，是否是一个普通文件
		if (!file.exists() || !file.isFile()) {
			throw new FileNotFoundException();
		}

		// 2. 建立FileReder 输入字符流管道
		FileReader fr = new FileReader(file);

		// 3. 读取数据
		int content = -1;

		while ((content = fr.read()) != -1) { // read是每一次都读取一个字符，返回值为Int
			System.out.print((char) content);
		}

		// 4. 关闭资源
		fr.close();
		long end = System.currentTimeMillis();
		System.out.println("耗时:" + (end - start));
	}
}

```

### （二）写入数据
- 输出字符流：
  Writer 输出字符流的基类/父类   抽象类
  FileWriter 文件操作的输出字符流
  
- 操作流程：
  - 找到目标文件
  - 建立输出管道
  - 写入数据
  - 关闭资源
  
- 注意
  	
  - 如果写入的数据不多，程序如果中止，在目标文件中没有任何的数据，当程序继续运行，数据会写入到文件中；
  - 如果写入数据很大，程序如果中止，可能会在目标文件中写入一部分数据，当程序继续运行之后的数据也会写入到文件中；
  
- **Java字符流的一个特征**：
  		目的是减少对于磁盘的写入操作，写入数据到硬盘是对硬盘有一定损耗的。Java中就是用了一种缓冲机制，当调用FileWriter的writer方法，并不是直接写入数据到硬盘，而是先行保存在FileWriter类对象里面 缓冲区中，这个缓冲区是一个【字符数组】，这个数组默认的元素格式【1024个字符】
  
- **三种情况下可以直接把数据写入到文件中**：
  		
  - 关闭 FileWriter 资源
  
  - 缓冲区满了，会直接清空缓冲区，把数据写入到硬盘中
  - **调用 flush 方法，立即清空缓冲区，直接写入数据到硬盘**
  
- 注意事项：
  	FileWriter 在操作文件时，如果文件不存在，而且文件夹权限允许，可以直接创建文件如果想要追加写，调用 `FileWriter(File file,boolean append);`
```java
package com.qfedu.d_writer;

import java.io.File;
import java.io.FileWriter;
import java.io.IOException;
import java.util.Scanner;

public class Demo1 {
	
	public static void main(String[] args) throws IOException {
		writerTest();
	}
	
	public static void writerTest() throws IOException {
		//1. 找到目标文件
		File file = new File("C:/aaa/writerTest.txt");
		
		//2. 建立管道
		FileWriter fw = new FileWriter(file);
		
		//3. 准备要写入的数据
		String str = "态度决定一切，细节决定成败！！！"
				+ "态度决定一切，细节决定成败！！！"
				+ "态度决定一切，细节决定成败！！！"
				+ "态度决定一切，细节决定成败！！！";
		
		for (int i = 0; i < 100; i++) {
			fw.write(str);
		}
		
		new Scanner(System.in).nextLine();
		
		//4. 关闭资源
		fw.close();
	}
}
```

进行复制操作：
下面的代码在复制之后会缺少数据；同时判断哪些可以使用字符流，哪些可以使用字节流
```java
package com.qfedu.c_reader;

import java.io.File;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;

/*
 有问题：
  	每一次复制之后的文件都是缺少固定的字符个数
  	
  	是因为使用字符流读取数据时，数据会进行一个解码的过程，根据当前系统规定默认字符集进行解码
  	而在解码的过程中，如果解码的数据不能和当前字符集里面的数据进行匹配，那么对于字符流来说
  	这个一个无效数据，会被扔掉，这就是为什么复制图片无法成功，一直缺少数据
  	
  	字节流基本上可以用在所有的文件操作中
  	字符流只能用到txt文本可以打开的可视化文件中
 */

public class CopyImage {
	public static void main(String[] args) throws IOException {
		//1. 找到源文件
		File srcFile = new File("C:\\Users\\刘晓磊\\Desktop\\1.jpg");
		
		if (!srcFile.exists() || !srcFile.isFile()) {
			throw new FileNotFoundException();
		}
		
		//2. 确定目标文件
		File dstFile = new File("C:\\Users\\刘晓磊\\Desktop\\3.jpg");
		
		//3. 打开读取和写入通道
		FileReader fr = new FileReader(srcFile);
		FileWriter fw = new FileWriter(dstFile);
		
		//4. 准备字符缓冲区（读多少写多少）
		int length = -1;
		char[] buffer = new char[1024];
		
		while ((length = fr.read(buffer)) != -1) {
			fw.write(buffer, 0, length);
		}
		
		//5. 关闭资源
		fw.close();
		fr.close();
		
	}
}
```
