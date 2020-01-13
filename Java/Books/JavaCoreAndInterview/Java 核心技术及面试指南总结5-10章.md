---
style: summer
---
## 章五：SQL、JDBC 和数据库编程

### 一、SQL 语句注意事项

- 尽量别写 `select *`；一方面获得值当数据库变化时会改变，同时数据量过大造成性能问题；select 语句执行时间 = 数据库服务器执行该 SQL 的时间 + 结果返回时间；

- 注意 `count(*)` 和 `count(字段名)`
当正好该字段允许空值时候，两者返回记录条数的不一致；
**但是当该字段正好为主键的时候，因为主键上有索引，因此效率比 count(*) 高；**

- 在使用 insert 插入使用应该使用字段列表；

- MySQL 和 SQL Server 支持一次性插入多条数据，Oracle 不支持；但是不能无限多的插入，否则会出错；插入的多个值使用 `,` 分开: `values ('',''), ('','')`。

- delete 中可以使用 in 语句同时删除多个记录；但是同样不能过多；用法：`delect from A where id in(1, 2, 3, xxx)`；

- 使用 merge 语句实现：无则插入有则更新；但是 Oracle、SQLserver 不支持；
Demo：根据图书进货表中的 ISBN，匹配图书表，没有记录就增加，有就用图书进货表中信息更新图书表中信息（只更新以 Java 开头的书籍）
```sql
merge into图书表a
using图书进货表b
on(a.ISBN=b.ISBN)
when matched then update set a.图书名称 = b.图书名称
    where b.图书名称 like 'java%' 
when not macthed then 
    insert into a(ISBN, 图书名称 values(b.ISBN,b.图书名称）
        where b.图书名称 like 'java%' 
```

- 存储过程 
  - 这是个争议点：**存储过程只在创建时进行编译， 以后每次执行时都不需要重新编译**， 这样能提高数据库执行速度。
 但是，一般在存储过程中会有 insert、delete 或 update 的语句的集合，在创建存储过程时，这些语句确实会被编译。不过如果对比—下在存储过程里和在 JDBC(或者 Hibernate)等场合里多次执行 insert 或 delete、update 语句，那么会发现两者的性能差异并不是很明显，更何况在JDBC等场合下还可以通过 PreparedStatement 对象的批处理来优化执行性能。

- 如果针对某个业务逻辑，要对多个表进行多次 insert、delete、update 或 select 操作， 那么可以**把这些操作汇集成一个存储过程** 。 这样以后每次执行业务时， 只需要调用这个存储过程即可， 这样能提升代码的可重用性， 这也是存储过程的价值所在。

- **存储过程的移植性很差**，如针对 MySOL 数据库的存储过程不能在 Oracle 上使用，但是一般也不换数据库；

- **存储过程中很难调试，很难找到具体错误**，如在—个存储过程中有 5 个 insert 语句，分别向 5 张不同的表中插入数据，在某次通过JDBC执行该存储过程时， 向第三张表中插数据的insert语句发生—个 “主键冲突＂ 的错误，这时从 Java 语句抛出的异常来看， 只能知道 ”哪个执行存储过程出错＂ ，至于是存储过程中的哪句语句出的异常，这就只能靠自己去查了。

- **用批处理方式的性能要比用存储过程的好** ， 所以在项目里并没有用存储过程来处理大批量的insert、 delete或update的操作。


### 二、使用 JDBC 开发

屏蔽了数据库中的实现细节，只要数据表名不变、表中字段不变，不同种类数据库之间只要修改很少的 jdbc 代码就可以实现迁移；

