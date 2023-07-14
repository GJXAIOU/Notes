泛型的用途就是保证类型安全
在编译之后的字节码文件中会将泛型擦除. 这个目的是为了能让 java5 的代码在 java5 之前的版本中运行

往集合类中加入元素必须符合集合类的泛型要求



- ​	泛型用在类或接口上

  ```java
  /**
   * 用在类上保证在这个类中的方法上的参数都是 T 类型
   */
  class DemoClass<T> {
    private T t;
    public void set(T t) {
      this.t = t;
    }
    
    public T get() {
      return this.t;
    }
  }
  
  // 使用时可以保证传入参数类型和返回值类型都是定义 DemoClass 的类型
  DemoClass<String> demoClass = new DemoClass<String>();
  demoClass.set("String here");
  String t = demoClass.get();
  ```

  

- 泛型用在接口上是类似的

- ```java
  interface DemoInterface<T1, T2> {
    T2 doSomething(T1 t1);
    T1 doReverse(T2 t2);
  }
  ```



- 泛型用在方法上(包括构造方法)

```java
// <T> 表示此方法使用泛型
// T 表示方法返回 T 类型
// 
// 此处传入的数组和第二个参数类型应该是同样的
// 比如 String[], String
public <T> T doSomething(T[] tArray, T target) {
  // ...
}

public class DemoClass<T> {
  private T x;
  private T y;
  public DemoClass(T x, T y) {
    this.x = x;
    this.y = y;
  }
  
  public <E> DemoClass(E e) {
    // <E> 是泛型参数
    // 构造方法参数 E表示传入的类型需要是 E
    // 这种没有任何意义, 限制不了传入参数的类型, 所以构造方法的泛型应该配合类泛型来使用
  }
}
```



- 泛型用在数组上

  - 泛型数组不能直接初始化

    ```java
    public class DemoClass<T> {
      private T[] tArr; // This is fine
      private T[] initArray = new T[10]; // Compiler error
    }
    ```

  - 泛型数组是协变类型

    ```java
    Object[] objArr = new String[10];
    objArr[0] = "X";
    objArr[1] = 10; // Compiler error
    ```

- 泛型通配符 <?>

  - 用在 List —> List<?>. 表示可以接收任意类型的泛型 List. 只能 remove 和 clear. 但不能在这个 List<*?*> 中进行 add. 因为编译时不知道这个 list 中存的是什么类型. add 任何类型都不安全.
  - <? extends T> 任何 T 类型的子类都认为是合法的
  - <? super T>  任何 T 类型的父类都认为是合法的