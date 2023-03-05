# 60策略模式（上）：如何避免冗长的if-else/switch分支判断代码？

模板模式主要起到代码复用和扩展的作用。除此之外，我们还讲到了回调，它跟模板模式的作用类似，但使用起来更加灵活。它们之间的主要区别在于代码实现，**模板模式基于继承来实现，回调基于组合来实现。**

策略模式最常见的应用场景：利用它来避免冗长的 if-else 或 switch 分支判断。同时也可以像模板模式那样，提供框架的扩展点等等。

上文：解策略模式的原理和实现，以及如何用它来避免分支判断逻辑。

下文：通过一个具体的例子，来详细讲解策略模式的应用场景以及真正的设计意图。

## 一、策略模式的原理与实现

策略模式（Strategy Design Pattern）。在 GoF 的《设计模式》中定义如下：

> Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.
>
> 定义一族算法类，将每个算法分别封装起来，让它们可以互相替换。策略模式可以使算法的变化独立于使用它们的客户端（这里的客户端代指使用算法的代码）。

==工厂模式是解耦对象的创建和使用，观察者模式是解耦观察者和被观察者。**策略模式解耦的是策略的定义、创建、使用这三部分**。==

## 二、策略的定义

**策略类的定义比较简单，包含一个策略接口和一组实现这个接口的策略类。因为所有的策略类都实现相同的接口，所以，客户端代码基于接口而非实现编程，可以灵活地替换不同的策略**。示例代码如下所示：

```java
public interface Strategy {
    void algorithmInterface();
} 

public class ConcreteStrategyA implements Strategy {
    @Override
    public void algorithmInterface() {
        //具体的算法...
    }
} 

public class ConcreteStrategyB implements Strategy {
    @Override
    public void algorithmInterface() {
        //具体的算法...
    }
}
```

## 三、策略的创建

因为策略模式会包含一组策略，在使用它们的时候，一般会通过类型（type）来判断创建哪个策略来使用。为了封装创建逻辑，我们需要对客户端代码屏蔽创建细节。我们可以把根据 type 创建策略的逻辑抽离出来，放到工厂类中。示例代码如下所示：

```java
public class StrategyFactory {
    private static final Map<String, Strategy> strategies = new HashMap<>();
    static {
        strategies.put("A", new ConcreteStrategyA());
        strategies.put("B", new ConcreteStrategyB());
    }
    
    public static Strategy getStrategy(String type) {
        if (type == null || type.isEmpty()) {
            throw new IllegalArgumentException("type should not be empty.");
        }
        return strategies.get(type);
    }
}
```

如果策略类是无状态的，不包含成员变量，只是纯粹的算法实现，这样的策略对象是可以被共享使用的，不需要在每次调用 getStrategy() 的时候，都创建一个新的策略对象。针对这种情况，我们可以使用上面这种工厂类的实现方式，事先创建好每个策略对象，缓存到工厂类中，用的时候直接返回。

相反，如果策略类是有状态的，根据业务场景的需要，我们希望每次从工厂方法中，获得的都是新创建的策略对象，而不是缓存好可共享的策略对象，那我们就需要按照如下方式来实现策略工厂类。

```java
public class StrategyFactory {
    public static Strategy getStrategy(String type) {
        if (type == null || type.isEmpty()) {
            throw new IllegalArgumentException("type should not be empty.");
        } 
        if (type.equals("A")) {
            return new ConcreteStrategyA();
        } else if (type.equals("B")) {
            return new ConcreteStrategyB();
        } 
        return null;
    }
}
```

## 四、策略的使用

策略模式包含一组可选策略，客户端代码一般如何确定使用哪个策略呢？最常见的是**运行时动态确定使用哪种策略，这也是策略模式最典型的应用场景。**

「运行时动态」是指事先并不知道会使用哪个策略，而是在程序运行期间，根据配置、用户输入、计算结果等这些不确定因素，动态决定使用哪种策略。示例如下：

