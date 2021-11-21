希望JSON Parser的性能有二进制协议一样好，比如和protobuf一样

使用https://github.com/eishay/jvm-serializers/提供的程序进行测试得到的结果：

|            | 序列化时间 | 反序列化时间 | 大小 | 压缩后大小 |
| ---------- | ---------- | ------------ | ---- | ---------- |
| java序列化 | 8654       | 43787        | 889  | 541        |
| hessian    | 6725       | 10460        | 501  | 313        |
| protobuf   | 2964       | 1745         | 239  | 149        |
| thrift     | 3177       | 1949         | 349  | 197        |
| avro       | 3520       | 1948         | 221  | 133        |
| json-lib   | 45788      | 149741       | 485  | 263        |
| jackson    | 3052       | 4161         | 503  | 271        |
| fastjson   | 2595       | 1472         | 468  | 251        |

这是一个 468bytes 的 JSON Bytes 测试，从测试结果来看，无论序列化和反序列化，Fastjson 超越了 protobuf， 它比 java deserialize 快超过 30 多倍，比 json-lib 快 100 倍。由于 Fastjson 的存在，你可以放心使用 json 统一协议，达到文本协议的可维护性，二进制协议的性能。

JSON 处理主要包括两个部分，serialize 和 deserialize。serialize 就是把 Java 对象变成 JSON String 或者 JSON Bytes。Deserialize 是把 JSON String 或者 Json Bytes 变成 java 对象。其实这个过程有些 JSON 库是分三部分的，json string <－-> json tree <－-> java object。Fastjson 也支持这种转换方式，但是这种转换方式因为有多余的步骤，性能不好，不推荐使用。

## 为什么Fastjson能够做到这么快？

### 一、Fastjson 中 Serialzie 的优化实现

- 自行编写类似 StringBuilder 的工具类 SerializeWriter。
    把 java 对象序列化成 json 文本，是不可能使用字符串直接拼接的，因为这样性能很差。比字符串拼接更好的办法是使用java.lang.StringBuilder。StringBuilder 虽然速度很好了，但还能够进一步提升性能的，fastjson 中提供了一个类似 StringBuilder的类 com.alibaba.fastjson.serializer.SerializeWriter。

    SerializeWriter 提供一些针对性的方法减少数组越界检查。例如 `public void writeIntAndChar(int i, char c) {}`，这样的方法一次性把两个值写到 buf 中去，能够减少一次越界检查。目前 SerializeWriter 还有一些关键的方法能够减少越界检查的还没实现。也就是说，如果实现了，能够进一步提升 serialize 的性能。

- 使用 ThreadLocal 来缓存 buf。
    这个办法能够减少对象分配和 gc，从而提升性能。SerializeWriter 中包含了一个 char[] buf，每序列化一次，都要做一次分配，使用 ThreadLocal 优化，能够提升性能。

- 使用 asm 避免反射
    获取 java bean 的属性值，需要调用反射，fastjson 引入了 asm 的来避免反射导致的开销。fastjson 内置的 asm 是基于objectweb asm 3.3.1 改造的，只保留必要的部分，fastjson asm 部分不到1000行代码，引入了 asm 的同时不导致大小变大太多。

- 使用一个特殊的 IdentityHashMap 优化性能。
    fastjson 对每种类型使用一种 serializer，于是就存在 class -> JavaBeanSerizlier 的映射。fastjson 使用 IdentityHashMap 而不是HashMap，避免 equals 操作。我们知道 HashMap 的算法的 transfer 操作，并发时可能导致死循环，但是 ConcurrentHashMap比 HashMap 系列会慢，因为其使用 volatile 和 lock。fastjson 自己实现了一个特别的 IdentityHashMap，去掉 transfer 操作的IdentityHashMap，能够在并发时工作，但是不会导致死循环。

- 缺省启用 sort field 输出
    json 的 object 是一种 key/value 结构，正常的 hashmap 是无序的，fastjson 缺省是排序输出的，这是为 deserialize 优化做准备。

- 集成 jdk 实现的一些优化算法
    在优化 fastjson 的过程中，参考了 jdk 内部实现的算法，比如 int to char[] 算法等等。

### 二、fastjson 的 deserializer 的主要优化算法

deserializer 也称为 parser 或者 decoder，fastjson 在这方面投入的优化精力最多。

- 读取 token 基于预测。
    所有的 parser 基本上都需要做词法处理，json 也不例外。fastjson 词法处理的时候，使用了基于预测的优化算法。比如 key 之后，最大的可能是冒号":"，value 之后，可能是有两个，逗号","或者右括号"}"。在 com.alibaba.fastjson.parser.JSONScanner 中提供了这样的方法：

    ```java
    public void nextToken(int expect) {  
        for (;;) {  
            switch (expect) {  
                case JSONToken.COMMA: //   
                    if (ch == ',') {  
                        token = JSONToken.COMMA;  
                        ch = buf[++bp];  
                        return;  
                    }  
    
                    if (ch == '}') {  
                        token = JSONToken.RBRACE;  
                        ch = buf[++bp];  
                        return;  
                    }  
    
                    if (ch == ']') {  
                        token = JSONToken.RBRACKET;  
                        ch = buf[++bp];  
                        return;  
                    }  
    
                    if (ch == EOI) {  
                        token = JSONToken.EOF;  
                        return;  
                    }  
                    break;  
                    // ... ...  
            }  
        } 
    ```

    从上面摘抄下来的代码看，基于预测能够做更少的处理就能够读取到 token。

