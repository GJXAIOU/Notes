---
style: summer
---

# Java 核心技术及面试指南总结
@toc

# 章一 ：基本知识

## 一、代码调试

### （一）断点
在代码合适的地方加上断点（point），一般在循环等可能出错的地方加上断点，然后“Debug As” -> “Java Application” ，使用 Debug 模式启动程序。
F6 进入下行代码，F5,进入当前方法中， F8 跳转到下一个断点；

### （二）输入运行时候的参数
一般代码在上生产环境（Prod）之后需要在测试环境（QA）进行观察，程序传参：右击 -> “Run As ” -> “Run Configurations”,然后在标签：“Arguments”，在里面输入参数，最后点击“Run”按键。


## 二、技术学习
先广度再深度，掌握必备的知识体系，然后根据**实际需求**慢慢完善。

虽然在Java中也有（如Swing或AWT)基于本地窗口界面（俗称C/S架构）的开发接口， 但目前这些用得很少，不建议大家在初级阶段了解这些。
根据大多数程序员的实践经验， 我们更应该通过SpringMVC或MyBatis等基于Java Web的架构或技术来提升自己的市场竞争力，但本书讲到的JavaCore技术绝对不是没有价值的。架构技术，JavaWeb技术和Core技术在大多数项目中的不同分工， 如表1.1所示。
 
| 技术种类  | 具体的技术 | 应用的层面和作用
|---|---|---|
|架构技术   | 如实现负载均衡的 nginx, 实现消息服务的kafka  | 在架构层面为 整个（如在线购物）系统提供（如消息、 负载 均衡等）服务。一旦有流量提升的需求 ， 则可以采用扩展服务器的方式来应对  |
| Java Web技术  |Spring MVC+MyBatis 框架   | 如用户下了订单， 这个请求会从前端发送到后瑞， 用Spring MVC+MyBatis框架， 能很便捷地实现这一流程而且这套框架能很好地实现类似订单管理这样有前后瑞交互 的各类Web层面的业务  |
|Core技术   | 集合、 数据库 IO、 异常处理等技术  | 在实现诸多业务时（如订单管理业务中）会大量用到这些 技术  |

**JavaWeb技术可以用在宏观架构的方面， 而Core技术则会被大量地用在微观层面， 也就是业务逻辑中**。

![Java  Core可以延后学习知识点]($resource/Java%20%20Core%E5%8F%AF%E4%BB%A5%E5%BB%B6%E5%90%8E%E5%AD%A6%E4%B9%A0%E7%9F%A5%E8%AF%86%E7%82%B9.png)

![Java Web可以延后学习知识点]($resource/Java%20Web%E5%8F%AF%E4%BB%A5%E5%BB%B6%E5%90%8E%E5%AD%A6%E4%B9%A0%E7%9F%A5%E8%AF%86%E7%82%B9.png)



# 章二、基本语法中的常用技术点精讲

## 一、基本语法
- char 用来表示单个字符， 它有两种赋值方式： 用单引号（而不是双引号）来表示某个字符【推荐】， 或者用整数来表示这个字符所对应的 unicode 编码值。

- `i++`是指表达式运算完后再给 i 加1 , 而++i 是指先 i 加1然后再运算表达式的值。
```java
int i = 5;
int k = i++ * 3;
// 先运算表达式： i * 3 = 15,然后 i++;因此k = 15,i = 6;
```
为 了提升代码的可读性，**建议左加加和右加加操作不应（或尽量少）和其他操作符混用**，如果实在有必要， 要分开写；例如：`int k = i * 3; i++;`

- == 和 equals
对于基本数据类型（如int) ＝＝可以用来比较它们的值是否相等， 对于封装类型（如Integer) , ==是比较它们在内存中存放的地址是否一致，而封装在Integer中的equals方法才用来比较它们的值是否相等。

- 一旦在条件表达式中出现多个 ＆＆或|| 符号， 那么所用到的测试案例就得成指数倍上升，尽量减少使用，对于非要写可以采用 if 嵌套代替；
- 同上，为了避免条件表达式短路，**应当只在条件表达式中做简单的判断操作， 而不应进行数值运算**， 从而避免在写代码时出现短路现象。


## 二、 String

### （一）定义
```java
String str = "abc"; 
String str = new String("abc"); 
```
第1行中 ， 定义的是—个常量 ， 在第2行中 ， 诵过new关键字， 定义了—个变量;
在第1行**常量池**（而不是堆空间）中 开辟了—块空间，在其中存放了字符串abc, 并通过str对象指向这个常量对象。 
而在第2行**堆空间**中通过new关键字开辟了一块内存，在其中存放字符串abc, 并把内存的地址（也就是引用）赋予str变量。
**具体见之前的纸质笔记**

### （二）特性：内存值不变
`String abc = new String("123");`,可以想象—下系统的操作，首先在内存中的某个位置（假设1000号内存）开辟了—块空间， 其中存放123， 随后让abc指向了1000号内存，如图2.1所示。
这里说明—个现象，**String对象所指向内存地址中的值是不可变的**。具体来讲， 这里1000号内存中的值123在之后的操作过程中不能被改变。
可能这里会有疑问:不是可以通过abc = new String("456"); 的操作来改变abc值的吗？下面再 来通过图2.2来观察把abc设置成456的操作。

