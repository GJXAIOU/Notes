---
tag:
- 未看
flag: yellow
---
# Mybatis中SQL语句执行过程详解

2017年05月15日 20:37:24 [DreamMakers](https://me.csdn.net/andamajing) 阅读数 4222 文章标签： [数据库](https://so.csdn.net/so/search/s.do?q=%E6%95%B0%E6%8D%AE%E5%BA%93&t=blog)[mybatis](https://so.csdn.net/so/search/s.do?q=mybatis&t=blog)[架构](https://so.csdn.net/so/search/s.do?q=%E6%9E%B6%E6%9E%84&t=blog)[源码](https://so.csdn.net/so/search/s.do?q=%E6%BA%90%E7%A0%81&t=blog) 更多

分类专栏： [Mybatis](https://blog.csdn.net/andamajing/article/category/6902012) [Mybatis应用及原理探析](https://blog.csdn.net/andamajing/article/category/6902014) [深入了解Mybatis使用及实现原理](https://blog.csdn.net/andamajing/article/category/9268791)

[](http://creativecommons.org/licenses/by-sa/4.0/)版权声明：本文为博主原创文章，遵循[ CC 4.0 BY-SA ](http://creativecommons.org/licenses/by-sa/4.0/)版权协议，转载请附上原文出处链接和本声明。

本文链接：[https://blog.csdn.net/andamajing/article/details/72179560](https://blog.csdn.net/andamajing/article/details/72179560) 

前面的十来篇文章我们对Mybatis中的配置和使用已经进行了比较详细的说明，想了解的朋友可以查看一下我专栏中的其他文章。 

但是你对整个SQL语句操作的流程了解吗？如果你还不是很了解，那么可以继续往下看，如果你已经了解了，那么可以跳过啦![大笑](http://static.blog.csdn.net/xheditor/xheditor_emot/default/laugh.gif)（因为一大推的源码估计要看的你头晕啊！！！）

所有语句的执行都是通过SqlSession对象来操作的，SqlSession是由SqlSessionFactory类生成的。 

首先根据配置文件来创建一个SqlSessionFactory，然后调用openSession来获取一个SqlSession。我们从时序图来看看可能会更加清晰：

![](https://img-blog.csdn.net/20170515173800117?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

（1）生成SqlSessionFactory对象（默认实现是DefaultSqlSessionFactory）的过程

```
public SqlSessionFactory build(Reader reader, String environment, Properties properties) {    try {      XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);      return build(parser.parse());    } catch (Exception e) {      throw ExceptionFactory.wrapException("Error building SqlSession.", e);    } finally {      ErrorContext.instance().reset();      try {        reader.close();      } catch (IOException e) {        // Intentionally ignore. Prefer previous error.      }    }  }
```

```
// Mybatis 通过SqlSessionFactory获取SqlSession, 然后才能通过SqlSession与数据库进行交互	private static SqlSessionFactory getSessionFactory() {		SqlSessionFactory sessionFactory = null;		String resource = "configuration.xml";		try {			sessionFactory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsReader(resource));		} catch (IOException e) {			e.printStackTrace();		}		return sessionFactory;	}
```

（2）获取SqlSession对象

通过调用DefaultSqlSessionFactory的openSession()方法来获取SqlSession对象。

```
@Override  public SqlSession openSession() {    return openSessionFromDataSource(configuration.getDefaultExecutorType(), null, false);  }
```

```
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {    Transaction tx = null;    try {      final Environment environment = configuration.getEnvironment();      final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);      tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);      final Executor executor = configuration.newExecutor(tx, execType);      return new DefaultSqlSession(configuration, executor, autoCommit);    } catch (Exception e) {      closeTransaction(tx); // may have fetched a connection so lets call close()      throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);    } finally {      ErrorContext.instance().reset();    }  }
```

 通过代码可以看出，最终返回的是一个DefaultSqlSession实例对象。接下来就是根据这个DefaultSqlSession来获取对应的Mapper对象。

（3）获取MapperProxy对象

```
UserDao userMapper = sqlSession.getMapper(UserDao.class);  
```

 如上所示，通过SqlSession对象调用getMapper()方法来获取相应的Dao接口实现。我们通过代码跟踪一直往下看：

DefaultSqlSession类中： 

```
@Override  public <T> T getMapper(Class<T> type) {    return configuration.<T>getMapper(type, this);  }
```

Configuration类中： 

```
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {    return mapperRegistry.getMapper(type, sqlSession);  }
```

 MapperRegistry类中：

```
 @SuppressWarnings("unchecked")  public <T> T getMapper(Class<T> type, SqlSession sqlSession) {    final MapperProxyFactory<T> mapperProxyFactory = (MapperProxyFactory<T>) knownMappers.get(type);    if (mapperProxyFactory == null) {      throw new BindingException("Type " + type + " is not known to the MapperRegistry.");    }    try {      return mapperProxyFactory.newInstance(sqlSession);    } catch (Exception e) {      throw new BindingException("Error getting mapper instance. Cause: " + e, e);    }  }
```

 在MapperRegistry类中维护着一个Map，这个Map中存储着每个Mapper类型和其对应的代理对象工厂类，如下定义所示：

```
private final Map<Class<?>, MapperProxyFactory<?>> knownMappers = new HashMap<Class<?>, MapperProxyFactory<?>>();
```

 在mybatis初始化的过程中就根据配置文件<mappers>元素的配置，将相关的映射文件给加载到了内存，同时保存到了这个knownMappers中。这里，在调用getMapper()的时候，就会从这个knownMappers中寻找该Dao接口，如果没有找到，就直接抛出异常，说明没有在配置文件中配置说明，如果获取到了，那么就拿出其对应的代理对象工厂类出来，并从工厂类中通过newInstance()方法来获取一个代理对象。

MapperProxyFactory类中： 

```
@SuppressWarnings("unchecked")  protected T newInstance(MapperProxy<T> mapperProxy) {    return (T) Proxy.newProxyInstance(mapperInterface.getClassLoader(), new Class[] { mapperInterface }, mapperProxy);  }   public T newInstance(SqlSession sqlSession) {    final MapperProxy<T> mapperProxy = new MapperProxy<T>(sqlSession, mapperInterface, methodCache);    return newInstance(mapperProxy);  }
```

 从代码可以看出来，其实我们调用sqlSession.getMapper(UserDao.class)方法的时候,返回的是一个和UserDao接口对应的MapperProxy代理对象。如下定义所示，MapperProxy类是一个实现了InvocationHandler的代理类：

```
public class MapperProxy<T> implements InvocationHandler, Serializable
```

上面的代码，如果整理成时序图，如下所示：

![](https://img-blog.csdn.net/20170515184616400?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

拿到了Dao接口的代理对象后，我们应该就可以进行具体的增删改查了，我们继续往下看。

（4）Executor对象

当拿到了UserDao对象（其实是MapperProxy代理对象）后，我们调用Dao接口中定义的方法，如下所示：

```
User user = userMapper.findUserById(10);  
```

 这时候便调用了MapperProxy对象的invoke方法了；

```
@Override  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {    if (Object.class.equals(method.getDeclaringClass())) {      try {        return method.invoke(this, args);      } catch (Throwable t) {        throw ExceptionUtil.unwrapThrowable(t);      }    }    final MapperMethod mapperMethod = cachedMapperMethod(method);    return mapperMethod.execute(sqlSession, args);  }
```

 MapperMethod类中：

```
public Object execute(SqlSession sqlSession, Object[] args) {    Object result;    switch (command.getType()) {      case INSERT: {    	Object param = method.convertArgsToSqlCommandParam(args);        result = rowCountResult(sqlSession.insert(command.getName(), param));        break;      }      case UPDATE: {        Object param = method.convertArgsToSqlCommandParam(args);        result = rowCountResult(sqlSession.update(command.getName(), param));        break;      }      case DELETE: {        Object param = method.convertArgsToSqlCommandParam(args);        result = rowCountResult(sqlSession.delete(command.getName(), param));        break;      }      case SELECT:        if (method.returnsVoid() && method.hasResultHandler()) {          executeWithResultHandler(sqlSession, args);          result = null;        } else if (method.returnsMany()) {          result = executeForMany(sqlSession, args);        } else if (method.returnsMap()) {          result = executeForMap(sqlSession, args);        } else if (method.returnsCursor()) {          result = executeForCursor(sqlSession, args);        } else {          Object param = method.convertArgsToSqlCommandParam(args);          result = sqlSession.selectOne(command.getName(), param);        }        break;      case FLUSH:        result = sqlSession.flushStatements();        break;      default:        throw new BindingException("Unknown execution method for: " + command.getName());    }    if (result == null && method.getReturnType().isPrimitive() && !method.returnsVoid()) {      throw new BindingException("Mapper method '" + command.getName()           + " attempted to return null from a method with a primitive return type (" + method.getReturnType() + ").");    }    return result;  }
```

 从上面这个方法实现上可以看出，已经根据执行方法(CRUD)进行了不同的处理，我们简单看一个方法executeForMany，代码如下所示：

```
private <E> Object executeForMany(SqlSession sqlSession, Object[] args) {    List<E> result;    Object param = method.convertArgsToSqlCommandParam(args);    if (method.hasRowBounds()) {      RowBounds rowBounds = method.extractRowBounds(args);      result = sqlSession.<E>selectList(command.getName(), param, rowBounds);    } else {      result = sqlSession.<E>selectList(command.getName(), param);    }    // issue #510 Collections & arrays support    if (!method.getReturnType().isAssignableFrom(result.getClass())) {      if (method.getReturnType().isArray()) {        return convertToArray(result);      } else {        return convertToDeclaredCollection(sqlSession.getConfiguration(), result);      }    }    return result;  }
```

 通过观察这些代码，发现最终的实现都是通过sqlSession对象来进行操作的。我们继续往里看，看看selectList方法：

```
@Override  public <E> List<E> selectList(String statement) {    return this.selectList(statement, null);  }   @Override  public <E> List<E> selectList(String statement, Object parameter) {    return this.selectList(statement, parameter, RowBounds.DEFAULT);  }   @Override  public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {    try {      MappedStatement ms = configuration.getMappedStatement(statement);      return executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);    } catch (Exception e) {      throw ExceptionFactory.wrapException("Error querying database.  Cause: " + e, e);    } finally {      ErrorContext.instance().reset();    }  }
```

 可以看到，内部是把查询操作委托给了一个Executor对象（即executor.query()），Executor是一个接口，mybatis为其实现了一个抽象基类BaseExecutor，我们跟踪上面的代码中的query方法继续往里看：

BaseExecutor类中： 

```
@Override  public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {    BoundSql boundSql = ms.getBoundSql(parameter);    CacheKey key = createCacheKey(ms, parameter, rowBounds, boundSql);    return query(ms, parameter, rowBounds, resultHandler, key, boundSql); }   @SuppressWarnings("unchecked")  @Override  public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {    ErrorContext.instance().resource(ms.getResource()).activity("executing a query").object(ms.getId());    if (closed) {      throw new ExecutorException("Executor was closed.");    }    if (queryStack == 0 && ms.isFlushCacheRequired()) {      clearLocalCache();    }    List<E> list;    try {      queryStack++;      list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;      if (list != null) {        handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);      } else {        list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);      }    } finally {      queryStack--;    }    if (queryStack == 0) {      for (DeferredLoad deferredLoad : deferredLoads) {        deferredLoad.load();      }      // issue #601      deferredLoads.clear();      if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {        // issue #482        clearLocalCache();      }    }    return list;  }
```

 在上面的方法中，我们看到当list==null的时候会调用queryFromDatabase()方法，这个方法如下：

```
private <E> List<E> queryFromDatabase(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {    List<E> list;    localCache.putObject(key, EXECUTION_PLACEHOLDER);    try {      list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);    } finally {      localCache.removeObject(key);    }    localCache.putObject(key, list);    if (ms.getStatementType() == StatementType.CALLABLE) {      localOutputParameterCache.putObject(key, parameter);    }    return list;  }
```

 然后会调用doQuery()方法，BaseExecutor中的doQuery方法定义成了抽象方法，由具体的继承类进行个性化的实现。这里，我们拿mybatis中默认使用的SimpleExecutor来看看：

SimpleExecutor类中：

```
 @Override  public <E> List<E> doQuery(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) throws SQLException {    Statement stmt = null;    try {      Configuration configuration = ms.getConfiguration();      StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);      stmt = prepareStatement(handler, ms.getStatementLog());      return handler.<E>query(stmt, resultHandler);    } finally {      closeStatement(stmt);    }  }
```

 从这个方法可以看到，首先根据调用Configuration类的newStatementHandler方法来获取一个sql操作对象：

Configuration类中：

```
public StatementHandler newStatementHandler(Executor executor, MappedStatement mappedStatement, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {    StatementHandler statementHandler = new RoutingStatementHandler(executor, mappedStatement, parameterObject, rowBounds, resultHandler, boundSql);    statementHandler = (StatementHandler) interceptorChain.pluginAll(statementHandler);    return statementHandler;  }
```

RoutingStatementHandler类中： 

```
public RoutingStatementHandler(Executor executor, MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql) {     switch (ms.getStatementType()) {      case STATEMENT:        delegate = new SimpleStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);        break;      case PREPARED:        delegate = new PreparedStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);        break;      case CALLABLE:        delegate = new CallableStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);        break;      default:        throw new ExecutorException("Unknown statement type: " + ms.getStatementType());    }   }
```

 可以看到，这里根据配置来创建Statement、PreparedStatement或者CallableStatement三者之中的一个。然后调用相应的方法，如query（）方法。以SimpleStatementHandler为例，我们看看具体的sql操作：

```
 @Override  public <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException {    String sql = boundSql.getSql();    statement.execute(sql);    return resultSetHandler.<E>handleResultSets(statement);  }
```

 看到这里，我们终于看到了黎明的曙光，因为这里已经看到Jdbc中的数据库操作代码了，即statement.execute(sql)。在查询完之后使用resultSetHandler来进行查询结果集的处理。

上面的代码的整体流程图大概如下所示：

![](https://img-blog.csdn.net/20170515203114001?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

至此，一次完整的sql解析和处理过程便讲解完毕了，感兴趣的可以自己对着源码看看。