- 使用 jdbc 开发，通过 `getString(字段名)、getInt()、getfloat() 、getDate()` 等获取字段值；
- 一定要在 `finally` 中关闭所有的链接；
- MySQL 更换到 Oracle 时 JDBC 需要更改的地方；
  - 导入的驱动文件不再是 `mysql-connector-java.jar`，改为：`ojdbc.jar`；
  - 通过 `Class.forName(“com.musql.jdbc.Driver”)` 装载的驱动修改为 Oracle 的：`“oracle.jdbc.driver.OracleDriver”`;
  - `getConnection()` 方法中的数据库连接的 URL 需要由 `”jdbc: mysql://localhost:3306/数据库名“`  改为 `”jdbc:oracle:thin:@localhost:1521:数据库名“`


### 三、优化数据库部分代码

#### （一）将相对固定的连接信息写入配置文件

- 减少多个代码同时用到数据库连接参数需要多次配置；
- 对于测试和生产两种生产环境，可以使用两种不同的配置文件，使用命令行切换即可；

db.properties 写法：
```properties
driver = com.mysql.cj.jdbc.Driver
url = jdbc:mysql://localhost:3306/数据库名?serverTimezone = GMT%2B8
user = 用户名
password = 密码
```

#### （二）用 PreparedStatement 以批处理的方式操作数据库

**PreparedStatement 是预处理，一般用于批处理和防止 SQL 注入**；
当需要在一个方法中执行多个插入（更新或者删除）操作时候，可以使用批处理；

如果通过 `ps.executeUpdate` 的方式一条条地执行语句，那么每次执行语句都包括 “ 连接数据库＋执行语句＋释放数据库连接"  3 个动作。相比之下，如果用批处理（executeBatch）的方式， 那么耗费的代价是 “—次连接＋多次执行＋一次释放 ” ，这样就能省去多次连接和释放数据库资源从而提升操作性能；

- PreparedStatement里，占位符的编号是从 1 开始的，而不是从 0 开始的；
- 批量操作能提升效率， 但一次性操作多少 ，效率能提升多高？这在不同的数据库中是不同的，一般每批是操作500 - 1000条语句。不要太多不然缓存放不完；


#### （三）用 PreparedStatement 对象防止 SQL 注入

#### （四）使用 C3P0 连接池
如果操作很频繁，那么频繁的创建和关闭数据库连接动作会极大地减低系统的性能，在这种情况下，可以使用连接池；
- 常见连接池有：C3P0 和 DBCP

- C3P0 连接池的常用属性：
![C3P0连接池常见属性]($resource/C3P0%E8%BF%9E%E6%8E%A5%E6%B1%A0%E5%B8%B8%E8%A7%81%E5%B1%9E%E6%80%A7.png)

### 四、事务
事务(Transaction)是—组针对数据库的操作， 这些操作要么都做， 要么都不做， 是 一个不可分割的 SQL 语句集合。

#### （一）开启事务，合理的提交和回滚
在JDBC中， 一般采用如下的方法使用事务。

- 通过 `connection.setAutoCommit(false)` , 设置不是自动提交。在 JDBC 中，一般默认是自动提交，即有任何增删改的 SQL 语句都会立即执行。如果设置了非自动提交，要在用好事务后设置回 “ 自动提交” 。

- 在合适的地方用 `connection.commit();` 来提交事务，一般是在执行结束后提交事务， 这样就会同时执行事务中的所有操作。

- 可以通过 `connection.rollback()` 来回滚事务，回滚语句—般是放在 catch 从句中；一旦出现异常，就在 catch 中实现回滚；

#### （二）事务中的常见问题：脏读、幻读和不可重复读
在项目中， 如果**同时对一张表有两个（或多个） 事务进行读写操作时**，很容易出现数据读写错误的问题，具体表现形式有脏读、幻读和不可重复读。

- **脏读 (dirtyread) 是指—个事务读取了另—个事务尚未提交的数据**
A 值为 1，B 将其值设置为 2（但是尚未提交这个修改事务），A 读到值变为 2，结果 B 回滚了事务又将 A 值改为 1,则 A 读到的 2 就是一个脏数据，对应的操作为**脏读**；
**避免方法：** 如果在第—个事务提交前，任何其他事务不可读取其修改过的值，则可以避免出现该问题。

