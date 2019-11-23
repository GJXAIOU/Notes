# spring Boot 配置文件

## 一、配置文件

- SpringBoot 使用一个全局的配置文件，配置文件名是固定的，下面两者选一即可；

  - application.properties（默认创建项目时只有这个）

  - application.yml

- 配置文件的作用：修改SpringBoot自动配置的默认值；

    默认启动是因为SpringBoot在底层都给我们自动配置好；

- YAML（YAML Ain't Markup Language）

  - YAML  A Markup Language：是一个标记语言

  - YAML   isn't Markup Language：不是一个标记语言；

- 标记语言：

​	以前的配置文件，大多都使用的是  **xxxx.xml** 文件；

- YAML：**以数据为中心**，比json、xml等更适合做配置文件；

​	YAML：配置例子

```yaml
server:
  port: 8081
```

​	对应的 XML 配置：

```xml
<server>
	<port>8081</port>
</server>
```



## 二、YAML语法

### （一）基本语法

- `k: v:` 表示一对键值对（空格必须有），冒号后面都得加空格；

- 以**空格**的缩进来控制层级关系，即只要是左对齐的一列数据，都是同一个层级的；

- 属性和值也是大小写敏感；

```yaml
server:
    port: 8081
    path: /hello
```

### （二）值的写法

#### 1.字面量：普通的值（数字，字符串，布尔）

- 使用 `k: v:` 格式直接写字面量即可；

- 字符串默认不用加上单引号或者双引号；

- `""`：双引号会转义特殊字符，特殊字符会作为本身想表示的意思；

​				`name: "zhangsan \n lisi"`   输出：`zhangsan 换行  lisi`

​		`''`：单引号不会转义字符串里面的特殊字符，特殊字符最终只是一个普通的字符串数据；

​				`name: ‘zhangsan \n lisi’`   输出：`zhangsan \n  lisi`



#### 2.对象、Map（属性和值，即键值对）：

​	k: v：在下一行来写对象的属性和值的关系，注意缩进

```yaml
friends:
		lastName: zhangsan
		age: 20
```

行内写法：

```yaml
friends: {lastName: zhangsan,age: 18}
```



#### 3.数组（List、Set）：

用 `- 值`表示数组中的一个元素

```yaml
pets:
 - cat
 - dog
 - pig
```

行内写法

```yaml
pets: [cat,dog,pig]
```



## （三）配置文件值注入

#### 1.两种配置文件方式

yml 配置文件：

```yaml
person:
    lastName: hello
    age: 18
    boss: false
    birth: 2017/12/12
    maps: {k1: v1,k2: 12}
    lists:
      - lisi
      - zhaoliu
    dog:
      name: 小狗
      age: 12
```

上面的 yml  文件对应 properties 文件写法：

```properties
# idea.properties配置文件默认是：utf-8
# 配置 person 的值
person.last-name=张三
person.age=12
person.birth=2017/12/15
person.boss=false
person.maps.k1=v1
person.maps.k2=14
person.lists=a,b,c
person.dog.name=hello
person.dog.age=15

```

#### 2.对应的两种属性注入方式

属性值注入方式一：javaBean：具体的属性值在上面的 yml  配置文件中

```java
// 只有该组件是容器中的组件，才能使用容器提供的@ConfigurationProperties功能，因此要加@Component；
@Component
// @ConfigurationProperties：告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定，最终作用是：将配置文件中配置的每一个属性的值，映射到这个组件中
// prefix = "person"：配置文件中哪个下面的所有属性进行一一映射
@ConfigurationProperties(prefix = "person")
@Getter
@Setter
public class Person {
    private String lastName;
    private Integer age;
    private Boolean boss;
    private Date birth;

    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
}
```

属性值注入方式二：加载 properties 中属性值

取值方式类似于之前 spring 中的配置文件

```java
<bean class="Person">
    <property name="lastName" value="字面量，布尔值等等"></property>
<bean/>
```

可以使用${key}从环境变量、配置文件中获取值，或者 #{SpEL} 使用表达式

