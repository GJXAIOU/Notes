# JavaEEDay19-文件操作
## 一、复习：泛型

- 为了解决数据类型一致化问题,避免没有意义的强制类型转换(放进什么数据，拿出来什么数据)，控制输入数据的格式；

- 定义泛型泛型使用的格式：
<大写字母> 一般使用E 或者 T ，仅仅是占位符；

###  （一）泛型函数中使用
格式：
```java
权限修饰符 <自定义泛型> 返回值类型(可以使用泛型) 函数名(形式参数列表“自定义泛型”) {
  同样可以使用自定义泛型
}
```

### （二）泛型在类中使用
格式：
```java
class 类名<自定义泛型> {
  非静态的成员变量或者成员方法都可以使用类中定义的<自定义泛型>；  
  静态方法不能使用类中自定义泛型，但是可以方法中自己定义泛型；  
}
```

例如：`Arrays.sort(T[] t, Comparator<? super T> c)`

###  （三）泛型在接口中使用
格式：
```java
interface 接口名<自定义泛型> {
  //成员变量 缺省属性：public static final
  //成员方法 缺省属性：abstract
}
```

- 一个类遵从带有自定义泛型的接口有两种方式：
  例如接口如下：
      interface A<T> {
          public void testA(T t);
      }
  
1. 方式1：更加自由，在创建类对象时，才对泛型进行约束
```java
class Test1<T> implements A<T> {
  public void testA(T t) {
      //实现方法
  }
}
```

2. 方式2 :遵从接口时，接口直接确定了泛型的具体类型 
```java
class Test2 implements A<String> {
  public void testA(String t) {
      //实现方法
  }
}
```

- 泛型的上下限：
`<? super T>`  表示数据类型是T对象或者其父类对象
`<? extends T>`表示数据类型是T对象或者其子类对象
  
## Map
  `Map<K, V> ` 键值对
  - 两个实现类：
    -  HashMap
    -  TreeMap
  
  - 常见的方法：
```java
put(K key , V value);
putAll(Map<? extends k, ? extends V> map)

clear();
remove(Object k);

size();
containsKey(Object key);
containsValue(Object Value);

keySet();
values();

get(Object k);
```
 

## 简单介绍了一个匿名内部类
```java
 new Comparator<Student>() {
      @Override   
      public int compare(Student o1, Student o2) {
          return o1.getAge() - o2.getAge();
      }
  }
```


## Map 补充

```java
package study;
import java.util.Collection;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map.Entry;
import java.util.Set;

public class Demo1 {
	public static void main(String[] args) {
		HashMap<String, Integer> map = new HashMap<String, Integer>();
		
		map.put("MacBook Pro", 22888);
		map.put("iPhoneX", 8398);
		map.put("iPad Pro", 5198);
		
		System.out.println(map);
		
		//第一种遍历方式：借助于keySet，通过key值拿到value值
		Set<String> set = map.keySet();
		
		//使用Set集合的Iterator迭代器
		Iterator<String> it = set.iterator();
		while (it.hasNext()) {
			String key = it.next(); //获取每一个Map中的key值
			int value = map.get(key);
			
			System.out.println(key + "=" + value);
		}
		//以上方法，不太合适，实际上获取的key值，再借助于Map里面的get方法，获取对应的Value
		//实际上并没有获取到完整的键值对
		
		
		
		//第二种方式：借助于values
		Collection<Integer> c = map.values();
		
		for (Integer integer : c) {
			System.out.println(integer);
		}
		
		//以上方法也不适合，因为只能拿到value值，获取不到对应的key
		
		
		
		//在Java中万物皆对象，因此可以将键值对看成是一个类【必须会】
		/*
	 	将键值对，认为是一个对象，组成一个类，称之为Entry
		 class Entry<K, V> {
		 	K key;
		 	V value;
		 } 
		 这里可以认为在Map集合中，保存的每个键值对都是一个Entry对象
		 将这些Entry对象获取并做成一个集合，然后对该集合进行遍历；
		 entrySet();
		 Map.Entry；Entry为Map的子接口，称为内部类
		 */
		Set<Entry<String, Integer>> entrySet = map.entrySet();
		
		Iterator<Entry<String, Integer>> it2 = entrySet.iterator();
		
		while (it2.hasNext()) {
			System.out.println(it2.next());
		}
	}
}

```


