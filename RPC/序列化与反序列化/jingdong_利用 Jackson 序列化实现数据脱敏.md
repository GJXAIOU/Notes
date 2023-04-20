# 利用 Jackson 序列化实现数据脱敏

> 原文：https://xie.infoq.cn/article/be6766cef2a14370b06bb8db9

## 1.背景

在项目中有些敏感信息不能直接展示，比如客户手机号、身份证、车牌号等信息，展示时均需要进行数据脱敏，防止泄露客户隐私。脱敏即是对数据的部分信息用脱敏符号（*）处理。

## 2.目标

- 在服务端返回数据时，利用 Jackson 序列化完成数据脱敏，达到对敏感信息脱敏展示。
- 降低重复开发量，提升开发效率
- 形成统一有效的脱敏规则
- 可基于重写默认脱敏实现的 desensitize 方法，实现可扩展、可自定义的个性化业务场景的脱敏需求

## 3.主要实现

## 3.1 基于 Jackson 的自定义脱敏序列化实现

**StdSerializer**：所有标准序列化程序所使用的基类，这个是编写自定义序列化程序所推荐使用的基类。

**ContextualSerializer：**是 Jackson 提供的另一个序列化相关的接口，它的作用是通过字段已知的上下文信息定制 JsonSerializer。



```java
package com.jd.ccmp.ctm.constraints.serializer;




import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.databind.BeanProperty;
import com.fasterxml.jackson.databind.JsonSerializer;
import com.fasterxml.jackson.databind.SerializerProvider;
import com.fasterxml.jackson.databind.ser.ContextualSerializer;
import com.fasterxml.jackson.databind.ser.std.StdSerializer;
import com.jd.ccmp.ctm.constraints.Symbol;
import com.jd.ccmp.ctm.constraints.annotation.Desensitize;
import com.jd.ccmp.ctm.constraints.desensitization.Desensitization;
import com.jd.ccmp.ctm.constraints.desensitization.DesensitizationFactory;
import com.jd.ccmp.ctm.constraints.desensitization.DefaultDesensitization;




import java.io.IOException;




/**
 * 脱敏序列化器
 *
 * @author zhangxiaoxu15
 * @date 2022/2/8 11:10
 */
public class ObjectDesensitizeSerializer extends StdSerializer<Object> implements ContextualSerializer {
    private static final long serialVersionUID = -7868746622368564541L;
    private transient Desensitization<Object> desensitization;
    protected ObjectDesensitizeSerializer() {
        super(Object.class);
    }
    public Desensitization<Object> getDesensitization() {
        return desensitization;
    }
    public void setDesensitization(Desensitization<Object> desensitization) {
        this.desensitization = desensitization;
    }
    @Override
    public JsonSerializer<Object> createContextual(SerializerProvider prov, BeanProperty property) {
//获取属性注解
        Desensitize annotation = property.getAnnotation(Desensitize.class);
        return createContextual(annotation.desensitization());
    }
    @SuppressWarnings("unchecked")
    public JsonSerializer<Object> createContextual(Class<? extends Desensitization<?>> clazz) {
        ObjectDesensitizeSerializer serializer = new ObjectDesensitizeSerializer();
        if (clazz != DefaultDesensitization.class) {
            serializer.setDesensitization((Desensitization<Object>) DesensitizationFactory.getDesensitization(clazz));
        }
        return serializer;
    }
    @Override
    public void serialize(Object value, JsonGenerator gen, SerializerProvider provider) throws IOException {
        Desensitization<Object> objectDesensitization = getDesensitization();
        if (objectDesensitization != null) {
            try {
                gen.writeObject(objectDesensitization.desensitize(value));
            } catch (Exception e) {
                gen.writeObject(value);
            }
        } else if (value instanceof String) {
            gen.writeString(Symbol.getSymbol(((String) value).length(), Symbol.STAR));
        } else {
            gen.writeObject(value);
        }

```



注：createContextual 可以获得字段的类型以及注解。当字段拥有自定义注解时，取出注解中的值创建定制的序列化方式，这样在 serialize 方法中便可以得到这个值了。**createContextual** 方法只会在第一次序列化字段时调用（因为字段的上下文信息在运行期不会改变），所以无需关心性能问题。

### 3.2 定义脱敏接口、以及工厂实现

