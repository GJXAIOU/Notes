# mybatis 详解（四）------properties以及别名定义
[原文链接](https://www.cnblogs.com/ysocean/p/7287972.html)

@toc

　　上一篇博客我们介绍了mybatis的增删改查入门实例，我们发现在 mybatis-configuration.xml 的配置文件中，对数据库的配置都是硬编码在这个xml文件中，如下图，那么我们如何改进这个写法呢？

　　![](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170804220404022-1871062647.png)
【注：硬编码和软编码的区别】
计算机科学中，只有硬编码（hardcode），以及非硬编码，有人也成为“软编码”。
    硬编码和软编码的区别是：软编码可以在运行时确定，修改；而硬编码是不能够改变的。所有的硬编码和软编码的区别都可以有这个意思扩展开。
    在计算机程序或文本编辑中，硬编码是指将可变变量用一个固定值来代替的方法。用这种方法编译后，如果以后需要更改此变量就非常困难了。大部分程序语言里，可以将一个固定数值定义为一个标记，然后用这个特殊标记来取代变量名称。当标记名称改变时，变量名不变，这样，当重新编译整个程序时，所有变量都不再是固定值，这样就更容易的实现了改变变量的目的。
    尽管通过编辑器的查找替换功能也能实现整个变量名称的替换，但也很有可能出现多换或者少换的情况，而在计算机 程序中，任何小错误的出现都是不可饶恕的。最好的方法是单独为变量名划分空间，来实现这种变化，就如同前面说的那样，将需要改变的变量名暂时用一个定义好 的标记名称来代替就是一种很好的方法。通常情况下，都应该避免使用硬编码方法。　 　　
    java小例子： `int a=2,b=2;` 　　
    硬编码：`if(a==2) return false;` 　　
    非硬编码 `if(a==b) return true;` 　　（就是把数值写成常数而不是变量 ）
    一个简单的版本：如求圆的面积 的问题 PI（3.14） 　　
    那么`3.14*r*r` 就是硬编码，而 `PI*r*r` 就不是硬编码。

### 1、我们将 数据库的配置语句写在 db.properties 文件中
```db_properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/ssm
jdbc.username=root
jdbc.password=root
```

### 2、在  mybatis-configuration.xml 中加载db.properties文件并读取

```MyBatis_xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <!-- 加载数据库属性文件 -->
  <properties resource="db.properties">
  </properties>

 <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <!--dataSource 元素使用标准的 JDBC 数据源接口来配置 JDBC 连接对象源  -->
      <dataSource type="POOLED">
        <property name="driver" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
      </dataSource>
    </environment>
  </environments>
</configuration>
```
　　如果数据库有变化，我们就可以通过修改 db.properties 文件来修改，而不用去修改 mybatis-configuration.xml 文件

注意：我们也可以在<properties></properties>中手动增加属性
```MyBatis_xml
<!-- 加载数据库属性文件 -->
<properties resource="db.properties">
    <property name="username" value="aaa"/>
</properties>
```
　　那么这个时候是读取的username 是以 db.properties 文件中的 root 为准，还是以自己配置的 aaa 为准呢?

我们先看一段 properties 文件加载的源码
```db_properties
private void propertiesElement(XNode context) throws Exception {
  if (context != null) {
    /**
     *  解析properties 属性中指定的属性。
     */
    Properties defaults = context.getChildrenAsProperties();
    String resource = context.getStringAttribute("resource"); //resource 制定的属性路径
    String url = context.getStringAttribute("url"); //url制定的属性路径
    if (resource != null && url != null) {
      throw new BuilderException("The properties element cannot specify both a URL and a resource based property file reference.  Please specify one or the other.");
    }
    /**
     * 根据 properties 元素中的 resource 属性读取类路径下属性文件，并覆盖properties 属性中指定的同名属性。
     */
    if (resource != null) {
      defaults.putAll(Resources.getResourceAsProperties(resource));
    } else if (url != null) {
      /**
       * 根据properties元素中的url属性指定的路径读取属性文件，并覆盖properties 属性中指定的同名属性。
       */
      defaults.putAll(Resources.getUrlAsProperties(url));
    }
    /**
     *  获取方法参数传递的properties
     *  创建XMLConfigBuilder实例时，this.configuration.setVariables(props);
     */
    Properties vars = configuration.getVariables();
    if (vars != null) {
      defaults.putAll(vars);
    }
    parser.setVariables(defaults);
    configuration.setVariables(defaults);
  }
}
```

### 通过源码我们可以分析读取优先级：

1、在 properties 内部**自定义的属性值第一个被读取**

2、**然后读取 resource 路径表示文件中的属性，如果有它会覆盖已经读取的属性**；如果 resource 路径不存在，那么读取 url 表示路径文件中的属性，如果有它会覆盖第一步读取的属性值

3、最后读取 parameterType 传递的属性值，它会覆盖已读取的同名的属性

前面两步好理解，第三步我们可以举个例子来看：

我们在 userMapper.xml 文件中进行模糊查询
```UserMapper_xml
<select id="selectLikeUserName" resultType="com.ys.po.User" parameterType="String">
    select * from user where username like '%${jdbc.username}%'
    <!-- select * from user where username like #{username} -->
</select>
```

　这个时候你会发现无论你后台传给这个查询语句什么参数，都是 select * from user where username like '%root%'

# mybatis 的别名配置　　

　　在 userMapper.xml 文件中，我们可以看到resultType 和 parameterType 需要指定，这这个值往往都是全路径，不方便开发，那么我们就可以对这些属性进行一些别名设置

![MyBatis别名设置]($resource/MyBatis%E5%88%AB%E5%90%8D%E8%AE%BE%E7%BD%AE.png)

## 1、mybatis 默认支持的别名

　　![](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170804230414350-900097506.png)

　　![](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170804230557694-737797674.png)



## 2、自定义别名　　

### 一、定义单个别名

　首先在全局配置文件 mybatis-configuration.xml 文件中添加如下配置：是在<configuration>标签下
```MyBatis_xml
<!-- 定义别名 -->
<typeAliases>
    <typeAlias type="com.ys.po.User" alias="user"/>
</typeAliases>
```

第二步通过 user 引用

![引用]($resource/%E5%BC%95%E7%94%A8.png)

### 二、批量定义别名

在全局配置文件 mybatis-configuration.xml 文件中添加如下配置：是在<configuration>标签下
```MyBatis_xml
<!-- 定义别名 -->
<typeAliases>
    <!-- mybatis自动扫描包中的po类，自动定义别名，别名是类名(首字母大写或小写都可以,一般用小写) -->
    <package name="com.ys.po"/>
</typeAliases>
```

引用的时候类名的首字母大小写都可以