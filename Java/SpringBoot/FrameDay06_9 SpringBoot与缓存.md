## JSR-107

Java Caching 定义了5个核心接口，分别是 CachingProvider, CacheManager, Cache, Entry 和 Expiry。

- CachingProvider 定义了创建、配置、获取、**管理和控制多个 CacheManager**。一个应用可以在运行期访问多个 CachingProvider。通过该 Provider 来管理和控制缓存控制器：CacheManager
- CacheManager 定义了创建、配置、获取、**管理和控制多个唯一命名的 Cache**，这些 Cache 存在于CacheManager 的上下文中。一个 CacheManager 仅被一个 CachingProvider 所拥有。
- Cache 是一个类似 Map 的数据结构并临时存储以 Key 为索引的值。一个 Cache 仅被一个 CacheManager 所拥有。
- Entry 是一个存储在 Cache 中的 key-value 对。
- Expiry 每一个存储在Cache中的条目有一个定义的有效期。一旦超过这个时间，条目为过期的状态。一旦过期，条目将不可访问、更新和删除。缓存有效期可以通过ExpiryPolicy设置。



![image-20201113134229077](FrameDay06_9%20SpringBoot%E4%B8%8E%E7%BC%93%E5%AD%98.resource/image-20201113134229077.png)

如果要使用 JSR 107 需要导入以下包

```xml
<dependency>
    <groupId>javax.cache</groupId>
    <artifactId>cache-api</artifactId>
</dependency>
```

分析该 jar 包：

- `javax.cache.spi.CachingProvider.class` 接口里面主要是：`CacheManager getCacheManager(URI var1, ClassLoader var2, Properties var3);` 该方法主要用户获取  CacheManager 。

- `javax.cache.CacheManager.class`接口主要是创建或者获取一个缓存组件。

    ```java
    <K, V, C extends Configuration<K, V>> Cache<K, V> createCache(String var1, C var2) throws IllegalArgumentException;
    
    <K, V> Cache<K, V> getCache(String var1, Class<K> var2, Class<V> var3);
    
    <K, V> Cache<K, V> getCache(String var1);
    ```

## Spring 缓存抽象

- Spring 从 3.1 开始定义了 `org.springframework.cache.Cache`和 `org.springframework.cache.CacheManager` 接口来统一不同的缓存技术；
- 并支持使用 `JCache（JSR-107）`注解简化我们开发；（当然 Spring 也提供了自身的操作缓存的组件）
    •Cache接口为缓存的组件规范定义，包含缓存的各种操作集合；
    •Cache接口下Spring提供了各种xxxCache的实现；如RedisCache，EhCacheCache , ConcurrentMapCache等；
    •每次调用需要缓存功能的方法时，Spring会检查检查指定参数的指定的目标方法是否已经被调用过；如果有就直接从缓存中获取方法调用后的结果，如果没有就调用方法并缓存结果后返回给用户。下次调用直接从缓存中获取。
    •使用Spring缓存抽象时我们需要关注以下两点；
    1、确定方法需要被缓存以及他们的缓存策略
    2、从缓存中读取之前缓存存储的数据



#### 几个重要概念&缓存注解

| 注解和接口名称 | 作用    |
| -------------- | ------------ |
| Cache          | 缓存接口，定义缓存操作。实现有：RedisCache、EhCacheCache、ConcurrentMapCache等 |
| CacheManager   | 缓存管理器，管理各种缓存（Cache）组件     |
| @Cacheable     | 主要针对方法配置，能够根据方法的请求参数对其结果进行缓存（就是表示该方法返回结果是缓存的）「该方法一旦有数据就直接从缓存中拿了，不再调用」 |
| @CacheEvict    | 清空缓存          |
| @CachePut      | 保证方法被调用，又希望结果被缓存。「该方法每次都会被调用，调用完成之后结果放入缓存，会刷新缓存中的值从而保证缓存值的更新」 |
| @EnableCaching | 开启基于注解的缓存            |
| keyGenerator   | 缓存数据时key生成策略          |
| serialize      | 缓存数据时value序列化策略    |







