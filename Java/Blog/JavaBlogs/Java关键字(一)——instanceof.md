# Java关键字(一)——instanceof

[原文链接](https://www.cnblogs.com/ysocean/p/8486500.html)



　　instanceof 严格来说是Java中的一个双目运算符，用来测试一个对象是否为一个类的实例，用法为：



`boolean` `result = obj ``instanceof` `Class`

 |

　　其中 obj 为一个对象，Class 表示一个类或者一个接口，当 obj 为 Class 的对象，或者是其直接或间接子类，或者是其接口的实现类，结果result 都返回 true，否则返回false。

　　注意：编译器会检查 obj 是否能转换成右边的class类型，如果不能转换则直接报错，如果不能确定类型，则通过编译，具体看运行时定。

[回到顶部](https://www.cnblogs.com/ysocean/p/8486500.html#_labelTop)

### 1、obj 必须为引用类型，不能是基本类型

| 

1

2

3

 | 

`int` `i = ``0``;`

`System.out.println(i ``instanceof` `Integer);``//编译不通过`

`System.out.println(i ``instanceof` `Object);``//编译不通过`

 |

　　instanceof 运算符只能用作对象的判断。

[回到顶部](https://www.cnblogs.com/ysocean/p/8486500.html#_labelTop)

### 2、obj 为 null

| 

1

 | 

`System.out.println(``null` `instanceof` `Object);``//false`

 |

　　关于 null 类型的描述在官方文档：[https://docs.oracle.com/javase/specs/jls/se7/html/jls-4.html#jls-4.1](https://docs.oracle.com/javase/specs/jls/se7/html/jls-4.html#jls-4.1) 有一些介绍。一般我们知道Java分为两种数据类型，一种是基本数据类型，有八个分别是 byte  short  int  long  float  double  char boolean,一种是引用类型，包括类，接口，数组等等。而Java中还有一种特殊的 null 类型，该类型没有名字，所以不可能声明为 null 类型的变量或者转换为 null 类型，null 引用是 null 类型表达式唯一可能的值，null 引用也可以转换为任意引用类型。我们不需要对 null 类型有多深刻的了解，我们只需要知道 null 是可以成为任意引用类型的**特殊符号**。

　　在 [JavaSE规范](https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html#jls-15.20.2) 中对 instanceof 运算符的规定就是：如果 obj 为 null，那么将返回 false。

[回到顶部](https://www.cnblogs.com/ysocean/p/8486500.html#_labelTop)

### 3、obj 为 class 类的实例对象

| 

1

2

 | 

`Integer integer = ``new` `Integer(``1``);`

`System.out.println(integer ``instanceof`  `Integer);``//true`

 |

　　这没什么好说的，最普遍的一种用法。

[回到顶部](https://www.cnblogs.com/ysocean/p/8486500.html#_labelTop)

### 4、obj 为 class 接口的实现类

　　了解Java 集合的，我们知道集合中有个上层接口 List，其有个典型实现类 ArrayList

| 

1

2

 | 

`public` `class` `ArrayList<E> ``extends` `AbstractList<E>`

`implements` `List<E>, RandomAccess, Cloneable, java.io.Serializable`

 |

　　所以我们可以用 instanceof 运算符判断 某个对象是否是 List 接口的实现类，如果是返回 true，否则返回 false

| 

1

2

 | 

`ArrayList arrayList = ``new` `ArrayList();`

`System.out.println(arrayList ``instanceof` `List);``//true`

 |

　　或者反过来也是返回 true

| 

1

2

 | 

`List list = ``new` `ArrayList();`

`System.out.println(list ``instanceof` `ArrayList);``//true`

 |

[回到顶部](https://www.cnblogs.com/ysocean/p/8486500.html#_labelTop)

### 5、obj 为 class 类的直接或间接子类

　　我们新建一个父类 Person.class，然后在创建它的一个子类 Man.class

| 

1

2

3

 | 

`public` `class` `Person {`

`}`

 |

　　Man.class

| 

1

2

3

 | 

`public` `class` `Man ``extends` `Person{`

`}`

 |

　　测试：

| 

1

2

3

4

5

6

 | 

`Person p1 = ``new` `Person();`

`Person p2 = ``new` `Man();`

`Man m1 = ``new` `Man();`

`System.out.println(p1 ``instanceof` `Man);``//false`

`System.out.println(p2 ``instanceof` `Man);``//true`

`System.out.println(m1 ``instanceof` `Man);``//true`

 |

　　注意第一种情况， p1 instanceof Man ，Man 是 Person 的子类，Person 不是 Man 的子类，所以返回结果为 false。

[回到顶部](https://www.cnblogs.com/ysocean/p/8486500.html#_labelTop)

### 6、问题

　　前面我们说过**编译器会检查 obj 是否能转换成右边的class类型，如果不能转换则直接报错，如果不能确定类型，则通过编译，具体看运行时定。**

看如下几个例子：

| 

1

2

3

4

5

6

 | 

`Person p1 = ``new` `Person();`

`System.out.println(p1 ``instanceof` `String);``//编译报错`

`System.out.println(p1 ``instanceof` `List);``//false`

`System.out.println(p1 ``instanceof` `List<?>);``//false`

`System.out.println(p1 ``instanceof` `List<Person>);``//编译报错`

 |

　　按照我们上面的说法，这里就存在问题了，Person 的对象 p1 很明显不能转换为 String 对象，那么自然 Person 的对象 p1 instanceof String 不能通过编译，但为什么 p1 instanceof List 却能通过编译呢？而 instanceof List<Person> 又不能通过编译了？

[回到顶部](https://www.cnblogs.com/ysocean/p/8486500.html#_labelTop)

### 7、深究原理

　　我们可以看Java语言规范Java SE 8 版：[https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html#jls-15.20.2](https://docs.oracle.com/javase/specs/jls/se8/html/jls-15.html#jls-15.20.2)

![](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180302000448613-26394231.png)

 　　如果用伪代码描述：

| 

1

2

3

4

5

6

7

8

9

10

11

 | 

`boolean` `result;`

`if` `(obj == ``null``) {`

`result = ``false``;`

`} ``else` `{`

`try` `{`

`T temp = (T) obj; ``// checkcast`

`result = ``true``;`

`} ``catch` `(ClassCastException e) {`

`result = ``false``;`

`}`

`}`

 |

　　也就是说有表达式 obj instanceof T，instanceof 运算符的 obj 操作数的类型必须是引用类型或空类型; 否则，会发生编译时错误。 

　　如果 obj 强制转换为 T 时发生编译错误，则关系表达式的 instanceof 同样会产生编译时错误。 在这种情况下，表达式实例的结果永远为false。

　　在运行时，如果 T 的值不为null，并且 obj 可以转换为 T 而不引发ClassCastException，则instanceof运算符的结果为true。 否则结果是错误的

　　简单来说就是：**如果 obj 不为 null 并且 (T) obj 不抛 ClassCastException 异常则该表达式值为 true ，否则值为 false 。**

所以对于上面提出的问题就很好理解了，为什么 p1 instanceof String 编译报错，因为(String)p1 是不能通过编译的，而 (List)p1 可以通过编译。

[回到顶部](https://www.cnblogs.com/ysocean/p/8486500.html#_labelTop)

### 8、instanceof 的实现策略

　　JavaSE 8 instanceof 的实现算法：[https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.instanceof](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html#jvms-6.5.instanceof)

　　![](https://images2018.cnblogs.com/blog/1120165/201803/1120165-20180302002919162-2045599504.png)

　　1、obj如果为null，则返回false；否则设S为obj的类型对象，剩下的问题就是检查S是否为T的子类型；

　　2、如果S == T，则返回true；

　　3、接下来分为3种情况，之所以要分情况是因为instanceof要做的是“子类型检查”，而Java语言的类型系统里数组类型、接口类型与普通类类型三者的子类型规定都不一样，必须分开来讨论。

　　①、S是数组类型：如果 T 是一个类类型，那么T必须是Object；如果 T 是接口类型，那么 T 必须是由数组实现的接口之一；

　　②、接口类型：对接口类型的 instanceof 就直接遍历S里记录的它所实现的接口，看有没有跟T一致的；

　　③、类类型：对类类型的 instanceof 则是遍历S的super链（继承链）一直到Object，看有没有跟T一致的。遍历类的super链意味着这个算法的性能会受类的继承深度的影响。

参考链接：https://www.zhihu.com/question/21574535