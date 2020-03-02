# Spring注解驱动开发（一）

![image-20200223204936048](Untitled.resource/image-20200223204936048.png)

## 一、容器

## 1. 组件注册 @Configuration 和 @Bean 的注入

对于下面对象

```java
package com.gjxaiou.bean;

/**
 * @Author GJXAIOU
 * @Date 2020/3/2 9:44
 */

import lombok.*;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@ToString
public class Person {
    private String name;
    private int age;
}

```



### 1）使用xml方式

我们一起注入一个bean使用xml来配置，实现注册组件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <bean id="person" class="com.cuzz.bean.Person">
        <property name="name" value="cuzz"></property>
        <property name="age" value="18"></property>
    </bean>
    
</beans>
```

我可以使用`ClassPathXmlApplicationContext`来获取

```java
/**
 * @Author: cuzz
 * @Date: 2018/9/23 10:48
 * @Description:
 */
public class MainTest {
    public static void main(String[] args) {
        // 返回 IOC 容器 applicationContaxt
        ApplicationContext  applicationContext = new ClassPathXmlApplicationContext("bean.xml");
        // 用id获取
        Person bean = (Person) applicationContext.getBean("person");
        System.out.println(bean);
    }
}
```

输出`Person(name=cuzz, age=18)`

### 2 ) 注解

编写一个配置类，等同于之前的配置文件

```java
/**
 * @Author: cuzz
 * @Date: 2018/9/23 10:55
 * @Description: 配置类
 */
@Configuration // 告诉Spring这是一个配置类
public class MainConfig {
    // 给容器中注册一个Bean,类型为返回值类型,id默认用方法名，即直接使用 @Bean 即可
    // 也可以指定id
    @Bean(value = "person01")
    public Person person() {
        return new Person("vhsj", 16);
    }
}
```

可以通过`AnnotationConfigApplicationContext`来获取，并且获取id

```java
/**
 * @Author: cuzz
 * @Date: 2018/9/23 10:59
 * @Description:
 */
public class MainTest {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MainConfig.class);
        Person person = (Person) context.getBean(Person.class);
        System.out.println(person);

        String[] names = context.getBeanNamesForType(Person.class);
        for (String name: names) {
            System.out.println(name);
        }
    }
}
```

输出

```
Person(name=vhsj, age=16)
person01
```

由于给bean添加一个一个value，可以改变默认id



**上面需要每一个都配置扫描，可以批量扫描**

## 2. 组件注册@ComponentScan

### 1） 使用xml

只要标注了注解就能扫描到如:@Controller @Service @Repository @component

```xml
<context:component-scan base-package="com.gjxaiou"></context:component-scan>
```

### 2） 使用注解

在配置类中添加

```java
package com.gjxaiou.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

/**
 * @Author GJXAIOU
 * @Date 2020/3/2 10:23
 */
// 告诉 Spring 此为配置类
@Configuration
// 扫描指定包
@ComponentScan(value = "com.gjxaiou")
public class MainConfig {
}

```

然后添加dao/service/controller 等供测试

```java
package com.gjxaiou.dao;

import org.springframework.stereotype.Repository;

/**
 * @Author GJXAIOU
 * @Date 2020/3/2 10:19
 */
@Repository
public class BookDao {
}

```



```java
package com.gjxaiou.service;

import org.springframework.stereotype.Service;

/**
 * @Author GJXAIOU
 * @Date 2020/3/2 10:20
 */
@Service
public class BookService {
}

```



```java
package com.gjxaiou.controller;

import org.springframework.stereotype.Controller;

/**
 * @Author GJXAIOU
 * @Date 2020/3/2 10:19
 */
@Controller
public class BookController {
}

```





测试

```java
/**
 * @Author: cuzz
 * @Date: 2018/9/23 13:03
 * @Description:
 */
public class IOCTest {

    @Test
    public void test01() {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
        // 获取所有bean定义的名字
        String[] beanNames = applicationContext.getBeanDefinitionNames();
        for (String name : beanNames) {
            System.out.println(name);
        }
    }
}
```

输出结果

```
// IOC 自身需要装载的组件
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory

// mainConfig 本身也是一个组件，因为 @Configuration 本身源码上也带有 @Component，所以也是一个组件
mainConfig
bookController
bookDao
bookService
```

可以看出添加@Controller @Service @Repository @component注解的都可以扫描到



还可以指定添加某些类，和排除某些类，进入ComponentScan注解中有下面两个方法

```java
ComponentScan.Filter[] includeFilters() default {};
ComponentScan.Filter[] excludeFilters() default {};

includeFilters = Filter[] ：指定扫描的时候只需要包含哪些组件
excludeFilters = Filter[] ：指定扫描的时候按照什么规则排除那些组件
```

配置类，排除Controller 和 Service，如果要 配置只包含某些，就要将下面的 excludeFilters 替换为 includeFilters 即可，并且使 `useDefaultFilters = false`  即可，完整为：`@ComponentScan(value = "com.gjxaiou", includeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class,Service.class}, useDefaultFilters = false)
})`

```java
@Configuration // 告诉Spring这是一个配置类
@ComponentScan(value = "com.cuzz", excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class,Service.class})
})
public class MainConfig {

}
```

运行测试方法，可以得出没有Controller类的

```
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalRequiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
mainConfig
bookDao
```

**同样可以同时制定多个扫描规则**

```java
package com.gjxaiou.config;

import com.gjxaiou.service.BookService;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.ComponentScans;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.FilterType;
import org.springframework.stereotype.Controller;
import org.springframework.stereotype.Repository;
import org.springframework.stereotype.Service;