![string内存值不变]($resource/string%E5%86%85%E5%AD%98%E5%80%BC%E4%B8%8D%E5%8F%98.png)

系统会重新开辟新的内存（假设2000号），在其中存放456, 随后abc 会指向它从而得到新的值。此时，1000号内存空间的值确实没改变，但 abc已经不再指向它了。此时存放123的1000号内存 由于 没有对象会引用，因此它就成了内存碎片， 要到下次Java虚拟机回收内存时才会被回收。

```java
String a = "123456789"; 
System. out.println(a.substring(0,5));//12345 
String b = "123456789";//假设1000号内存放123456789 
b.substring(0,5); //开辟了2000号内存， 存放12345 
//b = b.substring(0,5); //开辟3000号内存， 存放12345 
System.out.println(b); //1000号内存依然是123456789
```
- 同样出于” 不可变” 的特性，针对String的操作需要用另外—个String对象来接收， 否则会得到预期之外的效果。因此写代码时应尽量避免在循环中（或其他清况下）大批量出现针对字符串的操作（不仅限于连接）。

- **尽可能使用常量** ， 如String a = "123", 避免使用变量， 如String a = new String(“123);
- **尽量避免大规模地针对String的（如连接字符串）操作** ， 因为这样会频繁地生成内存碎片， 导致内存性能问题。 如果遇到这种业务需求， **应改用后面提到的StringBuilder 对象**;

## 三、StringBuilder
StringBuilder是字符串变量， 不是像String那样是不可变的， 而是可改变的对象， 当用它进行字符串操作时， **是在—个对象上操作的**， 这样就不会像String那样创建一些额外的对象；

```java
StringBuilder builder = new StringBuilder(); 
builder.append("123") .append(456); //123456 System.out.println(builder.substring(0,4));//1234 System.out.println(builder);//123456 
builder.replace(0,3,"a"); 
System.out.println(builder); //a456 
builder.deleteCharAt(O); 
System.out.println(builder); //456 
```
会经常错误地认为substring会自动地把结果写回到StringBuilder对象中 ，但事实上这个方法的返回类型是String, **需要用String val = builder.substring(0,4); 的方法来接收返回值** 。

## 四、 print()
System.out.println方法 能根据不同类型的输入参数（如 int、float、Integer等）适当地完成输出动作， 称其为多态；**虽然 sysout 的作用的输出字符串，但是 Int /float 等等会自动转换成封装类，然后输出时候默认调用 tostring 方法**。

如果一个类中没有重写 tostring 方法，当打印类名的时候，默认调用 Object 类中的 tostring 方法，由于 Object中的toString方法 是返回该对象的内存地址， 因此可以看到`类名@内存地址`的输出。


## 五、 方法的参数和返回值
```java
/**
 * @author GJXAIOU
 * @create 2019-08-19-18:59
 */
class Person{
    void hello(String name){
        name = "hello";
    }
}

public class Function{
    public static void main(String[] args) {
        String ha = "haha";
        Person person = new Person();
        person.hello(ha);
        System.out.println(ha);
    }
}
// output : haha
```

person.hello(ha) 输入的参数 ha 的内存地址是 1000 (其中的值是 haha), 但在调用方法时，会把1000号内存做个副本放到2000号内存中， 2000号内存的值也是 haha, 在调用第2行方法时，是针对这个副本（也就是2000号内存）而非1000号内存操作的,方法执行完成之后，2000 号内存这个副本就删除了；**由于是针对副本进行操作的， 因此在方法内部针对参数做的操作不会返回**。

## 六、访问控制符

|访问控制符 | 同类 | 同包 | 子类 | 不同包
|---|---|---|---|---
|public|可以|可以|可以|可以
|protected|可以|可以|可以|不可以
|默认|可以|可以|不可以|不可以
|private|可以|不可以|不可以|不可以

## 七、静态
- 静态类中只能使用静态方法、静态变量；
- 静态变量相当于全局变量；
- 接口的缺省属性是：`public static final`

### （一）抽象类和接口的区别
**抽象类是对概念的归纳， 接口是对功能的归纳**（可以解释，不能继承多个类但是可以实现多个接口，即不能多个父类但是可以有多种能力）
生活中的 Demo ：
![抽象类和接口区别]($resource/%E6%8A%BD%E8%B1%A1%E7%B1%BB%E5%92%8C%E6%8E%A5%E5%8F%A3%E5%8C%BA%E5%88%AB.png)


## 八、继承
- 子类覆盖父类的方法：重写   ；子类方法不能缩小父类方法的访问权限，只能相等或者扩大；
- 子类方法不能抛出比父类方法更多的异常；


## 九、垃圾回收

当虚拟机回收类时， 会自动地调用该类的finalize方法， 如果类中没定义， 会调用Object类中的finalize, 而Object中的finalize方法是空白的。
首先也要了解finalize方法有什么用，它什么时候会被调用；其次，强烈建议 大家在自定义的类中不要重写finalize 方法， 如果写错 ， 就会导致类无法被回收， 从而导致内存使用量直线上升最后抛出内存溢出的错误。


