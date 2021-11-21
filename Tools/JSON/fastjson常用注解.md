# JSON API

JSON 这个类是 fastjson API 的入口，主要的功能都通过这个类提供。

## 一、序列化 API

```java
package com.alibaba.fastjson;

public abstract class JSON {
    // 将Java对象序列化为JSON字符串，支持各种各种Java基本类型和JavaBean
    public static String toJSONString(Object object, SerializerFeature... features);

    // 将Java对象序列化为JSON字符串，返回JSON字符串的utf-8 bytes
    public static byte[] toJSONBytes(Object object, SerializerFeature... features);

    // 将Java对象序列化为JSON字符串，写入到Writer中
    public static void writeJSONString(Writer writer, 
                                       Object object, 
                                       SerializerFeature... features);

    // 将Java对象序列化为JSON字符串，按UTF-8编码写入到OutputStream中
    public static final int writeJSONString(OutputStream os, // 
                                            Object object, // 
                                            SerializerFeature... features);
}
```

## 二、JSON字符串反序列化API

```java
package com.alibaba.fastjson;

public abstract class JSON {
    // 将JSON字符串反序列化为JavaBean
    public static <T> T parseObject(String jsonStr, 
                                    Class<T> clazz, 
                                    Feature... features);

    // 将JSON字符串反序列化为JavaBean
    public static <T> T parseObject(byte[] jsonBytes,  // UTF-8格式的JSON字符串
                                    Class<T> clazz, 
                                    Feature... features);

    // 将JSON字符串反序列化为泛型类型的JavaBean
    public static <T> T parseObject(String text, 
                                    TypeReference<T> type, 
                                    Feature... features);

    // 将JSON字符串反序列为JSONObject
    public static JSONObject parseObject(String text);
}
```

## 三、JSONField

### 1. JSONField 介绍

注意：1、若属性是私有的，必须有 set* 方法。否则无法反序列化。

```java
package com.alibaba.fastjson.annotation;

public @interface JSONField {
    // 配置序列化和反序列化的顺序，1.1.42版本之后才支持
    int ordinal() default 0;

    // 指定字段的名称
    String name() default "";

    // 指定字段的格式，对日期格式有用
    String format() default "";

    // 是否序列化
    boolean serialize() default true;

    // 是否反序列化
    boolean deserialize() default true;
}
```

### 2. JSONField配置方式

FieldInfo 可以配置在 getter/setter 方法或者字段上，即指定序列化时候该字段的 Key，例如：

- 配置在 getter/setter 上

    ```java
    public class A {
        private int id;
    
        @JSONField(name="ID")
        public int getId() {return id;}
        @JSONField(name="ID")
        public void setId(int value) {this.id = id;}
    }
    ```

- 配置在 field 上

    ```java
    public class A {
        @JSONField(name="ID")
        private int id;
    
        public int getId() {return id;}
        public void setId(int value) {this.id = id;}
    }
    ```

### 3. 使用format配置日期格式化

```java
public class A {
    // 配置date序列化和反序列使用yyyyMMdd日期格式
    @JSONField(format="yyyyMMdd")
    public Date date;
}
```

### 4. 使用serialize/deserialize指定字段不序列化

```java
// 方式一：分开设置  
public class A {
      @JSONField(serialize=false)
      public Date date;
 }

 public class A {
      @JSONField(deserialize=false)
      public Date date;
 }

// 方式二：一次性设置
public class A{
	@JSONField(serialize = false,deserialize = false)
    public Date date;
}
```

### 5. 使用ordinal指定字段的顺序

缺省 fastjson 序列化一 个java bean，**是根据 fieldName 的字母序进行序列化的**，你可以通过 ordinal 指定字段的顺序。这个特性需要 1.1.42 以上版本。

```java
public static class VO {
    @JSONField(ordinal = 3)
    private int f0;

    @JSONField(ordinal = 2)
    private int f1;

    @JSONField(ordinal = 1)
    private int f2;
}
```

