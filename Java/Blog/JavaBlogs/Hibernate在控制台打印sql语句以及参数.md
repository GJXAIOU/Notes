# Hibernate在控制台打印sql语句以及参数

[原文地址](https://blog.csdn.net/Randy_Wang_/article/details/79460306)

最近在工作中使用hibernate，遇到了sql语句错误，为了查看具体的错误，整理了hibernate在控制台及日志打印sql语句及参数的方法

# 环境说明

 IntelliJ IDEA 2017.3.4 版本；SpringBoot 2.0.0.RELEASE；hibernate用的是JPA自带。

# 打印sql语句到控制台

 首先，我使用的是application.properties配置文件，使用yml也可以达到同样的效果。
 
 在网上查这个问题查了好久，基本上都是xml配置，在此不多说；
 
 正确的properties配置项应该如下图所示：

**在jpa下一级不直接是hibernate，而是properties。**
```
spring.jpa.properties.hibernate.show_sql=true          //控制台是否打印
spring.jpa.properties.hibernate.format_sql=true        //格式化sql语句
spring.jpa.properties.hibernate.use_sql_comments=true  //指出是什么操作生成了该语句
```
 此时，在控制台看到的现象：Hibernate: 
 
 ![](https://img-blog.csdn.net/20180306170047795?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUmFuZHlfV2FuZ18=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 
 可以看到，控制台打印了一条经过格式化之后的sql语句，并标明了这条语句是在Hibernate插入TaskWebSiteRev这个对象到数据库的时候生成的。

**打印sql语句中的参数值**

 经过上面的步骤，我们已经可以在控制台打印出格式化之后的sql语句，但是大多数情况下，我们还需要具体的sql参数值，这个时候我们就需要配置 **日志配置文件**。

博主使用的是slf4j的日志，配置文件用的是logback.xml，配置方式如下：
```language
<logger name="org.hibernate.SQL" level="DEBUG"/>
 <logger name="org.hibernate.engine.QueryParameters" level="DEBUG"/>
 <logger name="org.hibernate.engine.query.HQLQueryPlan" level="DEBUG"/>
```
 直接把这三个配置丢到xml的根节点下就可以~来看一下效果：
 
 ![](https://img-blog.csdn.net/20180306170921800?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUmFuZHlfV2FuZ18=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 
 可以看到控制台依次输出了sql参数，并且将这些参数在数据库中的类型也一并输出了。

**打印sql语句到日志**

 在上述步骤的基础上，在logback.xml中增加两项配置：
```
 <logger name="org.hibernate.type.descriptor.sql.BasicBinder" level="TRACE"/>
<logger name="org.hibernate.type.descriptor.sql.BasicExtractor" level="TRACE"/>
```
这样就会在日志中将sql语句打印出来，在无法查看控制台的情况下，就可以使用这个方法啦。效果如图所示：

 ![](https://img-blog.csdn.net/20180306171908839?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUmFuZHlfV2FuZ18=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**注：**

 以上提到的配置如果全部配置，在控制台会有冗余的打印信息：
 
 ![](https://img-blog.csdn.net/20180306172131370?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvUmFuZHlfV2FuZ18=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 建议根据需要，只配置打印到控制台或打印到日志其中一种。

**以上就是hibernate在控制台输出sql语句及其参数值的方法，有不对的地方请指正，也欢迎大家分享自己知道的更方便的方法~**