```java
package com.gjxaiou.springboot.bean;

@Component
public class Person {


    @Value("${person.last-name}")
    private String lastName;
    @Value("#{11*2}")
    private Integer age;
    @Value("true")
    private Boolean boss;

    private Date birth;
    @Value("${person.maps}")
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
```

附：Dog Bean：

```java
@Getter
@Setter
public class Dog {
    private String name;
    private Integer age;
}
```



附：我们可以导入配置文件处理器（pom.xml），以后编写配置(配置文件进行绑定)就有提示了

```xml
<!--导入配置文件处理器，配置文件进行绑定就会有提示-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-configuration-processor</artifactId>
			<optional>true</optional>
		</dependency>
```

#### 3.测试

测试类的值是否导入：

```java
package com.gjxaiou.springboot;

/**
 * SpringBoot单元测试;
 * 可以在测试期间很方便的类似编码一样进行自动注入等容器的功能
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBoot02ConfigApplicationTests {

	@Autowired
	Person person;

	@Autowired
	ApplicationContext ioc;

	@Test
	public void testHelloService(){
		boolean b = ioc.containsBean("helloService02");
		System.out.println(b);
	}

	@Test
	public void contextLoads() {
		System.out.println(person);
	}
}
```



#### 5.@Value获取值和@ConfigurationProperties获取值比较

|                                | @ConfigurationProperties | @Value         |
| ------------------------------ | ------------------------ | -------------- |
| 功能                           | 批量注入配置文件中的属性 | 需要一个个指定 |
| 松散绑定（松散语法）           | 支持（见下面说明）       | 不支持         |
| SpEL（例如  # {表达式}）       | 不支持                   | 支持           |
| JSR303数据校验                 | 支持                     | 不支持         |
| 复杂类型封装（例如  map 类型） | 支持                     | 不支持         |

配置文件yml还是properties他们都能获取到值；

如果说，我们只是在某个业务逻辑中需要获取一下配置文件中的某项值，使用@Value；

如果说，我们专门编写了一个javaBean来和配置文件进行映射，我们就直接使用@ConfigurationProperties；

**上面名词解释**

- **松散绑定**：就是代码中的变量名 和 yml 文件中的变量名的对应绑定

以 person.firstName 变量为例，对应的 yml 文件中可以为：`person.firstName` `person.first-name` `person.first_name` `PERSON_FIRSTNAME`都可以，其中 `-` 和 `_` 都表示大写。

- **数据校验**：类上面需要增加：`@Validated`注解，同时需要校验的变量名上面加上对应想校验的格式；示例如下：

```java
@Component
@ConfigurationProperties(prefix = "person")
@Validated
public class Person {

    // 表示要校验 lastName 的值是否为邮箱格式
    @Email
    private String lastName;
```



#### 6.@PropertySource&@ImportResource&@Bean

- @**PropertySource**：作用是加载指定的配置文件；

    因为上面的  @ConfigurationProperties(prefix = "person")默认从全局配置文件中获取值；如果这两个命令都写并且默认的配置文件和 person.properties 中都有属性值，则加载全局配置文件中的值；

```java
@PropertySource(value = {"classpath:person.properties"})
@Component
public class Person {
    private String lastName;
    private Integer age;
    private Boolean boss;

```

对应的 person.properties

```properties
person.last-name=李四
person.age=12
person.birth=2017/12/15
person.boss=false
person.maps.k1=v1
person.maps.k2=14
person.lists=a,b,c
person.dog.name=dog
person.dog.age=15
```



- @**ImportResource**：导入 Spring 的配置文件，让配置文件里面的内容生效；

首先可以创建一个 service 类：HelloService.java;

然后创建一个 spring的 xml 配置文件 bean.xml，配置如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="helloService" class="com.gjxaiou.springboot.service.HelloService"></bean>

</beans>
```

Spring Boot 里面没有 Spring 的配置文件，而我们自己编写的配置文件，也不能自动识别，因此想让 Spring 的配置文件生效，加载进来；就使用 `@ImportResource`标注在一个配置类上（这里的注解是标注在主配置类：SpringBootConfigApplication.java 上）

```java
// 导入Spring的配置文件让其生效
@ImportResource(locations = {"classpath:beans.xml"})
```

然后在测试类中：可以验证该类是否在 IOC 容器中

```java
package com.gjxaiou.springboot;