### 6. 使用serializeUsing制定属性的序列化类

在 fastjson 1.2.16 版本之后，JSONField 支持新的定制化配置 serializeUsing，可以单独对某一个类的某个属性定制序列化，比如：

```java
public static class Model {
    @JSONField(serializeUsing = ModelValueSerializer.class)
    public int value;
}

// 这里的 object 是 value 值，不是 key
public static class ModelValueSerializer implements ObjectSerializer {
    @Override
    public void write(JSONSerializer serializer, Object object, Object fieldName, Type fieldType,
                      int features) throws IOException {
        Integer value = (Integer) object;
        String text = value + "元";
        serializer.write(text);
    }
}
```

测试代码

```java
Model model = new Model();
model.value = 100;
String json = JSON.toJSONString(model);
Assert.assertEquals("{\"value\":\"100元\"}", json);
```

## 四、JSONPath

### 1. JSONPath介绍

fastjson 1.2.0 之后的版本支持 JSONPath。这是一个很强大的功能，可以在 java 框架中当作对象查询语言（OQL）来使用。

### 2. API

```java
package com.alibaba.fastjson;

public class JSONPath {          
    //  求值，静态方法
    public static Object eval(Object rootObject, String path);

    //  求值，静态方法，按需计算，性能更好
    public static Object extract(String json, String path);

    // 计算Size，Map非空元素个数，对象非空元素个数，Collection的Size，数组的长度。其他无法求值返回-1
    public static int size(Object rootObject, String path);

    // 是否包含，path中是否存在对象
    public static boolean contains(Object rootObject, String path) { }

    // 是否包含，path中是否存在指定值，如果是集合或者数组，在集合中查找value是否存在
    public static boolean containsValue(Object rootObject, String path, Object value) { }

    // 修改制定路径的值，如果修改成功，返回true，否则返回false
    public static boolean set(Object rootObject, String path, Object value) {}

    // 在数组或者集合中添加元素
    public static boolean arrayAdd(Object rootObject, String path, Object... values);

    // 获取，Map的KeySet，对象非空属性的名称。数组、Collection等不支持类型返回null。
    public static Set<?> keySet(Object rootObject, String path);
}
```

建议缓存 JSONPath 对象，这样能够提高求值的性能。

### 3. 支持语法

| JSONPATH                  | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| $                         | 根对象，例如$.name                                           |
| [num]                     | 数组访问，其中num是数字，可以是负数。例如$[0].leader.departments[-1].name |
| [num0,num1,num2...]       | 数组多个元素访问，其中num是数字，可以是负数，返回数组中的多个元素。例如$[0,3,-2,5] |
| [start:end]               | 数组范围访问，其中start和end是开始小表和结束下标，可以是负数，返回数组中的多个元素。例如$[0:5] |
| [start:end :step]         | 数组范围访问，其中start和end是开始小表和结束下标，可以是负数；step是步长，返回数组中的多个元素。例如$[0:5:2] |
| [?(key)]                  | 对象属性非空过滤，例如$.departs[?(name)]                     |
| [key > 123]               | 数值类型对象属性比较过滤，例如$.departs[id >= 123]，比较操作符支持=,!=,>,>=,<,<= |
| [key = '123']             | 字符串类型对象属性比较过滤，例如$.departs[name = '123']，比较操作符支持=,!=,>,>=,<,<= |
| [key like 'aa%']          | 字符串类型like过滤， 例如$.departs[name like 'sz*']，通配符只支持% 支持not like |
| [key rlike 'regexpr']     | 字符串类型正则匹配过滤， 例如departs[name like 'aa(.)*']， 正则语法为jdk的正则语法，支持not rlike |
| [key in ('v0', 'v1')]     | IN过滤, 支持字符串和数值类型 例如: $.departs[name in ('wenshao','Yako')] $.departs[id not in (101,102)] |
| [key between 234 and 456] | BETWEEN过滤, 支持数值类型，支持not between 例如: $.departs[id between 101 and 201] $.departs[id not between 101 and 201] |
| length() 或者 size()      | 数组长度。例如$.values.size() 支持类型java.util.Map和java.util.Collection和数组 |
| keySet()                  | 获取Map的keySet或者对象的非空属性名称。例如$.val.keySet() 支持类型：Map和普通对象 不支持：Collection和数组（返回null） |
| .                         | 属性访问，例如$.name                                         |
| ..                        | deepScan属性访问，例如$..name                                |
| *                         | 对象的所有属性，例如$.leader.*                               |
| ['key']                   | 属性访问。例如$['name']                                      |
| ['key0','key1']           | 多个属性访问。例如$['id','name']                             |

