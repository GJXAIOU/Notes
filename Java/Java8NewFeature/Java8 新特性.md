# Java 8 新特性

[TOC]

## 一、Lambda 表达式与函数式接口

### (一) Lambda 表达式

在编程语言中，**Lambda 是用于表示匿名函数或者闭包的运算符**。其实现了将函数作为参数传递给一个方法以及声明返回一个函数的方法，即实现了「函数式编程」。

#### 1.Lambda 表达式概要和作用

- Lambda 表达式是一种匿名函数，它是没有声明的方法，即没有访问修饰符、返回值声明和名字。

- 在 python、JavaScript 中，Lambda 表达式的类型是函数。**但在 Java 中，Lambda 表达式是对象**，他们必须依附于一类特别的对象类型——函数式接口
- **Lambda 表达式主要用于传递行为**，而不仅仅是值。从而提升抽象层次，使得 API 重用性更好。

#### 2.Lambda 基本语法

```java
// 概述结构
(argument) -> {body}
// 两种主要的示例结构
(arg1, arg2...) -> { body }   // 省略类型声明
(type1 arg1, type2 arg2...) -> { body }  // 补充上完整的类型声明
```

#### 3.Lambda 结构

- 一个 Lambda 表达式可以有零个或多个参数，其主体可以包含零条或者多条语句。
- 参数的类型既可以明确声明，也可以根据上下文来推断。例如：`(int a)` 与 `(a)` 效果相同。
- 所有参数需包含在圆括号内，参数之间用逗号相隔。例如：`(a, b)` 或 `(int a, int b)` 或 `(String a, int b, float c)` 

- 空圆括号代表参数集为空。例如：`() -> 42`

- 当只有一个参数，且其类型可推导时，圆括号 `()` 可省略。例如：`a -> return a * a;`
- 如果 Lambda 表达式的主体只有一条语句，花括号 `{}` 可省略。匿名函数的返回类型与该主体表达式一致。
- 如果 Lambda 表达式的主体包含一条以上语句，则表达式必须包含在花括号 `{}` 中（形成代码块）。匿名函数的返回类型与代码块的返回类型一致，若没有返回则为空。

#### 4.测试代码

- 从匿名内部类到 Lambda

  ```java
  package com.gjxaiou.jdk8.lambdaAndFunctionInterface;
  
  import javax.swing.*;
  import java.awt.event.ActionEvent;
  import java.awt.event.ActionListener;
  
  /**
   * 功能需求： 对一个按钮 button 注册一个事件监听器，该监听器的作用是当按钮发生某个动作则监听器的特定方法会被调用
   */
  public class SwingTest {
  
      public static void main(String[] args) {
          JFrame jframe = new JFrame("My JFrame");
          JButton jButton = new JButton("My JButton");
  
          /**
           * 实现方式一：原始方式：使用匿名内部类
           */
          jButton.addActionListener(new ActionListener() {
              @Override
              public void actionPerformed(ActionEvent e) {
                  System.out.println("Button Pressed!");
                  System.out.println("hello world");
                  System.out.println("executed");
              }
          });
  
          /**
           * 实现方式二：Lambda 表达式
           */
          // 这里 ActionEvent 如果不写，Java 编译可以进行类型推断进行推断处理。
          jButton.addActionListener((ActionEvent event) -> {
              System.out.println("Button Pressed!");
              System.out.println("hello world");
              System.out.println("executed");
          });
  
          // 将按钮 button 添加到 frame
          jframe.add(jButton);
          jframe.pack();
          jframe.setVisible(true);
          jframe.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
      }
  }
  
  // ActionListener 接口源码
  // public interface ActionListener extends EventListener {
  //    public void actionPerformed(ActionEvent e);
  // }
  ```

- 在集合遍历中的使用 Lambda

  ```java
  package com.gjxaiou.jdk8;
  
  import java.util.Arrays;
  import java.util.List;
  import java.util.function.Consumer;
  
  public class CollectionTest {
  
      public static void main(String[] args) {
          List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8);
  
          // 方式一：遍历集合方式
          for (int i = 0; i < list.size(); ++i) {
              System.out.println(list.get(i));
          }
          System.out.println("----------");
  
          // 方式二：增强 for 循环
          for (Integer i : list) {
              System.out.println(i);
          }
          System.out.println("----------");
  
          // 方式三：使用 Consumer 函数式接口
          // forEach() 是 Iterable 接口的默认实现方法，则其实现类就继承了该方法，List 是 Iterable 的一个实现类。【List 继承 Collection 接口，而 Collection 接口继承了 Iterable】
       // Consumer 函数式接口中的 JavaDoc 文档说明：接收另一个参数并且不返回结果值，可能会修改入参值。
          list.forEach(new Consumer<Integer>() {
              @Override
              public void accept(Integer integer) {
                  System.out.println(integer);
              }
          });
          System.out.println("----------");
  
          // 方式三改为使用 Lambda 表达式
          list.forEach(integer -> {
              System.out.println(integer);
          });
          System.out.println("----------");
          
          // 方式四：使用方法引用（method reference），因为函数式接口的实例可以通过 Lambda 表达式、方法引用、构造方法引用来创建。
          list.forEach(System.out::println);
      }
  }
  ```

- 验证 Lambda 表达式是对象

  ```java
  package com.gjxaiou.jdk8.lambdaAndFunctionInterface;
  
  // 首先提供一个函数式接口
  @FunctionalInterface
  interface MyInterface {
      void test();
  
      // 重写 Object 类中的方法，则该接口中仍然只有上面一个抽象方法，则仍然可以标注 @FunctionalInterface
      @Override
      String toString();
  }
  
  public class Test2 {
      public void myTest(MyInterface myInterface) {
          myInterface.test();
          System.out.println("-----------------");
      }
  
      public static void main(String[] args) {
          Test2 test2 = new Test2();
  
          // 调用方式：Lambda 表达式
          test2.myTest(() -> {
              System.out.println("this is myTest by Lambda 表达式");
          });
  
          // 上面等价于下面的，就是  MyInterface 的一个实现。
          // 后面的 Lambda 表达式赋值给了前面的对象引用 myInterface
          MyInterface myInterface = () -> {
              System.out.println("hello");
          };
  
          // 该 Lambda 表达式的类型
          System.out.println(myInterface.getClass());
          // 该 Lambda 表达式的父类型
          System.out.println("父类" + myInterface.getClass().getSuperclass());
          // 该 Lambda 表达式实现的接口
          System.out.println("实现以下接口：" + myInterface.getClass().getInterfaces()[0]);
      }
  }
  /** output:
   * this is myTest by Lambda 表达式
   * -----------------
   * class com.gjxaiou.jdk8.lambdaAndFunctionInterface.Test2$$Lambda$2/764977973
   * 父类class java.lang.Object
   * 实现以下接口：interface com.gjxaiou.jdk8.lambdaAndFunctionInterface.MyInterface
   */
  ```

- 验证 Lambda 表达式是对象方式二：

```java
package com.gjxaiou.jdk8.lambdaAndFunctionInterface;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.function.Function;

/**
 * 定义两个函数式接口
 */
@FunctionalInterface
interface TheInterface {
    void myMethod();
}

@FunctionalInterface
interface TheInterface2 {
    void myMethod2();
}


public class Test3 {
    public static void main(String[] args) {
        /**
         * 验证：Lambda 表达式是属于什么类型必须通过上下文信息来断定，这里的表达式：() -> {}; 需要前面的引用类型才能判断类型。
         */
        // 因为抽象方法不接受参数不返回值。
        TheInterface i1 = () -> {
        };
        System.out.println(i1.getClass().getInterfaces()[0]);

        TheInterface2 i2 = () -> {
        };
        System.out.println(i2.getClass().getInterfaces()[0]);

        /**
         * 示例二：创建线程方式，这里使用 Lambda 形式来实现 Runnable 接口的形式。
         * Runnable 接口也是一个函数式接口
         */
        new Thread(() -> System.out.println("hello world")).start();

        /**
         * 示例三：列表遍历修改
         */
        
        // 需求一：遍历列表然后将元素全部大写然后输出
        List<String> list = Arrays.asList("hello", "world", "hello world");
        // forEach 中接受一个 Consumer 接口，该接口接受一个参数并且不返回值。
        list.forEach(item -> System.out.println(item.toUpperCase()));


        // 需求二：遍历 list1 得到值然后添加到目标集合 list2 中，然后打印
        List<String> list2 = new ArrayList<>();
        list.forEach(item -> list2.add(item.toUpperCase()));
        list2.forEach(item -> System.out.println(item));

        // 需求二的语句简化：使用 stream 流的形式的两种写法
        list.stream().map(item -> item.toUpperCase()).forEach(item -> System.out.println(item));
        // map() 本身接收的是一个函数式接口 Function 的参数，该函数式接口的抽象方法为接收一个参数然后返回一个值，除了使用上面的 Lambda 表达式之外还可以使用下面的方法引用。
        list.stream().map(String::toUpperCase).forEach(item -> System.out.println(item));

        // 方法引用：Function 函数式接口
        // 因为 toUpperCase() 方法是一个实例方法，因此使用的时候肯定是由一个 String 对象来调用该方法。对于像 String::toUpperCase 这种
        // 类::实例方法 形式的方法引用，则该实例方法的第一个输入参数一定就是调用该方法的对象，这里就是 String 对象，正好对应 Lambda 表达式中的第一个参数（对应上面
        // Lambda 表达式写法就是 item，也就是 Function 的 String 类型来源）。
        Function<String, String> function = String::toUpperCase;
        System.out.println(function.getClass().getInterfaces()[0]);
    }
}
/** output:
 * interface com.gjxaiou.jdk8.lambdaAndFunctionInterface.TheInterface
 * interface com.gjxaiou.jdk8.lambdaAndFunctionInterface.TheInterface2
 * hello world
 * HELLO
 * WORLD
 * HELLO WORLD
 * HELLO
 * WORLD
 * HELLO WORLD
 * HELLO
 * WORLD
 * HELLO WORLD
 * HELLO
 * WORLD
 * HELLO WORLD
 * interface java.util.function.Function
 *
 * Process finished with exit code 0
 */
```

- Lambda 表达式在排序中使用 

  ```java
  package com.gjxaiou.jdk8.lambdaAndFunctionInterface;
  
  import java.util.Arrays;
  import java.util.Collections;
  import java.util.Comparator;
  import java.util.List;
  
  public class StringComparator {
  
      public static void main(String[] args) {
          List<String> names = Arrays.asList("zhangsan", "lisi", "wangwu", "zhaoliu");
  
          /**
           * 倒序排序方式一：匿名内部类
           */
          // Comparator 也是函数式接口，
          Collections.sort(names, new Comparator<String>() {
              @Override
              public int compare(String o1, String o2) {
                  return o2.compareTo(o1);
              }
          });
          System.out.println(names);
  
          /**
           * 方式二：Lambda 表达式
           * sort(list 列表，Comparator 函数式接口)，该接口接收两个参数，返回一个参数
           */
          Collections.sort(names, (o1, o2) -> {
              return o2.compareTo(o1);
          });
  
          // 方式二的简化版本
          Collections.sort(names, (o1, o2) -> o2.compareTo(o1));
  
          // 方式二的简化版本
          Collections.sort(names, Comparator.reverseOrder());
          System.out.println(names);
      }
  }
  /** output:
   * [zhaoliu, zhangsan, wangwu, lisi]
   * [zhaoliu, zhangsan, wangwu, lisi]
   */
  ```

### (二) 函数式接口：FunctionalInterface

接口里面有且只有一个抽象实例方法的时候可以在接口上面使用 `@FunctionInterface` 注解来标注这个接口，当然该接口中**可以拥有默认实现方法和静态方法**。

```java
@FunctionalInterface
public interface Demo {
    // 唯一的实例方法
    boolean hasDemo();

    // 可以允许有一个默认实现方法
    default boolean hasDemo2() {
        return false;
    }
}
```

**函数式接口的实例可以通过 Lambda 表达式、方法引用或者构造方法引用来创建**。例如在 `Runnable` 接口上面有 `@FunctionInterface` 标识，表示该接口是函数式接口，因此可以使用 Lambda 表达式方式来创建。

#### 函数式接口定义

- 如果一个接口只有一个**抽象方法**，那么该接口就是一个函数式接口。即是没有声明 `@FunctionalInterface` 注解，那么编译器依然会将该接口看作是函数式接口。

- 如果我们在某个接口上声明了 `@FunctionalInterface` 注解，那么编译器就会按照函数式接口的定义来要求该接口。即如果不满足下面两个条件就是报错。

    - 该元素是一个接口类型并且不是一个注解类型、枚举类型、 class 类型。
    - 该注解类型满足函数式接口的要求（即只能有一个抽象的成员方法）。

