# FrameDay01_3 Mybatis

**主要内容**
* Auto Mapping 单表实现(别名方式)
* <resultMap> 实现单表配置
* 单个对象关联查询 (N+1,外连接) 集合对象关联查询
* 注解开发
* MyBatis 运行原理


## 一、MyBatis 实现多表查询

### （一）Mybatis 实现多表查询方式

一共三种方式
- 业务装配：对两个表编写单表查询语句,在业务(Service)把查询的两个结果进行关联；
- 使用Auto  Mapping 特性,在实现两表联合查询时通过别名完成映射；
- 使用 MyBatis 的 <resultMap> 标签进行实现；

### （二）多表查询时,类中包含另一个类的对象的分类

- 单个对象
- 集合对象

## 二、resultMap 标签

相当于现在直接设置映射关系，进行两者（SQL 查询结果和实体类之间）的匹配。

- <resultMap>标签写在 实体类 Mapper.xml  中,由程序员控制SQL 查询结果与实体类的映射关系；

- 默认 MyBatis 使用 Auto  Mapping 特性进行映射：即保持查询的数据库中的列名和实体类中的属性名相同即可；

- 使用<resultMap>标签时,<select>标签不写 resultType  属性,而是使用 resultMap 属性来引用<resultMap>标签.

### （一）使用 resultMap 实现单表映射关系

-  首先进行数据库设计
示例：数据库表 teacher 中两个字段： id、name

- 然后进行实体类设计
```java
public class Teacher{
   private int id1;
   private String name;
}
```

- 然后实现 TeacherMapper.xml  代码
```xml
<!-- 其中 type 为返回值类型-->
<resultMap type="teacher" id="mymap">
    <!-- 主键使用id 标签配置映射关系-->
    <id column="id" property="id1" />
    
    <!-- 其他列使用result 标签配置映射关系-->
    <result column="name" property="name1"/>
</resultMap>
```

返回值类型

<resultMap type=_"__t__ea__c__h__e__r__"_id=_"__m__y__m__ap__"_>

<!--  主键使用  id  标签配置映射关系  -->

数据库 实体类

<id column=_"__i__d__"_property=_"__i__d1__"_/>

<!--  其他列使用  result  标签配置映射关系  -->

<result column=_"__na__m__e__"_property=_"__na__m__e1__"_/>

</resultMap>

这个代码对应于：

<select id = "selAll" resultType = "teacher"> select * from teacher

</select>