## 十、构造函数
如果子类定义了有参构造函数但是没有写方法体，同时父类中只有一个有参构造函数会报错，因为子类的构造函数中没有方法体，系统会默认添加`super()`，即调用父类中无参的构造方法，但是因为没有所有报错；可以在父类中添加一个无参构造方法或者使用：`super(i)`调用有参构造方法；



# 章三：集合类和常用的数据结构
- 根据数据存储格式的不同，可以将集合分为两类：
  - 一类是以Collection为基类的**线性表类**， 如数组和Arraylist、Set、Vector、Queue、stack等；
  - —类是以Map为基类的**键值对类**， 如HashMap和 HashTable 。
- 集合是容器， 其中不仅可以存储String、int等数据类型， 还可以存储自定义的Class类型数据。
- Set 也属于线性表，但是不能存储重复元素，而且在一个 Set 对象中最多只能存储一个 null 对象。


## 一、 ArrayList 和 Linkedlist
**Arraylist是基于数组实现的, 而LinkedList是基于双向链表实现的**。
list是封装了针对线性表操作的接口，Arraylist和Linkedlist是两个实现类；

- 数组中第 i 个元素位置：＝第0号元素的位置+(i-1) X 每个元素的长度。

- 如果 一次性地通过add从集合尾部添加元素，添加 完成后只需读取—次，那么建议使用Arraylist。
- 如果要频繁地添加元素, 或者在完成添加元素后会 频繁地通过indexOf 方法从集合里查找元素 ,可以使用Linketlist, 原因是它的随机添加和随机查找的总时间消耗要小于 ArrayList。
- 注意如果在代码中indexOf 的操作过于频繁从而成为项目运行的 ＂瓶颈” 时，可 以考虑后面提到的HashMap对象 。
-  Arraylist和Linkedlist都是线程不安全的。

- Vector和Arraylist (或者再加 上Linkedlist)集合对象之间有什么区别？
  - Arraylist是基于数组的， 而Vector对象也是基于数组 实现
  - (1) Vector是线程安全的， Arraylist是线程不安全的， 所以在插入等操作中，Vector 需要—定开销来维护线程安全 ，而大多数的程序都 运行在单线程环境下，无须考虑线程安全问题，所以大多数的单线程环境下Arraylist的性能要优于Vector。
  - (2)例如 ， 刚开始程序员创建了长度是10的Vector和Arraylist, 当插入第11个元素时， 它们都会扩充 ，Vector会扩充100%, 而 ArrayList 会扩充50%, 所以从节省内存空间的角度来看，建议使用Arraylist。

- 泛型 
ArrayList list = new ArrayList（）；中 list 对象实际上存储的是 Object 类型的数据；

## 二、 Set 集合去重
例如在 set 集合中添加自定义类对象，怎么实现去重（重复的不添加）

- 步一：自定义类对象遵从 Comparable 接口；
- 步二：重写 ComparaTo 方法；
```java
package chapter3;
import	java.util.TreeSet;
import	java.util.HashSet;

/**
 * @author GJXAIOU
 * @create 2019-08-17-14:45
 */

// 步一：自定义类遵从Comparable
class Student implements Comparable{
    private int id;

    public Student(int id) {
        this.id = id;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    /**
     * 重写了用于判断两个Student对象是否相等的equals方法， 如果不 重写， 将用Object的方法
     *（这个方法是通过判断两个对象的地址是否—致来判断两个对象是否相等的）。
     * TreeSet对象不会根据equals方法判断是否重复，也就是说， 即使程序员注释了这个方法，
     * 也不会影响运行结果，但在自定义类中，重写equals方法是很好的习惯。
     * @param s 传入Student类对象
     * @return Boolean类型
     */
    public boolean equals(Student s) {
        if (s.getId() == this.id){
            return true;
        }else {
            return false;
        }
    }

    // 步二：实现comparaTo方法
    @Override
    public int compareTo(Object o) {
        // 判断是否是学生类型
        if (o instanceof Student) {
            Student student = (Student) o;
            // 如果是学生类型，如果学号相等则不加入set
            if (this.getId() == student.getId()) {
                return 0;
            }else {
                return student.getId() > this.getId() ? 1 : -1;
            }
            // 不是学生类型对象则不加入
        }else {
            return  0;
        }
    }
}

public class SetDupDemo {
    public static void main(String[] args) {
        HashSet hashSet = new HashSet();
        hashSet.add(1);
        hashSet.add(1);
        System.out.println(hashSet.size());

        Student student1 = new Student(1);
        Student student2 = new Student(1);
        TreeSet<Student> treeSet = new TreeSet<>();
        treeSet.add(student1);
        treeSet.add(student2);
        System.out.println(treeSet.size());
    }
}

```


## 三、 TreeSet、HashSet 和 LinkedHashSet 区别
HashSet是基于哈希表(Hash表）实现的，它不能保证线程安全 ， 其中允许存在null 元素， 但null元素只能有1个。
当程序员向HashSet中插入—个对象时 ， HashSet会调用该对象的hashCode()方法（如果该对象没定义， 会调用 Object)来得到该对象的hashCode值；然后会根据hashCode 值来决定该对象在HashSet中的存放位置， 如果遇到两个对象的hashCode值—致的清况， 则说明它们相等 ， HashSet同样不会允许插入重复的元素。HashSet不能保证插入次序和遍历次序—致。
LinkedHashSet同样是基于Hash表，它也 是根据元素的hashCode值来决定元素的存储位置的， 但是它同时也采用了链表的方式来保证插入次序和遍历次序一致；