- sort field fast match 算法
    fastjson 的 serialize 是按照 key 的顺序进行的，于是 fastjson 做 deserializer 时候，采用一种优化算法，就是假设 key/value 的内容是有序的，读取的时候只需要做 key 的匹配，而不需要把 key 从输入中读取出来。通过这个优化，使得 fastjson 在处理json 文本的时候，少读取超过 50% 的 token，这个是一个十分关键的优化算法。基于这个算法，使用 asm 实现，性能提升十分明显，超过 300％ 的性能提升。

    ```java
    { "id" : 123, "name" : "魏加流", "salary" : 56789.79}  
      ------      --------          ----------   
    ```

    在上面例子看，虚线标注的三个部分是 key，如果 key_id、key_name、key_salary 这三个 key 是顺序的，就可以做优化处理，这三个 key 不需要被读取出来，只需要比较就可以了。

    这种算法分两种模式，一种是快速模式，一种是常规模式。快速模式是假定 key 是顺序的，能快速处理，如果发现不能够快速处理，则退回常规模式。保证性能的同时，不会影响功能。

    在这个例子中，常规模式需要处理 13 个 token，快速模式只需要处理 6 个 token。

    实现 sort field fast match 算法的代码在这个类 `com.alibaba.fastjson.parser.deserializer.ASMDeserializerFactory.java`，是使用 asm 针对每种类型的 VO 动态创建一个类实现的。
    这里是有一个用于演示sort field fast match算法的代码：
    http://code.alibabatech.com/svn/fastjson/trunk/fastjson/src/test/java/data/media/ImageDeserializer.java

    ```java
    // 用于快速匹配的每个字段的前缀  
    char[] size_   = "\"size\":".toCharArray();  
    char[] uri_    = "\"uri\":".toCharArray();  
    char[] titile_ = "\"title\":".toCharArray();  
    char[] width_  = "\"width\":".toCharArray();  
    char[] height_ = "\"height\":".toCharArray();  
      
    // 保存parse开始时的lexer状态信息  
    int mark = lexer.getBufferPosition();  
    char mark_ch = lexer.getCurrent();  
    int mark_token = lexer.token();  
      
    int height = lexer.scanFieldInt(height_);  
    if (lexer.matchStat == JSONScanner.NOT_MATCH) {  
        // 退出快速模式, 进入常规模式  
        lexer.reset(mark, mark_ch, mark_token);  
        return (T) super.deserialze(parser, clazz);  
    }  
      
    String value = lexer.scanFieldString(size_);  
    if (lexer.matchStat == JSONScanner.NOT_MATCH) {  
        // 退出快速模式, 进入常规模式  
        lexer.reset(mark, mark_ch, mark_token);  
        return (T) super.deserialze(parser, clazz);  
    }  
    Size size = Size.valueOf(value);  
      
    // ... ...  
      
    // batch set  
    Image image = new Image();  
    image.setSize(size);  
    image.setUri(uri);  
    image.setTitle(title);  
    image.setWidth(width);  
    image.setHeight(height);  
      
    return (T) image;  
    ```

- 使用 asm 避免反射
    deserialize 的时候，会使用 asm 来构造对象，并且做 batch set，也就是说合并连续调用多个 setter 方法，而不是分散调用，这个能够提升性能。

- 对 utf-8 的 json bytes，针对性使用优化的版本来转换编码。
    这个类是com.alibaba.fastjson.util.UTF8Decoder，来源于JDK中的UTF8Decoder，但是它使用ThreadLocal Cache Buffer，避免转换时分配 char[] 的开销。
    ThreadLocal Cache 的实现是这个类 com.alibaba.fastjson.util.ThreadLocalCache。第一次1k，如果不够，会增长，最多增长到128k。

    ```java
    //代码摘抄自com.alibaba.fastjson.JSON  
    public static final <T> T parseObject(byte[] input, int off, int len, CharsetDecoder charsetDecoder, Type clazz,  
                                          Feature... features) {  
        charsetDecoder.reset();  
      
        int scaleLength = (int) (len * (double) charsetDecoder.maxCharsPerByte());  
        char[] chars = ThreadLocalCache.getChars(scaleLength); // 使用ThreadLocalCache，避免频繁分配内存  
      
        ByteBuffer byteBuf = ByteBuffer.wrap(input, off, len);  
        CharBuffer charByte = CharBuffer.wrap(chars);  
        IOUtils.decode(charsetDecoder, byteBuf, charByte);  
      
        int position = charByte.position();  
      
        return (T) parseObject(chars, position, clazz, features);  
    } 
    ```

- symbolTable 算法。
    我们看 xml 或者 javac 的 parser 实现，经常会看到有一个这样的东西 symbol table，它就是把一些经常使用的关键字缓存起来，在遍历 char[] 的时候，同时把 hash 计算好，通过这个 hash 值在 hashtable 中来获取缓存好的 symbol，避免创建新的字符串对象。这种优化在 fastjson 里面用在 key 的读取，以及 enum value 的读取。这是也是 parse 性能优化的关键算法之一。

    以下是摘抄自 JSONScanner 类中的代码，这段代码用于读取类型为 enum 的 value。

    ```java
    int hash = 0;  
    for (;;) {  
        ch = buf[index++];  
        if (ch == '\"') {  
            bp = index;  
            this.ch = ch = buf[bp];  
            strVal = symbolTable.addSymbol(buf, start, index - start - 1, hash); // 通过symbolTable来获得缓存好的symbol，包括fieldName、enumValue  
            break;  
        }  
          
        hash = 31 * hash + ch; // 在token scan的过程中计算好hash  
      
        // ... ...  
    }  
    ```