![文本框: <select  id="selAll"  resultMap="mymap"> select  *  from  teacher
</select>
](file:///C:/Users/gjx16/AppData/Local/Temp/msohtmlclip1/01/clip_image004.gif)

4. 使用 resultMap 实现关联单个对象(N+1 方式)

4.1 N+1 查询方式：先查询出某个表的全部信息,根据这个表的信息查询另一个表的信息.

4.2 与业务装配的区别:

4.3.1 在 service 里面写的代码,由 mybatis 完成装配

4.3 实现步骤:

4.3.1     |  |
|  | ![文本框: public  class  Student  { private  int  id; private  String  name; private  int  age; private  int  tid;
private  Teacher  teacher;
](file:///C:/Users/gjx16/AppData/Local/Temp/msohtmlclip1/01/clip_image005.gif) | 
在 Student 实现类中包含了一个 Teacher  对象

4.3.2 在 TeacherMapper  中提供一个查询

![文本框: <select  id="selById"  resultType="teacher" parameterType="int">
select  *  from  teacher  where  id=#{0}

</select>
](file:///C:/Users/gjx16/AppData/Local/Temp/msohtmlclip1/01/clip_image006.gif)

4.3.3 在 StudentMapper 中

4.3.3.1 <association> 表示当装配一个对象时使用

4.3.3.2 property: 是对象在类中的属性名

4.3.3.3 select:通过哪个查询查询出这个对象的信息

4.3.3.4 column: 把当前表的哪个列的值做为参数传递给另

一个查询

4.3.3.5 ![](file:///C:/Users/gjx16/AppData/Local/Temp/msohtmlclip1/01/clip_image007.gif)大前提使用 N+1 方式.时：如果列名和属性名相同可以不配置,使用 Auto  mapping 特性.但是 mybatis 默认只会给列装配一次

<resultMap type=_"__s__t__ude__n__t__"_id=_"__s__t__u__M__a__p__"_>

<id property=_"id"_ column=_"id"_/>

<result property=_"name"_ column=_"name"_/>

<result property=_"age"_ column=_"age"_/>

<result property=_"tid"_ column=_"tid"_/>

<!-- 如果关联一个对象，使用association标签，调用teacher中的查询，如果关联多个对象，使用collection 标签 -->

**<association property=_"teacher"_** **select=_"com.bjsxt.mapper.TeacherMapper.selById"_ column=_"tid"_></**

**association>**//  老师查询中需要一个Int类型的参数，这里要告诉他传入哪一列的值 </resultMap>

<select id=_"__s__e__l__A__l__l__"_resultMap=_"__s__t__u__M__a__p__"_> select *  from  student

</select>

4.3.3.6 把上面代码简化成

![文本框: <resultMap  type="student"  id="stuMap">

<result  column="tid"  property="tid"/>
<!--  如果关联一个对象  -->
<association  property="teacher" select="com.bjsxt.mapper.TeacherMapper.selById" column="tid"></association>
</resultMap>

<select  id="selAll"  resultMap="stuMap"> select  *  from  student
</select>
](file:///C:/Users/gjx16/AppData/Local/Temp/msohtmlclip1/01/clip_image008.gif)

5. 使用 resultMap 实现关联单个对象(联合查询方式)

5.1 只需要编写一个 SQL,在 StudentMapper 中添加下面效果

5.1.1 <association/>只要专配一个对象就用这个标签

5.1.2 此时把<association/>小的<resultMap>看待

5.1.3 javaType  属性:<association/>专配完后返回一个什么类型的对象.取值是一个类(或类的别名)

![文本框: <resultMap  type="Student"  id="stuMap1">

<id  column="sid"  property="id"/>

<result  column="sname"  property="name"/>

<result  column="age"  property="age"/>

<result  column="tid"  property="tid"/>

<association  property="teacher"
](file:///C:/Users/gjx16/AppData/Local/Temp/msohtmlclip1/01/clip_image009.gif)

![文本框: javaType="Teacher"  >

<id  column="tid"  property="id"/>

<result  column="tname"  property="name"/>

</association>

</resultMap>

<select  id="selAll1"  resultMap="stuMap1"> select  s.id  sid,s.name  sname,age  age,t.id
tid,t.name  tname  FROM  student  s  left  outer  join  teacher t  on  s.tid=t.id
</select>
](file:///C:/Users/gjx16/AppData/Local/Temp/msohtmlclip1/01/clip_image010.gif)

6. N+1 方式和联合查询方式对比

6.1 N+1:需求不确定时.

6.2 联合查询:需求中确定查询时两个表一定都查询.

7. N+1 名称由来

7.1 举例:学生中有 3 条数据

7.2 需求:查询所有学生信息级授课老师信息

7.3 需要执行的 SQL 命令

7.3.1 查询全部学生信息:select  *  from 学生

7.3.2 执行 3 遍 select  *  from 老师 where  id=学生的外键

7.4 使用多条 SQl 命令查询两表数据时,如果希望把需要的数据都查询出来,需要执行 N+1 条 SQl 才能把所有数据库查询出来.

7.5 缺点:

7.5.1 效率低

7.6 优点:

7.6.1 如果有的时候不需要查询学生是同时查询老师.只需要执行一个 select  *  from  student;

7.7 适用场景: 有的时候需要查询学生同时查询老师,有的时候只需要查询学生.

7.8 如果解决 N+1 查询带来的效率低的问题

7.8.1 默认带的前提: 每次都是两个都查询.

7.8.2 使用两表联合查询.

三**.**使用**<resultMap>**查询关联集合对象**(N+1)**

# 实现查询老师的时候，把关联的学生也查询出来

1.     |  |
|  | ![文本框: public  class  Teacher  { private  int  id; private  String  name;
private  List<Student>  list;
](file:///C:/Users/gjx16/AppData/Local/Temp/msohtmlclip1/01/clip_image011.gif) | 
在 Teacher  中添加 List<Student>

2. 在 StudentMapper.xml  中添加通过 tid 查询

![文本框: <select  id="selByTid"  parameterType="int" resultType="student">
select  *  from  student  where  tid=#{0}

</select>
](file:///C:/Users/gjx16/AppData/Local/Temp/msohtmlclip1/01/clip_image012.gif)

3. 在 TeacherMapper.xml  中添加查询全部

 |  |
|  | ![文本框: <resultMap  type="teacher"  id="mymap">

<id  column="id"  property="id"/>

<result  column="name"  property="name"/>

<collection  property="list" select="com.bjsxt.mapper.StudentMapper.selByTid" column="id"></collection>
</resultMap>

<select  id="selAll"  resultMap="mymap"> select  *  from  teacher
</select>
](file:///C:/Users/gjx16/AppData/Local/Temp/msohtmlclip1/01/clip_image013.gif) | 
3.1 <collection/> 当属性是集合类型时使用的标签.

四**.**使用**<resultMap>** 实现加载集合数据**(**联合查询方式**)**

1.在 teacherMapper.xml 中添加

1.1 mybatis 可以通过主键判断对象是否被加载过.

1.2     |  |
|  | ![文本框: <resultMap  type="teacher"  id="mymap1">

<id  column="tid"  property="id"/>

<result  column="tname"  property="name"/>
](file:///C:/Users/gjx16/AppData/Local/Temp/msohtmlclip1/01/clip_image014.gif) | 
不需要担心创建重复 Teacher

![文本框: <collection  property="list"  ofType="student"  >

<id  column="sid"  property="id"/>

<result  column="sname"  property="name"/>

<result  column="age"  property="age"/>

<result  column="tid"  property="tid"/>

</collection>

</resultMap>

<select  id="selAll1"  resultMap="mymap1">

select  t.id  tid,t.name  tname,s.id  sid,s.name sname,age,tid  from  teacher  t  LEFT  JOIN  student  s  on t.id=s.tid;
</select>
](file:///C:/Users/gjx16/AppData/Local/Temp/msohtmlclip1/01/clip_image015.gif)

五**.**使用 **Auto Mapping** 结合别名实现多表查

# 询**.**只能查询对象，查询结合只能使用上面的方法

5.1 只能使用多表联合查询方式.

不能使用N+1

5.2 要求:查询出的列别和属性名相同.

5.3 实现方式

 |  |
|  | ![](file:///C:/Users/gjx16/AppData/Local/Temp/msohtmlclip1/01/clip_image016.gif) | 
5.3.1 .在 SQL 是关键字符,两侧添加反单引号

![文本框: select  t.id  `teacher.id`,t.name

`teacher.name`,s.id  id,s.name  name,age,tid

from  student  s  LEFT  JOIN  teacher  t  on  t.id=s.tid

</select>
](file:///C:/Users/gjx16/AppData/Local/Temp/msohtmlclip1/01/clip_image017.gif)

六**.MyBatis** 注解

注解是写在类中的，不是XML中的

1. 注解:为了简化配置文件.

2. Mybatis 的注解简化 mapper.xml  文件.

2.1 如果涉及动态 SQL 依然使用 mapper.xml

3. mapper.xml  和注解可以共存.

4. 使用注解时 mybatis.xml 中<mappers>使用下面两种方式

在mybatis中可以使用class = "/.....TeacherMapper."

4.1 <package/>

4.2 <mapper  class=””/>

测试的时候使用：因为是接口：TeacherMapper  tm = session.getMapper(TeachterMapper.class)

5. 实现查询

list<> = tm.sellAll;

![文本框: @Select("select  *  from  teacher")

List<Teacher>  selAll();
](file:///C:/Users/gjx16/AppData/Local/Temp/msohtmlclip1/01/clip_image018.gif)

6.     |  |
|  | ![文本框: @Insert("insert  into  teacher values(default,#{name})")
int  insTeacher(Teacher  teacher);
](file:///C:/Users/gjx16/AppData/Local/Temp/msohtmlclip1/01/clip_image019.gif) | 
实现新增

7. 实现修改

![文本框: @Update("update  teacher  set  name=#{name}  where id=#{id}")
int  updTeacher(Teacher  teacher);
](file:///C:/Users/gjx16/AppData/Local/Temp/msohtmlclip1/01/clip_image020.gif)

8.     |  |
|  | ![文本框: @Delete("delete  from  teacher  where  id=#{0}")

int  delById(int  id);
](file:///C:/Users/gjx16/AppData/Local/Temp/msohtmlclip1/01/clip_image021.gif) | 
实现删除

9. 使用注解实现<resultMap>功能

9.1 以 N+1 举例

9.2     |  |
|  | ![文本框: @Select("select  *  from  student  where  tid=#{0}")

List<Student>  selByTid(int  tid);
](file:///C:/Users/gjx16/AppData/Local/Temp/msohtmlclip1/01/clip_image022.gif) | 
在 StudentMapper 接口添加查询

9.3 在 TeacherMapper  接口添加

下面代码

9.3.1 @Results() 相当于<resultMap>

9.3.2 @Result() 相当于<id/>或<result/>

9.3.2.1 @Result(id=true) 相当与<id/>

9.3.3 @Many() 相当于<collection/>

9.3.4  |  |
|  | ![文本框: @Results(value={

@Result(id=true,property="id",column="id"), @Result(property="name",column="name"),

@Result(property="list",column="id",many=@Many(sele
](file:///C:/Users/gjx16/AppData/Local/Temp/msohtmlclip1/01/clip_image023.gif) | 
@One() 相当于<association/>

![文本框: ct="com.bjsxt.mapper.StudentMapper.selByTid"))

})
@Select("select  *  from  teacher") List<Teacher>  selTeacher();
](file:///C:/Users/gjx16/AppData/Local/Temp/msohtmlclip1/01/clip_image024.gif)

七**.** 运行原理

1. 运行过程中涉及到的类

1.1 Resources MyBatis 中 IO 流的工具类

1.1 加载配置文件

1.2 SqlSessionFactoryBuilder() 构建器

1.2.1 作用:创建 SqlSessionFactory 接口的实现类

1.3 XMLConfigBuilder MyBatis 全局配置文件内容构建器类

1.3.1 作用负责读取流内容并转换为 JAVA  代码.

1.4 Configuration 封装了全局配置文件所有配置信息.

1.4.1 全局配置文件内容存放在 Configuration 中

1.5 DefaultSqlSessionFactory 是SqlSessionFactory 接口的实现类

1.6 Transaction  事务类

16.1 每一个 SqlSession 会带有一个 Transaction 对象.

1.7 TransactionFactory  事务工厂

1.7.1 负责生产 Transaction

1.8 Executor MyBatis 执行器

1.8.1 作用:负责执行 SQL 命令

1.8.2 相当于 JDBC 中 statement  对象(或 PreparedStatement

或 CallableStatement)

1.8.3 默认的执行器 SimpleExcutor

1.8.4 批量操作 BatchExcutor

1.8.5 通过 openSession(参数控制)

1.9 DefaultSqlSession 是 SqlSession 接口的实现类

1.10 ExceptionFactory  MyBatis 中异常工厂

2. 流程图

![](file:///C:/Users/gjx16/AppData/Local/Temp/msohtmlclip1/01/clip_image030.jpg)

90 旰巴咒卫动节蹈噢

Resources加载全局配置文件

实例化SqlSessionfactoryBuilder构建器由XMlConfi的uilder解析配置文件流

把配置信息存放在Configura tion中

实例化SqlSessionfactory实现类Default SqlSessionFactory由TransactionFactory创建一个Transaction事务对象

创建执行器Excu tor

![实现CURD](file:///C:/Users/gjx16/AppData/Local/Temp/msohtmlclip1/01/clip_image031.gif)创建SqlSession接口实现类Default  SqlSessi  n

![关闭](file:///C:/Users/gjx16/AppData/Local/Temp/msohtmlclip1/01/clip_image032.gif)事务提交

![](file:///C:/Users/gjx16/AppData/Local/Temp/msohtmlclip1/01/clip_image034.jpg)

3.文字解释

在 MyBatis 运行开始时需要先通过 Resources 加载全局配置文件.下面需要实例化 SqlSessionFactoryBuilder 构建器.帮助 SqlSessionFactory 接口实现类 DefaultSqlSessionFactory.

在实例化 DefaultSqlSessionFactory 之前需要先创建 XmlConfigBuilder 解析全局配置文件流,并把解析结果存放在 Configuration 中.之后把Configuratin 传递给 DefaultSqlSessionFactory.到此 SqlSessionFactory 工厂创建成功.

由 SqlSessionFactory  工厂创建 SqlSession.

每次创建 SqlSession 时,都需要由 TransactionFactory  创建 Transaction 对象, 同时还需要创建 SqlSession 的执行器 Excutor, 最后实例化DefaultSqlSession,传递给 SqlSession 接口.

根据项目需求使用 SqlSession 接口中的 API 完成具体的事务操作. 如果事务执行失败,需要进行 rollback 回滚事务.

如果事务执行成功提交给数据库.关闭 SqlSession

到此就是 MyBatis 的运行原理.(面试官说的.)