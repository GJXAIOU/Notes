# Mybatis配置之别名配置元素详述

[原文链接](https://blog.csdn.net/andamajing/article/details/71503263)

在前面的文章Mybatis配置之<properties>属性配置元素详述，我们讲述了<properties>标签元素的配置和使用方法。在这篇文章中，我们来说说**<typeAliases>标签元素，这个元素主要是用于对类型进行别名控制**，具体什么意思呢？我们下面用一个示例说明，看了之后我相信你就会明白了。

这里我们贴出之前的UserDao对应的mapper文件，如下所示：
```
<?xml version="1.0" encoding="UTF-8" ?>   
<!DOCTYPE mapper   
PUBLIC "-//ibatis.apache.org//DTD Mapper 3.0//EN"  
"http://ibatis.apache.org/dtd/ibatis-3-mapper.dtd">
<mapper namespace="com.majing.learning.mybatis.dao.UserDao">
 
	<select id="findUserById" resultType="com.majing.learning.mybatis.entity.User">
		select * from user where id = #{id}
	</select>
	
	<insert id="addUser" parameterType="com.majing.learning.mybatis.entity.User" useGeneratedKeys="true" keyProperty="id">
		insert into user(name,password,age) values(#{name},#{password},#{age})
	</insert>
	
	<delete id="deleteUser" parameterType="int">
		delete from user where id = #{id}
	</delete>
	
	<update id="updateUser" parameterType="com.majing.learning.mybatis.entity.User">
		update user set name = #{name}, password = #{password}, age = #{age} where id = #{id}
	</update>
 
</mapper>
```

 从这个配置文件中，我们可以看到<select>、<insert>和<update>三个标签元素的resultType都是User对象，需要设置这个User对象的类全限定名，即packname.classname。

我们发现一个问题，那就是这个类名，我们需要写多次，如果要改这个类名的话，我们需要在多个地方进行修改。很明显，这样配置的话很容易造成修改上的遗漏，同时也书写上也比较麻烦。因此，**MyBatis为我们提供了一个简单方便的配置方法，那就是使用<typeAliases>标签元素，给实体类设置一个别名**。

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration  
	PUBLIC "-//mybatis.org//DTD Config 3.0//EN"  
	"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
 
	<properties resource="mysql.properties">
	
	</properties>
 
	<settings>
		<setting name="logImpl" value="LOG4J" />
	</settings>
	
	<typeAliases>
 		<typeAlias alias="User" type="com.majing.learning.mybatis.entity.User"/> 
	</typeAliases>
      
	<!---省略其他配置---->
 
</configuration>
```

如上所示，我们**在原来的mybatis配置文件中增加了<typeAliases>标签，并将com.majing.learning.mybatis.entity.User这个实体类重命名为User**，然后我们在mapper配置文件中就可以如下使用了。

备注：这里需要注意的是，**typeAliases配置需要放置在settings之后**，否则会出异常！！！ 

```
<?xml version="1.0" encoding="UTF-8" ?>   
<!DOCTYPE mapper   
PUBLIC "-//ibatis.apache.org//DTD Mapper 3.0//EN"  
"http://ibatis.apache.org/dtd/ibatis-3-mapper.dtd">
<mapper namespace="com.majing.learning.mybatis.dao.UserDao">
 
	<select id="findUserById" resultType="User">
		select * from user where id = #{id}
	</select>
	
	<insert id="addUser" parameterType="User" useGeneratedKeys="true" keyProperty="id">
		insert into user(name,password,age) values(#{name},#{password},#{age})
	</insert>
	
	<delete id="deleteUser" parameterType="int">
		delete from user where id = #{id}
	</delete>
	
	<update id="updateUser" parameterType="User">
		update user set name = #{name}, password = #{password}, age = #{age} where id = #{id}
	</update>
 
</mapper>
```

 这样即使实体类名修改了，所需要修改的地方也只有一处，便于集中管理。

也许你会有疑问，如果实体类比较多怎么办？还不是要配置很多实体类和别名，NO,NO,NO！下面跟大家说说另一种配置方法。

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration  
	PUBLIC "-//mybatis.org//DTD Config 3.0//EN"  
	"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
 
	<properties resource="mysql.properties">
		
	</properties>
 
	<settings>
		<setting name="logImpl" value="LOG4J" />
	</settings>
	
	<typeAliases>
		<package name="com.majing.learning.mybatis.entity"/>
	</typeAliases>
 
	<!---省略其他配置---->
 
</configuration>
```

 在这里，**我们不再使用<typeAliases>标签下<typeAliase>，而是使用<package>标签，表示扫描该包名下的所有类（除了接口和匿名内部类），如果类名上有注解，则使用注解指定的名称作为别名，如果没有则使用类名首字母小写作为别名，如`com.majing.learning.mybatis.entity.User`这个类如果没有设置@Alias注解，则此时会被关联到user这个别名上**。

因此，按照上面的配置，我们还需要将实体类做一下调整，如下两种方式所示：

（1）给实体类添加@Alias注解 

```
package com.majing.learning.mybatis.entity;
 
import org.apache.ibatis.type.Alias;
 
@Alias(value="User")
public class User {
// 具体类的内容
	
}
```

 （2）实体类不加注解的情况下，修改mapper文件中引用的类型别名，改为小写，如下所示：

```
<?xml version="1.0" encoding="UTF-8" ?>   
<!DOCTYPE mapper   
PUBLIC "-//ibatis.apache.org//DTD Mapper 3.0//EN"  
"http://ibatis.apache.org/dtd/ibatis-3-mapper.dtd">
<mapper namespace="com.majing.learning.mybatis.dao.UserDao">
 
	<select id="findUserById" resultType="user">
		select * from user where id = #{id}
	</select>
	
	<insert id="addUser" parameterType="user" useGeneratedKeys="true" keyProperty="id">
		insert into user(name,password,age) values(#{name},#{password},#{age})
	</insert>
	
	<delete id="deleteUser" parameterType="int">
		delete from user where id = #{id}
	</delete>
	
	<update id="updateUser" parameterType="user">
		update user set name = #{name}, password = #{password}, age = #{age} where id = #{id}
	</update>
 
</mapper>
```

最后想说，mybatis为我们已经实现了很多别名，已经为许多常见的 Java 类型内建了相应的类型别名。它们都是大小写不敏感的，需要注意的是由基本类型名称重复导致的特殊处理。

| 别名 | 映射的类型 |
| --- | --- |
| _byte | byte |
| _long | long |
| _short | short |
| _int | int |
| _integer | int |
| _double | double |
| _float | float |
| _boolean | boolean |
| string | String |
| byte | Byte |
| long | Long |
| short | Short |
| int | Integer |
| integer | Integer |
| double | Double |
| float | Float |
| boolean | Boolean |
| date | Date |
| decimal | BigDecimal |
| bigdecimal | BigDecimal |
| object | Object |
| map | Map |
| hashmap | HashMap |
| list | List |
| arraylist | ArrayList |
| collection | Collection |
| iterator | Iterator |

  至此，关于别名的全部使用方法这里便介绍完成了。