# 文件操作

 IO:
Input Output
read  write

只考虑文件操作情况下：
 - 读取操作是：input read 从硬盘(存储设备) 到 内存
 - 写入操作是：output write 从内存 到 硬盘(存储设备)

如何操作文件或者文件夹：
创建，删除，剪切，复制，重命名，粘贴

Java里万物皆对象,计算机世界里万物皆文件

- Sun提供了一个File类，专门来操作【文件】和【文件夹】

```java
 如何创建一个File类对象：
File(String pathName);
根据文件路径创建File类对象，可以是【绝对路径】可以是【相对路径】

File(String parent, String child);
根据提供的父目录【文件夹】地址，和当前文件夹下的【子文件】或者【子文件夹】创建File类对象

File(File parent, String child);
根据创建好的父目录的File类对象，和这个目录下的【子文件】或者【子文件夹】创建File类对象
```
【pathName】
\n \t \\ 转义字符
假如实际地址为：C:\aaa\bbb 如果在代码中使用 "C:\\aaa\\bbb" LOW

**推荐使用 `/ `来区分路径,不使用反斜杠**
Java是一个跨平台的语言
Java提供了一个系统变量:File.separator 可以根据不同的系统换，自动填充文件分隔符
示例：
```java
public class Demo1 {
	public static void main(String[] args) {
		File file1 = new File("C:\\aaa"); //采用Windows的分隔符
		File file2 = new File("C:/aaa"); //用在Linux和Windows通用的分隔符
		File file3 = new File("C:" + File.separator + "aaa"); //这里使用Java系统变量
		
		System.out.println(file1);
		System.out.println(file2);
		System.out.println(file3);
		
		File file4 = new File("C:\\aaa", "1.txt"); //父目录 子目录 
		System.out.println(file4); //返回的是路径
		
		File file5 = new File(file1, "1.txt");
		System.out.println(file5);
	}
}
```

File 操作常见方法：
```java
isFile(); //是否为文件
isDirectory(); //是否为目录
createFile()
renameTo
copy
```


## 可以利用File类对象，创建文件或者文件夹
	
- boolean createNewFile(); 创建文件
	使用File类对象，创建File类对象里面保存的地址 指定 的 普通文件
	返回值boolean: 创建成功返回true，创建失败返回false
	返回false失败原因:
		1. 该文件已经存在
		2. 操作路径非法，例如：文件指定所在文件夹不存在
		3. 操作的文件夹没有写入权限
		4. 硬盘坏了
【要求】
	创建文件，必须带有文件后缀名！！！
	.java .class .doc .txt .xml .html .css .js .md .jsp 
	.m .h .c .cpp .php .net .ppt .xls .exe .zip .rar .mp4
	.rmvb
	
	
- boolean mkdir(); make direcotry 创建文件夹
	使用File类对象里面保存的文件夹地址，创建对应的文件夹
	- 返回值：boolean 创建成功返回true 创建失败返回false
	- 失败原因：
		1. 已经存在该文件夹
		2. 指定创建文件夹的父目录没有写入权限
		3. 要创建文件夹的父目录不存在
- boolean mkdirs();
	使用File类对象里面保存的文件夹路径地址，创建指定文件夹，如果该路径中的【中间文件夹】不存在,把中间路径，同时创建
	- 返回值：boolean 创建成功返回true 创建失败返回false
	- 失败原因:
		1. 已经存在该文件夹
		2. 指定创建文件夹没有写入权限
	C:/aaa/ccc/ddd/eee/fff/ggg/hhh/iii/jjj
	
- boolean renameTo(File dest);
	- 功能1：
		重命名！！！文件 或者 文件夹
	- 功能2：
		剪切，移动到另一个位置
	
	作业：
	测试，renameTo操作一个非空文件夹！！！

