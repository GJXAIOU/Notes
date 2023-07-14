# 深入浅出序列化（1）——JDK序列化和Hessian序列化

### 一、JDK 序列化

作为一个成熟的编程语言，Java 本身就已经提供了序列化的方法了，因此我们也选择把他作为第一个介绍的序列化方式。

JDK 自带的序列化方式，使用起来非常方便，只需要序列化的类实现了 Serializable 接口即可，Serializable 接口没有定义任何方法和属性，所以只是起到了**标识**的作用，**表示这个类是可以被序列化的。如果想把一个 Java 对象变为 byte[] 数组，需要使用ObjectOutputStream**。它负责把一个 Java 对象写入一个字节流：

```yaml
public class Main {
    public static void main(String[] args) throws IOException {
        ByteArrayOutputStream buffer = new ByteArrayOutputStream();
        try (ObjectOutputStream output = new ObjectOutputStream(buffer)) {
            // 写入int:
            output.writeInt(12345);
            // 写入String:
            output.writeUTF("Hello");
            // 写入Object:
            output.writeObject(Double.valueOf(123.456));
        }
        System.out.println(Arrays.toString(buffer.toByteArray()));
    }
}
```

如果没有实现 Serializable 接口而进行序列化操作就会抛出 NotSerializableException 异常

上面代码输出的是如下的 Byte 数组：

![image-20230304232352089](Hessian%E5%BA%8F%E5%88%97%E5%8C%96.resource/image-20230304232352089.png)

#### serialVersionUID

在反序列化时，JVM 需要知道所属的 class 文件，在序列化的时候 JVM 会记录 class 文件的版本号，也即 serialVersionUID 这一变量。该变量默认是由 JVM 自动生成，也可以手动定义。反序列化时 JVM 会按版本号找指定版本的 class 文件进行反序列化，如果 class 文件有版本号在序列化和反序列化时不一致就会导致反序列化失败，会抛异常提示版本号不一致，

#### 特点

JDK 序列化会把对象类的描述和所有属性的元数据都序列化为字节流，另外**继承的元数据也会序列化**，所以导致序列化的元素较多且字节流很大，但是由于序列化了所有信息所以相对而言更可靠。但是如果只需要序列化属性的值时就比较浪费。

而且因为 Java 的序列化机制可以导致一个实例**能直接从 byte[] 数组创建**，而不经过构造方法，因此，它存在一定的安全隐患。一个精心构造的 byte[] 数组被反序列化后可以执行特定的 Java 代码，从而导致严重的安全漏洞。

其次，由于这种方式是JDK自带，无法被多个语言通用，因此通常情况下不会使用该种方式进行序列化。

## 二、Hessian

Hessian 是一种动态类型、二进制序列化和 Web 服务协议，专为面向对象的传输而设计。