```java
// 策略接口：EvictionStrategy
// 策略类：LruEvictionStrategy、FifoEvictionStrategy、LfuEvictionStrategy...
// 策略工厂：EvictionStrategyFactory
public class UserCache {
    private Map<String, User> cacheData = new HashMap<>();
    private EvictionStrategy eviction;
    public UserCache(EvictionStrategy eviction) {
        this.eviction = eviction;
    } 
    //...
} 

// 运行时动态确定，根据配置文件的配置决定使用哪种策略
public class Application {
    public static void main(String[] args) throws Exception {
        EvictionStrategy evictionStrategy = null;
        Properties props = new Properties();
        props.load(new FileInputStream("./config.properties"));
        String type = props.getProperty("eviction_type");
        evictionStrategy = EvictionStrategyFactory.getEvictionStrategy(type);
        UserCache userCache = new UserCache(evictionStrategy);
        //...
    }
}

// 非运行时动态确定，在代码中指定使用哪种策略
public class Application {
    public static void main(String[] args) {
        //...
        EvictionStrategy evictionStrategy = new LruEvictionStrategy();
        UserCache userCache = new UserCache(evictionStrategy);
        //...
    }
}
```

第二个 Application 中的使用方式即为「非运行时动态确定」，并不能发挥策略模式的优势。在这种应用场景下，策略模式实际上退化成了“面向对象的多态特性”或“基于接口而非实现编程原则”。

## 五、如何利用策略模式避免分支判断？

**实际上，策略模式和状态模式都能够移除分支判断逻辑。对于使用哪种模式，具体还要看应用场景来定。 策略模式适用于根据不同类型待动态，决定使用哪种策略这样一种应用场景**。

先通过一个例子看下 if-else 或 switch-case 分支判断逻辑是如何产生的。在这个例子中，我们没有使用策略模式，而是将策略的定义、创建、使用直接耦合在一起。

```java
public class OrderService {
    public double discount(Order order) {
        double discount = 0.0;
        OrderType type = order.getType();
        if (type.equals(OrderType.NORMAL)) { // 普通订单
            //...省略折扣计算算法代码
        } else if (type.equals(OrderType.GROUPON)) { // 团购订单
            //...省略折扣计算算法代码
        } else if (type.equals(OrderType.PROMOTION)) { // 促销订单
            //...省略折扣计算算法代码
        }
        return discount;
    }
}
```

如何来移除掉分支判断逻辑呢？我们使用策略模式对上面的代码重构，将不同类型订单的打折策略设计成策略类，并由工厂类来负责创建策略对象。具体的代码如下所示：

```java
// 策略的定义
public interface DiscountStrategy {
    double calDiscount(Order order);
}

// 省略NormalDiscountStrategy、GrouponDiscountStrategy、PromotionDiscountStrateg
// 策略的创建
public class DiscountStrategyFactory {
    private static final Map<OrderType, DiscountStrategy> strategies = new HashMap<>();
        static {
        strategies.put(OrderType.NORMAL, new NormalDiscountStrategy());
        strategies.put(OrderType.GROUPON, new GrouponDiscountStrategy());
        strategies.put(OrderType.PROMOTION, new PromotionDiscountStrategy());
    } 
    public static DiscountStrategy getDiscountStrategy(OrderType type) {
        return strategies.get(type);
    }
} 

// 策略的使用
public class OrderService {
    public double discount(Order order) {
        OrderType type = order.getType();
        DiscountStrategy discountStrategy = DiscountStrategyFactory.getDiscountStrategy(type);
            return discountStrategy.calDiscount(order);
    }
}
```

重构之后的代码就没有了 if-else 分支判断语句了。实际上，这得益于策略工厂类。**在工厂类中，我们用 Map 来缓存策略，根据 type 直接从 Map 中获取对应的策略，从而避免 if- else 分支判断逻辑。后续的状态模式也使用同类方式来避免分支判断逻辑。本质上都是借助“查表法”，根据 type 查表（代码中的 strategies 就是表）替代根据 type 分支判断。**

但是，如果业务场景需要每次都创建不同的策略对象，我们就要用另外一种工厂类的实现方式了。具体的代码如下所示：

```java
public class DiscountStrategyFactory {
    public static DiscountStrategy getDiscountStrategy(OrderType type) {
        if (type == null) {
            throw new IllegalArgumentException("Type should not be null.");
        }
        if (type.equals(OrderType.NORMAL)) {
            return new NormalDiscountStrategy();
        } else if (type.equals(OrderType.GROUPON)) {
            return new GrouponDiscountStrategy();
        } else if (type.equals(OrderType.PROMOTION)) {
            return new PromotionDiscountStrategy();
        }
        return null;
    }
}
```