而 TreeSet是SortedSet接口的唯一实现 类， 它是用二叉树存储数据的方式来保证存储的元素 处于 有序状态，会发现输出的次序和插入次序不一致，而且数据已经被排序。不允许插入 null;

如果TreeSet中存储的不是基本数据 类型 ， 而是自定义的class, 那么这个类必须实现Comparable接口中的compareTo方法， TreeSet会根据compareTo中的定义来区分大小， 最终确定TreeSet中的次序。



## 四、 集合中存放的是引用
**自定义类是以引用的形式放入集合**
例如实例化一个对象，将其某个属性值设定为 X，然后创建两个不同的 ArrayList，将该实例的对象分别放入其中，则修改其中一个 list 中的对象的值，也会造成另一 list 中对象值的变化；


**解决方法：两个互不干扰** 即实现一个集合做备份，另一个集合做修改；

- 使用 clone,实现深复制
方法：
步一：自定义实现 Cloneable 接口
步二：实现 clone 方法
```java
// 1.自定义类实现Cloneable接口
class CarDeepCopy implements Cloneable {
    private int id;
    // 省略set和get方法
   
    // 调用父类的clone完成对象的复制
    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}

public class DeepCopy  {
    public static void main(String[] args) {
        CarDeepCopy carDeepCopy1 = new CarDeepCopy(1);
        // 通过clone方法将carDeepCopy1做备份
        CarDeepCopy carDeepCopy2 = null;
        try {
            // clone之后，系统会开辟新的空间存放与1相同的内容，2执行这块内存。
            carDeepCopy2 = (CarDeepCopy) carDeepCopy1.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }

        ArrayList<CarDeepCopy> carDeepCopies1 = new ArrayList<>();
        ArrayList<CarDeepCopy> carDeepCopies2 = new ArrayList<>();
        carDeepCopies1.add(carDeepCopy1);
        carDeepCopies2.add(carDeepCopy2);
        carDeepCopies1.get(0).setId(2);
        System.out.println(carDeepCopies2.get(0).getId());
    }

}// output : 1
```



## 五、 使用迭代器访问线性表
尽量不使用循环的方式遍历 ArrayList 或者 Linkedlist，推荐使用迭代器。优点：不论待访问的集合类型是什么，或者存储着什么类型的对象，迭代器都可以使用一种方式进行遍历；
```java
public class IteratorDemo {
    public static void main(String[] args) {
        ArrayList<String> strings = new ArrayList<>();
        for (int i = 0; i < 3; i++) {
            strings.add(Integer.valueOf(i).toString());
        }
        Iterator<String> iterator = strings.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }
}// output: 0 1 2
```

**在迭代器遍历的时候，不能同时修改待遍历的集合对象**
解决方法：
- 法一：使用 CopyOnWriteArrayList 代替 ArrayList；
- 法二：不使用迭代器，使用 for 循环；


## 六、 Hash
**正常情况：**
在Hash表中 ， 存放在其中的数据和它的存储位置是用Hash函数关联的。
存放一个数据，首先通过 Hash 函数计算 Hash 值，然后将数据放入索引号为对应 Hash 值的位置；这样如果要查找某个数据，只要 计算器 Hash 值，然后到对应索引位置拿值即可；

**针对 Hash 值冲突解决：**
如果不同数据经过 Hash 函数计算之后的 Hash 值相同，这采用“链地址法”，即将所有 Hash 值相同的对象建立一个同义词链表。
示例：放入 y 元素之后，发现该位置已经被 X 占用了，那么就会在 X 之后新建一个链表结点放入 Y，如果要查找 Y，首先计算之后发现该位置不是 Y，就会沿着链表依次查找得到。




### （一） HashMap 存入自定义类时候，必须重写 equals 方法和 HashCode 方法
如果大家要在 HashMap的 ＂键” 部分存放自定义的对象， 一定要在这个对象中用自己的equals和 hashCode 方法来覆盖Object中的同名方法。
```java
class KeyValue{
    // 重写的hashCode中的成员变量以及get方法返回值☆☆☆必须使用包装类
    private Integer id;
    public Integer getId(){
      return id;
    }
    // 省略构造方法、set和get方法

    @Override
    public boolean equals(Object o) {
        if (o == null || !(o instanceof KeyValue)){
            return false;
        }else {
            return this.getId().equals(((KeyValue) o).getId());
        }
    }

    @Override
    public int hashCode() {
        return id.hashCode();
    }
    
}

public class WithoutHashCode {
    public static void main(String[] args) {
        KeyValue keyValue1 = new KeyValue(1);
        KeyValue keyValue2 = new KeyValue(1);
        HashMap<KeyValue, String> keyValueStringHashMap = new HashMap<>();
        keyValueStringHashMap.put(keyValue1, "id is 1");
        System.out.println(keyValueStringHashMap.get(keyValue2));
    }
}// 返回值为：null
// 返回值为：id is 1
```
**如果不写 hashCode 和 equal 方法：**
当向HashMap中放入 keyValue1 时， 首先会调用 KeyValue 类的 hashCode方法计算它的 hash值， 随后把 keyValue1 放入hash值所指引的 内存位置。因为没有定义 hashCode 方法，会调用 Object 类的 hashCode 方法，其返回的是 keyValue1 的内存地址；
当调用 get 方法时候，会再次调用 hashCode 方法，返回 KeyValue2 的内存地址，根据得到的 hash 找到 KeyValue2 值，因为没有赋值所以为 null；