/**
 * @Author GJXAIOU
 * @Date 2020/3/2 10:23
 */
// 告诉 Spring 此为配置类
@Configuration
// 扫描指定包
@ComponentScans(value = {
        @ComponentScan(value = "com.gjxaiou",
        includeFilters = {@ComponentScan.Filter(type = FilterType.ANNOTATION, classes =
                {Controller.class, Service.class}),
                @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = BookService.class)},
        useDefaultFilters = false),
        @ComponentScan(value = "com.gjxaiou",
        excludeFilters = {@ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Repository.class})})})

public class MainConfig {
}

```

测试结果为：

```java
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
mainConfig
bookController
bookService
```





### 3 ) 自定义 TypeFilter 指定过滤规则

第一和第二比较常用

```
FilterType.ANNOTATION：按照注解
FilterType.ASSIGNABLE_TYPE：按照给定的类型；无论是该类型还是子类、实现类都会被注入
FilterType.ASPECTJ：使用ASPECTJ表达式
FilterType.REGEX：使用正则指定
FilterType.CUSTOM：使用自定义规则
```

自定义规则示例：新建一个MyTypeFilte类实现 TypeFilter 接口（该类必须实现该接口）

```java
/**
 * @Author: cuzz
 * @Date: 2018/9/23 15:03
 * @Description:
 */
public class MyTypeFilter implements TypeFilter{
    /**
     * metadataReader：读取到的当前正在扫描的类的信息
     * metadataReaderFactory:可以获取到其他任何类信息的
     */
    @Override
    public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory)
            throws IOException {
        //获取当前类注解的信息
        AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
        //获取当前正在扫描的类的类信息
        ClassMetadata classMetadata = metadataReader.getClassMetadata();
        //获取当前类资源（类的路径）
        Resource resource = metadataReader.getResource();

        String className = classMetadata.getClassName();
        System.out.println("--->"+className);
        // 这些类名中包含er就返回true，匹配成功就会被包含在容器中
        if(className.contains("er")){
            return true;
        }
        return false;
    }
}
```

使用自定义注解记得需要关闭默认过滤器`useDefaultFilters = false`

```java
/**
 * @Author: cuzz
 * @Date: 2018/9/23 10:55
 * @Description: 配置类
 */
@Configuration 
@ComponentScan(value = "com.cuzz",
        includeFilters = @ComponentScan.Filter(type = FilterType.CUSTOM,
                classes = MyTypeFilter.class),
        useDefaultFilters = false)
public class MainConfig {
    // 给容器中注册一个Bean,类型为返回值类型,id默认用方法名
    // 也可以指定id
    @Bean(value = "person01")
    public Person person() {
        return new Person("vhsj", 16);
    }
}
```

测试

```
--->com.cuzz.AppTest
--->com.cuzz.bean.MainTest
--->com.cuzz.config.IOCTest
--->com.cuzz.config.MainTest
--->com.cuzz.App
--->com.cuzz.bean.Person
--->com.cuzz.config.MyTypeFilter
--->com.cuzz.controller.BookController
--->com.cuzz.dao.BookDao
--->com.cuzz.sevice.BookService
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalRequiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
mainConfig     // 不是扫描的 
person		   // 这个是在bean中
myTypeFilter   // 有er
bookController // 有er
bookService    // 有er
person01       // 这个是在bean中
```

## 3. 组件注册@Scope设置作用域

首先新建 配置类，

```java
package com.gjxaiou.config;

import com.gjxaiou.bean.Person;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * @Author GJXAIOU
 * @Date 2020/3/2 11:32
 */
@Configuration
public class MainConfig2 {
    // 加入容器中
    @Bean("person")
    public Person person() {
        return new Person("张三", 25);
    }
}

```



### 1）Spring的bean默认是单例的

```java
import com.gjxaiou.config.MainConfig;
import org.junit.jupiter.api.Test;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * @Author GJXAIOU
 * @Date 2020/3/2 10:25
 */
public class IOCTest {

    @Test
    public void test02() {
        AnnotationConfigApplicationContext applicationContext =
                new AnnotationConfigApplicationContext(MainConfig2.class);
        // 获取所有bean定义的名字
        String[] beanNames = applicationContext.getBeanDefinitionNames();
        for (String name : beanNames) {
            System.out.println(name);
        }
        // 因为默认单例，如果两个对象相同
        Object bean = applicationContext.getBean("person");
        Object bean2 = applicationContext.getBean("person");
        System.out.println(bean == bean2);   // 输出true
    }

}

```

### 2）Scope的四个范围

```
ConfigurableBeanFactory#SCOPE_PROTOTYPE   // 多实例 每次获取时创建对象，不会放在ioc容器中
ConfigurableBeanFactory#SCOPE_SINGLETON   // 单实例 ioc容器启动时创建对象并且放入容器中，以后从容器中获取
WebApplicationContext#SCOPE_REQUEST       // web同一次请求创建一个实例
WebApplicationContext#SCOPE_SESSION       // web同一个session创建一个实例
```


如果我们把Scope修改

```java
/**
 * @Author: cuzz
 * @Date: 2018/9/23 15:40
 * @Description:
 */
@Configuration
public class MainConfig2 {

    @Scope(value = "prototype")
    @Bean
    public Person person() {
        return new Person("vhuj", 25);
    }
}
```

则测试输出false

**针对上面创建对象放入 IOC 容器中时机**：

首先是配置类：针对单例：

```java

@Configuration
public class MainConfig2 {

    @Bean
    public Person person() {
        system.out.println("给容器中添加 Person");
        return new Person("vhuj", 25);
    }
}
```

测试类：

```java
import com.gjxaiou.config.MainConfig;
import org.junit.jupiter.api.Test;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * @Author GJXAIOU
 * @Date 2020/3/2 10:25
 */