#### 3.2.1 脱敏器接口定义

```text
package com.jd.ccmp.ctm.constraints.desensitization;


/**
 * 脱敏器
 *
 * @author zhangxiaoxu15
 * @date 2022/2/8 10:56
 */
public interface Desensitization<T> {
    /**
     * 脱敏实现
     *
     * @param target 脱敏对象
     * @return 脱敏返回结果
     */
    T desensitize(T target);
}

```

#### 3.2.2 脱敏器工厂实现

```text
package com.jd.ccmp.ctm.constraints.desensitization;


import java.util.HashMap;
import java.util.Map;


/**
 * 工厂方法
 *
 * @author zhangxiaoxu15
 * @date 2022/2/8 10:58
 */
public class DesensitizationFactory {
    private DesensitizationFactory() {
    }
    private static final Map<Class<?>, Desensitization<?>> map = new HashMap<>();




    @SuppressWarnings("all")
    public static Desensitization<?> getDesensitization(Class<?> clazz) {
        if (clazz.isInterface()) {
            throw new UnsupportedOperationException("desensitization is interface, what is expected is an implementation class !");
        }
        return map.computeIfAbsent(clazz, key -> {
            try {
                return (Desensitization<?>) clazz.newInstance();
            } catch (InstantiationException | IllegalAccessException e) {
                throw new UnsupportedOperationException(e.getMessage(), e);
            }
        });
```

### 3.3 常用的脱敏器实现

#### 3.3.1 默认脱敏实现

可基于默认实现，扩展实现个性化场景



```text
package com.jd.ccmp.ctm.constraints.desensitization;


/**
 * 默认脱敏实现
 *
 * @author zhangxiaoxu15
 * @date 2022/2/8 11:01
 */
public interface DefaultDesensitization extends Desensitization<String> {
}

```

#### 3.3.2 手机号脱敏器

实现对手机号中间 4 位号码脱敏



```text
package com.jd.ccmp.ctm.constraints.desensitization;
import com.jd.ccmp.ctm.constraints.Symbol;
import java.util.regex.Matcher;
import java.util.regex.Pattern;


/**
 * 手机号脱敏器，保留前3位和后4位
 *
 * @author zhangxiaoxu15
 * @date 2022/2/8 11:02
 */
public class MobileNoDesensitization implements DefaultDesensitization {
    /**
     * 手机号正则
     */
    private static final Pattern DEFAULT_PATTERN = Pattern.compile("(13[0-9]|14[579]|15[0-3,5-9]|16[6]|17[0135678]|18[0-9]|19[89])\\d{8}");




    @Override
    public String desensitize(String target) {
        Matcher matcher = DEFAULT_PATTERN.matcher(target);
        while (matcher.find()) {
            String group = matcher.group();
            target = target.replace(group, group.substring(0, 3) + Symbol.getSymbol(4, Symbol.STAR) + group.substring(7, 11));
        }
        return target;

```

### 3.4 注解定义

通过 @JacksonAnnotationsInside 实现自定义注解，提高易用性



```text
package com.jd.ccmp.ctm.constraints.annotation;
import com.fasterxml.jackson.annotation.JacksonAnnotationsInside;
import com.fasterxml.jackson.databind.annotation.JsonSerialize;
import com.jd.ccmp.ctm.constraints.desensitization.Desensitization;
import com.jd.ccmp.ctm.constraints.serializer.ObjectDesensitizeSerializer;
import java.lang.annotation.*;


/**
 * 脱敏注解
 *
 * @author zhangxiaoxu15
 * @date 2022/2/8 11:09
 */
@Target({ElementType.FIELD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@JacksonAnnotationsInside
@JsonSerialize(using = ObjectDesensitizeSerializer.class)
@Documented
public @interface Desensitize {
    /**
     * 对象脱敏器实现
     */
    @SuppressWarnings("all")
    Class<? extends Desensitization<?>> desensitization();

```

#### 3.4.1 默认脱敏注解

```text
package com.jd.ccmp.ctm.constraints.annotation;
import com.fasterxml.jackson.annotation.JacksonAnnotationsInside;
import com.jd.ccmp.ctm.constraints.desensitization.DefaultDesensitization;
import java.lang.annotation.*;




/**
 * 默认脱敏注解
 *
 * @author zhangxiaoxu15
 * @date 2022/2/8 11:14
 */
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@JacksonAnnotationsInside
@Desensitize(desensitization = DefaultDesensitization.class)
@Documented
public @interface DefaultDesensitize {

```