程序测试：
```java
import java.io.File;
import java.io.IOException;

public class Demo2 {
	public static void main(String[] args) throws IOException {
		
		//创建文件
		File file1 = new File("C:/aaa/1.txt");
		
		boolean ret = file1.createNewFile();
		System.out.println("文件创建成功了吗?" + ret);
		
//		File file2 = new File("C:/bb/1.txt");
//		System.out.println(file2.createNewFile());
		
		File file2 = new File("C:/aaa/bbb");
		ret = file2.mkdir();
		System.out.println("文件夹创建成功了吗?" + ret);
		
		File file3 = new File("C:/aaa/ccc/ddd");
		ret = file3.mkdir();
		System.out.println("文件夹创建成功了吗?" + ret);
		
		File file4 = new File("C:/aaa/ccc/ddd/eee/fff/ggg/hhh/iii/jjj");
		ret = file4.mkdirs();
		System.out.println("文件夹创建成功了吗?" + ret);
		
		
		File dest1 = new File("C:/aaa/2.txt");
		//把C:/aaa/1.txt 重命名！！！
		ret = file1.renameTo(dest1); //里面的参数为命名文件的新抽象路径名称，因为要求File类型，不能使用字符串路径；
		System.out.println("重命名成功了吗?" + ret);
		
		ret = file2.renameTo(new File("C:/aaa/ddd")); //需要将路径创建为一个新的File类对象
		System.out.println("重命名文件夹成功了吗?" + ret);
		
		//测试剪切功能
		//原地址要带有文件或者文件夹名，而且目标地址也有带有文件或者文件夹名
		File txt3 = new File("C:/aaa/3.txt");
		File dest2 = new File("C:/Users/刘晓磊/Desktop/3.txt");
		
		ret = txt3.renameTo(dest2);
		System.out.println("剪切成功了吗?" + ret);
		
	}
}
```

- delete()
 		删除文件或者文件夹，但是如果操作文件夹的话，**只能删除空文件**夹,成功返回true ，失败返回false; 		
 		该删除操作不是把文件或者文件夹放入到回收站里，而是**直接从磁盘上抹去数据,该操作不可逆**。
 	
 - deleteOnExit()
 		当JVM虚拟机运行终止之后，删除指定的文件或者文件夹，而不是调用立即删除；	
 	- 用途：
 			用于删除程序运行结束之后残留的缓存文件或者运行日志文件，节约硬盘空间；

代码示例：
```java
import java.io.File;
import java.util.Scanner;
public class Demo3 {
	public static void main(String[] args) {
		File file1 = new File("C:/aaa/1.txt");
		
		System.out.println("删除成功了吗?" + file1.delete());
		
		File file2 = new File("C:/aaa/ddd");
		System.out.println("删除成功了吗?" + file2.delete());
		
		File file3 = new File("C:/aaa/ccc");
		System.out.println("删除成功了吗?" + file3.delete()); //返回false
		
		// deleteOnExit()使用方法
		File file4 = new File("C:/aaa/2.txt");
		Scanner sc = new Scanner(System.in);
		
		file4.deleteOnExit();
		sc.nextLine();
	}
}
```

 * exists(); 判断指定的文件或者文件夹是否存在
 * isFile(); 判断指定的File是文件吗？
 * isDirectory(); 判断指定的File是文件夹吗？
 * isHidden(); 判断指定的File是隐藏文件吗？
 * isAbsolute();  判断创建File类对象使用的是绝对路径吗？
 返回值全是boolean 

