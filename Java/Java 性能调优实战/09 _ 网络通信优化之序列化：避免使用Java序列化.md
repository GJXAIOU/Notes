# 09 \| 网络通信优化之序列化：避免使用Java序列化

作者: 刘超

完成时间:

总结时间:

![](<https://static001.geekbang.org/resource/image/2c/1f/2c4f5698f25ab257c99be7642e04281f.jpg>)

<audio><source src="https://static001.geekbang.org/resource/audio/a5/ca/a5a59e3de030f425f1c2a019b4ddedca.mp3" type="audio/mpeg"></audio>

你好，我是刘超。

当前大部分后端服务都是基于微服务架构实现的。服务按照业务划分被拆分，实现了服务的解耦，但同时也带来了新的问题，不同业务之间通信需要通过接口实现调用。两个服务之间要共享一个数据对象，就需要从对象转换成二进制流，通过网络传输，传送到对方服务，再转换回对象，供服务方法调用。**这个编码和解码过程我们称之为序列化与反序列化。**

在大量并发请求的情况下，如果序列化的速度慢，会导致请求响应时间增加；而序列化后的传输数据体积大，会导致网络吞吐量下降。所以一个优秀的序列化框架可以提高系统的整体性能。

我们知道，Java提供了RMI框架可以实现服务与服务之间的接口暴露和调用，RMI中对数据对象的序列化采用的是Java序列化。<span class="orange">而目前主流的微服务框架却几乎没有用到Java序列化，SpringCloud用的是Json序列化，Dubbo虽然兼容了Java序列化，但默认使用的是Hessian序列化。这是为什么呢？</span>

今天我们就来深入了解下Java序列化，再对比近两年比较火的Protobuf序列化，看看Protobuf是如何实现最优序列化的。

## Java序列化

在说缺陷之前，你先得知道什么是Java序列化以及它的实现原理。

<!-- [[[read_end]]] -->

Java提供了一种序列化机制，这种机制能够将一个对象序列化为二进制形式（字节数组），用于写入磁盘或输出到网络，同时也能从网络或磁盘中读取字节数组，反序列化成对象，在程序中使用。

![](<https://static001.geekbang.org/resource/image/bd/e2/bd4bc4b2746f4b005ca26042412f4ee2.png?wh=3255*439>)

JDK提供的两个输入、输出流对象ObjectInputStream和ObjectOutputStream，它们只能对实现了Serializable接口的类的对象进行反序列化和序列化。

ObjectOutputStream的默认序列化方式，仅对对象的非transient的实例变量进行序列化，而不会序列化对象的transient的实例变量，也不会序列化静态变量。

在实现了Serializable接口的类的对象中，会生成一个serialVersionUID的版本号，这个版本号有什么用呢？它会在反序列化过程中来验证序列化对象是否加载了反序列化的类，如果是具有相同类名的不同版本号的类，在反序列化中是无法获取对象的。

具体实现序列化的是writeObject和readObject，通常这两个方法是默认的，当然我们也可以在实现Serializable接口的类中对其进行重写，定制一套属于自己的序列化与反序列化机制。

另外，Java序列化的类中还定义了两个重写方法：writeReplace()和readResolve()，前者是用来在序列化之前替换序列化对象的，后者是用来在反序列化之后对返回对象进行处理的。

## Java序列化的缺陷

如果你用过一些RPC通信框架，你就会发现这些框架很少使用JDK提供的序列化。其实不用和不好用多半是挂钩的，下面我们就一起来看看JDK默认的序列化到底存在着哪些缺陷。

### 1\.无法跨语言

现在的系统设计越来越多元化，很多系统都使用了多种语言来编写应用程序。比如，我们公司开发的一些大型游戏就使用了多种语言，C++写游戏服务，Java/Go写周边服务，Python写一些监控应用。

而Java序列化目前只适用基于Java语言实现的框架，其它语言大部分都没有使用Java的序列化框架，也没有实现Java序列化这套协议。因此，如果是两个基于不同语言编写的应用程序相互通信，则无法实现两个应用服务之间传输对象的序列化与反序列化。

### 2\.易被攻击

Java官网安全编码指导方针中说明：“对不信任数据的反序列化，从本质上来说是危险的，应该予以避免”。可见Java序列化是不安全的。

我们知道对象是通过在ObjectInputStream上调用readObject()方法进行反序列化的，这个方法其实是一个神奇的构造器，它可以将类路径上几乎所有实现了Serializable接口的对象都实例化。

这也就意味着，在反序列化字节流的过程中，该方法可以执行任意类型的代码，这是非常危险的。

对于需要长时间进行反序列化的对象，不需要执行任何代码，也可以发起一次攻击。攻击者可以创建循环对象链，然后将序列化后的对象传输到程序中反序列化，这种情况会导致hashCode方法被调用次数呈次方爆发式增长, 从而引发栈溢出异常。例如下面这个案例就可以很好地说明。

```
Set root = new HashSet();  
Set s1 = root;  
Set s2 = new HashSet();  
for (int i = 0; i < 100; i++) {  
   Set t1 = new HashSet();  
   Set t2 = new HashSet();  
   t1.add("foo"); //使t2不等于t1  
   s1.add(t1);  
   s1.add(t2);  
   s2.add(t1);  
   s2.add(t2);  
   s1 = t1;  
   s2 = t2;   
}
```

2015年FoxGlove Security安全团队的breenmachine发布过一篇长博客，主要内容是：通过Apache Commons Collections，Java反序列化漏洞可以实现攻击。一度横扫了WebLogic、WebSphere、JBoss、Jenkins、OpenNMS的最新版，各大Java Web Server纷纷躺枪。

其实，Apache Commons Collections就是一个第三方基础库，它扩展了Java标准库里的Collection结构，提供了很多强有力的数据结构类型，并且实现了各种集合工具类。

实现攻击的原理就是：Apache Commons Collections允许链式的任意的类函数反射调用，攻击者通过“实现了Java序列化协议”的端口，把攻击代码上传到服务器上，再由Apache Commons Collections里的TransformedMap来执行。

**那么后来是如何解决这个漏洞的呢？**

很多序列化协议都制定了一套数据结构来保存和获取对象。例如，JSON序列化、ProtocolBuf等，它们只支持一些基本类型和数组数据类型，这样可以避免反序列化创建一些不确定的实例。虽然它们的设计简单，但足以满足当前大部分系统的数据传输需求。

我们也可以通过反序列化对象白名单来控制反序列化对象，可以重写resolveClass方法，并在该方法中校验对象名字。代码如下所示：

```
@Override
protected Class resolveClass(ObjectStreamClass desc) throws IOException,ClassNotFoundException {
if (!desc.getName().equals(Bicycle.class.getName())) {

throw new InvalidClassException(
"Unauthorized deserialization attempt", desc.getName());
}
return super.resolveClass(desc);
}
```

### 3\.序列化后的流太大

序列化后的二进制流大小能体现序列化的性能。序列化后的二进制数组越大，占用的存储空间就越多，存储硬件的成本就越高。如果我们是进行网络传输，则占用的带宽就更多，这时就会影响到系统的吞吐量。

Java序列化中使用了ObjectOutputStream来实现对象转二进制编码，那么这种序列化机制实现的二进制编码完成的二进制数组大小，相比于NIO中的ByteBuffer实现的二进制编码完成的数组大小，有没有区别呢？

我们可以通过一个简单的例子来验证下：

```
User user = new User();
    	user.setUserName("test");
    	user.setPassword("test");
    	
    	ByteArrayOutputStream os =new ByteArrayOutputStream();
    	ObjectOutputStream out = new ObjectOutputStream(os);
    	out.writeObject(user);
    	
    	byte[] testByte = os.toByteArray();
    	System.out.print("ObjectOutputStream 字节编码长度：" + testByte.length + "\n");
```

```
ByteBuffer byteBuffer = ByteBuffer.allocate( 2048);

        byte[] userName = user.getUserName().getBytes();
        byte[] password = user.getPassword().getBytes();
        byteBuffer.putInt(userName.length);
        byteBuffer.put(userName);
        byteBuffer.putInt(password.length);
        byteBuffer.put(password);
        
        byteBuffer.flip();
        byte[] bytes = new byte[byteBuffer.remaining()];
    	System.out.print("ByteBuffer 字节编码长度：" + bytes.length+ "\n");
```

运行结果：

```
ObjectOutputStream 字节编码长度：99
ByteBuffer 字节编码长度：16
```

这里我们可以清楚地看到：Java序列化实现的二进制编码完成的二进制数组大小，比ByteBuffer实现的二进制编码完成的二进制数组大小要大上几倍。因此，Java序列后的流会变大，最终会影响到系统的吞吐量。

### 4\.序列化性能太差

序列化的速度也是体现序列化性能的重要指标，如果序列化的速度慢，就会影响网络通信的效率，从而增加系统的响应时间。我们再来通过上面这个例子，来对比下Java序列化与NIO中的ByteBuffer编码的性能：

```
User user = new User();
    	user.setUserName("test");
    	user.setPassword("test");
    	
    	long startTime = System.currentTimeMillis();
    	
    	for(int i=0; i<1000; i++) {
    		ByteArrayOutputStream os =new ByteArrayOutputStream();
        	ObjectOutputStream out = new ObjectOutputStream(os);
        	out.writeObject(user);
        	out.flush();
        	out.close();
        	byte[] testByte = os.toByteArray();
        	os.close();
    	}
    
    	
    	long endTime = System.currentTimeMillis();
    	System.out.print("ObjectOutputStream 序列化时间：" + (endTime - startTime) + "\n");
```

```
long startTime1 = System.currentTimeMillis();
    	for(int i=0; i<1000; i++) {
    		ByteBuffer byteBuffer = ByteBuffer.allocate( 2048);

            byte[] userName = user.getUserName().getBytes();
            byte[] password = user.getPassword().getBytes();
            byteBuffer.putInt(userName.length);
            byteBuffer.put(userName);
            byteBuffer.putInt(password.length);
            byteBuffer.put(password);
            
            byteBuffer.flip();
            byte[] bytes = new byte[byteBuffer.remaining()];
    	}
    	long endTime1 = System.currentTimeMillis();
    	System.out.print("ByteBuffer 序列化时间：" + (endTime1 - startTime1)+ "\n");
```

运行结果：

```
ObjectOutputStream 序列化时间：29
ByteBuffer 序列化时间：6
```

通过以上案例，我们可以清楚地看到：Java序列化中的编码耗时要比ByteBuffer长很多。

## 使用Protobuf序列化替换Java序列化

目前业内优秀的序列化框架有很多，而且大部分都避免了Java默认序列化的一些缺陷。例如，最近几年比较流行的FastJson、Kryo、Protobuf、Hessian等。**我们完全可以找一种替换掉Java序列化，这里我推荐使用Protobuf序列化框架。**

Protobuf是由Google推出且支持多语言的序列化框架，目前在主流网站上的序列化框架性能对比测试报告中，Protobuf无论是编解码耗时，还是二进制流压缩大小，都名列前茅。

Protobuf以一个 .proto 后缀的文件为基础，这个文件描述了字段以及字段类型，通过工具可以生成不同语言的数据结构文件。在序列化该数据对象的时候，Protobuf通过.proto文件描述来生成Protocol Buffers格式的编码。

**这里拓展一点，我来讲下什么是Protocol Buffers存储格式以及它的实现原理。**

Protocol Buffers 是一种轻便高效的结构化数据存储格式。它使用T-L-V（标识 - 长度 - 字段值）的数据格式来存储数据，T代表字段的正数序列(tag)，Protocol Buffers 将对象中的每个字段和正数序列对应起来，对应关系的信息是由生成的代码来保证的。在序列化的时候用整数值来代替字段名称，于是传输流量就可以大幅缩减；L代表Value的字节长度，一般也只占一个字节；V则代表字段值经过编码后的值。这种数据格式不需要分隔符，也不需要空格，同时减少了冗余字段名。

Protobuf定义了一套自己的编码方式，几乎可以映射Java/Python等语言的所有基础数据类型。不同的编码方式对应不同的数据类型，还能采用不同的存储格式。如下图所示：

![](<https://static001.geekbang.org/resource/image/ec/eb/ec0ebe4f622e9edcd9de86cb92f15eeb.jpg?wh=1638*466>)

对于存储Varint编码数据，由于数据占用的存储空间是固定的，就不需要存储字节长度 Length，所以实际上Protocol Buffers的存储方式是 T - V，这样就又减少了一个字节的存储空间。

Protobuf定义的Varint编码方式是一种变长的编码方式，每个字节的最后一位(即最高位)是一个标志位(msb)，用0和1来表示，0表示当前字节已经是最后一个字节，1表示这个数字后面还有一个字节。

对于int32类型数字，一般需要4个字节表示，若采用Varint编码方式，对于很小的int32类型数字，就可以用1个字节来表示。对于大部分整数类型数据来说，一般都是小于256，所以这种操作可以起到很好地压缩数据的效果。

我们知道int32代表正负数，所以一般最后一位是用来表示正负值，现在Varint编码方式将最后一位用作了标志位，那还如何去表示正负整数呢？如果使用int32/int64表示负数就需要多个字节来表示，在Varint编码类型中，通过Zigzag编码进行转换，将负数转换成无符号数，再采用sint32/sint64来表示负数，这样就可以大大地减少编码后的字节数。

Protobuf的这种数据存储格式，不仅压缩存储数据的效果好， 在编码和解码的性能方面也很高效。Protobuf的编码和解码过程结合.proto文件格式，加上Protocol Buffer独特的编码格式，只需要简单的数据运算以及位移等操作就可以完成编码与解码。可以说Protobuf的整体性能非常优秀。

## 总结

无论是网路传输还是磁盘持久化数据，我们都需要将数据编码成字节码，而我们平时在程序中使用的数据都是基于内存的数据类型或者对象，我们需要通过编码将这些数据转化成二进制字节流；如果需要接收或者再使用时，又需要通过解码将二进制字节流转换成内存数据。我们通常将这两个过程称为序列化与反序列化。

Java默认的序列化是通过Serializable接口实现的，只要类实现了该接口，同时生成一个默认的版本号，我们无需手动设置，该类就会自动实现序列化与反序列化。

Java默认的序列化虽然实现方便，但却存在安全漏洞、不跨语言以及性能差等缺陷，所以我强烈建议你避免使用Java序列化。

纵观主流序列化框架，FastJson、Protobuf、Kryo是比较有特点的，而且性能以及安全方面都得到了业界的认可，我们可以结合自身业务来选择一种适合的序列化框架，来优化系统的序列化性能。

## 思考题

这是一个使用单例模式实现的类，如果我们将该类实现Java的Serializable接口，它还是单例吗？<span class="orange">如果要你来写一个实现了Java的Serializable接口的单例，你会怎么写呢？</span>

```
public class Singleton implements Serializable{

    private final static Singleton singleInstance = new Singleton();

    private Singleton(){}

    public static Singleton getInstance(){
       return singleInstance; 
    }
}
```

期待在留言区看到你的见解。也欢迎你点击“请朋友读”，把今天的内容分享给身边的朋友，邀请他一起学习。



## 精选留言(35)

- ![img](https://static001.geekbang.org/account/avatar/00/16/a4/9c/b32ed9e9.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  陆离

  序列化会通过反射调用无参构造器返回一个新对象，破坏单例模式。 解决方法是添加readResolve()方法，自定义返回对象策略。

  作者回复: 回答正确

  2019-06-08

  **7

  **98

- ![img](https://static001.geekbang.org/account/avatar/00/11/30/c1/2dde6700.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  密码123456

  看到提问，才发现这竟然不是单例。回想内容是因为。可以把类路径上几乎所有实现了 Serializable 接口的对象都实例化。还真不知道怎么写？内部私有类实现，这种可以吗？

  作者回复: 线上代码发生错位了，已修正。 导致这个问题的原因是序列化中的readObject会通过反射，调用没有参数的构造方法创建一个新的对象。 所以我们可以在被序列化类中重写readResolve方法。 private Object readResolve(){        return singleInstance; }

  2019-06-08

  **6

  **23

- ![img](https://static001.geekbang.org/account/avatar/00/19/64/14/2f3263e3.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  彦峰你要加油啊！

  稍微看了下评论,解决方法是在被序列化类中重写readResolve方法, 但是通过Jdk源码可以看出,虽然readResolve方法返回实例解决了单例模式被破坏的问题, 但实际上还是实例化了两次,只不过新创建的对象没有被返回而已.   如果创建对象的动作发生频率加快,就意味着内存分配开销也会随之增大,应该使用注册式单例来解决这个问题.

  2019-12-30

  **5

  **10

- ![img](09%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E5%BA%8F%E5%88%97%E5%8C%96%EF%BC%9A%E9%81%BF%E5%85%8D%E4%BD%BF%E7%94%A8Java%E5%BA%8F%E5%88%97%E5%8C%96.resource/resize,m_fill,h_34,w_34-1662425648297-1721.jpeg)

  尔冬橙

   它会在反序列化过程中来验证序列化对象是否加载了反序列化的类，如果是具有相同类名的不同版本号的类，在反序列化中是无法获取对象的。老师，这句话能举个例子么，没太明白

  作者回复: 在Class类文件中默认会有一个serialNo作为序列化对象的版本号，无论在序列化方还是在反序列化方的class类文件中都存在一个默认序列号，在序列化时，会将该版本号加载进去，在反序列化时，会校验该版本号。

  2019-09-14

  **

  **9

- ![img](https://static001.geekbang.org/account/avatar/00/12/69/c6/513df085.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  强哥

  首先为什么单例要实现Serializable接口呢？如果本身就不合理，那直接删掉Serializable即可，没必要为了本身的不合理，添加多余的方法，除非有特殊场景，否则这么这样的代码指定会被ugly

  2019-06-19

  **

  **7

- ![img](https://static001.geekbang.org/account/avatar/00/12/e1/df/6e6a4c6b.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  kevin

  老师请教下，为什么spring cloud不使用protobuf, thrift等性能更高、支持跨平台的序列化工具，而且使用json？

  作者回复: springcloud是spring生态中的一部分，就目前spring生态很少引入非生态框架。但是我们可以自己实现springcloud兼容protobuf序列化。

  2019-06-09

  **

  **7

- ![img](https://static001.geekbang.org/account/avatar/00/12/79/4b/740f91ca.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  -W.LI-

  文中说Java序列化，不会序列化静态变量，这个单例的静态变量会被怎么处理啊?

  作者回复: 是的，Java序列化会调用构造函数，构造出一个新对象

  2019-06-08

  **2

  **7

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/ibvULfLvUITGs8lOOdibnc1loJg1mKhSv8iaZYYcObNmMdHSicNng9ia2nISPYYg1zHZFR0CSLpDia3jcWKcKDMrPVFA/132)

  waniz

  老师您好，Java序列化将数据转化为二进制字节流，json序列化将数据转化为json字符串。但是在物理层数据都是以电信号或模拟信号传输。那么从应用层到物理层数据的编码状态究竟是怎么变化的?出发点不同，最后都是二进制传输…忘解惑

  作者回复: Java序列化是将Java对象转化为二进制流，而Json序列化是将Json字符串转为二进制的过程，只是包装的数据格式不一样。

  2019-07-05

  **2

  **6

- ![img](09%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E5%BA%8F%E5%88%97%E5%8C%96%EF%BC%9A%E9%81%BF%E5%85%8D%E4%BD%BF%E7%94%A8Java%E5%BA%8F%E5%88%97%E5%8C%96.resource/resize,m_fill,h_34,w_34-1662425648298-1725.jpeg)

  天天向上

  想知道为什么用非Java的序列化方式的也需要实现Serializable接口？

  作者回复: 这是Java说明需要序列化的一种标识

  2019-12-28

  **

  **4

- ![img](https://static001.geekbang.org/account/avatar/00/0f/67/f4/9a1feb59.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  钱

  课后思考及问题 1：老师能深入细致的讲讲JAVA怎么实现序列化的嘛？比如：一个对象怎么一步步神奇的变成一个字节数组的？对象中的属性、属性值、方法、以及此对象的信息怎么变成一个字节数组的？ 2：我们知道对象是通过在 ObjectInputStream 上调用 readObject() 方法进行反序列化的，这个方法其实是一个神奇的构造器，它可以将类路径上几乎所有实现了 Serializable 接口的对象都实例化。 这个神奇的构造器的实现原理是啥？一个字节数组他怎么将其转换为一个对象的？很好奇，他知道字节数组多少位表示啥意思？然后一段一段的取，一段的翻译嘛？老师给讲讲呗？ 老师深入讲一下原理实现细节，API式的讲解不过瘾，和老师要深入理解的风格也不符呀😄

  作者回复: 序列化是将一个对象通过某种数据结构包装好对象中的具体属性和值，转换为二进制进行网络传输的一个过程。例如一个int类型的属性，数组是1000，转换为二进制则是4个字节的byte数组了。 后面我会使用一个具体的例子来优化讲解这一讲。

  2019-09-07

  **

  **3

- ![img](https://static001.geekbang.org/account/avatar/00/15/63/85/9ccf1b19.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  草帽路飞

  老师，您好。Java 序列化的安全性中，序列化的时候执行按段循环对象链的代码为什么会导致 hashcode 成倍增长呀？

  2019-06-19

  **1

  **3

- ![img](09%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E5%BA%8F%E5%88%97%E5%8C%96%EF%BC%9A%E9%81%BF%E5%85%8D%E4%BD%BF%E7%94%A8Java%E5%BA%8F%E5%88%97%E5%8C%96.resource/resize,m_fill,h_34,w_34-1662425648298-1728.jpeg)

  Sdylan

  2019.10.14 打卡：选择序列化四个原则：编解码效率、所占空间、安全、是否支持多语言

  2019-10-14

  **

  **2

- ![img](https://static001.geekbang.org/account/avatar/00/15/ff/0a/12faa44e.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  晓杰

  不是单例，因为在反序列化的时候，会调用ObjectInputStream的readObject方法，该方法可以对实现序列化接口的类进行实例化，所以会破坏单例模式。 可以通过重写readResolve，返回单例对象的方式来避免这个问题

  作者回复: 正确

  2019-06-08

  **2

  **1

- ![img](https://wx.qlogo.cn/mmopen/vi_32/DYAIOgq83eotSSnZic41tGkbflx0ogIg3ia6g2muFY1hCgosL2t3icZm7I8Ax1hcv1jNgr6vrZ53dpBuGhaoc6DKg/132)

  张学磊

  上面说默认序列化方式不会序列化对象的 transient 的实例变量，也不会序列化静态变量，那这个单例的变量是静态的，是不是可以理解序列化成了一个空对象？

  2019-06-08

  **

  **1

- ![img](https://static001.geekbang.org/account/avatar/00/12/62/d3/663de972.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  懵逼猴

  java.io.ObjectInputStream#readObject()反序列化不会调用构造函数

  2022-05-31

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/2a/39/6e/0ea71f8a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  清风@知酒烈

  老师，我们项目也是使用dubbo，但是实体类都需要实现java的Serializable接口，不实现就会报错。不是说dubbo默认不使用java的序列化吗，为什么还是要实现Serializable呢？

  2022-05-30

  **1

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1f/5d/82/81b2ba91.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  keep_it_real

  Protobuf序列化的时候需要生成相对于的java文件，感觉里面多了好多没用的东西。不知道是不是我没用对。

  2022-05-16

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/1e/89/e6/cf1ea14c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)![img](https://static001.geekbang.org/resource/image/eb/56/eb5afbf568af2917033e5a860de0b756.png?x-oss-process=image/resize,w_14)

  NOFX

  Json、Protocol 反序列化是不是也会存在破坏单例的行为呢？

  2021-08-30

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/0f/81/d4/e92abeb4.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Jecy-8

  文中的图，readResolve()方法应该是在反序列化（图中是序列化）之后的操作方法吧，不知道我理解对不对

  2021-04-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/19/50/ae/4970425c.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Eros

  序列化会通过反射调用无参构造器返回一个新对象，破坏单例模式。 解决方法是在被序列化类中重写readResolve方法。

  2021-03-21

  **

  **

- ![img](https://thirdwx.qlogo.cn/mmopen/vi_32/PiajxSqBRaELvqowyjk03bEDYiaXcly9fic79mLxFb1IRAbatOrsyeibOgI7ux7JhDXty69iav7S9x9JRN2esfEzf4Q/132)

  Geek_ad8fb9

  我觉得反序列化的时候并不是通过反射调用私有构造函数，通过在idea中测试，发现确实也没有调用私有构造函数。 JVM中有一个后门允许在不调用任何构造函数的情况下创建对象。首先将新对象的字段初始化为其默认值(false，0，null等)，然后对象反序列化代码使用对象流中的值填充字段。 readResolve()可以跳过这个后门，自定义返回对象。

  2020-09-12

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz7/7gAOQWRvYmUAZMAAAAAB/9sAhAABAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAQEBAgICAgICAgICAgIDAwMDAwMDAwMDAQEBAQEBAQIBAQICAgECAgMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwMDAwP/wAARCADuAO4DAREAAhEBAxEB/8QAfAABAAICAwEBAAAAAAAAAAAAAAYHBAgBAwUCCgEBAAAAAAAAAAAAAAAAAAAAABAAAgIBAgIECwQJBQAAAAAAAAECAwQRBSEGMWESF0FRgVITk+MUVJTUIkJiB5EyhBVFhbXFNnFygqJTEQEAAAAAAAAAAAAAAAAAAAAA/9oADAMBAAIRAxEAPwD9vAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAGHmbhg7fD0mbl4+LF69l32wrctPBCMmpTfUk2BG7ue+Wqm4rNsua6XTi5DWvVKyutPyaoDmnnrlq19l506W9NPTYuSk2/xQqnGPlaQElxM7Dzq/S4WVj5VfhlRbC1Rfil2G3GXU9GBlAAAAAAAAAAAAAAAAAAAAAAAAAAAA4bUU5SajGKblJtJJJattvgkkBVHMnP8AJSswtilHSLcLdxcVLV9DWHCWsdF/6ST1+6uiQFW35F+VbK/Jutvum9Z23WSssk/xTm3JgdIADvx8nIxLY34t9uPdD9W2myVc1412otPR6cV0MC1uWufvTTrwd8cITlpCrcYpQhKT4KOXBaQrbf346R8aXFgWmnrxXFPimvCAAAAAAAAAAAAAAAAAAAAAAAAAAFUfmBzHKLexYVjjrGMtxsg+LU12oYia6E4tSn400vOQFTAAAAAAAuDkDmSWRFbHm2OVtUHLb7ZvWU6oLWeK2+LdMV2ofgTX3UBaAAAAAAAAAAAAAAAAAAAAAAAABi52XDAwsvNs4wxce6+S10cvRQlNQX4ptaLrYGr+RfblX3ZN8nO7Itsutk/vWWSc5Pq4sDpAAAAAABlYWXbgZeNmUPS3Guruhx0TcJJ9mWnTGa4NeFMDaDGvrysejJqeteRTVfW/HC2EbI/9ZAdwAAAAAAAAAAAAAAAAAAAAAACJc8WurlncOzwdrxateqeVT2v0wTXlA18AAAAAAAAAbFcnXSu5a2mcnq402U/8cfJuoivJGtASYAAAAAAAAAAAAAAAAAAAAAABFOdqXdyzuSjxlWse7yVZVMp/or1YGvQAAAAAAAADY3lGiWPy3tNclo5Yzv8AF9nJusyYvyxtQEjAAAAAAAAAAAAAAAAAAAAAAAdGVj15eNkYty1qyaLaLF4exbCVctOvSXADWDNxLsDLycLIj2bsa6dM/E3B6KUfHCa0afhTAxQAAAAAAZ224Nu55+LgUp+kyboV6pa9iDetljXm1VpyfUgNnqaoUU1UVLs101wqrj4oVxUILyRQHYAAAAAAAAAAAAAAAAAAAAAAAAVrz5yzPNh++cGtzyaK1HNpgtZX0QX2bopcZW0R4NdLhp5ujCmQAAAAAAXbyLyzPbaXumdW4ZuVX2aKprSWNjS0bck+Mbr9FqumMeHS2gLDAAAAAAAAAAAAAAAAAAAAAAAAAACuOZOQ6c+dmbtDrxcubc7cWX2cbIm+LlW0n7vbLw8OxJ+bxbCpM7bM/bLXVn4l+NPVpekg1CenhrsWtdseuLaAwQAHo7ftO47raqsDEuyZapSlCOlVevhtul2aql/uaAt3lrkWjbJ15u5yry86Gk6qYrXFxpripfaSd90X0NpRi+hNpSAsIAAAAAAAAAAAAAAAAAAAAAAAAAAAAD4sqrug67a4W1y/WhZCM4P/AFjJNMDw7eVuXrm5T2jCTfT6Kr0C49VLrQHNPK/L1ElKvaMJtcU7alfo/Gle7FqB7cK4VQVdcIVwitIwhFQhFeJRikkgPsAAAAAAAAAAAAAAAAAAAAAAAAAY2XmYuBRPKzL68aiv9ay2XZjq+iKXTKcvBFJt+BARGf5g8uRk4q3LsSeinDFkoy60pyhPR9aQHz3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAHeHy752b8r7QB3h8u+dm/K+0Ad4fLvnZvyvtAMjG575cybY1PKtxnJpRnk0Trq1fglZHtxrXXLRLxgTCMozjGUZKUZJSjKLTjKLWqlFrVNNPgwOQAAAAAAAAAAAAAAAAAAAAUZ+YW43ZG9ywHOSx9vqpUa9fsu7IphkTta8MnCyMepLrYECAAAAAAAAAAAF0/lxuN2Tt+Zg2zlOO320uhyerhTlK1qpPzYWUSa8Xa06NALHAAAAAAAAAAAAAAAAAAAABr3zx/lO6fsX9OxAImAAAAAAAAAAALY/K/+Ofyz+4AWwAAAAAAAAAAAAAAAAAAAADXvnj/ACndP2L+nYgETAAAAAAAAAAAFsflf/HP5Z/cALYAAAAAAAAAAAAAAAAAAAABVvMfJG7bxvOZuONkbdCjI937Eb7cmNq9DiUUS7Ua8S2C1nU2tJPgB4fdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAHdrvvxe0+vzPoAJvyby1ncvfvH323Et989z9F7rZdPs+7+9dvt+loo019OtNNfD0ATcAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//Z)

  耿嘉艺

  思考题中除了重写readResolve方法，是不是重写readObject方法也可以？还有可不可以讲一下在反序列化代码中，哪里显示了通过反射调用了无参构造方法，最好能给下该代码在源码中的位置，谢谢

  2020-06-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/cc/0d/89435926.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  10年以后

  序列化

  2020-03-29

  **

  **

- ![img](09%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E5%BA%8F%E5%88%97%E5%8C%96%EF%BC%9A%E9%81%BF%E5%85%8D%E4%BD%BF%E7%94%A8Java%E5%BA%8F%E5%88%97%E5%8C%96.resource/resize,m_fill,h_34,w_34-1662425648297-1721.jpeg)

  尔冬橙

  序列化对象中加载反序列化的类怎么理解？

  作者回复: 我也没有看懂这句话，请问在哪里看到的？

  2020-02-15

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/15/4b/92/03338a22.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  王圣军

  我们有时基于网络通讯的传输，有的不是直接使用实例序列化后传输，很多就是字符串转换为二进制流进行传输，这个应该是不涉及到序列化和反序列化的吧？

  作者回复: 这也是一个序列化过程，只不过是字符串对象转二进制的过程

  2019-12-26

  **

  **

- ![img](http://thirdwx.qlogo.cn/mmopen/vi_32/SM4fwn9uFicXU8cQ1rNF2LQdKNbZI1FX1jmdwaE2MTrBawbugj4TQKjMKWG0sGbmqQickyARXZFS8NZtobvoWTHA/132)

  td901105

  老师我想问一下如果使用非Java的序列化方式的话需要实现Serializable接口吗？

  作者回复: 需要的

  2019-12-05

  **3

  **

- ![img](09%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E5%BA%8F%E5%88%97%E5%8C%96%EF%BC%9A%E9%81%BF%E5%85%8D%E4%BD%BF%E7%94%A8Java%E5%BA%8F%E5%88%97%E5%8C%96.resource/resize,m_fill,h_34,w_34-1662425648297-1721.jpeg)

  尔冬橙

  那么java的NIO用到了java的序列化和反序列化么？有一样的问题么

  作者回复: NIO是一种通信模型，并没有包含序列化的内容，可以自己选择使用哪一种序列化

  2019-09-14

  **

  **

- ![img](09%20_%20%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1%E4%BC%98%E5%8C%96%E4%B9%8B%E5%BA%8F%E5%88%97%E5%8C%96%EF%BC%9A%E9%81%BF%E5%85%8D%E4%BD%BF%E7%94%A8Java%E5%BA%8F%E5%88%97%E5%8C%96.resource/resize,m_fill,h_34,w_34-1662425648297-1721.jpeg)

  尔冬橙

   老师，所有在网络中传输信息都是要序列化么

  作者回复: 是的，不序列化与反序列化，则无法在内存中获取具体的信息用于业务逻辑中。

  2019-09-14

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/9c/f4/7e14ff8a.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Jerry

  老师，我试着写了一下这个问题的代码，但是发现Singleton 序列化和反序列化后会产生两个instance, 这个问题怎么解？以下是我的代码， ''' ... public static void serializeMe() {        try {            ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("SingletonData.txt"));            oos.writeObject(Singleton.getInstance());        } catch (IOException e) {            e.printStackTrace();        }    }     public static Object deSerializeMe() {        Object obj = null;        try {            ObjectInputStream ois = new ObjectInputStream(new FileInputStream("SingletonData.txt"));            obj = ois.readObject();        }  catch (IOException | ClassNotFoundException e) {            e.printStackTrace();        }        return obj;    }     private static class Singleton implements Serializable {        int i;        private static Singleton instance = null;        private Singleton() {            System.out.println("Executing constructor");            i = 1;        }         // key change here !!!        private void readObject(ObjectInputStream ois) throws IOException, ClassNotFoundException {            System.out.println("readObject before [i=" + i + "]");            ois.defaultReadObject();            instance = this; // After de-serialization, two objects of 'Singleton' class are created.            System.out.println("readObject after [i=" + i + "]");        }         // thread unsafe here, don't care        public static Singleton getInstance() {            if (instance == null) {                instance = new Singleton();            }            System.out.println("An instance is returned");            return instance;        }         // readResolve() is usually called after readObject()        public Object readResolve() {            System.out.println("Executing readResolve");            return instance;        }         @Override        public String toString() {            return "Singleton [i=" + i + "]";        }    } '''

  2019-07-25

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/17/1c/a2/2c3572de.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  辉

  hession是通过什么做序列化的？

  作者回复: hessian是用的hessian2序列化。

  2019-07-19

  **

  **

- ![img](data:image/jpeg;base64,/9j/4QAYRXhpZgAASUkqAAgAAAAAAAAAAAAAAP/sABFEdWNreQABAAQAAABkAAD/4QN5aHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wLwA8P3hwYWNrZXQgYmVnaW49Iu+7vyIgaWQ9Ilc1TTBNcENlaGlIenJlU3pOVGN6a2M5ZCI/PiA8eDp4bXBtZXRhIHhtbG5zOng9ImFkb2JlOm5zOm1ldGEvIiB4OnhtcHRrPSJBZG9iZSBYTVAgQ29yZSA1LjYtYzE0MCA3OS4xNjA0NTEsIDIwMTcvMDUvMDYtMDE6MDg6MjEgICAgICAgICI+IDxyZGY6UkRGIHhtbG5zOnJkZj0iaHR0cDovL3d3dy53My5vcmcvMTk5OS8wMi8yMi1yZGYtc3ludGF4LW5zIyI+IDxyZGY6RGVzY3JpcHRpb24gcmRmOmFib3V0PSIiIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bXBNTTpPcmlnaW5hbERvY3VtZW50SUQ9InhtcC5kaWQ6YWE3YmZhMDItMzBhMC00MDg3LTg3MmYtOGMwMjMxNjNhZWRjIiB4bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjI2MTlEODM3NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjI2MTlEODM2NTgzMTExRTk5NDY4Qjk3QUFCNDFBN0QzIiB4bXA6Q3JlYXRvclRvb2w9IkFkb2JlIFBob3Rvc2hvcCBDQyAyMDE1IChNYWNpbnRvc2gpIj4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6OTYyRTNCMDNBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6OTYyRTNCMDRBREI4MTFFOEFFNTJDODlGREQ1OTUzMDMiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRm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)

  17702158422

  可以定义一个静态属性 boolean flag = false, 在构造函数里 判断 flag是否为ture, 如果为true则抛出异常，否则将flag赋值为 true ，则可以在运行期防止反序列化时通过反射破坏单例模式

  2019-06-19

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/4f/78/c3d8ecb0.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  undifined

  老师 我们有一个需求，需要将一些更新前的数据保存起来用于回滚，保存的对象有一个 value 属性是 Object 类型的，赋值 BigDecimal 后使用 FastJson 序列化保存到数据库，回滚的时候再反序列化变成了Integer，考虑将 FastJson 改成 JDK 的序列化，但是又担心会造成性能问题，请问老师有什么建议吗

  作者回复: 请问改成JDK序列化的目的是什么？

  2019-06-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/10/bb/f1061601.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Demon.Lee![img](https://static001.geekbang.org/resource/image/89/43/89yyff4c4c2e2b73ce4931bb01a6a943.png)

  图一中，输入流ObjectInputStream应该是反序列吧，输出流ObjectOutputStream应该是序列化吧，老师我理解错了？

  作者回复: 是的，你理解没有错。ObjectInputStream对应readObject，ObjectOutputStream对应writeObject。

  2019-06-11

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/10/b3/c5/7fc124e2.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  Liam

  在java序列号安全性那里有个疑问，为什么反序列化会导致hashCode方法的频繁调用呢，反序列化时调用hashCode的作用是

  作者回复: 这里不是序列化调用hashcode方法，而是序列化时，运行这段代码。

  2019-06-09

  **

  **

- ![img](https://static001.geekbang.org/account/avatar/00/13/b7/ae/a25fcb73.jpg?x-oss-process=image/resize,m_fill,h_34,w_34)

  colin

  Protobuf的格式感觉喝字节码有点类似

  作者回复: 这个形容非常到位

  2019-06-08