**写了 hashCode 和 equal 方法之后：**
这里返回的是 id 的 hashCode，因为 id 一样，所以有返回值;

**建议使用迭代器遍历 HashMap 键值对**
```java
public class HashMapIterator {
    public static void main(String[] args) {
        // 使用两个HashMap存放数据表和文件中的学生信息
        HashMap<Integer, String> stringStringHashMap1 = new HashMap<>();
        HashMap<Integer, String> stringStringHashMap2 = new HashMap<>();

        // 模拟从数据表和文件中读取学生信息并插入HashMap
        stringStringHashMap1.put(1, "A1");
        stringStringHashMap1.put(2, "A1");
        stringStringHashMap1.put(3, "A1");
        stringStringHashMap2.put(2, "A2");
        stringStringHashMap2.put(3, "A1");
        stringStringHashMap2.put(4, "A2");

        Integer idInDB = null;
        String classNameInDB = null;
        String idInFile = null;
        String classNameInFile = null;

        // 通过iterator遍历stringstringHashMap1
        Iterator<Entry<Integer, String>> iterator = stringStringHashMap1.entrySet().iterator();

        while (iterator.hasNext()) {
            Map.Entry<Integer, String> entry = (Map.Entry<Integer, String>) iterator.next();
            // 得到键值对
            idInDB = entry.getKey();
            classNameInDB = entry.getValue();

            // 如果存在于数据表但是不存在文件中，则删除
            if (!stringStringHashMap2.containsKey(idInDB)) {
                System.out.println("删除id为：" + idInDB);
            }else {
                // 如果两边都存在，则比较文件中数据和数据表里面的班级名
                classNameInFile = stringStringHashMap2.get(idInDB);
                // 如果不一致，则用文件里的班级名更新数据表
                if (!classNameInFile.equals(classNameInDB)){
                    System.out.println("更新id为：" + idInDB);
                }
            }
        }
    }
}
//output: 
// 删除id为：1
//更新id为：2

```


**总结：使用功能 iterator 遍历 HashMap 步骤**
- 初始化迭代器，迭代器中 Entry 的泛型设置和待访问的 HashMap 一致；
- 用迭代器的hasNext方法来判断是否有下一个元素；
- 通过迭代器的next方法得到下—个元素， 并用**entry对象**来接收。注意 ， 这里Map.Entry中定义的泛型要与待谝历的HashMap相—致。
- 通过entry.getKey和entry.getValue方法来得到 其中 —个元素的键和值;

```java
 Iterator<Entry<Integer, String>> iterator = stringStringHashMap1.entrySet().iterator();

        while (iterator.hasNext()) {
            Map.Entry<Integer, String> entry = (Map.Entry<Integer, String>) iterator.next();
            // 得到键值对
            idInDB = entry.getKey();
            classNameInDB = entry.getValue();
```

**综合比较 HashMap、HashTable、HashSet 对象**
- 三个对象都是基于 Hash 表实现；
- 在遍历时， 它们都是乱序的 ， 即输出的结果未必和输入顺序一致， 原因是在**其中存储元素的位置是和元素的值有一定的关联关系（通过hashCode 方法关联）的， 而不是顺序插入的** 。如果要保证遍历的顺序， 可以使用LinkedHashSet或LinkedHashMap等，但是实际使用不多。
- HashMap、 HashTable是键值对的集合， 而HashSet是线性表类的集合 。
- 为了保证HashSet中 自定义对象的唯—性 ， 必须要在自定义类中重写equals和 hashCode方法。 如果在HashMap和HashTable的键部分存放的也是 自定义的类， 那么也要重写其中的 equals和hashCode方法。
- HashMap和HashTable的区别。 HashMap是线程不 安全的 ， 而HashTable是线程安全的；所以 HashMap可以是轻量级的（对应的 HashTable就是重量级的） 。这样， 在单线程清况下， HashMap的性能要优于HashTable。此外， HashTable 不允许将null值作为键 ， 而HashMap可以 ， 但是大家尽量不要在HashMap中使用null作为键 。


## 七、 Collections 类中常见操控集合的方法