以下两种写法的语义是相同的：

```
$.store.book[0].title
```

和

```
$['store']['book'][0]['title']
```

### 4. 语法示例

| JSONPath | 语义              |
| -------- | ----------------- |
| $        | 根对象            |
| $[-1]    | 最后元素          |
| $[:-2]   | 第1个至倒数第2个  |
| $[1:]    | 第2个之后所有元素 |
| $[1,2,3] | 集合中1,2,3个元素 |

### 5. API 示例

### 5.1 例1

```java
public void test_entity() throws Exception {
    Entity entity = new Entity(123, new Object());

    Assert.assertSame(entity.getValue(), JSONPath.eval(entity, "$.value")); 
    Assert.assertTrue(JSONPath.contains(entity, "$.value"));
    Assert.assertTrue(JSONPath.containsValue(entity, "$.id", 123));
    Assert.assertTrue(JSONPath.containsValue(entity, "$.value", entity.getValue())); 
    Assert.assertEquals(2, JSONPath.size(entity, "$"));
    Assert.assertEquals(0, JSONPath.size(new Object[], "$")); 
}

@Data
public static class Entity {
    private Integer id;
    private String name;
    private Object value;

    public Entity() {}
    public Entity(Integer id, Object value) { this.id = id; this.value = value; }
    public Entity(Integer id, String name) { this.id = id; this.name = name; }
    public Entity(String name) { this.name = name; }

    public Integer getId() { return id; }
    public Object getValue() { return value; }        
    public String getName() { return name; }

    public void setId(Integer id) { this.id = id; }
    public void setName(String name) { this.name = name; }
    public void setValue(Object value) { this.value = value; }
}
```

### 5.2 例2

读取集合多个元素的某个属性

```java
List<Entity> entities = new ArrayList<Entity>();
entities.add(new Entity("wenshao"));
entities.add(new Entity("ljw2083"));

List<String> names = (List<String>)JSONPath.eval(entities, "$.name"); // 返回enties的所有名称
Assert.assertSame(entities.get(0).getName(), names.get(0));
Assert.assertSame(entities.get(1).getName(), names.get(1));
```

### 5.3 例3

返回集合中多个元素

```java
List<Entity> entities = new ArrayList<Entity>();
entities.add(new Entity("wenshao"));
entities.add(new Entity("ljw2083"));
entities.add(new Entity("Yako"));

List<Entity> result = (List<Entity>)JSONPath.eval(entities, "[1,2]"); // 返回下标为1和2的元素
Assert.assertEquals(2, result.size());
Assert.assertSame(entities.get(1), result.get(0));
Assert.assertSame(entities.get(2), result.get(1));
```

### 5.4 例4

按范围返回集合的子集

```java
List<Entity> entities = new ArrayList<Entity>();
entities.add(new Entity("wenshao"));
entities.add(new Entity("ljw2083"));
entities.add(new Entity("Yako"));

List<Entity> result = (List<Entity>)JSONPath.eval(entities, "[0:2]"); // 返回下标从0到2的元素
Assert.assertEquals(3, result.size());
Assert.assertSame(entities.get(0), result.get(0));
Assert.assertSame(entities.get(1), result.get(1));
Assert.assertSame(entities.get(2), result.get(1));
```