- **幻读 (phantom read) 是指一个事务的操作会导致另一个事务前后两次查询的结果不同**
A 事务查询值为 1 的数据，读取到共 10 条记录，然后事务 B 插入一条数据值为 1，A 再次以同样条件读取就会得到 11 条；
**避免方法：** 如果在操作事务完成数据处理之前 ， 任何其他事务都不可以添加新数据， 则可避免该问题。

- **不可重复读(non-repeatable read)是指—个事务的操作导致另—个事务前后两次读取到不同的数据**。例如， 同—查询 在同一事务中多次进行， 由于其他事务提交了所做的修改（或和加或删除等操作） ， 这样每次查询会返回不同的结果集，这就是不可重复读。
A 读到的值为 1，但是针对这个值的修改没有完成，事务 B 修改了值为 2，并提交了事务，则事务 A 再次读取值变为了 2；
**避免方法：** 只有在修改事务完全提交之后才允许读取数据；

- 幻读和不可重复读两者都表现为两次读取的结果不一致.
- 但如果你从控制的角度来看,   两者的区别就比较大
  - 对于幻读,   要锁住满足条件及其相近的记录  ：即需要锁住表
  - 对于不可重复读,   只需要锁住满足条件的记录 ： 即需要锁住行
  - 幻读的重点在于 insert；
  - 不可重复读的重点在于：update 和 delete

如果使用锁机制来实现这两种隔离级别，在可重复读中，该sql第一次读取到数据后，就将这些数据加锁，其它事务无法修改这些数据，就可以实现可重复 读了。但这种方法却无法锁住insert的数据，所以当事务A先前读取了数据，或者修改了全部数据，事务B还是可以insert数据提交，这时事务A就会 发现莫名其妙多了一条之前没有的数据，这就是幻读，不能通过行锁来避免。需要Serializable隔离级别 ，读用读锁，写用写锁，读锁和写锁互斥，这么做可以有效的避免幻读、不可重复读、脏读等问题，但会极大的降低数据库的并发能力。

**所以说不可重复读和幻读最大的区别，就在于如何通过锁机制来解决他们产生的问题。**
上文说的，是使用悲观锁机制来处理这两种问题，但是MySQL、ORACLE、PostgreSQL等成熟的数据库，出于性能考虑，都是使用了以乐观锁为理论基础的MVCC（多版本并发控制）来避免这两种问题。

*   悲观锁
正如其名，它指的是对数据被外界（包括本系统当前的其他事务，以及来自外部系统的事务处理）修改持保守态度，因此，在整个数据处理过程中，将数据处 于锁定状态。悲观锁的实现，往往依靠数据库提供的锁机制（也只有数据库层提供的锁机制才能真正保证数据访问的排他性，否则，即使在本系统中实现了加锁机 制，也无法保证外部系统不会修改数据）。

在悲观锁的情况下，为了保证事务的隔离性，就需要一致性锁定读。读取数据时给加锁，其它事务无法修改这些数据。修改删除数据时也要加锁，其它事务无法读取这些数据。

*   乐观锁
相对悲观锁而言，乐观锁机制采取了更加宽松的加锁机制。悲观锁大多数情况下依靠数据库的锁机制实现，以保证操作最大程度的独占性。但随之而来的就是数据库性能的大量开销，特别是对长事务而言，这样的开销往往无法承受。

而乐观锁机制在一定程度上解决了这个问题。乐观锁，大多是基于数据版本（ Version ）记录机制实现。何谓数据版本？即为数据增加一个版本标识，在基于数据库表的版本解决方案中，一般是通过为数据库表增加一个 “version” 字段来实现。读取出数据时，将此版本号一同读出，之后更新时，对此版本号加一。此时，将提交数据的版本数据与数据库表对应记录的当前版本信息进行比对，如 果提交的数据版本号大于数据库表当前版本号，则予以更新，否则认为是过期数据。

