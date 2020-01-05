# Spring Boot 与日志

## 一、日志框架

### （一）常见日志框架

JUL、JCL、Jboss-logging、logback、log4j、log4j2、slf4j....



### （二）日志分类

- 日志门面（日志的抽象层）
  - ~~JCL(Jakarta  Commons Logging)~~
  -  SLF4j（Simple  Logging Facade for Java）
  -  ~~jboss-logging~~
- 日志的实现
  - Log4j 
  - JUL（java.util.logging）  
  - Log4j2  
  - **Logback**

一般选一个门面（抽象层）同时选一个实现；

SpringBoot：底层是 Spring 框架，**Spring 框架默认是用 JCL**；**==SpringBoot 选用 SLF4j 和 logback；==**



## 二、SLF4j使用

### （一）如何在系统中使用SLF4j 

开发的时候，日志记录方法的调用，不应该来直接调用日志的实现类（logback），而是**调用日志抽象层**(slf4j)里面的方法；

- **首先给系统里面导入 slf4j 的 jar 和  logback 的实现 jar**
- 然后具体需要日志的类中实现即可；

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
```



**日志框架搭配方法**

![images/concrete-bindings.png](FrameDay06_3%20SpringBoot%E4%B8%8E%E6%97%A5%E5%BF%97.resource/concrete-bindings.png)

每一个日志的实现框架（如以前应用的 log4j）都有自己的配置文件。使用 slf4j 以后，**配置文件还是做成日志实现框架自己本身的配置文件；**相当于用谁实现就写谁的配置文件，slf4j 只是提供一个接口层；

### （二）统一日志记录

如果系统实现（slf4j + logback）: 但是同时该系统依赖 Spring（commons-logging）、Hibernate（jboss-logging）、MyBatis、xxxx等框架，但是框架本身底层就有日志记录还各不相同；

目标：统一日志记录，即别的框架和我一起统一使用 slf4j 进行输出；

**如何让系统中所有的日志都统一到slf4j；**

- 将系统中其他日志框架先排除出去；

- 用中间包来替换原有的日志框架；

- 导入slf4j 其他的具体实现；

<img src="FrameDay06_3%20SpringBoot%E4%B8%8E%E6%97%A5%E5%BF%97.resource/legacy.png" alt="legacy"  />

## （三）SpringBoot日志关系

**首先导入 Maven 依赖**：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
```

SpringBoot 使用它来做日志功能；下面这个依赖中封装了所有的日志启动场景

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
</dependency>
```

spring boot 底层依赖关系（在项目（这里是 web 项目）的 pom.xml 配置文件上生成）

![](FrameDay06_3%20SpringBoot%E4%B8%8E%E6%97%A5%E5%BF%97.resource/%25E6%2590%259C%25E7%258B%2597%25E6%2588%25AA%25E5%259B%25BE20180131220946.png)

- 总结：
  - SpringBoot 底层也是使用 slf4j + logback 的方式进行日志记录
  - SpringBoot 也把其他的日志都替换成了slf4j（因此导入了 jul-to-slf4j 等 Jar包）；

- 以中间替换包 jcl-over-slf4j.jar 为例：

  首先点击 jar 包发现里面的包名仍然还是：org.apache.commons.logging，同时里面的类也是 LogFactory，下面就是该类的部分截图，可以看出该类里面的实现是调用了 SLF4JLogFactory()

```java
@SuppressWarnings("rawtypes")
public abstract class LogFactory {

    static String UNSUPPORTED_OPERATION_IN_JCL_OVER_SLF4J = "http://www.slf4j.org/codes.html#unsupported_operation_in_jcl_over_slf4j";