@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBoot02ConfigApplicationTests {

	@Autowired
	ApplicationContext ioc;

	@Test
	public void testHelloService(){
		boolean b = ioc.containsBean("helloService");
		System.out.println(b);
	}
}
```



- **@Bean**

SpringBoot推荐给容器中添加组件的方式：推荐使用全注解的方式

1、首先要写个配置类，同时加上**@Configuration**------>类似于上面的Spring配置文件

2、使用**@Bean**给容器中添加组件

```java
/**
 * @Configuration：指明当前类是一个配置类；就是来替代之前的Spring配置文件
 * 之前在配置文件中用<bean><bean/>标签添加组件，这里使用 @Bean 注解
 *
 */
@Configuration
public class MyAppConfig {

    //作用：将方法的返回值添加到容器中；容器中这个组件默认的id就是方法名
    @Bean
    public HelloService helloService(){
        System.out.println("配置类@Bean给容器中添加组件了...");
        return new HelloService();
    }
}
```

## （四）配置文件占位符

### 1、随机数

```java
${random.value}、${random.int}、${random.long}
${random.int(10)}、${random.int[1024,65536]}

```



### 2、占位符获取之前配置的值，如果没有可以是用:指定默认值

这里的属性值是配置在默认的 application.properties 中，配置在其他之后就是需要在下面的 Java 代码中添加以下位置即可；

```properties
person.last-name=张三${random.uuid}
person.age=${random.int}
person.birth=2017/12/15
person.boss=false
person.maps.k1=v1
person.maps.k2=14
person.lists=a,b,c
# 因为这里没有 person.hello ，因此使用 ${person.hello}，最终结果显示就是：${person.hello}，这里的 : 表示如果没有值，就赋值 hello1 作为默认值
person.dog.name=${person.hello:hello1}_dog
#person.dog.name=${person.last-name}
person.dog.age=15
```

对应的Person 类配置和上面一样：

然后测试类为：

```java
package com.gjxaiou.springboot;

import com.gjxaiou.springboot.bean.Person;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.context.ApplicationContext;
import org.springframework.test.context.junit4.SpringRunner;


/**
 * SpringBoot单元测试;
 * 可以在测试期间很方便的类似编码一样进行自动注入等容器的功能
 */
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBoot02ConfigApplicationTests {

	@Autowired
	Person person;

	@Test
	public void contextLoads() {
		System.out.println(person);
	}
}

```





## （五）Profile，是 spring 用于多环境支持的

就是测试、开发等等的配置文件是不相同的，方便进行切换

### 方式一：多Profile文件

我们在主配置文件编写的时候，文件名可以是   `application-{profile}.properties/yml`

例如配置文件为：`application-dev.properties/application-product.properties/application.properties`

默认使用application.properties的配置；如果想使用使其配置文件，需要在 `application.properties` 中输入：`spring.profiles.active=dev` 就激活使用第一个配置文件；



### 方式二：yml支持多文档块方式

下面都是在 application.yml 中配置，使用 `---` 分割成不同的文档块；最上面是默认的，如果想激活其他的，只要在最上面的 里面添加 :`active`即可

```yml
server:
  port: 8081
spring:
  profiles:
    active: prod

---
server:
  port: 8083
spring:
  profiles: dev


---

server:
  port: 8084
spring:
  profiles: prod  #指定属于哪个环境
```



### 附：激活指定 profile

- （针对方式一）在配置文件中指定  spring.profiles.active=dev（针对 properties 文件）

- 命令行（针对方式一、二）：IDEA 中 Run/Debug Configurations 中添加的 SpringBoot 右边对应的：`program arguments `中输入下面语句`--spring.profiles.active=dev`，如果打包之后运行可以使用：`java -jar spring-boot-02-config-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev`指定

​		可以直接在测试的时候，配置传入命令行参数

- 虚拟机参数（针对方式一、二）：IDEA上面配置中的：`VM options`中配置：`-Dspring.profiles.active=dev`

![image-20191123183557192](FrameDay06_2%20SpringBoot%E9%85%8D%E7%BD%AE.resource/image-20191123183557192.png)

## 六、配置文件加载位置

springboot 启动会按照顺序扫描以下位置的 application.properties 或者 application.yml 文件作为 Spring boot的默认配置文件：file 表示当前项目的文件夹，classpath：表示类路径，就是 resources 目录下

```java
–file:./config/

