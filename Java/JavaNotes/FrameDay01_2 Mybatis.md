# FrameDay01_2 Mybatis

**主要内容：**
* MyBatis 接口绑定方案及多参数传递动态 SQL
* 缓存
* ThreadLocal讲解
* 使用 Filter 模拟 OpenSessionInView 实现编程式事务

## 一、MyBatis 接口绑定方案及多参数传递

### （一）接口绑定
- **作用**: 实现创建一个接口后把 mapper.xml  由mybatis 生成接口的实现类,**通过调用接口对象就可以获取 mapper.xml 中编写的 sql**。后面 mybatis 和 spring 整合时使用的是这个方案.

- **实现步骤**:
  - 创建一个接口 ：例如：`LogMapper`
    - **接口 Mapper 名和接口名与 mapper.xml  中<mapper>namespace 相同**
目录示例：`com.gjxaiou.mapper`包下面包含一个接口：`LogMapper`和`LogMapper.xml`，然后 `LogMapper.xml` 中的<mapper>namespace 标签格式为：
`<mapper namespace = "com.gjxaiou.mapper.LogMapper>   </mapper>`
    - **接口中方法名和 mapper.xml  标签的 id 属性相同；**
  - 在 mybatis.xml 中使用<package>进行扫描接口和 mapper.xml；

- **代码实现步骤**:
  - 首先在 mybatis.xml 中全局配置文件中的`<mappers>`下使用`<package>`
```java
<mappers>
     <package name="com.gjxaiou.mapper"/>
</mappers>
```

  - 然后在 `com.gjxaiou.mapper` 包下新建接口：`LogMapper`
```java
public  interface  LogMapper  { 
    List<Log>  selAll();
}
```
  - 然后在 `com.gjxaiou.mapper` 中新建一个 `LogMapper.xml`
  其中 namespace 的值必须和接口全限定路径(包名+类名)一致，且使用的 id 值必须和接口中方法名相同。同时如果接口中方法为多个参数,可以省略 parameterType
```xml
<mapper  namespace="com.gjxaiou.mapper.LogMapper">
  <select  id="selAll"  resultType="log"> 
      select  *  from  log
  </select>
</mapper>
```

**绑定的使用：在其它 Java 类中**
```java
public class Test {
	public static void main(String[] args) throws IOException {
		InputStream is = Resources.getResourceAsStream("mybatis.xml");
		SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
		SqlSession session = factory.openSession();		

		LogMapper logMapper = session.getMapper(LogMapper.class);
		List<Log> list = logMapper.selAll();
		for (Log log : list) {
			System.out.println(log);
		}
```

### （二）**多参数实现办法**
一共有两种方法：一般方法和使用注解

- 方法一：一般方法
首先在在接口 `LogMapper` 中声明方法
`List<Log>  selByAccInAccout(String accin, String accout);`
然后在 LogMapper.xml  中添加即可，`#{}`中可以使用 param1,param2
```xml
<select id="selByAccInAccout" resultType="log">
     select * from log where accin=#{param1} and accout=#{param2}
</select>
```

-   方法二：可以使用注解方式
  首先在接口中声明方法
```java
/**
mybatis 把参数转换为 map 了,其中@Param("key")  参数内容就是 map 的 value
*/

List<Log> selByAccInAccout(@Param("accin")  String accin123,@Param("accout")  String  accout3454235);
```
然后在 mapper.xml  中添加
```xml
<!--  当多参数时,不需要写 parameterType  -->
<select  id="selByAccInAccout"  resultType="log"  > 
    select * from log where accin=#{accin} and accout=#{accout}
</select>
```
注：`#{}` 里面写@Param(“内容”)参数中内容。

**多参数的使用**
当然 log 是存在数据库中的一个表中，同时要新建实体类的；
在其它 Java 类中使用：
```java
public class Test {
   public static void main(String[] args) throws IOException {
      InputStream is = Resources.getResourceAsStream("mybatis.xml");
      SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
      SqlSession session = factory.openSession();

      LogMapper logMapper = session.getMapper(LogMapper.class);
      List<Log> list = logMapper.selByAccInAccout("3", "1");
      for (Log log : list) {
          System.out.println(log);
      }

      session.close();
      System.out.println("程序执行结束");
    }
}
```

## 二、动态 SQL

1. 根据不同的条件需要执行不同的 SQL 命令，称为动态 SQL
2. **MyBatis 中动态 SQL 的实现是在 `实体类mapper.xml`  中添加逻辑判断即可**。
注：以下的 xml 配置均在 LogMapper.xml 中；
### （一）If 使用
```xml
<select  id="selByAccinAccout"  resultType="log"> 
      select  *  from  log  where  1=1
<!--  OGNL 表达式,直接写 key 或对象的属性.不需要添加任何特字符号，这里的 and 执行的时候都会被转换为 & -->
      <if  test = "accin != null and accin != ''"> 
          and  accin = #{accin}
      </if>

      <if  test= "accout != null and accout != ''"> 
          and  accout=#{accout}
      </if>
</select>
```

### （二）where 使用
- 当编写 where 标签时,如果内容中第一个是 and 就会去掉第一个 and；
- 如果 <where> 中有内容会生成 where 关键字,如果没有内容不生成 where 关键字；

- 使用示例 ：效果：比直接使用 <if> 少写了  where 1 = 1;
```xml
<select  id="selByAccinAccout"  resultType="log"> 
    select  *  from  log
  <where>
     <if  test="accin!=null  and  accin!=''"> 
         and  accin=#{accin}
     </if>
  
     <if  test="accout!=null  and  accout!=''"> 
         and  accout=#{accout}
     </if>
  </where>
</select>
```


### （三）choose、when、otherwise 使用