### 5.5 例5

通过条件过滤，返回集合的子集

```java
List<Entity> entities = new ArrayList<Entity>();
entities.add(new Entity(1001, "ljw2083"));
entities.add(new Entity(1002, "wenshao"));
entities.add(new Entity(1003, "yakolee"));
entities.add(new Entity(1004, null));

List<Object> result = (List<Object>) JSONPath.eval(entities, "[id in (1001)]");
Assert.assertEquals(1, result.size());
Assert.assertSame(entities.get(0), result.get(0));
```

### 5.6 例6

根据属性值过滤条件判断是否返回对象，修改对象，数组属性添加元素

```java
Entity entity = new Entity(1001, "ljw2083");
Assert.assertSame(entity , JSONPath.eval(entity, "[id = 1001]"));
Assert.assertNull(JSONPath.eval(entity, "[id = 1002]"));

JSONPath.set(entity, "id", 123456); //将id字段修改为123456
Assert.assertEquals(123456, entity.getId().intValue());

JSONPath.set(entity, "value", new int[0]); //将value字段赋值为长度为0的数组
JSONPath.arrayAdd(entity, "value", 1, 2, 3); //将value字段的数组添加元素1,2,3
```

### 5.7 例7

```java
Map root = Collections.singletonMap("company", //
                                    Collections.singletonMap("departs", //
                                                             Arrays.asList( //
                                                                 Collections.singletonMap("id",
                                                                                          1001), //
                                                                 Collections.singletonMap("id",
                                                                                          1002), //
                                                                 Collections.singletonMap("id", 1003) //
                                                             ) //
                                                            ));

List<Object> ids = (List<Object>) JSONPath.eval(root, "$..id");
assertEquals(3, ids.size());
assertEquals(1001, ids.get(0));
assertEquals(1002, ids.get(1));
assertEquals(1003, ids.get(2));
```

### 5.8 例8 keySet

使用keySet抽取对象的属性名，null值属性的名字并不包含在keySet结果中，使用时需要注意，详细可参考示例。

```java
Entity e = new Entity();
e.setId(null);
e.setName("hello");
Map<String, Entity> map = Collections.singletonMap("e", e);
Collection<String> result;

// id is null, excluded by keySet
result = (Collection<String>)JSONPath.eval(map, "$.e.keySet()");
assertEquals(1, result.size());
Assert.assertTrue(result.contains("name"));

e.setId(1L);
result = (Collection<String>)JSONPath.eval(map, "$.e.keySet()");
Assert.assertEquals(2, result.size());
Assert.assertTrue(result.contains("id")); // included
Assert.assertTrue(result.contains("name"));

// Same result
Assert.assertEquals(result, JSONPath.keySet(map, "$.e"));
Assert.assertEquals(result, new JSONPath("$.e").keySet(map));
```

## 六、toJSONString

将 java 对象序列化为 JSON 字符串，fastjson提供了一个最简单的入口

```java
package com.alibaba.fastjson;

public abstract class JSON {
    public static String toJSONString(Object object);
}
```

### Sample

```java
import com.alibaba.fastjson.JSON;

Model model = new Model();
model.id = 1001;

String json = JSON.toJSONString(model);
```

## 七、writeJSONString

在1.2.11版本中，JSON类新增对OutputStream/Writer直接支持。

```java
package com.alibaba.fastjson;

public abstract class JSON {
    public static final int writeJSONString(OutputStream os, 
                                            Object object, 
                                            SerializerFeature... features) throws IOException;

    public static final int writeJSONString(OutputStream os, 
                                            Charset charset, 
                                            Object object, 
                                            SerializerFeature... features) throws IOException;

    public static final int writeJSONString(Writer os, 
                                            Object object, 
                                            SerializerFeature... features) throws IOException;
}
```