#### 3.4.2 手机号脱敏注解

```text
package com.jd.ccmp.ctm.constraints.annotation;
import com.fasterxml.jackson.annotation.JacksonAnnotationsInside;
import com.jd.ccmp.ctm.constraints.desensitization.MobileNoDesensitization;
import java.lang.annotation.*;


/**
 * 手机号脱敏
 *
 * @author zhangxiaoxu15
 * @date 2022/2/8 11:18
 */
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@JacksonAnnotationsInside
@Desensitize(desensitization = MobileNoDesensitization.class)
@Documented
public @interface MobileNoDesensitize {
}

```

### 3.5 定义脱敏符号

支持指定脱敏符号，例如* 或是 ^_^



```text
package com.jd.ccmp.ctm.constraints;
import java.util.stream.Collectors;
import java.util.stream.IntStream;


/**
 * 脱敏符号
 *
 * @author zhangxiaoxu15
 * @date 2022/2/8 10:53
 */
public class Symbol {
    /**
     * '*'脱敏符
     */
    public static final String STAR = "*";
    private Symbol() {}
    /**
     * 获取符号
     *
     * @param number 符号个数
     * @param symbol 符号
     */
    public static String getSymbol(int number, String symbol) {
        return IntStream.range(0, number).mapToObj(i -> symbol).collect(Collectors.joining());
    }

```

## 4.使用样例 &执行流程剖析

![img](jingdong_%E5%88%A9%E7%94%A8%20Jackson%20%E5%BA%8F%E5%88%97%E5%8C%96%E5%AE%9E%E7%8E%B0%E6%95%B0%E6%8D%AE%E8%84%B1%E6%95%8F.resource/ef9eae02f241e1609e5cef0a9d0bf9f1.png)



**程序类图**



![img](jingdong_%E5%88%A9%E7%94%A8%20Jackson%20%E5%BA%8F%E5%88%97%E5%8C%96%E5%AE%9E%E7%8E%B0%E6%95%B0%E6%8D%AE%E8%84%B1%E6%95%8F.resource/e13e616e0824678d5f5d87c047e5362a.png)



```undefined
**执行流程剖析**
 1.调用JsonUtil.toJsonString()开始执行序列化
 2.识别属性mobile上的注解@MobileNoDesensitize(上文3.4.2)
 3.调用ObjectDesensitizeSerializer#createContextual(上文3.1 & 3.2)，返回JsonSerializer
 4.调用手机号脱敏实现MobileNoDesensitization#desensitize(上文3.3.2)
 5.输出脱敏后的序列化结果，{"mobile":"133****5678"}
```

不难发现核心执行流程是第 3 步，但是 @MobileNoDesensitize 与 ObjectDesensitizeSerializer 又是如何联系起来的呢？

- 尝试梳理下引用链路：@MobileNoDesensitize -> @Desensitize -> @JsonSerialize -> ObjectDesensitizeSerializer
- 但是，在 ObjectDesensitizeSerializer 的实现中，我们似乎却没有发现上述链路的直接调用关系
- 这就不得不说下 Jackson 元注解的概念

```text
**Jackson元注解**
1.提到元注解这个词，大家会想到@Target、@Retention、@Documented、@Inherited
2.Jackson也以同样的思路设计了@JacksonAnnotationsInside


/**
 * Meta-annotation (annotations used on other annotations)
 * used for indicating that instead of using target annotation
 * (annotation annotated with this annotation),
 * Jackson should use meta-annotations it has.
 * This can be useful in creating "combo-annotations" by having
 * a container annotation, which needs to be annotated with this
 * annotation as well as all annotations it 'contains'.
 * 
 * @since 2.0
 */
@Target({ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@JacksonAnnotation
public @interface JacksonAnnotationsInside
{
}

```



正是通过”combo-annotations”(组合注解、捆绑注解)的机制，实现了指示 Jackson 应该使用其拥有的元注释，而不是使用目标注释，从而实现了自定义脱敏实现设计目标。

## 5.总结

以上就是利用 Jackson 序列化实现数据脱敏的全过程，如有此类需求的同学可以借鉴上面的实现方法。