- **只要有一个成立,其他都不执行**；

- 代码示例
如果 accin 和 accout 都不是 null 或不是””生成的 sql 中只有 where  accin=?
```xml
<select  id = "selByAccinAccout"  resultType = "log"> 
     select * from log
     <where>
        <choose>
            <when test = "accin != null and accin != ''">
                   and accin = #{accin}
            </when>

            <when test = "accout != null and accout != ''"> 
                   and accout = #{accout}
            </when>
       </choose>
    </where>
</select>
```

###  （四）set 使用

- 作用:去掉最后一个逗号
<set>用在修改 SQL 中 set 从句，如果<set>里面有内容则生成 set 关键字,没有就不生成

- 示例
其中：`id=#{id}` 目的防止<set>中没有内容,mybatis 不生成 set 关键字,如果修改中没有 set 从句 SQL 语法错误.
```xml
<update  id="upd"  parameterType="log"  > 
    update  log
    <set>
        id=#{id},
        <if  test="accIn!=null  and  accIn!=''"> 
            accin=#{accIn},
        </if>

        <if  test="accOut!=null  and  accOut!=''">
             accout=#{accOut},
        </if>
    </set>
    where id=#{id}
</update>
```

### （四） Trim 使用
里面的主要方法如下：
-  prefix 在前面添加内容
-  prefixOverrides 去掉前面内容
-  suffix 在后面添加内容
-  suffixOverrieds 去掉后面内容

-  执行顺序：首先去掉内容然后添加内容；
代码示例
```xml
<update  id = "upd"  parameterType = "log"> 
    update  log
    <trim prefix = "set" suffixOverrides = ","> 
        a = a,
    </trim> 
    where  id=100
</update>
```

### （五）bind 使用

- 作用:给参数重新赋值
- 使用场景:
  -  模糊查询：就是在 SQL 语句中，将用户输入的数据前后加上 `%` 然后执行语句；见示例
  -  在原内容前或后添加内容 ：将用户输入的数据进行格式化之后存入数据库；
```xml
<select  id="selByLog"  parameterType="log" resultType="log">
    <bind  name="accin"  value="'%'+accin+'%'"/> 
        #{money}
</select>

```

###  （六）foreach标签使用

- **用于循环参数内容，还具备在内容的前后添加内容，以及添加分隔符功能**；

- 适用场景：主要用于 `in` 查询以及批量新增中(但是 mybatis 中 foreach  效率比较低)

**批量新增操作：**
- 默认的批量新增的 SQL 语句为：`insert into log  VALUES (default,1,2,3),(default,2,3,4),(default,3,4,5)`

- 在执行批处理的时候，需要将 openSession()必须指定下面命令
`factory.openSession(ExecutorType.BATCH);`，这里底层 是 JDBC 的 PreparedStatement.addBatch();

- foreach示例
  - collection=”” 要遍历的集合
  - item 迭代变量, 可以使用`#{迭代变量名}`获取内容
  - open 循环后左侧添加的内容
  - close 循环后右侧添加的内容
  - separator 每次循环时，元素之间的分隔符
```xml
<select  id="selIn"  parameterType="list" resultType="log">
    select  *  from  log  where  id  in
    <foreach  collection="list"  item="abc"  open="(" close=")" separator=",">
        #{abc}
     </foreach>
</select>
```


### （七）<sql> 和 <include> 搭配使用

- **某些 SQL 片段如果希望复用,可以使用<sql>定义这个片段**；
```xml
<sql  id="mysql"> 
    id,accin,accout,money
</sql>
```

- 可以在<select>或<delete>或<update>或<insert>中使用<include> 引用
```xml
<select  id="">
    select  <include  refid="mysql"></include>
    from  log
</select>
```


## 三、ThreadLocal
主要优化 service中的方法

- 线程容器：给线程绑定一个 Object 内容后只要线程不变,可以随时取出。但是一旦改变线程,无法取出内容。

- 语法示例
```java
final ThreadLocal<String> threadLocal = ThreadLocal<>();
    threadLocal.set("测试");
    new Thread(){
        public  void  run()  {
            String  result  =  threadLocal.get();
            System.out.println("结果:"+result);
        };
    }.start();
```


## 四、缓存

1. 应用程序和数据库交互的过程是一个相对比较耗时的过程；
2. 缓存存在的意义：让应用程序减少对数据库的访问，提升程序运行效率；

- **MyBatis 中默认 SqlSession 缓存开启**
  - 同一个 SqlSession 对象调用同一个<select>时,只有第一次访问数据库,第一次之后把查询结果缓存到 SqlSession 缓存区(内存)中
  - 缓存的是 statement  对象.(简单记忆必须是用一个<select>)
    -  在 myabtis  时一个<select>对应一个 statement  对象
  -  有效范围必须是同一个 SqlSession 对象

- 缓存流程
  - 步骤一：先去缓存区中找是否存在 statement
  - 步骤二：如果存在返回结果
  - 步骤三：如果没有缓存 statement  对象,去数据库获取数据
  - 步骤四：数据库返回查询结果
  - 步骤五：把查询结果放到对应的缓存区中

![缓存]($resource/%E7%BC%93%E5%AD%98.png)

- SqlSessionFactory 缓存（二级缓存）
  - 有效范围:同一个 factory 内，哪个 SqlSession 都可以获取
  - **当数据频繁被使用,很少被修改的使用使用二级缓存**；
  - 使用二级缓存步骤
    - 在 mapper.xml  中添加
    - 如果不写 readOnly=”true”需要把实体类序列化；
`<cache readOnly = "true"></cache>`
  - 当 SqlSession 对象 close()时或 commit() 时会把 SqlSession 缓存的数据刷 (flush) 到 SqlSessionFactory 缓存区中
