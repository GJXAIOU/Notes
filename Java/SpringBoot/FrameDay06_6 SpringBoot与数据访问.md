# Spring Boot 与数据访问

[TOC]

对于数据访问层，无论是 SQL 还是 NOSQL，Spring Boot 默认采用整合 Spring Data 的方式进行统一处理，添加大量自动配置，屏蔽了很多设置。引入各种 xxxTemplate，xxxRepository 来简化我们对数据访问层的操作。对我们来说只需要进行简单的设置即可。我们将在数据访问章节测试使用 SQL 相关、NOSQL 在缓存、消息、检索等章节测试。主要包括：JDBC、MyBatis、JPA。

## 一、整合 JDBC

步骤一：在 pom.xml 导入对应的 starter，以及 mysql 的驱动

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

步骤二：在 `application.yml` 中配置以下属性用于访问数据库。

```yaml
spring:
  datasource:
    username: root
    password: GJXAIOU
    url: jdbc:mysql://localhost:3306
    # 使用的 mysql 版本为：5.7.25
    driver-class-name: com.mysql.jdbc.Driver
```

经过上面配置后，通过下面代码进行测试 Springboot 使用的默认数据源是：

```java
package com.gjxaiou.springbootdata;

import org.junit.jupiter.api.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.SQLException;

@RunWith(SpringRunner.class)
@SpringBootTest
class SpringbootdataApplicationTests {
    @Autowired
    DataSource dataSource;

    @Test
    void contextLoads() throws SQLException {
        // output：class com.zaxxer.hikari.HikariDataSource
        // springboot 1.X 版本输出为：org.apache.tomcat.jdbc.pool.DataSource
        System.out.println(dataSource.getClass());
        Connection connection = dataSource.getConnection();
        // output：HikariProxyConnection@398088176 wrapping com.mysql.cj.jdbc.ConnectionImpl@3481ff98
        System.out.println(connection);
        connection.close();
    }
}
```

默认是用 `org.apache.tomcat.jdbc.pool.DataSource` 作为数据源；

Springboot 中有关数据源的相关配置都在 `DataSourceProperties` 类里面；

自动配置原理在 `org.springframework.boot.autoconfigure.jdbc` 包中。

- 类一：参考 `DataSourceConfiguration`类，主要作用就是向容器中添加数据源。即根据配置创建数据源，默认使用hikari连接池；可以使用 `spring.datasource.type` 指定自定义的数据源类型；

2、SpringBoot 默认可以支持以下三种数据源，见 `DataSourceConfiguration`类

- `org.apache.tomcat.jdbc.pool.DataSource`

- `com.zaxxer.hikari.HikariDataSource`
- `org.apache.commons.dbcp.BasicDataSource`

3、自定义数据源类型

```java
/**
 * Generic DataSource configuration.
 */
@ConditionalOnMissingBean(DataSource.class)
@ConditionalOnProperty(name = "spring.datasource.type")
static class Generic {

   @Bean
   public DataSource dataSource(DataSourceProperties properties) {
       //使用DataSourceBuilder创建数据源，利用反射创建响应type的数据源，并且绑定相关属性
      return properties.initializeDataSourceBuilder().build();
   }

}
```

4、数据源自动配置类：`DataSourceAutoConfiguration`

该类中 `dataSourceInitializer()` 方法返回值是 `DataSourceInitializer` 类型，该类是实现了 `ApplicationListener`接口，这是一个监听器。

DataSourceInitializer 类的作用：

- 在初始化的时候（即 `init()`方法中）掉用   `runSchemaScripts();` 运行建表语句；

- 在监听到某个事件的时候（即 `onApplicationEvent()` 方法中）`runDataScripts();`运行插入数据的sql语句；

默认只需要将文件命名为：

```properties
建表语句：schema-*.sql
数据文件：data-*.sql
默认规则：schema.sql，schema-all.sql；
如果不想使用默认规则，可以使用下面配置在 application.yml 中自定义位置和文件名称   
	schema:
      - classpath:department.sql
      指定位置
```

