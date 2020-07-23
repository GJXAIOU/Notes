---
tag:
- 未看
flag: yellow
---

# Mybatis中的数据源与连接池详解

[原文链接](https://blog.csdn.net/andamajing/article/details/71715846)

在前面的文章Mybatis配置之 **<environments> 配置元素详述中我们已经知道里面可以配置两个元素，一个是数据源及连接池的配置，一个是事务管理器的配置**。在上篇文章中我们只是简单的描述了一下，从这篇文章开始，我们将分两篇博文，分别对这两个问题进行详细说明。

这篇文章我们先来了解一下数据源及连接池的配置。

（1）Mybatis中支持的数据源  

在上篇文章中，我们知道Mybatis中支持三种形式数据源的配置，分别为：UNPOOLED、POOLED和JNDI，如下红色区域所示： 

![](https://img-blog.csdn.net/20170512124019425?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

在Mybatis内部定义了一个接口DataSourceFactory，而支持的三种形式都需要实现这个接口。DataSourceFactory接口定义如下：

```
package org.apache.ibatis.datasource; import java.util.Properties;import javax.sql.DataSource; /** * @author Clinton Begin */public interface DataSourceFactory {   void setProperties(Properties props);   DataSource getDataSource(); }
```

 与UNPOOLED、POOLED和JNDI相对应的，在mybatis内部定义实现了DataSourceFactory接口的三个类，分别为UnpooledDataSourceFactory、PooledDataSourceFactory和JndiDataSourceFactory。

具体结构如下所示：

![](https://img-blog.csdn.net/20170512125841268?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) 

与这些数据源工厂类相对应的也定义了相应的数据源对象，其中UnpooledDataSourceFactory和PooledDataSourceFactory工厂返回的分别是UnpooledDataSource和PooledDataSource，而JndiDataSourceFactory则是根据配置返回相应的数据源。  

![](https://img-blog.csdn.net/20170512125100546?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

（2）mybatis中数据源的创建过程

首先从配置文件开始看起：

```
<!-- 数据库连接池 -->			<dataSource type="POOLED">				<property name="driver" value="${driver}" />				<property name="url" value="${url}" />				<property name="username" value="${username}" />				<property name="password" value="${password}" />			</dataSource>
```

（a）在mybatis初始化的时候，在解析到<dataSource>节点时，会根据相应的type类型设置来创建相应的数据源工厂类实例，如下所示：

```
DataSourceFactory dsFactory = dataSourceElement(child.evalNode("dataSource"));
```

```
private DataSourceFactory dataSourceElement(XNode context) throws Exception {    if (context != null) {      String type = context.getStringAttribute("type");      Properties props = context.getChildrenAsProperties();      DataSourceFactory factory = (DataSourceFactory) resolveClass(type).newInstance();      factory.setProperties(props);      return factory;    }    throw new BuilderException("Environment declaration requires a DataSourceFactory.");  }
```

在上面代码里，根据type类型去寻找相应的数据源工厂类并实例化一个。具体每一个配置对应什么类，在Configuration类中已经进行了声明，如下所示： 

```
typeAliasRegistry.registerAlias("JNDI", JndiDataSourceFactory.class);typeAliasRegistry.registerAlias("POOLED", PooledDataSourceFactory.class);typeAliasRegistry.registerAlias("UNPOOLED", UnpooledDataSourceFactory.class);
```

（b）之后，从数据源工厂类实例中通过getDataSource()方法获取一个DataSource对象；

（c）MyBatis创建了DataSource实例后，会将其放到Configuration对象内的Environment对象中， 供以后使用。如下所示：

```
DataSourceFactory dsFactory = dataSourceElement(child.evalNode("dataSource"));          DataSource dataSource = dsFactory.getDataSource();          Environment.Builder environmentBuilder = new Environment.Builder(id)              .transactionFactory(txFactory)              .dataSource(dataSource);          configuration.setEnvironment(environmentBuilder.build());
```

 （3）数据源DataSource对象什么时候创建数据库连接

当我们需要创建SqlSession对象并需要执行SQL语句时，这时候MyBatis才会去调用dataSource对象来创建java.sql.Connection对象。也就是说，java.sql.Connection对象的创建一直延迟到执行SQL语句的时候。  

```
public void testFindUserById(){		SqlSession sqlSession = getSessionFactory().openSession(true);  		UserDao userMapper = sqlSession.getMapper(UserDao.class);   		User user = userMapper.findUserById(10);  		System.out.println("记录为："+user);	}
```

 对于上面这段代码，我们通过调试会发现，在前两句的时候其实是没有创建数据库连接的，而是在执行userMapper.findUserById()方法的时候才触发了数据库连接的创建。

（4）非池化的数据源UnpooledDataSource 

我们先直接从代码入手：

```
@Override  public Connection getConnection() throws SQLException {    return doGetConnection(username, password);  }
```

```
private Connection doGetConnection(String username, String password) throws SQLException {    Properties props = new Properties();    if (driverProperties != null) {      props.putAll(driverProperties);    }    if (username != null) {      props.setProperty("user", username);    }    if (password != null) {      props.setProperty("password", password);    }    return doGetConnection(props);  }
```

```
private Connection doGetConnection(Properties properties) throws SQLException {    initializeDriver();    Connection connection = DriverManager.getConnection(url, properties);    configureConnection(connection);    return connection;  }
```

 从上面的代码可以知道UnpooledDataSource创建数据库连接的主要流程，具体时序图如下所示：

（a）调用initializeDriver()方法进行驱动的初始化；

判断driver驱动是否已经加载到内存中，如果还没有加载，则会动态地加载driver类，并实例化一个Driver对象，使用DriverManager.registerDriver()方法将其注册到内存中，以供后续使用。 

（b）调用DriverManager.getConnection()获取数据库连接；

（c）对数据库连接进行一些设置，并返回数据库连接Connection; 

设置数据库连接是否自动提交，设置事务级别等。

![](https://img-blog.csdn.net/20170512164355919?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) 

有人可能会有疑问，这里的username和password是什么传递给数据源的呢？

这个问题其实上面已经提到过了，在mybatis初始化的时候，就已经解析了<dataSource>元素，并将其下相关的<property>配置作为数据源的配置初始化进去了。也就是下面这段逻辑：

```
Properties props = context.getChildrenAsProperties();      DataSourceFactory factory = (DataSourceFactory) resolveClass(type).newInstance();      factory.setProperties(props);
```

 至此，对于UnpooledDataSource数据源算是有比较清楚的了解了。下面我们看看带连接池的PooledDataSource

（5）带连接池的PooledDataSource 

为什么要使用带连接池的数据源呢，最根本的原因还是因为每次创建连接开销比较大，频繁的创建和关闭数据库连接将会严重的影响性能。因此，常用的做法是维护一个数据库连接池，每次使用完之后并不是直接关闭数据库连接，再后面如果需要创建数据库连接的时候直接拿之前释放的数据库连接使用，避免频繁创建和关闭数据库连接造成的开销。

在mybatis中，定义了一个数据库连接池状态的类PoolState，在这个类里，除维护了数据源实例，还维护着数据库连接。数据库连接被分成了两种状态类型并存放在两个列表中：idleConnections和activeConnections。

idleConnections:

空闲(idle)状态PooledConnection对象被放置到此集合中，表示当前闲置的没有被使用的PooledConnection集合，调用PooledDataSource的getConnection()方法时，会优先从此集合中取PooledConnection对象。当用完一个java.sql.Connection对象时，MyBatis会将其包裹成PooledConnection对象放到此集合中。

activeConnections:活动(active)状态的PooledConnection对象被放置到名为activeConnections的ArrayList中，表示当前正在被使用的PooledConnection集合，调用PooledDataSource的getConnection()方法时，会优先从idleConnections集合中取PooledConnection对象,如果没有，则看此集合是否已满，如果未满，PooledDataSource会创建出一个PooledConnection，添加到此集合中，并返回。

![](https://img-blog.csdn.net/20170514134254024?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) 

下面我们看看怎么从连接池中获取一个数据库连接，还是从PooledDataSource类开始看起。

```
@Override  public Connection getConnection() throws SQLException {    return popConnection(dataSource.getUsername(), dataSource.getPassword()).getProxyConnection();  }   @Override  public Connection getConnection(String username, String password) throws SQLException {    return popConnection(username, password).getProxyConnection();  }
```

这里都是调用了popConnection()方法，然后返回其代理对象。

```
private PooledConnection popConnection(String username, String password) throws SQLException {    boolean countedWait = false;    PooledConnection conn = null;    long t = System.currentTimeMillis();    int localBadConnectionCount = 0;     while (conn == null) {      synchronized (state) {        if (!state.idleConnections.isEmpty()) {          // Pool has available connection          conn = state.idleConnections.remove(0);          if (log.isDebugEnabled()) {            log.debug("Checked out connection " + conn.getRealHashCode() + " from pool.");          }        } else {          // Pool does not have available connection          if (state.activeConnections.size() < poolMaximumActiveConnections) {            // Can create new connection            conn = new PooledConnection(dataSource.getConnection(), this);            if (log.isDebugEnabled()) {              log.debug("Created connection " + conn.getRealHashCode() + ".");            }          } else {            // Cannot create new connection            PooledConnection oldestActiveConnection = state.activeConnections.get(0);            long longestCheckoutTime = oldestActiveConnection.getCheckoutTime();            if (longestCheckoutTime > poolMaximumCheckoutTime) {              // Can claim overdue connection              state.claimedOverdueConnectionCount++;              state.accumulatedCheckoutTimeOfOverdueConnections += longestCheckoutTime;              state.accumulatedCheckoutTime += longestCheckoutTime;              state.activeConnections.remove(oldestActiveConnection);              if (!oldestActiveConnection.getRealConnection().getAutoCommit()) {                try {                  oldestActiveConnection.getRealConnection().rollback();                } catch (SQLException e) {                  log.debug("Bad connection. Could not roll back");                }                }              conn = new PooledConnection(oldestActiveConnection.getRealConnection(), this);              oldestActiveConnection.invalidate();              if (log.isDebugEnabled()) {                log.debug("Claimed overdue connection " + conn.getRealHashCode() + ".");              }            } else {              // Must wait              try {                if (!countedWait) {                  state.hadToWaitCount++;                  countedWait = true;                }                if (log.isDebugEnabled()) {                  log.debug("Waiting as long as " + poolTimeToWait + " milliseconds for connection.");                }                long wt = System.currentTimeMillis();                state.wait(poolTimeToWait);                state.accumulatedWaitTime += System.currentTimeMillis() - wt;              } catch (InterruptedException e) {                break;              }            }          }        }        if (conn != null) {          if (conn.isValid()) {            if (!conn.getRealConnection().getAutoCommit()) {              conn.getRealConnection().rollback();            }            conn.setConnectionTypeCode(assembleConnectionTypeCode(dataSource.getUrl(), username, password));            conn.setCheckoutTimestamp(System.currentTimeMillis());            conn.setLastUsedTimestamp(System.currentTimeMillis());            state.activeConnections.add(conn);            state.requestCount++;            state.accumulatedRequestTime += System.currentTimeMillis() - t;          } else {            if (log.isDebugEnabled()) {              log.debug("A bad connection (" + conn.getRealHashCode() + ") was returned from the pool, getting another connection.");            }            state.badConnectionCount++;            localBadConnectionCount++;            conn = null;            if (localBadConnectionCount > (poolMaximumIdleConnections + 3)) {              if (log.isDebugEnabled()) {                log.debug("PooledDataSource: Could not get a good connection to the database.");              }              throw new SQLException("PooledDataSource: Could not get a good connection to the database.");            }          }        }      }     }     if (conn == null) {      if (log.isDebugEnabled()) {        log.debug("PooledDataSource: Unknown severe error condition.  The connection pool returned a null connection.");      }      throw new SQLException("PooledDataSource: Unknown severe error condition.  The connection pool returned a null connection.");    }     return conn;  }
```

我们看下上面的方法都做了什么：
1.  先看是否有空闲(idle)状态下的PooledConnection对象，如果有，就直接返回一个可用的PooledConnection对象；否则进行第2步。
2\. 查看活动状态的PooledConnection池activeConnections是否已满；如果没有满，则创建一个新的PooledConnection对象，然后放到activeConnections池中，然后返回此PooledConnection对象；否则进行第三步；
3.  看最先进入activeConnections池中的PooledConnection对象是否已经过期：如果已经过期，从activeConnections池中移除此对象，然后创建一个新的PooledConnection对象，添加到activeConnections中，然后将此对象返回；否则进行第4步；
4.  线程等待，循环2步。

流程图如下所示：

![](https://img-blog.csdn.net/20170514140700209?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center) 

当我们拿到数据库连接PooledConnection后，我们在使用完之后一般来说就要关闭这个数据库连接，但是，对于池化来说，我们关闭了一个数据库连接并不是真正意义上想关闭这个连接，而是想把它放回到数据库连接池中。

怎么实现呢？mybatis中使用了代理模式有效的解决了该问题。就是返回给外部使用的数据库连接其实是一个代理对象（通过调用getProxyConnection()返回的对象）。这个代理对象时在真实数据库连接创建的时候被创建的，如下所示：

```
public PooledConnection(Connection connection, PooledDataSource dataSource) {    this.hashCode = connection.hashCode();    this.realConnection = connection;    this.dataSource = dataSource;    this.createdTimestamp = System.currentTimeMillis();    this.lastUsedTimestamp = System.currentTimeMillis();    this.valid = true;    this.proxyConnection = (Connection) Proxy.newProxyInstance(Connection.class.getClassLoader(), IFACES, this);  }
```

而在调用这个代理对象的各个方法的时候，都是通过反射的方式，从invoke()方法进入，我们来看看：

```
 @Override  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {    String methodName = method.getName();    if (CLOSE.hashCode() == methodName.hashCode() && CLOSE.equals(methodName)) {      dataSource.pushConnection(this);      return null;    } else {      try {        if (!Object.class.equals(method.getDeclaringClass())) {          // issue #579 toString() should never fail          // throw an SQLException instead of a Runtime          checkConnection();        }        return method.invoke(realConnection, args);      } catch (Throwable t) {        throw ExceptionUtil.unwrapThrowable(t);      }    }  }
```

```
private static final String CLOSE = "close";
```

我们可以看到，这里做了一个特殊处理，那就是判断调用的方法名是否是close()方法，如果是的话，就调用数据源对象的pushConnection()方法将数据库连接放回到连接池中，如下所示：

```
protected void pushConnection(PooledConnection conn) throws SQLException {     synchronized (state) {      state.activeConnections.remove(conn);      if (conn.isValid()) {        if (state.idleConnections.size() < poolMaximumIdleConnections && conn.getConnectionTypeCode() == expectedConnectionTypeCode) {          state.accumulatedCheckoutTime += conn.getCheckoutTime();          if (!conn.getRealConnection().getAutoCommit()) {            conn.getRealConnection().rollback();          }          PooledConnection newConn = new PooledConnection(conn.getRealConnection(), this);          state.idleConnections.add(newConn);          newConn.setCreatedTimestamp(conn.getCreatedTimestamp());          newConn.setLastUsedTimestamp(conn.getLastUsedTimestamp());          conn.invalidate();          if (log.isDebugEnabled()) {            log.debug("Returned connection " + newConn.getRealHashCode() + " to pool.");          }          state.notifyAll();        } else {          state.accumulatedCheckoutTime += conn.getCheckoutTime();          if (!conn.getRealConnection().getAutoCommit()) {            conn.getRealConnection().rollback();          }          conn.getRealConnection().close();          if (log.isDebugEnabled()) {            log.debug("Closed connection " + conn.getRealHashCode() + ".");          }          conn.invalidate();        }      } else {        if (log.isDebugEnabled()) {          log.debug("A bad connection (" + conn.getRealHashCode() + ") attempted to return to the pool, discarding connection.");        }        state.badConnectionCount++;      }    }  }
```

简单的说下上面这个方法的逻辑：

1\. 首先将当前数据库连接从活动数据库连接集合activeConnections中移除；

2\. 判断当前数据库连接是否有效，如果无效，则跳转到第4步；如果有效，则继续下面的判断；

3\. 判断当前idleConnections集合中的闲置数据库连接数量是否没超过设置的阈值且是当前数据库连接池的创建出来的链接，如果是，则将该数据库连接放回到idleConnections集合中并且通知在此据库连接池上等待的请求对象线程，如果不是，则将数据库连接关闭；

4\. 将连接池中的坏数据库连接数+1，并返回；

（6）JNDI类型的数据源工厂JndiDataSourceFactory

对于JNDI类型的数据源的获取比较简单，mybatis中定义了一个JndiDataSourceFactory类用来创建通过JNDI形式创建的数据源。这个类源码如下： 

```
public class JndiDataSourceFactory implements DataSourceFactory {
 
  public static final String INITIAL_CONTEXT = "initial_context";
  public static final String DATA_SOURCE = "data_source";
  public static final String ENV_PREFIX = "env.";
 
  private DataSource dataSource;
 
  @Override
  public void setProperties(Properties properties) {
    try {
      InitialContext initCtx;
      Properties env = getEnvProperties(properties);
      if (env == null) {
        initCtx = new InitialContext();
      } else {
        initCtx = new InitialContext(env);
      }
 
      if (properties.containsKey(INITIAL_CONTEXT)
          && properties.containsKey(DATA_SOURCE)) {
        Context ctx = (Context) initCtx.lookup(properties.getProperty(INITIAL_CONTEXT));
        dataSource = (DataSource) ctx.lookup(properties.getProperty(DATA_SOURCE));
      } else if (properties.containsKey(DATA_SOURCE)) {
        dataSource = (DataSource) initCtx.lookup(properties.getProperty(DATA_SOURCE));
      }
 
    } catch (NamingException e) {
      throw new DataSourceException("There was an error configuring JndiDataSourceTransactionPool. Cause: " + e, e);
    }
  }
 
  @Override
  public DataSource getDataSource() {
    return dataSource;
  }
 
  private static Properties getEnvProperties(Properties allProps) {
    final String PREFIX = ENV_PREFIX;
    Properties contextProperties = null;
    for (Entry<Object, Object> entry : allProps.entrySet()) {
      String key = (String) entry.getKey();
      String value = (String) entry.getValue();
      if (key.startsWith(PREFIX)) {
        if (contextProperties == null) {
          contextProperties = new Properties();
        }
        contextProperties.put(key.substring(PREFIX.length()), value);
      }
    }
    return contextProperties;
  }
 
}
```

因为这块没看明白，对JNDI不是太熟悉，所以这块就不解释了，回头对这块了解了之后再行补充说明。如果有懂的朋友，也可以留言说明一下，谢谢。