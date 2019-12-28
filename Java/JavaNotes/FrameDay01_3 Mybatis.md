# FrameDay01_3 Mybatis

**主要内容**
* Auto Mapping 单表实现(别名方式)
* `<resultMap>` 实现单表配置
* 单个对象关联查询 (N+1,外连接) 集合对象关联查询
* 注解开发
* MyBatis 运行原理


## 一、MyBatis 实现多表查询

### （一）Mybatis 实现多表查询方式

一共三种方式
- 业务装配：对两个表编写单表查询语句,在业务(Service)把查询的两个结果进行关联；
- 使用`Auto  Mapping` 特性,在实现两表联合查询时通过别名完成映射；
- 使用 MyBatis 的 `<resultMap>` 标签进行实现；

### （二）多表查询时,类中包含另一个类的对象的分类

- 单个对象
- 集合对象

## 二、resultMap 标签

相当于现在直接设置映射关系，进行两者（SQL 查询结果和实体类之间）的匹配。

- `<resultMap>` 标签写在 `实体类Mapper.xml`  中，由程序员控制SQL 查询结果与实体类的映射关系；

- 默认 MyBatis 使用 `Auto  Mapping` 特性进行映射：即保持查询的数据库中的列名和实体类中的属性名相同即可；

- 使用 `<resultMap>`标签时,<select>标签不写 resultType  属性,而是使用 resultMap 属性来引用<resultMap>标签.

### （一）使用 resultMap 实现单表映射关系

-  首先进行数据库设计
示例：数据库表 teacher 中两个字段： id、name

- 然后进行实体类设计
```java
public class Teacher{
   private int id1;
   private String name1;
}
```

- 然后实现 TeacherMapper.xml  代码
```xml
<!-- 其中 type 的值为返回值类型-->
<resultMap type="teacher" id="mymap">
    <!-- 主键使用id 标签配置映射关系-->
    <!-- column 值为数据库中列名； property 值为实体类中的属性名 -->
    <id column="id" property="id1" />
    
    <!-- 其他列使用result 标签配置映射关系-->
    <result column="name" property="name1"/>
</resultMap>

<select id="selAll" resultMap="mymap">
      select * from teacher
</select>
```
上面代码如果使用原来的数据库中字段的名称和实体类相同的话，代码如下：
```xml
<select id = "selAll" resultType = "teacher">
    select * from teacher
</select>
```


###  （二）使用 resultMap 实现关联单个对象(N+1 方式)

- N+1 查询方式：先查询出某个表的全部信息，根据这个表的信息查询另一个表的信息；
- N+1 查询方式与业务装配的区别:
原来在 service 里面写的代码，现在由 mybatis 完成装配；

#### 实现步骤:

- 首先在 Student 实现类中包含了一个 Teacher  对象
```java
public class Student {
    private int id;
    private String name;
    private int age;
    private int tid;
    private Teacher teacher;
    // 该 POJO 类中其他信息省略
}
```

- 然后在 TeacherMapper.xml  中提供一个查询
```xml
<select id="selById" resultType="teacher" parameterType="int">
      select * from teacher where id=#{0}
</select>
```

- 最后在下面的 StudentMapper.xml 中完成装配

- `<association>` 表示当装配一个对象时使用
- `property`: 是对象在类中的属性名
- `select`:通过哪个查询查询出这个对象的信息
- `column`: 把当前表的哪个列的值做为参数传递给另一个查询
- 大前提使用 N+1 方式时：**如果列名和属性名相同可以不配置**,使用 Auto  mapping 特性.但是 mybatis 默认只会给列装配一次

```xml
<resultMap type="student" id="stuMap">
    <id property="id" column="id"/>
    <result property="name" column="name"/>
    <result property="age" column="age"/>
    <result property="tid" column="tid"/>

<!-- 如果关联一个对象，使用 association 标签，调用 teacher 中的查询，如果关联多个对象，使用 collection 标签 -->
// 老师查询中需要一个 Int 类型的参数，这里要通过 column 告诉他传入哪一列的值
    <association property="teacher" select="com.gjxaiou.mapper.TeacherMapper.selById"  column="tid">
    </association>
</resultMap>

<select id="selAll" resultMap="stuMap">
      select * from student
</select>
```


- 因为这里属性名和字段名相同，因此可以把上面代码简化成
```xml
<resultMap type="student" id="stuMap">
    <result column="tid" property="tid"/>
    <!-- 如果关联一个对象-->
    <association property="teacher"
select="com.gjxaiou.mapper.TeacherMapper.selById"
column="tid"></association>
</resultMap>

<select id="selAll" resultMap="stuMap">
      select * from student
</select>
```


