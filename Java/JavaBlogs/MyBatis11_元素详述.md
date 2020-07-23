---
tag:
- 未看
flag: yellow
---
# Mybatis配置之元素详述

2017年05月15日 10:12:06 [DreamMakers](https://me.csdn.net/andamajing) 阅读数 9651

[](http://creativecommons.org/licenses/by-sa/4.0/)版权声明：本文为博主原创文章，遵循[ CC 4.0 BY-SA ](http://creativecommons.org/licenses/by-sa/4.0/)版权协议，转载请附上原文出处链接和本声明。

本文链接：[https://blog.csdn.net/andamajing/article/details/72085696](https://blog.csdn.net/andamajing/article/details/72085696) 

在前面的若干篇文章中，我们已经对mybatis中主要的配置元素做了讲述，还剩下一个比较重要的元素，那就是<mappers>元素。

这个元素是干嘛用的呢？<mappers>用来在mybatis初始化的时候，告诉mybatis需要引入哪些Mapper映射文件。那什么又是Mapper映射文件呢？它是Java实体类与数据库对象之间的桥梁。在实际的使用过程中，一般一个Mapper文件对应一个数据库操作Dao接口。

在mybatis中，mappers必须配置！那么怎么配置呢？我们先从源码上看下这个节点是怎么解析的。

```
private void mapperElement(XNode parent) throws Exception {    if (parent != null) {      for (XNode child : parent.getChildren()) {        if ("package".equals(child.getName())) {          String mapperPackage = child.getStringAttribute("name");          configuration.addMappers(mapperPackage);        } else {          String resource = child.getStringAttribute("resource");          String url = child.getStringAttribute("url");          String mapperClass = child.getStringAttribute("class");          if (resource != null && url == null && mapperClass == null) {            ErrorContext.instance().resource(resource);            InputStream inputStream = Resources.getResourceAsStream(resource);            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());            mapperParser.parse();          } else if (resource == null && url != null && mapperClass == null) {            ErrorContext.instance().resource(url);            InputStream inputStream = Resources.getUrlAsStream(url);            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, url, configuration.getSqlFragments());            mapperParser.parse();          } else if (resource == null && url == null && mapperClass != null) {            Class<?> mapperInterface = Resources.classForName(mapperClass);            configuration.addMapper(mapperInterface);          } else {            throw new BuilderException("A mapper element may only specify a url, resource or class, but not more than one.");          }        }      }    }  }
```

 上面的代码便是Mybatis中<mappers>节点的解析入口了，我们来简单看下。

从代码看，我们知道分为两大类，一种是配置<package>子元素，另一种是配置<mapper>元素（从http://mybatis.org/dtd/mybatis-3-config.dtd文件限制的）。如下截图所示，

![](https://img-blog.csdn.net/20170515090624015?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

这两种方式都是怎么配置呢？<package>配置很简单，只需要配置一个name属性用于设置包名。<mapper>的配置则比较多样化，支持三种属性方式设置，分别为resource、url和class三个属性。具体的配置如下所示：

```
<configuration>    ......    <mappers>      <!-- 第一种方式：通过resource指定 -->    <mapper resource="com/dy/dao/userDao.xml"/>         <!-- 第二种方式， 通过class指定接口，进而将接口与对应的xml文件形成映射关系             不过，使用这种方式必须保证 接口与mapper文件同名(不区分大小写)，              我这儿接口是UserDao,那么意味着mapper文件为UserDao.xml      <mapper class="com.dy.dao.UserDao"/>      -->            <!-- 第三种方式，直接指定包，自动扫描，与方法二同理       <package name="com.dy.dao"/>      -->      <!-- 第四种方式：通过url指定mapper文件位置      <mapper url="file://........"/>       -->  </mappers>    ......  </configuration>
```

 关于<mappers>元素的配置讲解大概就这么多，其实核心的不在这里怎么配置，而是每个mapper映射文件才是重点。从下篇文章开始，我们将对mapper映射文件包含的元素进行详细说明，敬请期待。