代码示例：
```java
import java.io.File;
public class Demo4 {
	public static void main(String[] args) {
		File file = new File("C:/aaa/1.txt");
		
		System.out.println("这个文件或者文件夹存在吗?" + file.exists());
		System.out.println("这个File类对象是一个文件类对象吗?" + file.isFile());
		//这里使用了匿名对象，我一般称之为 一点到底！！！
		System.out.println("这个File类对象是一个文件夹类对象吗?" + new File("C:/aaa").isDirectory());
		System.out.println("这个File类对象是一个隐藏文件对象吗?" + file.isHidden());
		System.out.println("创建File类对象使用了绝对路径吗?" + file.isAbsolute());
		System.out.println("创建File类对象使用了绝对路径吗?" + new File(".").isAbsolute()); //. 表示当前目录		
	
	}
}
```
  	
 **以下方法和文件是否【存在无关】**！！！！！！
  	
- getName(); 获取路径中的文件名或者文件夹名
- getPath(); 获取File类对象里面保存的路径
- getAbsolutePath(); 获取File对象里面保存路径对应的绝对路径
- getParent(); 获取当前文件或者文件夹的父目录，如果没有返回null
  	
- lashModified(); 文件最后的修改时间
	UNIX时间戳
	计算机元年到修改时间的秒数：
	1970年01月01日 00:00:00
	2017年12月08日 15:24:50

- length(); 文件的大小(字节数)，如果文件不存在，或者是一个文件夹，返回0L

代码示例：
```java
import java.io.File;
import java.sql.Date;
import java.text.SimpleDateFormat;
public class Demo5 {
	public static void main(String[] args) {
		File file = new File("E:/aaa/1.txt");
	
		System.out.println(file.getName()); //1.txt
		System.out.println(file.getPath()); //E:/aaa/1.txt
		System.out.println(file.getAbsolutePath()); //E:/aaa/1.txt
		System.out.println(file.getParent()); //E:/aaa
		
		long last = new File("C:/aaa/1.txt").lastModified();
		
		Date date = new Date(last);
		SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
		System.out.println(format.format(date));
		
		//System.out.println(last); //直接使用这个会得到秒数
	}
}
```
其他方法：返回值  方法  含义
- static File[]       listRoots(); 
 	获取当前计算机中所有的盘符，针对Windows操作系统
 	Linux/UNIX没有区分盘符的概念 只有一个根目录 / 
 	
- String[]        list();
 	获取指定【文件夹】里面所有子文件和子文件，返回一个String类型的的数组
 	
- File[]    listFiles();
 	获取指定【文件夹】里面所有子文件和子文件，返回一个File类型的的数组
 	
- 遍历一个文件夹：
 文件夹:
       文件夹1;
       文件夹2;
       文件夹3;
………………
文件:
		文件1;
		文件2;
		文件3;
		………………

代码示例：
```java
import java.io.File;
public class Demo6 {
	public static void main(String[] args) {
		File[] roots = File.listRoots();
		for (File file : roots) {
			System.out.println(file.getPath());
		}
		System.out.println("---------------------------------");
		
		File file = new File("C:\\Program Files\\Java\\jdk1.8.0_131");
		String[] allFilePath = file.list();
		
		for (String string : allFilePath) {
			System.out.println(string);
		}
		
		System.out.println("---------------------------------");
		
		File[] allFileObject = file.listFiles();
		for (File file2 : allFileObject) {
			System.out.println(file2);
		}
		
	}
}
```

-  遍历一个文件夹：
 		文件夹:
 			文件夹1;
 			文件夹2;
 			文件夹3;
 			………………
 		文件:
			文件1;
			文件2;
			文件3;
			………………
1. 首先利用listFiles()方法，获取指定文件夹下的所有子文件和子文件夹的File类对象数组
2. 利用isFile或者isDirectory判断是否是文件或者是文件夹
3. 区别对象，利用不同的函数 

找出指定文件夹下的所有.java文件?   endWith() 和 isFile()