–file:./

–classpath:/config/

–classpath:/
```

**优先级由高到底，高优先级的配置会覆盖低优先级的配置**；

**SpringBoot会从这四个位置全部加载主配置文件；互补配置**；



- 我们还可以通过spring.config.location来改变默认的配置文件位置

    **项目打包好以后，我们可以使用命令行参数的形式，启动项目的时候来指定配置文件的新位置；指定配置文件和默认加载的这些配置文件共同起作用形成互补配置；**

​    命令为：`java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --spring.config.location=D:/application.properties`

## 七、外部配置加载顺序

**==SpringBoot也可以从以下位置加载配置； 优先级从高到低；高优先级的配置覆盖低优先级的配置，所有的配置会形成互补配置==**

- **1.命令行参数**

因为项目在打包的时候，只会将 main 包下面的 java  和 resources 中的内容进行打包，因此像上面直接在项目根目录下面配置的配置文件是不会被打包的；

所有的配置都可以在命令行上进行指定(也是互补方式)

`java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --server.port=8087  --server.context-path=/abc`

多个配置用空格分开； --配置项=值，一般适用于小部分的值



- 2.来自java:comp/env的JNDI属性

- 3.Java系统属性（System.getProperties()）

- 4.操作系统环境变量

- 5.RandomValuePropertySource配置的random.*属性值



==**由jar包外向jar包内进行寻找；**==

==**优先加载带profile**==

- **6.jar包外部设置的的application-{profile}.properties或application.yml(带spring.profile)配置文件**（相当于已经将项目打包之后，然后新建一个配置文件放在 jar 包同文件夹下面）

- **7.jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置文件**



==**再来加载不带profile**==

- **8.jar包外部的application.properties或application.yml(不带spring.profile)配置文件**

- **9.jar包内部的application.properties或application.yml(不带spring.profile)配置文件**



- 10.@Configuration注解类上的@PropertySource

- 11.通过SpringApplication.setDefaultProperties指定的默认属性

所有支持的配置加载来源[参考官方文档](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#boot-features-external-config)

## 八、自动配置原理

配置文件到底能写什么？怎么写？自动配置原理；

[配置文件能配置的属性参照](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#common-application-properties)



### （一）**自动配置原理：**

- SpringBoot启动的时候加载主配置类，然后加载主配置类上面的 @SpringBootApplication ，该注解（点进去），最主要作用就是开启了自动配置功能 @EnableAutoConfiguration；

-  **@EnableAutoConfiguration 作用：**（点进去）
     - 利用 @Import 中的 `EnableAutoConfigurationImportSelector` 选择器给容器中导入一些组件；
    - 点进去然后查看其父类 AutoConfigurationImportSelector 中的selectImports()方法的内容，就知道导入哪些内容了；
    - 上面那个方法最终返回 List<String> configurations = getCandidateConfigurations(annotationMetadata,   attributes);获取候选的配置，getCandidateConfigurations 中最重要的是：SpringFactoriesLoader.loadFactoryNames() 方法，该方法作用（点进去）是从类路径下面得到资源，得到的资源是：扫描所有jar包类路径下  META-INF/spring.factories 文件（LoadFactoryNames()方法往下看）然后扫描文件的 URL 把扫描到的这些文件的内容包装成properties对象，从properties中获取到EnableAutoConfiguration.class类（类名）对应的值，然后把他们添加在容器中。

**==因此其作用是：将类路径下  META-INF/spring.factories 里面配置的所有EnableAutoConfiguration的值加入到了容器中；==**容器中会有以下类：

```properties
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
...
org.springframework.boot.autoconfigure.websocket.WebSocketAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.WebSocketMessagingAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration
```

每一个这样的  `xxxAutoConfiguration`类都是容器中的一个组件，都加入到容器中；用他们来做自动配置；

- 每一个自动配置类进行自动配置功能

以**HttpEncodingAutoConfiguration（Http编码自动配置）**为例解释自动配置原理；下面就是该类的具体内容

```java
// 表示这是一个配置类，和以前编写的配置文件一样，也可以给容器中添加组件
@Configuration   
//启动指定类（括号中的类，点击这个类就发现该类上面标注了 @configurationProperties 注解，类的内容见下一个代码块）的ConfigurationProperties功能；将配置文件中对应的值和HttpEncodingProperties绑定起来；并把HttpEncodingProperties加入到ioc容器中
@EnableConfigurationProperties(HttpEncodingProperties.class)  
// 底层是：Spring的@Conditional注解（Spring注解版），作用是：根据不同的条件，如果满足指定的条件，整个配置类里面的配置才会生效；   这里作用是：判断当前应用是否是web应用，如果是，当前配置类生效
@ConditionalOnWebApplication 
// 判断当前项目有没有这个类（CharacterEncodingFilter）；该类是SpringMVC中进行乱码解决的过滤器；
@ConditionalOnClass(CharacterEncodingFilter.class)  
// 判断配置文件中是否存在某个配置，这里就是：spring.http.encoding.enabled；后面的 matchIfMissing 表示如果不存在，判断也是成立的，即即使我们配置文件中不配置pring.http.encoding.enabled=true，也是默认生效的；
@ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true)  
public class HttpEncodingAutoConfiguration {
  