- **函数式接口中可以有：抽象方法（有且只有一个）、默认方法、静态方法、重写 Object 类的方法**。

    > 如果一个接口中声明了一个抽象方法（该方法重写了 Object 类中的一个 public 方法），则该接口的抽象方法的个数不会加一（即该抽象方法不算进抽象方法判断条件中的个数）。

    ```java
    @FunctionalInterface
    interface MyInterface {
        void test();
        // 重写 Object 类中的方法，则该接口中仍然只有上面一个抽象方法，则仍然可以标注 @FunctionalInterface
        @Override
        String toString();
    }
    ```

    原因：因为如果 `MyInterface` 接口有实现类，则该实现类一定有这里面两个方法的实现，因为 `java.lang.Object` 类是所有类的父类，所以该实现类会直接或者间接的继承 Object 类，所以就相当于将 `toString()` 方法继承过来了，即实现类 Object 类中的 `toString()` 方法，但是 `test()` 方法是要求子类进行自己实现的。函数式接口的定义是：该接口只有一个抽象方法，并且该方法由子类进行实现。

    测试使用方式：

    ```java
    package com.gjxaiou.jdk8;
    
    @FunctionalInterface
    interface MyInterface {
        void test();
        // 重写 Object 类中的方法，则该接口中仍然只有上面一个抽象方法，则仍然可以标注 @FunctionalInterface
        @Override
        String toString();
    }
    
    
    public class Test2 {
        public void myTest(MyInterface myInterface) {
            myInterface.test();
            System.out.println("-----------------");
        }
    
        public static void main(String[] args) {
            Test2 test2 = new Test2();
    
            // 调用方式一：匿名内部类
            test2.myTest(new MyInterface() {
                @Override
                public void test() {
                    System.out.println("this is myTest by 匿名内部类");
                }
            });
    
            // 调用方式二：Lambda 表达式
            // 因为函数式接口中只有唯一的抽象方法，所以 sout 即是对该方法的实现，() 表示没有参数，但是 () 得有。
            test2.myTest(() -> {
                System.out.println("this is myTest by Lambda 表达式");
            });
        }
    }
    ```

    输出结果：

    ```java
    this is myTest by 匿名内部�?
    -----------------
    this is myTest by Lambda 表达�?
    -----------------
    ```


#### 函数式接口：Function

```java
package java.util.function;

import java.util.Objects;

@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```

Function 函数式接口代表一个函数，该函数**接收一个参数并且返回一个结果**。

```java
public class Test3 {
    public static void main(String[] args) {      

        // 需求：遍历 list1 得到值然后添加到目标集合 list2 中，然后打印
        List<String> list = Arrays.asList("hello", "world", "hello world");
        List<String> list2 = new ArrayList<>();

        /*
		 * stream 的形式的两种写法
		*/
        // map() 本身接收的是一个函数式接口 Function 的参数，该函数式接口的抽象方法为接收一个参数然后返回一个值
        list.stream().map(item -> item.toUpperCase()).forEach(item -> System.out.println(item));

        // 除了使用上面的 Lambda 表达式之外还可以使用下面的方法引用。
        list.stream().map(String::toUpperCase).forEach(item -> System.out.println(item));

        // 方法引用，Function 函数式接口
        // 因为 toUpperCase() 方法是一个实例方法，因此使用的时候肯定是有一个 String 对象来调用该方法。对于像 String::toUpperCase 这种
        // 类::实例方法 形式的方法引用，则该实例方法的第一个输入参数一定就是调用该方法的对象，这里就是 String 对象，正好对应 Lambda 表达式中的第一个参数（对应上面
        // Lambda 表达式写法就是 item，也就是 Function 的 String 类型来源）。
        Function<String, String> function = String::toUpperCase;
        System.out.println(function.getClass().getInterfaces()[0]);
    }
}
```

**高阶函数**：如果一个函数接收一个函数作为参数，或者返回一个函数作为返回值，那么该函数就叫做高阶函数。

```java
package com.gjxaiou.jdk8.lambdaAndFunctionInterface;

import java.util.function.Function;

public class FunctionTest {

    // 接收一个 Function 类型的参数，该函数式接口中有唯一的抽象方法： R apply(T t);
    public int compute(int a, Function<Integer, Integer> function) {
        // apply() 这个动作行为是使用者进行传递的
        int result = function.apply(a);
        return result;
    }

    // Function 中输入一个整型，输出一个字符串
    public String convert(int a, Function<Integer, String> function) {
        return function.apply(a);
    }

    /**
     * 之前的实现方法，需要将每一个行为都提前定义好，然后调用。
     */
    public int method1(int a) {
        return 2 * a;
    }

    public int method2(int a) {
        return 5 + a;
    }


    public static void main(String[] args) {
        FunctionTest test = new FunctionTest();

        // 将一个行为：value -> {return 2 * value;} 作为参数进行传递（对应的就是 apply()），该行为中输入参数为 value，输出为： 2 * value
        System.out.println(test.compute(1, value -> {
            return 2 * value;
        }));
        System.out.println(test.compute(2, value -> 5 + value));

        System.out.println(test.convert(5, value -> value + "hello world"));

        System.out.println(test.method1(2));

        /**
         * 行为是调用的时候才确定的，当然可以先定义好 Lambda 表达式然后传递
         */
        Function<Integer, Integer> function = value -> value * 2;

        System.out.println(test.compute(4, function));
    }
}
```

**Function 函数式接口中两个默认方法讲解**

- `compose()`方法（先参数后当前对象）先里面后外面

    ```java
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }
    ```

    该方法对多个 Function 进行了组合，首先调用 before 这个 Function，就是相当于调用其 `apply()` 方法对输入进行处理，即 `before.apply(v)`，输出结果又传递给了当前对象的 `apply()` 方法，即：`apply(before.apply(v))`，从而形成两个 Function 的串联。当然可以多个 Function 形成串联调用。

- `andThen()` 方法（先当前对象后参数）

    ```java
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }
    ```

    顺序正好与上面相反，首先会对输入应用当前的 Function，然后对得到的结果引用参数的 Function。

    两个方法的具体测试如下：

```java
package com.gjxaiou.jdk8.lambdaAndFunctionInterface;

import java.util.function.Function;

public class FunctionTest2 {
    public static void main(String[] args) {
        FunctionTest2 test = new FunctionTest2();
        // 先应用后面，然后应用前面
        System.out.println(test.compute(1, value -> value + " hello", value -> value + " world"));
        // 先应用前面，然后应用后面
        System.out.println(test.compute2(2, value -> value + " hello ", value -> value + "world"));
    }

    // 使用 compose，先对输入参数应用 function2 的 apply(),然后将结果作为 function1 的 apply() 的输入
    public String compute(int a, Function<String, String> function1,
                          Function<Integer, String> function2) {
        return function1.compose(function2).apply(a);
    }

    // 使用 andThen
    public String compute2(int a, Function<Integer, String> function1,
                           Function<String, String> function2) {
        return function1.andThen(function2).apply(a);
    }
}
// output：
1 world hello
2 hello world
```

#### 3.函数式接口：BiFunction(Bidirectional Function)

该函数**输入两个参数，输出一个结果**。

```java
@FunctionalInterface
public interface BiFunction<T, U, R> {
    R apply(T t, U u);
}
```

- `andThen()` 方法

    ```java
    default <V> BiFunction<T, U, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t, U u) -> after.apply(apply(t, u));
    }
    ```

    流程为：先应用当前的 BiFunction 的 `apply()`，然后应用参数 after 的 `apply()`，针对当前的 BiFunction 需要接收两个参数，并且只返回一个值，返回的一个值是作为 after 的输入参数。

    **为什么没有 Compose**，因为 BiFunction 作为输入参数，应该先执行的，返回结果是一个值，这样在执行当前的 BiFunction 的时候没有足够的输入参数用于执行。

    方法使用测试如下：

    首先定义一个集合类：

    ```java
    package com.gjxaiou.jdk8;
    
    @Getter
    @Setter
    public class Person {
        private String username;
        private int age;
    
        public Person(String username, int age) {
            this.username = username;
            this.age = age;
        }
    }
    ```

    完整的测试程序

```java
package com.gjxaiou.jdk8.lambdaAndFunctionInterface;

public class BiFunctionTest {
    public static void main(String[] args) {
        BiFunctionTest test = new BiFunctionTest();

        /**
         * 实现加减操作
         */
        System.out.println(test.compute3(1, 2, (value1, value2) -> value1 + value2));
        System.out.println(test.compute3(1, 2, (value1, value2) -> value1 - value2));

        // 25，测试 andThen
        System.out.println(test.compute4(2, 3, (value1, value2) -> value1 + value2,
                value -> value * value));

        // 根据指定函数遍历列表
        Person person1 = new Person("zhangSan", 20);
        Person person2 = new Person("liSi", 30);
        Person person3 = new Person("wangWu", 40);
        List<Person> persons = Arrays.asList(person1, person2, person3);

        List<Person> personResult = test.getPersonsByAge(20, persons, (age, personList) ->
                personList.stream().filter(person -> person.getAge() <= age).collect(Collectors.toList())
        );

        personResult.forEach(person -> System.out.println(person.getAge() + person.getUsername()));
    }

    // 输入两个参数，输出一个结果
    public int compute3(int a, int b, BiFunction<Integer, Integer, Integer> biFunction) {
        return biFunction.apply(a, b);
    }

    // 首先应用于当前函数，然后将结果作用于参数的 Function
    public int compute4(int a, int b, BiFunction<Integer, Integer, Integer> biFunction,
                        Function<Integer, Integer> function) {
        return biFunction.andThen(function).apply(a, b);
    }

    public List<Person> getPersonsByAge(int age, List<Person> persons, BiFunction<Integer,
            List<Person>, List<Person>> biFunction) {
        return biFunction.apply(age, persons);
    }
}
/**
 * output:
 * 3
 * -1
 * 25
 * 20zhangSan
 */
```

#### 4.函数式接口：Predicate

predicate 就是判断给定参数是否符合某个条件。

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}
```

例如上面的 `Stream<T> filter(Predicate<? super T> predicate);`方法里面就是接受一个 Predicate 类型的参数类型，用来判断 filter() 方法里面的条件是真还是假，如果是假的就从 stream 流中过滤掉，如果为真则放入流中供后续操作使用。

```java
package com.shengsiyuan.jdk8;

import java.util.function.Predicate;

public class PredicateTest {

    public static void main(String[] args) {
        // 输入一个参数，返回一个 boolean 值
        Predicate<String> predicate = p -> p.length() > 5;
        System.out.println(predicate.test("hello1"));
    }
}
```

测试代码：

```java
package com.gjxaiou.jdk8.lambdaAndFunctionInterface;


import java.util.Arrays;
import java.util.List;
import java.util.function.Predicate;

/**
 * 定义一个集合，按照按照不同条件进行输出
 */
public class PredicateTest2 {

    public static void main(String[] args) {

        List<Integer> list = Arrays.asList(1, 2, 4, 5, 6);

        PredicateTest2 predicateTest2 = new PredicateTest2();

        predicateTest2.conditionFilter(list, item -> item % 2 == 0);
        predicateTest2.conditionFilter(list, item -> item % 2 != 0);
        predicateTest2.conditionFilter(list, item -> item > 5);
        predicateTest2.conditionFilter(list, item -> item < 3);
        // 打印集合中所有元素
        predicateTest2.conditionFilter(list, item -> true);
        predicateTest2.conditionFilter(list, item -> false);
    }

    public void conditionFilter(List<Integer> list, Predicate<Integer> predicate) {
        for (Integer integer : list) {
            if (predicate.test(integer)) {
                System.out.println(integer);
            }
        }
        System.out.println("---------------");
    }

    // 原始方式，每个判定条件都需要定义新的方法
    public void findAllEvens(List<Integer> list) {
        for (Integer integer : list) {
            if (integer % 2 == 0) {
                System.out.println(integer);
            }
        }
    }
}
// output：
2
4
6
---------------
1
5
---------------
6
---------------
1
2
---------------
1
2
4
5
6
---------------
---------------
```

该接口中其他主要方法分析：

- `and` 方法

    即是一种短路的逻辑与的计算，当前 Predicate 计算与参数的 Predicate ，返回值是一个复合的 Predicate。

    ```java
    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }
    ```

- `negate`  方法

    取反操作。

    ```java
    default Predicate<T> negate() {
        return (t) -> !test(t);
    }
    ```

- `or` 方法

    逻辑或的操作

    ```java
    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }
    ```

- `isEqual` 方法

    判断是否相等

    ```java
    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
            ? Objects::isNull
                : object -> targetRef.equals(object);
    }
    ```


测试代码：

```java
package com.shengsiyuan.jdk8;

import java.util.Arrays;
import java.util.Date;
import java.util.List;
import java.util.function.Predicate;

/**
 * @Author GJXAIOU
 * @Date 2020/10/24 14:41
 */
public class PredicateTest3 {

    public static void main(String[] args) {

        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        PredicateTest3 predicateTest2 = new PredicateTest3();

        predicateTest2.conditionFilter2(list, item -> item > 5, item -> item % 2 == 0);
        System.out.println("---------");

        System.out.println(predicateTest2.isEqual(new Date()).test(new Date()));
    }

    public void conditionFilter2(List<Integer> list, Predicate<Integer> predicate,
                                 Predicate<Integer> predicate2) {
        for (Integer integer : list) {
            if (predicate.and(predicate2).negate().test(integer)) {
                System.out.println(integer);
            }
        }
    }

    public Predicate<Date> isEqual(Object object) {
        return Predicate.isEqual(object);
    }
}
```



#### 5.函数式接口：Supplier

不接受任何参数并且返回一个结果

演示使用：

```java
package com.gjxaiou.jdk8.lambdaAndFunctionInterface;