@Cacheable/@CachePut/@CacheEvict 主要的参数

|参数|含义|示例|
|--- |--- |--- |
|value |缓存的名称，在spring 配置文件中定义，必须指定至少一个 | 例如：@Cacheable(value=”mycache”)  或者 @Cacheable(value={”cache1”,”cache2”}|
|key | 缓存的key，可以为空，如果指定要按照SpEL 表达式编写，如果不指定，则缺省按照方 法的所有参数进行组合 | 例如： @Cacheable(value=”testcache”,key=”#userName”) |
|condition | 缓存的条件，可以为空，使用SpEL 编写，返回true 或者false，只有为true 才进行缓存/清除缓存，在调用方法之前之后都能判断 | 例如：@Cacheable(value=”testcache”, condition=”#userName.length()>2”)|
|allEntries(@CacheEvict )|是否清空所有缓存内容，缺省为false，如果指定为true，则方法调用后将立即清空所有缓存|例如：@CachEvict(value=”testcache”,allEntries=true)|
|beforeInvocation(@CacheEvict)|是否在方法执行前就清空，缺省为false，如果指定为true，则在方法还没有执行的时候就清空缓存，缺省情况下，如果方法执行抛出异常，则不会清空缓存|例如：@CachEvict(value=”testcache”，beforeInvocation=true)|
|unless(@CachePut)(@Cacheable)|用于否决缓存的，不像condition，该表达式只在方法执行之后判断，此时可以拿到返回值result进行判断。条件为true不会缓存，fasle才缓存|例如：@Cacheable(value=”testcache”,unless=”#result == null”)|



## 整合Redis

步骤如下：

### 一、搭建基本环境

- 导入数据库文件 创建出 department 和employee表（库名为：`spring_cache`）

- 创建 javaBean 封装数据库中数据

- 整合 MyBatis 操作数据库
    - 配置数据源信息：在 application.properties 中配置

    - 使用注解版的 MyBatis；

        即在主程序类上使用： `@MapperScan` 指定需要扫描的 mapper 接口所在的包

###  二、快速体验缓存

- 开启基于注解的缓存

    即在主类上标注： `@EnableCaching`

- 给对应的方法标注缓存注解即可

​    包括：   `@Cacheable`、`@CacheEvict`、`@CachePut`

**`@Cacheable` 详解**

@Cacheable 作用将方法的运行结果进行缓存；以后再要相同的数据，直接从缓存中获取，不用调用方法；

几大主要属性：

- `cacheNames/value`：指定缓存组件的名字;将方法的返回结果放在哪个缓存中，是数组的方式，可以指定多个缓存；

    因为原本是 CacheManager 管理多个 Cache 组件的，对缓存的真正 CRUD 操作在 Cache 组件中，每一个缓存组件有自己唯一一个名字；

![image-20201113161328301](FrameDay06_9%20SpringBoot%E4%B8%8E%E7%BC%93%E5%AD%98.resource/image-20201113161328301.png)

- key：缓存数据使用的key；可以用它来指定 key 的值。默认是使用方法参数的值

    例如方法参数传递一个 `1`，则最终结构为：`1-方法的返回值`
    同样可以公国编写 SpEL 来指定；可以用的 SpEL 如下所示。示例如下：

    ```java
    // 指定缓存的名称为：tmp，同时 #id 表示获取参数 id 的值作为 key
    @Cacheable(cacheNames = "tmp", key = "#id")
    public Employee getEmp(Integer id){ 
     		// XXXXX
     }
    ```

    |名字|位置|描述|示例|
| --- | --- | --- | --- |
    |methodName|root object|当前被调用的方法名；例如设定 key 格式为：`getEmp[值]` 写法为：`@Cacheable(key="#root.methodName+'[' + #id + ']'")`|#root.methodName|
    |method|root object|当前被调用的方法|#root.method.name|
    |target|root object|当前被调用的目标对象|#root.target|
    |targetClass|root object|当前被调用的目标对象类|#root.targetClass|
    |args|root object|当前被调用的方法的参数列表|#root.args[0]|
    |caches|root object|当前方法调用使用的缓存列表（如@Cacheable(value={"cache1", "cache2"})），则有两个cache|#root.caches[0].name|
    |argument name|evaluation context|方法参数的名字. 可以直接#参数名，也可以使用#p0或#a0 的形式，0代表参数的索引；|#iban、#a0 、#p0|
    |result|evaluation context|方法执行后的返回值（仅当方法执行之后的判断有效，如‘unless’，’cache put’的表达式’cache evict’的表达式beforeInvocation=false）|#result|
    
- keyGenerator：key 的生成器；也可以自己指定 key 的生成器的组件id，自己写一个 KeyGenerator，然后在 `@Cacheable(keyGenerator = "myKeyGenerator")`指定即可。
         注意： key/keyGenerator：二选一使用;
         
     ~~~java
     package com.atguigu.cache.config;
     
     import org.springframework.cache.interceptor.KeyGenerator;
     import org.springframework.context.annotation.Bean;
     import org.springframework.context.annotation.Configuration;
     
     import java.lang.reflect.Method;
     import java.util.Arrays;
     
     @Configuration
     public class MyCacheConfig {
         @Bean("myKeyGenerator")
         public KeyGenerator keyGenerator() {
             return new KeyGenerator() {
                 @Override
                 public Object generate(Object target, Method method, Object... params) {
                     return method.getName() + "[" + Arrays.asList(params).toString() + "]";
                 }
             };
         }
     }
     
     // Lambda 写法
     @Configuration
     public class MyCacheConfig {
         @Bean("myKeyGenerator")
         public KeyGenerator keyGenerator() {
             return (target, method, params) -> method.getName() + "[" + Arrays.asList(params).toString() + "]";
         }
     }
     ```
~~~
     
-  cacheManager：指定对应的缓存管理器；
- cacheResolver：指定获取解析器，和上面  cacheManager 功能一下，两者二选一

-  condition：指定符合条件的情况下才缓存；同样支持 SpEL 表达式

    ```java
    // 示例如下：
    @Cacheable(condition = "#id>0")
    @Cacheable(condition = "#a0>1")：第一个参数的值》1的时候才进行缓存
    ```

-  unless：用于否定缓存；当 unless 指定的条件为 true，方法的返回值就不会被缓存；注意：Unless 可以获取到结果进行判断，同样支持 SpEL 表达式

    ```java
    unless = "#result == null"
    unless = "#a0==2":如果第一个参数的值是2，结果不缓存；
    ```

- sync：是否使用异步模式 ，异步时候不支持 `unless` 属性。异步就是方法返回结果和将结果写入缓存中。

### 原理

  如果引入缓存，就会包括一个对应的自动配置类；CacheAutoConfiguration

```java
@Import({CacheAutoConfiguration.CacheConfigurationImportSelector.class})
public class CacheAutoConfiguration {
    // XXXXXX
}
```

其中注解中的 `CacheConfigurationImportSelector` 类结构为：

```java
static class CacheConfigurationImportSelector implements ImportSelector {
    CacheConfigurationImportSelector() {
    }

    // 获取所有的缓存配置类
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        CacheType[] types = CacheType.values();
        String[] imports = new String[types.length];

        for(int i = 0; i < types.length; ++i) {
            imports[i] = CacheConfigurations.getConfigurationClass(types[i]);
        }

        return imports;
    }
}
```

  缓存的配置类包括以下这些，在上面的 `return imports`处打断点执行可以获得。

```java
	org.springframework.boot.autoconfigure.cache.GenericCacheConfiguration
    org.springframework.boot.autoconfigure.cache.JCacheCacheConfiguration
    org.springframework.boot.autoconfigure.cache.EhCacheCacheConfiguration
    org.springframework.boot.autoconfigure.cache.HazelcastCacheConfiguration
    org.springframework.boot.autoconfigure.cache.InfinispanCacheConfiguration
    org.springframework.boot.autoconfigure.cache.CouchbaseCacheConfiguration
    org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration
    org.springframework.boot.autoconfigure.cache.CaffeineCacheConfiguration
    org.springframework.boot.autoconfigure.cache.GuavaCacheConfiguration
    org.springframework.boot.autoconfigure.cache.SimpleCacheConfiguration【默认】
    org.springframework.boot.autoconfigure.cache.NoOpCacheConfiguration
```

 其中每个类是否生效可以看类上面注解的规定，例如 `RedisCacheConfiguration`

```java
@Configuration
@AutoConfigureAfter({RedisAutoConfiguration.class})
// 容器中有组件 RedisTemplate 
@ConditionalOnBean({RedisTemplate.class})
// 容器中没有组件 CacheManager
@ConditionalOnMissingBean({CacheManager.class})
// 并且满足 CacheCondition 条件
@Conditional({CacheCondition.class})
class RedisCacheConfiguration {
    // XXX
}  
```

  如果想看当前项目中哪个缓存配置类匹配生效了，可以在 `application.properties` 中增加 `debug = true` 然后启动项目，在控制台中可以看到

```java
// 因为已经配置 Redis 了，所以匹配了这个，默认应该是：SimpleCacheConfiguration
RedisCacheConfiguration matched:
      - Cache org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration automatic cache type (CacheCondition)
      - @ConditionalOnBean (types: org.springframework.data.redis.core.RedisTemplate; SearchStrategy: all) found beans 'stringRedisTemplate', 'redisTemplate'; @ConditionalOnMissingBean (types: org.springframework.cache.CacheManager; SearchStrategy: all) did not find any beans (OnBeanCondition)
```

**`SimpleCacheConfiguration`** 分析

作用就是给容器中注册了一个缓存管理器CacheManager：ConcurrentMapCacheManager，该缓存管理器实现了 `CacheManager` 接口，该接口源码见下：

```java
public interface CacheManager {
    Cache getCache(String var1);

    Collection<String> getCacheNames();
}
```

所以 ConcurrentMapCacheManager 类中对应实现为：

```java
public Cache getCache(String name) {
    // cacheMap 类型为 ConcurrentHashMap，存放的 key-value 为：缓存名称： 缓存组件对象
    Cache cache = (Cache)this.cacheMap.get(name);
    if (cache == null && this.dynamic) {
        synchronized(this.cacheMap) {
            cache = (Cache)this.cacheMap.get(name);
            if (cache == null) {
                // 如果为空则创建一个该类型的缓存组件
                cache = this.createConcurrentMapCache(name);
                this.cacheMap.put(name, cache);
            }
        }
    }
    return cache;
}
```

所以 `getCache()` 方法为：可以获取和创建ConcurrentMapCache类型的缓存组件；他的作用将数据保存在ConcurrentMap 中；

####     标注 `@Cacheable` 的方法运行流程：

- 步骤一：在方法运行之前，先去调用上面的 `getCache()` 方法查询 Cache（缓存组件），即按照方法上面标注的 cacheNames 指定的名字获取；（CacheManager 先获取相应的缓存），第一次获取缓存如果没有 Cache 组件会自动创建。

- 步骤二：通过 ConcurrentMapCache 类中的 `lookup(Object key)` 方法去 Cache 中查找缓存的内容，使用一个 key，如果没有指定则默认就是方法的参数；
    这里的 key 是按照某种策略生成的（见 `CacheAspectSupport` 类中的 `findCachedItem()` 方法里面的 `Object key = generateKey(context, result);`）；

    默认 `generateKey()` 方法是调用使用 `keyGenerator.generate()` 生成的，默认使用 SimpleKeyGenerator 生成 key；
    SimpleKeyGenerator生成key的默认策略；

    - 如果没有参数；`key = new SimpleKey()；`
    - 如果有一个参数：`key = 参数的值`
    - 如果有多个参数：`key = new SimpleKey(params)；`

- 步骤三：没有查到缓存就调用目标方法；
- 步骤四：将目标方法返回的结果，放进缓存中

**总结**：@Cacheable 标注的方法执行之前先来检查缓存中有没有这个数据，默认按照参数的值作为 key 去查询缓存，如果没有就运行方法并将结果放入缓存；以后再来调用就可以直接使用缓存中的数据；     

**核心**：

- 使用 CacheManager【默认是 ConcurrentMapCacheManager】按照名字得到 Cache【默认是 ConcurrentMapCache】组件
- key 使用 keyGenerator 生成的，默认是 SimpleKeyGenerator



`@CachePut(/*value = "emp",*/key = "#result.id")`

既调用方法，又更新缓存数据，适用于修改了数据库某个数据，同时刷新缓存。

运行时机：首先调用目标方法，然后将目标方法的结果缓存起来。【注意 key 要指定一样】



`@CacheEvict()`：缓存清楚，就是删除数据之后同时将缓存中数据也删除。

可以通过 `key` 来指定要删除的 key。

可以通过指定 `allEntries=true`删除所有缓存。

默认清除缓存是在方法执行之后执行，如果想让清除缓存在方法执行之前执行可以通过 `beforeInvocation=true`来指定（即无论方法执行是否成功缓存都清除）。



`@Caching()`：包括上面三个注解的所有规则，可以用于指定复杂的操作。

```java
// @Caching 定义复杂的缓存规则
// 根据名称来查询，同时按照返回的查询结果的 id 和 email 为 key 将返回结果放入缓存中。因为有 @CachePut 注解，所以整个方法肯定执行
@Caching(
    cacheable = {
        @Cacheable(value="emp",key = "#lastName")
    },
    put = {
        @CachePut(value="emp",key = "#result.id"),
        @CachePut(value="emp",key = "#result.email")
    }
)
public Employee getEmpByLastName(String lastName){
    return employeeMapper.getEmpByLastName(lastName);
}
```



`@CacheConfig()` 配置在类上，可以统一指定该类的所有方法遵循某些缓存规定（抽取缓存公共注解），无需逐个方法指定。





 默认使用的是 `ConcurrentMapCacheManager==>ConcurrentMapCache`；将数据保存在 `ConcurrentMap<Object, Object>`中

 开发中使用缓存中间件；redis、memcached、ehcache；

###  三、整合redis作为缓存

 Redis 是一个开源（BSD许可）的，内存中的数据结构存储系统，它可以用作数据库、缓存和消息中间件。

   1、安装redis：使用docker；

   2、引入redis的starter

   3、配置redis

   4、测试缓存

​    原理：CacheManager===Cache 缓存组件来实际给缓存中存取数据

  1）、引入redis的starter，容器中保存的是 RedisCacheManager；即导入了 CacheManager，所以其它配置类就不再匹配了。

  2）、RedisCacheManager 帮我们创建 RedisCache 来作为缓存组件（见该类中的 `createCache()` 方法）；RedisCache通过操作redis缓存数据的

  3）、默认保存数据 k-v 都是Object；利用序列化保存；如何保存为json

​      1、引入了 redis 的 starter，所以 cacheManager 变为 RedisCacheManager；

​      2、默认创建的 RedisCacheManager 操作 redis 的时候使用的是 `RedisTemplate<Object, Object>`

​      3、RedisTemplate<Object, Object> 是 默认使用 jdk 的序列化机制

   4）、自定义CacheManager；

​	即使用 RedisCacheManager 但是不再使用默认的 RedisTemplate