5、操作数据库：自动配置了JdbcTemplate操作数据库

## 二、整合 Druid 数据源

步骤一：导入 jar 包

```xml
<!--引入druid数据源-->
<!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.1.20</version>
</dependency>
```

步骤二：在 `application.yml`中使用 `type` 来指定数据库源：其他基本配置如上

```yml
spring:
  datasource:
    username: root
    password: 123456
    url: jdbc:mysql://192.168.15.22:3306/jdbc
    driver-class-name: com.mysql.jdbc.Driver
    type: com.alibaba.druid.pool.DruidDataSource

    # 该数据源其他属性的配置
    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
    # 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
    filters: stat,wall,log4j
    maxPoolPreparedStatementPerConnectionSize: 20
    useGlobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
#   schema:
#      - classpath:department.sql
```

上面的「该数据源其他属性的配置」默认是识别不了无法生效的，因为从上面的测试代码，在 `System.out.println(dataSource.getClass());` 断点就会发现：上面配置属性是有值的，但是下面的配置没有应用成功。

![image-20201109105056182](FrameDay06_6%20SpringBoot%E4%B8%8E%E6%95%B0%E6%8D%AE%E8%AE%BF%E9%97%AE.resource/image-20201109105056182.png)

步骤三：需要自定义配置类使得自定义配置生效。

```java
package com.gjxaiou.config;

import com.alibaba.druid.pool.DruidDataSource;
import com.alibaba.druid.support.http.StatViewServlet;
import com.alibaba.druid.support.http.WebStatFilter;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;
import java.util.Arrays;
import java.util.HashMap;
import java.util.Map;

@Configuration
public class DruidConfig {

    // 因为 application.yml 中所有关于数据源的属性配置都是以 spring.datasource 开头，将这些属性与 DruidDataSource 中属性对应。
    @ConfigurationProperties(prefix = "spring.datasource")
    @Bean
    public DataSource druid() {
        return new DruidDataSource();
    }

    // 配置 Druid 的监控
    // 1、配置一个管理后台的 Servlet
    @Bean
    public ServletRegistrationBean statViewServlet() {
        ServletRegistrationBean bean = new ServletRegistrationBean(new StatViewServlet(), "/druid" +
                "/*");
        // 可以设置以下默认属性，属性名称在 ResourceServlet 类中有说明。
        Map<String, String> initParams = new HashMap<>();

        initParams.put("loginUsername", "root");
        initParams.put("loginPassword", "GJXAIOU");
        initParams.put("allow", "");//默认就是允许所有访问
        initParams.put("deny", "localhost");

        bean.setInitParameters(initParams);
        return bean;
    }


    // 2、配置一个 web 监控的 filter
    @Bean
    public FilterRegistrationBean webStatFilter() {
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new WebStatFilter());

        Map<String, String> initParams = new HashMap<>();
        initParams.put("exclusions", "*.js,*.css,/druid/*");
        bean.setInitParameters(initParams);
        bean.setUrlPatterns(Arrays.asList("/*"));
        return bean;
    }
}
```

测试类输出为：

```java
class com.alibaba.druid.pool.DruidDataSource
com.mysql.cj.jdbc.ConnectionImpl@24eb65e3
```

==这里属性注入仍然有问题，并没有注入=

## 三、整合 MyBatis

步骤一：在 pom.xml 导入相关依赖

```xml
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
     # 从命名可以看出，该依赖是 MyBatis 自身提供适配 springboot 的依赖，非 springboot 官方依赖
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.1</version>
</dependency>
```

依赖结构为：

![image-20201112105042115](FrameDay06_6%20SpringBoot%E4%B8%8E%E6%95%B0%E6%8D%AE%E8%AE%BF%E9%97%AE.resource/image-20201112105042115.png)

步骤：

- 配置数据源相关属性（见上一节Druid），在 `application.yml` 中进行属性配置，然后建立配置类。

