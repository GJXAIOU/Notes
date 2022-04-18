做 Redis 缓存配置时，Spring 找不到 Redis 的连接配置



问题解决， 这是由于配置了两个连接池，一个是 druid 连接池，一个是 Redis 的缓存连接。而 spring 加载完 druid 连接池后就不加载了，导致 Redis 的连接配置读取不进来。

解决办法：在加载连接池的配置加上 ignore-unresolvable="true"，两个连接池都要加上

比如：

spring-mybatis.xml 配置文件中

`<context:property-placeholder location="classpath:druid-config.properties" ignore-unresolvable="true"/>`

spring-redis.xml 配置文件中

`<context:property-placeholder location="classpath:redis.properties" ignore-unresolvable="true"/>`