public class IOCTest {

    @Test
    public void test02() {
        // 只创建 IOC 容器，先不获取 Bean
        AnnotationConfigApplicationContext applicationContext =
                new AnnotationConfigApplicationContext(MainConfig2.class);
    }

}

```

结果说明在 IOC 容器创建的时候，该方法就被调用了，创建了对象然后给容器中添加了 Person，以后每次获取直接从容器中拿，不会再次创建。

`给容器中添加 Person`

**多例模式：**

配置类：

```java

@Configuration
public class MainConfig2 {
	@Scope("prototype")
    @Bean
    public Person person() {
        system.out.println("给容器中添加 Person");
        return new Person("vhuj", 25);
    }
}
```

如果测试类没有变化，发现执行之后是没有任何输出的，只有修改测试类。添加获取 Bean 方法之后：

```java
import com.gjxaiou.config.MainConfig;
import org.junit.jupiter.api.Test;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * @Author GJXAIOU
 * @Date 2020/3/2 10:25
 */
public class IOCTest {

    @Test
    public void test02() {
        AnnotationConfigApplicationContext applicationContext =
                new AnnotationConfigApplicationContext(MainConfig2.class);
      	system.out.println("IOC 容器创建完成");
        Object bean = applicationContext.getBean("person");
    }

}

```

输出结果为：在创建完成，需要获取 Bean 的时候才会创建该 Person 对象

```java
IOC 容器创建完成
给容器中添加 person
```

同时如果多次使用 ` Object bean = applicationContext.getBean("person");` 获取的时候，每次获取的时候都会添加一次。

```java
IOC 容器创建完成
给容器中添加 person
给容器中添加 person
```



## 4. 组件注册@Lazy-bean懒加载

### 1）懒加载

懒加载的是针对单实例Bean，因为单实例 Bean 默认是在容器启动的时创建的，我们可以设置懒加载容器启动是不创建对象，在第一次使用（获取）Bean创建对象，并初始化

### 2 ) 测试

先给添加一个@Lazy注解

```java
@Configuration
public class MainConfig2 {

    @Lazy
    @Bean
    public Person person() {
        System.out.println("给容器中添加Person...");
        return new Person("vhuj", 25);
    }
}
```

编写一个测试方法

```java
    @Test
    public void test03() {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);

        System.out.println("ioc容器创建完成...");
        Object bean = applicationContext.getBean("person");
    }
```

输出如下：如果没有获取 Bean 的语句的话，只会输出：`ioc容器创建完成...`

```
ioc容器创建完成...
给容器中添加Person...
```

添加一个@Lazy是在第一次获取时，创建对象，以后获取就不需要创建了，直接从容器中获取，因为它是单实例

## 5. 组件注册@Conditional按条件注册

按照一定条件进行判断，满足条件才会给容器中注册Bean，不再是使用 @Bean 就直接给容器中注册 Bean了。

这里 @Conditional 标注在每个具体的方法上，也可以标注在 类 上，对整个类进行统一处理。

### 1 ) 编写自己的Condition类

如果系统是windows，给容器中注入"bill"

如果系统是linux，给容器中注入"linus"

编写WindowCondition类并重写matches方法，linux 的同下，这里忽略，如果要测试 Linux，可以将 VM arguments 修改为：`-Dos.name=linux`

  ```java
/**
   * @Author: cuzz
   * @Date: 2018/9/23 20:30
   * @Description: 判断是否是windows
   */
  public class WindowCondition implements Condition{
  
      /**
       * @param context 判断条件能使用的上下文（环境）
       * @param metadata 注释信息
       * @return boolean
       */
      @Override
      public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
          // 获取 IOC 容器运行环境
          Environment environment = context.getEnvironment();
          String property = environment.getProperty("os.name");
          if (property.contains("Windows")) {
              return true;
          }
          return false;
      }
  }
  ```

context 还有以下方法

  ```java
  // 能获取ioc使用的beanfactory
  ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
  // 能获取到类加载器
  ClassLoader classLoader = context.getClassLoader();
  // 获取到环境变量
  Environment environment = context.getEnvironment();
  // 获取到Bean定义的注册类
  BeanDefinitionRegistry registry = context.getRegistry();
  ```

 ### 2）配置类

添加Bean添加Condition条件

```java
@Configuration
public class MainConfig2 {

    // （）里面是条件
    @Conditional({WindowCondition.class})
    @Bean("bill")
    public Person person01() {
        return new Person("Bill Gates", 60);
    }
    @Conditional({LinuxCondition.class})
    @Bean("linux")
    public Person person02() {
        return new Person("linus", 45);
    }

}
```



### 3 ) 测试

```java
    @Test
    public void test04() {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);

        // 获取环境变量
        ConfigurableEnvironment environment = applicationContext.getEnvironment();
        String property = environment.getProperty("os.name");
        System.out.println(property);

        // 获取所有bean定义的名字
        String[] beanNames = applicationContext.getBeanDefinitionNames();
        for (String name : beanNames) {
            System.out.println(name);
        }

        // key 是id
        Map<String, Person> map = applicationContext.getBeansOfType(Person.class);
        System.out.println(map);
    }