```java
public class Demo7 {
	public static void main(String[] args) throws FileNotFoundException {
		
		findAllJavaFile("C:\\Users\\刘晓磊\\Desktop\\test");
		
		File srcDir = new File("C:\\Program Files\\Java\\jdk1.8.0_131");
		
		if (srcDir.exists() && srcDir.isDirectory()) {
			
			File[] allFiles = srcDir.listFiles();
			
			showAllFiles(allFiles);
			showAllDirectories(allFiles);
		} 
	}
	
	private static void showAllFiles(File[] allFiles) {
		System.out.println("普通文件:");
		for (File file : allFiles) {
			if (file.isFile()) {
				System.out.println("\t" + file.getName());
			}
		}
	}
	
	private static void showAllDirectories(File[] allFiles) {
		System.out.println("文件夹:");
		for (File file : allFiles) {
			if (file.isDirectory()) {
				System.out.println("\t" + file.getName());
			}
		}
	}
	
	public static void findAllJavaFile(String pathname) throws FileNotFoundException {
		//如果给定的路径是一个null，无法使用
		if (null == pathname) {
			throw new NullPointerException();
		}
		
		File srcDir = new File(pathname);
		
		//判断文件是否存在，是否是一个文件夹
		if (!srcDir.exists()) { //srcDir.exists() == false
			throw new FileNotFoundException();
		} else if (!srcDir.isDirectory()) { //srcDir.isDirectory() == false
			//运行时异常，不用再函数中声明
			throw new RuntimeException("该文件不是一个文件夹");
		}
			
		File[] allFiles = srcDir.listFiles();
		
		for (File file : allFiles) {
			//文件名后缀为.java 并且 是一个普通文件
			if (file.getName().endsWith(".java") && file.isFile()) {
				System.out.println(file.getName());
			}
		}
	
	} 
}


```

- 归档文件

如果是普通文件，按照文件的后缀名创建对应的全大写文件夹，把文件剪切进去
    特殊情况，如果是没有文件后缀名的未知普通文件，就放入到 others 文件夹；
如果是文件夹，创建 subDir 文件夹，剪切进去；


  - 思路：
    - 1.获取指定的文件夹路径，进行判断；
    - 2.获取这个文件夹下面的所有子文件和子文件夹的 File[]类对象；
    - 3.遍历这个 File[] 类型的数组，按照文件和文件夹进行区别操作；
         - 3.1.如果是文件：
           - 首先获取这个文件的文件后缀；
           - 然后创建以这个文件后缀名全大写的文件夹；
           - 最后移动该文件到这个文件夹；
           - 注：如果一个文件没有后缀名，就放入到 Others 目录下；
         - 3.2.如果是文件夹：
           - 首先创建 subDir 目录；
           - 移动该文件夹放入到 subDir 里面；