  	//他已经和SpringBoot的配置文件映射了，它是获取配置文件中的值的
  	private final HttpEncodingProperties properties;
  
   //只有一个有参构造器的情况下，参数的值就会从容器中拿，使用上面的 @EnableConfigurationProperties 注解将括号中的类加入 IOC 容器中
  	public HttpEncodingAutoConfiguration(HttpEncodingProperties properties) {
		this.properties = properties;
	}
  
    @Bean   // @Bean 表示给容器中添加一个组件（这里对应的组件就是CharacterEncodingFilter），这个组件的某些值需要从properties中获取
	@ConditionalOnMissingBean(CharacterEncodingFilter.class) //判断容器没有这个组件？
	public CharacterEncodingFilter characterEncodingFilter() {
		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
		filter.setEncoding(this.properties.getCharset().name());
		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
		return filter;
	}
```

上面整个配置类作用就是：根据当前不同的条件判断，决定这个配置类是否生效

一但这个配置类生效；这个配置类就会给容器中添加各种组件；这些组件的属性是从对应的properties类中获取的，这些类里面的每一个属性又是和配置文件绑定的；



- 所有在配置文件中能配置的属性都是在xxxxProperties类中封装着‘；例如配置文件（spring.http.encoding）能配置什么就可以参照某个功能对应的这个属性类（HttpEncodingProperties）

```java
@ConfigurationProperties(prefix = "spring.http.encoding")  //从配置文件中获取指定的值和bean的属性进行绑定
public class HttpEncodingProperties {

