做Redis缓存配置时，Spring找不到Redis的连接配置



问题解决， 这是由于配置了两个连接池，一个是druid连接池，一个是Redis的缓存连接。而spring加载完druid连接池后就不加载了，导致Redis的连接配置读取不进来。

解决办法：在加载连接池的配置加上ignore-unresolvable="true"，两个连接池都要加上

比如：

spring-mybatis.xml配置文件中

`<context:property-placeholder location="classpath:druid-config.properties" ignore-unresolvable="true"/>`

spring-redis.xml配置文件中

`<context:property-placeholder location="classpath:redis.properties" ignore-unresolvable="true"/>`