```

发现只有“bill”这个Bean被注入

```
Windows 7
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalRequiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
mainConfig2
bill
{bill=Person(name=Bill Gates, age=60)}
```

### 总结

给容器中注册组件：

- 包扫描 + 组件上标注注解（@Controller/@Service/@Repository/@Component）->只适用于自己写的组件类
- @Bean 方式：（使用构造器新建组件对象，然后在方法上标注 @Bean）可以导入第三方包中的组件
- @Import 方式：快速给容器中导入一个组件
    - @import(要导入到容器中的组件)；容器中就会自动注册这个组件，id 默认为全类名
    - @ImportSelector：返回需要导入的组件的全类名数组
    - @ImportBeanDefinitionRegister：手工注册 Bean 到容器中

- 使用 Spring 提供的 FactoryBean 
    - 默认是获取工厂 Bean 调用 getObject 创建的对象；
    - 如果要获取工厂 Bean 本身，我们需要给 id 前面加上一个 &

## 6. 组件注册@Improt给容器中快速导入一个组件

### 1 ) @Import导入

@Import可以导入第三方包，或则自己写的类，比较方便，Id默认为全类名

比如我们新建一个类

```java
/**
 * @Author: cuzz
 * @Date: 2018/9/23 21:08
 * @Description:
 */
public class Color {
}
```

我们只需要在配置类添加一个@Import把这个类导入

```java
@Import({Color.class})
// @Import 可以同时导入多个组件
@Import({Color.class,Red.class})
@Configuration
public class MainConfig2 {}
```

### 2 ) ImportSelector接口导入的选择器

返回导入组件需要的全类名的数组

```java
public interface ImportSelector {

	/**
	 * Select and return the names of which class(es) should be imported based on
	 * the {@link AnnotationMetadata} of the importing @{@link Configuration} class.
	 */
	String[] selectImports(AnnotationMetadata importingClassMetadata);

}
```

编写一个MyImportSelector类实现ImportSelector接口，在该类中编写逻辑来实现具体导入哪些组件，不是导入该类到 IOC 容器，是将该类中 selectImports 方法返回的全类名导入到容器中。

```java
/**
 * @Author: cuzz
 * @Date: 2018/9/23 21:15
 * @Description:
 */
public class MyImportSelector implements ImportSelector{

    // 返回值就是导入容器组件的全类名
    // AnnotationMetadata:当前类标注的@Import注解类的所有注解信息
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[] {"com.cuzz.bean.Car"};
    }
}
```

在配置类中，通过@Import导入

```java
/**
 * @Author: cuzz
 * @Date: 2018/9/23 15:40
 * @Description: 配置类
 */
@Import({Color.class, MyImportSelector.class})
@Configuration
public class MainConfig2 {}
```

测试结果，`com.cuzz.bean.Car`注入了

```
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalRequiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
mainConfig2
com.cuzz.bean.Color
com.cuzz.bean.Car
```

### 3 ) ImportBeanDefinitionRegistrar接口选择器

```java
public interface ImportBeanDefinitionRegistrar {

	/**
	 * Register bean definitions as necessary based on the given annotation metadata of
	 * the importing {@code @Configuration} class.
	 * <p>Note that {@link BeanDefinitionRegistryPostProcessor} types may <em>not</em> be
	 * registered here, due to lifecycle constraints related to {@code @Configuration}
	 * class processing.
	 * @param importingClassMetadata annotation metadata of the importing class
	 * @param registry current bean definition registry
	 */
	public void registerBeanDefinitions(
			AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry);

}
```

编写一个ImportBeanDefinitionRegistrar实现类，作用是将所有需要添加到容器中的 Bean，通过调用 BeanDefinitionRegistry.registerBeanDefinition 手工注册进来。

```java
/**
 * @Author: cuzz
 * @Date: 2018/9/23 21:29
 * @Description:
 */
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    /**
     * @param importingClassMetadata 当前类的注解信息
     * @param registry ： Bean 定义的注册类
     */
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        // 查询容器
        boolean b = registry.containsBeanDefinition("com.cuzz.bean.Car");
        // 如果有car, 注册一个汽油类
        if (b == true) {
            // 需要添加一个bean的定义信息，定义信息包括 Bean 的类型和 Bean 的 scope 等等。
            RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(Petrol.class);
            // 注册一个bean, 指定bean名
            registry.registerBeanDefinition("petrol", rootBeanDefinition);
        }

    }
}
```

配置类

```java
/**
 * @Author: cuzz
 * @Date: 2018/9/23 15:40
 * @Description: 配置类
 */
@Import({Color.class, MyImportSelector.class, MyImportBeanDefinitionRegistrar.class})
@Configuration
public class MainConfig2 {}
```

这里创建 petrol 类忽略

测试结果，**出现了petrol**

```
org.springframework.context.annotation.internalConfigurationAnnotationProcessor
org.springframework.context.annotation.internalAutowiredAnnotationProcessor
org.springframework.context.annotation.internalRequiredAnnotationProcessor
org.springframework.context.annotation.internalCommonAnnotationProcessor
org.springframework.context.event.internalEventListenerProcessor
org.springframework.context.event.internalEventListenerFactory
mainConfig2
com.cuzz.bean.Color
com.cuzz.bean.Car 
petrol
```

## 7. 组件注册使用FactoryBean注册组件

编写一个ColorFactoryBean类

```java

// 创建一个 Spring定义的工厂Bean
public class ColorFactoryBean implements FactoryBean<Color> {
    // 返回一个Color对象，该对象会添加到容器中
    @Override
    public Color getObject() throws Exception {
        return new Color();
    }

    @Override
    public Class<?> getObjectType() {
        return Color.class;
    }
    // 是否为单例，true 表示是单例，在容器中就只会保存一份，即多次使用 getObject() 获取到对象是一样的
    @Override
    public boolean isSingleton() {
        return true;
    }
}
```

将工厂类注入到容器中

```java