###  （三）使用 resultMap 实现关联单个对象(联合查询方式)

- 实现只需要编写一个 SQL,在 StudentMapper.xml 中添加下面效果
  -  `<association/>` 只要装配 一个对象就用这个标签
  -  此时把 `<association/>` 小的<resultMap>看待
  -  `javaType`  属性:<association/>装配完后返回一个什么类型的对象.取值是一个类(或类的别名)

```xml
<resultMap type="Student" id="stuMap1">
    <id column="sid" property="id"/>
    <result column="sname" property="name"/>
    <result column="age" property="age"/>
    <result column="tid" property="tid"/>
    
    <association property="teacher" javaType="Teacher" >
    <id column="tid" property="id"/>
    <result column="tname" property="name"/>
    </association>
</resultMap>

<select id="selAll1" resultMap="stuMap1">
    select s.id sid,s.name sname,age age,t.id tid,t.name tname FROM student s left outer join teacher t on s.tid=t.id
</select>
```


###  （四）N+1 方式和联合查询方式对比

-  N+1：适用于需求不确定时；
-  联合查询：需求中确定查询时两个表一定都查询；

###  （五）N+1 名称由来

- 举例:学生中有 3 条数据
- 需求:查询所有学生信息及授课老师信息
- 需要执行的 SQL 命令
  - 查询全部学生信息：`select  *  from 学生`
  - 执行 3 遍： `select  *  from 老师 where  id=学生的外键`

- 使用多条 SQL 命令查询两表数据时,如果希望把需要的数据都查询出来,需要执行 N+1 条 SQL 才能把所有数据库查询出来；

- 缺点：效率低
- 优点：如果有的时候不需要查询学生是同时查询老师，只需要执行一个 `select  *  from  student;`

- 适用场景: 有的时候需要查询学生同时查询老师,有的时候只需要查询学生.

- 如果解决 N+1 查询带来的效率低的问题
  - 默认带的前提: 每次都是两个都查询；
  - 使用两表联合查询；

### （六）使用 `<resultMap>` 查询关联集合对象(N+1 方式)

 实现查询老师的时候，把关联的学生也查询出来
以为老师和学生是一对多的关系，因此查询结果是一个集合；
- 首先在 Teacher  实体类中添加 List<Student>
```java
public class Teacher {
    private int id;
    private String name;
    private List<Student> list;
    // 省略其他 get、set 方法
}
```

- 然后在 StudentMapper.xml  中添加通过 tid 查询
```xml
<select id="selByTid" parameterType="int" resultType="student">
      select * from student where tid=#{0}
</select>
```


- 然后在 TeacherMapper.xml  中添加查询全部
其中 `<collection/>` 是当属性是集合类型时使用的标签；
```xml
<resultMap type="teacher" id="mymap">
    <id column="id" property="id"/>
    <result column="name" property="name"/>

    <collection property="list" select="com.gjxaiou.mapper.StudentMapper.selByTid" column="id">
    </collection>    
</resultMap>

<select id="selAll" resultMap="mymap">
      select * from teacher
</select>
```


### （七）使用 `<resultMap>` 实现加载集合数据(联合查询方式)

- 首先在 teacherMapper.xml 中添加
mybatis 可以通过主键判断对象是否被加载过，因此不需要担心创建重复 Teacher
```xml
<resultMap type="teacher" id="mymap1">
    <id column="tid" property="id"/>
    <result column="tname" property="name"/>
    <collection property="list" ofType="student" >
        <id column="sid" property="id"/>
        <result column="sname" property="name"/>
        <result column="age" property="age"/>
        <result column="tid" property="tid"/>
    </collection>
</resultMap>

<select id="selAll1" resultMap="mymap1">
    select t.id tid,t.name tname,s.id sid,s.name sname,age,tid from teacher t LEFT JOIN student s on t.id=s.tid;
</select>
```


## 三、使用 Auto Mapping 结合别名实现多表查询
只能查询对象，查询结合只能使用上面的方法

- 只能使用多表联合查询方式，不能使用 N+1 方式
- 要求:查询出的列名和属性名相同.

###  实现方式

- 因为`.`在 SQL 是关键字符，因此两侧添加反单引号；
```xml
<select id="selAll" resultType="student">
    select t.id `teacher.id`, t.name `teacher.name`, s.id id, s.name name, age, tid from student s LEFT JOIN teacher t on t.id=s.tid
</select>
```



## 四、MyBatis 注解

