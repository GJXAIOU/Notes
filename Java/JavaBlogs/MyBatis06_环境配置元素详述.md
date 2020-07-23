# Mybatis配置之配置元素详述

[原文链接](https://blog.csdn.net/andamajing/article/details/71616712)

在这篇文章中，我们接着前文继续往下看其他的配置元素，今天的主角就是我们的**<environments>元素，该元素用于对我们需要访问的数据库配置进行设置**，我们先来看一下配置：

```
<environments default="development">
		<environment id="development">
			<!-- 使用jdbc事务管理 -->
			<transactionManager type="JDBC" />
			<!-- 数据库连接池 -->
			<dataSource type="POOLED">
				<property name="driver" value="${driver}" />
				<property name="url" value="${url}" />
				<property name="username" value="${username}" />
				<property name="password" value="${password}" />
			</dataSource>
		</environment>
</environments>
```

 从上面看，我们知道<environments>下面可以配置多个<environment>元素节点，而**每个<environment>节点我们可以配置两个东西，一个是事务管理器配置<transactionManager>，另一个是数据源配置<dataSource>**。

我们先从源码开始看起，看看这块是怎么解析的，然后再具体看里面都要配置什么哪些参数。

还是从解析的入口开始看起：

![](https://img-blog.csdn.net/20170511123828382?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

进入方法内部：

![](https://img-blog.csdn.net/20170511124034429?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

从代码看，就是**首先获取<environments>标签元素的default属性，这个属性作用就是指定当前情况下使用哪个数据库配置，也就是使用哪个<environment>节点的配置，default的值就是配置的<environment>标签元素的id值**。

正如上面代码中isSpecifiedEnvironment(id)方法一样，在遍历所有<environment>的时候一次判断相应的id是否是default设置的值，如果是，则使用当前<environment>元素进行数据库连接的初始化。

isSpecifiedEnvironment方法如下所示：

![](https://img-blog.csdn.net/20170511124525058?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

紧接着，下面的代码就是用设置的事务管理器和数据源构造相应的对象了。

```
TransactionFactory txFactory = transactionManagerElement(child.evalNode("transactionManager"));
DataSourceFactory dsFactory = dataSourceElement(child.evalNode("dataSource"));
DataSource dataSource = dsFactory.getDataSource();
Environment.Builder environmentBuilder = new Environment.Builder(id)
              .transactionFactory(txFactory)
              .dataSource(dataSource);
configuration.setEnvironment(environmentBuilder.build());
```


## 事务管理器
 我们首先从事务管理器的解析开始，进入到transactionManagerElement()方法内：

```
private TransactionFactory transactionManagerElement(XNode context) throws Exception {
    if (context != null) {
      String type = context.getStringAttribute("type");
      Properties props = context.getChildrenAsProperties();
      TransactionFactory factory = (TransactionFactory) resolveClass(type).newInstance();
      factory.setProperties(props);
      return factory;
    }
    throw new BuilderException("Environment declaration requires a TransactionFactory.");
  }
```

 我们可以看到，这里其实是**根据<transactionManager>这个元素的type属性来找相应的事务管理器的**。

在**Mybatis 里面支持两种配置：JDBC 和 MANAGED**。这里根据 type 的设置值来返回相应的事务管理器。我们看下，在 Mybatis 的 Configuration 类中，已经将这两种配置及对应的事务管理器做了某种关联，如下所示：

```
public Configuration() {
    typeAliasRegistry.registerAlias("JDBC", JdbcTransactionFactory.class);
    typeAliasRegistry.registerAlias("MANAGED", ManagedTransactionFactory.class);
 
    typeAliasRegistry.registerAlias("JNDI", JndiDataSourceFactory.class);
    typeAliasRegistry.registerAlias("POOLED", PooledDataSourceFactory.class);
    typeAliasRegistry.registerAlias("UNPOOLED", UnpooledDataSourceFactory.class);
 
    typeAliasRegistry.registerAlias("PERPETUAL", PerpetualCache.class);
    typeAliasRegistry.registerAlias("FIFO", FifoCache.class);
    typeAliasRegistry.registerAlias("LRU", LruCache.class);
    typeAliasRegistry.registerAlias("SOFT", SoftCache.class);
    typeAliasRegistry.registerAlias("WEAK", WeakCache.class);
 
    typeAliasRegistry.registerAlias("DB_VENDOR", VendorDatabaseIdProvider.class);
 
    typeAliasRegistry.registerAlias("XML", XMLLanguageDriver.class);
    typeAliasRegistry.registerAlias("RAW", RawLanguageDriver.class);
 
    typeAliasRegistry.registerAlias("SLF4J", Slf4jImpl.class);
    typeAliasRegistry.registerAlias("COMMONS_LOGGING", JakartaCommonsLoggingImpl.class);
    typeAliasRegistry.registerAlias("LOG4J", Log4jImpl.class);
    typeAliasRegistry.registerAlias("LOG4J2", Log4j2Impl.class);
    typeAliasRegistry.registerAlias("JDK_LOGGING", Jdk14LoggingImpl.class);
    typeAliasRegistry.registerAlias("STDOUT_LOGGING", StdOutImpl.class);
    typeAliasRegistry.registerAlias("NO_LOGGING", NoLoggingImpl.class);
 
    typeAliasRegistry.registerAlias("CGLIB", CglibProxyFactory.class);
    typeAliasRegistry.registerAlias("JAVASSIST", JavassistProxyFactory.class);
 
    languageRegistry.setDefaultDriverClass(XMLLanguageDriver.class);
    languageRegistry.register(RawLanguageDriver.class);
  }
```

 这两种事务管理器的区别：

JDBC：这个配置就是直接使用了 JDBC 的提交和回滚设置，它依赖于从数据源得到的连接来管理事务作用域。

MANAGED：这个配置几乎没做什么。它从来不提交或回滚一个连接，而是让容器来管理事务的整个生命周期（比如 JEE 应用服务器的上下文）。 默认情况下它会关闭连接，然而一些容器并不希望这样，因此需要将 closeConnection 属性设置为 false 来阻止它默认的关闭行为。例如:

```
<transactionManager type="MANAGED">
  <property name="closeConnection" value="false"/>
</transactionManager>
```

 备注：如果你正在使用 Spring + MyBatis，则没有必要配置事务管理器， 因为 **Spring 模块会使用自带的管理器来覆盖前面的配置**。


## 数据源配置

说完了事务管理器，紧接着，我们来看看数据源配置。

**dataSource 元素使用标准的 JDBC 数据源接口来配置 JDBC 连接对象的资源**。Mybatis支持三种内建的数据源类型，分别是UNPOOLED、POOLED和JNDI，即我们在配置<dataSource>元素的type属性时，我们可以直接支持设置这三个值。

下面分别对这三种类型做一个简单的说明：

- （1）UNPOOLED
这个数据源的实现只是每次被请求时打开和关闭连接。虽然一点慢，它对在及时可用连接方面没有性能要求的简单应用程序是一个很好的选择。 不同的数据库在这方面表现也是不一样的，所以对某些数据库来说使用连接池并不重要，这个配置也是理想的。
  - UNPOOLED 类型的数据源仅仅需要配置以下 5 种属性：
    * driver ： 这是 JDBC 驱动的 Java 类的完全限定名（并不是JDBC驱动中可能包含的数据源类）。
    * url ：这是数据库的 JDBC URL 地址。
    * username ： 登录数据库的用户名。
    * password ：登录数据库的密码。
    * defaultTransactionIsolationLevel ： 默认的连接事务隔离级别。 

作为可选项，你也可以传递属性给数据库驱动。要这样做，属性的前缀为“driver.”，例如：`driver.encoding=UTF8`

这将通过DriverManager.getConnection(url,driverProperties)方法传递值为 UTF8 的 encoding 属性给数据库驱动。

- （2）POOLED  
这种数据源的实现利用“池”的概念将 JDBC 连接对象组织起来，避免了创建新的连接实例时所必需的初始化和认证时间。 这是一种使得并发 Web 应用快速响应请求的流行处理方式。
  - 除了上述提到 UNPOOLED 下的属性外，会有更多属性用来配置 POOLED 的数据源： 
    * poolMaximumActiveConnections ： 在任意时间可以存在的活动（也就是正在使用）连接数量，默认值：10 
    * poolMaximumIdleConnections ：任意时间可能存在的空闲连接数。 
    * poolMaximumCheckoutTime ：在被强制返回之前，池中连接被检出（checked out）时间，默认值：20000 毫秒（即 20 秒） 
    * poolTimeToWait ：这是一个底层设置，如果获取连接花费的相当长的时间，它会给连接池打印状态日志并重新尝试获取一个连接（避免在误配置的情况下一直安静的失败），默认值：20000 毫秒（即 20 秒）。 
    * poolPingQuery ： 发送到数据库的侦测查询，用来检验连接是否处在正常工作秩序中并准备接受请求。默认是“NO PING QUERY SET”，这会导致多数数据库驱动失败时带有一个恰当的错误消息。 
    * poolPingEnabled ： 是否启用侦测查询。若开启，也必须使用一个可执行的 SQL 语句设置 poolPingQuery 属性（最好是一个非常快的 SQL），默认值：false。
    -  poolPingConnectionsNotUsedFor ： 配置 poolPingQuery 的使用频度。这可以被设置成匹配具体的数据库连接超时时间，来避免不必要的侦测，默认值：0（即所有连接每一时刻都被侦测 — 当然仅当 poolPingEnabled 为 true 时适用）。 

- （3）JNDI
这个数据源的实现是为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的引用。
  - 这种数据源配置只需要两个属性：
    - `initial_context` ： 这个属性用来在 InitialContext 中寻找上下文（即，`initialContext.lookup(initial_context)）`。这是个可选属性，如果忽略，那么 data_source 属性将会直接从 InitialContext 中寻找。
    - `data_source` ： 这是引用数据源实例位置的上下文的路径。提供了 initial_context 配置时会在其返回的上下文中进行查找，没有提供时则直接在 InitialContext 中查找。

和其他数据源配置类似，可以通过添加前缀“env.”直接把属性传递给初始上下文。比如：`env.encoding=UTF8`
这就会在初始上下文（InitialContext）实例化时往它的构造方法传递值为 UTF8 的 encoding 属性。

至此，关于<environments>元素的相关配置使用便介绍完毕了。