@Configuration
public class MainConfig2 {
	@Bean
    public ColorFactoryBean colorFactoryBean() {
        return new ColorFactoryBean();
    }

} 

```

测试

```java
    @Test
    public void test05() {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);

        Object bean = applicationContext.getBean("colorFactoryBean");
        // 工厂 bean 获取的是调用getClass()方法创建的对象
        System.out.println("colorFactoryBean的类型是: " + bean.getClass());
    }
```

输出，**发现此时的bean调用的方法是getObjectType方法**，虽然看上去装载的是 ColorFactoryBean，但是工厂 bean 获取的是调用getClass()方法创建的对象

```
colorFactoryBean的类型是: class com.cuzz.bean.Color
```

同时如果多次使用 ` Object bean = applicationContext.getBean("colorFactoryBean");`，结果获取到的多个 Bean 是同一个 Bean。但是如果上面的工厂 Bean 的 isSingleton() 方法返回值为 false，即是多实例，则多次获取 Bean，每次都会调用 工厂 Bean的 getObject() 方法，获取多个不同的 Bean。

**如果需要获取BeanFactory本身，可以在id前面加一个“&”标识**

```java
    @Test
    public void test05() {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);

        Object bean = applicationContext.getBean("colorFactoryBean");
        // 工厂bean调用的是getClass()方法
        System.out.println("colorFactoryBean的类型是: " + bean.getClass());
        Object bean2 = applicationContext.getBean("&colorFactoryBean");
        // 工厂bean调用的是getClass()方法
        System.out.println("colorFactoryBean的类型是: " + bean2.getClass());
    }
```

此时输出

```
colorFactoryBean的类型是: class com.cuzz.bean.Color
colorFactoryBean的类型是: class com.cuzz.bean.ColorFactoryBean
```







## Bean 生命周期

# Spring注解驱动开发（二）

## 1. 声明周期@Bean指定初始化和销毁方法

###  1 ) Bean的生命周期

Bean的创建、初始化和销毁是由容器帮我们管理的

我们可以自定义初始化和销毁方法，容器在 Bean 进行到当前生命周期的时候来调用我们自定义的初始化和销毁方法

构造（对象创建）

​	单实例： 在容器启动的时候创建

​	多实例： 在每次获取的时候创建对象



**具体实现方式**

- 指定初始化方法和销毁方法
    - 通过 @Bean 指定 init–method 和 destory-method
- 通过让 Bean 实现 InitializingBean （定义初始化逻辑）和实现 DisposableBean（定义逻辑销毁）

- 使用 JSR250 中提供的注解（Java 提供）
    - @PostConstruct：在 Bean 创建完成并且属性赋值完成之后，来执行初始化方法；
    - @PreDestory：在容器销毁 Bean 之前通知费我们进行清理工作。
- 使用接口 BeanPostProcessor ：Bean 的后置处理器，**在 Bean 初始化前后**进行一些处理工作
    - postProcessBeforeInitialization：在初始化之前工作
    - postProcessAfterInitialization：在初始化之后工作

### 2 ) 指定初始化方法

**初始化：**对象创建完成后，并赋值化，调用初始化方法

**销毁：**单实例是在容器关闭的时候销毁，多实例容器不会管理这个Bean，容器不会调用销毁方法

编写一个Car类

```java
/**
 * @Author: cuzz
 * @Date: 2018/9/23 21:20
 * @Description:
 */
public class Car {

    public Car () {
        System.out.println("car constructor...");
    }

    public void init() {
        System.out.println("car...init...");
    }

    public void destroy() {
        System.out.println("car...destroy...");
    }
}
```



在xml中我们可以指定`init-method`和`destroy-method`方法，如

```xml
<bean id="car" class="com.cuzz.bean.Car" init-method="init" destroy-method="destroy"></bean>
```

使用注解我们可以

```java
/**
 * @Author: cuzz
 * @Date: 2018/9/24 12:49
 * @Description: 配置类
 */
@Configuration
public class MainConfigOfLifecycle {

    @Bean(initMethod = "init", destroyMethod = "destroy")
    public Car car() {
        return new Car();
    }

}
```

测试

```java
/**
 * @Author: cuzz
 * @Date: 2018/9/24 13:00
 * @Description:
 */
public class IOCTestLifeCycle {

    @Test
    public void test01() {
        // 创建ioc容器
        AnnotationConfigApplicationContext applicationContext =
                new AnnotationConfigApplicationContext(MainConfigOfLifecycle.class);
        System.out.println("容器创建完成...");
        // 关闭容器
        System.out.println("--->开始关闭容器");
        applicationContext.close();
        System.out.println("--->已经关闭容器");
    }
}
```

可以看出先创建car，再调用init方法，在容器关闭时销毁实例

```
car constructor...
car...init...
容器创建完成...
--->开始关闭容器
car...destroy...
--->已经关闭容器
```

在配置数据源的时候，有很多属性赋值，销毁的时候要把连接给断开



**针对多实例**

配置类：

```java
@Configuration
public class MainConfigOfLifecycle {
	@Scope("prototype")
    @Bean(initMethod = "init", destroyMethod = "destroy")
    public Car car() {
        return new Car();
    }
}
```

测试类

```java
/**
 * @Author: cuzz
 * @Date: 2018/9/24 13:00
 * @Description:
 */
public class IOCTestLifeCycle {

