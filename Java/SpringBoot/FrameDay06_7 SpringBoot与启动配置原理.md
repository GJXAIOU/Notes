# 七、Spring Boot启动配置原理

「注」以下分析均基于 `1.5.9.RELEASE` ，2.0 以上版本会有所不同。

## 运行流程：run()

- 准备环境
    - 执行ApplicationContextInitializer. initialize()
    - 监听器SpringApplicationRunListener回调contextPrepared
    - 加载主配置类定义信息
    - 监听器SpringApplicationRunListener回调contextLoaded
- 刷新启动IOC容器；
    - 扫描加载所有容器中的组件
    - 包括从META-INF/spring.factories中获取的所有EnableAutoConfiguration组件
- 回调容器中所有的ApplicationRunner、CommandLineRunner的run方法
- 监听器SpringApplicationRunListener回调finished

## 自动配置原理

几个重要的事件回调机制

配置在META-INF/spring.factories

**ApplicationContextInitializer**

**SpringApplicationRunListener**



只需要放在ioc容器中

**ApplicationRunner**

**CommandLineRunner**



## 启动原理

在主配置类的 `SpringApplication.run(SpringBoot07Application.class, args);` 打断点调试进入，底层调用为：

```java
// 下面为：SpringApplication 类中两个重载的方法
public static ConfigurableApplicationContext run(Object source, String... args) {
    // 第一步到这里，就是直接调用带下面的 run() 方法。
    return run(new Object[]{source}, args);
}

public static ConfigurableApplicationContext run(Object[] sources, String[] args) {
    // 第二步，该步骤分为两步：1. new 一个 SpringApplication 对象，2.执行 run() 方法
    return (new SpringApplication(sources)).run(args);
}
```

所以启动过程分为两步

- 步骤一：创建 SpringApplication 对象
- 步骤二：执行  run() 方法

### 步骤总结

启动即指 `SpringApplication.run(主程序类, args)` 的执行步骤

- 步骤一：创建 SpringApplicaton 对象，即 `new SpringApplication(主程序类)`
    - 首先判断是否 web 应用
    - 加载并保存所有 ApplicationContextInitializer（在 `META-INF/spring.factories`中）
    - 加载并保存所有 ApplicationListener
    - 获取到主程序类
- 执行 `run()` 方法
    - 回调所有的 SpringApplicationRunListener（META-INF/spring.factories）的 starting
    - 获取 ApplicationArguments
    - 准备环境&回调所有监听器（SpringApplicationRunListener ）的 environmentPrepared
    - 打印 banner 信息
    - 创建 IOC 容器对象
        - AnnotationConfigEmbeddedWebApplicationContext（web 环境容器）
        - AnnotationConfigApplicationContext（普通环境容器）

### 1、创建 SpringApplication 对象

步骤一：调用 `initialize()` 方法

```java
// 通过 SpringApplication 的构造方法来创建对象
public SpringApplication(Object... sources) {
    // 基本属性赋值
    this.bannerMode = Mode.CONSOLE;
    this.logStartupInfo = true;
    this.addCommandLineProperties = true;
    this.headless = true;
    this.registerShutdownHook = true;
    this.additionalProfiles = new HashSet();
    // 调用 initialize() 方法
    this.initialize(sources);
}
```

`initialize()` 方法主要代码见下：

```java
private void initialize(Object[] sources) {
    // 保存主配置类（sources 就是主配置类）
    if (sources != null && sources.length > 0) {
        this.sources.addAll(Arrays.asList(sources));
    }
    // 判断当前应用是否是一个web应用
    this.webEnvironment = deduceWebEnvironment();
  // 详解一：从类路径下找到META-INF/spring.factories配置的所有ApplicationContextInitializer；然后保存起来（详解见下面）
    setInitializers((Collection) getSpringFactoriesInstances(
        ApplicationContextInitializer.class));
    // 从类路径下找到 META-INF/spring.factories 配置的所有 ApplicationListener
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    // 从多个配置类中找到有main方法的主配置类
    this.mainApplicationClass = deduceMainApplicationClass();
}
```

详解一调用步骤：`getSpringFactoriesInstances()` -> `getSpringFactoriesInstances()` -> `SpringFactoriesLoader.loadFactoryNames(type, classLoader)` -> `classLoader.getResources("META-INF/spring.factories") `

详解一中找到的所有 initializer

![image-20201112133514770](FrameDay06_7%20SpringBoot%E4%B8%8E%E5%90%AF%E5%8A%A8%E9%85%8D%E7%BD%AE%E5%8E%9F%E7%90%86.resource/image-20201112133514770.png)

详解二中找到的所有 Listener

![](FrameDay06_7%20SpringBoot%E4%B8%8E%E5%90%AF%E5%8A%A8%E9%85%8D%E7%BD%AE%E5%8E%9F%E7%90%86.resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20180306145855.png)

### 2、运行run方法

`run()` 方法源代码见下：

