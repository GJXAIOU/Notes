# mybatis 详解（二）---入门实例（基于XML）

[原文链接](https://www.cnblogs.com/ysocean/p/7277545.html)


通过上一小节，mybatis 和 jdbc 的区别，我们对 mybatis有了一个大致的了解，下面我们通过一个入门实例来对mybatis有更近一步的了解。

我们用 mybatis 来对 user 表进行增删改查操作。


## 1、创建MySQL数据库：mybatisDemo和表：user

这里我们就不写脚本创建了，创建完成后，再向其中插入几条数据即可。
user 表字段如下：

![](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170803001845334-1050272765.png)

![](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170803082356912-1932891072.png)



## 2、建立一个Java工程，并导入相应的jar包，具体目录如下

注意：log4j和Junit不是必须的，但是我们为了查看日志以及便于测试，加入了这两个jar包
![](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170803002013365-1628654827.png)

### 3、在 MyBatisTest 工程中添加数据库配置文件 mybatis-configuration.xml
```MyBatis_xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
 
<!-- 注意：environments标签，当mybatis和spring整合之后，这个标签是不用配置的 -->
 
<!-- 可以配置多个运行环境，但是每个 SqlSessionFactory 实例只能选择一个运行环境：development:开发模式 或者work：工作模式-->
 <environments default="development">
 <!--id属性必须和上面的default一样  -->
    <environment id="development">
    <!--事务管理器
        一、JDBC：这个配置直接简单使用了 JDBC 的提交和回滚设置。它依赖于从数据源得到的连接来管理事务范围
        二、MANAGED：这个配置几乎没做什么。它从来不提交或回滚一个连接。而它会让容器来管理事务的整个生命周期。比如 spring 或 JEE 应用服务器的上下文，默认情况下，它会关闭连接。然而一些容器并不希望这样，因此如果你需要从连接中停止它，就可以将 closeConnection 属性设置为 false，比如：
            <transactionManager type="MANAGED">
                <property name="closeConnection" value="false"/>
            </transactionManager>
      -->
      <transactionManager type="JDBC"/>
      <!--dataSource 元素使用标准的 JDBC 数据源接口来配置 JDBC 连接对象源  -->
      <dataSource type="POOLED">
        <property name="driver" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/mybatisdemo"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
      </dataSource>
    </environment>
  </environments>
   
</configuration>
```



## 4、定义表所对应的实体类
![定义数据表对应的实体类]($resource/%E5%AE%9A%E4%B9%89%E6%95%B0%E6%8D%AE%E8%A1%A8%E5%AF%B9%E5%BA%94%E7%9A%84%E5%AE%9E%E4%BD%93%E7%B1%BB.png)

```User_java
package com.ys.po;
import java.util.Date;
 
public class User {
    private int id;
    private String username;
    private String sex;
    private Date birthday;
    private String address;
   // 省略 get、set 方法以及重写 toString 方法
}
```



## 5、定义操作 user 表的sql映射文件 UserMapper.xml　　

![定义表的SQL映射文件]($resource/%E5%AE%9A%E4%B9%89%E8%A1%A8%E7%9A%84SQL%E6%98%A0%E5%B0%84%E6%96%87%E4%BB%B6.png)

```UserMapper_xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.ys.po.userMapper">
 
    <!-- 根据 id 查询 user 表中的数据
       id：唯一标识符，此文件中的 id 值不能重复, 可以自定义；
       resultType：返回值类型，一条数据库记录也就对应实体类的一个对象；
       parameterType：参数类型，也就是查询条件的类型；
    -->
    <select id="selectUserById"
            resultType="com.ys.po.User" parameterType="int">
        <!-- 这里和普通的 sql 查询语句差不多，对于只有一个参数，后面的 #{id}表示占位符，里面不一定要写 id, 写啥都可以，但是不要空着，如果有多个参数则必须写 pojo 类里面的属性 -->
        select * from user where id = #{id}
    </select>
   
     
    <!-- 查询 user 表的所有数据
        注意：因为是查询所有数据，所以返回的应该是一个集合,这个集合里面每个元素都是User类型
     -->
    <select id="selectUserAll" resultType="com.ys.po.User">
        select * from user
    </select>
     
    <!-- 模糊查询：根据 user 表的username字段
            下面两种写法都可以，但是要注意
            1、${value}里面必须要写value，不然会报错
            2、${}表示拼接 sql 字符串，将接收到的参数不加任何修饰拼接在sql语句中
            3、使用${}会造成 sql 注入
     -->
    <select id="selectLikeUserName" resultType="com.ys.po.User" parameterType="String">
        select * from user where username like '%${value}%'
        <!-- select * from user where username like #{username} -->
    </select>
     
    <!-- 向 user 表插入一条数据 -->
    <insert id="insertUser" parameterType="com.ys.po.User">
        insert into user(id,username,sex,birthday,address)
            value(#{id},#{username},#{sex},#{birthday},#{address})
    </insert>
     
    <!-- 根据 id 更新 user 表的数据 -->
    <update id="updateUserById" parameterType="com.ys.po.User">
        update user set username=#{username} where id=#{id}
    </update>
     
    <!-- 根据 id 删除 user 表的数据 -->
    <delete id="deleteUserById" parameterType="int">
        delete from user where id=#{id}
    </delete>
</mapper>
```
parameterType 这个属性表示的是传入参数类型。
注意：写的不是包路径，而是某个类的全类名，那么表示的是传入的参数类型是这个类的类型。所以不管是 int ，String 这样的数据类型，或者是全类名，都是表示参数类型，并不冲突。


## 6、向 mybatis-configuration.xml 配置文件中注册 userMapper.xml 文件
在 configuration 中注册配置文件：
```MyBatis_xml
<mappers>
       <!-- 注册userMapper.xml文件，
       userMapper.xml位于com.ys.mapper这个包下，所以resource写成com/ys/mapper/userMapper.xml-->
       <mapper resource="com/ys/mapper/userMapper.xml"/>
</mappers>
```


##  7、创建测试类
```CRUDTest_java
package com.ys.test;
 
import java.io.InputStream;
import java.util.List;
 
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Before;
import org.junit.Test;
 
import com.ys.po.User;
 
public class CRUDTest {
    //定义 SqlSession
    SqlSession session =null;
     
    @Before
    public void init(){
        //定义mybatis全局配置文件
        String resource = "mybatis-configuration.xml";
        //加载 mybatis 全局配置文件
        InputStream inputStream = CRUDTest.class.getClassLoader()
                                    .getResourceAsStream(resource);
        //构建sqlSession的工厂
        SqlSessionFactory sessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        //根据 sqlSessionFactory 产生 session
        session = sessionFactory.openSession();
    }
     
    //根据id查询user表数据
    @Test
    public void testSelectUserById(){
        /*这个字符串由 userMapper.xml 文件中 两个部分构成
            <mapper namespace="com.ys.po.userMapper"> 的 namespace 的值
            <select id="selectUserById" > id 值*/
        String statement = "com.ys.po.userMapper.selectUserById";
        User user = session.selectOne(statement, 1);
        System.out.println(user);
        session.close();
    }
     
    //查询所有user表所有数据
    @Test
    public void testSelectUserAll(){
        String statement = "com.ys.po.userMapper.selectUserAll";
        List<User> listUser = session.selectList(statement);
        for(User user : listUser){
            System.out.println(user);
        }
        session.close();
    }
     
    //模糊查询：根据 user 表的username字段
    @Test
    public void testSelectLikeUserName(){
        String statement = "com.ys.po.userMapper.selectLikeUserName";
        List<User> listUser = session.selectList(statement, "%t%");
        for(User user : listUser){
            System.out.println(user);
        }
        session.close();
         
    }
    //向 user 表中插入一条数据
    @Test
    public void testInsertUser(){
        String statement = "com.ys.po.userMapper.insertUser";
        User user = new User();
        user.setUsername("Bob");
        user.setSex("女");
        session.insert(statement, user);
        //提交插入的数据
        session.commit();
        session.close();
    }
     
    //根据 id 更新 user 表的数据
    @Test
    public void testUpdateUserById(){
        String statement = "com.ys.po.userMapper.updateUserById";
        //如果设置的 id不存在，那么数据库没有数据更改
        User user = new User();
        user.setId(4);
        user.setUsername("jim");
        session.update(statement, user);
        session.commit();
        session.close();
    }
     
 
    //根据 id 删除 user 表的数据
    @Test
    public void testDeleteUserById(){
        String statement = "com.ys.po.userMapper.deleteUserById";
        session.delete(statement,4);
        session.commit();
        session.close();
    }
}
```


### 补充：如何得到插入数据之后的主键值？

- 第一种：数据库设置主键自增机制
userMapper.xml 文件中定义：
```UserMapper_xml
<!-- 向 user 表插入一条数据 -->
    <insert id="insertUser" parameterType="com.ys.po.User">
        <!-- 将插入的数据主键返回到 user 对象中
             keyProperty:将查询到的主键设置到parameterType 指定到对象的那个属性
             select LAST_INSERT_ID()：查询上一次执行insert 操作返回的主键id值，只适用于自增主键
             resultType:指定 select LAST_INSERT_ID() 的结果类型
             order:AFTER，相对于 select LAST_INSERT_ID()操作的顺序
         -->
        <selectKey keyProperty="id" resultType="int" order="AFTER">
            select LAST_INSERT_ID()
        </selectKey>
        insert into user(username,sex,birthday,address)
            value(#{username},#{sex},#{birthday},#{address})
    </insert>
```

测试：
```InsertUserTest_java
//向 user 表中插入一条数据并获取主键值
    @Test
    public void InsertUserTest(){
        String statement = "com.ys.po.userMapper.insertUser";
        User user = new User();
        user.setUsername("Bob");
        user.setSex("女");
        session.insert(statement, user);
        //提交插入的数据
        session.commit();
        //打印主键值
        System.out.println(user.getId());
        session.close();
    }
```

- 第二种：非自增主键机制
```UserMapper_xml
<!-- 向 user 表插入一条数据 -->
    <insert id="insertUser" parameterType="com.ys.po.User">
        <!-- 将插入的数据主键返回到 user 对象中
        流程是：首先通过 select UUID()得到主键值，然后设置到 user 对象的id中，在进行 insert 操作
             keyProperty:将查询到的主键设置到parameterType 指定到对象的那个属性
             select UUID()：得到主键的id值，注意这里是字符串
             resultType:指定 select UUID() 的结果类型
             order:BEFORE，相对于 select UUID()操作的顺序
         -->
        <selectKey keyProperty="id" resultType="String" order="BEFORE">
            select UUID()
        </selectKey>
        insert into user(id,username,sex,birthday,address)
            value(#{id},#{username},#{sex},#{birthday},#{address})
    </insert>
```


### 总结：
①、parameterType：指定输入参数的类型

②、resultType：指定输出结果的类型，在select中如果查询结果是集合，那么也表示集合中每个元素的类型

③、`#{}`：表示占位符，用来接收输入参数，类型可以是简单类型，pojo,HashMap等等
如果接收简单类型，#{}可以写成 value 或者其他名称
如果接收 pojo 对象值，通过 OGNL 读取对象中的属性值，即属性.属性.属性...的方式获取属性值

④、`${}`：表示一个拼接符，会引起 sql 注入，不建议使用　　
用来接收输入参数，类型可以是简单类型，pojo,HashMap等等
如果接收简单类型，${}里面只能是 value
如果接收 pojo 对象值，通过 OGNL 读取对象中的属性值，即属性.属性.属性...的方式获取属性值