import java.util.function.Supplier;

public class SupplierTest {

    public static void main(String[] args) {
        Supplier<String> supplier = () -> "hello world";
        System.out.println(supplier.get());
    }
}
// output：
hello world
```

一般可以用作工厂

```java
package com.gjxaiou.jdk8.lambdaAndFunctionInterface;

public class Student {

    private String name = "zhangsan";

    private int age = 20;
    
    // 省略有参、无参构造方法；get 和 set 方法
}
```

测试程序：通过 Supperlier

```java
package com.gjxaiou.jdk8.lambdaAndFunctionInterface;


import java.util.function.Supplier;

public class StudentTest {

    public static void main(String[] args) {
        // 获取 Student 对象
        Supplier<Student> supplier = () -> new Student();
        System.out.println(supplier.get().getName());

        System.out.println("-------");

        // 方式二：使用构造方法引用来简化
        Supplier<Student> supplier2 = Student::new;
        System.out.println(supplier2.get().getName());
    }
}
```



#### 6.函数式接口：BinaryOperator

是 BiFunction 的一种具体化，即输入是两个参数，输出是一个参数，**但是三个参数的类型是一样的**。使用其重写上面的计算方法如下：

```java
package com.shengsiyuan.jdk8;


import java.util.Comparator;
import java.util.function.BinaryOperator;

public class BinaryOperatorTest {

    public static void main(String[] args) {
        BinaryOperatorTest binaryOperatorTest = new BinaryOperatorTest();

        System.out.println(binaryOperatorTest.compute(1, 2, (a, b) -> a + b));
        System.out.println(binaryOperatorTest.compute(1, 2, (a, b) -> a - b));

        System.out.println("----------");

        // 可以定义不同的比较规则
        System.out.println(binaryOperatorTest.getShort("hello123", "world",
                (a, b) -> a.length() - b.length()));
        System.out.println(binaryOperatorTest.getShort("hello123", "world",
                (a, b) -> a.charAt(0) - b.charAt(0)));

    }

    public int compute(int a, int b, BinaryOperator<Integer> binaryOperator) {
        return binaryOperator.apply(a, b);
    }

    public String getShort(String a, String b, Comparator<String> comparator) {
        return BinaryOperator.maxBy(comparator).apply(a, b);
    }
}

```

## 二、Optional

为了解决：NPE：NullPointerException

根据该接口的 JavaDoc：是一个容器对象，可能包括或者不包括一个非空值（即里面可能为对象或者为空），可以通过 `isPresent()`判断是否存在，通过 `get()`获取。

**注意**：因为 Optional 并没有实现 Serializable 接口，所以不要将 Optional 作为方法参数传递或者在类中定义 Optional 类型的成员变量，**通常 Optional 只作为方法的返回值来避免 NPE**。

- 源码中构造方法分析：`empty()`、`of()`、`ofNullable()`

    ```java
    public final class Optional<T> {
    
        private static final Optional<?> EMPTY = new Optional<>();
    	// Optional 是一个容器，里面包含一个泛型 T 类型的值：value
        private final T value;
    
        private Optional() {
            this.value = null;
        }
    
        private Optional(T value) {
            this.value = Objects.requireNonNull(value);
        }
        
        /**
         * 使用 Optional 构造对象提供了下面三种工厂方法
         */
        
        // 方式一：通过该工厂方法，构造出一个容器里面为空的 Optional 对象
        public static<T> Optional<T> empty() {
            @SuppressWarnings("unchecked")
            Optional<T> t = (Optional<T>) EMPTY;
            return t;
        }
    
        // 方式二：该方法调用上面的构造方法创建对象，如果参数值 value 为空则抛出异常，否则创建一个包含该对象的 Optional 对象
        public static <T> Optional<T> of(T value) {
            return new Optional<>(value);
        }
    
        // 方式三：构造一个里面可能为空或者不为空的 Optional 对象
        public static <T> Optional<T> ofNullable(T value) {
            return value == null ? empty() : of(value);
        }
    }
    
    // 附 Objects 中的 requireNonNull() 方法
        public static <T> T requireNonNull(T obj) {
            if (obj == null)
                throw new NullPointerException();
            return obj;
        }
    ```

- `isPresent()` 和 `get()` 、`ifPresent()` 方法（推荐使用）

    ```java
    public T get() {
        if (value == null) {
            throw new NoSuchElementException("No value present");
        }
        return value;
    }
    
    public boolean isPresent() {
        return value != null;
    }
    
    // 如果 value 不为空就指向函数，反之什么都不执行
    public void ifPresent(Consumer<? super T> consumer) {
        if (value != null)
            consumer.accept(value);
    }
    ```
    
    各个方法的测试：
    
    **需求和处理**：单个对象：如果存在则返回该对象，如果不存在则返回空。对象列表：如果存在则返回列表，如果不存在则返回一个空列表（不要返回 null）。
    
    需求：Company 和 Employee 关系为：1对多。
    
    ```java
    package com.gjxaiou.jdk8.lambdaAndFunctionInterface;
    import java.util.List;
    
    public class Company {
        private String name;
        private List<Employee> employees;
       // 省略有参、无参构造方法，以及 get、set 方法
    }
    
    //--------------------------------------------------------
    package com.gjxaiou.jdk8.lambdaAndFunctionInterface;
    
    public class Employee {
        private String name;
        
        // 省略有参、无参构造方法，以及 get、set 方法
    }
    ```
    
    测试程序代码：
    
    ```java
    package com.gjxaiou.jdk8.lambdaAndFunctionInterface;
    
    import java.util.*;
    
    public class OptionalTest2 {
    
        public static void main(String[] args) {
            /**
             * Part1:单个对象处理
             */
            /**
             * 创建一个包装了 "hello" 字符串的 Optional 容器对象
             * 因为 Optional 的构造方法是私有的，所以只能通过其的几个工厂方法来创建对象，如 of/ofNullable/empty 。
             */
            Optional<String> optional = Optional.ofNullable("hello");
            // 使用 ofNullable() 里面是 null 不会抛出异常
            Optional<String> optional2 = Optional.ofNullable(null);
            optional2.ifPresent(item -> System.out.println(item));
            System.out.println("-----------------");
    
            // 使用方式一（ isPresent() 不推荐）：
            if (optional.isPresent()) {
                System.out.println("取出方式一（不推荐）：" + optional.get());
            }
    
            // 推荐的 Optional 使用方式：ifPresent() + Lambda
            optional.ifPresent(item -> System.out.println("推荐的方式：" + item));
            System.out.println("-------");
    
            // optional 不为空则输出自身，如果没有值值输出后面的备选值
            System.out.println(optional.orElse("world"));
            System.out.println("---------");
    
            System.out.println(optional.orElseGet(() -> "niHao"));
    
            /**
             * Part 2：集合处理
             * 需求：取出 company 对象对应的 employee 列表，如果里面不包含任何 employee 信息则返回一个空集合，如果不为空则返回对应列表。
             */
            Employee employee = new Employee("zhangSan");
            Employee employee2 = new Employee("liSi");
            List<Employee> employees = Arrays.asList(employee, employee2);
            Company company = new Company("company1", employees);
    
            // 情况二：使用该行则返回为 空列表
            company.setEmployees(null);
    
            // 构造容器
            Optional<Company> optional3 = Optional.ofNullable(company);
    
            // map() 里面应该是 Function，这里的 Function 对象含义就是输入一个 theCompany,输出 theCompany.getEmployees()
            System.out.println(optional3.map(theCompany -> theCompany.getEmployees()).orElse(Collections.emptyList()));
        }
    }
    ```

## 三、方法引用 method reference

- **方法引用实际上是 Lambda 表达式的一种语法糖**（本质上没有提供新的功能，但是通过更加简洁方便的方式让使用者进行调用）。

- **只有 Lambda 表达式正好是一个现有的方法的时候才可以使用方法引用来替换**，并不是所有的 Lambda 表达式都可以使用方法引用来替换。

- 可以将方法引用看作是一个「函数指针」（即指向函数的指针（Java 中实际不存在））。 即 `System.out::println` 指向 ` System.out.println(item)` 函数。

```java
package com.gjxaiou.jdk8.lambdaAndFunctionInterface;

import java.util.Arrays;
import java.util.List;

public class MethodReferenceDemo {

    public static void main(String[] args) {
        List<String> list = Arrays.asList("hello", "world");
        // Lambda 表达式
        list.forEach(item -> System.out.println(item));
        System.out.println("-----------");
        // method reference
        list.forEach(System.out::println);
        System.out.println("-----------");
        // 字符串拼接输出，结尾不回车
        list.forEach(System.out::format);
        System.out.println("-----------");
    }
}
/** output:
 * hello
 * world
 * -----------
 * hello
 * world
 * -----------
 * helloworld-----------
 */
```



### 方法引用的四种使用方式

- 方式一：`类名::静态方法名`

    ```java
    package com.gjxaiou.jdk8.methodreference;
    
    public class Student {
    
        private String name;
        private int score;
    
       // 省略 Getter/Setter 方法以及构造方法
    
        // 提供两个静态方法
        public static int compareStudentByScore(Student student1, Student student2) {
            return student1.getScore() - student2.getScore();
        }
    
        public static int compareStudentByName(Student student1, Student student2) {
            return student1.getName().compareToIgnoreCase(student2.getName());
        }
    }
    ```
    
    测试使用 `类::静态方法`
    
    ```java
    package com.gjxaiou.jdk8.methodreference;
    
    import java.util.Arrays;
    import java.util.Collections;
    import java.util.List;
    import java.util.function.Function;
    import java.util.function.Supplier;
    
    public class MethodReferenceTest {
    public static void main(String[] args) {
            Student student1 = new Student("zhangsan", 10);
            Student student2 = new Student("lisi", 90);
            Student student3 = new Student("wangwu", 100);
            List<Student> students = Arrays.asList(student1, student2, student3);
    
            /**
             * 方式一：类::静态方法
             */
            // 方式一：默认方式：Lambda 表达式， list 自身就有 sort 方法(默认方法)
            students.sort((studentParam1, studentParam2) ->
                    Student.compareStudentByScore(studentParam1, studentParam2));
            students.forEach(student -> System.out.println(student.getScore() + " " + student.getName()));
            System.out.println("-------");
    
            // 方式二：方法引用，正好有方法与上面 Lambda 表达式含义相同
            students.sort(Student::compareStudentByScore);
            students.forEach(student -> System.out.println(student.getScore() + " " + student.getName()));
            System.out.println("-------");
    
            // 使用方式同上
            students.sort(Student::compareStudentByName);
            students.forEach(student -> System.out.println(student.getName() + " " + student.getScore()));
            System.out.println("-------");
        }
    }
    /** output:
    10 zhangsan
    90 lisi
    100 wangwu
    -------
    10 zhangsan
    90 lisi
    100 wangwu
    -------
    lisi 90
    wangwu 100
    zhangsan 10
    -------
    */
    ```
    
- 方式二：`引用名（对象名）::实例方法名`

    首先新建一个类来创建比较的实例方法，**避免在 Student 类中掺杂不相关的代码**。

    ```java
    package com.gjxaiou.jdk8.methodreference;
    
    public class StudentComparator {
    
        public int compareStudentByScore(Student student1, Student student2) {
            return student1.getScore() - student2.getScore();
        }
    
        public int compareStudentByName(Student student1, Student student2) {
            return student1.getName().compareToIgnoreCase(student2.getName());
        }
    }
    ```

    Student 类的内容同上。

    测试：`对象::实例方法`

    ```java
    package com.gjxaiou.jdk8.methodreference;
    
    import java.util.Arrays;
    import java.util.Collections;
    import java.util.List;
    import java.util.function.Function;
    import java.util.function.Supplier;
    
    public class MethodReferenceTest {
        public static void main(String[] args) {
            Student student1 = new Student("zhangsan", 10);
            Student student2 = new Student("lisi", 90);
            Student student3 = new Student("wangwu", 100);
            List<Student> students = Arrays.asList(student1, student2, student3);
            
             /**
             * 方式二：引用名（对象名）::实例方法名
             */
            StudentComparator studentComparator = new StudentComparator();
    
            // Lambda 表达式
            students.sort((studentParam1, studentParam2) ->
                    studentComparator.compareStudentByScore(studentParam1, studentParam2));
            students.forEach(student -> System.out.println(student.getScore() + " " + student.getName()));
            System.out.println("-------");
    
            // 方法引用
            students.sort(studentComparator::compareStudentByScore);
            students.forEach(student -> System.out.println(student.getScore() + " " + student.getName()));
            System.out.println("-------");
    
            // 使用方式如上
            students.sort(studentComparator::compareStudentByName);
            students.forEach(student -> System.out.println(student.getName() + " " + student.getScore()));
            System.out.println("-------");
        }
    }
    /** output:
    10 zhangsan
    90 lisi
    100 wangwu
    -------
    10 zhangsan
    90 lisi
    100 wangwu
    -------
    lisi 90
    wangwu 100
    zhangsan 10
    -------
    */
    ```

- 第三种：`类名::实例方法名`

    上述的 Student 中的两个比较方法在设计层面是不符合的，因为两个方法和所在的 Student 类没有任何关系（即放在其他类中照样可以运行）。应该像下面一样定义：

    ```java
    package com.gjxaiou.jdk8.methodreference;
    
    public class Student {
        private String name;
        private int score;
    
        public Student(String name, int score) {
            this.name = name;
            this.score = score;
        }
    
      	// 省略 Getter 和 Setter 方法
    
        // 调用方法的 Student 类和传入的 Student 类进行比较
        public int compareByScore(Student student) {
            return this.getScore() - student.getScore();
        }
    
        public int compareByName(Student student) {
            return this.getName().compareToIgnoreCase(student.getName());
        }
    }
    ```

    测试程序

    ```java
    package com.gjxaiou.jdk8.methodreference;
    
    import java.util.Arrays;
    import java.util.Collections;
    import java.util.List;
    import java.util.function.Function;
    import java.util.function.Supplier;
    
    public class MethodReferenceTest {
    
        public static void main(String[] args) {
            Student student1 = new Student("zhangsan", 10);
            Student student2 = new Student("lisi", 90);
            Student student3 = new Student("wangwu", 50);
            Student student4 = new Student("zhaoliu", 40);
    
            List<Student> students = Arrays.asList(student1, student2, student3, student4);
    
             /**
             * 方式三：类名::实例方法名
             */
            // compareByScore() 方法是 sort() 方法里面接收的 Lambda 表达式的第一个参数调用的。如果接收多个参数，则除了第一个后面的所有参数都作为
            // compareByScore 的方法参数传递进去。所以调用该方法的是待排序的两个 Student 对象的第一个，第二个对象作为参数传入该方法。
            students.sort(Student::compareByScore);
            students.forEach(student -> System.out.println(student.getScore() + " " + student.getName()));
            System.out.println("-----------");
            students.sort(Student::compareByName);
            students.forEach(student -> System.out.println(student.getName() + " " + student.getScore()));
            System.out.println("-----------");
    
            // 第二个示例：使用 JDK 自带的排序方法进行排序
            List<String> cities = Arrays.asList("qingdao", "chongqing", "tianjin", "beijing");
            // lambda 表达式
            Collections.sort(cities, (city1, city2) -> city1.compareToIgnoreCase(city2));
            cities.forEach(city -> System.out.println(city));
    		System.out.println("-----------");
            
            Collections.sort(cities, String::compareToIgnoreCase);
            cities.forEach(System.out::println);
            System.out.println("-----------");
        }
    }
    /** output:
    10 zhangsan
    90 lisi
    100 wangwu
    -----------
    lisi 90
    wangwu 100
    zhangsan 10
    -----------
    beijing
    chongqing
    qingdao
    tianjin
    -----------
    beijing
    chongqing
    qingdao
    tianjin
    -----------
    */
    ```

- 第四种：构造方法引用: `类名::new`

    ```java
    package com.gjxaiou.jdk8.methodreference;
    
    
    import java.util.Arrays;
    import java.util.Collections;
    import java.util.List;
    import java.util.function.Function;
    import java.util.function.Supplier;
    
    public class MethodReferenceTest {
    
        public String getString(Supplier<String> supplier) {
            return supplier.get() + "test";
        }
    
        public String getString2(String str, Function<String, String> function) {
            return function.apply(str);
        }
    
        public static void main(String[] args) {
            /**
             * 方式四：类名::new
             */
            MethodReferenceTest methodReferenceTest = new MethodReferenceTest();
            System.out.println(methodReferenceTest.getString(String::new));
            System.out.println(methodReferenceTest.getString2("hello", String::new));
        }
    }
    /** output:
    test
    hello
    */
    ```
    
    

## 四、默认方法

**接口中为什么有默认方法**：主要为了和之前版本兼容。例如在 List 接口中的默认方法：`sort()`，如果不设置为默认方法而是抽象方法的话，则以前实现该 List 接口的类都要实现该方法。而使用默认方法相当于在 List 接口中将 `sort()` 已经进行了实现，则对于实现类来说相当于直接继承了该 `sort()`方法，可以直接使用了。

测试程序：定义两个接口，两个接口有一样的默认方法，但是实现不同：

```java
package com.gjxaiou.jdk8.defaultmethod;

