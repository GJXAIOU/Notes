# 06 \| Stream如何提高遍历集合效率？

上一讲中，我在讲List集合类，那我想你一定也知道集合的顶端接口Collection。在Java8中，Collection新增了两个流方法，分别是Stream()和parallelStream()。

通过英文名不难猜测，这两个方法肯定和Stream有关，那进一步猜测，是不是和我们熟悉的InputStream和OutputStream也有关系呢？集合类中新增的两个Stream方法到底有什么作用？今天，我们就来深入了解下Stream。

## 什么是Stream？

现在很多大数据量系统中都存在分表分库的情况。

例如，电商系统中的订单表，常常使用用户ID的Hash值来实现分表分库，这样是为了减少单个表的数据量，优化用户查询订单的速度。

但在后台管理员审核订单时，他们需要将各个数据源的数据查询到应用层之后进行合并操作。

例如，当我们需要查询出过滤条件下的所有订单，并按照订单的某个条件进行排序，单个数据源查询出来的数据是可以按照某个条件进行排序的，但多个数据源查询出来已经排序好的数据，并不代表合并后是正确的排序，所以我们需要在应用层对合并数据集合重新进行排序。

在Java8之前，我们通常是通过for循环或者Iterator迭代来重新排序合并数据，又或者通过重新定义Collections.sorts的Comparator方法来实现，这两种方式对于大数据量系统来说，效率并不是很理想。

Java8中添加了一个新的接口类Stream，他和我们之前接触的字节流概念不太一样，Java8集合中的Stream相当于高级版的Iterator，他可以通过Lambda 表达式对集合进行各种非常便利、高效的聚合操作（Aggregate Operation），或者大批量数据操作 (Bulk Data Operation)。

Stream的聚合操作与数据库SQL的聚合操作sorted、filter、map等类似。我们在应用层就可以高效地实现类似数据库SQL的聚合操作了，而在数据操作方面，Stream不仅可以通过串行的方式实现数据操作，还可以通过并行的方式处理大批量数据，提高数据的处理效率。

**接下来我们就用一个简单的例子来体验下Stream的简洁与强大。**

这个Demo的需求是过滤分组一所中学里身高在160cm以上的男女同学，我们先用传统的迭代方式来实现，代码如下：

```java
Map<String, List<Student>> stuMap = new HashMap<String, List<Student>>();
        for (Student stu: studentsList) {
            if (stu.getHeight() > 160) { //如果身高大于160
                if (stuMap.get(stu.getSex()) == null) { //该性别还没分类
                    List<Student> list = new ArrayList<Student>(); //新建该性别学生的列表
                    list.add(stu);//将学生放进去列表
                    stuMap.put(stu.getSex(), list);//将列表放到map中
                } else { //该性别分类已存在
                    stuMap.get(stu.getSex()).add(stu);//该性别分类已存在，则直接放进去即可
                }
            }
        }
```

我们再使用Java8中的Stream API进行实现：

1\.串行实现

```java
Map<String, List<Student>> stuMap = stuList.stream().filter((Student s) -> s.getHeight() > 160) .collect(Collectors.groupingBy(Student ::getSex));
```

2\.并行实现

```java
Map<String, List<Student>> stuMap = stuList.parallelStream().filter((Student s) -> s.getHeight() > 160) .collect(Collectors.groupingBy(Student ::getSex));
```

通过上面两个简单的例子，我们可以发现，Stream结合Lambda表达式实现遍历筛选功能非常得简洁和便捷。

## Stream如何优化遍历？

上面我们初步了解了Java8中的Stream API，那Stream是如何做到优化迭代的呢？并行又是如何实现的？下面我们就透过Stream源码剖析Stream的实现原理。

### 1\.Stream操作分类

在了解Stream的实现原理之前，我们先来了解下Stream的操作分类，因为他的操作分类其实是实现高效迭代大数据集合的重要原因之一。为什么这样说，分析完你就清楚了。

官方将Stream中的操作分为两大类：中间操作（Intermediate operations）和终结操作（Terminal operations）。中间操作只对操作进行了记录，即只会返回一个流，不会进行计算操作，而终结操作是实现了计算操作。

中间操作又可以分为无状态（Stateless）与有状态（Stateful）操作，前者是指元素的处理不受之前元素的影响，后者是指该操作只有拿到所有元素之后才能继续下去。

终结操作又可以分为短路（Short-circuiting）与非短路（Unshort-circuiting）操作，前者是指遇到某些符合条件的元素就可以得到最终结果，后者是指必须处理完所有元素才能得到最终结果。操作分类详情如下图所示：