这种实现方式相当于把原来的 if-else 分支逻辑，从 OrderService 类中转移到了工厂类中，实际上并没有真正将它移除。关于这个问题如何解决，我今天先暂时卖个关子。你可以在留言区说说你的想法，我在下一节课中再讲解。

## 六、重点回顾

策略模式定义一族算法类，将每个算法分别封装起来，让它们可以互相替换。**策略模式可以使算法的变化独立于使用它们的客户端（这里的客户端代指使用算法的代码）。**

策略模式用来解耦策略的定义、创建、使用。实际上，一个完整的策略模式就是由这三个部分组成的。

策略类的定义比较简单，包含一个策略接口和一组实现这个接口的策略类。策略的创建由工厂类来完成，封装策略创建的细节。

策略模式包含一组策略可选，客户端代码如何选择使用哪个策略，有两种确定方法：编译时静态确定和运行时动态确定。其中，“运行时动态确定”才是策略模式最典型的应用场景。

除此之外，我们还可以通过策略模式来移除 if-else 分支判断。实际上，这得益于策略工厂类，更本质上点讲，是借助“查表法”，根据 type 查表替代根据 type 分支判断。

## 七、课堂讨论

在策略工厂类中，如果每次都要返回新的策略对象，我们还是需要在工厂类中编写 if-else 分支判断逻辑，那这个问题该如何解决呢？

**精选回答**：

- 仍然可以用查表法，只不过存储的不再是实例，而是 class，使用时获取对应的 class，再通过反射创建实例；

- 一般而言Java web开发中我们均使用spring框架，可以使用运行时自定义注解给具体的策略类打上注解，将具体的策略类放于spring 容器中，工厂中注入直接根据类型获取即可，不使用spring框架的话，也可以用Java的反射做到获取到具体的策略类；

- 思考题，“工厂的工厂”，对每个策略类都建立相应的工厂类，根据 type 查表得到工厂类，通过工厂类来创建新的策略对象。

- 对于课后思考题，可以使用反射实现。对于各个策略类，可以是用表记录。

    也在思考一个问题，对于争哥举的购物的例子，如果现实情况并非单一策略，而是不同策略的组合呢？例如我既满足满减同时也能使用优惠券呢？这种情况简单的解决方法就是再定一个新策略。至于策略组合爆炸的问题，显然不是所有策略的组合现实中都是合理的。

- 可用Spring的applicationContext.getBeansOfType(Interface.class);





# 61策略模式（下）：如何实现一个支持给不同大小文件排序的小程序？

本问题通过结合“给文件排序”示例，详述策略模式的设计意图和应用场景。

同时会通过一步一步地分析、重构，展示一个设计模式是如何“创造”出来的。通过今天的学习，会发现，**设计原则和思想其实比设计模式更加普适和重要，掌握了代码的设计原则和思想，我们甚至可以自己创造出来新的设计模式**。

## 一、问题与解决思路

假设有这样一个需求，希望写一个小程序，实现对一个文件进行排序的功能。文件中只包含整型数，并且，相邻的数字通过逗号来区隔。如果由你来编写这样一个小程序，你会如何来实现呢？你可以把它当作面试题，先自己思考一下，再来看我下面的讲解。

思考一：只需要将文件中的内容读取出来，并且通过逗号分割成一个一个的数字，放到内存数组中，然后编写某种排序算法（比如快排），或者直接使用编程语言提供的排序函数，对数组进行排序，最后再将数组中的数据写入文件就可以了。

但是，如果文件很大呢？比如有 10GB 大小，因为内存有限（比如只有 8GB 大小），我们没办法一次性加载文件中的所有数据到内存中，这个时候，我们就要利用外部排序算法（具体怎么做，可以参看我的另一个专栏《数据结构与算法之美》中的“排序”相关章节）了。

如果文件更大，比如有 100GB 大小，我们为了利用 CPU 多核的优势，可以在外部排序的基础之上进行优化，加入多线程并发排序的功能，这就有点类似“单机版”的 MapReduce。

如果文件非常大，比如有 1TB 大小，即便是单机多线程排序，这也算很慢了。这个时候，我们可以使用真正的 MapReduce 框架，利用多机的处理能力，提高排序的效率。

## 二、代码实现与分析