public interface MyInterface1 {
    default void myMethod() {
        System.out.println("MyInterface1");
    }
}

// ------------------------------------------

package com.gjxaiou.jdk8.defaultmethod;

public interface MyInterface2 {
    default void myMethod() {
        System.out.println("MyInterface2");
    }
}
```

如果自定义类实现其中任意一个接口，是可以直接调用 `myMethod()`方法的。

```java
package com.gjxaiou.jdk8.defaultmethod;

public class MyClass implements MyInterface2{
    public static void main(String[] args) {
        MyClass myClass = new MyClass();
        myClass.myMethod();
    }
}
```

但是如果同时实现两个接口，会报错，除了重写 `myMethod()`方法

```java
public class MyClass implements MyInterface1, MyInterface2 {
    @Override
    public void myMethod() {
        // 重写该方法
    }

    public static void main(String[] args) {
        MyClass myClass = new MyClass();
        myClass.myMethod();
    }
}
```

如果第一个接口有实现类，第二个接口没有，则如果 MyClass 既继承类该实现类也实现了第二个接口，则里面的 `myMethod()`方法是：**实现类里面的**。「Java 认为实现类（更加具体）优先级高于接口（认为是一种契约）」

```java
package com.gjxaiou.jdk8.defaultmethod;

public class MyInterface1Impl implements MyInterface1 {
    @Override
    public void myMethod() {
        System.out.println("MyInterface1Impl");
    }
}
```

```java
package com.gjxaiou.jdk8.defaultmethod;

public class MyClass extends MyInterface1Impl implements MyInterface2 {

    public static void main(String[] args) {
        MyClass myClass = new MyClass();
        myClass.myMethod();
    }
}
```

输出结果为：`MyInterface1Impl`



### 所有函数式接口总结与测试

![img](Java8%20%E6%96%B0%E7%89%B9%E6%80%A7.resource/640.jpg)

测试程序

```java
package com.gjxaiou.jdk8.lambdaAndFunctionInterface;

import java.math.BigDecimal;
import java.util.function.*;

public class Test {
    public static void main(String[] args) {
        Predicate<Integer> predicate = x -> x > 185;
        Student student = new Student("9龙", 23, 175);
        System.out.println(
                "9龙的身高高于185吗？：" + predicate.test(student.getStature()));

        Consumer<String> consumer = System.out::println;
        consumer.accept("命运由我不由天");

        Function<Student, String> function = Student::getName;
        String name = function.apply(student);
        System.out.println(name);

        Supplier<Integer> supplier =
                () -> Integer.valueOf(BigDecimal.TEN.toString());
        System.out.println(supplier.get());

        UnaryOperator<Boolean> unaryOperator = uglily -> !uglily;
        Boolean apply2 = unaryOperator.apply(true);
        System.out.println(apply2);

        BinaryOperator<Integer> operator = (x, y) -> x * y;
        Integer integer = operator.apply(2, 3);
        System.out.println(integer);

        test(() -> "我是一个演示的函数式接口");
    }

    /**
     * 演示自定义函数式接口使用
     *
     * @param worker
     */
    public static void test(Worker worker) {
        String work = worker.work();
        System.out.println(work);
    }

    public interface Worker {
        String work();
    }

    static class Student {
        String name;
        int age;
        int height;
        int stature;

        public Student(String name, int age, int height) {
            this.name = name;
            this.age = age;
            this.height = height;
        }

        public String getName() {
            return name;
        }

        public Integer getStature() {
            return stature;
        }
    }
}
/**
 * output:
 * 9龙的身高高于185吗？：false
 * 命运由我不由天
 * 9龙
 * 10
 * false
 * 6
 * 我是一个演示的函数式接口
 */
```



## 五、Stream 流

### （一）基本概念

Stream 流由三部分组成：源 + 零个或者多个中间操作 + 终止操作

Stream 流操作的分类：惰性求值 和 及早求值

- 惰性求值

    `stream.xxx().yyy().count();`  其中 `steam` 就是源，`xxx()` 和 `yyy()` 都是中间操作，即惰性求值，`count()` 为终止操作，即及早求值。只有调用 `count()` 方法时候中间操作才会被发起，如果没有调用（如没有写 `.count()` 方法)，则中间操作都不会执行。

- 及早求值

    `count()` 本身就是一个及早求值。

- Collection 提供了新的 `stream()` 方法
- **流不存储值，通过管道的方式获取值，值还是存储在数据提供方**。
- 本质是函数式的，对流的操作会生成一个结果，但并不会修改底层的数据源，集合可以作为流的底层数据源
- 延迟查找，很多流操作（过滤、映射、排序等）都可以延迟实现



**常见的测试 Demo**

1.Stream 流的三种创建方式

```java
package com.gjxaiou.jdk8.stream;

import java.util.Arrays;
import java.util.List;
import java.util.stream.Stream;

/**
 * Steam 流的三种创建方式
 */
public class StreamTest1 {

    public static void main(String[] args) {
        // 方式一：直接创建元素填充
        Stream stream1 = Stream.of("hello", "world", "hello world");

        // 方式二：主要针对数组
        String[] myArray = new String[]{"hello", "world", "hello world"};
        Stream stream2 = Stream.of(myArray);
        Stream stream3 = Arrays.stream(myArray);

        // 方式三：主要针对 List
        List<String> list = Arrays.asList(myArray);
        Stream stream4 = list.stream();
    }
}
```

2.使用测试 1

```java
package com.gjxaiou.jdk8.stream;

import java.util.Arrays;
import java.util.List;
import java.util.stream.IntStream;

public class StreamTest2 {

    public static void main(String[] args) {
        IntStream.of(new int[]{5, 6, 7}).forEach(System.out::println);
        System.out.println("-----");

        // 包含前一个，不包含后一个
        IntStream.range(3, 8).forEach(System.out::println);
        System.out.println("-----");

        // 包含前一个，也包含后一个
        IntStream.rangeClosed(3, 8).forEach(System.out::println);


        /**
         * 将 List 中所有元素 * 2 之后相加和
         */
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6);

        // map() 参数是一个 Function，即接收一个值，返回一个值
        System.out.println(list.stream().map(i -> 2 * i).reduce(0, Integer::sum));
    }
}
    // forEach 方法源码
    void forEach(IntConsumer action);
	// IntStream 源码	
    public static IntStream of(int... values) {
        return Arrays.stream(values);
    }        