要说明的是，MVCC的实现没有固定的规范，每个数据库都会有不同的实现方式，这里讨论的是InnoDB的MVCC。

#### （三）事务隔离级别
用户可以通过事务隔离级别来解决上述在事务中读写不一致的问题。在JDBC中，有5个常量来描述事务隔离级别，级别从低到高依次如下。

(1)读取未提交：`TRANSACTION_READ_UNCOMMITTED`, 允许脏读、不可重复读和幻读。
(2)读取提交：`TRANSACTION_READ_COMMITTED`, 禁止脏读，但允许不可重复读和幻读。【Oracle 默认级别】
(3)可重读：`TRANSACTION_REPEATABLE_READ`,禁止脏读和不可重复读，但允许幻读。【MySQL 默认级别】
(4)可串化：`TRANSACTION_SER IALIZABLE`, 禁止脏读、不可重复读和幻读。
(5)还有一个常量是`TRANSACTION_NONE`,如果读取这个值，那么将使用当前数据库说指定的事务隔离级别；

综上所述，如果设置高级别的事务隔离级别，那么数据库系统就要采取额外的措施来保证这个设置。会造成后续的操作和功能等待。

√: 可能出现    ×: 不会出现
| | 脏读 | 不可重复读 | 幻读 |
|---|----|---|---
| Read uncommitted | √ | √ | √ |
| Read committed | × | √ | √ |
| Repeatable read | × | × | √ |
| Serializable | × | × | × |



## 章六、反射机制和代理模式


### 字节码和反射机制
字节码 (Byte Code) 是Java 语言跨平台特性的重要保障， 也是反射机制的重要基础。通过反射机制 ， 我们不仅能看到一个类的属性和方法， 还能在—个类中调用另一个类的方法，但前提是要有相关类的字节码文件（也就是 class 文件）。

- .Java 文件编译生成的 .class 文件可以在各种平台中运行（只要其有 Java 运行环境）；
- Class 类的全称是 Java.lang.Class, 当一个类或接口（总之是 Java 文件被编译后的 class 文件）被装入Java 虚拟机 (JVM) 时便会产生一个与它相关联的 java.lang.Class 对象， 在反射部分的代码中， 我们一般通过 Class 来访问和使用目标类的属性和方法。



### 反射的常见用法

一是 “查看＇ ， 如输入某个类的属性方法等信息；
二是 ＂ 装载'' 如装载指定的类到内存中；
三是 “调用” 如通过输入参数， 调用指定的方法。

#### 查看
- 查看属性的修饰符、类型和名称 Demo：
```java
class MyValClass {
	private int val1;
	public String val2;
	final protected String val3 = "Java";
}

public class ReflectionReadVar{ 
	public static void main(String[] args) {
		Class<MyValClass> clazz = MyValClass.class;
		//获取这个类的所有属性
        Field[] fields = clazz.getDeclaredFields();
	    for(Field field : fields) {
		   //输出修饰符
		   System.out.print(Modifier.toString(field.getModifiers()) + "\t");
		   //输出属性的类型
		   System.out.print(field.getGenericType().toString() + "\t");
		   //输出属性的名字
		   System.out.println(field.getName());
	    }
	}
}
/**output:
 * private	int	val1
 * public	class java.lang.String	val2
 * protected final	class java.lang.String	val3
 */
```

- 查看方法的返回值类型、参数和名称
```java
class MyFuncClass {
	public MyFuncClass(){}
	public MyFuncClass(int i){}
	private void f1(){}
	protected int f2(int i){return 0;}
	public String f2(String s) {return "Java";}
}

public class ReflectionReadFunc {
	public static void main(String[] args) {
		Class<MyFuncClass> clazz = MyFuncClass.class;
		// 返回所有的方法，但是不包括继承的方法和构造方法
        Method[] methods = clazz.getDeclaredMethods();
        for (Method method : methods) {
            System.out.println(method);
        }
        System.out.println("***********");
       
        //得到所有的构造函数
        Constructor[] c1 = clazz.getDeclaredConstructors();
        //输出所有的构造函数
        for(Constructor ct : c1){
            System.out.println(ct);
        }
	}
}
/**
 * output:
 * protected int chapter6.MyFuncClass.f2(int)
 * public java.lang.String chapter6.MyFuncClass.f2(java.lang.String)
 * private void chapter6.MyFuncClass.f1()
 * ***********
 * public chapter6.MyFuncClass()
 * public chapter6.MyFuncClass(int)
 */
```