先用最简单直接的方式实现将它实现出来。下面的代码实现中，我只给出了跟设计模式相关的骨架代码，并没有给出每种排序算法的具体代码实现。

```java
public class Sorter {
    private static final long GB = 1000 * 1000 * 1000;
    public void sortFile(String filePath) {
        // 省略校验逻辑
        File file = new File(filePath);
        long fileSize = file.length();
        if (fileSize < 6 * GB) { // [0, 6GB)
            quickSort(filePath);
        } else if (fileSize < 10 * GB) { // [6GB, 10GB)
            externalSort(filePath);
        } else if (fileSize < 100 * GB) { // [10GB, 100GB)
            concurrentExternalSort(filePath);
        } else { // [100GB, ~)
            mapreduceSort(filePath);
        }
    } 
    
    private void quickSort(String filePath) {
        // 快速排序
    } 
    private void externalSort(String filePath) {
        // 外部排序
    } 
    private void concurrentExternalSort(String filePath) {
        // 多线程外部排序
    } 
    private void mapreduceSort(String filePath) {
        // 利用MapReduce多机排序
    }
}

public class SortingTool {
    public static void main(String[] args) {
        Sorter sorter = new Sorter();
        sorter.sortFile(args[0]);
    }
}
```

因为编码规范中要求函数的行数不能过多，最好不要超过一屏的大小。所以，为了避免 sortFile() 函数过长，我们把每种排序算法从 sortFile() 函数中抽离出来，拆分成 4 个独立的排序函数。

如果只是开发一个简单的工具，那上面的代码实现就足够了。毕竟，代码不多，后续修改、扩展的需求也不多，怎么写都不会导致代码不可维护。但是，如果我们是在开发一个大型项目，排序文件只是其中的一个功能模块，那我们就要在代码设计、代码质量上下点儿功夫了。只有每个小的功能模块都写好，整个项目的代码才能不差。

在刚刚的代码中，我们并没有给出每种排序算法的代码实现。实际上，如果自己实现一下的话，你会发现，每种排序算法的实现逻辑都比较复杂，代码行数都比较多。所有排序算法的代码实现都堆在 Sorter 一个类中，这就会导致这个类的代码很多。而在“编码规范”那一部分中，我们也讲到，一个类的代码太多也会影响到可读性、可维护性。除此之外，所有的排序算法都设计成 Sorter 的私有函数，也会影响代码的可复用性。

## 三、代码优化与重构

只要掌握了我们之前讲过的设计原则和思想，针对上面的问题，即便我们想不到该用什么设计模式来重构，也应该能知道该如何解决，那就是将 Sorter 类中的某些代码拆分出来，独立成职责更加单一的小类。实际上，拆分是应对类或者函数代码过多、应对代码复杂性的一个常用手段。按照这个解决思路，我们对代码进行重构。重构之后的代码如下所示：

```java
public interface ISortAlg {
    void sort(String filePath);
} 

public class QuickSort implements ISortAlg {
    @Override
    public void sort(String filePath) {
        //...
    }
} 

public class ExternalSort implements ISortAlg {
    @Override
    public void sort(String filePath) {
        //...
    }
} 

public class ConcurrentExternalSort implements ISortAlg {
    @Override
    public void sort(String filePath) {
        //...
    }
}

public class MapReduceSort implements ISortAlg {
    @Override
    public void sort(String filePath) {
        //...
    }
}

public class Sorter {
    private static final long GB = 1000 * 1000 * 1000;
    public void sortFile(String filePath) {
        // 省略校验逻辑
        File file = new File(filePath);
        long fileSize = file.length();
        ISortAlg sortAlg;
        if (fileSize < 6 * GB) { // [0, 6GB)
            sortAlg = new QuickSort();
        } else if (fileSize < 10 * GB) { // [6GB, 10GB)
            sortAlg = new ExternalSort();
        } else if (fileSize < 100 * GB) { // [10GB, 100GB)
            sortAlg = new ConcurrentExternalSort();
        } else { // [100GB, ~)
            sortAlg = new MapReduceSort();
        }
        sortAlg.sort(filePath);
    }
}
```

拆分之后，每个类的代码都不会太多，每个类的逻辑都不会太复杂，代码的可读性、可维护性提高了。除此之外，**我们将排序算法设计成独立的类，跟具体的业务逻辑（代码中的if-else 那部分逻辑）解耦，也让排序算法能够复用。这一步实际上就是策略模式的第一步，也就是将策略的定义分离出来。**