### （一）通过 sort 方法对集合进行排序
**主要针对没有自动排序功能的集合**，像 TreeSet 会自动调用 compareTo 方法对放入的对象进行排序，当然可以重写 compareTo 方法。
**一共两种方案：**
- 方案一：在实体类中遵循 compareable 接口，并且实现 compareTo 方法，然后调用 Collections.sort()方法；
- 方案二：
这里以 ArrayList 为例，其不支持自动排序。
```java
public class SortForList {
    public static void main(String[] args) {
        // 方法一：实体类中实现compareTo方法，然后调用Collections.sort();方法
        // 这里使用的实体类是之前已经实现compareTo方法的student类
        Student student1 = new Student(1);
        Student student2 = new Student(2);

        ArrayList<Student> students = new ArrayList<>();
        students.add(student1);
        students.add(student2);

        Collections.sort(students);

        // 方法二：不需要在实体类中定义compareTo方法，以及实现compareable接口
        // 直接在这里定义Collections.sort方法
        Collections.sort(students, new Comparator<Student> () {
            @Override
            public int compare(Student student1, Student student2) {
                if (student1.getId() == student2.getId()){
                    return 0;
                }else {
                    return student1.getId() > student2.getId() ? 1 : -1;
                }
            }
        });

        // 通过迭代器遍历ArrayList集合
        Iterator<Student> iterator = students.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next().getId());
        }
    }
}
//output: 1  2 
```

### （二）将线程由不安全变成安全

Arraylist、LinkedList和HashMap昙有线程不安全的因素， 而HashTable是线程安全的。若既要用到Arraylist (或Linkedlist)的特性， 又要保证它 是线程安全的，这时就可以用到Collections中的synchronizedXxx方法。

- 通过synchronizedList方法， 可以把一个List (如Arraylist 或Linkedlist)包装成线程安全的；代码为：`Collections.synchronizedList(stuList);`
- synchronizedSet方法,同理；
- synchronizedMap方法，同理；


## 八、泛型

可以在定义集合时候设置对泛型的约束，也可以在定义类和方法时候设置；
下面是在定义类和方法的时候设置泛型，这样 new 对象的时候可以执行类型；
```java
// 定义类时直接加上泛型
public class GenericClass<T> {
    private List<T> productList;

    public List<T> getProductList() {
        return productList;
    }

    public void setProductList(List<T> productList) {
        this.productList = productList;
    }

    // 构造函数
    public GenericClass() {
        productList = new ArrayList<T>();
    }

    // 添加元素的方法，参数类型为T
    void addItem(T item){
        productList.add(item);
    }

    // 打印所有对象
    public void printAllItems(){
        Iterator<T> iterator = productList.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next().toString());
        }
    }

    public static void main(String[] args) {
        GenericClass genericClass = new GenericClass<String>();
        genericClass.addItem("hello");
        genericClass.addItem("world");
        
        GenericClass<Integer> integerGenericClass = new GenericClass<>();
        integerGenericClass.addItem(1);
        integerGenericClass.addItem(2);

        genericClass.printAllItems();
        integerGenericClass.printAllItems();
    }
}

```

### （一）泛型的继承和通配符
在定义泛型时 ， 可以通过extends来限定泛型类型的上限， 也可以通过super来限定其下限，List<? super Father> dest。就表示dest存放的对象应当 “以Father为子类 ” ；换句话说， 在dest 中，可以存放任何子类是Father类的对象 。
—般从List<? extends Father> src的集合中读取元素（即可以遍历它，也可以使用 get） ， 而向List<? super Father> dest的集合中写元素（可以使用 add）,**但是他们都不能反过来用，即 extend 不能使用 add,super 不能使用 get**



# 章四：异常处理和 IO 操作


## 一、异常处理的常规知识点

- 建议的异常处理定式：
```java
try{
        // 需要监控的可能会抛出异常的代码块
    }catch (Exception e) {
        // 出现异常后的代码块
    }finally {
        // 回收资源操作
    }
```

**只有在 catch 中执行：system.exit(0)时候，finally 里面的语句才不会执行。**

- 运行期异常类：
SQLException、IOException、RuntimeException 等等不会强制要求使用 try 捕获，但是自己要记得添加；

### （一）throw、throws

- 如果在某个函数的声明 位置出现了用 throws 报出异常的清况， 那么就需要用 try ... catch 代码块来包含调用的代码， 否则也会出现语法错误。
- throws 出现在声明方法的位置，throw 出现在函数体中；

### （二）finally 子句
**可以回收只在 try-catch 中遇到的，以后不再使用的对象；**
 - 如果在 try... catch 部分用 Connection 对象连接了数据库，而且在后继部分不会再用到这个连接对象， 那么—定要在 finally 从句中关闭该连接对象；
 - 如果在try... catch部分用到了一些 IO 对象进行了读写操作，那么也—定要在 finally中关闭这些 IO 对象；
 - 如果在 try... catch 部分用到了 ArrayList、LinkedList、 HashMap 等集合对象，而且这些对象之后不会再被用到，那么在 finally中建议通过调用 clear 方法来清空这些集合；
 - 在 try... catch 语句中有—个对象 obj 指向了—块比较大的内存空间，而且之后不会再被用到，那么在finally从句中建议写上 obj = null;


### （三）子类方法中不应该抛出比父类范围更广的异常
同样：如果父类的方法中没有抛出异常，当子类覆盖父类的方法时候，也不应该抛出异常；
这是根据设计原则，基类应该充分考虑运行环境的险恶程度；
```java
class Base{
    void f() throws SQLException {
    }
}
class Child extends Base{
    // 当覆盖了父类的 f 方法之后，并且抛出的异常范围比父类大，报错
    void f() throws Exception {
    }
}
```