    @Test
    public void test01() {
        // 创建ioc容器
        AnnotationConfigApplicationContext applicationContext =
                new AnnotationConfigApplicationContext(MainConfigOfLifecycle.class);
        System.out.println("容器创建完成...");
        applicationContext.getBean("car");
        // 关闭容器
        System.out.println("--->开始关闭容器");
        applicationContext.close();
        System.out.println("--->已经关闭容器");
    }
}
```

只有在 getBean 时候才会初始化。



## 2. 生命周期InitializingBean和DisposableBean

### 1 ) InitializingBean

可以通过Bean实现InitializingBean来定义初始化逻辑，是设置好所有属性会调用`afterPropertiesSet()`方法

```java
public interface InitializingBean {

	/**
	 * Invoked by a BeanFactory after it has set all bean properties supplied
	 * (and satisfied BeanFactoryAware and ApplicationContextAware).
	 * <p>This method allows the bean instance to perform initialization only
	 * possible when all bean properties have been set and to throw an
	 * exception in the event of misconfiguration.
	 * @throws Exception in the event of misconfiguration (such
	 * as failure to set an essential property) or if initialization fails.
	 */
	void afterPropertiesSet() throws Exception;

}
```

### 2）DisposableBean

可以通过Bean实现DisposableBean来定义销毁逻辑，会调用destroy()方法

```java
public interface DisposableBean {

	/**
	 * Invoked by a BeanFactory on destruction of a singleton.
	 * @throws Exception in case of shutdown errors.
	 * Exceptions will get logged but not rethrown to allow
	 * other beans to release their resources too.
	 */
	void destroy() throws Exception;

}
```

### 3）例子

编写一个Cat类

```java

public class Cat implements InitializingBean, DisposableBean{

    public Cat() {
        System.out.println("cat constructor...");
    }


    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("cat...init...");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("cat...destroy...");
    }

}
```

然后将 cat 注入即可，这里忽略

测试

```
cat constructor...
cat...init...
容器创建完成...
--->开始关闭容器
cat...destroy...
--->已经关闭容器
```



## 3. 生命周期@PostContruct和@PreDestroy注解

@PostContruct在Bean创建完成并且属性赋值完成，来执行初始化

@PreDestroy在容器销毁Bean之前通知我们进行清理工作

编写一个Dog类，并把他注入到配置类中

```java
@Component
public class Dog {

    public Dog() {
        System.out.println("dog constructor...");
    }

   // 对象创建并且执行构造器（赋值）之后调用
    @PostConstruct
    public void postConstruct() {
        System.out.println("post construct...");
    }

    @PreDestroy
    public void preDestroy() {
        System.out.println("pre destroy...");
    }
}
```

测试结果

```
dog constructor...
post construct...
容器创建完成...
--->开始关闭容器
pre destroy...
--->已经关闭容器
```

## 4. 生命周期BeanPostProscessor后置处理器

在Bean初始化前后做一些处理

```java
public interface BeanPostProcessor {
	// 在初始化之前工作
	Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;
	// 在初始化之后工作
	Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;

}
```

编写一个MyBeanPostProcessor实现BeanPostProcessor接口

```java
/**
 * @Author: cuzz
 * @Date: 2018/9/24 14:21
 * @Description: 后置处理器，初始化前后进行处理工作
 */
// 将后置处理器加入到容器中
@Component
public class MyBeanPostProcessor implements BeanPostProcessor{
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("--->postProcessBeforeInitialization..." +"bean在容器中的名字" + beanName +"==>" + bean);
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("--->postProcessAfterInitialization..." + beanName +"==>" + bean);
        return bean;
    }
}
```

测试

```
--->postProcessBeforeInitialization...org.springframework.context.event.internalEventListenerProcessor==>org.springframework.context.event.EventListenerMethodProcessor@1dc67c2
--->postProcessAfterInitialization...org.springframework.context.event.internalEventListenerProcessor==>org.springframework.context.event.EventListenerMethodProcessor@1dc67c2
--->postProcessBeforeInitialization...org.springframework.context.event.internalEventListenerFactory==>org.springframework.context.event.DefaultEventListenerFactory@2bd765
--->postProcessAfterInitialization...org.springframework.context.event.internalEventListenerFactory==>org.springframework.context.event.DefaultEventListenerFactory@2bd765
cat constructor...
--->postProcessBeforeInitialization...cat==>com.cuzz.bean.Cat@1d3b207
cat...init...
--->postProcessAfterInitialization...cat==>com.cuzz.bean.Cat@1d3b207
容器创建完成...
--->开始关闭容器
cat...destroy...
--->已经关闭容器
```

在实例创建之前后创建之后会被执行

## 5. 生命周期BeanPostProcessor原理

通过debug到populateBean，先给属性赋值在执行initializeBean方法

```java
try {
    // 给 Bean 进行属性赋值
    populateBean(beanName, mbd, instanceWrapper);
    if (exposedObject != null) {
        exposedObject = initializeBean(beanName, exposedObject, mbd);
    }
}
```

initializeBean方法时，

```java
protected Object initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd) {


    Object wrappedBean = bean;
    if (mbd == null || !mbd.isSynthetic()) {
        // 执行before方法
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
    }
	...
    try {
        // 执行初始化
        invokeInitMethods(beanName, wrappedBean, mbd);
    }

    if (mbd == null || !mbd.isSynthetic()) {
        // 执行after方法
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    }
    return wrappedBean;
}
```



**Spring底层对`BeanPostProcessor`的使用**：

Bean赋值、注入其他组件、@Autowired、生命周期注解功能、@Async等等都使用到了BeanPostProcessor这个接口的实现类，很重要







# Spring注解驱动开发（三）

# 1. 属性赋值@value赋值

使用@Value（）赋值，其中括号中可以写以下内容

- 基本数值
- 可以写SPEL表达式 #{}
- 可以 ${} 获取配置文件信息（在运行的环境变量中的值）



使用xml时候导入配置文件是

```XML
<context:property-placeholder location="classpath:person.properties"/>
```

使用注解可以在配置类添加一个@PropertySource注解把配置文件中k/v保存到运行的环境中

使用${key}来获取

**配置文件内容**：person.properties

```properties
person.nickName = 三三
```



**配置类**

```java
/**
 * @Author: cuzz
 * @Date: 2018/9/24 18:43
 * @Description:
 */