实际上，上面的代码还可以继续优化。每种排序类都是无状态的，我们没必要在每次使用的时候，都重新创建一个新的对象。所以，我们可以使用工厂模式对对象的创建进行封装。按照这个思路，我们对代码进行重构。重构之后的代码如下所示：

```java
public class SortAlgFactory {
    private static final Map<String, ISortAlg> algs = new HashMap<>();
    static {
        algs.put("QuickSort", new QuickSort());
        algs.put("ExternalSort", new ExternalSort());
        algs.put("ConcurrentExternalSort", new ConcurrentExternalSort());
        algs.put("MapReduceSort", new MapReduceSort());
    } 

    public static ISortAlg getSortAlg(String type) {
        if (type == null || type.isEmpty()) {
            throw new IllegalArgumentException("type should not be empty.");
        }
        return algs.get(type);
    }
} 

public class Sorter {
    private static final long GB = 1000 * 1000 * 1000;
    public void sortFile(String filePath) {
        // 省略校验逻辑
        File file = new File(filePath);
        long fileSize = file.length();
        ISortAlg sortAlg;
        if (fileSize < 6 * GB) { // [0, 6GB)
            sortAlg = SortAlgFactory.getSortAlg("QuickSort");
        } else if (fileSize < 10 * GB) { // [6GB, 10GB)
            sortAlg = SortAlgFactory.getSortAlg("ExternalSort");
        } else if (fileSize < 100 * GB) { // [10GB, 100GB)
            sortAlg = SortAlgFactory.getSortAlg("ConcurrentExternalSort");
        } else { // [100GB, ~)
            sortAlg = SortAlgFactory.getSortAlg("MapReduceSort");
        }
        sortAlg.sort(filePath);
    }
}
```

经过上面两次重构之后，现在的代码实际上已经符合策略模式的代码结构了。我们通过策略模式将策略的定义、创建、使用解耦，让每一部分都不至于太复杂。不过，Sorter 类中的sortFile() 函数还是有一堆 if-else 逻辑。这里的 if-else 逻辑分支不多、也不复杂，这样写完全没问题。但如果你特别想将 if-else 分支判断移除掉，那也是有办法的。我直接给出代码，你一看就能明白。实际上，这也是基于查表法来解决的，其中的“algs”就是“表”。

```java
public class Sorter {
    private static final long GB = 1000 * 1000 * 1000;
    private static final List<AlgRange> algs = new ArrayList<>();
    static {
        algs.add(new AlgRange(0, 6*GB, SortAlgFactory.getSortAlg("QuickSort")));
        algs.add(new AlgRange(6*GB, 10*GB, SortAlgFactory.getSortAlg("ExternalSort")));
        algs.add(new AlgRange(10*GB, 100*GB, SortAlgFactory.getSortAlg("ConcurrentE")));
        algs.add(new AlgRange(100*GB, Long.MAX_VALUE, SortAlgFactory.getSortAlg("Ma")));
    } 
    public void sortFile(String filePath) {
        // 省略校验逻辑
        File file = new File(filePath);
        long fileSize = file.length();
        ISortAlg sortAlg = null;
        for (AlgRange algRange : algs) {
            if (algRange.inRange(fileSize)) {
                sortAlg = algRange.getAlg();
                break;
            }
        }
        sortAlg.sort(filePath);
    } 
    
    private static class AlgRange {
        private long start;
        private long end;
        private ISortAlg alg;
        public AlgRange(long start, long end, ISortAlg alg) {
            this.start = start;
            this.end = end;
            this.alg = alg;
        }
        public ISortAlg getAlg() {
            return alg;
        } 
        public boolean inRange(long size) {
            return size >= start && size < end;
        }
    }
}
```

现在的代码实现就更加优美了。我们把可变的部分隔离到了策略工厂类和 Sorter 类中的静态代码段中。当要添加一个新的排序算法时，我们只需要修改策略工厂类和 Sort 类中的静态代码段，其他代码都不需要修改，这样就将代码改动最小化、集中化了。

但是即便这样，当我们添加新的排序算法的时候，还是需要修改代码，并不完全符合开闭原则。有什么办法让我们完全满足开闭原则呢？