### （四）异常处理的使用要点
- 尽量用 try... catch ... finally 的语句来处理异常， 在 finally 中应当尽可能地回收内存资源。
- 尽量减少用 try 监控的代码块。
- 先用专业的异常来处理， 最后用 Exception 异常来处理；（针对多个 catch 时候，范围最大的放在最后）；
- 在 catch 从句中， 不要只简单地抛出异常， 要尽可能地处理异常。即除了使用 `e.printStackTrace();` 抛出异常信息之外，尽量给出异常的处理方法；
- 出现异常后，应尽量保证项目不会终止，把异常造成的影响降到最低。即不同的业务使用不同的 try。


## 二、常见的 IO 读写操作
**这部分偏重于实践而不是类语法；**
- Demo1：实现在指定路径下寻找是否有文件拓展名为 .csv 的文件
demo：注意：该程序中文件夹中的层次不能太深，不宜超过 5 层；
```java
package chapter3;
import	java.io.File;

/** 实现在指定路径下寻找是否有文件拓展名为 .csv 的文件
 * @author GJXAIOU
 * @create 2019-08-18-20:02
 */
public class VisitFolder {
    static void getCSVInFloder(String filePath){
        File folderName = new File(filePath);
        File[] files = folderName.listFiles();

        if (files == null || files.length == 0) {
            return ;
        }
        String fileName = null;
        // 遍历整个文件夹
        for (File file : files) {
            // 如果是文件夹，则递归调用
            if (file.isDirectory()) {
                getCSVInFloder(file.getAbsolutePath());
            }else {
                // 如果是文件，判断文件后缀名是否为 .csv
                fileName = file.getName();
                if (fileName.substring(fileName.lastIndexOf(".") + 1).equals("csv")){
                    System.out.println(file.getAbsolutePath());
                }
            }
        }
    }

    public static void main(String[] args) {
        getCSVInFloder("C:/User/");
    }
}

```

- Demo2：读完 C:/1.csv 之后将其移到 history 目录中
```java
/** 实现复制文件的功能:这是使用这个 buffer 缓冲区一次读写，一般用于小文件
 * @author GJXAIOU
 * @create 2019-08-18-20:18
 */
public class CopyFile {
    public void fileCopy(String src, String dst){
        InputStream inputStream = null;
        OutputStream outputStream = null;

        // 创建输入输出流对象
        try {
            inputStream = new FileInputStream(src);
            outputStream = new FileOutputStream(dst);

            // 获取文件长度，并以此创建缓存
            int fileLength = inputStream.available();
            byte[] buffer = new byte[fileLength];

            // 读取文件，将文件内容放入 buffer 数组
            inputStream.read(buffer);
            // 将buffer数组中的数据写到目标文件中
            outputStream.write(buffer);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
            throw new RuntimeException("文件未找到");
        } catch (IOException e) {
            e.printStackTrace();
            throw new RuntimeException("文件长度为 0，请先写入内容；");
        }finally {
            if (inputStream != null) {
                try {
                    inputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (outputStream != null) {
                try {
                    outputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}

```


### （一）默认输入输出设备与重定向

- Demo1: 将键盘输入的内容输出到显示屏( idea 中相当于在 Console 输出 )
```java
/**将键盘输入的内容输出到显示屏
 * @author GJXAIOU
 * @create 2019-08-18-20:46
 */
public class IOInOutExample {
    public static void main(String[] args) {
        // 从 system.in 即键盘读内容，并放入缓存 bufferReader
        BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(System.in));
        String read = null;

        try {
            // 从缓存里面读取数据
            read = bufferedReader.readLine();
        } catch (IOException e) {
            e.printStackTrace();
        }
        System.out.println("output : " + read);
    }
}
```

- Demo2 ：  输出重定向：将输出至文件
```java
/** 输出重定向：将输出至文件
 * @author GJXAIOU
 * @create 2019-08-18-20:53
 */
public class IORedirectOutput {
    public static void main(String[] args) {
        FileOutputStream fos = null;
        BufferedOutputStream bos = null;
        PrintStream ps = null;
        // fos 对象中包含了往文件写的内容
        try {
            // fos 对象包含了往文件写的内容
            fos = new FileOutputStream("c:/out.txt");
            // bos 对象包含了缓存向外写的内容
            // 将 bos 内存缓冲区里存的内容放入 fos 的 FileOutputStream 类型的对象
            bos = new BufferedOutputStream(fos);
            ps = new PrintStream(bos);
            
            // 设置重定向
            System.setOut(ps);
            
            // 这里向文件输出
            System.out.println("redirect to out.txt");
            
            // 强制将缓存中内容输出
            ps.flush();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }finally {
            ps.close();
            if (fos != null) {
                try {
                    fos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (bos != null) {
                try {
                    bos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }

        }

    }
}
// output: out.txt文件中有："redirect to out.txt"
```

