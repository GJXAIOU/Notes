# Java 枚举(enum) 详解7种常见的用法
[原文地址链接](https://blog.csdn.net/qq_27093465/article/details/52180865)

@toc

JDK1.5引入了新的类型——枚举。在 Java 中它虽然算个“小”功能，却给我的开发带来了“大”方便。


## 用法一：常量
在JDK1.5 之前，我们定义常量都是： public static final.... 。通过枚举可以把相关的常量分组到一个枚举类型里，而且枚举提供了比常量更多的方法。 
```java
public enum Color {
    RED, GREEN, BLANK, YELLOW  
} 
```

## 用法二：switch
switch语句使用枚举，能让我们的代码可读性更强。 
```java
enum SignalEnum {  
    GREEN, YELLOW, RED  
}  
public class TrafficLight {  
    SignalEnum color = SignalEnum.RED;  
    public void change() {  
        switch (color) {  
        case RED:  
            color = SignalEnum.GREEN;  
            break;  
        case YELLOW:  
            color = SignalEnum.RED;  
            break;  
        case GREEN:  
            color = SignalEnum.YELLOW;  
            break;  
        }  
    }  
}  
```

## 用法三：向枚举中添加新方法
**如果打算自定义自己的方法，那么必须在enum实例序列的最后添加一个分号**。
而且 Java 要求必须先定义 enum 实例。 
```java
public enum Color {  
    RED("红色", 1), GREEN("绿色", 2), BLANK("白色", 3), YELLO("黄色", 4);  

    private String name;  
    private int index;  
  
    private Color(String name, int index) {  
        this.name = name;  
        this.index = index;  
    }  
    public String getName() {  
        return name;  
    }  
    public void setName(String name) {  
        this.name = name;  
    }  
    public int getIndex() {  
        return index;  
    }  
    public void setIndex(int index) {  
        this.index = index;  
    }  

    public static String getName(int index) {  
        for (Color c : Color.values()) {  
            if (c.getIndex() == index) {  
                return c.name;  
            }  
        }  
        return null;  
    }  
    
}  
```

## 用法四：覆盖枚举的方法
下面给出一个toString()方法覆盖的例子。 
```java
public enum Color {  
    RED("红色", 1), GREEN("绿色", 2), BLANK("白色", 3), YELLO("黄色", 4);  
  
    private String name;  
    private int index;  
    // 构造方法 
    private Color(String name, int index) {  
        this.name = name;  
        this.index = index;  
    }  
    //覆盖方法  
    @Override  
    public String toString() {  
        return this.index+"_"+this.name;  
    }  
}  
```

## 用法五：实现接口

所有的枚举都继承自java.lang.Enum类。由于Java 不支持多继承，所以枚举对象不能再继承其他类。 
```
public interface Behaviour {  
    void print();  
    String getInfo();  
}  
public enum Color implements Behaviour{  
    RED("红色", 1), GREEN("绿色", 2), BLANK("白色", 3), YELLO("黄色", 4);  
    // 成员变量  
    private String name;  
    private int index;  
    // 构造方法  
    private Color(String name, int index) {  
        this.name = name;  
        this.index = index;  
    }  
//接口方法  
    @Override  
    public String getInfo() {  
        return this.name;  
    }  
    //接口方法  
    @Override  
    public void print() {  
        System.out.println(this.index+":"+this.name);  
    }  
}  
```

## 用法六：使用接口组织枚举
```java
public interface Food {  
    enum Coffee implements Food{  
        BLACK_COFFEE,DECAF_COFFEE,LATTE,CAPPUCCINO  
    }  
    enum Dessert implements Food{  
        FRUIT, CAKE, GELATO  
    }  
}  
```

```java
       /**
     * 测试继承接口的枚举的使用
     */
    private static void testImplementsInterface() {
        for (Food.DessertEnum dessertEnum : Food.DessertEnum.values()) {
            System.out.print(dessertEnum + "  ");
        }
        System.out.println();
        //我这地方这么写，是因为我在自己测试的时候，把这个coffee单独到一个文件去实现那个food接口，而不是在那个接口的内部。
        for (CoffeeEnum coffee : CoffeeEnum.values()) {
            System.out.print(coffee + "  ");
        }
        System.out.println();
        //搞个实现接口，来组织枚举，简单讲，就是分类吧。如果大量使用枚举的话，这么干，在写代码的时候，就很方便调用啦。
        //还有就是个“多态”的功能吧，
        Food food = Food.DessertEnum.CAKE;
        System.out.println(food);
        food = CoffeeEnum.BLACK_COFFEE;
        System.out.println(food);
    }
```
运行结果
```
FRUIT CAKE GELATO
BLACK_COFFEE DECAF_COFFEE LATTE CAPPUCCINO
CAKE
BLACK_COFFEE
```

## 用法七：关于枚举集合的使用

java.util.EnumSet和java.util.EnumMap是两个枚举集合。EnumSet保证集合中的元素不重复；EnumMap中的 key是enum类型，而value则可以是任意类型。关于这个两个集合的使用就不在这里赘述，可以参考JDK文档。

**下面我把自己的使用理解给整理一下。**

**下面是我自己的测试代码。**

```
package com.lxk.enumTest;
 
/**
 * Java枚举用法测试
 * <p>
 * Created by lxk on 2016/12/15
 */
public class EnumTest {
    public static void main(String[] args) {
        forEnum();
        useEnumInJava();
    }
 
    /**
     * 循环枚举,输出ordinal属性；若枚举有内部属性，则也输出。(说的就是我定义的TYPE类型的枚举的typeName属性)
     */
    private static void forEnum() {
        for (SimpleEnum simpleEnum : SimpleEnum.values()) {
            System.out.println(simpleEnum + "  ordinal  " + simpleEnum.ordinal());
        }
        System.out.println("------------------");
        for (TYPE type : TYPE.values()) {
            System.out.println("type = " + type + "    type.name = " + type.name() + "   typeName = " + type.getTypeName() + "   ordinal = " + type.ordinal());
        }
    }
 
    /**
     * 在Java代码使用枚举
     */
    private static void useEnumInJava() {
        String typeName = "f5";
        TYPE type = TYPE.fromTypeName(typeName);
        if (TYPE.BALANCE.equals(type)) {
            System.out.println("根据字符串获得的枚举类型实例跟枚举常量一致");
        } else {
            System.out.println("大师兄代码错误");
        }
 
    }
 
    /**
     * 季节枚举(不带参数的枚举常量)这个是最简单的枚举使用实例
     * Ordinal 属性，对应的就是排列顺序，从0开始。
     */
    private enum SimpleEnum {
        SPRING,
        SUMMER,
        AUTUMN,
        WINTER
    }
 
 
    /**
     * 常用类型(带参数的枚举常量，这个只是在书上不常见，实际使用还是很多的，看懂这个，使用就不是问题啦。)
     */
    private enum TYPE {
        FIREWALL("firewall"),
        SECRET("secretMac"),
        BALANCE("f5");
 
        private String typeName;
 
        TYPE(String typeName) {
            this.typeName = typeName;
        }
 
        /**
         * 根据类型的名称，返回类型的枚举实例。
         *
         * @param typeName 类型名称
         */
        public static TYPE fromTypeName(String typeName) {
            for (TYPE type : TYPE.values()) {
                if (type.getTypeName().equals(typeName)) {
                    return type;
                }
            }
            return null;
        }
 
        public String getTypeName() {
            return this.typeName;
        }
    }
}
```

**然后是测试的结果图：**
![测试结果图]($resource/%E6%B5%8B%E8%AF%95%E7%BB%93%E6%9E%9C%E5%9B%BE.jpg)

简单的例子，大家基本都用过，看不懂的基本都是第二个例子。可以看到，在第二个例子里面，后面带有参数，其实可以这么理解。

enum这个关键字，可以理解为跟class差不多，这也个单独的类。可以看到，上面的例子里面有属性，有构造方法，有getter，也可以有setter，但是一般都是构造传参数。还有其他自定义方法。那么在这些东西前面的，以逗号隔开的，最后以分号结尾的，这部分叫做，这个枚举的实例。也可以理解为，class  new 出来的实例对象。这下就好理解了。只是，class，new对象，可以自己随便new，想几个就几个，而这个enum关键字，他就不行，他的实例对象，只能在这个enum里面体现。也就是说，他对应的实例是有限的。这也就是枚举的好处了，限制了某些东西的范围，举个栗子：一年四季，只能有春夏秋冬，你要是字符串表示的话，那就海了去了，但是，要用枚举类型的话，你在enum的大括号里面把所有的选项，全列出来，那么这个季节的属性，对应的值，只能在里面挑。不能有其他的。

我上面的例子，就是根据typeName，你可以从那些例子里面挑选到唯一的一个TYPE类型的枚举实例--TYPE.BALANCE。注意方法

TYPE type = TYPE.fromTypeName(typeName);
这个方法的返回类型就是这个TYPE枚举类型的。
这下就好理解，这个枚举是怎么在工作了吧

再补充一下：

上面那个带参数的枚举类型的实例里面实际上是三个属性，除了我自定义的typeName以外，还有2个是系统自带的。看下面源码的图：

![](https://img-blog.csdn.net/20161222185233843)

看到这里之后，不知道你能不能理解下面图片内说明的话：下面图片主要说明在使用枚举时，的规范和标准。希望可以在实际开发时候用到

![](https://img-blog.csdn.net/20170112172420090)

![](https://img-blog.csdn.net/20170112172408793)

**最后补充一点：**

也许你知道呢，但是也许你不知道呢？我是真的不知道，测了之后才知道！！！

**枚举类型对象之间的值比较，是可以使用==，直接来比较值，是否相等的，不是必须使用equals方法的哟。**

具体，请参考下面的链接：

[java 枚举类比较是用==还是equals？](http://blog.csdn.net/qq_27093465/article/details/70237349)

**2017.11.07 更新**

有的老铁，说这个switch case怎么写，我就在下面再啰嗦一下。

```
 private static void testSwitchCase() {
    String typeName = "f5";
    //这几行注释呢，你可以试着三选一，测试一下效果。
    //String typeName = "firewall";
    //String typeName = "secretMac";
    TypeEnum typeEnum = TypeEnum.fromTypeName(typeName);
    if (typeEnum == null) {
        return;
    }
    switch (typeEnum) {
        case FIREWALL:
            System.out.println("枚举名称(即默认自带的属性 name 的值)是：" + typeEnum.name());
            System.out.println("排序值(默认自带的属性 ordinal 的值)是：" + typeEnum.ordinal());
            System.out.println("枚举的自定义属性 typeName 的值是：" + typeEnum.getTypeName());
            break;
        case SECRET:
            System.out.println("枚举名称(即默认自带的属性 name 的值)是：" + typeEnum.name());
            System.out.println("排序值(默认自带的属性 ordinal 的值)是：" + typeEnum.ordinal());
            System.out.println("枚举的自定义属性 typeName 的值是：" + typeEnum.getTypeName());
            break;
        case BALANCE:
            System.out.println("枚举名称(即默认自带的属性 name 的值)是：" + typeEnum.name());
            System.out.println("排序值(默认自带的属性 ordinal 的值)是：" + typeEnum.ordinal());
            System.out.println("枚举的自定义属性 typeName 的值是：" + typeEnum.getTypeName());
            break;
        default:
            System.out.println("default");
    }
}
```

然后，就是运行结果的截图。

![](https://img-blog.csdn.net/20171107142120698)

枚举，是个对象，就像定义的Student类，Person类，等等一些个类一样。因此可以有构造方法、属性以及其他方法；
枚举中有两个自带的默认属性：`name`和`ordinal`,其他属性可以自己添加；不能对系统自带的name属性，在构造函数里面赋值。

**补充知识：**
就是这个枚举类型，一旦创建，且被使用（比如，存数据库啥的）之后，持久化后的对象信息里面就保存了这个枚举信息。**应当禁止这个变更的操作，只能重新创建，用新的代替旧的，不能直接把旧的给改了**，因为，就数据在逆转成对象的时候，如果，旧的枚举不在了，那么就会400还是500的报错或者是空指针的bug。这也是需要关注的一个问题。希望注意下，不然等到出bug了再想到这个问题，就不好了。