**对于 Java 语言来说，我们可以通过反射来避免对策略工厂类的修改。具体是这么做的：我们通过一个配置文件或者自定义的 annotation 来标注都有哪些策略类；策略工厂类读取配置文件或者搜索被 annotation 标注的策略类，然后通过反射了动态地加载这些策略类、创建策略对象；当我们新添加一个策略的时候，只需要将这个新添加的策略类添加到配置文件或者用 annotation 标注即可。还记得上一节课的课堂讨论题吗？我们也可以用这种方法来解决。**

对于 Sorter 来说，我们可以通过同样的方法来避免修改。我们通过将文件大小区间和算法之间的对应关系放到配置文件中。当添加新的排序算法时，我们只需要改动配置文件即可， 不需要改动代码。

## 四、重点回顾

如果 if-else 分支判断不复杂、代码不多，这并没有任何问题，毕竟 if-else 分支判断几乎是所有编程语言都会提供的语法，存在即有理由。遵循 KISS 原则，怎么简单怎么来，就是最好的设计。非得用策略模式，搞出 n 多类，反倒是一种过度设计。

一提到策略模式，有人就觉得，它的作用是避免 if-else 分支判断逻辑。实际上，这种认识是很片面的。策略模式主要的作用还是解耦策略的定义、创建和使用，控制代码的复杂度， 让每个部分都不至于过于复杂、代码量过多。除此之外，对于复杂代码来说，策略模式还能让其满足开闭原则，添加新策略的时候，最小化、集中化代码改动，减少引入 bug 的风险。

实际上，设计原则和思想比设计模式更加普适和重要。掌握了代码的设计原则和思想，我们能更清楚的了解，为什么要用某种设计模式，就能更恰到好处地应用设计模式。

## 五、课堂讨论

1.  在过去的项目开发中，你有没有用过策略模式，都是为了解决什么问题才使用的？

2.  你可以说一说，在什么情况下，我们才有必要去掉代码中的 if-else 或者 switch-case 分支逻辑呢？

**精选留言**：

- "设计原则和思想比设计模式更加普适和重要"，被这句话一下子点醒了。可以这样说，设计原则和思想是更高层次的理论和指导原则，设计模式只是这些理论和指导原则下，根据经验和场景，总结出来的编程范式。

- 策略集合的信息也可以定义成枚举，可以放在数据库，可以放kv配置中心，等等。都是一样的道理。

    策略模式是对策略的定义，创建和使用解耦。定义和创建的过程与业务逻辑关系不大，写在一起会影响可读性。

- 1.为了让调度的代码更优雅时使用。（就调度策略的代码而言，可读性高。理解结构后， 阅读的心智负担低，因为调度的原理已经抽象成了共同的type查表，无需逐行检阅分支判断。像一些与持久数据相关的策略，有时为了兼容老数据或则平滑过度，无法全采用type 查表，这时就需要结合if来 实现。所以采用if会让我误以为是这种场景，进而逐行检阅）。

- 策略模式平时开发用的比较多，主要目的还是解耦策略的定义和使用。在Java中，比较喜欢用枚举策略模式。可以定义一个枚举类，提供静态方法根据传入的Type动态返回对应的枚举类。然后在枚举类中定义执行策略的抽象方法，这样迫使每个枚举都得去实现。对于策略不是非常复杂的情况下，这样可以集中管理这一批策略，新增策略的时候也只要在这个枚举类中添加。但是如果策略很复杂会导致这个类非常庞大，还是用传统的方法不同…

- 1，最近在做规则引擎，前端用户通过页面设置出业务执行的流程图，而流程图中包含数据源节点、条件判断节点、业务分发节点、业务事件执行节点。每一种节点执行的业务校验规则都不同，这个时候适合策略模式。使用策略模式的好处是：以后随着业务的发展很有可能出现其他类型的节点，所以这个时候采用策略模式非常合适，易扩展易维护。另外在整个流程流转的规则上采用了模板方法。…

- 1.在之前,有一个用户拥有不同的用户组,对不同的用户组拥有这不同的处理逻辑,这就是一种策略模式,通过用户中的不同用户组名,来获取不同的用户组实例,执行其中的处理方式 2.如果对于判断逻辑中,需要执行的逻辑较为复杂,可以抽取出相同的接口的话,就使用策略模式,将其抽取出接口,并利用策略工厂进行获取策略实例,从而执行对应处理
