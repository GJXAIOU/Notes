# 76开源实战一（上）：通过剖析Java JDK源码学习灵活应用设计模式

> 2020-04-27 王争
>
> 设计模式之美 进入课程 
>
> 从今天开始，我们就正式地进入到实战环节。实战环节包括两部分，一部分是开源项目实战，另一部分是项目实战。
>
> ![](media/image9.png)在开源项目实战部分，我会带你剖析几个经典的开源项目中用到的设计原则、思想和模式，



> 这其中就包括对 Java JDK、Unix、Google Guava、Spring、MyBatis 这样五个开源项目
>
> 的分析。在项目实战部分，我们精心挑选了几个实战项目，手把手地带你利用之前学过的设计原则、思想、模式，来对它们进行分析、设计和代码实现，这其中就包括鉴权限流、幂等重试、灰度发布这样三个项目。
>
> 接下来的两节课，我们重点剖析 Java JDK 中用到的几种常见的设计模式。学习的目的是让你体会，在真实的项目开发中，要学会活学活用，切不可过于死板，生搬硬套设计模式的设计与实现。除此之外，针对每个模式，我们不可能像前面学习理论知识那样，分析得细致入微，很多都是点到为止。在已经具备之前理论知识的前提下，我想你可以跟着我的指引自己去研究，有哪里不懂的话，也可以再回过头去看下之前的理论讲解。
>
> 话不多说，让我们正式开始今天的学习吧！

# 工厂模式在 Calendar 类中的应用

> 在前面讲到工厂模式的时候，大部分工厂类都是以 Factory 作为后缀来命名，并且工厂类主要负责创建对象这样一件事情。但在实际的项目开发中，工厂类的设计更加灵活。那我们就来看下，工厂模式在 Java JDK 中的一个应用：java.util.Calendar。从命名上，我们无法看出它是一个工厂类。
>
> Calendar 类提供了大量跟日期相关的功能代码，同时，又提供了一个 getInstance() 工厂方法，用来根据不同的 TimeZone 和 Locale 创建不同的 Calendar 子类对象。也就是说， 功能代码和工厂方法代码耦合在了一个类中。所以，即便我们去查看它的源码，如果不细心的话，也很难发现它用到了工厂模式。同时，因为它不单单是一个工厂类，所以，它并没有以 Factory 作为后缀来命名。
>
> Calendar 类的相关代码如下所示，大部分代码都已经省略，我只给出了 getInstance() 工厂方法的代码实现。从代码中，我们可以看出，getInstance() 方法可以根据不同TimeZone 和 Locale，创建不同的 Calendar 子类对象，比如 BuddhistCalendar、JapaneseImperialCalendar、GregorianCalendar，这些细节完全封装在工厂方法中，使 用者只需要传递当前的时区和地址，就能够获得一个 Calendar 类对象来使用，而获得的对象具体是哪个 Calendar 子类的对象，使用者在使用的时候并不关心。
>
> 10
>
> 11
>
> 12
>
> 13
>
> 14
>
> 15
>
> 16
>
> 17
>
> 18
>
> 19
>
> 20
>
> 21
>
> 22
>
> 23
>
> 24
>
> 25
>
> 26
>
> 27
>
> 28
>
> 29
>
> 30
>
> 31
>
> 32
>
> 33
>
> 34
>
> 35
>
> 36
>
> 37
>
> 38
>
> 39
>
> 40
>
> 41
>
> 42
>
> 43
>
> 44
>
> 45 }
>
> if (provider != null) { try {
>
> return provider.getInstance(zone, aLocale);
>
> } catch (IllegalArgumentException iae) {
>
> // fall back to the default instantiation
>
> }
>
> }
>
> Calendar cal = null;
>
> if (aLocale.hasExtensions()) {
>
> String caltype = aLocale.getUnicodeLocaleType("ca"); if (caltype != null) {
>
> switch (caltype) { case "buddhist":
>
> cal = new BuddhistCalendar(zone, aLocale); break;
>
> case "japanese":
>
> cal = new JapaneseImperialCalendar(zone, aLocale); break;
>
> case "gregory":
>
> cal = new GregorianCalendar(zone, aLocale); break;
>
> }
>
> }
>
> }
>
> if (cal == null) {
>
> if (aLocale.getLanguage() == "th" && aLocale.getCountry() == "TH") { cal = new BuddhistCalendar(zone, aLocale);
>
> } else if (aLocale.getVariant() == "JP" && aLocale.getLanguage() == "ja" cal = new JapaneseImperialCalendar(zone, aLocale);
>
> } else {
>
> cal = new GregorianCalendar(zone, aLocale);
>
> }
>
> }
>
> return cal;
>
> 46
>
> 47 }
>
> //...