    static LogFactory logFactory = new SLF4JLogFactory();
```

![](FrameDay06_3%20SpringBoot%E4%B8%8E%E6%97%A5%E5%BF%97.resource/%25E6%2590%259C%25E7%258B%2597%25E6%2588%25AA%25E5%259B%25BE20180131221411.png)



- 因此如果我们要引入其他框架，一定要把这个框架的默认日志依赖移除掉

例如Spring框架用的是commons-logging；可以在上面的图中点击:spring-boot-starter-logging，就可以跳到 pom.xml 文件中，可以看到下面的配置，确实排除了 commons-logging

```xml
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
			<exclusions>
				<exclusion>
					<groupId>commons-logging</groupId>
					<artifactId>commons-logging</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
```

**==SpringBoot能自动适配所有的日志，而且底层使用slf4j+logback的方式记录日志，引入其他框架的时候，只需要把这个框架依赖的日志框架排除掉即可；==**

## 四、日志使用

### （一）默认配置

SpringBoot 默认帮我们配置好了日志，因此空项目直接运行下面窗口中会有一系列的日志输出；

同时如果自己想配置日志输入示例如下：这里是在 测试类中配置

```java
	//日志记录器：logger
	Logger logger = LoggerFactory.getLogger(getClass());
	@Test
	public void contextLoads() {

		//日志的级别；
		//由低到高   trace<debug<info<warn<error
		//可以调整输出的日志级别；日志就只会在这个级别以以后的高级别生效
		logger.trace("这是trace日志...");
		logger.debug("这是debug日志...");
		//SpringBoot默认给我们使用的是info级别的，没有指定级别包/类的就用SpringBoot默认规定的级别；
		logger.info("这是info日志...");
		logger.warn("这是warn日志...");
		logger.error("这是error日志...");


	}
```

日志的输出格式含义：

    %d 表示日期时间，
    %thread 表示线程名，
    %-5level： 级别从左显示5个字符宽度
    %logger{50}  表示logger名字最长50个字符，否则按照句点分割
    %msg： 日志消息
    %n 是换行符
    
    示例：  %d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n

SpringBoot修改日志的默认配置(在 application.properties 配置文件中)

```properties
# 日志的输出级别可以控制到包/类
logging.level.com.gjxaiou=trace

#logging.path=
# 当 logging.path 没有值的时候，即不指定路径只是配置 logging.file=日志文件名，就会在当前项目下生成该日志文件名的日志；当然可以指定完整的日志文件输出路径，就会在指定路径下面输出日志文件；
#logging.file=G:/springboot.log

# 只指定 path 不指定 file 的情况下：使用下面配置就是在当前磁盘的根路径下创建spring文件夹和里面的log文件夹；使用 spring.log 作为默认文件
logging.path=/spring/log

#  在控制台输出的日志的格式
logging.pattern.console=%d{yyyy-MM-dd} [%thread] %-5level %logger{50} - %msg%n
# 指定文件中日志输出的格式
logging.pattern.file=%d{yyyy-MM-dd} === [%thread] === %-5level === %logger{50} ==== %msg%n
```

| logging.file | logging.path | Example  | Description                        |
| ------------ | ------------ | -------- | ---------------------------------- |
| (none)       | (none)       |          | 只在控制台输出                     |
| 指定文件名   | (none)       | my.log   | 输出日志到my.log文件               |
| (none)       | 指定目录     | /var/log | 输出到指定目录的 spring.log 文件中 |

### （二）指定配置

给类路径下放上每个日志框架自己的配置文件即可，这样SpringBoot就不使用他默认配置的了，文件名如下：

| Logging System          | Customization                                                |
| ----------------------- | ------------------------------------------------------------ |
| Logback                 | `logback-spring.xml`, `logback-spring.groovy`, `logback.xml` or `logback.groovy` |
| Log4j2                  | `log4j2-spring.xml` or `log4j2.xml`                          |
| JDK (Java Util Logging) | `logging.properties`                                         |

logback.xml：直接就被日志框架识别了，相当于绕过了 springboot；

**logback-spring.xml**：日志框架就不直接加载日志的配置项（因为日志框架只认识 logback.xml），由SpringBoot解析日志配置，就可以使用SpringBoot的高级Profile功能

```xml
<springProfile name="staging">
    <!-- configuration to be enabled when the "staging" profile is active -->
  	可以指定某段配置只在某个环境下生效
</springProfile>


```

如在开发环境输入》》》不在开发环境输出》》》：直接运行就行，这里肯定不是开发环境（在 application.properties 中使用 spring.profiles.active=dev 可以进行指定）

```xml
<appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <!--
        日志输出格式：
			%d表示日期时间，
			%thread表示线程名，
			%-5level：级别从左显示5个字符宽度
			%logger{50} 表示logger名字最长50个字符，否则按照句点分割。 
			%msg：日志消息，
			%n是换行符
        -->
        <layout class="ch.qos.logback.classic.PatternLayout">
            <springProfile name="dev">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ----> [%thread] ---> %-5level %logger{50} - %msg%n</pattern>
            </springProfile>
            <springProfile name="!dev">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ==== [%thread] ==== %-5level %logger{50} - %msg%n</pattern>
            </springProfile>
        </layout>
    </appender>

```



如果使用logback.xml作为日志配置文件，还要使用profile功能，会有以下错误

 `no applicable action for [springProfile]`

## 五、切换日志框架

可以按照slf4j的日志适配图（见上面），进行相关的切换；

slf4j+log4j的方式；不推荐

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <exclusions>
    <exclusion>
      <artifactId>logback-classic</artifactId>
      <groupId>ch.qos.logback</groupId>
    </exclusion>
    <exclusion>
      <artifactId>log4j-over-slf4j</artifactId>
      <groupId>org.slf4j</groupId>
    </exclusion>
  </exclusions>
</dependency>

<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
</dependency>


```

切换为log4j2：不推荐

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <artifactId>spring-boot-starter-logging</artifactId>
            <groupId>org.springframework.boot</groupId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>

```