#### 装载
- 通过 forName 和 newInstance 方法加载类
  - Class.forName 方法最常用的用法就是装载数据库的驱动；
  - 本质上 forName 的作用仅仅是返回一个 Class 类型的对象，newInstance 方法作用是加载类；即 newInstance 作用是通过 Java 虚拟机的类加载机制把指定的类加载到内存中。但是 newInstance 方法只能调用无参构造函数进行加载，如果有参数得使用 new 关键字。
```java
class MyClass {
	public void print() {
		System.out.println("Java");
	}
}

public class ForClassDemo {
	public static void main(String[] args) {
		MyClass myClassObj = new MyClass();
		myClassObj.print();//输出是Java
		System.out.println("*************");

		try {
		    // forName() 中的类名应该是完整的类名
			Class<?> clazz = Class.forName("chapter6.MyClass");
			MyClass myClass = (MyClass)clazz.newInstance();
			myClass.print();//输出是Java
		} catch (ClassNotFoundException e) {
			e.printStackTrace();
		} catch (InstantiationException e) {
			e.printStackTrace();
		} catch (IllegalAccessException e) {
			e.printStackTrace();
		}
	}
}
/**
 * output:
 * Java
 * *************
 * Java
 */
```



#### 通过反射机制调用类的方法
- 通过什么方式调用：
- 如何输入参数：
- 如何得到返回结果：
demo：
```java
class Person {
	private String name;

	public Person(String name) {
		this.name = name;
	}

	public void saySkill(String skill) {
		System.out.println("Name is:" + name + ", skill is:" + skill);
	}

	public int addSalary(int current) {
		return current + 100;
	}
}

public class CallFuncDemo {
	public static void main(String[] args) {
		Class c1azz = null;
		Constructor c = null;
		try {
		// 通过反射调用类的构造函数来创建对象
			// 得到 Class 类型的对象，其中包含了 Person 类的信息
			c1azz = Class.forName("chapter6.Person");
			// 得到 Person 类带参数的构造函数；c 值为：public chapter6.Person(java.lang.String)
			c = c1azz.getDeclaredConstructor(String.class);
			// 通过带参数的构造函数创建一个Person类型对象
			Person p = (Person)c.newInstance("Peter");
			//output: Name is:Peter, skill is:java
			p.saySkill("Java");

			// 调用方法，必须传递对象实例，同时传递参数值
			Method method1 = c1azz.getMethod("saySkill", String.class);
			// 因为没返回值，所以能直接调
			// 参数一指定该方法由哪个对象调用，参数二指定该方法的参数
            method1.invoke(p, "C#");
            
            Method method2 = c1azz.getMethod("addSalary", int.class);
            Object invoke = method2.invoke(p, 100);
            //输出200
			System.out.println(invoke);
				} catch (ClassNotFoundException e) {
			e.printStackTrace();
		} catch (NoSuchMethodException  e1) {
			e1.printStackTrace();
		}
		catch (InstantiationException e) {
			e.printStackTrace();
		} catch (IllegalAccessException e) {
			e.printStackTrace();
		} catch (InvocationTargetException e) {
			e.printStackTrace();
		}
	}
}
/**output:
 * Name is:Peter, skill is:Java
 * Name is:Peter, skill is:C#
 * 200
 */
```


### 代理模式和反射机制
反射的使用场景之一就是在代理模式；通过反射机制实现动态代理功能；

~~P190-197~~





## 章七：多线程和并发编程