- 给数据库建表

    这里将建表语句放入 `resources/sql`文件夹下面，然后在 `application.yml` 中配置 SQL 文件位置。完整的配置为：

    ```yaml
    spring:
      datasource:
        #   数据源基本配置
        username: root
        password: 123456
        driver-class-name: com.mysql.jdbc.Driver
        url: jdbc:mysql://192.168.15.22:3306/mybatis
        type: com.alibaba.druid.pool.DruidDataSource
        #   数据源其他配置
        initialSize: 5
        minIdle: 5
        maxActive: 20
        maxWait: 60000
        timeBetweenEvictionRunsMillis: 60000
        minEvictableIdleTimeMillis: 300000
        validationQuery: SELECT 1 FROM DUAL
        testWhileIdle: true
        testOnBorrow: false
        testOnReturn: false
        poolPreparedStatements: true
        #   配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
        filters: stat,wall,log4j
        maxPoolPreparedStatementPerConnectionSize: 20
        useGlobalDataSourceStat: true
        connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
    mybatis:
    
    
    # 创建完成之后注释掉，防止下次项目启动再次创建
    #    schema:
    #      - classpath:sql/department.sql
    #      - classpath:sql/employee.sql
    ```

- 创建两个表对应的 JavaBean

    ```java
    package com.atguigu.springboot.bean;
    
    public class Employee {
        private Integer id;
        private String lastName;
        private Integer gender;
        private String email;
        private Integer dId;
    	// 省略对应的 Getter 和 Setter 方法
    }
    ```

    ```java
    package com.atguigu.springboot.bean;
    
    public class Department {
        private Integer id;
        private String departmentName;
        // 省略对应的 Getter 和 Setter 方法
    }
    ```

### 	4）使用 MyBatis 操纵数据库（注解版）

```java
// 指定这是一个操作数据库的 mapper
@Mapper
public interface DepartmentMapper {

    @Select("select * from department where id=#{id}")
    public Department getDeptById(Integer id);

    @Delete("delete from department where id=#{id}")
    public int deleteDeptById(Integer id);

    @Options(useGeneratedKeys = true,keyProperty = "id")
    @Insert("insert into department(departmentName) values(#{departmentName})")
    public int insertDept(Department department);

    @Update("update department set departmentName=#{departmentName} where id=#{id}")
    public int updateDept(Department department);
}
```

对应的测试 Controller，省略了 service 层

```java
package com.atguigu.springboot.controller;

import com.atguigu.springboot.bean.Department;
import com.atguigu.springboot.bean.Employee;
import com.atguigu.springboot.mapper.DepartmentMapper;
import com.atguigu.springboot.mapper.EmployeeMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class DeptController {

    @Autowired
    DepartmentMapper departmentMapper;

    @Autowired
    EmployeeMapper employeeMapper;


    @GetMapping("/dept/{id}")
    public Department getDepartment(@PathVariable("id") Integer id) {
        return departmentMapper.getDeptById(id);
    }

    @GetMapping("/dept")
    public Department insertDept(Department department) {
        departmentMapper.insertDept(department);
        return department;
    }

    @GetMapping("/emp/{id}")
    public Employee getEmp(@PathVariable("id") Integer id) {
        return employeeMapper.getEmpById(id);
    }
}
```

问题：

因为JavaBean 中的 `departmentName` 和数据库中字段 `departmentName` 名称相同，但是如果将数据库中字段名称改为：`department_name` 则如果使用配置文件的方式，可以在对应的 xml 中打开驼峰命名转换。但是在注解版中使用以下方式。

在 MyBatis 的自动配置类：`MybatisAutoConfiguration` 中 ，通过 `sqlSessionFactory()`方法给容器中创建 SqlSessionFactory 的时候，通过获取属性值得到一个 configuration：`org.apache.ibatis.session.Configuration configuration = this.properties.getConfiguration();`  下面的 97 行中的 ConfigurationCustomizer 就是用于定制的。

因此如果自定义MyBatis的配置规则；给容器中添加一个ConfigurationCustomizer；

