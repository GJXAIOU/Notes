## TreeSet集合如何保证元素唯一


## TreeSet：

### 1.特点

TreeSet是用来排序的, 可以指定一个顺序, 对象存入之后会按照指定的顺序排列

### 2.使用方式

**a.自然顺序(Comparable)**

*   TreeSet类的add()方法中会把存入的对象提升为Comparable类型

*   调用对象的compareTo()方法和集合中的对象比较(当前存入的是谁,谁就会调用compareTo方法)

*   根据compareTo()方法返回的结果进行存储

**b.比较器顺序(Comparator)**

*   创建TreeSet的时候可以制定 一个Comparator

*   如果传入了Comparator的子类对象, 那么TreeSet就会按照比较器中的顺序排序

*   add()方法内部会自动调用Comparator接口中compare()方法排序

*   调用的对象(就是当前存入的对象)是compare方法的第一个参数,集合中的对象(已经添加进去的对象)是compare方法的第二个参数

**c.两种方式的区别**

*   TreeSet构造函数什么都不传, 默认按照类中Comparable的顺序(没有就报错ClassCastException)

*   TreeSet如果传入Comparator, 就优先按照Comparator

## 1\. TreeSet存储Integer类型的元素

```
package online.msym.set;
import java.util.Comparator;
import java.util.TreeSet;
import online.msym.bean.Person;
public class Demo3_TreeSet {
    /**
     * @param args
     * TreeSet集合是用来对象元素进行排序的,同样他也可以保证元素的唯一    
     */
    public static void main(String[] args) {
        demo1();        
    }
    public static void demo1() {
        TreeSet<Integer> ts = new TreeSet<>();
        ts.add(3);
        ts.add(1);
        ts.add(1);
        ts.add(2);
        ts.add(2);
        ts.add(3);
        ts.add(3);        
        System.out.println("TreeSet存储Integer类型的元素: " + ts);
    }
}
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/JfTPiahTHJhq7GdnjvDGx9OicYMEibmOn97nGicgUhziaRfia06fXkKYHILEs6UmHt4lUQ4xl2QTwiaiaFRfEiaoZcIm8ibg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 2\. TreeSet存储自定义对象

```
package online.msym.set;
import java.util.Comparator;
import java.util.TreeSet;
import online.msym.bean.Person;
public class Demo3_TreeSet {
    /**
     * @param args
     * TreeSet集合是用来对象元素进行排序的,同样他也可以保证元素的唯一
     * 当compareTo方法返回0的时候集合中只有一个元素
     * 当compareTo方法返回正数的时候集合会怎么存就怎么取
     * 当compareTo方法返回负数的时候集合会倒序存储
     */
    public static void main(String[] args) {        
        demo2();        
    }
    public static void demo2() {
        //因为TreeSet要对元素进行排序，那你排序的依据是什么，姓名还是年龄还是其它的，得告诉它，怎么告诉?
        //需要让Person类实现Comparable接口重写compareTo方法
        TreeSet<Person> ts = new TreeSet<>();
        ts.add(new Person("张三", 23));
        ts.add(new Person("李四", 13));
        ts.add(new Person("周七", 13));
        ts.add(new Person("王五", 43));
        ts.add(new Person("赵六", 33));

        System.out.println(ts);
    }

}
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/JfTPiahTHJhq7GdnjvDGx9OicYMEibmOn97hic8Jg4zenCPFq4X9Ungvb8NxhJCudr6hXRxeHIYYFLlLvIXTRFnsjw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

注意上面的输出，跟添加的顺序相反，那是因为compareTo方法的返回值

**Person实体类：**

```
package online.msym.bean;
//为了简化代码，这里没有hashCode和equals方法，用的话可以直接将上面的Person类中的hashCode和equals复制过来
public class Person implements Comparable<Person> {
    private String name;
    private int age;
    public Person() {
        super();

    }
    public Person(String name, int age) {
        super();
        this.name = name;
        this.age = age;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    @Override
    public String toString() {
        return "Person [name=" + name + ", age=" + age + "]\n";
    }        
    public int compareTo(Person o) {
        //return 0;
        //return 1;
        return -1
    }

}
```

## TreeSet保证元素唯一和自然排序的原理和图解

TreeSet保证元素唯一和自然排序的原理和图解,小的放左侧,大的放右侧

![](https://mmbiz.qpic.cn/mmbiz_jpg/JfTPiahTHJhq7GdnjvDGx9OicYMEibmOn97y2whcxBPMIpqLOibibnE0FKwgibr1fcOCaAFx3165wyiatZia0v6gPU5jWQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz_jpg/JfTPiahTHJhq7GdnjvDGx9OicYMEibmOn97oib4Lez3DomW6ia8BJCzBPI0oT2CmmQFsdPViaRAZ7slNJegK2pPJ0nibA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**按照Person类的年龄排序的话,Person类改写为:**

```
//注意使用此类时要生成空参有参构造,set和get方法,hashCode和equals方法
package online.msym.bean;
public class Person implements Comparable<Person> {
    private String name;
    private int age;    
    @Override
    //按照年龄排序
    public int compareTo(Person o) {
       int num = this.age - o.age; //按照年龄比较 return num;     //return num == 0 ? this.name.compareTo(o.name) : num;//姓名是比较的次要条件
    }    
}
```

**TreeSet保证元素唯一和比较器排序的原理:**

定义比较器是实现Comparator接口,重写compare方法和equals方法,但是由于所有的类默认继承Object,而Object中有equals方法,所以自定义比较器类时,不用重写equals方法,只需要重写compare方法

字符串长度比较器图解:

![](https://mmbiz.qpic.cn/mmbiz_jpg/JfTPiahTHJhq7GdnjvDGx9OicYMEibmOn97cCbTSfO84FIGlliaBcTBmibnhW7k5O2B6XFoBYick3kH9niatefC6yAUnQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

下面采用了两种方式：匿名的内部类的比较器对象，自定义的比较器对象，

```
package online.msym.test;
import java.util.Comparator;
import java.util.TreeSet;
import online.msym.bean.Person;
public class Demo3_TreeSet {
    public static void main(String[] args) {
        //demo1();
        demo2();
    }
    private static void demo2() {
        // 需求:将字符串按照长度排序, （利用匿名内部类对象, 长度从大到小, 长度相同按照字母倒序）
        TreeSet<String> ts = new TreeSet<>(new Comparator<String>() {
            @Override
            public int compare(String s1, String s2) {
                int num = s2.length() - s1.length(); // 长度为主要条件
                return num == 0 ? s2.compareTo(s1) : num; // 内容为次要条件
            }
        }); 
        ts.add("aaaaaaaa");
        ts.add("z");
        ts.add("wc");
        ts.add("nba");
        ts.add("cba");
        System.out.println(ts);
    }
    private static void demo1() {
        // 需求:将字符串按照长度排序，（传递一个自定义的比较器对象）
        TreeSet<String> ts = new TreeSet<>(new CompareByLen()); // Comparator c = new CompareByLen();
        ts.add("aaaaaaaa");
        ts.add("z");
        ts.add("wc");
        ts.add("nba");
        ts.add("cba");
        System.out.println(ts);
    }
}
class CompareByLen /* extends Object */implements Comparator<String> {//实现一个比较器类
    @Override
    public int compare(String s1, String s2) { // 按照字符串的长度比较
        int num = s1.length() - s2.length(); // 长度为主要条件
        return num == 0 ? s1.compareTo(s2) : num; // 内容为次要条件
    }
}
```