// 导入配置文件，使用 @PropertySource 读取配置文件中的 key/value 保存到环境变量中，加载完外部的配置文件以后使用 ${} 取出配置文件的值
@PropertySource(value = {"classpath:/person.properties"})
@Configuration
public class MainConfigOfPropertyValue {

    @Bean
    public Person person() {
        return new Person();
    }
}
```



```java
@Data
public class Person {

    @Value("vhuj")
    private String name;

    @Value("#{20-2}")
    private Integer age;
// 读取配置文件中值
    @Value("${person.nickName}")
    private String nickName;
}
```

测试

```java
public class IOCTestPropertyValue{  
    AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfPropertyValue.class);
@Test
    public void test01() {
        printBean(applicationContext);
        System.out.println("---------------------------");

        Person person = (Person) applicationContext.getBean("person");
        System.out.println(person);

        System.out.println("---------------------------");
        // 因为配置文件中的 key-value 都加载到了环境变量中，所以可以根据环境变量的 key 直接获取值
        ConfigurableEnvironment environment = applicationContext.getEnvironment();
        String property = environment.getProperty("person.nickName");
        System.out.printlin(property);

    }
}
```

输出

```
---------------------------
Person(name=vhuj, age=18, nickName=三三)
---------------------------
```

# 2. 自动装配@Autowired@Qualifier@Primary

- 自动转配：

​	Spring利用依赖注入（DI），完成对 **IOC 容器中各个组件**的依赖关系赋值

-  @Autowired自动注入:
    - 默认优先按照类型去容器中寻找对应的组件，如果找到去赋值

    - 如果找到多个相同类型的组件，再将属性名（`BookDao bookdao`）作为组件的id去容器中查找

        ```java
        // 如果在 MainConfigOfAutowired 再次注入一个 bookDao，所以该类型的组件就有两个了
        @configuration
        @ComponentScan({"com.gjxaiou.dao", "com.gjxaiou.service"})
        public clas MainConfigOfAutowired{
          @Bean("bookDao2")
            public BookDao bookDao(){
            	return bookDao;
            }
        }
        ```

        

    - 还可以在 @Autowird 注解上面添加使用`@Qualifier("bookdao")`明确指定需要装配的 id

- 如果使用自动装配，被装配的组件必须在容器中存在，如果不存在报错，当然们可以指定    `@Autowired(required=false)`，指定非必须

- @Primary 让 Spring 自动装配时首先装配使用该注解的组件（存在多个同类型的组件的时候），与 @Autowired 结合使用。例如将该注解加在 MainConfigOfAutowired 的 bookDao 上，同时没有使用 @Qualifier 注解指定的情况下，自动装配时候就会优先加在该组件，如果指定了就会加在指定的组件。





测试：

```java
public class IOCTestAutowired{

	@Test
    public void test01(){
    
    	AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext("MainConfigOfAutowired.class");
        BookService bookService = applicationContext.getBean(BookService.class);
        // 看看 bookService 中有没有 bookDao 对象
        System.out.println(bookService);
        // 看和容器中的 Dao 是不是一样的
        BookDao bookDao = applicationContext.getBean(BookDao.class);
        System.out.println(bookDao);

        applicationContext.close();
    
    }

}
```

配置类：

```java
@configuration
// 这里使用包扫描的方式，将这些组件都加入容器中
@ComponentScan({"com.gjxaiou.dao", "com.gjxaiou.service"})
public clas MainConfigOfAutowired{

}
```

Service 组件中需要使用到 Dao 组件

```java
@Service
@toString
public class BookService{
    @Autowired
	private BookDao bookDao;
   
    public void print(){
    	System.out.println(bookDao);
    }
}
```

Dao 组件为：

```java
// 名字默认为类名首字母小写
@Repoitory
public class BookDao{
	

}
```

# 3. 自动装配@Resource和@Inject

Spring还支持使用@Resource (JSR250) 和@Inject (JSR330) 注解，这两个是java规范注解

- @Resource

    - @Resource和@Autowired一样实现自动装配功能，默认是按组件名称进行装配的：@Resource(name=”bookDao”)

    - 没有支持@Primary和@Autowird(required=false)的功能

- @Inject

    - 需要导入依赖 javax.inject
    - 和 Autowired 功能几乎一致，支持与 @Primary 一起使用，但是没有类似于 @Autowird(required=false)的功能 

本质上是使用 AutowiredAnnotationBeanPostProcessor 解析完成自动装配功能 

# 4. 自动装配其他地方的自动装配

@Autowired：可以在构造器、参数、方法，属性等使用。

除了下面代码示例外最后一种标注到方法位子上@Bean+方法参数，参数从容器中获取

```java
// 如果下面代码中只是,即不使用 @Component 将 Boss 加入到容器中
public class Boss{
 private Car car;
    // 加上一个有参构造方法
}
// 可以直接在配置类中将 Boss 类和其依赖的来加入
public class MainConfigOfAutowired{
    @Bean
    // 当然可以在参数位置加上 @Autowired，也可以不加
	public Boss boss(Car car){
        Boss boss = new Boss();
        boss.setBoss(car);
    return boss;
    }    
}
```



```java
package com.gjxaiou.bean
@Component
public class Boss {
    // 方式一：标注在属性上
    @Autowired
    private Car car;
	