```java
package com.atguigu.springboot.config;

import org.apache.ibatis.session.Configuration;
import org.mybatis.spring.boot.autoconfigure.ConfigurationCustomizer;
import org.springframework.context.annotation.Bean;

@org.springframework.context.annotation.Configuration
public class MyBatisConfig {

    @Bean
    public ConfigurationCustomizer configurationCustomizer() {
        return new ConfigurationCustomizer() {
            @Override
            public void customize(Configuration configuration) {
                // 开启驼峰命名法的映射
                configuration.setMapUnderscoreToCamelCase(true);
            }
        };
    }
}
```

补充：如果有很多 XXXmapper 类，可以使用批量扫描包的形式避免在每个 mapper 类上都加上 `@Mapper` 注解 

```java
@MapperScan(value = "com.atguigu.springboot.mapper")
@SpringBootApplication
public class SpringBoot06DataMybatisApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringBoot06DataMybatisApplication.class, args);
	}
}
```

### 5）、配置文件版

步骤一：首先提供一个 mapper 接口

```java
package com.atguigu.springboot.mapper;

import com.atguigu.springboot.bean.Employee;

//@Mapper或者@MapperScan将接口扫描装配到容器中
public interface EmployeeMapper {
    public Employee getEmpById(Integer id);
    public void insertEmp(Employee employee);
}
```

然后需要在 resource 文件夹下面提供一个对应的在 mapper 文件（SQL 映射文件）

```java
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <!--首先进行接口映射-->
<mapper namespace="com.atguigu.springboot.mapper.EmployeeMapper">

    <select id="getEmpById" resultType="com.atguigu.springboot.bean.Employee">
        SELECT * FROM employee WHERE id=#{id}
    </select>

    <insert id="insertEmp">
        INSERT INTO employee(lastName,email,gender,d_id) VALUES (#{lastName},#{email},#{gender},#{dId})
    </insert>
</mapper>
```

需要在 yaml 中进行配置是的 MyBatis 知道上面 mapper 文件的存在。

```yaml
spring:
  datasource:
    #   数据源基本配置
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://192.168.15.22:3306/mybatis
    type: com.alibaba.druid.pool.DruidDataSource
    #   数据源其他配置
    initialSize: 5
    minIdle: 5
    maxActive: 20
    maxWait: 60000
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
    #   配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
    filters: stat,wall,log4j
    maxPoolPreparedStatementPerConnectionSize: 20
    useGlobalDataSourceStat: true
    connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=500
mybatis:
  # 指定全局配置文件位置
  config-location: classpath:mybatis/mybatis-config.xml
  # 指定sql映射文件位置
  mapper-locations: classpath:mybatis/mapper/*.xml

#    schema:
#      - classpath:sql/department.sql
#      - classpath:sql/employee.sql

```

同样要进行驼峰命名法的对应配置，在 MyBatis 的配置文件中进行配置。

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <settings>
        <setting name="mapUnderscoreToCamelCase" value="true"/>
    </settings>
