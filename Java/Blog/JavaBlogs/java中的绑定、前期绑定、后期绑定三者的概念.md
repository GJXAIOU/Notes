# java中的绑定、前期绑定、后期绑定三者的概念
[原文地址](https://blog.csdn.net/q35445762/article/details/46863819)


绑定指的是一个**方法的调用与方法所在的类(方法主体)关联**起来。

前期绑定：在程序执行前方法已经被绑定，此时由编译器或其它连接程序实现。例如：C。

后期绑定：在运行时根据具体对象的类型进行绑定。

**在 Java中，几乎所有的方法都是后期绑定的**，在运行时动态绑定方法属于子类还是基类。但是也有特殊，**针对static方法和final方法**由于不能被继承，因此在编译时就可以确定他们的值，**他们是属于前期绑定的**。

特别说明的一点是，private 声明的方法和成员变量不能被子类继承，所有的**private方法都被隐式的指定为final的**(由此我们也可以知道：**将方法声明为final类型的一是为了防止方法被覆盖，二是为了有效的关闭java中的动态绑定**)。java 中的后期绑定是有 JVM 来实现的，我们不用去显式的声明它，而 C++则不同,必须明确的声明某个方法具备后期绑定。

精确使用的方法是编译器绑定，在编译阶段，最佳方法名依赖于参数的静态和控制引用的静态类型所适合的方法。在这一点上，设置方法的名称，这一步叫静态重载。
决定方法是哪一个类的版本，这通过由虚拟机推断出这个对象的运行时类型来完成，一旦知道运行时类型，虚拟机就唤起继承机制，寻找方法的最终版本。这叫做动态绑定。

**重载函数的实际调用版本由编译器绑定决定，而覆盖函数的实际调用版本由动态绑定决定**。 

在处理**java类中的成员变量（包括静态和非静态）**时，并不是采用运行时绑定，而**是一般意义上的静态绑定**。所以在向上转型的情况下，对象的方法可以找到子类，而对象的属性还是父类的属性。 

最简单的办法是将该成员变量封装成方法 getter 形式。



示例程序：
```java
class Base{ 
    //成员变量，子类也有同样的成员变量名
    public String test="Base Field"; 
    
    //静态方法，子类也有同样签名的静态方法
    public static void staticMethod(){
        System.out.println("Base staticMethod()");
    } 
    
    //子类将对此方法进行覆盖
    public void notStaticMethod(){
        System.out.println("Base notStaticMethod()");
    }

} 

public class Derive extends Base{ 
  public String test="Derive Field"; 
  public static void staticMethod(){
     System.out.println("Derive staticMethod()");
  }
   
   @Override 
   public void notStaticMethod(){
        System.out.println("Derive notStaticMethod()");
    } 
    
    //输出成员变量的值，验证其为前期绑定。
    public static void testFieldBind(Base base){
        System.out.println(base.test);
    } 
    
    //静态方法，验证其为前期绑定。
    public static void testStaticMethodBind(Base base){ 
    //使用Base.test()更加合理，这里为了更为直观的展示前期绑定才使用这种表示。
       base.staticMethod();
    } 
    
    //调用非静态方法，验证其为后期绑定。
    public static void testNotStaticMethodBind(Base base){
        base.notStaticMethod();
    } 
    
    public static void main(String[] args){
        Derive d=new Derive();
        testFieldBind(d);
        testStaticMethodBind(d);
        testNotStaticMethodBind(d);
    }
} 
/*程序输出:
Base Field
Base staticMethod()
Derive notStaticMethod() */
```