```java
public ConfigurableApplicationContext run(String... args) {
   StopWatch stopWatch = new StopWatch();
   stopWatch.start();
   // 声明一个空的 IOC 容器
   ConfigurableApplicationContext context = null;
   FailureAnalyzers analyzers = null;
   configureHeadlessProperty();
    
   // 获取SpringApplicationRunListeners；从类路径下META-INF/spring.factories
   SpringApplicationRunListeners listeners = getRunListeners(args);
    //回调所有的获取SpringApplicationRunListener.starting()方法
   listeners.starting();
   try {
       //封装命令行参数（参数就是 args）
      ApplicationArguments applicationArguments = new DefaultApplicationArguments(
            args);
      //准备环境
      ConfigurableEnvironment environment = prepareEnvironment(listeners,
            applicationArguments);
       //创建环境完成后,回调SpringApplicationRunListener.environmentPrepared()；表示环境准备完成
       
      // 这里就是打印出 Spring 图标了
      Banner printedBanner = printBanner(environment);
       
       //创建ApplicationContext；内部决定创建web的ioc还是普通的ioc
      context = createApplicationContext();
       // 用于出现异常做异常分析报告的
      analyzers = new FailureAnalyzers(context);
       
       //准备上下文环境;内部就是将environment保存到ioc中；而且applyInitializers()；
       //applyInitializers()：回调之前保存的所有的ApplicationContextInitializer的initialize方法
       //同时回调所有的SpringApplicationRunListener的contextPrepared()；
      prepareContext(context, environment, listeners, applicationArguments,
            printedBanner);
       //prepareContext运行完成以后回调所有的SpringApplicationRunListener的contextLoaded（）；
       
       //刷新容器；即ioc容器初始化（即加载 IOC容器里面所有的组件，会扫描所有的配置类和 @Bean 来创建对象初始化，如果是web应用还会创建嵌入式的Tomcat）；Spring注解版
       //这里就是扫描，创建，加载所有组件的地方；（配置类，组件，自动配置）
      refreshContext(context);
       //从ioc容器中获取所有的ApplicationRunner和CommandLineRunner进行回调
       //优先级就是：ApplicationRunner先回调，CommandLineRunner再回调
      afterRefresh(context, applicationArguments);
       //所有的SpringApplicationRunListener回调finished方法
      listeners.finished(context, null);
      stopWatch.stop();
      if (this.logStartupInfo) {
         new StartupInfoLogger(this.mainApplicationClass)
               .logStarted(getApplicationLog(), stopWatch);
      }
       //整个SpringBoot应用启动完成以后返回启动的ioc容器；
      return context;
   }
   catch (Throwable ex) {
      handleRunFailure(context, listeners, analyzers, ex);
      throw new IllegalStateException(ex);
   }
}
```

## 3、事件监听机制

下面两个配置在META-INF/spring.factories

**ApplicationContextInitializer**

```java
public class HelloApplicationContextInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext> {
    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        System.out.println("ApplicationContextInitializer...initialize..."+applicationContext);
    }
}

```

**SpringApplicationRunListener**

```java
public class HelloSpringApplicationRunListener implements SpringApplicationRunListener {

    //必须有的构造器
    public HelloSpringApplicationRunListener(SpringApplication application, String[] args){

    }

    @Override
    public void starting() {
        System.out.println("SpringApplicationRunListener...starting...");
    }

    @Override
    public void environmentPrepared(ConfigurableEnvironment environment) {
        Object o = environment.getSystemProperties().get("os.name");
        System.out.println("SpringApplicationRunListener...environmentPrepared.."+o);
    }

    @Override
    public void contextPrepared(ConfigurableApplicationContext context) {
        System.out.println("SpringApplicationRunListener...contextPrepared...");
    }

    @Override
    public void contextLoaded(ConfigurableApplicationContext context) {
        System.out.println("SpringApplicationRunListener...contextLoaded...");
    }

    @Override
    public void finished(ConfigurableApplicationContext context, Throwable exception) {
        System.out.println("SpringApplicationRunListener...finished...");
    }
}

```

如果要起作用需要配置在（ resources/META-INF/spring.factories）

```properties
org.springframework.context.ApplicationContextInitializer=\
com.atguigu.springboot.listener.HelloApplicationContextInitializer

org.springframework.boot.SpringApplicationRunListener=\
com.atguigu.springboot.listener.HelloSpringApplicationRunListener
```





下面两个只需要放在ioc容器中

**ApplicationRunner**

```java
@Component
public class HelloApplicationRunner implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("ApplicationRunner...run....");
    }
}
```



**CommandLineRunner**

```java
@Component
public class HelloCommandLineRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("CommandLineRunner...run..."+ Arrays.asList(args));
    }
}
```

### 二、自动配置

- Spring Boot 启动扫描所有 jar 包的 `META-INF/spring.factories` 中配置的 `EnableAutoConfiguration` 组件。
- `spring-boot-autoconfigure.jar\META-INF\spring.factories` 有启动时需要加载的`EnableAutoConfiguration` 组件配置
- 配置文件中使用 `debug=true` 可以观看到当前启用的自动配置的信息
- 自动配置会为容器中添加大量组件
- Spring Boot 在做任何功能都需要从容器中获取这个功能的组件
- Spring Boot 总是遵循一个标准；容器中有我们自己配置的组件就用我们配置的，没有就用自动配置默认注册进来的组件；