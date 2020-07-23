# 通过Mybatis实现数据的增删改查（CRUD）操作

[原文链接](https://blog.csdn.net/andamajing/article/details/71216067)

在上一篇文章中，我们应该算是简单的进入了MyBatis的世界，在这篇文章中，我们从简单的增删改查说起，毕竟对于数据库操作来说，这几种操作是肯定逃不掉的。

在这篇文章中，我们不在对所有需要的东西全部列举出来，而是在上一篇文章的基础上进行修改。

首先，我们需要修改的就是接口类，提供增删改查的方法，如下所示：

```
package com.majing.learning.mybatis.dao;
import com.majing.learning.mybatis.entity.User;
 
public interface UserDao {
	/**
	 * 查询
	 */
	User findUserById (int userId);
	/**
	 * 增加
	 */
	void addUser(User user);
	/**
	 * 删除
	 */
	void deleteUser(int userId);
	/**
	 * 更新
	 */
	void updateUser(User user);
}
```

 紧接着，当然我们需要修改UserDaoMapper.xml文件，毕竟我们**要为每个方法提供对应的sql实现**。下面给出调整后的配置文件：

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

 至此，我们所需要修改的东西应该就没了，下面我们来写测试用例，先插入一条记录，然后更新这条记录，然后再查询之前的记录，最后删除这条记录。测试代码如下：

```
package com.majing.learning.mybatis;
  
public class UserDaoTest {
	
	@Test
	public void findUserById(){
		SqlSession sqlSession = getSessionFactory().openSession();  
		UserDao userMapper = sqlSession.getMapper(UserDao.class);  
		
		//增加一条新记录
		User user = new User();
		user.setName("majing");
		user.setPassword("19880101");
		user.setAge(29);
		userMapper.addUser(user);
		
		//更新该记录
		user.setName("new_majing");
		userMapper.updateUser(user);
		
		
		//查询该记录
		int id = user.getId();
		user = null;
		user = userMapper.findUserById(id);  
		System.out.println("更新后记录为："+user);
		
		//删除记录
		System.out.println("尝试删除该记录...");
		userMapper.deleteUser(id);
		
		user = userMapper.findUserById(id);  
		if(user==null){
			System.out.println("该记录已删除！");
		}else{
			System.out.println("该记录未被成功删除！");
		}
	}
 
	// Mybatis 通过SqlSessionFactory获取SqlSession, 然后才能通过SqlSession与数据库进行交互
	private static SqlSessionFactory getSessionFactory() {
		SqlSessionFactory sessionFactory = null;
		String resource = "configuration.xml";
		try {
			sessionFactory = new SqlSessionFactoryBuilder().build(Resources.getResourceAsReader(resource));
		} catch (IOException e) {
			e.printStackTrace();
		}
		return sessionFactory;
	}
}
```

 如果我们执行这个单元测试类，会得到如下的输出，一切都是这么正常，So easy：

![](https://img-blog.csdn.net/20170505172105717?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

但是
如果单步调试的话，我们会发现虽然每一步都按照我们的意愿进行了操作（增删改查），但是目标数据库中却始终一条记录都没有！！！数据根本没有插入到数据库中。

其实我们在获取数据库连接的时候采用了MyBatis的默认配置，即不会自动提交。所以这里我们需要设置一下，在获取连接时设置sql语句是自动提交的。如下所示：

```
SqlSession sqlSession = getSessionFactory().openSession(true);  
```

 这下就一切正常了，不信的可以试试。这也告诉我们一个道理，其实看别人写的东西都很简单，但是其实往往看到的和自己动手操作的往往自己得到的并不一样，只有自己动手写过才能更清晰的知道其中的知识，也掌握的更牢固。这也是我为什么学什么都自己尝试一遍的原因。