```

-  Collection 方法分析

  对流中的元素执行一个可变的汇聚操作（看 JavaDoc 文档中的分析，可以分解为三个步骤，分别对应下面三个函数式接口）

  ```java
  <R> R collect(Supplier<R> supplier,
                BiConsumer<R, ? super T> accumulator,
                BiConsumer<R, R> combiner);
  
  <R, A> R collect(Collector<? super T, A, R> collector);
  ```

  测试方法：

  ```java
  package com.gjxaiou.jdk8.stream;
  
  import java.util.*;
  import java.util.stream.Collectors;
  import java.util.stream.Stream;
  
  public class StreamTest4 {
  
      public static void main(String[] args) {
          Stream<String> stream = Stream.of("hello", "world", "helloWorld");
  
          // 将流转换为字符串数组
          String[] stringArray = stream.toArray(length -> new String[length]);
          // 使用方法引用改造上面的 Lambda 表达式写法
          // String[] stringArray = stream.toArray(String[]::new);
          Arrays.asList(stringArray).forEach(System.out::println);
  
          /**
           * 将流转换为一个 List
           */
          Stream<String> stream2 = Stream.of("hello", "world", "helloWorld");
          // collect() 是一个终止方法
          List<String> list = stream2.collect(Collectors.toList());
          // 上面一行等价于下面这个，本质是 Collectors.toList() 就是对下面代码的封装
          List<String> list1 = stream2.collect(
                  () -> new ArrayList(), // 不接受参数，返回一个结果，也是最终结果
                  (theList, item) -> theList.add(item),// 两个参数，进行遍历然后加入集合
                  (theList1, theList2) -> theList1.addAll(theList2)
          );// 将每次遍历形成的 List2 加入最终返回的 List2 中
          // 使用方法引用简化上面代码
          List<String> list2 = stream2.collect(LinkedList::new, LinkedList::add, LinkedList::addAll);
  
          list.forEach(System.out::println);
  
          // 流转换为集合的方式二
          Stream<String> stream3 = Stream.of("hello", "world", "helloWorld");
          // 可以自定义返回 List 的类型
          List<String> list3 = stream3.collect(Collectors.toCollection(ArrayList::new));
          list3.forEach(System.out::println);
  
          // 自定义转换为 Set 类型
          Stream<String> stream4 = Stream.of("hello", "world", "helloWorld");
          Set<String> set = stream4.collect(Collectors.toCollection(TreeSet::new));
          System.out.println(set.getClass());
  
          set.forEach(System.out::println);
  
          // 转换为 String 对象
          Stream<String> stream5 = Stream.of("hello", "world", "helloWorld");
          String str = stream5.collect(Collectors.joining()).toString();
          System.out.println(str);
      }
  }
  ```

- flatMap 方法

  例如一个集合中有三个元素，每个元素都是一个 List，如果使用原始的 map 进行映射的时候，得到的还是还是元素，每个元素都是一个 List，其中值是原始元素的逐个映射。但是 flatMap 将原始集合中的所有小元素逐个映射的结构放在一个 List 中。进行扁平化。

  ```java
  package com.gjxaiou.jdk8.stream;
  
  import java.util.Arrays;
  import java.util.List;
  import java.util.stream.Collectors;
  import java.util.stream.Stream;
  
  public class StreamTest5 {
      public static void main(String[] args) {
          List<String> list = Arrays.asList("hello", "world", "helloworld", "test");
          // 将集合中每个元素转换为大写然后输出
          list.stream().map(String::toUpperCase).collect(Collectors.toList()).forEach(System.out::println);
  
          System.out.println("----------");
  
          List<Integer> list2 = Arrays.asList(1, 2, 3, 4, 5);
          list2.stream().map(item -> item * item).collect(Collectors.toList()).forEach(System.out::println);
  
          System.out.println("----------");
  
          // flatMap：平整化的 Map
          Stream<List<Integer>> stream = Stream.of(Arrays.asList(1),
                  Arrays.asList(2, 3), Arrays.asList(4, 5, 6));
  
          // theList -> theList.stream() 将每个 List 都转换为 Stream，然后通过 flatMap 将所有 Stream 打平
          stream.flatMap(theList -> theList.stream()).map(item -> item * item).forEach(System.out::println);
      }
  }
  ```

- `generate()`、`iterate()`方法以及需要注意的知识点

    ```java
    package com.gjxaiou.jdk8.stream;
    
    import java.util.IntSummaryStatistics;
    import java.util.UUID;
    import java.util.stream.Stream;
    
    public class StreamTest6 {
        public static void main(String[] args) {
            // generate() 返回
            Stream<String> stream = Stream.generate(UUID.randomUUID()::toString);
            stream.findFirst().ifPresent(System.out::println);
    
            // 没有 limit() 就是无限流
            Stream<Integer> stream2 = Stream.iterate(1, item -> item + 2).limit(6);
    
            // 找出该流中大于 2 的元素，然后将每个元素乘以 2，然后忽略掉流中的前两个元素，然后再取流中的前两个元素，最后求出流中元素的总和。
            System.out.println(stream2.filter(item -> item > 2).mapToInt(item -> item * 2).skip(2).limit(2).sum());
    
            // 因为像 min() 、max() 返回 OptionalInt，因为防止如果如空则可以规避 NPE 问题，而上面的 sum() 不存在 NPE 问题。
            stream2.filter(item -> item > 2).mapToInt(item -> item * 2).skip(2).limit(2).max().ifPresent(System.out::println);
    
            // 先得到结果
            IntSummaryStatistics summaryStatistics = stream2.filter(item -> item > 2).
                    mapToInt(item -> item * 2).skip(2).limit(2).summaryStatistics();
    
            System.out.println(summaryStatistics.getMin());
            System.out.println(summaryStatistics.getCount());
            System.out.println(summaryStatistics.getMax());
    
            /**
             * 知识点：Stream 流不能重复使用
             */
            // 注意 stream 中每个操作都会生成一个全新的 stream 流
            System.out.println(stream2);
            // stream2.filter() 会返回一个新的流对象。
            System.out.println(stream2.filter(item -> item > 2));
            // 这行会报错：因为流不能在使用过或者关闭之后再次使用。因为这里操作的 stream2 不是上一行的返回值，而是原始的 stream2。
            System.out.println(stream2.distinct());
    
            // 下面是正确方式
            System.out.println(stream2);
            Stream<Integer> stream3 = stream2.filter(item -> item > 2);
            System.out.println(stream3);
            Stream<Integer> stream4 = stream3.distinct();
            System.out.println(stream4);
        }
    }
    ```
    
-  知识点：中间操作和终止操作的测试

    ```java
    package com.gjxaiou.jdk8.stream;
    
    
    import java.util.Arrays;
    import java.util.List;
    
    /**
     * 知识点：中间操作和终止操作
     * 每个元素首字母大写然后输出
     */
    public class StreamTest7 {
    
        public static void main(String[] args) {
            List<String> list = Arrays.asList("hello", "world", "hello world");
    
            list.stream().map(item -> item.substring(0, 1).toUpperCase() + item.substring(1)).
                    forEach(System.out::println);
    
    
            // 这个是不会执行的，因为 map() 是一个中间操作，在没有终止操作的情况下是不会执行的。因为是惰性求值
            list.stream().map(item -> {
                String result = item.substring(0, 1).toUpperCase() + item.substring(1);
                System.out.println("test");
                return result;
            });
    
            // 这个会执行
            list.stream().map(item -> {
                String result = item.substring(0, 1).toUpperCase() + item.substring(1);
                System.out.println("test");
                return result;
            }).forEach(System.out::println);
        }
    }
    ```

- `distinct()` 方法

    ```java
    package com.gjxaiou.jdk8.stream;
    
    import java.util.stream.IntStream;
    
    public class StreamTest8 {
    
        public static void main(String[] args) {
            // 运行这行，输出 0 1 但是程序一直无法执行完成，因为 iterate() 迭代输出导致 distinct() 一直无法结束
            IntStream.iterate(0, i -> (i + 1) % 2).distinct().limit(6).forEach(System.out::println);
            // 正确的使用方式
            IntStream.iterate(0, i -> (i + 1) % 2).limit(6).distinct().forEach(System.out::println);
        }
    }
    ```
    
    #### 内部迭代和外部迭代
    
    - 外部迭代：例如 for 循环
    
    - 内部迭代：Stream 流
    
        内部迭代方式不一定是按照方法调用的顺序进行执行，内部会进行优化。所有的中间指令可以看做是在一个容器中，然后针对流中每一个元素进行应用。如果看成 for 循环则最终仅仅遍历了一遍。
    
    <img src="Java8%20%E6%96%B0%E7%89%B9%E6%80%A7.resource/image-20201028152857282-1604234304841.png" alt="image-20201028152857282" style="zoom:50%;" />
    
    

集合关注的是数据和数据存储本身。

流关注的是数据的计算。

流和迭代器两者类似的点：流是无法重复使用或者消费的。

#### 并发流

测试两者时间

```java
package com.gjxaiou.jdk8.stream;

import java.util.ArrayList;
import java.util.List;
import java.util.UUID;
import java.util.concurrent.TimeUnit;

public class StreamTest9 {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>(5000000);
        for (int i = 0; i < 5000000; ++i) {
            list.add(UUID.randomUUID().toString());
        }

        System.out.println("开始排序");
        long startTime = System.nanoTime();

        // 分别执行下面两句看执行时间差距
        // 排序耗时：4583
        // list.stream().sorted().count();
        // 排序耗时：1591
        list.parallelStream().sorted().count();

        long endTime = System.nanoTime();
        long millis = TimeUnit.NANOSECONDS.toMillis(endTime - startTime);
        System.out.println("排序耗时：" + millis);
    }
}
```



#### 短路运算

```java
package com.gjxaiou.jdk8.stream;


import java.util.Arrays;
import java.util.List;

public class StreamTest10 {

    public static void main(String[] args) {
        // 将列表中长度为 5 的第一个单词打印
        List<String> list = Arrays.asList("hello", "world", "hello world");

        list.stream().mapToInt(item -> item.length()).filter(length -> length == 5).
                findFirst().ifPresent(System.out::println);

        // 会对流中每个元素依次应用第一个操作、第二个操作、。。。。并且存在短路运算（前面为假则后面就不执行了）
        list.stream().mapToInt(item -> {
            int length = item.length();
            System.out.println(item);
            return length;
        }).filter(length -> length == 5).findFirst().ifPresent(System.out::println);

    }
}
```



**Map 和 Flatmap 区别**

```java
package com.gjxaiou.jdk8.stream;

import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

/**
 * 知识点：map 和 flatMap 区别
 */
public class StreamTest11 {

    public static void main(String[] args) {
        /**
         * 需求：对 List 中所有元素按照空格划分之后去重输出
         */
        List<String> list = Arrays.asList("hello welcome", "world hello",
                "hello world hello", "hello welcome");

        // 错误用法：因为 map() 返回一个 String[]，四个元素划分为四个 String[]，并且四个肯定不相同，所以最终相当于输出了所有元素。
        List<String[]> result = list.stream().map(item -> item.split(" ")).distinct().
                collect(Collectors.toList());
        result.forEach(item -> Arrays.asList(item).forEach(System.out::println));

        List<String> result2 =
                list.stream().map(item -> item.split(" ")). flatMap(Arrays::stream). 	  distinct().collect(Collectors.toList());

        result2.forEach(System.out::println);
    }
}
```

示例 2

```java
package com.gjxaiou.jdk8.stream;

import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

/**
 * 需求：第一个集合中每个元素都和第二个集合中每个元素进行拼接。
 * 知识点：flatMap
 */
public class StreamTest12 {

    public static void main(String[] args) {
        List<String> list1 = Arrays.asList("Hi",  "你好");
        List<String> list2 = Arrays.asList("zhangsan", "lisi", "wangwu");

        List<String> result = list1.stream().flatMap(item -> list2.stream().map(item2 -> item + " " + item2)).
                collect(Collectors.toList());
        result.forEach(System.out::println);
    }
}
/** output:
 * Hi zhangsan
 * Hi lisi
 * Hi wangwu
 * 你好 zhangsan
 * 你好 lisi
 * 你好 wangwu
 */
```



#### 分组和分区

```java
package com.gjxaiou.jdk8.stream;

import java.util.Arrays;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public class StreamTest13 {

    public static void main(String[] args)
        Student student1 = new Student("zhangsan", 100, 20);
        Student student2 = new Student("lisi", 90, 20);
        Student student3 = new Student("wangwu", 90, 30);
        Student student4 = new Student("zhangsan", 80, 40);

        List<Student> students = Arrays.asList(student1, student2, student3, student4);

        // 按照姓名进行分组
        Map<String, List<Student>> map = students.stream().
                collect(Collectors.groupingBy(Student::getName));
        // 按照分数进行分组
        Map<Integer, List<Student>> map2 = students.stream().
                collect(Collectors.groupingBy(Student::getScore));

        // 分组之后，每个分组有多少个元素
        Map<String, Long> map3 = students.stream().
                collect(Collectors.groupingBy(Student::getName, Collectors.counting()));

        // 分组之后，每个分组的分数平均值
        Map<String, Double> map4 = students.stream().
                collect(Collectors.groupingBy(Student::getName,
                        Collectors.averagingDouble(Student::getScore)));

        // 分区，只有两组（符合和不符合）
        Map<Boolean, List<Student>> map5 = students.stream().
                collect(Collectors.partitioningBy(student -> student.getScore() >= 90));

        System.out.println(map);
    }
}
```



### （二）源码分析

### Collector 源码分析

JavaDoc 文档还是需要读一遍。

1. `collect()` 方法是一个收集器
2. `Collector` 是一个接口，也是 `collect()` 方法的入参类型。

**Collector** 接口源码分析

```java
// T 是流中每个元素的类型
// 中间累积结果容器类型
// 最终结果类型
public interface Collector<T, A, R> {}
```

Collector 是一个可变的汇聚操作，作用是将输入元素累积到一个可变的结果容器中。它会在所有的元素都处理完毕之后，将累积的结果转换为一个最终的表示（这是一个可选的操作）；同时其支持串行和并行两种方式执行。

Collectors 本身提供了关于 Collector 的常见汇聚实现， Collectors 本身实际上是一个工厂。

Collector 是由四个函数组成的

- `supplier`：提供一个可变的结果容器

- `accumulator`：往结果容器中不断的累加流中迭代遍历的每个元素

- `combiner`：将多个部分结果合并为一个最终结果，主要是并发方面。

    因为该函数用于多线程并行情况下，

    示例：有四个 4 线程同时去执行，那么就会生成 4 个部分结果。

    例如四个结果分别为：1,2,3,4，组合可能性之一为：

    1, 2 -> 5 ：折叠的含义：如 1 和 2 的操作，例如是往集合中添加元素，则可以生成一个新的集合返回，然后将 1， 2 集合中元素都添加到新的集合中或者可以将 1 放入 2 的集合，然后将 2 集合进行返回。

    5, 3 -> 6

    6, 4 -> 7

- `finisher`：可选操作，将累积的中间结果转换为一种最终的表示。

#### collector 的同一性和结合性分析

为了保证串行和并行都能等到等价结果，收集器 Collector 函数必须满足两个条件：

- `identity`：同一性：即对于任意累加结果和一个空的结果容器进行 combine 合并结果和原先结果一下。即 对于中间部分累加结果 a，`a = combiner.apply(a, supplier.get())`，因为 `supplier.get()` 获得的就是一个空的集合。

    示例程序

    ```java
    BiFunction<List<String>, List<String>, List<String>> resultList =
        (List<String> list1, List<String> list2) -> {
        list1.addAll(list2);
        return list1;
    };
    ```

- `associativity`：结合性：

    要求得到的两个结果 r1 和 r2 是一样的

    ```java
    /**
      * 串行执行（没有分割）
    */
    // a1 就是得到的空的结果容器
    A a1 = supplier.get();
    // accept() 方法的两个参数：每次累加得到的中间结果，流中下一个待处理的元素。该步骤处理完成之后, t1 就放入 a1 中了。
    accumulator.accept(a1, t1);
    accumulator.accept(a1, t2);
    // 累积结果转换为最终结果类型 R
    R r1 = finisher.apply(a1);
    
    /*
      并行执行（结果进行了分割）
    */
    // 相当于第一个线程开启一个空的结果容器
    A a2 = supplier.get();
    accumulator.accept(a2, t1);
    // 第二个线程开启一个空的结果容器
    A a3 = supplier.get();
    accumulator.accept(a3, t2);
    // 通过 combiner.apply() 将两个结果合并为一个结果，然后通过 finisher 进行结果类型转换。
    R r2 = finisher.apply(combiner.apply(a2, a3));
    ```



### Collectors

该类本质上是一个工厂，提供了各种收集器。



测试程序

首先还是创建了一个 POJO 类

```java
package com.gjxaiou.jdk8.streamSourceCode;