### Sample

```java
import com.alibaba.fastjson;
import java.nio.charset.Charset;
class Model {
    public int value;
}

Model model = new Model();
model.value = 1001;

OutputStream os = ...;
JSON.writeJSONString(os, model);
JSON.writeJSONString(os, Charset.from("GB18030"), model);

Writer writer = ...;
JSON.writeJSONString(os, model);
```



## 八、parseObjectInputStream

在 1.2.11 版本中，fastjson 新增加了对 InputStream 的支持支持。

```java
package com.alibaba.fastjson;

public abstract class JSON {
    public static <T> T parseObject(InputStream is, //
                                    Type type, //
                                    Feature... features) throws IOException;

    public static <T> T parseObject(InputStream is, //
                                    Charset charset, //
                                    Type type, //
                                    Feature... features) throws IOException;
}
```

### Sample

```java
import com.alibaba.fastjson;
import java.nio.charset.Charset;
class Model {
    public int value;
}

InputStream is = ...
Model model = JSON.parseObject(is, Model.class);
Model model2 = JSON.parseObject(is, Charset.from("UTF-8"), Model.class);
```



## 九、Stream

当需要处理超大JSON文本时，需要Stream API，在fastjson-1.1.32版本中开始提供Stream API。

### 序列化

- 超大 JSON 数组序列化

    如果你的 JSON 格式是一个巨大的 JSON 数组，有很多元素，则先调用 startArray，然后挨个写入对象，然后调用 endArray。

    ```java
    JSONWriter writer = new JSONWriter(new FileWriter("/tmp/huge.json"));
    writer.startArray();
    for (int i = 0; i < 1000 * 1000; ++i) {
        writer.writeValue(new VO());
    }
    writer.endArray();
    writer.close();
    ```

- 超大 JSON 对象序列化

    如果你的 JSON 格式是一个巨大的 JSONObject，有很多 Key/Valu e对，则先调用 startObject，然后挨个写入 Key 和 Value，然后调用 endObject。

    ```java
    JSONWriter writer = new JSONWriter(new FileWriter("/tmp/huge.json"));
    writer.startObject();
    for (int i = 0; i < 1000 * 1000; ++i) {
        writer.writeKey("x" + i);
        writer.writeValue(new VO());
    }
    writer.endObject();
    writer.close();
    ```

### 反序列化

```java
JSONReader reader = new JSONReader(new FileReader("/tmp/huge.json"));
reader.startArray();
while(reader.hasNext()) {
    VO vo = reader.readObject(VO.class);
    // handle vo ...
}
reader.endArray();
reader.close();
```

```java
JSONReader reader = new JSONReader(new FileReader("/tmp/huge.json"));
reader.startObject();
while(reader.hasNext()) {
    String key = reader.readString();
    VO vo = reader.readObject(VO.class);
    // handle vo ...
}
reader.endObject();
reader.close();
```

## 十、DataBind

基础 POJO

```java
@Data
public class Group {
    private Long       id;
    private String     name;
    private List<User> users = new ArrayList<User>();
}

@Data
public class User {
    private Long   id;
    private String name;
}
```

### Encode

```java
import com.alibaba.fastjson.JSON;

Group group = new Group();
group.setId(0L);
group.setName("admin");

User guestUser = new User();
guestUser.setId(2L);
guestUser.setName("guest");

User rootUser = new User();
rootUser.setId(3L);
rootUser.setName("root");

group.addUser(guestUser);
group.addUser(rootUser);

String jsonString = JSON.toJSONString(group);

System.out.println(jsonString);
```

输出结果为：

```json
{"id":0,"name":"admin","users":[{"id":2,"name":"guest"},{"id":3,"name":"root"}]}
```

### Decode

```java
String jsonString = ...;
Group group = JSON.parseObject(jsonString, Group.class);
```