### 线程的基本概念和实现多线程的基本方法

实现多线程的两种基本方法，一种是通过extends Thread 类的方式来实现，另一种是通过impleme nts Runnable 接口的方式来实现。
通过这两种方法创建的线程都无法返回结果值， 而在本节的后半部分，将讲述通过Callable 让线程返回执行结果的方法。


#### 线程和进程

进程是实现某个独立功能的程序，它是操作系统（如windows系统）进行资源分配和调度的一个独立单位，也是可以独立运行的一段程序。
线程是—种轻量级的进程， 它是—个程序中实现单—功能的—个指令序列，是一个程序的—部分，不能单独运行， 必须在一个进程环境中运行。

**两者区别：**
- 进程间相互独立， 但同—进程的各线程会共享该进程所拥有的资源，而进程则是用独占的方式来占有资洒，也就是说，进程间不能共享资源。
- 线程上下文切换（如—个从线程切换到另外—个线程）要比进程上下文切换速度快得多。
- 每个线程都有—个运行的入口 ， 顺序执行序列和出口，但是线程不能独立执行，必须依靠进程来调度和控制线程的执行。
- 一般操作系统级别会偏重于 “进程” 的角度和管理，而应用项目（如某个在线购物平台）会偏重于 “线程” ， 如在某应用项目中的某些组件可以以多线程的方式同时执行。也就是说 ， 在编程时会更偏重千 “多线程” ，而不是 “多进程” 。

#### 线程的生命周期
在Java 中，—个线程的生命周期中会有4种状态：初始化、可执行、阻塞和死亡状态
- 我们可以通过new语句创建一个线程对象，这时还没有调用它的start()方法，此时线程也没有分配到任何系统资源，这时称为初始化状态。
- 当我们调用了start()方法之后，它会自动调用线程对象的run()方法，此时线程如果分配到了CPU时间就可以开始运行，否则等待分配CPU时间，但无论是否分配到了CPU时间， 线程此刻都处于可执行状态。
- 通过某些方法，如sleep()方法或wait()方法，我们可以把线程从可执行状态挂起。此时线程不会分配到CPU时间， 因此无法执行， 这时称为阻塞状态。
- 当线程睡眠了sleep参数所指定的时间后， 能自动地再次进入可执行状态，这时也可以通过notify()方法把因调用wait()方法而处于阻塞状态的线程变为可执行状态， 此刻该线程又有机会得到CPU时间继续运行了。
- 线程run()方法中的逻辑正常运行结束后就进入了 死亡状态。 调用stop()方法或 destroy()方法时也会非正常地终止当前线程， 使其进入死亡状态， 之后该线程就不存在了。

![线程间的状态转换]($resource/%E7%BA%BF%E7%A8%8B%E9%97%B4%E7%9A%84%E7%8A%B6%E6%80%81%E8%BD%AC%E6%8D%A2.png)

#### 通过 extends Thread 实现多线程
因为线程的调度工作由操作系统完成，因此线程的执行次序是不可控的，导致多线程运行时每次过程都不一致；
Demo:
```java
package chapter7;

public class SimpleThread extends Thread {
	// 线程编号
	int index;

	// 通过构造函数指定该线程的编号
	public SimpleThread(int index) {
		this.index = index;
		System.out.println("Create Thread[" + index + "]");
	}

	// run方法，当调用线程的start方法时会自动调用该方法，此时线程进入可执行状态
	@Override
	public void run() {
		for (int j = 0; j <= 3; j++) {
			System.out.println("Thread[" + index + "]:running time " + j);
		}
		// 当前线程运行结束
		System.out.println("Thread[" + index + "] finish");
	}

	public static void main(String args[]) {
	    int threadCount = 3;
		for (int j = 0; j < threadCount; j++) {
			// 实例化该类型的对象，并直接调用start方法直接把线程拉起
			// 这个方法会自动调用run方法
			Thread t = new SimpleThread(j + 1);
			t.start();
		}
	}
}
/**
 * output:
 * Create Thread[1]
 * Create Thread[2]
 * Create Thread[3]
 * Thread[1]:running time 0
 * Thread[2]:running time 0
 * Thread[1]:running time 1
 * Thread[1]:running time 2
 * Thread[1]:running time 3
 * Thread[3]:running time 0
 * Thread[2]:running time 1
 * Thread[2]:running time 2
 * Thread[2]:running time 3
 * Thread[2] finish
 * Thread[3]:running time 1
 * Thread[1] finish
 * Thread[3]:running time 2
 * Thread[3]:running time 3
 * Thread[3] finish
 */

```