public class Student {
    private String name;
    private int score;

    public Student(String name, int score) {
        this.name = name;
        this.score = score;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getScore() {
        return score;
    }

    public void setScore(int score) {
        this.score = score;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", score=" + score +
                '}';
    }
}
```

测试程序 1

```java
package com.gjxaiou.jdk8.streamSourceCode;


import java.util.*;

import static java.util.stream.Collectors.*;

public class StreamTest1 {

    public static void main(String[] args) {
        Student student1 = new Student("zhangsan", 80);
        Student student2 = new Student("lisi", 90);
        Student student3 = new Student("wangwu", 100);
        Student student4 = new Student("zhaoliu", 90);
        Student student5 = new Student("zhaoliu", 90);

        List<Student> students = Arrays.asList(student1, student2, student3, student4, student5);

        // 测试 1：集合转换为流然后转换回集合
        List<Student> students1 = students.stream().collect(toList());
        students1.forEach(System.out::println);
        System.out.println("------------");

        // 测试2：集合转换为流然后得到集合中元素数量
        System.out.println("count: " + students.stream().collect(counting()));
        System.out.println("count: " + students.stream().count());
        System.out.println("------------");

        // 打印分数最小、最大，平均，总数的学生，摘要信息。
        students.stream().collect(minBy(Comparator.comparingInt(Student::getScore))).ifPresent(System.out::println);
        students.stream().collect(maxBy(Comparator.comparingInt(Student::getScore))).ifPresent(System.out::println);
        System.out.println(students.stream().collect(averagingInt(Student::getScore)));
        System.out.println(students.stream().collect(summingInt(Student::getScore)));
        IntSummaryStatistics intSummaryStatistics =
                students.stream().collect(summarizingInt(Student::getScore));
        System.out.println(intSummaryStatistics);
        System.out.println("------------");

        // 字符串拼接：joining() 收集器
        System.out.println(students.stream().map(Student::getName).collect(joining()));
        System.out.println(students.stream().map(Student::getName).collect(joining(", ")));
        System.out.println(students.stream().map(Student::getName).collect(joining(", ", "<begin>" +
                " ", " <end>")));
        System.out.println("------------");

        // gropingBy 进行分组嵌套
        Map<Integer, Map<String, List<Student>>> map = students.stream().
                collect(groupingBy(Student::getScore, groupingBy(Student::getName)));
        System.out.println(map);
        System.out.println("------------");

        // partitioningBy 进行分区嵌套
        Map<Boolean, List<Student>> map2 = students.stream().
                collect(partitioningBy(student -> student.getScore() > 80));
        System.out.println(map2);
        System.out.println("------------");

        Map<Boolean, Map<Boolean, List<Student>>> map3 = students.stream().
                collect(partitioningBy(student -> student.getScore() > 80,
                        partitioningBy(student -> student.getScore() > 90)));
        System.out.println(map3);
        System.out.println("------------");

        Map<Boolean, Long> map4 = students.stream().
                collect(partitioningBy(student -> student.getScore() > 80, counting()));
        System.out.println(map4);
        System.out.println("------------");

        // 首先按照名字进行分组，
        Map<String, Student> map5 = students.stream().
                collect(groupingBy(Student::getName,
                        collectingAndThen(minBy(Comparator.comparingInt(Student::getScore)),
                        Optional::get)));
        System.out.println(map5);
    }
}
```

### 比较器：Comparator

测试程序：

```java
package com.gjxaiou.jdk8.streamSourceCode;


import java.util.Arrays;
import java.util.Collections;
import java.util.Comparator;
import java.util.List;

public class MyComparatorTest {

    public static void main(String[] args) {
        List<String> list = Arrays.asList("nihao", "hello", "world", "welcome");

        //
        Collections.sort(list, (item1, item2) -> item1.length() - item2.length());
        Collections.sort(list, (item1, item2) -> item2.length() - item1.length());

        // 按照字符串长度降序排列，使用方法引用
        Collections.sort(list, Comparator.comparingInt(String::length).reversed());

        // 使用 Lambda 表达式实现上面同样功能，同时这里参数类型是无法推断出来的，需要显式指定。如果不写的情况下，默认是 Object，具体看源码。
        Collections.sort(list, Comparator.comparingInt((String item) -> item.length()).reversed());

        /**
         * 方式二：
         */
        list.sort(Comparator.comparingInt(String::length).reversed());
        list.sort(Comparator.comparingInt((String item) -> item.length()).reversed());

        // 双重比较，thenComparing 只有在前面相同才会执行，具体看源码。
        Collections.sort(list,
                Comparator.comparingInt(String::length).thenComparing(String.CASE_INSENSITIVE_ORDER));

        // 相当于实现了后面的 String.CASE_INSENSITIVE_ORDER 方法
        Collections.sort(list, Comparator.comparingInt(String::length).
                thenComparing((item1, item2) -> item1.toLowerCase().compareTo(item2.toLowerCase())));

        // 使用方法引用实现
        Collections.sort(list, Comparator.comparingInt(String::length).
                thenComparing(Comparator.comparing(String::toLowerCase)));

        Collections.sort(list, Comparator.comparingInt(String::length).
                thenComparing(Comparator.comparing(String::toLowerCase,
                        Comparator.reverseOrder())));

        Collections.sort(list, Comparator.comparingInt(String::length).reversed().
                thenComparing(Comparator.comparing(String::toLowerCase,
                        Comparator.reverseOrder())));

        Collections.sort(list, Comparator.comparingInt(String::length).reversed().
                thenComparing(Comparator.comparing(String::toLowerCase, Comparator.reverseOrder())).
                thenComparing(Comparator.reverseOrder()));

        System.out.println(list);
    }
}
```



#### 自定义实现收集器 Collector

```java
package com.gjxaiou.jdk8.streamSourceCode;


import java.util.*;
import java.util.function.BiConsumer;
import java.util.function.BinaryOperator;
import java.util.function.Function;
import java.util.function.Supplier;
import java.util.stream.Collector;

import static java.util.stream.Collector.Characteristics.IDENTITY_FINISH;
import static java.util.stream.Collector.Characteristics.UNORDERED;

public class MySetCollector<T> implements Collector<T, Set<T>, Set<T>>{

    // 返回一个空的结果容器
    @Override
    public Supplier<Set<T>> supplier() {
        System.out.println("supplier invoked!");
        return HashSet<T>::new;
    }

    @Override
    public BiConsumer<Set<T>, T> accumulator() {
        System.out.println("accumulator invoked!");
        // 这里不能使用具体的 Set 实现类，因为要兼容上面的 Supplier 返回值。（返回值可能是 Set 的任意实现类）
        return Set<T>::add;
    }

    @Override
    public BinaryOperator<Set<T>> combiner() {
        System.out.println("combiner invoked!");
        return (set1, set2) -> {
            set1.addAll(set2);
            return set1;
        };
    }

    // collect() 源码里面有说明，如果中间结果和最终结果类型一样，则 finisher() 不调用，直接进行强制类型转换。
    @Override
    public Function<Set<T>, Set<T>> finisher() {
        System.out.println("finisher invoked!");
        // 下面这行等价于： return t -> t;
        return Function.identity();
     //   throw new UnsupportedOperationException();
    }

    @Override
    public Set<Characteristics> characteristics() {
        System.out.println("characteristics invoked!");
        return Collections.unmodifiableSet(EnumSet.of(IDENTITY_FINISH, UNORDERED));
    }

    public static void main(String[] args) {
        List<String> list = Arrays.asList("hello", "world", "welcome", "hello");
        Set<String> set = list.stream().collect(new MySetCollector<>());

        System.out.println(set);
    }
}
```

另一种集合的 Collect() 测试

```java
package com.gjxaiou.jdk8.streamSourceCode;


import java.util.*;
import java.util.function.BiConsumer;
import java.util.function.BinaryOperator;
import java.util.function.Function;
import java.util.function.Supplier;
import java.util.stream.Collector;

/**
 * 输入：Set<String>
 * 输出：Map<String,String>
 *
 * @param <T>
 */
public class MySetCollector2<T> implements Collector<T, Set<T>, Map<T, T>> {

    @Override
    public Supplier<Set<T>> supplier() {
        System.out.println("supplier invoked!");

        //  return HashSet<T>::new;
        return () -> {
            System.out.println("-----------");
            return new HashSet<T>();
        };
        /**
         *     A a1 = supplier.get();
         *     accumulator.accept(a1, t1);
         *     accumulator.accept(a1, t2);
         *     R r1 = finisher.apply(a1);  // result without splitting
         *
         *     A a2 = supplier.get();
         *     accumulator.accept(a2, t1);
         *     A a3 = supplier.get();
         *     accumulator.accept(a3, t2);
         *     R r2 = finisher.apply(combiner.apply(a2, a3));  // result with splitting
         *
         */
    }

    // 接收两个参数（结果容器类型，流中元素类型），并且不返回值
    @Override
    public BiConsumer<Set<T>, T> accumulator() {
        System.out.println("accumulator invoked!");

        return (set, item) -> {
            // 并发异常根源：多个线程使用 accumulator()
            // 遍历同一个结合进行添加元素的时候，但是这里又进行了遍历所以爆出异常，具体异常分析见：ConcurrentModificationException 源码。
            System.out.println("accumulator: " + set + ", " + Thread.currentThread().getName());
            set.add(item);
        };
    }

    // 并发流时候才会调用
    @Override
    public BinaryOperator<Set<T>> combiner() {
        System.out.println("combiner invoked!");

        return (set1, set2) -> {
            System.out.println("set1: " + set1);
            System.out.println("set2: " + set2);
            set1.addAll(set2);
            return set1;
        };
    }

    // 中间类型和最终类型不一样，需要使用 finisher() 转换
    @Override
    public Function<Set<T>, Map<T, T>> finisher() {
        System.out.println("finisher invoked!");

        return set -> {
            Map<T, T> map = new TreeMap<>();
            set.stream().forEach(item -> map.put(item, item));
            return map;
        };
    }

    // 这一块见：Collector 接口中的 Characteristics 枚举类的 JavaDoc 文档
    @Override
    public Set<Characteristics> characteristics() {
        System.out.println("characteristics invoked!");

        return Collections.unmodifiableSet(EnumSet.of(Characteristics.UNORDERED));
        // 测试情况一：这样会在运行时候报错，因为如果加上这个标识表示中间结果和最终结果类型一致，会直接使用强制类型转换，不会再使用 finisher
        //  return Collections.unmodifiableSet(EnumSet.of(Characteristics.IDENTITY_FINISH));
        // 测试情况二：会爆出并发异常，因为下面使用了 parallelStream()，如果加上这个是相当于多个线程操作同一个结果容器（combiner
        // () 也就不会被调用了），不加表示多个线程操作多个结果容器（和线程数相同）。
        // return Collections.unmodifiableSet(EnumSet.of(Characteristics.CONCURRENT));
    }

    public static void main(String[] args) {
        System.out.println(Runtime.getRuntime().availableProcessors());

        for (int i = 0; i < 1; ++i) {
            List<String> list = Arrays.asList("hello", "world", "welcome", "hello", "a", "b", "c"
                    , "d", "e", "f", "g");
            Set<String> set = new HashSet<>();
            set.addAll(list);

            System.out.println("set: " + set);
            Map<String, String> map = set.parallelStream().collect(new MySetCollector2<>());
            System.out.println(map);
        }
    }
}
```



#### Collectors 类源码分析

对于 Collectors 静态工厂类来说，其实现一共分为两种情况：

- 通过 CollectorImpl 来实现
- 通过 reducing 方法来实现，reducing 方法本身又是通过 CollectorImpl 实现的。



基本方法除外，主要分析以下方法

- `groupingBy()` 通过三个重载方法调用来实现。

    ```java
    public static <T, K> Collector<T, ?, Map<K, List<T>>>
        groupingBy(Function<? super T, ? extends K> classifier) {
        return groupingBy(classifier, toList());
    }
    ```

    返回值为：`Collector<T, ?, Map<K, List<T>>>`，参数分别为：输入类型，中间类型（？表示都行），结果类型。return 语句调用的方法为：

    ```java
    // T 是流中元素类型，K 是分类器返回的结果的类型(返回的 Map 的 key)，D 是返回结果的值的类型。
    public static <T, K, A, D>
        Collector<T, ?, Map<K, D>> groupingBy(Function<? super T, ? extends K> classifier, // 分类器函数   
                                              Collector<? super T, A, D> downstream // 收集器对象
                                             ) {
        return groupingBy(classifier, HashMap::new, downstream);
    }
    