官方介绍👉[Hessian 2.0 Serialization Protocol](https://link.juejin.cn/?target=http%3A%2F%2Fhessian.caucho.com%2Fdoc%2Fhessian-serialization.html)

和 JDK 自带的序列化方式类似，Hessian 采用的也是二进制协议，**只不过 Hessian 序列化之后，字节数更小，性能更优**。目前Hessian已经出到2.0版本，相较于1.0的Hessian性能更优。相较于JDK自带的序列化，Hessian的设计目标更明确👇

![image-20230304233130666](Hessian%E5%BA%8F%E5%88%97%E5%8C%96.resource/image-20230304233130666.png)

#### 序列化实现方式

之所以说 Hessian 序列化之后的数据字节数更小，和他的实现方式密不可分，以 int 存储整数为例，我们通过官网的说明可以发现存储逻辑如下👇

![image-20230304233149155](Hessian%E5%BA%8F%E5%88%97%E5%8C%96.resource/image-20230304233149155.png)

翻译一下就是：

> 一个32位有符号的整数。一个整数由八位数x49('I')表示，后面是整数的4个八位数，以高位优先（big-endian）顺序排列。

简单来说就是，如果要存储一个数据为 1 的整数，Hessian会存储成`I 1`这样的形式。

存对象也很简单，如下：

![image-20230304233342336](Hessian%E5%BA%8F%E5%88%97%E5%8C%96.resource/image-20230304233342336.png)

对于Hessian支持的数据结构，官网均有序列化的语法，详情可参考👉[Serialization](https://link.juejin.cn/?target=http%3A%2F%2Fhessian.caucho.com%2Fdoc%2Fhessian-serialization.html%23anchor4)

而且，和JDK自带序列化不同的是，如果一个对象之前出现过，hessian会直接插入一个R index这样的块来表示一个引用位置，从而省去再次序列化和反序列化的时间。

也正是因为如此，Hessian的序列化速度相较于JDK序列化才更快。只不过Java序列化会把要序列化的对象类的元数据和业务数据全部序列化从字节流，并且会保留完整的继承关系，因此相较于Hessian序列化更加可靠。

不过相较于JDK的序列化，Hessian另一个优势在于，这是一个跨语言的序列化方式，这意味着序列化后的数据可以被其他语言使用，兼容性更好。

#### 不服跑个分？

空口无凭，我们以一个小demo来测试一下：

![img](Hessian%E5%BA%8F%E5%88%97%E5%8C%96.resource/1d8ac85be211445eab3202d0eaa6eefdtplv-k3u1fbpfcp-zoom-in-crop-mark4536000.webp)

使用之前记得先引入依赖哦

```xml
<!-- https://mvnrepository.com/artifact/com.caucho/hessian -->
<dependency>
    <groupId>com.caucho</groupId>
    <artifactId>hessian</artifactId>
    <version>4.0.65</version>
</dependency>
复制代码
```

运行的结果也和我们预想的一样：

```
hessian序列化长度：47
jdk序列化长度：99
```

## 总结

通过上文的介绍，想必你也了解了JDK自带序列化方式和Hessian序列化之间的一些区别了，JDK序列化选择将所有的信息保存下来，因此可靠性更好。与此同时，由于采取了不同的序列化方案，Hessian在体积和速度上相较于JDK序列化更优秀，且由于Hessian设计之初就考虑到跨语言的需求因此在兼容性方面也更胜一筹。

之后我们将会介绍其他的一些序列化协议，如果你觉得本文对你有所帮助，不妨点赞关注支持一下～

# 深入浅出序列化（2）——Kryo序列化

前一篇文章我们介绍了 Java 中的两个常见的序列化方式，JDK 序列化和 Hessian2 序列化，本文我们接着来讲述一个后起之秀——Kryo 序列化，它号称 Java 中最快的序列化框架。那么话不多说，就让我们来看看这个后起之秀到底有什么能耐吧。

## Kryo 序列化

Kryo 是一个快速序列化/反序列化工具，依赖于字节码生成机制（底层使用了 ASM 库)，因此在序列化速度上有一定的优势，但正因如此，其使用也只能限制在基于 JVM 的语言上。

> 网上有很多资料说 Kryo 只能在 Java 上使用，这点是不对的，事实上除 Java 外，Scala 和 Kotlin 这些基于 JVM 的语言同样可以使用 Kryo 实现序列化。

和 Hessian 类似，Kryo 序列化出的结果，是其自定义的、独有的一种格式。由于其序列化出的结果是二进制的，也即 byte[]，因此像 Redis 这样可以存储二进制数据的存储引擎是可以直接将 Kryo 序列化出来的数据存进去。当然你也可以选择转换成 String 的形式存储在其他存储引擎中（性能有损耗）。

由于其优秀的性能，目前 Kryo 已经成为多个知名 Java 框架的底层序列化协议，包括但不限于 👇

- [Apache Fluo](https://link.juejin.cn/?target=https%3A%2F%2Ffluo.apache.org%2F) (Kryo is default serialization for Fluo Recipes)
- [Apache Hive](https://link.juejin.cn/?target=http%3A%2F%2Fhive.apache.org%2F) (query plan serialization)
- [Apache Spark](https://link.juejin.cn/?target=http%3A%2F%2Fspark.apache.org%2F) (shuffled/cached data serialization)
- [Storm](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fnathanmarz%2Fstorm%2Fwiki%2FSerialization) (distributed realtime computation system, in turn used by [many others](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fnathanmarz%2Fstorm%2Fwiki%2FPowered-By))
- [Apache Dubbo](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fapache%2Fincubator-dubbo) (high performance, open source RPC framework)
- ……

官网地址在：[github.com/EsotericSof…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FEsotericSoftware%2Fkryo)

## 基础用法

介绍了这么多，接下来我们就来看看 Kryo 的基础用法吧。其实对于序列化框架来说，API 基本都差不多，毕竟入参和出参通常都是确定的（需要序列化的对象/序列化的结果）。在使用 Kryo 之前，我们需要引入相应的依赖

```xml
<dependency>
  <groupId>com.esotericsoftware</groupId>
  <artifactId>kryo</artifactId>
  <version>5.2.0</version>
</dependency>
```

基本使用如下所示 👇

```ini
import com.esotericsoftware.kryo.Kryo;
import com.esotericsoftware.kryo.io.Input;
import com.esotericsoftware.kryo.io.Output;
import java.io.*;

public class HelloKryo {
   static public void main(String[] args) throws Exception {
       Kryo kryo = new Kryo();
       kryo.register(SomeClass.class);

       SomeClass object = new SomeClass();
       object.value = "Hello Kryo!";

       Output output = new Output(new FileOutputStream("file.bin"));
       kryo.writeObject(output, object);
       output.close();

       Input input = new Input(new FileInputStream("file.bin"));
       SomeClass object2 = kryo.readObject(input, SomeClass.class);
       input.close();
       System.out.println(object2.value);
  }

   static public class SomeClass {
       String value;
  }
}
```

Kryo 类会自动执行序列化。Output 类和 Input 类负责处理缓冲字节，并写入到流中。

## Kryo 的序列化

作为一个灵活的序列化框架，Kryo 并不关心读写的数据，作为开发者，你可以随意使用 Kryo 提供的那些开箱即用的序列化器。

### Kryo 的注册

和很多其他的序列化框架一样，Kryo 为了提供性能和减小序列化结果体积，提供注册的序列化对象类的方式。在注册时，会为该序列化类生成 int ID，后续在序列化时使用 int ID 唯一标识该类型。注册的方式如下：

```arduino
kryo.register(SomeClass.class);
```

或者

```arduino
kryo.register(SomeClass.class, 1);
```

可以明确指定注册类的 int ID，但是该 ID 必须大于等于 0。如果不提供，内部将会使用 int++的方式维护一个有序的 int ID 生成。

### Kryo 的序列化器

Kryo 支持多种序列化器，通过源码我们可窥知一二 👇

![img](Hessian%E5%BA%8F%E5%88%97%E5%8C%96.resource/7a4343a40be943518dcce04714cb4bd1tplv-k3u1fbpfcp-zoom-in-crop-mark4536000.webp)

具体可参考 👉[「Kryo 支持的序列化类型」](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FEsotericSoftware%2Fkryo%2Fblob%2Fmaster%2Fsrc%2Fcom%2Fesotericsoftware%2Fkryo%2FKryo.java%23L179)

虽然 Kryo 提供的序列化器可以读写大多数对象，但开发者也可以轻松的制定自己的序列化器。篇幅限制，这里就不展开说明了，仅以默认的序列化器为例。

## 对象引用

在新版本的 Kryo 中，默认情况下是不启用对象引用的。这意味着如果一个对象多次出现在一个对象图中，它将被多次写入，并将被反序列化为多个不同的对象。

举个例子，当开启了引用属性，每个对象第一次出现在对象图中，会在记录时写入一个 varint，用于标记。当此后有同一对象出现时，只会记录一个 varint，以此达到节省空间的目标。此举虽然会节省序列化空间，但是是一种用时间换空间的做法，会影响序列化的性能，这是因为在写入/读取对象时都需要进行追踪。

开发者可以使用 kryo 自带的 `setReferences` 方法来决定是否启用 Kryo 的引用功能。

## 线程不安全

Kryo 不是线程安全的。每个线程都应该有自己的 Kryo 对象、输入和输出实例。

因此在多线程环境中，可以考虑使用 ThreadLocal 或者对象池来保证线程安全性。

### ThreadLocal + Kryo 解决线程不安全

ThreadLocal 是一种典型的牺牲空间来换取并发安全的方式，它会为每个线程都单独创建本线程专用的 kryo 对象。对于每条线程的每个 kryo 对象来说，都是顺序执行的，因此天然避免了并发安全问题。创建方法如下：

```ini
static private final ThreadLocal<Kryo> kryos = new ThreadLocal<Kryo>() {
  protected Kryo initialValue() {
     Kryo kryo = new Kryo();
     // 在此处配置kryo对象的使用示例，如循环引用等
     return kryo;
  };
};

Kryo kryo = kryos.get();
复制代码
```

之后，仅需要通过 `kryos.get()` 方法从线程上下文中取出对象即可使用。

### 对象池 + Kryo 解决线程不安全

**「池」**是一种非常重要的编程思想，连接池、线程池、对象池等都是**「复用」**思想的体现，通过将创建的“对象”保存在某一个“容器”中，以便后续反复使用，避免创建、销毁的产生的性能损耗，以此达到提升整体性能的作用。

Kryo 对象池原理也是如此。Kryo 框架自带了对象池的实现，整个使用过程不外乎**创建池、从池中获取对象、归还对象**三步，以下为代码实例。

```java
// Pool constructor arguments: thread safe, soft references, maximum capacity
Pool<Kryo> kryoPool = new Pool<Kryo>(true, false, 8) {
  protected Kryo create () {
     Kryo kryo = new Kryo();
     // Kryo 配置
     return kryo;
  }
};

// 获取池中的Kryo对象
Kryo kryo = kryoPool.obtain();
// 将kryo对象归还到池中
kryoPool.free(kryo);
复制代码
```

创建 Kryo 池时需要传入三个参数，其中第一个参数用于指定是否在 Pool 内部使用同步，如果指定为 true，则允许被多个线程并发访问。第三个参数适用于指定对象池的大小的，这两个参数较容易理解，因此重点来说一下第二个参数。

如果将第二个参数设置为 true，Kryo 池将会使用 java.lang.ref.SoftReference 来存储对象。这允许池中的对象在 JVM 的内存压力大时被垃圾回收。Pool clean 会删除所有对象已经被垃圾回收的软引用。当没有设置最大容量时，这可以减少池的大小。当池子有最大容量时，没有必要调用 clean，因为如果达到了最大容量，Pool free 会尝试删除一个空引用。

创建玩 Kryo 池后，使用 kryo 就变得异常简单了，只需调用 `kryoPool.obtain()` 方法即可，使用完毕后再调用 `kryoPool.free(kryo)` 归还对象，就完成了一次完整的租赁使用。

理论上，只要对象池大小评估得当，就能在占用极小内存空间的情况下完美解决并发安全问题。如果想要封装一个 Kryo 的序列化方法，可以参考如下的代码 👇

```scss
public static byte[] serialize(Object obj) {
   Kryo kryo = kryoPool.obtain();
   // 使用 Output 对象池会导致序列化重复的错误（getBuffer返回了Output对象的buffer引用）
   try (Output opt = new Output(1024, -1)) {
       kryo.writeClassAndObject(opt, obj);
       opt.flush();
       return opt.getBuffer();
  }finally {
       kryoPool.free(kryo);
  }
}
复制代码
```

## 小结

相较于 JDK 自带的序列化方式，Kryo 的性能更快，并且由于 Kryo 允许多引用和循环引用，在存储开销上也更小。

只不过，虽然 Kryo 拥有非常好的性能，但其自身却舍去了很多特性，例如线程安全、对序列化对象的字段修改等。虽然这些弊端可以通过 Kryo 良好的扩展性得到一定的满足，但是对于开发者来说仍然具有一定的上手难度，不过这并不能影响其在 Java 中的地位。

Kryo 还有很多优秀的特性，详情可参考 👉[EsotericSoftware/kryo](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2FEsotericSoftware%2Fkryo)

最后，如果你觉得本文对你有帮助的话，不要吝啬你的关注和点赞，也欢迎读者在评论区留言讨论，一起进步啊～