![](<https://static001.geekbang.org/resource/image/ea/94/ea8dfeebeae8f05ae809ee61b3bf3094.jpg?wh=2036*1438>)

<span class="orange">我们通常还会将中间操作称为懒操作，也正是由这种懒操作结合终结操作、数据源构成的处理管道（Pipeline），实现了Stream的高效。</span>

### 2\.Stream源码实现

在了解Stream如何工作之前，我们先来了解下Stream包是由哪些主要结构类组合而成的，各个类的职责是什么。参照下图：

![](<https://static001.geekbang.org/resource/image/fc/00/fc256f9f8f9e3224aac10b2ee8940e00.jpg?wh=698*428>)

BaseStream和Stream为最顶端的接口类。BaseStream主要定义了流的基本接口方法，例如，spliterator、isParallel等；Stream则定义了一些流的常用操作方法，例如，map、filter等。

ReferencePipeline是一个结构类，他通过定义内部类组装了各种操作流。他定义了Head、StatelessOp、StatefulOp三个内部类，实现了BaseStream与Stream的接口方法。

Sink接口是定义每个Stream操作之间关系的协议，他包含begin()、end()、cancellationRequested()、accpt()四个方法。ReferencePipeline最终会将整个Stream流操作组装成一个调用链，而这条调用链上的各个Stream操作的上下关系就是通过Sink接口协议来定义实现的。

### 3\.Stream操作叠加

我们知道，一个Stream的各个操作是由处理管道组装，并统一完成数据处理的。在JDK中每次的中断操作会以使用阶段（Stage）命名。

管道结构通常是由ReferencePipeline类实现的，前面讲解Stream包结构时，我提到过ReferencePipeline包含了Head、StatelessOp、StatefulOp三种内部类。

Head类主要用来定义数据源操作，在我们初次调用names.stream()方法时，会初次加载Head对象，此时为加载数据源操作；接着加载的是中间操作，分别为无状态中间操作StatelessOp对象和有状态操作StatefulOp对象，此时的Stage并没有执行，而是通过AbstractPipeline生成了一个中间操作Stage链表；当我们调用终结操作时，会生成一个最终的Stage，通过这个Stage触发之前的中间操作，从最后一个Stage开始，递归产生一个Sink链。如下图所示：

![](<https://static001.geekbang.org/resource/image/f5/19/f548ce93fef2d41b03274295aa0a0419.jpg?wh=1854*364>)

**下面我们再通过一个例子来感受下Stream的操作分类是如何实现高效迭代大数据集合的。**

```java
List<String> names = Arrays.asList("张三", "李四", "王老五", "李三", "刘老四", "王小二", "张四", "张五六七");

String maxLenStartWithZ = names.stream()
    	            .filter(name -> name.startsWith("张"))
    	            .mapToInt(String::length)
    	            .max()
    	            .toString();
```

这个例子的需求是查找出一个长度最长，并且以张为姓氏的名字。从代码角度来看，你可能会认为是这样的操作流程：首先遍历一次集合，得到以“张”开头的所有名字；然后遍历一次filter得到的集合，将名字转换成数字长度；最后再从长度集合中找到最长的那个名字并且返回。

这里我要很明确地告诉你，实际情况并非如此。我们来逐步分析下这个方法里所有的操作是如何执行的。

首先 ，因为names是ArrayList集合，所以names.stream()方法将会调用集合类基础接口Collection的Stream方法：

```java
default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
    }
```

然后，Stream方法就会调用StreamSupport类的Stream方法，方法中初始化了一个ReferencePipeline的Head内部类对象：

```java
public static <T> Stream<T> stream(Spliterator<T> spliterator, boolean parallel) {
        Objects.requireNonNull(spliterator);
        return new ReferencePipeline.Head<>(spliterator,
                                            StreamOpFlag.fromCharacteristics(spliterator),
                                            parallel);
    }
```

再调用filter和map方法，这两个方法都是无状态的中间操作，所以执行filter和map操作时，并没有进行任何的操作，而是分别创建了一个Stage来标识用户的每一次操作。

而通常情况下Stream的操作又需要一个回调函数，所以一个完整的Stage是由数据来源、操作、回调函数组成的三元组来表示。如下图所示，分别是ReferencePipeline的filter方法和map方法：

```java
@Override
    public final Stream<P_OUT> filter(Predicate<? super P_OUT> predicate) {
        Objects.requireNonNull(predicate);
        return new StatelessOp<P_OUT, P_OUT>(this, StreamShape.REFERENCE,
                                     StreamOpFlag.NOT_SIZED) {
            @Override
            Sink<P_OUT> opWrapSink(int flags, Sink<P_OUT> sink) {
                return new Sink.ChainedReference<P_OUT, P_OUT>(sink) {
                    @Override
                    public void begin(long size) {
                        downstream.begin(-1);
                    }

                    @Override
                    public void accept(P_OUT u) {
                        if (predicate.test(u))
                            downstream.accept(u);
                    }
                };
            }
        };
    }
```

```java
@Override
    @SuppressWarnings("unchecked")
    public final <R> Stream<R> map(Function<? super P_OUT, ? extends R> mapper) {
        Objects.requireNonNull(mapper);
        return new StatelessOp<P_OUT, R>(this, StreamShape.REFERENCE,
                                     StreamOpFlag.NOT_SORTED | StreamOpFlag.NOT_DISTINCT) {
            @Override
            Sink<P_OUT> opWrapSink(int flags, Sink<R> sink) {
                return new Sink.ChainedReference<P_OUT, R>(sink) {
                    @Override
                    public void accept(P_OUT u) {
                        downstream.accept(mapper.apply(u));
                    }
                };
            }
        };
    }
```

new StatelessOp将会调用父类AbstractPipeline的构造函数，这个构造函数将前后的Stage联系起来，生成一个Stage链表：

```java
AbstractPipeline(AbstractPipeline<?, E_IN, ?> previousStage, int opFlags) {
        if (previousStage.linkedOrConsumed)
            throw new IllegalStateException(MSG_STREAM_LINKED);
        previousStage.linkedOrConsumed = true;
        previousStage.nextStage = this;//将当前的stage的next指针指向之前的stage

        this.previousStage = previousStage;//赋值当前stage当全局变量previousStage 
        this.sourceOrOpFlags = opFlags & StreamOpFlag.OP_MASK;
        this.combinedFlags = StreamOpFlag.combineOpFlags(opFlags, previousStage.combinedFlags);
        this.sourceStage = previousStage.sourceStage;
        if (opIsStateful())
            sourceStage.sourceAnyStateful = true;
        this.depth = previousStage.depth + 1;
    }
```

因为在创建每一个Stage时，都会包含一个opWrapSink()方法，该方法会把一个操作的具体实现封装在Sink类中，Sink采用（处理->转发）的模式来叠加操作。

当执行max方法时，会调用ReferencePipeline的max方法，此时由于max方法是终结操作，所以会创建一个TerminalOp操作，同时创建一个ReducingSink，并且将操作封装在Sink类中。

```java
@Override
    public final Optional<P_OUT> max(Comparator<? super P_OUT> comparator) {
        return reduce(BinaryOperator.maxBy(comparator));
    }
```

最后，调用AbstractPipeline的wrapSink方法，该方法会调用opWrapSink生成一个Sink链表，Sink链表中的每一个Sink都封装了一个操作的具体实现。

```java
@Override
    @SuppressWarnings("unchecked")
    final <P_IN> Sink<P_IN> wrapSink(Sink<E_OUT> sink) {
        Objects.requireNonNull(sink);

        for ( @SuppressWarnings("rawtypes") AbstractPipeline p=AbstractPipeline.this; p.depth > 0; p=p.previousStage) {
            sink = p.opWrapSink(p.previousStage.combinedFlags, sink);
        }
        return (Sink<P_IN>) sink;
    }
```

当Sink链表生成完成后，Stream开始执行，通过spliterator迭代集合，执行Sink链表中的具体操作。

```java
@Override
    final <P_IN> void copyInto(Sink<P_IN> wrappedSink, Spliterator<P_IN> spliterator) {
        Objects.requireNonNull(wrappedSink);

        if (!StreamOpFlag.SHORT_CIRCUIT.isKnown(getStreamAndOpFlags())) {
            wrappedSink.begin(spliterator.getExactSizeIfKnown());
            spliterator.forEachRemaining(wrappedSink);
            wrappedSink.end();
        }
        else {
            copyIntoWithCancel(wrappedSink, spliterator);
        }
    }
```

Java8中的Spliterator的forEachRemaining会迭代集合，每迭代一次，都会执行一次filter操作，如果filter操作通过，就会触发map操作，然后将结果放入到临时数组object中，再进行下一次的迭代。完成中间操作后，就会触发终结操作max。

这就是串行处理方式了，那么Stream的另一种处理数据的方式又是怎么操作的呢？

### 4\.Stream并行处理

Stream处理数据的方式有两种，串行处理和并行处理。要实现并行处理，我们只需要在例子的代码中新增一个Parallel()方法，代码如下所示：

```java
List<String> names = Arrays.asList("张三", "李四", "王老五", "李三", "刘老四", "王小二", "张四", "张五六七");

String maxLenStartWithZ = names.stream()
                    .parallel()
    	            .filter(name -> name.startsWith("张"))
    	            .mapToInt(String::length)
    	            .max()
    	            .toString();
```

Stream的并行处理在执行终结操作之前，跟串行处理的实现是一样的。而在调用终结方法之后，实现的方式就有点不太一样，会调用TerminalOp的evaluateParallel方法进行并行处理。

```java
final <R> R evaluate(TerminalOp<E_OUT, R> terminalOp) {
        assert getOutputShape() == terminalOp.inputShape();
        if (linkedOrConsumed)
            throw new IllegalStateException(MSG_STREAM_LINKED);
        linkedOrConsumed = true;

        return isParallel()
               ? terminalOp.evaluateParallel(this, sourceSpliterator(terminalOp.getOpFlags()))
               : terminalOp.evaluateSequential(this, sourceSpliterator(terminalOp.getOpFlags()));
    }
```

这里的并行处理指的是，Stream结合了ForkJoin框架，对Stream 处理进行了分片，Splititerator中的estimateSize方法会估算出分片的数据量。

ForkJoin框架和估算算法，在这里我就不具体讲解了，如果感兴趣，你可以深入源码分析下该算法的实现。

通过预估的数据量获取最小处理单元的阈值，如果当前分片大小大于最小处理单元的阈值，就继续切分集合。每个分片将会生成一个Sink链表，当所有的分片操作完成后，ForkJoin框架将会合并分片任何结果集。

## 合理使用Stream

看到这里，你应该对Stream API是如何优化集合遍历有个清晰的认知了。Stream API用起来简洁，还能并行处理，那是不是使用Stream API，系统性能就更好呢？通过一组测试，我们一探究竟。

我们将对常规的迭代、Stream串行迭代以及Stream并行迭代进行性能测试对比，迭代循环中，我们将对数据进行过滤、分组等操作。分别进行以下几组测试：

- 多核CPU服务器配置环境下，对比长度100的int数组的性能；
- 多核CPU服务器配置环境下，对比长度1.00E+8的int数组的性能；
- 多核CPU服务器配置环境下，对比长度1.00E+8对象数组过滤分组的性能；
- 单核CPU服务器配置环境下，对比长度1.00E+8对象数组过滤分组的性能。

<!-- -->

由于篇幅有限，我这里直接给出统计结果，你也可以自己去验证一下，具体的测试代码可以在[Github](<https://github.com/nickliuchao/stream>)上查看。通过以上测试，我统计出的测试结果如下（迭代使用时间）：

- 常规的迭代<Stream并行迭代<Stream串行迭代
- Stream并行迭代<常规的迭代<Stream串行迭代
- Stream并行迭代<常规的迭代<Stream串行迭代
- 常规的迭代<Stream串行迭代<Stream并行迭代

通过以上测试结果，我们可以看到：在循环迭代次数较少的情况下，常规的迭代方式性能反而更好；在单核CPU服务器配置环境中，也是常规迭代方式更有优势；而在大数据循环迭代中，如果服务器是多核CPU的情况下，Stream的并行迭代优势明显。所以我们在平时处理大数据的集合时，应该尽量考虑将应用部署在多核CPU环境下，并且使用Stream的并行迭代方式进行处理。

用事实说话，我们看到其实使用Stream未必可以使系统性能更佳，还是要结合应用场景进行选择，也就是合理地使用Stream。

## 总结

纵观Stream的设计实现，非常值得我们学习。从大的设计方向上来说，Stream将整个操作分解为了链式结构，不仅简化了遍历操作，还为实现了并行计算打下了基础。

从小的分类方向上来说，Stream将遍历元素的操作和对元素的计算分为中间操作和终结操作，而中间操作又根据元素之间状态有无干扰分为有状态和无状态操作，实现了链结构中的不同阶段。

**在串行处理操作中，**Stream在执行每一步中间操作时，并不会做实际的数据操作处理，而是将这些中间操作串联起来，最终由终结操作触发，生成一个数据处理链表，通过Java8中的Spliterator迭代器进行数据处理；此时，每执行一次迭代，就对所有的无状态的中间操作进行数据处理，而对有状态的中间操作，就需要迭代处理完所有的数据，再进行处理操作；最后就是进行终结操作的数据处理。

**在并行处理操作中，**Stream对中间操作基本跟串行处理方式是一样的，但在终结操作中，Stream将结合ForkJoin框架对集合进行切片处理，ForkJoin框架将每个切片的处理结果Join合并起来。最后就是要注意Stream的使用场景。

## 思考题

这里有一个简单的并行处理案例，请你找出其中存在的问题。

```java
//使用一个容器装载100个数字，通过Stream并行处理的方式将容器中为单数的数字转移到容器parallelList
List<Integer> integerList= new ArrayList<Integer>();

for (int i = 0; i <100; i++) {
      integerList.add(i);
}

List<Integer> parallelList = new ArrayList<Integer>() ;
integerList.stream()
           .parallel()
           .filter(i->i%2==1)
           .forEach(i->parallelList.add(i));
```

期待在留言区看到你的答案。也欢迎你点击“请朋友读”，把今天的内容分享给身边的朋友，邀请他一起学习。



## 精选留言(65)

- ![img](https://static001.geekbang.org/account/avatar/00/18/74/21/6c64afa9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  奔跑的猪

  实践中大量采用stream大概有2年了吧，先是在Team内推广，后来在CodeReview中强制要求。 个人以为，出发点并不是出于性能考虑，而是结合lambda，在编程思维上的转变，将大家对代码的关注点放在“行为传递”上面，而不是参数传递，阅读时也能省去模板语法产生的“噪音”。

  2019-07-26

  **1

  **76

- ![img](06%20_%20Stream%E5%A6%82%E4%BD%95%E6%8F%90%E9%AB%98%E9%81%8D%E5%8E%86%E9%9B%86%E5%90%88%E6%95%88%E7%8E%87%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425595605-1251.jpeg)

  陆离

  思考题中这样的方式会造成null值和缺值 因为arraylist不是线程安全的，例如线程一在size++后准备给index为size+1的位置赋值，这个时候第二个线程又给size++，这个线程一赋值的index就变成了size+2,在线程一赋值后，线程二又在size+2的位置赋值。 这样的结果就是size+1的位置没有值null,size+2的位置为线程二赋的值，线程一赋的值被覆盖。 正确的方式应该是使用collect()

  2019-06-01

  **2

  **38

- ![img](https://static001.geekbang.org/account/avatar/00/12/03/c7/bd45f0c9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  小白猪

  思考题，由于流是并行处理，parallelList会存在并发问题，应该使用collect方法聚合

  2019-06-01

  **3

  **20

- ![img](https://static001.geekbang.org/account/avatar/00/12/78/dc/0c9c9b0f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  (´田ω田`)

  感觉这一节课已经值回了整个课程的票价，给老师点赞！ 思考题：Stream并行执行，无法确认每个元素的处理顺序，最后parallelList中的数字是无序的

  作者回复: 思考题中的问题是在并行操作arraylist时，需要考虑线程安全问题

  2019-06-01

  **

  **12

- ![img](06%20_%20Stream%E5%A6%82%E4%BD%95%E6%8F%90%E9%AB%98%E9%81%8D%E5%8E%86%E9%9B%86%E5%90%88%E6%95%88%E7%8E%87%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425595605-1254.jpeg)

  一路看风景

  老师您好，在容器盛行的微服务环境下，以及大数据处理流行的潮流中，我觉得stream的应用空间多少有些尴尬呢，不知是不是我的理解有误。即：单核容器运行的环境下stream没了性能优势，大数据的处理又有大数据平台去完成使命，所以是不是意味着我们可以从stream得到的最大收益变成了流式编程和函数式编程带来的代码易读和易用性了呢？

  作者回复: 是的，但未必所有公司都有构建大数据的能力，而且一些公司有自己的中间件团队，例如文章开始说到的分表分库的查询操作，使用stream的并行操作就有优势了

  2019-06-01

  **2

  **11

- ![img](06%20_%20Stream%E5%A6%82%E4%BD%95%E6%8F%90%E9%AB%98%E9%81%8D%E5%8E%86%E9%9B%86%E5%90%88%E6%95%88%E7%8E%87%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425595605-1255.jpeg)

  小辉辉

  ArrayList是线程不安全的集合，而当前又用了并行流去处理，所以会出现有异常、少数据或者正常输出结果这三种情况。

  2019-06-02

  **

  **10

- ![img](06%20_%20Stream%E5%A6%82%E4%BD%95%E6%8F%90%E9%AB%98%E9%81%8D%E5%8E%86%E9%9B%86%E5%90%88%E6%95%88%E7%8E%87%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425595605-1256.jpeg)

  Liam

  parallel Stream 的并发机制是通过ForkJoinPool实现的，它的通用线程池是一个无界队列，想问下，数据量很大的时候，比如1w个元素，它分片的依据是什么，每个分片多大；子任务比较多的时候，会不会严重消耗内存以及频繁触发GC等

  2019-06-01

  **

  **9

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/6PbKL8YRE2wzqdoxcS5E88Wvot8Vv0Kuo92BUKPlWISPfGjSXCmK7vD12aBdibwY6q11gbkPxK4Weje2xCcCdEw/132)

  阿厚

  老师，请教2个问题： 1.有什么分表分库中间件推荐么？ 2.分表分库以后，查询分页怎么办呢？

  作者回复: 之前用过sharing-jdbc以及mycat，一个明显的区别是sharing-jdbc是嵌入方式，而mycat是基于proxy，所以理论上来说 proxy方式会有性能损耗。现在我们在使用sharing-jdbc，这里不打广告，两个中间件都有自己的优势。 分页查询是基于我这篇文章说的方式，将每个分表的数据结果集查询出来，通过归并排序计算出。 具体的实现方式有区别，本次专栏的后面课程也会具体讲到。

  2019-06-06

  **

  **8

- ![img](https://static001.geekbang.org/account/avatar/00/17/38/c3/f18411f9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  圣西罗

  老师，现在网上有些说法做测试用lambda比普通for循环速度慢五倍，因此有人拒绝用。实际情况是什么样呢？如果我自己想测，应该怎么尽可能排除外因干扰，测一下他们的实际效率对比？

  作者回复: 当应用程序以前没有使用lambda表达式时，会动态生成lambda目标对象，这是导致慢的实际原因。我们可以在运行加载后，也就是初次测试之后，紧接着后面加几个for循环，再测试几次，对比下性能。 虽然单独使用lambda表达式在初次运行时要比传统方式慢很多，但结合stream的并行操作，在多核环境下还有有优势的。

  2019-06-01

  **2

  **6

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTIy5ULaodUwsLoPuk1wd22hqXsaBbibNEqXM0kgrCTYDGKYQkZICYEyH9wMj4hyUicuQwHdDuOKRj0g/132)

  辉煌码农

  allMatch为什么是短路呢，短路的如何定义的呢

  作者回复: 终结操作又可以分为短路（Short-circuiting）与非短路（Unshort-circuiting）操作，前者是指遇到某些符合条件的元素就可以得到最终结果，后者是指必须处理完所有元素才能得到最终结果。 allMatch也是属于遇到某些条件的情况下可以终结的操作，即找到一个不合法条件的，短路返回false ，无需等待其他的处理结果，所以也属于短路。

  2020-01-02

  **

  **5

- ![img](https://static001.geekbang.org/account/avatar/00/12/7b/57/a9b04544.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  QQ怪

  是不是该把思考题中的arraylist换成线程安全的copyOnwriteList就可以解决线程不安全问题?

  作者回复: 对的，但copyOnwriteList更适合某一时间段统一新增，且新增时避免大量操作容器发生。比较适合在深夜更新黑名单类似的业务。

  2019-06-03

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/10/e9/52/aa3be800.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Loubobooo

  parallelList集合里呈现的是无序的数字，是这样吗？

  作者回复: 对的，可能会出现少数字、无序以及异常情况

  2019-06-01

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/17/1e/89/25b12054.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Andy

  感觉stream这种中间操作和终结操作 跟spark中转换操作和处理操作 思想很像，懒加载

  作者回复: 是的，好的实现思想会被应用到各个地方

  2019-10-21

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/11/4c/59/c75cb36d.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  N

  老师有个问题请教一下，公司业务代码中有大量stream对集合遍历，过滤，聚合的用法，但都是串行的，因为大部分数据量不是很大，请问数据量多大的时候才有必要使用并行提高效率呢？

  作者回复: 上万数量级使用并行可以提高效率。

  2019-07-14

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/12/95/52/ad190682.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Mr wind

  为什么聚合操作是线程安全的呢。

  作者回复: Java8 Stream的collect方法是线程安全的。

  2019-12-08

  **

  **3

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  开心小毛

  使用stream是否有节省内存消耗的考虑，例如当需要遍历一个含上万条目的数据库查询结果。

  2019-11-19

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/13/26/38/ef063dc2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  Darren

  最终的结果无序，且可能结果都是不正确的，因为ArrayList是线程不安全的

  2019-06-04

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/2a/54/c9990105.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  bro.

  老师，这么早更新，读完感觉跟rxjava设计思想很接近，不订阅前面过滤条件都不会真正的运行！

  2019-06-01

  **1

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/11/a0/cb/aab3b3e7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  张三丰

  老师，有个地方想不明白，串行化的stream执行filter,map的时候为何比直接使用for循环快？stream是每迭代一次就执行一次filter,map。而直接for也可以循环一次执行filter,map。它们时间复杂度是一样的。

  2020-04-17

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/6d/56/65b05765.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Allen

  对一个集合进行并行处理 我对比下来 使用显式的线程池进行多线程处理要快于使用 parallel stream  而且使用线程池应该能保证系统线程资源不被耗尽吧 

  作者回复: 是的，原理都差不多，parallel stream写起来更简便

  2020-04-13

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/13/31/16/f2269e73.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  better

  目前刚毕业，读起这篇文章觉得有点吃力，特别是到了Stream的源码开始那里，后面基本都看不懂了，老师，是因为现在的实战经验还不够吗

  作者回复: 这篇源码是比较难理解，尝试多读几遍

  2019-09-24

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/17/11/6b/8034959a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  迎风劲草

  老师，为什么stream操作，就比自己循环的效率高呢，没看懂。

  作者回复: 这里强调的是使用stream的并发处理大数据时，效率高于传统的遍历处理。

  2019-06-05

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/e6/41/83275db5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  乐

  那请问老师，思考题中如何解决这种并发时的线程安全问题？是使用 CopyOnWriteArrayList 还是使用 .collect(Collectors.toList())？

  作者回复: 两者都可以，不过这里如果要使用线程安全集合，可以使用vector。

  2019-06-05

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/14/37/8e/cf0b4575.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  郑晨Cc

  课程好值啊 全是干货

  2019-06-03

  **

  **1

- ![img](06%20_%20Stream%E5%A6%82%E4%BD%95%E6%8F%90%E9%AB%98%E9%81%8D%E5%8E%86%E9%9B%86%E5%90%88%E6%95%88%E7%8E%87%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425595605-1256.jpeg)

  Liam

  并发操作一个ArrayList，会有线程安全问题？

  作者回复: 对的

  2019-06-01

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/11/7a/b9/c3d3a92f.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  G小调

  两个问题 1.可能parallelList数组越界 2.数据可能缺少

  2022-04-27

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/e4/fb/ff564de5.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  六六大顺

  i->i%2==1 这个地方推荐使用i->i%2!=0

  2022-02-02

  **

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTJmUb3iazHcLxaVRgpBNUDKYDsibZJ1Z9kaBziaJkbI37FknKUBa4ZTib9pj2ibhUUXe59Jn6yo4FuVC3g/132)

  Geek_95ce9d

  伐值可还行

  2021-05-11

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7/7gAOQWRvYmUAZMAAAAAB/9sAhAABAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAgICAgICAgICAgIDAwMDAwMDAwMDAQEBAQEBAQIBAQICAgECAgMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwP/wAARCADuAO4DAREAAhEBAxEB/8QAfAABAAICAwEBAAAAAAAAAAAAAAYHBAgBAwUCCgEBAAAAAAAAAAAAAAAAAAAAABAAAgIBAgIECwQJBQAAAAAAAAECAwQRBSEGMWESF0FRgVITk+MUVJTUIkJiB5EyhBVFhbXFNnFygqJTEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwD9vAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGHmbhg7fD0mbl4+LF69l32wrctPBCMmpTfUk2BG7ue+Wqm4rNsua6XTi5DWvVKyutPyaoDmnnrlq19l506W9NPTYuSk2/xQqnGPlaQElxM7Dzq/S4WVj5VfhlRbC1Rfil2G3GXU9GBlAAAAAAAAAAAAAAAAAAAAAAAAAAAA4bUU5SajGKblJtJJJattvgkkBVHMnP8AJSswtilHSLcLdxcVLV9DWHCWsdF/6ST1+6uiQFW35F+VbK/Jutvum9Z23WSssk/xTm3JgdIADvx8nIxLY34t9uPdD9W2myVc1412otPR6cV0MC1uWufvTTrwd8cITlpCrcYpQhKT4KOXBaQrbf346R8aXFgWmnrxXFPimvCAAAAAAAAAAAAAAAAAAAAAAAAAAFUfmBzHKLexYVjjrGMtxsg+LU12oYia6E4tSn400vOQFTAAAAAAAuDkDmSWRFbHm2OVtUHLb7ZvWU6oLWeK2+LdMV2ofgTX3UBaAAAAAAAAAAAAAAAAAAAAAAAABi52XDAwsvNs4wxce6+S10cvRQlNQX4ptaLrYGr+RfblX3ZN8nO7Itsutk/vWWSc5Pq4sDpAAAAAABlYWXbgZeNmUPS3Guruhx0TcJJ9mWnTGa4NeFMDaDGvrysejJqeteRTVfW/HC2EbI/9ZAdwAAAAAAAAAAAAAAAAAAAAAACJc8WurlncOzwdrxateqeVT2v0wTXlA18AAAAAAAAAbFcnXSu5a2mcnq402U/8cfJuoivJGtASYAAAAAAAAAAAAAAAAAAAAAABFOdqXdyzuSjxlWse7yVZVMp/or1YGvQAAAAAAAADY3lGiWPy3tNclo5Yzv8AF9nJusyYvyxtQEjAAAAAAAAAAAAAAAAAAAAAAAdGVj15eNkYty1qyaLaLF4exbCVctOvSXADWDNxLsDLycLIj2bsa6dM/E3B6KUfHCa0afhTAxQAAAAAAZ224Nu55+LgUp+kyboV6pa9iDetljXm1VpyfUgNnqaoUU1UVLs101wqrj4oVxUILyRQHYAAAAAAAAAAAAAAAAAAAAAAAAVrz5yzPNh++cGtzyaK1HNpgtZX0QX2bopcZW0R4NdLhp5ujCmQAAAAAAXbyLyzPbaXumdW4ZuVX2aKprSWNjS0bck+Mbr9FqumMeHS2gLDAAAAAAAAAAAAAAAAAAAAAAAAAACuOZOQ6c+dmbtDrxcubc7cWX2cbIm+LlW0n7vbLw8OxJ+bxbCpM7bM/bLXVn4l+NPVpekg1CenhrsWtdseuLaAwQAHo7ftO47raqsDEuyZapSlCOlVevhtul2aql/uaAt3lrkWjbJ15u5yry86Gk6qYrXFxpripfaSd90X0NpRi+hNpSAsIAAAAAAAAAAAAAAAAAAAAAAAAAAAAD4sqrug67a4W1y/WhZCM4P/AFjJNMDw7eVuXrm5T2jCTfT6Kr0C49VLrQHNPK/L1ElKvaMJtcU7alfo/Gle7FqB7cK4VQVdcIVwitIwhFQhFeJRikkgPsAAAAAAAAAAAAAAAAAAAAAAAAAY2XmYuBRPKzL68aiv9ay2XZjq+iKXTKcvBFJt+BARGf5g8uRk4q3LsSeinDFkoy60pyhPR9aQHz3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAMjG575cybY1PKtxnJpRnk0Trq1fglZHtxrXXLRLxgTCMozjGUZKUZJSjKLTjKLWqlFrVNNPgwOQAAAAAAAAAAAAAAAAAAAAUZ+YW43ZG9ywHOSx9vqpUa9fsu7IphkTta8MnCyMepLrYECAAAAAAAAAAAF0/lxuN2Tt+Zg2zlOO320uhyerhTlK1qpPzYWUSa8Xa06NALHAAAAAAAAAAAAAAAAAAAABr3zx/lO6fsX9OxAImAAAAAAAAAAALY/K/+Ofyz+4AWwAAAAAAAAAAAAAAAAAAAADXvnj/ACndP2L+nYgETAAAAAAAAAAAFsflf/HP5Z/cALYAAAAAAAAAAAAAAAAAAAABVvMfJG7bxvOZuONkbdCjI937Eb7cmNq9DiUUS7Ua8S2C1nU2tJPgB4fdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAJvyby1ncvfvH323Et989z9F7rZdPs+7+9dvt+loo019OtNNfD0ATcAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//Z)

  Geek_926921

  stream 可以去看《Java 8 实战》这本书，里面有详细介绍stream流的演变过程，以及forkJoin的原理。

  2021-05-09

  **

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/DYAIOgq83erHVh5AkbfJMK2xQlM0vow6UlsOAUQI47tia6SnQsQAujd0yGwRnOtibrEevkzEcdatzBdnCPnd8GyA/132)

  Geek_d2186f

  对过前几课的学习，跟老师学习最深的新的，就是学习源码，了解代码运行的本质，感受很深，在这里咨询一下老师，感觉有时候给一个大框架，看代码知其代码干什么，不知道整体框架中功能，所以咨询一下，这种东西是需要对技术文档有个全面通读，然后了解原因的方法是否更好呢？谢谢

  2020-09-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/15/ae/504940ec.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  李昊

  没有终结操作

  2020-08-18

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/10/43/79/18073134.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  test

  如果换成这样：        List<Integer> parallelList = integerList.stream()                .parallel()                .filter(i->i%2==1)                .collect(Collectors.toList()); 是不是变成串行了的呢？

  2020-08-14

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/3a/6d/910b2445.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  fastkdm

  老师，在介绍AbstractPipeline构造函数的时候，previousStage.nextStage = this;这一行的注释写反了，应该是将之前stage的next指针指向当前的stage，然后下一步才是更新缓存上一个stage

  2020-05-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/6d/56/65b05765.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Allen

  parallelList 类型改成 CopyOnWriteArrayList 应该就可以了

  2020-04-13

  **

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTIZtZz0LgYRdEibwdYuvoiavKqBBuTsRlldPGJnJXlyelaE4HG2qFvmChX3UEibwEBgcfsvicJxyTjEEQ/132)

  2YSP

  这里的源码有点难理解，看了好几遍才看懂。原来小数据的情况下常规迭代更有性能优势，涨知识了。

  2020-04-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/06/16/e85c1fa8.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  滴答丶滴

  老师，请问，parallelStream()  与 parallel() 有什么区别嘛？

  2020-04-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/cc/0d/89435926.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  10年以后

  stream流

  2020-03-29

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/88/6e/3bd860d3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  .

  stream使代码更加简洁优雅

  2020-03-18

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/14/83/19/0a3fe8c1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Evan

  1、在并情况，ArrayList 是线程不安全的，可能会少数据。 2、数据返回 collect 方法聚合

  2020-03-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/18/ca/0e/5009c5ff.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  遇见

  "这个例子的需求是查找出一个长度最长，并且以张为姓氏的名字" 文稿中的代码只能获取到名字最长的长度吧? 是获取不到名字最长的名字的, 最后的toString只能得到 "OptionalInt[4]" 得不到 "张五六七" 改成: "names.stream()                .filter(name -> name.startsWith("张")).max(Comparator.comparingInt(String::length))                .ifPresent(System.out::println);" 才可以打印出来名字

  作者回复: 是的

  2019-12-17

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/29/a5/9c6e7526.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  丽儿

  有点类似于spark中rdd操作

  2019-12-05

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/15/03/c0fe1dbf.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  考休

  并行操作中，采用的ArrayList容器是线程不安全的，会造成共享数据错误的问题。

  作者回复: 是的

  2019-11-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/18/5f/10/ed332d5a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  warriorSL

  最后这个问题，我在实际的编程中也出现过，先声明了一个arrayList用于存放结果，后面使用parallelStream对数据进行加工，然后add进list中，最后又对arrayList进行遍历，总会出现npe的报错，直到打日志才发现，里面有个元素是null

  2019-10-18

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/0f/63/77/423345ab.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Sdylan

  2019.10.12 打卡  目前开发和生产用的是jdk7，读此篇 扩展一下视野。后续细读

  2019-10-12

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/08/5b/2a342424.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  青莲

  并行处理，ArrayList不是安全的并发容器，全出现添加数小于等于50

  2019-09-24

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/67/f4/9a1feb59.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  钱

  课后思考及问题 JDK8 目前还未使用，课后思考题中ArraryList是非线程安全的并发执行会存在多线程安全的问题。 目前所知Stream在多核机器上执行性能更佳，单核不能发挥并行的威力反而会因为线程的上下文切换导致性能下降。 感觉老师设计的思考题没有仅仅围绕性能优化的思路，另外每节感觉没有一个毕竟明显的关联关系，可能和课程定位相关吧😄像柯南一样每集的凶手都不一样!

  2019-09-07

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/12/b2/b3/798a4bb2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  帽子丨影

  两个问题请教下：1.作者结尾说道，pall在中间操作的处理上跟串行一样，是指执行中间操作也是串行的还是说生成sink时是一样的。第二个问题是，以文中取张姓最长的名字长度的例子中普通的方法仅需一次迭代，而使用Stream，中间操作需要迭代一次，终结操作又需要迭代一次是吗？

  2019-08-06

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/EcYNib1bnDf5dz6JcrE8AoyZYMdqic2VNmbBtCcVZTO9EoDZZxqlQDEqQKo6klCCmklOtN9m0dTd2AOXqSneJYLw/132)

  博弈

  ArrayList是非线程安全的，在多线程环境下会出现无需，数据不全，异常等情况

  作者回复: 对的

  2019-08-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/ae/8c/5c92c95e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  张健

  老师，拿你的测试代码，测试的时候，发现在1.00E+8 大小下，速度快慢和执行代码的顺序有关，调整顺序执行后，结果完全不一样，最先执行的永远最慢 SerialStreamTest.SerialStreamForObjectTest(studentsList); ParallelStreamTest.ParallelStreamForObjectTest(studentsList); IteratorTest.IteratorForObjectTest(studentsList);

  作者回复: 建议多次执行，取平均值

  2019-07-23

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/d9/8b/76c27279.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  师志强

  通过对比发现（在多核场景），能用stream并行就用，不能用就用常规，stream串行好像没有任何优势可言。是不是多有场景中都不建议使用stream串行呢？

  作者回复: 是的，stream在多核机器下并行处理大数据量优势明显。

  2019-07-08

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/d8/be/49d49db2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  一路奔跑

  ArraryList是非线性安全的，并行流处理会出现越界或者重复或者少元素的情况！这个坑我踩过！

  作者回复: 过来人，印象应该深刻

  2019-07-08

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/16/00/c7/59caefa7.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  ok绷

  parallelList是非线程安全的，可以使用线程安全的集合类，但是不知道到使用stream的collect方法可以吗？

  作者回复: 可以

  2019-07-02

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/63/6b/34b89fae.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  男朋友

  让我想到了REDIS,虽然是单线程的,但是redis是等数据来了才处理,而不是一连接就处理或者等待的.

  2019-07-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/71/05/db554eba.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  👽

  首先是测试结果，无序的问题是其一，不过也不算问题。另一个问题是，有可能会最后结果只有47~49个值的现象（实际值应该为50个）。并且多次循环的话会报下标越界。 自认为是ArrayList的并发问题。

  作者回复: 对了！

  2019-06-10

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/8f/82/374f43a1.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  假装自己不胖

  例子中查询长度最长并且以张为姓氏的名字,如果有两个会怎么样

  作者回复: 结果是一样的，算出最大值。可以复制代码实践下，注意复制后的空格符问题。

  2019-06-04

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/fe/44/3e3040ac.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  程序员人生

  是因为并行处理并且List是非线程安全的缘故吗？那段代码执行几次后会出现null，把parallel去掉就好了

  作者回复: 是的，但如果需要并行计算时，我们又怎么去处理这类问题呢？

  2019-06-03

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/fe/44/3e3040ac.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  程序员人生

  ArrayList不是线程安全的，而parallel()又是并行流，是不是会有问题？

  2019-06-03

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/Q0j4TwGTfTJJYJ74BKhY0wt5qbCj91ArzdVZ6rvibyMqQZ8iaZBibwNQC0AxvHPy0AvJBI8mleicT4UlF7jChiaJFXg/132)

  fl

  可以具体解释下spliterator吗

  2019-06-01

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/PiajxSqBRaEIvUlicgrWtibbDzwhLw5cQrDSy2JuE1mVvmXq11KQIwpLicgDuWfpp9asE0VCN6HhibPDWn7wBc2lfmA/132)

  a、

  运行老师的那段代码,发现会提示ArrayIndexOutOfBoundsException,想到arrayslist是非线程安全的，于是就把parallelList改成了Vector，运行多次，并未发现异常。然后看到有学员说用collect方法，我就改成conllect方法，也没有出现异常。对比了两个方法，发现用verctor比用collect方法的性能要高，但是collect方法出来的list是排好序的，而Vector是乱序的，于是我把数据量调到了1千万，Vector再加上排序，发现也比collect方法要快。不是很清楚，为什么并行的处理大数据量也比加锁的要慢？

  2019-06-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/11/5f/27/a6873bc9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  我知道了嗯

  思考题结果是无序的并且有null值?  这是为什么

  作者回复: ArrayList不是线程安全的，在并行操作时，会出现多线程操作问题，例如出现null值，有可能是在扩容时，复制出现问题。同时也会出现值被覆盖的情况。

  2019-06-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/38/db/6825519a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  梁小航航

  老师，我有一个疑问，就是在例子中查询长度最长并且以张为姓氏的名字。代码在实际运行中maxLenStartwithZ 值为：OptionalInt.empty

  作者回复: 你好皮卡丘，运行结果是OptionalInt[4]，再排查下是不是复制代码存在差异。

  2019-06-01

  **2

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/e9/52/aa3be800.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Loubobooo

  嵌套循环多次，会出现数组越界的问题，推测应该是该流是并行处理，操作非安全类ArrayList存在并发问题

  2019-06-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/f1/c9/adc1df03.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  哲

  老师，我按你这例子写了一下，执行直接抛数组下标的异常了，这是为何？而且多执行几次，并不是每次都异常，期待您解一下我的疑惑

  作者回复: 由于arraylist为非线程安全，所以在并行操作时，会出现异常和无序的情况。

  2019-06-01

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/93/b2/abb7bfe3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  黄老邪

  思考题：ArrayList不是线程安全的集合类，并发操作可能会造成数据不准确 之前在使用的stream的时候只觉得写代码的时候更简洁方便了，没考虑性能的问题，日后还是要根据场景进行区分

  2019-06-01

  **

  **

- ![img](06%20_%20Stream%E5%A6%82%E4%BD%95%E6%8F%90%E9%AB%98%E9%81%8D%E5%8E%86%E9%9B%86%E5%90%88%E6%95%88%E7%8E%87%EF%BC%9F.resource/resize,m_fill,h_34,w_34-1662425595605-1251.jpeg)

  陆离

  老师，我一般使用stream的原因是它这种DSL风格使代码很简洁，并且封装了map，reduce一些操作，最重要的是可并行。 但是stream高效这块我很疑惑，虽然它是在终止操作之前执行中间操作，但它在迭代那些filter不是也是使用的传统的方式吗，而且在数据量不是很大的情况下还会比传统方式要慢一些。

  作者回复: 对的，在串行时效率没有传统方式快，但数据量比较大时，并行的效率最好。

  2019-06-01

  **

  **
