# mybatis 详解（五）---动态SQL

[原文链接](https://www.cnblogs.com/ysocean/p/7289529.html)

前面几篇博客我们通过实例讲解了用mybatis对一张表进行的CRUD操作，但是我们发现写的 SQL 语句都比较简单，如果有比较复杂的业务，我们需要写复杂的 SQL 语句，往往需要拼接，而拼接 SQL ，稍微不注意，由于引号，空格等缺失可能都会导致错误。

那么怎么去解决这个问题呢？这就是本篇所讲的使用 mybatis 动态SQL，通过 if, choose, when, otherwise, trim, where, set, foreach等标签，可组合成非常灵活的SQL语句，从而在提高 SQL 语句的准确性的同时，也大大提高了开发人员的效率。

我们以 User 表为例来说明：
![表样式]($resource/%E8%A1%A8%E6%A0%B7%E5%BC%8F.png)


## 1、动态SQL:if 语句

　　根据 username 和 sex 来查询数据。如果username为空，那么将只根据sex来查询；反之只根据username来查询

　　首先不使用 动态SQL 来书写
```sql
<select id="selectUserByUsernameAndSex"
        resultType="user" parameterType="com.ys.po.User">
    <!-- 这里和普通的sql 查询语句差不多，对于只有一个参数，后面的 #{id}表示占位符，里面不一定要写id,
            写啥都可以，但是不要空着，如果有多个参数则必须写pojo类里面的属性 -->
    select * from user where username=#{username} and sex=#{sex}
</select>
```


　　上面的查询语句，我们可以发现，如果 #{username} 为空，那么查询结果也是空，如何解决这个问题呢？使用 if 来判断
```sql
<select id="selectUserByUsernameAndSex" resultType="user" parameterType="com.ys.po.User">
    select * from user where
        <if test="username != null">
           username=#{username}
        </if>
         
        <if test="username != null">
           and sex=#{sex}
        </if>
</select>
```
　　这样写我们可以看到，如果 sex 等于 null，那么查询语句为 select * from user where username=#{username},但是如果usename 为空呢？那么查询语句为 select * from user where and sex=#{sex}，这是错误的 SQL 语句，如何解决呢？请看下面的 where 语句

## 2、动态SQL:if+where 语句

```sql
<select id="selectUserByUsernameAndSex" resultType="user" parameterType="com.ys.po.User">
    select * from user
    <where>
        <if test="username != null">
           username=#{username}
        </if>
         
        <if test="username != null">
           and sex=#{sex}
        </if>
    </where>
</select>
```
　　这个“where”标签会知道如果它包含的标签中有返回值的话，它就插入一个‘where’。此外，如果标签返回的内容是以AND 或OR 开头的，则它会剔除掉。

## 3、动态SQL:if+set 语句

　　同理，上面的对于查询 SQL 语句包含 where 关键字，如果在进行更新操作的时候，含有 set 关键词，我们怎么处理呢？
```sql
<!-- 根据 id 更新 user 表的数据 -->
<update id="updateUserById" parameterType="com.ys.po.User">
    update user u
        <set>
            <if test="username != null and username != ''">
                u.username = #{username},
            </if>
            <if test="sex != null and sex != ''">
                u.sex = #{sex}
            </if>
        </set>
     
     where id=#{id}
</update>
```

　这样写，如果第一个条件 username 为空，那么 sql 语句为：update user u set u.sex=? where id=?
　如果第一个条件不为空，那么 sql 语句为：update user u set u.username = ? ,u.sex = ? where id=?

## 4、动态SQL:choose(when,otherwise) 语句

　　有时候，我们不想用到所有的查询条件，只想选择其中的一个，查询条件有一个满足即可，使用 choose 标签可以解决此类问题，类似于 Java 的 switch 语句

```sql
<select id="selectUserByChoose" resultType="com.ys.po.User" parameterType="com.ys.po.User">
      select * from user
      <where>
          <choose>
              <when test="id !='' and id != null">
                  id=#{id}
              </when>
              <when test="username !='' and username != null">
                  and username=#{username}
              </when>
              <otherwise>
                  and sex=#{sex}
              </otherwise>
          </choose>
      </where>
  </select>
```

也就是说，这里我们有三个条件，id,username,sex，只能选择一个作为查询条件

如果 id 不为空，那么查询语句为：select * from user where  id=?

如果 id 为空，那么看username 是否为空，如果不为空，那么语句为 select * from user where  username=?;

如果 username 为空，那么查询语句为 select * from user where sex=?


## 5、动态SQL:trim 语句

　　trim标记是一个格式化的标记，可以完成set或者是where标记的功能

　　①、用 trim 改写上面第二点的 if+where 语句

```sql
<select id="selectUserByUsernameAndSex" resultType="user" parameterType="com.ys.po.User">
        select * from user
        <!-- <where>
            <if test="username != null">
               username=#{username}
            </if>
             
            <if test="username != null">
               and sex=#{sex}
            </if>
        </where>  -->
        <trim prefix="where" prefixOverrides="and | or">
            <if test="username != null">
               and username=#{username}
            </if>
            <if test="sex != null">
               and sex=#{sex}
            </if>
        </trim>
    </select>
```

prefix：前缀　　　　　　

prefixoverride：去掉第一个and或者是or

②、用 trim 改写上面第三点的 if+set 语句

```sql
<!-- 根据 id 更新 user 表的数据 -->
    <update id="updateUserById" parameterType="com.ys.po.User">
        update user u
            <!-- <set>
                <if test="username != null and username != ''">
                    u.username = #{username},
                </if>
                <if test="sex != null and sex != ''">
                    u.sex = #{sex}
                </if>
            </set> -->
            <trim prefix="set" suffixOverrides=",">
                <if test="username != null and username != ''">
                    u.username = #{username},
                </if>
                <if test="sex != null and sex != ''">
                    u.sex = #{sex},
                </if>
            </trim>
         
         where id=#{id}
    </update>
```

　　suffix：后缀　　

　　suffixoverride：去掉最后一个逗号（也可以是其他的标记，就像是上面前缀中的and一样）

[回到顶部](https://www.cnblogs.com/ysocean/p/7289529.html#_labelTop)

### 6、动态SQL: SQL 片段

　　有时候可能某个 sql 语句我们用的特别多，为了增加代码的重用性，简化代码，我们需要将这些代码抽取出来，然后使用时直接调用。

　　比如：假如我们需要经常根据用户名和性别来进行联合查询，那么我们就把这个代码抽取出来，如下：
```sql
<!-- 定义 sql 片段 -->
<sql id="selectUserByUserNameAndSexSQL">
    <if test="username != null and username != ''">
        AND username = #{username}
    </if>
    <if test="sex != null and sex != ''">
        AND sex = #{sex}
    </if>
</sql>
```

　　引用 sql 片段
```sql
<select id="selectUserByUsernameAndSex" resultType="user" parameterType="com.ys.po.User">
    select * from user
    <trim prefix="where" prefixOverrides="and | or">
        <!-- 引用 sql 片段，如果refid 指定的不在本文件中，那么需要在前面加上 namespace -->
        <include refid="selectUserByUserNameAndSexSQL"></include>
        <!-- 在这里还可以引用其他的 sql 片段 -->
    </trim>
</select>
```

　　注意：①、最好基于 单表来定义 sql 片段，提高片段的可重用性

　　　　　②、在 sql 片段中最好不要包括 where 


## 7、动态SQL: foreach 语句

　　需求：我们需要查询 user 表中 id 分别为1,2,3的用户

　　sql语句：select * from user where id=1 or id=2 or id=3

　　　　　　 select * from user where id in (1,2,3)

①、建立一个 UserVo 类，里面封装一个 List<Integer> ids 的属性

```sql
package com.ys.vo;
 
import java.util.List;
 
public class UserVo {
    //封装多个用户的id
    private List<Integer> ids;
 
    public List<Integer> getIds() {
        return ids;
    }
 
    public void setIds(List<Integer> ids) {
        this.ids = ids;
    }
 
}　　
```

②、我们用 foreach 来改写 select * from user where id=1 or id=2 or id=3

```sql
<select id="selectUserByListId" parameterType="com.ys.vo.UserVo" resultType="com.ys.po.User">
    select * from user
    <where>
        <!--
            collection:指定输入对象中的集合属性
            item:每次遍历生成的对象
            open:开始遍历时的拼接字符串
            close:结束时拼接的字符串
            separator:遍历对象之间需要拼接的字符串
            select * from user where 1=1 and (id=1 or id=2 or id=3)
          -->
        <foreach collection="ids" item="id" open="and (" close=")" separator="or">
            id=#{id}
        </foreach>
    </where>
</select>
```

　　测试：
```sql
//根据id集合查询user表数据
@Test
public void testSelectUserByListId(){
    String statement = "com.ys.po.userMapper.selectUserByListId";
    UserVo uv = new UserVo();
    List<Integer> ids = new ArrayList<>();
    ids.add(1);
    ids.add(2);
    ids.add(3);
    uv.setIds(ids);
    List<User> listUser = session.selectList(statement, uv);
    for(User u : listUser){
        System.out.println(u);
    }
    session.close();
}
```


③、我们用 foreach 来改写 select * from user where id in (1,2,3)

```sql
<select id="selectUserByListId" parameterType="com.ys.vo.UserVo" resultType="com.ys.po.User">
        select * from user
        <where>
            <!--
                collection:指定输入对象中的集合属性
                item:每次遍历生成的对象
                open:开始遍历时的拼接字符串
                close:结束时拼接的字符串
                separator:遍历对象之间需要拼接的字符串
                select * from user where 1=1 and id in (1,2,3)
              -->
            <foreach collection="ids" item="id" open="and id in (" close=") " separator=",">
                #{id}
            </foreach>
        </where>
    </select>
```


### 8、总结

　　其实动态 sql 语句的编写往往就是一个拼接的问题，为了保证拼接准确，我们最好首先要写原生的 sql 语句出来，然后在通过 mybatis 动态sql 对照着改，防止出错。