    ```

     return 调用的方法为：与上面区别：可以定义工厂，不一定使用 HashMap

    入参：

    `Function<? super T, ? extends K> classifier,`：Function 类型的分类器对象，输入为 T，返回为 K。
    `Supplier<M> mapFactory,`：默认就是 HashMap
    `Collector<? super T, A, D> downstream`：输入 T类型，中间累加器为 A 类型，返回 D 类型。

    ```java
    public static <T, K, D, A, M extends Map<K, D>>
        Collector<T, ?, M> groupingBy(Function<? super T, ? extends K> classifier,
                                      Supplier<M> mapFactory,
                                      Collector<? super T, A, D> downstream) {
        Supplier<A> downstreamSupplier = downstream.supplier();
        BiConsumer<A, ? super T> downstreamAccumulator = downstream.accumulator();
        BiConsumer<Map<K, A>, T> accumulator = (m, t) -> {
            // 分类器调用执行完的 K 类型就是分类的依据，即 Map 的 key
            K key = Objects.requireNonNull(classifier.apply(t), "element cannot be mapped to a null key"); 
            // A 是中间容器类型，是一个可变类型
            A container = m.computeIfAbsent(key, k -> downstreamSupplier.get());
            downstreamAccumulator.accept(container, t);
        };
        // 完成两个 Map 合并
        BinaryOperator<Map<K, A>> merger = Collectors.<K, A, Map<K, A>>mapMerger(downstream.combiner());
        @SuppressWarnings("unchecked")
        Supplier<Map<K, A>> mangledFactory = (Supplier<Map<K, A>>) mapFactory;
    
        if (downstream.characteristics().contains(Collector.Characteristics.IDENTITY_FINISH)) {
            return new CollectorImpl<>(mangledFactory, accumulator, merger, CH_ID);
        }
        else {
            @SuppressWarnings("unchecked")
            Function<A, A> downstreamFinisher = (Function<A, A>) downstream.finisher();
            Function<Map<K, A>, M> finisher = intermediate -> {
                intermediate.replaceAll((k, v) -> downstreamFinisher.apply(v));
                @SuppressWarnings("unchecked")
                M castResult = (M) intermediate;
                return castResult;
            };
            return new CollectorImpl<>(mangledFactory, accumulator, merger, finisher, CH_NOID);
        }
    }
    ```

- `groupingByConcurrent()` 是支持并发的，然后源码上将上面的 HashMap 替换为了 ConcurrentMap，当然最复杂的重载方法中有 synchronized 方法。具体看源码。



- `partitioningBy()` 分区方法。当然也有重载方法。





### 回归 Stream 

旁支知识点：

- `AutoCloseable` 接口：实现资源的自动关闭。

    ```java
    package com.gjxaiou.jdk8.streamSourceCode;
    
    public class AutoCloseableTest implements AutoCloseable {
    
        public void doSomething() {
            System.out.println("doSomething invoked!");
        }
    
        @Override
        public void close() throws Exception {
            System.out.println("close invoked!");
        }
    