程序代码：
```java
package study;

import java.io.File;
import java.io.FileNotFoundException;
import java.io.IOException;

public class Demo1 {
	public static void main(String[] args)throws NullPointerException, IOException {
		achieveDirectoryAtPath("C:/test");
	}
	
	public static void achieveDirectoryAtPath(String path) throws NullPointerException, IOException{
		//1.判断路径
		if (null == path) {
			throw new NullPointerException("地址不能为null");
		}
		
		//2.创建指定地址对应的File类对象
		File file = new File(path);
		
		//3.判断这个是不是一个文件夹，并且是否存在
		if ((!file.exists() || !file.isDirectory())) {
			//等价于：file.exists() == false || file.isDirectory() == false;
			throw new FileNotFoundException("没有指定文件");
		}
		
		//在归档之前判断是否进行过归档操作，如果有则不再进行归档
		if (new File(file, "file.lock").exists()) {
			System.out.println("归档已完成");
			return ;
		}
		
		//4.获取指定文件夹下面的所有子文件和子文件夹的File类对象数组
		File[] allFiles = file.listFiles();
		
		//5.遍历该File数组，判断是文件还是文件夹，区别对待
		for (File fileItem : allFiles) {
			if (fileItem.isFile()) {
				//文件操作file是当前要归档的文件夹，filename是要处理的文件名字
				achieveFile(file,fileItem.getName());
			}else {
				//文件夹操作
				achieveDri(file, fileItem.getName());
			}
		}//for	
		
		//6.归档完成，上锁，防止再次归档
		File lockFile = new File(file, "file.lock");
		lockFile.createNewFile();
	}
	

	
	private static void achieveFile(File file, String fileName) {
		//1.找到对应的文件后缀名
		int index = -1;
		String dirToCreate = null;
		File srcFile = null; //原地址 
		File dstFile = null;
		
		//2.获取要创建的文件夹路径的String类型字符串
		//判断文件是否存在文件后缀名，如果没有'.'则是不存在，有点但是点后面没有内容也是没有后缀名
		if ((index = fileName.lastIndexOf('.')) != -1 && fileName.substring(index + 1).length() !=0) { //不能包括index,因此加1
			/*上面的If语句等价于下面
			 * index = fileName.lastIndexOf('.');
			if (index != -1) {				
			}
			*/
			
			 /*
			String parent = file.getAbsolutePath();
			String dirName = fileName.substring(index + 1).toUpperCase();
			dirToCreate = parent + File.separator + dirName;
			*/
			//上面代码的简化版
			/*
			 * File.separator作用：这是File类提供的一个更加运行环境而决定的文件路径分隔符，在Windows中是\,
			 * 在Linux和Unix中是/，这样写的好处是可以让代码的通用性和可移植性更好
			 */
			dirToCreate =file.getAbsolutePath() + file.separator + fileName.substring(index + 1).toUpperCase();			
		}else {
			dirToCreate = file.getAbsolutePath() + file.separator + "other";
		}
		
		new File(dirToCreate).mkdir();  //创建File匿名对象，调用mkdir（）
		
		srcFile = new File(file, fileName);
		dstFile = new File(dirToCreate,fileName);
		
		srcFile.renameTo(dstFile);
	}
	 
	private static void achieveDri(File file, String dirName) {
		//创建subDir
		File subDir = new File(file, "subDir");
		subDir.mkdir();
		
		File srcFile = new File(file, dirName);
		File dstFile = new File(subDir, dirName);
		
		srcFile.renameTo(dstFile);
	}
}
```

## 单例
目的：使整个程序的运行过程中，有且只有一个当前类的类对象存在；

方案一：在整个程序的运行过程中，有且只调用一个构造方法（相当于仅仅实例化一次）
缺点：第三方调用者不知道这个规定，可以通过简单的 new 关键字进行调用；
同时如果使用 private 关键字修饰构造函数，则构造方法在类外无法使用且类外没有对象，则所有和对象相关的方法均不能使用，需要不通过对象就可以调用方法：
使用 static 关键字获取或者创建类对象
- 该方法提供给类外使用，因此权限修饰符为【public】
- 调用时候不能通过类对象进行调用，需要类名进行调用，使用【static】关键字
- 该方法是获取当前类的类对象，所以返回值应该是当前类对象类型
- 方法名称：【getInstance()】
- 参数要和类内私有化的构造方法所需要的参数一致：【int num】

但是仍然需要判断一下当前程序中有没有已经创建的类对象，如果没有就调用私有化的构造方法进行创建，否则不再创建；
这里需要一个当前类的数据类型来保存这个变量，该变量保存已经创建对象的地址。
- 因为函数局部变量的生存期只有在这个方法内部，第二次再次调用的时候，之前的局部变量已经销毁；
- 同时因为这是静态方法，不能使用非静态的成员变量，只能使用静态成员变量；

```java
package study;

class Demo{
	int num;
	private static Demo demo = null; //不能在类外更改，防止再次初始化
	
	private Demo(int num) {
		this.num = num;
	}
	
	public void test() {
		System.out.println(this.getClass() + "的test方法");
	}
	
	public static Demo getInstance(int  num) {
		//判断是否已经创建类对象		
		if (demo == null) {
			demo = new Demo(num);
		}
		//私有化的构造方法可以在类内使用
		return demo;
	}
}


public class SingleDemo {
	public static void main(String[] args) {
		//Demo1 demo1 = new Demo1(3);
		
		Demo demo1 = Demo.getInstance(3);
		Demo demo2 = Demo.getInstance(4);
		System.out.println(demo1);
		System.out.println(demo2);
	}
}
```
