# mybatis 详解（三）---入门实例（基于注解）
[原文链接](https://www.cnblogs.com/ysocean/p/7282639.html)

@toc

注意：注解配置我们不需要 UserMapper.xml 文件了,使用注解接口 UserMapper.java。

##  1、创建MySQL数据库：mybatisDemo和表：user 
详情参考：mybatis 详解（二）------入门实例（基于XML） 一致

## 2、建立一个Java工程，并导入相应的jar包，具体目录如下
详情参考：mybatis 详解（二）------入门实例（基于XML）]一致


##  3、在 MyBatisTest 工程中添加数据库配置文件 mybatis-configuration.xml
详情参考：mybatis 详解（二）------入门实例（基于XML） 一致

## 4、定义表所对应的实体类
详情参考：mybatis 详解（二）------入门实例（基于XML) 一致


## 5、定义操作 user 表的注解接口 UserMapper.java
```UserMapper_java
package com.ys.annocation;
 
import org.apache.ibatis.annotations.Delete;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.annotations.Update;
 
import com.ys.po.User;
 
public interface UserMapper {
    //根据 id 查询 user 表数据
    @Select("select * from user where id = #{id}")
    public User selectUserById(int id) throws Exception;
 
    //向 user 表插入一条数据
    @Insert("insert into user(username,sex,birthday,address) value(#{username},#{sex},#{birthday},#{address})")
    public void insertUser(User user) throws Exception;
     
    //根据 id 修改 user 表数据
    @Update("update user set username=#{username},sex=#{sex} where id=#{id}")
    public void updateUserById(User user) throws Exception;
     
    //根据 id 删除 user 表数据
    @Delete("delete from user where id=#{id}")
    public void deleteUserById(int id) throws Exception;
     
}
```


## 6、向 mybatis-configuration.xml 配置文件中注册 UserMapper.java 文件
```MyBatis_xml
<mappers>
       <mapper class="com.ys.annocation.UserMapper"/>
</mappers>
```

　
##  7、创建测试类
```UserAnnocationTest_java
package com.ys.test;
 
import java.io.InputStream;
 
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.junit.Before;
import org.junit.Test;
 
import com.ys.annocation.UserMapper;
import com.ys.po.User;
 
public class UserAnnocationTest {
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
     
    //注解的增删改查方法测试
    @Test
    public void testAnncationCRUD() throws Exception{
        //根据session获取 UserMapper接口
        UserMapper userMapper = session.getMapper(UserMapper.class);
        //调用selectUserById()方法
        User user = userMapper.selectUserById(1);
        System.out.println(user);
         
        //调用  insertUser() 方法
        User user1 = new User();
        user1.setUsername("aliks");
        user1.setSex("不详");
        userMapper.insertUser(user1);
         
        //调用 updateUserById() 方法
        User user2 = new User();
        user2.setId(6);
        user2.setUsername("lbj");
        userMapper.updateUserById(user2);
         
        //调用 () 方法
        userMapper.deleteUserById(6);
         
        session.commit();
        session.close();
    }
}
```