## 十一、ParseProcess

### 简介

ParseProcess 是编程扩展**定制反序列化**的接口。fastjson 支持如下 ParseProcess：

- ExtraProcessor 用于处理多余的字段
- ExtraTypeProvider 用于处理多余字段时提供类型信息

### 使用ExtraProcessor 处理多余字段

```java
public static class VO {
    private int id;
    private Map<String, Object> attributes = new HashMap<String, Object>();
    public int getId() { return id; }
    public void setId(int id) { this.id = id;}
    public Map<String, Object> getAttributes() { return attributes;}
}

ExtraProcessor processor = new ExtraProcessor() {
    public void processExtra(Object object, String key, Object value) {
        VO vo = (VO) object;
        vo.getAttributes().put(key, value);
    }
};

VO vo = JSON.parseObject("{\"id\":123,\"name\":\"abc\"}", VO.class, processor);
Assert.assertEquals(123, vo.getId());
Assert.assertEquals("abc", vo.getAttributes().get("name"));
```

### 使用ExtraTypeProvider 为多余的字段提供类型

```java
public static class VO {
    private int id;
    private Map<String, Object> attributes = new HashMap<String, Object>();
    public int getId() { return id; }
    public void setId(int id) { this.id = id;}
    public Map<String, Object> getAttributes() { return attributes;}
}
    
class MyExtraProcessor implements ExtraProcessor, ExtraTypeProvider {
    public void processExtra(Object object, String key, Object value) {
        VO vo = (VO) object;
        vo.getAttributes().put(key, value);
    }
    
    public Type getExtraType(Object object, String key) {
        if ("value".equals(key)) {
            return int.class;
        }
        return null;
    }
};
ExtraProcessor processor = new MyExtraProcessor();
    
VO vo = JSON.parseObject("{\"id\":123,\"value\":\"123456\"}", VO.class, processor);
Assert.assertEquals(123, vo.getId());
Assert.assertEquals(123456, vo.getAttributes().get("value")); // value本应该是字符串类型的，通过getExtraType的处理变成Integer类型了。

```

## 十二、SerializeFilter

### 简介

SerializeFilter 是通过编程扩展的方式定制序列化。fastjson 支持 6 种 SerializeFilter，用于不同场景的定制序列化。

- PropertyPreFilter 根据 PropertyName 判断是否序列化
- PropertyFilter 根据 PropertyName 和 PropertyValue 来判断是否序列化
- NameFilter 修改 Key，如果需要修改 Key,process 返回值则可
- ValueFilter 修改 Value
- BeforeFilter 序列化时在最前添加内容
- AfterFilter 序列化时在最后添加内容

### PropertyFilter 根据PropertyName和PropertyValue来判断是否序列化

```java
  public interface PropertyFilter extends SerializeFilter {
      boolean apply(Object object, String propertyName, Object propertyValue);
  }
```

可以通过扩展实现根据object或者属性名称或者属性值进行判断是否需要序列化。例如：

```java
PropertyFilter filter = new PropertyFilter() {

    public boolean apply(Object source, String name, Object value) {
        if ("id".equals(name)) {
            int id = ((Integer) value).intValue();
            return id >= 100;
        }
        return false;
    }
};

JSON.toJSONString(obj, filter); // 序列化的时候传入filter
```

### PropertyPreFilter 根据PropertyName判断是否序列化

和PropertyFilter不同只根据object和name进行判断，在调用getter之前，这样避免了getter调用可能存在的异常。

```java
 public interface PropertyPreFilter extends SerializeFilter {
      boolean apply(JSONSerializer serializer, Object object, String name);
  }
```

### NameFilter 序列化时修改Key

如果需要修改Key,process返回值则可

```java
public interface NameFilter extends SerializeFilter {
    String process(Object object, String propertyName, Object propertyValue);
}
```

fastjson内置一个PascalNameFilter，用于输出将首字符大写的Pascal风格。 例如：