        public static void main(String[] args) throws Exception {
            // close() 会被自动调用
            try (AutoCloseableTest autoCloseableTest = new AutoCloseableTest()) {
                autoCloseableTest.doSomething();
            }
            /**
             * 之前的方式
             */
            try {
    
            } catch () {
    
            } finally {
                // 显示关闭资源
            }
        }
    }
    ```

- 定义

    `public interface Stream<T> extends BaseStream<T, Stream<T>> {}`

- 首先分析 BaseStream

    ```
    public interface BaseStream<T, S extends BaseStream<T, S>>
            extends AutoCloseable {
    ```

    对 BaseStream 中的方法分析测试

    ```java
    /**
    	Returns an equivalent stream with an additional close handler. Close handlers are run when the close() method is called on the stream, and are executed in the order they were added. All close handlers are run, even if earlier close handlers throw exceptions. If any close handler throws an exception, the first exception thrown will be relayed to the caller of close(), with any remaining exceptions added to that exception as suppressed exceptions (unless one of the remaining exceptions is the same exception as the first exception, since an exception cannot suppress itself.) May return itself.
    */
    S onClose(Runnable closeHandler);
    ```

    测试程序：主要针对上面的 JavaDoc 文档进行测试

    ```java
    package com.gjxaiou.jdk8.streamSourceCode;
    
    
    import java.util.Arrays;
    import java.util.List;
    import java.util.stream.Stream;
    
    public class StreamTest2 {
    
        public static void main(String[] args) {
            List<String> list = Arrays.asList("hello", "world", "hello world");
    
            NullPointerException nullPointerException = new NullPointerException("my exception");
    
            try (Stream<String> stream = list.stream()) {
                stream.onClose(() -> {
                    System.out.println("aaa");
                    // 情况一：  某一个关闭处理器抛出异常，不妨碍其他关闭处理器执行。如果抛出多个异常，第一个异常会传递给调用者，后面的异常都是压制的异常。
                    // throw new NullPointerException("first exception");
                    // 情况二：都抛出一样的异常：如果剩余的异常和抛出的第一个异常是同一个异常，则不会不压制，因为一个异常不能压制自己。
                    // throw nullPointerException;
                    // 情况三：这是两个异常，第一个会被抛出，然后第二个会被压制。
                    throw new NullPointerException("first exception");
                }).onClose(() -> {
                    System.out.println("bbb");
                    // throw new ArithmeticException("second exception");
                    // throw nullPointerException;
                    throw new NullPointerException("second exception");
                }).forEach(System.out::println);
            }
        }
    }
    ```

    #### Spliterator 源码分析

    Spliterator 是接口，Spliterators 是工具类，Collector 是接口，Collectors 是工具类







**Consumer 和 IntConsumer 区别**

```java
package com.gjxaiou.jdk8.streamSourceCode;

import java.util.function.Consumer;
import java.util.function.IntConsumer;

public class ConsumerTest {

    public void test(Consumer<Integer> consumer) {
        consumer.accept(100);
    }

    public static void main(String[] args) {
        ConsumerTest consumerTest = new ConsumerTest();

        Consumer<Integer> consumer = i -> System.out.println(i);
        IntConsumer intConsumer = i -> System.out.println(i);

        System.out.println(consumer instanceof Consumer);
        System.out.println(intConsumer instanceof IntConsumer);

        consumerTest.test(consumer); //面向对象方式
        consumerTest.test(consumer::accept); //函数式方式
        consumerTest.test(intConsumer::accept); //函数式方式
    }
}
```

## 六、时间与日期 API

### （一）第三方库：Joda -time

```java
package com.gjxaiou.joda;

import org.joda.time.DateTime;
import org.joda.time.LocalDate;

public class JodaTest2 {
    public static void main(String[] args) {
        // 获取今天时间
        DateTime today = new DateTime();
        // 获取明天时间
        DateTime tomorrow = today.plusDays(1);

        // 输出：2021-09-02
        System.out.println(today.toString("yyyy-MM-dd"));
        // 输出：2021-09-02 12:23:23
        System.out.println(today.toString("yyyy-MM-dd HH:mm:ss"));
        // 输出：2021-09-03
        System.out.println(tomorrow.toString("yyyy-MM-dd"));

        // 获取月份第一天
        DateTime d1 = today.withDayOfMonth(1);
        // 输出：2021-09-01
        System.out.println(d1.toString("yyyy-MM-dd"));

        // 当前日期
        LocalDate localDate = new LocalDate();
        // 输出：2021-09-02
        System.out.println(localDate);

        // 距离当前日期的后三个月的第一天时间。
        localDate = localDate.plusMonths(3).dayOfMonth().withMinimumValue();
        // 输出：2021-12-01
        System.out.println(localDate);

        // 计算 2 年前第 3 个月最后 1 天的日期
        DateTime dateTime = new DateTime();
        DateTime dateTime2 = dateTime.minusYears(2).monthOfYear().
                setCopy(3).dayOfMonth().withMinimumValue();
        // 输出：2019-03-01
        System.out.println(dateTime2.toString("yyyy-MM-dd"));
    }
}
```

测试二：

```java
package com.gjxaiou.joda;


import org.joda.time.DateTime;
import org.joda.time.DateTimeZone;
import org.joda.time.format.DateTimeFormat;

import java.util.Date;

public class JodaTest3 {
    /**
     * 给定字符串类型的 UTC 时间转换为标准日期类型
     */
    // 标准 UTC 时间：2014-11-04T09:22:54.876Z   后面 Z 表示没有时区
    public static Date convertUTC2Date(String utcDate) {
        try {
            DateTime dateTime = DateTime.parse(utcDate, DateTimeFormat.forPattern("yyyy-MM-dd'T" +
                    "'HH:mm:ss.SSSZ"));
            return dateTime.toDate();
        } catch (Exception ex) {
            return null;
        }
    }

    // 给定日期类型，转换为 UTC 类型
    public static String convertDate2UTC(Date javaDate) {
        DateTime dateTime = new DateTime(javaDate, DateTimeZone.UTC);
        return dateTime.toString();
    }

    public static String convertDate2LocalByDateFormat(Date javaDate, String dateFormat) {
        DateTime dateTime = new DateTime(javaDate);
        return dateTime.toString(dateFormat);
    }

    public static void main(String[] args) {
        System.out.println(JodaTest3.convertUTC2Date("2014-11-04T09:22:54.876Z"));
        System.out.println(JodaTest3.convertDate2UTC(new Date()));
        System.out.println(JodaTest3.convertDate2LocalByDateFormat(new Date(), "yyyy-MM-dd " +
                "HH:mm:ss"));
    }
}
/**
 * Tue Nov 04 17:22:54 CST 2014  // 转换之后是有时区的
 * 2020-11-01T11:17:15.946Z
 * 2020-11-01 19:17:15
 */
```

### （二）Java8 中时间 API

**Joda Time 和 Java 8 中所有的日期和时间的 API 都是不可变对象，每次创建的时候都会返回全新的对象**，所以是线程安全的。

```java
package com.gjxaiou.joda;

import java.time.*;
import java.time.temporal.ChronoUnit;
import java.util.Set;
import java.util.TreeSet;

public class Java8TimeTest {

    public static void main(String[] args) {
        // LocalDate 是没有时区的时间。
        LocalDate localDate = LocalDate.now();
        // 输出：2023-02-17
        System.out.println(localDate);
        // 输出：2023, FEBRUARY: 2, 17
        System.out.println(localDate.getYear() + ", " + localDate.getMonth() + ": " + localDate.getMonthValue() + ", " + localDate.getDayOfMonth());

        // 根据年月日构造时间
        LocalDate localDate2 = LocalDate.of(2017, 3, 3);
        // 输出：2017-03-03
        System.out.println(localDate2);

        LocalDate localDate3 = LocalDate.of(2010, 3, 25);
        // MonthDay 是只关注月和日，可以用于一些重复式的。
        MonthDay monthDay = MonthDay.of(localDate3.getMonth(), localDate3.getDayOfMonth());
        MonthDay monthDay2 = MonthDay.from(LocalDate.of(2011, 3, 26));
        // 输出：not equals
        if (monthDay.equals(monthDay2)) {
            System.out.println("equals");
        } else {
            System.out.println("not equals");
        }

        LocalTime time = LocalTime.now();
        // 输出：00:31:24.847
        System.out.println(time);
        LocalTime time2 = time.plusHours(3).plusMinutes(20);
        // 输出：03:51:24.847
        System.out.println(time2);

        // 当前日期的下 2 周
        LocalDate localDate1 = localDate.plus(2, ChronoUnit.WEEKS);
        // 输出：2023-03-03
        System.out.println(localDate1);
        
        // 当前日期的前 2 月
        LocalDate localDate4 = localDate.minus(2, ChronoUnit.MONTHS);
        // 输出：2022-12-17
        System.out.println(localDate4);

        // 电脑默认时区
        Clock clock = Clock.systemDefaultZone();
        // 输出：1676565084847
        System.out.println(clock.millis());

        // 判断当前日期和指定日期之间的关系
        LocalDate localDate5 = LocalDate.now();
        LocalDate localDate6 = LocalDate.of(2017, 3, 18);

        // 下面三个输出：true  false  false
        System.out.println(localDate5.isAfter(localDate6));
        System.out.println(localDate5.isBefore(localDate6));
        System.out.println(localDate5.equals(localDate6));
        System.out.println("-------");

        // 时区列表
        Set<String> set = ZoneId.getAvailableZoneIds();
        Set<String> treeSet = new TreeSet<String>() {
            {
                addAll(set);
            }
        };
        
        /**
         * 输出所有时区，示例如下：
         * Africa/Abidjan
		 * Africa/Accra
		 * Africa/Addis_Ababa
		 */
        treeSet.stream().forEach(System.out::println);
        System.out.println("-------");

        // 构造时区
        ZoneId zoneId = ZoneId.of("Asia/Shanghai");
        LocalDateTime localDateTime = LocalDateTime.now();
        // 输出：2023-02-17T00:31:24.862
        System.out.println(localDateTime);

        // 构造带有时区的时间
        ZonedDateTime zonedDateTime = ZonedDateTime.of(localDateTime, zoneId);
        // 输出：2023-02-17T00:31:24.862+08:00[Asia/Shanghai]
        System.out.println(zonedDateTime);
        System.out.println("-------");

        YearMonth yearMonth = YearMonth.now();
        // 输出：2023-02
        System.out.println(yearMonth);
        // 输出：28
        System.out.println(yearMonth.lengthOfMonth());
        // 是不是闰年
        // 输出：false
        System.out.println(yearMonth.isLeapYear());
        System.out.println("-------");

        YearMonth yearMonth1 = YearMonth.of(2016, 2);
        // 输出：2016-02
        System.out.println(yearMonth1);
        // 输出：29
        System.out.println(yearMonth1.lengthOfMonth());
        // 输出：366
        System.out.println(yearMonth1.lengthOfYear());
        // 输出：true
        System.out.println(yearMonth1.isLeapYear());
        System.out.println("-------");

        // 求时间间隔
        LocalDate localDate7 = LocalDate.now();
        LocalDate localDate8 = LocalDate.of(2021, 3, 16);

        Period period = Period.between(localDate7, localDate8);
        // 输出：间隔 -1 年 -11 月 -1 天
        System.out.println("间隔 " + period.getYears() + " 年 " + period.getMonths() + " 月 " + period.getDays() + " 天 ");
        System.out.println("-------");

        // 输出：2023-02-16T16:31:24.864Z
        System.out.println(Instant.now());
    }
}
```

传统的时间日期 API 包括：Date/Calendar/DateFormat。Java 8 借鉴第三方优秀开源库 Joda-time，重新设计了一套 API。

### 表示时刻的 Instant

Instant 和 Date 一样，表示一个时间戳，用于描述一个时刻，只不过它较 Date 而言，可以描述更加精确的时刻。并且 Instant 是时区无关的。

Date 最多可以表示毫秒级别的时刻，而 Instant 可以表示纳秒级别的时刻。例如：

- public static Instant now()：根据系统当前时间创建一个 Instant 实例，表示当前时刻
- public static Instant ofEpochSecond(long epochSecond)：通过传入一个标准时间的偏移值来构建一个 Instant 实例
- public static Instant ofEpochMilli(long epochMilli)：通过毫秒数值直接构建一个 Instant 实例

```java
package com.gjxaiou.joda.instant;

import java.time.Instant;

public class InstantTest {
	public static void main(String[] args) {
		//创建 Instant 实例
		Instant instant = Instant.now();
		// 输出：2021-09-01T16:29:34.905Z
		System.out.println(instant);

		Instant instant1 = Instant.ofEpochSecond(20);
		// 输出：1970-01-01T00:00:20Z
		System.out.println(instant1);

		Instant instant2 = Instant.ofEpochSecond(30, 100);
		// 输出：1970-01-01T00:00:30.000000100Z
		System.out.println(instant2);

		Instant instant3 = Instant.ofEpochMilli(1000);
		// 输出：1970-01-01T00:00:01Z
		System.out.println(instant3);
	}
}
```

可以看到，Instant 和 Date 不同的是，它是时区无关的，始终是格林零时区相关的，也即是输出的结果始终格林零时区时间。

### 处理日期的 LocalDate

不同于 Calendar 既能处理日期又能处理时间，java.time 的新式 API 分离开日期和时间，用单独的类进行处理。LocalDate 专注于处理日期相关信息。

LocalDate 依然是一个不可变类，它关注时间中年月日部分，我们可以通过以下的方法构建和初始化一个 LocalDate 实例：

- public static LocalDate now()：截断当前系统时间的年月日信息并初始化一个实例对象
- public static LocalDate of(int year, int month, int dayOfMonth)：显式指定年月日信息
- public static LocalDate ofYearDay(int year, int dayOfYear)：根据 dayOfYear 可以推出 month 和 dayOfMonth
- public static LocalDate ofEpochDay(long epochDay)：相对于格林零时区时间的日偏移量

```java
package com.gjxaiou.joda.localDate;

import java.time.LocalDate;

public class LocalDateTest {
	public static void main(String[] args) {
		// 构建 LocalDate 实例
		LocalDate localDate = LocalDate.now();
		// 输出：2021-09-02
		System.out.println(localDate);

		LocalDate localDate1 = LocalDate.of(2017, 7, 22);
		// 输出：2017-07-22
		System.out.println(localDate1);

		LocalDate localDate2 = LocalDate.ofYearDay(2018, 100);
		// 输出：2018-04-10
		System.out.println(localDate2);

		LocalDate localDate3 = LocalDate.ofEpochDay(10);
		// 输出：1970-01-11
		System.out.println(localDate3);
	}
}
```

**需要注意一点，LocalDate 会根据系统中当前时刻和默认时区计算出年月日的信息。**

除此之外，LocalDate 中还有大量关于日期的常用方法：

- public int getYear()：获取年份信息
- public int getMonthValue()：获取月份信息
- public int getDayOfMonth()：获取当前日是这个月的第几天
- public int getDayOfYear()：获取当前日是这一年的第几天
- public boolean isLeapYear()：是否是闰年
- public int lengthOfYear()：获取这一年有多少天
- public DayOfWeek getDayOfWeek()：返回星期信息

### 处理时间的 LocalTime

类似于 LocalDate，LocalTime 专注于时间的处理，它提供小时，分钟，秒，毫微秒的各种处理，我们依然可以通过类似的方式创建一个 LocalTime 实例。

- public static LocalTime now()：根据系统当前时刻获取其中的时间部分内容
- public static LocalTime of(int hour, int minute)：显式传入小时和分钟来构建一个实例对象
- public static LocalTime of(int hour, int minute, int second)：通过传入时分秒构造实例
- public static LocalTime of(int hour, int minute, int second, int nanoOfSecond)：传入时分秒和毫微秒构建一个实例
- public static LocalTime ofSecondOfDay(long secondOfDay)：传入一个长整型数值代表当前日已经过去的秒数
- public static LocalTime ofNanoOfDay(long nanoOfDay)：传入一个长整型代表当前日已经过去的毫微秒数

同样的，LocalTime 默认使用系统默认时区处理时间：

```java
package com.gjxaiou.joda.localTime;

import java.time.LocalTime;

public class LocalTimeTest {
	public static void main(String[] a) {
		LocalTime localTime = LocalTime.now();
		// 输出：00:36:37.187
		System.out.println(localTime);

		LocalTime localTime1 = LocalTime.of(23, 59);
		// 输出：23:59
		System.out.println(localTime1);

		LocalTime localTime2 = LocalTime.ofSecondOfDay(10);
		// 输出：00:00:10
		System.out.println(localTime2);
	}
}
```

当然，LocalTime 中也同样封装了很多好用的工具方法，例如：

- public int getHour()
- public int getMinute()
- public int getSecond()
- public int getNano()
- public LocalTime withHour(int hour)：修改当前 LocalTime 实例中的 hour 属性并重新返回一个新的实例
- public LocalTime withMinute(int minute)：类似
- public LocalTime withSecond(int second)

LocalDateTime 类则是集成了 LocalDate 和 LocalTime，它既能表示日期，又能表述时间信息，方法都类似，只是有一部分涉及时区的转换内容，我们待会说。

### 时区相关的日期时间处理 ZonedDateTime

无论是我们的 LocalDate，或是 LocalTime，甚至是 LocalDateTime，它们基本是时区无关的，内部并没有存储时区属性，而基本用的系统默认时区。往往有些场景之下，缺乏一定的灵活性。

ZonedDateTime 可以被理解为 LocalDateTime 的外层封装，它的内部存储了一个 LocalDateTime 的实例，专门用于普通的日期时间处理。此外，它还定义了 ZoneId 和 ZoneOffset 来描述时区的概念。

ZonedDateTime 和 LocalDateTime 的一个很大的不同点在于，后者内部并没有存储时区，所以对于系统的依赖性很强，往往换一个时区可能就会导致程序中的日期时间不一致。

而后者则可以通过传入时区的名称，使用 ZoneId 进行匹配存储，也可以通过传入与零时区的偏移量，使用 ZoneOffset 存储时区信息。

所以，构建一个 ZonedDateTime 实例有以下几种方式：

- public static ZonedDateTime now()：系统将以默认时区计算并存储日期时间信息
- public static ZonedDateTime now(ZoneId zone)：指定时区
- public static ZonedDateTime of(LocalDate date, LocalTime time, ZoneId zone)：指定日期时间和时区
- public static ZonedDateTime of(LocalDateTime localDateTime, ZoneId zone)
- public static ZonedDateTime ofInstant(Instant instant, ZoneId zone)：通过时刻和时区构建实例对象

```java
public static void main(String[] a){
    ZonedDateTime zonedDateTime = ZonedDateTime.now();
    System.out.println(zonedDateTime);

    LocalDateTime localDateTime = LocalDateTime.now();
    ZoneId zoneId = ZoneId.of("America/Los_Angeles");
    ZonedDateTime zonedDateTime1 = ZonedDateTime.of(localDateTime,zoneId);
    System.out.println(zonedDateTime1);

    Instant instant = Instant.now();
    ZoneId zoneId1 = ZoneId.of("GMT");
    ZonedDateTime zonedDateTime2 = ZonedDateTime.ofInstant(instant,zoneId1);
    System.out.println(zonedDateTime2);
}

```

输出结果：

```
2018-04-23T16:10:29.510+08:00[Asia/Shanghai]
2018-04-23T16:10:29.511-07:00[America/Los_Angeles]
2018-04-23T08:10:29.532Z[GMT]
```

简单解释一下，首先第一个输出应该没什么问题，系统保存当前系统日期和时间以及默认的时区。

第二个小例子，LocalDateTime 实例保存了时区无关的当前日期时间信息，也就是这里的年月日时分秒，接着构建一个 ZonedDateTime 实例并传入一个美国时区（西七区）。你会发现输出的日期时间为西七区的 16 点 29 分。

像这种关联了时区的日期时间就很能够解决那种，换时区导致程序中时间错乱的问题。因为我关联了时区，无论你程序换到什么地方运行了，**日期+时区** 本就已经唯一确定了某个时刻，**就相当于我在存储某个时刻的时候，我说明了这是某某时区的某某时间，即便你换了一个地区，你也不至于把这个时间按自己当前的时区进行解析并直接使用了吧。**

第三个小例子就更加的直接明了了，构建 ZonedDateTime 实例的时候，给定一个时刻和一个时区，而这个时刻值就是相对于给定时区的标准时间所经过的毫秒数。

有关 ZonedDateTime 的其他日期时间的处理方法和 LocalDateTime 是一样的，因为 ZonedDateTime 是直接封装了一个 LocalDateTime 实例对象，所以所有相关日期时间的操作都会间接的调用 LocalDateTime 实例的方法，我们不再赘述。

### 格式化日期时间

Java 8 的新式日期时间 API 中，DateTimeFormatter 作为格式化日期时间的主要类，它与之前的 DateFormat 类最大的不同就在于它是线程安全的，其他的使用上的操作基本类似。我们看看：

```java
public static void main(String[] a){
    DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy年MM月dd日 HH:mm:ss");
    LocalDateTime localDateTime = LocalDateTime.now();
    System.out.println(formatter.format(localDateTime));

    String str = "2008年08月23日 23:59:59";
    DateTimeFormatter formatter2 = DateTimeFormatter.ofPattern("yyyy年MM月dd日 HH:mm:ss");
    LocalDateTime localDateTime2 = LocalDateTime.parse(str,formatter2);
    System.out.println(localDateTime2);

}
```

输出结果：

```
2018年04月23日 17:27:24
2008-08-23T23:59:59
```

格式化主要有两种情况，一种是将日期时间格式化成字符串，另一种则是将格式化的字符串装换成日期时间对象。

DateTimeFormatter 提供将 format 方法将一个日期时间对象转换成格式化的字符串，但是反过来的操作却建议使用具体的日期时间类自己的 parse 方法，这样可以省去类型转换的步骤。

### 时间差

现实项目中，我们也经常会遇到计算两个时间点之间的差值的情况，最粗暴的办法是，全部幻化成毫秒数并进行减法运算，最后在转换回日期时间对象。

但是 java.time 包中提供了两个日期时间之间的差值的计算方法，我们一起看看。

关于时间差的计算，主要涉及到两个类：

- Period：处理两个日期之间的差值
- Duration：处理两个时间之间的差值

例如：

```java
public static void main(String[] args){
    LocalDate date = LocalDate.of(2017,7,22);
    LocalDate date1 = LocalDate.now();
    Period period = Period.between(date,date1);
    System.out.println(period.getYears() + "年" +
            period.getMonths() + "月" +
            period.getDays() + "天");

    LocalTime time = LocalTime.of(20,30);
    LocalTime time1 = LocalTime.of(23,59);
    Duration duration = Duration.between(time,time1);
    System.out.println(duration.toMinutes() + "分钟");
}
```

输出结果：

```
0年9月1天
209分钟
```

显然，年月日的日期间差值的计算使用 Period 类足以，而时分秒毫秒的时间的差值计算则需要使用 Duration 类。

最后，关于 java.time 包下的新式日期时间 API，我们简单的学习了下，并没有深入到源码实现层次进行介绍，因为底层涉及大量的系统接口，涉及到大量的抽象类和实现类，有兴趣的朋友可以自行阅读 jdk 的源码深入学习。