#### 通过 Implements Runnable 实现多线程（线程优先级）
一般用于该类已经通过 extends 继承了一个类，则不能再使用 extends 继承 Thread 类来实现多线程，可以采用 implements Runnable 方式实现多线程。

默认 1-10 共 10 个优先级别，数字越小级别越高，默认级别为 5，**但是高优先级的线程仅仅是比低优先级的先运行的概率大，不是绝对的能先执行。**
```java
package chapter7;

//实现Runnbale接口，此时这个类就可以extends其他的父类了
public class ThreadPriority implements Runnable {
	// 线程编号
	int number;

	public ThreadPriority(int num) {
		number = num;
		System.out.println("Create Thread[" + number + "]");
	}

	// run方法，当调用线程的start方法时会调用该方法
	@Override
	public void run() {
		for (int i = 0; i <= 3; i++) {
			System.out.println("Thread[" + number + "]:Count " + i);
		}
	}

	public static void main(String args[]) {
		// 定义线程t1，并设置其优先级为5
		Thread t1 = new Thread(new ThreadPriority(1));
		t1.setPriority(1);
		// 定义线程t2，并设置其优先级为7
		Thread t2 = new Thread(new ThreadPriority(2));
		t2.setPriority(7);
		// 启动这两个线程
		t1.start();
		t2.start();
	}
}
/**
 * output:
 * Create Thread[1]
 * Create Thread[2]
 * Thread[1]:Count 0
 * Thread[2]:Count 0
 * Thread[1]:Count 1
 * Thread[2]:Count 1
 * Thread[1]:Count 2
 * Thread[2]:Count 2
 * Thread[1]:Count 3
 * Thread[2]:Count 3
 */
```



### 多线程的竞争与同步

#### 通过 sleep 方法让线程释放 CPU 资源
通过线程类 Thread 的一个 sleep(参数为毫秒数) 静态方法，让当前运行的线程在这段时间内进入阻塞状态。 阻塞状态过去后， 该线程会重新进入可执行状态。
示例：比如某个线程因数据库连接异常而无法连接到数据库， 这时我们可以通过sleep方法让该线程阻塞一段时间， 过后再重新连接。

Demo：
```java
package chapter7;

public class ThreadSleep extends Thread {
	@Override
	public void run() {
		Long curTime = System.currentTimeMillis();
		// sleep方法会抛出InterruptedException异常
		// 需要用try-catch语句进行捕捉
		try {
			sleep(2000);
		} catch (InterruptedException e) {
		}
		System.out.println("ts线程阻塞的时间" + (System.currentTimeMillis() - curTime) + "毫秒");
	}

	public static void main(String arg[]) {
		ThreadSleep ts = new ThreadSleep();
		ts.start();
		Long curTime = System.currentTimeMillis();
        try {  
            Thread.sleep(1000);  
        } catch (InterruptedException e) {
            e.printStackTrace();  
        }  
        System.out.println("主线程阻塞的时间" + (System.currentTimeMillis() - curTime) + "毫秒");
	}
}
/**
 * output:
 * 主线程阻塞的时间1000毫秒
 * ts线程阻塞的时间2001毫秒
 */

```

####  Synchronized 作用在方法上
解决多线程并发执行某个方法或者代码引发不同线程同时修改同块存储空间。