    // 默认情况下，在 IOC 中的组件，容器启动的时候会调用无参构造器创建对象，在进行初始化赋值等操作，这里实现有参构造器。
    // 方法三：标注在有参构造器上，构造器中要用的组件也是从容器中获取（如果组件只有一个有参构造器，则该有参构造器的 @Autowired 可以省略）
    @Autowired
    public Boss( Car car) {
        this.car = car;
       System.out.println("Boss  有参构造器");
    }

    public Car getCar() {
        return car;
    }
	
    // 方式二：标注在 set 方法上
    @Autowired		 // 参数
    // 标注在方法上的时候， Spring 容器创建当前对象的时候，就会调用方法完成赋值，这里的方法使用的参数是自定义类型，所有直接从 IOC 容器中获取
    public void setCar( Car car) {
        this.car = car;
    }
    
    // 方法四：标注在有参构造器或者 set 方法的参数上面：XXX(@Autowired Car car) 
}
```

对应的 car 也使用 @Component 加载到容器中，代码通上。

然后在对应的配置类 MainConfigOfAutowired 上面扫描将 Boss 加入

```java
@configuration
// 这里使用包扫描的方式，将这些组件都加入容器中
@ComponentScan({"com.gjxaiou.dao", "com.gjxaiou.service", "com.gjxaiou.bean"})
public clas MainConfigOfAutowired{

}
```



# 5. 自动装配Aware注入Spring底层注解

自定义组件想要使用 Spring 容器底层的一些组件（如 ApplicationContext，BeanFactory 等等），需要自定义组件实现 xxxAware，在创建对象的时候会调用接口规定的方法注入相关的组件。

```java
/**
 * Marker superinterface indicating that a bean is eligible to be
 * notified by the Spring container of a particular framework object
 * through a callback-style method. Actual method signature is
 * determined by individual subinterfaces, but should typically
 * consist of just one void-returning method that accepts a single
 * argument.
 */
public interface Aware {

}
```

我们实现几个常见的Aware接口

```java
/**
 * @Author: cuzz
 * @Date: 2018/9/25 10:18
 * @Description:
 */
@Component
public class Red implements BeanNameAware ,BeanFactoryAware, ApplicationContextAware {
    private ApplicationContext applicationContext;

    @Override
    public void setBeanName(String name) {
        System.out.println("当前Bean的名字: " + name);
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("当前的BeanFactory: " + beanFactory);
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
        System.out.println("传入的ioc: " + applicationContext);
    }
}
```

注入到配置中测试

```java
/**
 * @Author: cuzz
 * @Date: 2018/9/25 10:28
 * @Description:
 */
public class IOCTestAware {

    @Test
    public void test01() {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfAware.class);

    }
}
```

测试结果

```
当前Bean的名字: red
当前的BeanFactory: org.springframework.beans.factory.support.DefaultListableBeanFactory@159c4b8: defining beans [org.springframework.context.annotation.internalConfigurationAnnotationProcessor,org.springframework.context.annotation.internalAutowiredAnnotationProcessor,org.springframework.context.annotation.internalRequiredAnnotationProcessor,org.springframework.context.annotation.internalCommonAnnotationProcessor,org.springframework.context.event.internalEventListenerProcessor,org.springframework.context.event.internalEventListenerFactory,mainConfigOfAware,red]; root of factory hierarchy
传入的ioc: org.springframework.context.annotation.AnnotationConfigApplicationContext@1e89d68: startup date [Tue Sep 25 10:29:17 CST 2018]; root of context hierarchy

```

**总结**：作用：把Spring自定义组件注入到容器中

**原理：**

```java
public interface ApplicationContextAware extends Aware {}
```

`xxxAware`都是通过`xxxProcessor`来处理的

比如：`ApplicationContextAware`  对应`ApplicationContextAwareProcessor`

# 6. 自动装配@Profile环境搭建

Profile是 Spring 为我们提供可以根据当前环境，动态的激活和切换一系组件的功能，例如开发环境、测试环境、生产环境中的数据库数据源在不改变代码情况下进行改变。

首先通过配置类将数据源加入容器中，然后通过 @Profile 来指定组件在哪个环境的情况下才能被注册到容器中。只有该环境被激活才会被注册到容器中，默认环境为 “Default”。**当然该注解也可以标注在类上**。

注：没有标注环境标识的 Bean 在任何环境下都是加载的。

```java
@Configuration
public class MainConfigOfProfile{
    @Profile("test")
	@Bean("testDataSource")
    public DataSource dataSourceTest() throws Exception{
    	ComboPoolDataSource dataSource = new ComboPoolDataSource();
        dataSource.setUser("root");
        dataSource.setPassword("12345");
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        return dataSource;
    }
    
    	@Profile("dev")
    	@Bean("devDataSource")
    public DataSource dataSourceDev() throws Exception{
    	ComboPoolDataSource dataSource = new ComboPoolDataSource();
        dataSource.setUser("root");
        dataSource.setPassword("12345");
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/dev");
        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        return dataSource;
    }
}
```

切换环境方式：

- 使用命令动态参数激活：虚拟机参数位子加载 `-Dspring.profiles.active=test`

- 使用代码激活环境，见下：

```java

public class IOCTestProfile {

    @Test
    public void test01() {
        // 1. 使用无参构造器创建一个applicationContext
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
        // 2. 设置要激活的环境
        applicationContext.getEnvironment().setActiveProfiles("test");
        // 3. 注册主配置类
        applicationContext.register(MainConfigOfProfile.class);
        // 4. 启动刷新容器
        applicationContext.refresh();
    }
}
```