```java
import com.alibaba.fastjson.serializer.PascalNameFilter;

Object obj = ...;
String jsonStr = JSON.toJSONString(obj, new PascalNameFilter());
```

### ValueFilter 序列化时修改Value

```java
public interface ValueFilter extends SerializeFilter {
    Object process(Object object, String propertyName, Object propertyValue);
}
```

### BeforeFilter 序列化时在最前添加内容

> 在序列化对象的所有属性之前执行某些操作,例如调用 writeKeyValue 添加内容

```java
public abstract class BeforeFilter implements SerializeFilter {
    protected final void writeKeyValue(String key, Object value) { ... }
    // 需要实现的抽象方法，在实现中调用writeKeyValue添加内容
    public abstract void writeBefore(Object object);
}
```

### AfterFilter 序列化时在最后添加内容

> 在序列化对象的所有属性之后执行某些操作,例如调用 writeKeyValue 添加内容

```java
public abstract class AfterFilter implements SerializeFilter {
    protected final void writeKeyValue(String key, Object value) { ... }
    // 需要实现的抽象方法，在实现中调用writeKeyValue添加内容
    public abstract void writeAfter(Object object);
}
```

## 十三、BeanToArray

在 fastjson中 ，支持一种叫做 BeanToArray 的映射模式。普通模式下，JavaBean 映射成 json object，BeanToArray 模式映射为 json array。

### Sample 1

```java
class Model {
    public int id;
    public String name;
}

Model model = new Model();
model.id = 1001;
model.name = "gaotie";

// {"id":1001,"name":"gaotie"}
String text_normal = JSON.toJSONString(model); 

// [1001,"gaotie"]
String text_beanToArray = JSON.toJSONString(model, SerializerFeature.BeanToArray); 

// support beanToArray & normal mode
JSON.parseObject(text_beanToArray, Feature.SupportArrayToBean); 
```

**上面的例子中，BeanToArray 模式下，少了 Key 的输出，节省了空间，json 字符串较小，性能也会更好。**

### Sample 2

BeanToArray 可以局部使用，比如：

```java
class Company {
     public int code;
     public List<Department> departments = new ArrayList<Department>();
}

@JSONType(serialzeFeatures=SerializerFeature.BeanToArray, parseFeatures=Feature.SupportArrayToBean)
class Department {
     public int id;
     public String name;
     public Department() {}
     public Department(int id, String name) {this.id = id; this.name = name;}
}


Company company = new Company();
company.code = 100;
company.departments.add(new Department(1001, "Sales"));
company.departments.add(new Department(1002, "Financial"));

// {"code":10,"departments":[[1001,"Sales"],[1002,"Financial"]]}
String text = JSON.toJSONString(company); 
```

在这个例子中，如果Company的属性departments元素很多，局部采用BeanToArray就可以获得很好的性能，而整体又能够获得较好的可读性。

### Sample 3

上一个例子也可以这样写：

```java
class Company {
     public int code;

     @JSONField(serialzeFeatures=SerializerFeature.BeanToArray, parseFeatures=Feature.SupportArrayToBean)
     public List<Department> departments = new ArrayList<Department>();
}
```

### 性能

使用BeanToArray模式，可以获得媲美protobuf的性能。

```
                                   create     ser   deser   total   size  +dfl
protobuf                              244    2297    1296    3593    239   149
json/fastjson_array/databind          123    1289    1567    2856    281   163
msgpack/databind                      122    1525    2180    3705    233   146
json/fastjson/databind                120    2019    2610    4629    486   262
json/jackson+afterburner/databind     118    2142    3147    5289    485   261
json/jackson/databind                 124    2914    4411    7326    485   261
```

这里的json/fastjson_array/databind就是fastjson启用BeanToArray模式，total性能比protobuf好。具体看这里 https://github.com/alibaba/fastjson/wiki/Benchmark_1_2_11