# 建造者模式在 Calendar 类中的应用

> 还是刚刚的 Calendar 类，它不仅仅用到了工厂模式，还用到了建造者模式。我们知道，建造者模式有两种实现方法，一种是单独定义一个 Builder 类，另一种是将 Builder 实现为原始类的内部类。Calendar 就采用了第二种实现思路。我们先来看代码再讲解，相关代码我贴在了下面。

 复制代码

> 1 public abstract class Calendar implements Serializable, Cloneable, Comparable&lt;
>
> 2 //...

1.  public static class Builder {

2.  private static final int NFIELDS = FIELD\_COUNT + 1;

3.  private static final int WEEK\_YEAR = FIELD\_COUNT;

4.  private long instant;

5.  private int\[\] fields;

6.  private int nextStamp;

7.  private int maxFieldIndex;

8.  private String type;

9.  private TimeZone zone;

10. private boolean lenient = true;

11. private Locale locale;

12. private int firstDayOfWeek, minimalDaysInFirstWeek;

> 15
>
> 16 public Builder() {}
>
> 17

1.  public Builder setInstant(long instant) {

2.  if (fields != null) {

3.  throw new IllegalStateException();

> 21 }

1.  this.instant = instant;

2.  nextStamp = COMPUTED;

3.  return this;

> 25 }
>
> 26 //...省略n多set()方法
>
> 27

1.  public Calendar build() {

2.  if (locale == null) {

3.  locale = Locale.getDefault();

> 31 }

1.  if (zone == null) {

2.  zone = TimeZone.getDefault();

> 34 }

1.  Calendar cal;

2.  if (type == null) {

3.  type = locale.getUnicodeLocaleType("ca");

> 38 }

1.  if (type == null) {

2.  if (locale.getCountry() == "TH" && locale.getLanguage() == "th") {

3.  type = "buddhist";

4.  } else {

5.  type = "gregory";

> 44 }
>
> 45 }

1.  switch (type) {

2.  case "gregory":

3.  cal = new GregorianCalendar(zone, locale, true);

4.  break;

5.  case "iso8601":

> 51
>
> 52
>
> 53
>
> 54
>
> 55
>
> 56
>
> 57
>
> 58
>
> 59
>
> 60
>
> 61
>
> 62
>
> 63
>
> 64
>
> 65
>
> 66
>
> 67 }
>
> GregorianCalendar gcal = new GregorianCalendar(zone, locale, true);
>
> // make gcal a proleptic Gregorian gcal.setGregorianChange(new Date(Long.MIN\_VALUE));
>
> // and week definition to be compatible with ISO 8601 setWeekDefinition(MONDAY, 4);
>
> cal = gcal; break;
>
> case "buddhist":
>
> cal = new BuddhistCalendar(zone, locale); cal.clear();
>
> break;
>
> case "japanese":
>
> cal = new JapaneseImperialCalendar(zone, locale, true); break;
>
> default:
>
> throw new IllegalArgumentException("unknown calendar type: " + type)

1.  cal.setLenient(lenient);

2.  if (firstDayOfWeek != 0) {

3.  cal.setFirstDayOfWeek(firstDayOfWeek);

4.  cal.setMinimalDaysInFirstWeek(minimalDaysInFirstWeek);

> 72 }

1.  if (isInstantSet()) {

2.  cal.setTimeInMillis(instant);

3.  cal.complete();

4.  return cal;

> 77 }
>
> 78

1.  if (fields != null) {

2.  boolean weekDate = isSet(WEEK\_YEAR) && fields\[WEEK\_YEAR\] &gt; fields\[YEAR

3.  if (weekDate && !cal.isWeekDateSupported()) {

4.  throw new IllegalArgumentException("week date is unsupported by " +

> 83 }

1.  for (int stamp = MINIMUM\_USER\_STAMP; stamp &lt; nextStamp; stamp++) {

2.  for (int index = 0; index &lt;= maxFieldIndex; index++) {

3.  if (fields\[index\] == stamp) {

4.  cal.set(index, fields\[NFIELDS + index\]);

5.  break;

> 89 }
>
> 90 }
>
> 91 }
>
> 92

1.  if (weekDate) {

2.  int weekOfYear = isSet(WEEK\_OF\_YEAR) ? fields\[NFIELDS + WEEK\_OF\_YEAR

3.  int dayOfWeek = isSet(DAY\_OF\_WEEK) ? fields\[NFIELDS + DAY\_OF\_WEEK\] :

4.  cal.setWeekDate(fields\[NFIELDS + WEEK\_YEAR\], weekOfYear, dayOfWeek);

> 97 }
>
> 98 cal.complete();
>
> 99 }
>
> 100 return cal;
>
> 101 }
>
> 102 }
>
> 看了上面的代码，我有一个问题请你思考一下：既然已经有了 getInstance() 工厂方法来创建 Calendar 类对象，为什么还要用 Builder 来创建 Calendar 类对象呢？这两者之间的区别在哪里呢？
>
> 实际上，在前面讲到这两种模式的时候，我们对它们之间的区别做了详细的对比，现在，我们再来一块回顾一下。工厂模式是用来创建不同但是相关类型的对象（继承同一父类或者接口的一组子类），由给定的参数来决定创建哪种类型的对象。建造者模式用来创建一种类型的复杂对象，通过设置不同的可选参数，“定制化”地创建不同的对象。
>
> 网上有一个经典的例子很好地解释了两者的区别。
>
> 顾客走进一家餐馆点餐，我们利用工厂模式，根据用户不同的选择，来制作不同的食 物，比如披萨、汉堡、沙拉。对于披萨来说，用户又有各种配料可以定制，比如奶酪、西红柿、起司，我们通过建造者模式根据用户选择的不同配料来制作不同的披萨。
>
> 粗看 Calendar 的 Builder 类的 build() 方法，你可能会觉得它有点像工厂模式。你的感觉没错，前面一半代码确实跟 getInstance() 工厂方法类似，根据不同的 type 创建了不同的Calendar 子类。实际上，后面一半代码才属于标准的建造者模式，根据 setXXX() 方法设置的参数，来定制化刚刚创建的 Calendar 子类对象。
>
> 你可能会说，这还能算是建造者模式吗？我用 第 46 讲的一段话来回答你：
>
> 我们也不要太学院派，非得把工厂模式、建造者模式分得那么清楚，我们需要知道的 是，每个模式为什么这么设计，能解决什么问题。只有了解了这些最本质的东西，我们才能不生搬硬套，才能灵活应用，甚至可以混用各种模式，创造出新的模式来解决特定场景的问题。
>
> 实际上，从 Calendar 这个例子，我们也能学到，不要过于死板地套用各种模式的原理和实现，不要不敢做丝毫的改动。模式是死的，用的人是活的。在实际上的项目开发中，不仅各种模式可以混合在一起使用，而且具体的代码实现，也可以根据具体的功能需求做灵活的调整。

# 装饰器模式在 Collections 类中的应用

> 我们前面讲到，Java IO 类库是装饰器模式的非常经典的应用。实际上，Java 的Collections 类也用到了装饰器模式。
>
> Collections 类是一个集合容器的工具类，提供了很多静态方法，用来创建各种集合容器， 比如通过 unmodifiableColletion() 静态方法，来创建 UnmodifiableCollection 类对象。而这些容器类中的 UnmodifiableCollection 类、CheckedCollection 和SynchronizedCollection 类，就是针对 Collection 类的装饰器类。
>
> 因为刚刚提到的这三个装饰器类，在代码结构上几乎一样，所以，我们这里只拿其中的UnmodifiableCollection 类来举例讲解一下。UnmodifiableCollection 类是 Collections 类的一个内部类，相关代码我摘抄到了下面，你可以先看下。

 复制代码

1.  public class Collections {

2.  private Collections() {}

> 3

1.  public static &lt;T&gt; Collection&lt;T&gt; unmodifiableCollection(Collection&lt;? extends

2.  return new UnmodifiableCollection&lt;&gt;(c);

> 6 }
>
> 7

1.  static class UnmodifiableCollection&lt;E&gt; implements Collection&lt;E&gt;, Serializa

2.  private static final long serialVersionUID = 1820017752578914078L;

3.  final Collection&lt;? extends E&gt; c;

> 11

1.  UnmodifiableCollection(Collection&lt;? extends E&gt; c) {

2.  if (c==null)

3.  throw new NullPointerException();

4.  this.c = c;

> 16 }
>
> 17

1.  public int size() {return c.size();}

2.  public boolean isEmpty() {return c.isEmpty();}

3.  public boolean contains(Object o) {return c.contains(o);}

4.  public Object\[\] toArray() {return c.toArray();}

5.  public &lt;T&gt; T\[\] toArray(T\[\] a) {return c.toArray(a);}

6.  public String toString() {return c.toString();}

> 24

1.  public Iterator&lt;E&gt; iterator() {

2.  return new Iterator&lt;E&gt;() {

3.  private final Iterator&lt;? extends E&gt; i = c.iterator();

> 28

1.  public boolean hasNext() {return i.hasNext();}

2.  public E next() {return i.next();}

[TABLE]

[TABLE]

> 看了上面的代码，请你思考一下，为什么说 UnmodifiableCollection 类是 Collection 类的装饰器类呢？这两者之间可以看作简单的接口实现关系或者类继承关系吗？
>
> 我们前面讲过，装饰器模式中的装饰器类是对原始类功能的增强。尽管UnmodifiableCollection 类可以算是对 Collection 类的一种功能增强，但这点还不具备足够的说服力来断定 UnmodifiableCollection 就是 Collection 类的装饰器类。
>
> 实际上，最关键的一点是，UnmodifiableCollection 的构造函数接收一个 Collection 类对象，然后对其所有的函数进行了包裹（Wrap）：重新实现（比如 add() 函数）或者简单封装（比如 stream() 函数）。而简单的接口实现或者继承，并不会如此来实现UnmodifiableCollection 类。所以，从代码实现的角度来说，UnmodifiableCollection 类是典型的装饰器类。

# 适配器模式在 Collections 类中的应用

> 在 第 51 讲中我们讲到，适配器模式可以用来兼容老的版本接口。当时我们举了一个 JDK 的例子，这里我们再重新仔细看一下。
>
> 老版本的 JDK 提供了 Enumeration 类来遍历容器。新版本的 JDK 用 Iterator 类替代Enumeration 类来遍历容器。为了兼容老的客户端代码（使用老版本 JDK 的代码），我们保留了 Enumeration 类，并且在 Collections 类中，仍然保留了 enumaration() 静态方法
>
> （因为我们一般都是通过这个静态函数来创建一个容器的 Enumeration 类对象）。
>
> 不过，保留 Enumeration 类和 enumeration() 函数，都只是为了兼容，实际上，跟适配器没有一点关系。那到底哪一部分才是适配器呢？
>
> 在新版本的 JDK 中，Enumeration 类是适配器类。它适配的是客户端代码（使用Enumeration 类）和新版本 JDK 中新的迭代器 Iterator 类。不过，从代码实现的角度来
>
> 说，这个适配器模式的代码实现，跟经典的适配器模式的代码实现，差别稍微有点大。enumeration() 静态函数的逻辑和 Enumeration 适配器类的代码耦合在一起， enumeration() 静态函数直接通过 new 的方式创建了匿名类对象。具体的代码如下所示：

[TABLE]

# 重点回顾

> 好了，今天的内容到此就讲完了。我们一块来总结回顾一下，你需要重点掌握的内容。
>
> 今天，我重点讲了工厂模式、建造者模式、装饰器模式、适配器模式，这四种模式在 Java JDK 中的应用，主要目的是给你展示真实项目中是如何灵活应用设计模式的。
>
> 从今天的讲解中，我们可以学习到，尽管在之前的理论讲解中，我们都有讲到每个模式的经典代码实现，但是，在真实的项目开发中，这些模式的应用更加灵活，代码实现更加自由， 可以根据具体的业务场景、功能需求，对代码实现做很大的调整，甚至还可能会对模式本身的设计思路做调整。
>
> 比如，Java JDK 中的 Calendar 类，就耦合了业务功能代码、工厂方法、建造者类三种类型的代码，而且，在建造者类的 build 方法中，前半部分是工厂方法的代码实现，后半部分才是真正的建造者模式的代码实现。这也告诉我们，在项目中应用设计模式，切不可生搬硬套，过于学院派，要学会结合实际情况做灵活调整，做到心中无剑胜有剑。

# 课堂讨论

> 在 Java 中，经常用到的 StringBuilder 类是否是建造者模式的应用呢？你可以试着像我一样从源码的角度去剖析一下。
>
> 欢迎留言和我分享你的想法。如果有收获，也欢迎你把这篇文章分享给你的朋友。

![](media/image10.png)

> © 版权归极客邦科技所有，未经许可不得传播售卖。 页面已增加防盗追踪，如有侵权极客邦将依法追究其法律责任。
>
> 上一篇 75 \| 在实际的项目开发中，如何避免过度设计？又如何避免设计不足？
>
> 下一篇 77 \| 开源实战一（下）：通过剖析Java JDK源码学习灵活应用设计模式
>
> **精选留言 (10)**

## ![](media/image11.png)![](media/image12.png)Darren

> 2020-04-27
>
> 我觉得是，因为StringBuilder的主要方法append，其实就是类似于建造者模式中的set方法，只不过构建者模式的set方法可能是对象的不同属性，但append其实是在一直修改一个属性，且最后没有build(),但StringBuilder出现的目的其实是为了解决String不可变的问题，最终输出其实是String，所以可以类比toString()就是build()，所以认为算是建造者模式。
>
> 展开

![](media/image14.png)![](media/image15.png)4

## ![](media/image16.png)QQ怪

> 2020-04-27
>
> StringBuilder的append()方法使用了建造者模式，StringBuilder把构建者的角色交给了其的父类AbstractStringBuilder，最终调用的是父类的append（）方法
>
> 展开

![](media/image17.png)![](media/image18.png)2

## ![](media/image19.png)小晏子

> 2020-04-27
>
> 课后思考：
>
> 我的答案是算也不算…，如果按照学院派的思想，stringbuilder和GOF中的对于builder模式的定义完全不同，stringbuilder并不会创建新的string对象，只是将多个字符连接在一起，而builder模式的基本功能是生成新对象，两个本质就不一样了，从这个角度来讲，str ingbuilder不能算是builder模式。…
>
> 展开

![](media/image17.png)![](media/image18.png)1

## ![](media/image20.png)Jeff.Smile

> 2020-04-27
>
> 总结：
>
> 1 工厂模式：简单工厂模式+工厂方法模式

-   简单工厂模式直接在条件判断中根据不同参数将目标对象 new 了出来。

-   工厂方法模式是将目标对象的创建过程根据参数分类抽取到各自独立的工厂类中，以应对目标对象创建过程的复杂度。条件分支可以使用 map 来缓存起来！…

> 展开

![](media/image14.png)![](media/image15.png)1

## ![](media/image21.png)Demon.Lee

> 2020-04-27
>
> 如果说要创建一个复杂的String对象，那么通过StringBuilder的append()方法会非常方
>
> 便，最后通过toString()方法返回，从这个角度看算建造者模式。@Override
>
> public String toString() {
>
> // Create a copy, don't share the array…
>
> 展开

![](media/image22.png)![](media/image23.png)

## ![](media/image24.png)Jxin

> 2020-04-27
>
> 回答课后题：
>
> 我认为应该算是建造者模式。
>
> 拿餐厅类比，对于披萨加番茄，起司等不同配料来定制披萨，这属于建造者模式要解决…
>
> 展开

![](media/image25.png)![](media/image26.png)

## ![](media/image27.png)Heaven

> 2020-04-27
>
> 个人认为,是属于建造者模式的,在其中,最主要的append方法,是将其抛给了父类AbstractSt ringBuilder,然后返回自己,其父类AbstractStringBuilder中维护了一个数组,并且可以动然扩容,在我们最后获取结果的toString()方法中,就是直接new String对象,这种模式其实更像是装饰器模式的实现
>
> 展开

![](media/image25.png)![](media/image26.png)

## ![](media/image28.png)守拙

> 2020-04-27
>
> 课堂讨论:
>
> StringBuilder应用了Builder模式. 其主要方式是append(), 即通过不断append创建复杂对象.
>
> 不同于传统Builder模式的是:
>
> 1\. StringBuilder的目的是创建String, 但StringBuilder并不是String的内部类.…
>
> 展开

![](media/image29.png)![](media/image23.png)

## ![](media/image30.png)全炸攻城狮

> 2020-04-27
>
> 感觉不像建造者。append在StringBuilder的创建过程中不起任何作用，append真正用到的地方是StringBuilder创建好以后，对字符串的拼接。StringBuilder的创建是调用的构造
>
> 方法

![](media/image31.png)![](media/image32.png)

## ![](media/image33.png)Keep-Moving

> 2020-04-27
>
> 感觉UnmodifiableCollection 更像是 Collection 类的代理类

![](media/image34.png)![](media/image32.png)1