</configuration>
```

测试的 Controller 和上面的 DeptController 一样。

更多使用参照

http://www.mybatis.org/spring-boot-starter/mybatis-spring-boot-autoconfigure/



## 四、整合  SpringData JPA

### 1）、SpringData简介

是 Springboot 底层默认进行数据访问的技术。用于统一数据访问的 API。

Spring Data 项目的目的是为了简化构建基于Spring 框架应用的数据访问技术，包括非关系数据库、Map-Reduce 框架、云数据服务等等；另外也包含对关系数据库的访问支持。

Spring Data 包含多个子项目：

- Spring Data Commons
- Spring Data JPA
- Spring Data KeyValue
- Spring Data LDAP
- Spring Data MongoDB
- Spring Data Gemfire
- Spring Data REST
- Spring Data Redis
- Spring Data for Apache Cassandra
- Spring Data for Apache Solr
- Spring Data Couchbase (community module)
- Spring Data Elasticsearch (community module)
- Spring Data Neo4j (community module)



1、SpringData特点
SpringData为我们提供使用统一的API来对数据访问层进行操作；这主要是Spring Data Commons项目来实现的。Spring Data Commons让我们在使用关系型或者非关系型数据访问技术时都基于Spring提供的统一标准，标准包含了CRUD（创建、获取、更新、删除）、查询、排序和分页的相关操作。
2、统一的Repository接口
`Repository<T, ID extends Serializable>`：统一接口
`RevisionRepository<T, ID extends Serializable, N extends Number & Comparable<N>>`：基于乐观锁机制
`CrudRepository<T, ID extends Serializable>`：基本CRUD操作
`PagingAndSortingRepository<T, ID extends Serializable>`：基本CRUD及分页

![image-20201109141216520](FrameDay06_6%20SpringBoot%E4%B8%8E%E6%95%B0%E6%8D%AE%E8%AE%BF%E9%97%AE.resource/image-20201109141216520.png)

3、提供数据访问模板类xxxTemplate；
如：MongoTemplate、RedisTemplate等
4、JPA与Spring Data
1）、JpaRepository基本功能
编写接口继承JpaRepository既有crud及分页等基本功能
2）、定义符合规范的方法命名
在接口中只需要声明符合规范的方法，即拥有对应的功能

![image-20201109141441558](FrameDay06_6%20SpringBoot%E4%B8%8E%E6%95%B0%E6%8D%AE%E8%AE%BF%E9%97%AE.resource/image-20201109141441558.png)3）、@Query自定义查询，定制查询SQL
4）、Specifications查询（Spring Data JPA支持JPA2.0的Criteria查询）

![](FrameDay06_6%20SpringBoot%E4%B8%8E%E6%95%B0%E6%8D%AE%E8%AE%BF%E9%97%AE.resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20180306105412.png)

### 2）整合 SpringData JPA

JPA（Java Persistence API） Java 持久化 API，是一种规范。

JPA 也是基于 ORM（Object Relational Mapping）对象关系映射

步骤一：使用 `application.yaml` 进行数据源配置

```yaml
spring:
  datasource:
    url: jdbc:mysql://192.168.15.22/jpa
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver
  jpa:
    hibernate:
      # 启动时候更新或者创建数据表结构
      ddl-auto: update
    # 控制台显示SQL
    show-sql: true
```

1）、编写一个实体类（bean）和数据表进行映射，并且配置好映射关系；

```java
package com.atguigu.springboot.entity;

import javax.persistence.*;

//使用JPA注解配置映射关系
@Entity //告诉JPA这是一个实体类（和数据表映射的类）
@Table(name = "tbl_user") //@Table来指定和哪个数据表对应;如果省略默认表名就是user；
public class User {

    @Id //这是一个主键
    @GeneratedValue(strategy = GenerationType.IDENTITY)//自增主键
    private Integer id;

    @Column(name = "last_name", length = 50) //这是和数据表对应的一个列
    private String lastName;
    @Column //省略默认列名就是属性名
    private String email;
	// 省略 Getter 和 Setter 方法
}
```

2）、编写一个Dao接口来操作实体类对应的数据表（Repository）

```java
//继承JpaRepository来完成对数据库的操作
public interface UserRepository extends JpaRepository<User,Integer> {
}

```

3）、基本的配置JpaProperties

```yaml
spring:  
 jpa:
    hibernate:
#     更新或者创建数据表结构
      ddl-auto: update
#    控制台显示SQL
    show-sql: true
```

测试的 Controller

```java
package com.atguigu.springboot.controller;

import com.atguigu.springboot.entity.User;
import com.atguigu.springboot.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class UserController {

    @Autowired
    UserRepository userRepository;

    @GetMapping("/user/{id}")
    public User getUser(@PathVariable("id") Integer id) {
        User user = userRepository.findOne(id);
        return user;
    }

    @GetMapping("/user")
    public User insertUser(User user) {
        User save = userRepository.save(user);
        return save;
    }
}
```