-  Demo3 ：输入重定向：将文件中读取的内容输出到控制台上
```java
package chapter4;
import java.io.*;
import java.io.BufferedOutputStream;

/**
 * @author GJXAIOU
 * @create 2019-08-18-21:24
 */
public class IORedirectInput {
    public static void main(String[] args) {
        BufferedInputStream bufferedInputStream = null;
        DataInputStream dataInputStream = null;
        String tmp = null;

        try {
            // 将文件中读到的内容放入 bufferedInputStream 这个内存缓冲区
            bufferedInputStream = new BufferedInputStream(new FileInputStream("c:/in.txt"));
            // 设置重定向
            System.setIn(bufferedInputStream);

            // 从重定向的输入（文件）中读取内容，并逐行输出
            dataInputStream = new DataInputStream(System.in);
            while ((tmp = dataInputStream.readLine())!= null){
                System.out.println(tmp);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }catch (IOException e) {
            e.printStackTrace();
        }finally {
            try {
                bufferedInputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                dataInputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}

```

### （二）生成和解开压缩文件

- 压缩文件
```java
public static void main(String[] args) {
		// 设置压缩后和带压缩的文件名
		String zipFile = "c:\\redirect.zip";
		String file1 = "c:\\redirect.txt";
		int zipRes = -1;
		
		 FileOutputStream fout = null;
		 ZipOutputStream zout = null;
		 BufferedOutputStream bout = null;			
		 FileInputStream fisOne = null;
		 BufferedInputStream bisOne = null;
		
		try 
		{
			// 定义 IO 对象
			// 用 fout 的 FileOutputStream 类型的对象指向了待生成的 zip 文件，把fout的内容输出到zip文件中
            fout = new FileOutputStream(zipFile);
			zout = new ZipOutputStream(fout);
			bout = new BufferedOutputStream(zout);			
			fisOne = new FileInputStream(file1);
			bisOne = new BufferedInputStream(fisOne);
			zout.putNextEntry(new ZipEntry("redirect.txt"));
			// 逐行读文件，把读到的内容添加到压缩流中
			while ((zipRes = bisOne.read()) != -1) {
				bout.write(zipRes);
			}
			// 强制输出
			// 对缓冲区进行操作时,在合理的位置需要加上 flush 强制地输出缓冲区的内容 ，否则输出的内容可能与预期的不一致
			bout.flush();
		}
		catch (IOException e) {
			e.printStackTrace();
		}
		finally
		{
			try {
				bisOne.close();
				fout.close();
	            zout.close();			
	            fisOne.close();
	            bout.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}
```

- 解压文件
```java
public static void main(String[] args) {
		int cont;
		FileInputStream fin = null;
		ZipInputStream zin = null;
		ZipEntry ze = null;
		
		try 
		{
			fin = new FileInputStream("c:\\redirect.zip");
			zin = new ZipInputStream(new BufferedInputStream(fin));
			
			while ((ze = zin.getNextEntry()) != null) 
			{
				System.out.println("file name is:" + ze);
				while ((cont = zin.read()) != -1)
				{
					System.out.write(cont);
				}
				System.out.println();
			}
		} 
		catch (FileNotFoundException e){
			e.printStackTrace();
		} 
		catch (IOException e){
			e.printStackTrace();
		}
		finally
		{
			try {
				fin.close();
				zin.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}
```


###  总结

- "流对象”是处理所有 IO 问题的载体，它不仅封装了用于保存输入或输出信息的缓冲空间， 更包含了针对这个缓冲空间进行操作的方法；

- 从应用角度来看，Java流对象分为输入流(lnputStream) 和输出流(OutputStream)两类。 它们是处理所有输入输出工作的 两个基类， 其中分别包含了 read和write方法。
具体来讲， lnputStream有FilelnputStream和BufferlnputStream两个针对文件 和内存缓冲进行读操作的子类，而OutputStream也 对应 地 有FileOutputStream和BufferOutputStream。

- IO 流的标准输入 和输出设备分别是键盘和显示屏 ， 此外还可以通过System setOut 和 System.setln方法进行重定向 。
重定向 时 ， 注意要指定重定向后 的输入（或输出）设备， 如可以通过system.setln方法的参数指定输入；原是文件 ， 也可以通过system.setOut 的参数指定向文件输出 。
(4)在OutputStream的基础上 ， Java IO 类库还提供了功能更强大的 PrintStream, 通过它可以方便地输出各种类型的数据，而不是仅为 byte 型， 而PrintWriter 类则封装了 PrintStream的所有输出方法。



### 非阻塞性的 NIO 操作
**与性能优化和架构设计有关系**
前面介绍的 IO 操作其实是BIO(blocked IO, 阻塞性的 IO )操作， JDK1.4以上的版本提供了 —种新的NIO（UnBlocked IO），非阻塞性 IO。

![NIO和传统IO比较]($resource/NIO%E5%92%8C%E4%BC%A0%E7%BB%9FIO%E6%AF%94%E8%BE%83.png)


#### NIO 三个组件

Channel (通道） , Buffer (缓冲器）和Selector(选择器）是NIO最核心的三大组件；

诵道(Channel)与传统10中的流(Stream)类似，但通道是双向的，而Stream是单向的，输入流只负责输入，输出流只负责输出，**唯一能与通道交互的组件是缓冲器**，通过通道，可以从缓冲器中读写数据；同时可以通过选择器来管理多个通道的读写操作；
 P140 -  P152 未看