**注解是写在类中的，不是XML中的，相当于将 实体类Mapper.xml 中的内容直接在对应的接口中实现即可，不在需要 实体类Mapper.xml 文件。**

- 注解：为了简化配置文件；
- Mybatis 的注解简化 `实体类Mapper.xml`  文件；
  - 如果涉及动态 SQL 依然使用 mapper.xml

- mapper.xml  和注解可以共存.

- 使用注解时要在 mybatis.xml 中<mappers>标签中使用下面两种方式之一进行配置：
  - `<package/>`
  - `<mapper  class=””/>`

可以在测试的时候使用下面：因为是接口：
```java
TeacherMapper  tm = session.getMapper(TeachterMapper.class)
List<Teacher> = tm.selAll(info)
```

### 具体的使用方式
下面语句直接在 TeacherMapper.java 接口中书写即可
- 实现查询
```java
@Select("select * from teacher")
List<Teacher> selAll();
```

- 实现新增
```java
@Insert("insert into teacher values(default,#{name})")
int insTeacher(Teacher teacher);
```

- 实现修改
```java
@Update("update teacher set name=#{name} where id=#{id}")
int updTeacher(Teacher teacher);
```

- 实现删除
```java
@Delete("delete from teacher where id=#{0}")
int delById(int id);
```

- 使用注解实现<resultMap>功能
以 N+1 举例

  - 首先在 StudentMapper 接口添加查询
```java
@Select("select * from student where tid=#{0}")
List<Student> selByTid(int tid);
```

-  然后在 TeacherMapper  接口添加下面代码
  -  @Results() 相当于<resultMap>
  -  @Result() 相当于<id/>或<result/>
    -  @Result(id=true) 相当与<id/>
  - @Many() 相当于<collection/>
  - @One() 相当于<association/>
```java
@Results(value={
    @Result(id=true,property="id",column="id"),
    @Result(property="name",column="name"),
@Result(property="list",column="id",many=@Many(select="com.gjxaiou.mapper.StudentMapper.selByTid"))
    })
    @Select("select * from teacher")
    List<Teacher> selTeacher();
```


## 五、运行原理

### （一）MyBatis 运行过程中涉及到的类

- `Resources`： MyBatis 中 IO 流的工具类
  - 作用是：加载配置文件
- `SqlSessionFactoryBuilder()` ： 构建器
  - 作用：创建 SqlSessionFactory 接口的实现类
- `XMLConfigBuilder` ： MyBatis 全局配置文件内容构建器类
  - 作用：负责读取流内容并转换为 JAVA  代码
- `Configuration`： 封装了全局配置文件所有配置信息
  - 全局配置文件内容存放在 Configuration 中
- `DefaultSqlSessionFactory` ： 是SqlSessionFactory 接口的实现类
- `Transaction` ： 事务类
  - 每一个 SqlSession 会带有一个 Transaction 对象.
- `TransactionFactory`： 事务工厂
  -  负责生产 Transaction
- `Executor` ： MyBatis 执行器
  - 作用:负责执行 SQL 命令
  - 相当于 JDBC 中 statement  对象(或 PreparedStatement 或 CallableStatement)
  -  默认的执行器 SimpleExcutor
  - 批量操作 BatchExcutor
  - 通过 openSession(参数控制)
- `DefaultSqlSession` ：是 SqlSession 接口的实现类
-  `ExceptionFactory` ：  MyBatis 中异常工厂

###  （二）流程图

![MyBatis执行流程图](FrameDay01_3%20Mybatis.resource/MyBatis%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B%E5%9B%BE.png)



### （三）文字解释

在 MyBatis 运行开始时需要先通过 Resources 加载全局配置文件.下面需要实例化 SqlSessionFactoryBuilder 构建器.帮助 SqlSessionFactory 接口实现类 DefaultSqlSessionFactory.

在实例化 DefaultSqlSessionFactory 之前需要先创建 XmlConfigBuilder 解析全局配置文件流,并把解析结果存放在 Configuration 中.之后把Configuratin 传递给 DefaultSqlSessionFactory.到此 SqlSessionFactory 工厂创建成功.

由 SqlSessionFactory  工厂创建 SqlSession.

每次创建 SqlSession 时,都需要由 TransactionFactory  创建 Transaction 对象, 同时还需要创建 SqlSession 的执行器 Excutor, 最后实例化DefaultSqlSession,传递给 SqlSession 接口.

根据项目需求使用 SqlSession 接口中的 API 完成具体的事务操作. 如果事务执行失败,需要进行 rollback 回滚事务.

如果事务执行成功提交给数据库.关闭 SqlSession

到此就是 MyBatis 的运行原理.(面试官说的.)