   public static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");
```





**精髓：**

​	**1）、SpringBoot启动会加载大量的自动配置类**

​	**2）、我们看我们需要的功能有没有SpringBoot默认写好的自动配置类；**

​	**3）、如果有，我们再来看这个自动配置类中到底配置了哪些组件；（只要我们要用的组件有，我们就不需要再来配置了）**

​	**4）、给容器中自动配置类添加组件的时候，会从properties类中获取某些属性。我们就可以在配置文件中指定这些属性的值；**



xxxxAutoConfigurartion：自动配置类；作用是给容器中添加组件，同时也会有对应的xxxxProperties:封装配置文件中相关属性；

**示例解释**

使用 Ctrl + n 查找：`*AutoConfiguration`，这里以 CacheAutoconfiguration 为例，有相应的属性自动配置类：``CacheAutoconfiguration.java`对应就有相应的属性值的获取：见上面的 @EnableConfigurationProperties 中的括号中类，即 CacheProperties.class，点击之后就可以看出来可以配置哪些属性，属性前缀就是上面的注解@ConfigurationProperties中的内容，后面更上该类的变量名：例如可以在 properties 文件中配置：`spring.cache.type`

### (二)细节

#### 1、@Conditional派生注解（Spring注解版原生的@Conditional作用）

作用：必须是@Conditional指定的条件成立，才给容器中添加组件，配置配里面的所有内容才生效；

还是以上面Http 代码为例

```java
@Configuration   
@EnableConfigurationProperties(HttpEncodingProperties.class) 

// 这里就是三个 Condition 注解判断，只有他们都成立了配置才生效；
// @ConditionOnXX 本质上（点击）就是使用 Spring的 @condition 注解，@Condition(指定的条件判断类，点击该类，发现该类有一个match 方法，通过一系列的判断，最终返回 true/false)
@ConditionalOnWebApplication 
@ConditionalOnClass(CharacterEncodingFilter.class) 
@ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true) 
public class HttpEncodingAutoConfiguration {
  
 
  	private final HttpEncodingProperties properties;
  
 
  	public HttpEncodingAutoConfiguration(HttpEncodingProperties properties) {
		this.properties = properties;
	}
  
    // 同样这里注入的时候，进行了Condition的判断
    @Bean   
	@ConditionalOnMissingBean(CharacterEncodingFilter.class) //判断容器没有这个组件？
	public CharacterEncodingFilter characterEncodingFilter() {
		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
		filter.setEncoding(this.properties.getCharset().name());
		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
		return filter;
	}
```

spring boot 将 @condition 注解扩展了

| @Conditional扩展注解            | 作用（判断是否满足当前指定条件）                 |
| ------------------------------- | ------------------------------------------------ |
| @ConditionalOnJava              | 系统的java版本是否符合要求                       |
| @ConditionalOnBean              | 容器中存在指定Bean；                             |
| @ConditionalOnMissingBean       | 容器中不存在指定Bean；                           |
| @ConditionalOnExpression        | 满足SpEL表达式指定                               |
| @ConditionalOnClass             | 系统中有指定的类                                 |
| @ConditionalOnMissingClass      | 系统中没有指定的类                               |
| @ConditionalOnSingleCandidate   | 容器中只有一个指定的Bean，或者这个Bean是首选Bean |
| @ConditionalOnProperty          | 系统中指定的属性是否有指定的值                   |
| @ConditionalOnResource          | 类路径下是否存在指定资源文件                     |
| @ConditionalOnWebApplication    | 当前是web环境                                    |
| @ConditionalOnNotWebApplication | 当前不是web环境                                  |
| @ConditionalOnJndi              | JNDI存在指定项                                   |

**自动配置类必须在一定的条件下才能生效；**，条件就是上面的一系列 @condition 判断

我们怎么知道哪些自动配置类生效；所有的自动配置类都在 jar 包：spring-boot-autoconfigure 下的 spring.factories，点开每一个配置类都可以看出该控制类有没有生效（因为每个控制类都需要一定的condition，如果里面有冒红就是不生效），但是挨个点击判断太过于麻烦，

**==我们可以在配置文件 application.properties 通过启用  debug=true属性，开启 spring boot 的 debug 模式；来让控制台打印自动配置报告==**，这样我们就可以很方便的知道哪些自动配置类生效；

```java
=========================
AUTO-CONFIGURATION REPORT
=========================


Positive matches:（自动配置类启用的）
-----------------

   DispatcherServletAutoConfiguration matched:
      - @ConditionalOnClass found required class 'org.springframework.web.servlet.DispatcherServlet'; @ConditionalOnMissingClass did not find unwanted class (OnClassCondition)
      - @ConditionalOnWebApplication (required) found StandardServletEnvironment (OnWebApplicationCondition)
        
    
Negative matches:（没有启动，没有匹配成功的自动配置类）
-----------------

   ActiveMQAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required classes 'javax.jms.ConnectionFactory', 'org.apache.activemq.ActiveMQConnectionFactory' (OnClassCondition)

   AopAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required classes 'org.aspectj.lang.annotation.Aspect', 'org.aspectj.lang.reflect.Advice' (OnClassCondition)
        ..................................
```

