---
tags:
- 数据库
- MySQL

style: summer
---
@toc

# JavaEEDay32数据库

# 全篇总结：2019-8-1
##  针对库：
  - 创库：`create database demo;`
  - 查库：
```sql
# 查看数据库中数据表
show databases;

# 查看数据库默认字符集等信息
show create database demo;
```
  - 删库：`drop database demo;`

## 针对表：
  - 创表：
```
# 进入要创建的数据库
use demo; 
# 创建表格
create table people( 
  id   tinyint  not null  primary key  auto_increment,
  name  char(5) not null,
  gender  char(1),
  score  decimal(4, 2),
  home  enum("江苏", "上海", "杭州", "苏州"),
  hobby  set("游泳", "打球", "跑步")
)engine = InnoDB  default charset = utf8;
```
  - 查表：
```
# 看包含哪些表格
use demo;
show tables;
# 查看表格,每个字段以及属性
desc people;
# 查看表格存储引擎以及字符集
show create table people; 
```
  - 删表：`drop table people;`
  - 改表：
    - 改表名：`rename table people to person;`或者`alter table people rename to  person;`

## 针对字段：
- 添加新字段：`alter table people add tel tinyint(11);`默认是添加到第一列；
- 在指定位置添加字段：`alter table people add tel tinyint(11) after gender;`在 gender 后添加一列；
- 修改字段数据类型：`alter table people modify gender unsingned int;`
- 修改字段名称与数据类型：`alter table people change gender sex int;`
- 删除字段：`alter table people drop gender;`
- 添加主键：`alter table people add primary key(id);`

## 针对内容：
  - 添加数据：
```
# 对部分字段添加数据
insert into people(name, gender, score, home, hobby) values("张三", "男", 98.23, 2, 3);
insert into people(name, gender, score, home, hobby) values("李四五", "女",99.00,"江苏", "游泳,打球");

#对所有字段添加数据
insert into people values(3, "王六", "男", 97);
```
  - 删除数据：
    - 删除整表中数据：`delete from people;`
    - 删除部分表中数据：`delete from people where id = 2;` 删除 id=2 这一行；
  - 修改数据：`update 表名 set stuAge = 12,stuName = "赵六" where stuId = 2;` 没有 where 限定则表中所有数据都会改变；


## 查表：
### 单表查询：
- 查看所有数据：`select * from 表名;`
- 只查某个表中某些字段数据：`select 字段1，字段2 from 表名;`
- 查询带有条件的数据（以字段 3 值>10 为例）：`select 字段1，字段2 from 表名 where 字段3>10;`
- 查询字段去重之后的数据：`select distinct 字段 from 表名;`
- 获取表中某些字段的值，并且改变展示的字段名称，同时多条限制条件；`select stuName as "姓名" ,stuAge as "年龄" from stuInfo where stuAge >= 20 and stuScore <90;`或者的话可以使用 or;
- 排序查询（默认升序）：`select * from stuInfo order by stuScore asc;`降序改为 desc;
- 多重条件查询：优先主条件，主条件相同按照附条件；`select stuName,stuAge from stuInfo order by stuAge asc, stuScore desc;`
- 模糊查询：
  - 查询 某个字段以某个字符结尾的数据：`select * from 表名 where stuName like "%看";` 其中%为通配符，表示 0 到 n 个字符；
  - 查询某个字段以 看 字结尾，且前面只有一个字符的数据：`select * from 表名 where stuName like "_看";` 其中`_`为通配符，表示 有且仅有 1 个字符；
  - 查询某个字段包含 看 字字符：`select * from 表名 where stuName like "%看%";`
- 模拟分页显示数据：
  - 只获取前 3 条数据：`select * from 表名 limit 3;`
  - 从某个数据开始，向后获取几个数据：`select * from 表名 limit 0, 5;`从 第 0 个数据开始，向后获取 5 个数据；
- 内置函数：不推荐使用，应该是从数据库获取数据，然后使用 Java 代码处理数据；
  - 获取最大值：`select max(stuAge) as "最大年龄" from stuInfo;` 最小值为：min,平均值为：avg
  - 获取总数：`select count(*) as "总数" from stuInfo where stuAge > 20;`中间的 as 和 where 都是可用可以不用；

### 多表查询：
#### 1. 一对一查询
man 表中有 manID   manName  girlID; girl 表中有 girlID  girlName;
- 通过 man 和 girl 两表中通过 grilID 进行匹配展示数据:
`select * from man ,girl where man.girlID = girl.girlID;`
另一种方式：内联查询：可以是实现多个表联合查询，其中 on 和 where 类似 【推荐使用】
`select * from man inner join girl on man.girlID = girl.girlID;`


#### 2. 一对多查询
father 表中有：fatherID fatherName; son 表中有：sonID  sonName  fatherID;
- 查询某一个父亲下的孩子：`select * from father inner join son on son.fatherID = father.fatherID where fatherName = "XXX";`
将上面使用别名进行简化书写：`select * from father f inner join son s on s.fatherID = f.fatherID where fatherName = "XXX";`


#### 3. 多对多查询
**多对多查询需要使用中间表格**
student 表格中有：stuID  stuName ;
中间表格 stuToCouse 中有：stuToCouseID stuID couseID
couse 表格中有：couseID couseName

- 查询某一个学生选了哪些课程：`select * from student s inner join stuToCouse sc on s.stuID = sc.stuID inner join couse c on sc.couseID = c.couseID where s.stuName = "XXX";`
- 查询某一课程有哪些学生选择：`select * from couse c inner join stuToCouse sc on c.couseID = sc.couseID inner join student s on sc.stuID = s.stuID where c.couseName = "XXXX";`







## 一、数据库的组成
- 数据库服务器：装有数据库软件的一个电脑
- 数据库：软件：MySQL、Oracle
- 数据表：一个表格，里面放着一条一条的数据，类似于 Excel
- 字段：表示该数据时什么数据，例如：姓名、年龄、性别；
- 数据行：一个完整的数据

## 二、数据库分类
关系型数据库，非关系型数据库
- 关系型数据库：MySQL

## 三、SQL 语句
结构化查询语句：Structured Query Language


## 四、SQL 语句分类
DDL：数据定义
DML：数据操作
DQL：数据查询
DCL：数据控制
DTL：事务处理

CRUD 对应于：
create
read
update
delete


## 五、从命令行连接数据库
需要：数据库服务器地址  数据库访问用户名  当前访问用户名的密码  （本地连接 第一个可以不输入）
例如完整的数据库连接命名：`mysql -hlocalhost -uroot -p12345`
建议使用方式：`mysql -hlocalhost -uroot -p`然后输入的是加密的密码
- sql 语句以`;`结尾
- 退出命名：`quit`或者`exit`
- 清楚本次错误输入：`\c`



## 六、基本命令

| 命令 | 含义 |
|:--- | :--- |
show databases|查询所有的数据库  
create database hello|创建数据库 hello
drop database hello|删除数据库 hello（不可逆） 
use  hello|使用数据库 hello  （使用下面的表要先进入数据库）


- 创建数据表：`create table 表名(字段名 数据类型，字段名 数据类型);`
```sql
create table stuInfo(
  //格式为：字段名  字段数据类型
  //varchar(30)相比char是可变长的，括号里面数字为可以放置的字节长度
  //tinyint只占一个字节，更加节省空间
  stuID int,
  stuName varchar(30),
  stuGender tinyint,
  stuAge tinyint
);
```

- 删除数据表
`drop table stuInfo;`
- **查看表的详细信息**：`desc 表名;`实际命令为：`show columns from 表名;`
- 创建数据库的简要描述：`show create database stuInfo;`显示数据库默认字符集；（该库必须是提前创建好的）
- 创建数据表的简要描述：`show create table hello;` 显示数据表的 engine 和 charset
- 默认的存储引擎应该设置为:InnoDB，字符集设置为：utf8 
- 修改默认存储引擎和字符集的方式：
  - 方式一：只该表当前表的设置：
```sql
create table test(
  name varchar(30);
  age  tinyint;
)engine = InnoDB default charset = utf8;
```
  - 方式二：修改全局配置
  在 MySQL 的配置文件：`my.ini`中设置默认的存储引擎和字符集；


- 修改表格里面内容：
  - 添加新字段：`alter table stuInfo add StuDesc text after stuAge;` 在 stuAge 字段的后面增加 text 类型的新字段 stuDesc，如果在表的最后增加数据，不需要使用 after；
  - 修改老字段的数据类型：`alter tabel stuInfo modify stuName char(30);` 将 stuName 字段的数据类型改为:`char(30)`;
  - 删除已有字段：`alter table stuInfo drop stuDesc;` 删除 stuDesc 字段；
  - 同时修改字段名和数据类型：`alter table stuInfo change stuGender stuSex char(1);`


- 插入数据：属于 DML 操作
  - 选中一些字段添加数据：`insert into stuInfo(stuId, stuName) values(2,"张三");`剩余字段值按照默认值处理；
  - 全部字段添加数据：`insert into stuInfo values(3,"李四", "男" ,23,99);`
  不需要将 stuInfo 中所有字段都列出来：`insert into stuInfo(stuId, stuName,stuSex,stuAge,stuScore) values(1,"王五","男",22,98);`


### (一)MySQL 常用的数据类型
MySQL 5.0 以上版本中：
一个汉字占多少长度和编码有关：
utf8 ：一个汉字 = 3 个字节；
gbk ：一个汉字 = 2 个字节；
varchar(n)表示 n 个字符，无论汉字和英文，MySQL 都能存入 n 个字符，仅是实际字节长度有区别；

- 数值类型：

| 类型 | 大小(字节) | 范围（有符号） | 范围（无符号） | 用途 |
|---|---|---|---|---|
| tinyint | 1  | (-128，127) | (0，255) | 小整数值 |
| smallint | 2  | (-32 768，32 767) | (0，65 535) | 大整数值 |
| mediumint | 3  | (-838.8 万，838.8 万) | (0，1677.7 万) | 大整数值 |
| int 或 integer | 4  | (-21.4 亿，21.4 亿) | (0，42.9 亿) | 大整数值 |
| bigint | 8  | (-92 京，92 京) | (0，184 京) | 极大整数值 |
| folat | 4  | (-3.40E+38，-1.17E-38)，0，(1.175E-38，3.402E+38) | 0，(1.175 E-38，3.402E+38) | 单精度浮点数值 |
| double | 8  | (-1.797E+308，-2.225E-308)，0，(2.225E-308，1.797E+308) | 0，(2.225E-308，1.797E+308) | 双精度浮点数值 |
| decimal（以decimal(MD)为例）|若 M>D，为M+2否则为D+2 | 依赖于M和D的值 | 依赖M和D的值 | 小数值 |

-  日期和时间类型
表示时间值的日期和时间类型为DATETIME、DATE、TIMESTAMP、TIME和YEAR。
每个时间类型有一个有效值范围和一个"零"值，当指定不合法的MySQL不能表示的值时使用"零"值。

TIMESTAMP类型有专有的自动更新特性，将在后面描述。

| 类型 | 大小(字节) | 范围 | 格式 | 用途 |
|---|:---:|---|---|----|
| DATE | 3 | 1000-01-01/9999-12-31 | YYYY-MM-DD | 日期值 |
| TIME | 3 | '-838:59:59'/'838:59:59' | HH:MM:SS | 时间值或持续时间 |
| YEAR | 1 | 1901/2155 | YYYY | 年份值 |
| DATETIME | 8 | 1000-01-01 00:00:00/9999-12-31 23:59:59 | YYYY-MM-DD HH:MM:SS | 混合日期和时间值 |
| TIMESTAMP | 4 | 1970-01-01 00:00:00/2038 结束时间是第 **2147483647** 秒，北京时间 **2038-1-19 11:14:07**，格林尼治时间 2038年1月19日 凌晨 03:14:07| YYYYMMDD HHMMSS | 混合日期和时间值，时间戳 |



## 字符串类型

字符串类型指CHAR、VARCHAR、BINARY、VARBINARY、BLOB、TEXT、ENUM和SET。该节描述了这些类型如何工作以及如何在查询中使用这些类型。

| 类型 | 大小 | 用途 |
|---|---|---|
| CHAR | 0-255字节 | 定长字符串 |
| VARCHAR | 0-65535 字节 | 变长字符串 |
| TINYBLOB | 0-255字节 | 不超过 255 个字符的二进制字符串 |
| TINYTEXT | 0-255字节 | 短文本字符串 |
| BLOB | 0-65 535字节 | 二进制形式的长文本数据 |
| TEXT | 0-65 535字节 | 长文本数据 |
| MEDIUMBLOB | 0-16 777 215字节 | 二进制形式的中等长度文本数据 |
| MEDIUMTEXT | 0-16 777 215字节 | 中等长度文本数据 |
| LONGBLOB | 0-4 294 967 295字节 | 二进制形式的极大文本数据 |
| LONGTEXT | 0-4 294 967 295字节 | 极大文本数据 |

CHAR 和 VARCHAR 类型类似，但它们保存和检索的方式不同。它们的最大长度和是否尾部空格被保留等方面也不同。在存储或检索过程中不进行大小写转换。

BINARY 和 VARBINARY 类似于 CHAR 和 VARCHAR，不同的是它们包含二进制字符串而不要非二进制字符串。也就是说，它们包含字节字符串而不是字符字符串。这说明它们没有字符集，并且排序和比较基于列值字节的数值值。

BLOB 是一个二进制大对象，可以容纳可变数量的数据。有 4 种 BLOB 类型：TINYBLOB、BLOB、MEDIUMBLOB 和 LONGBLOB。它们区别在于可容纳存储范围不同。

有 4 种 TEXT 类型：TINYTEXT、TEXT、MEDIUMTEXT 和 LONGTEXT。对应的这 4 种 BLOB 类型，可存储的最大长度不同，可根据实际情况选择。
一般情况下图片、视频以及大体积文本会先上传到服务器指定保存该类型的文件夹下面，保存时候按照时间和随机数的关系给文件重命名（防止文件重名），同时将保存文件的服务器地址放入到数据库中，之后便可以通过数据库查询到视频地址，通过地址直接访问；

- 其他数据类型
  - timestamp：时间戳，默认可以设置 current_timestamp,可以用于记录当前时间；
  - enum:枚举，一般用于处理互斥的关系，例如：性别、籍贯等，即是单选。同时每一个选项表示的数值是从 1 开始的；

```sql
#首先创建一个带有枚举类型的数据表
create table test(
  testID int(4) not null，
  enumValue enum("南京","上海","杭州")
);
#上面南京对应的枚举值为1，杭州对应的枚举值为3

#采用使用枚举里面的具体数据进行插入数据
insert into test(testID,enumValue) values(1,"南京");
insert into test(testID,enumValue) values(2,"上海");

#采用使用枚举数据的特征进行插入数据，每一个枚举类型都是独立特有的，
#值是不同的，采用的是枚举数值；
insert into test(testID,enumValue) values(3,1);
insert into test(testID,enumValue) values(4,3);

#插入的枚举类型数值不能超过枚举的范围，下列语句错误；
insert into test(testID,enumValue) values(5,4);
```

- set：集合，处理并列关系。例如：多选
```sql
create table testSet(
#将testID设置为非空，且是自动增长的主键
  testID int(4) not null primary key auto_increment, 
  likes set("Reading","swimming","running")
);

insert into testSet(likes) values("Reading,swimming");
insert into testSet(likes) values(3); 
#set值以二进制表示：1 2 4 8 16  ；要的值加起来就行
```



### （二）查询语句 DQL 

- 查看所有数据：`select * from 表名;`
- 只查某个表中某些字段数据：`select 字段1，字段2 from 表名;`
- 查询带有条件的数据（以字段值>10 为例）：`select 字段1，字段2 from 表名 where 字段3>10;`
- 查询字段去重之后的数据：`select distinct 字段 from 表名;`
- 获取表中某些字段的值，并且改变展示的字段名称，同时多条限制条件；`select stuName as "姓名" ,stuAge as "年龄" from stuInfo where stuAge >= 20 and stuScore <90;`或者的话可以使用 or;

- 排序查询（默认升序）：`select * from stuInfo order by stuScore asc;`降序改为 desc;
- 多重条件查询：优先主条件，主条件相同按照附条件；`select stuName,stuAge from stuInfo order by stuAge asc, stuScore desc;`
- 模糊查询：
  - 查询 某个字段以某个字符结尾的数据：`select * from 表名 where stuName like "%看";` 其中%为通配符，表示 0 到 n 个字符；
  - 查询某个字段以 看 字结尾，且前面只有一个字符的数据：`select * from 表名 where stuName like "_看";` 其中`_`为通配符，表示 有且仅有 1 个字符；
  - 查询某个字段包含 看 字字符：`select * from 表名 where stuName like "%看%";`

- 模拟分页显示数据：
  - 只获取前 3 条数据：`select * from 表名 limit 3;`
  - 从某个数据开始，向后获取几个数据：`select * from 表名 limit 0, 5;`从 第 0 个数据开始，向后获取 5 个数据；

- 内置函数：不推荐使用，应该是从数据库获取数据，然后使用 Java 代码处理数据；
  - 获取最大值：`select max(stuAge) as "最大年龄" from stuInfo;` 最小值为：min,平均值为：avg
  - 获取总数：`select count(*) as "总数" from stuInfo where stuAge > 20;`中间的 as 和 where 都是可用可以不用；




### （三）删除数据
 - 删除数据表中的所有数据行：`delete from 表名;`
 - 删除带有条件的数据行:`delete from 表名 where stuId = 3;`where 后面的条件可以是：`>` `<` `>=` `<=` `=` `!=` `<>` `between and`
 - 使用 truncate 会清空整个数据表，但是不会影响数据表结构，同时会影响原来的自增条件，会从 1 开始；`truncate table 表名;`


### （四）修改更新数据
例如更新一些值为空的字段数据：`update 表名 set stuAge = 12,stuName = "赵六" where stuId = 2;` 注意增加 where 语句，否则会将这个表数据进行更新；



### （五）连表查询

#### 1. 一对一查询
man 表中有 manID   manName  girlID; girl 表中有 girlID  girlName;
- 通过 man 和 girl 两表中通过 grilID 进行匹配展示数据:
`select * from man ,girl where man.girlID = girl.girlID;`
另一种方式：内联查询：可以是实现多个表联合查询，其中 on 和 where 类似 【推荐使用】
`select * from man inner join girl on man.girlID = girl.girlID;`


#### 2. 一对多查询
father 表中有：fatherID fatherName; son 表中有：sonID  sonName  fatherID;
- 查询某一个父亲下的孩子：`select * from father inner join son on son.fatherID = father.fatherID where fatherName = "XXX";`
将上面使用别名进行简化书写：`select * from father f inner join son s on s.fatherID = f.fatherID where fatherName = "XXX";`


#### 3. 多对多查询
**多对多查询需要使用中间表格**
student 表格中有：stuID  stuName ;
中间表格 stuToCouse 中有：stuToCouseID stuID couseID
couse 表格中有：couseID couseName

- 查询某一个学生选了哪些课程：`select * from student s inner join stuToCouse sc on s.stuID = sc.stuID inner join couse c on sc.couseID = c.couseID where s.stuName = "XXX";`
- 查询某一课程有哪些学生选择：`select * from couse c inner join stuToCouse sc on c.couseID = sc.couseID inner join student s on sc.stuID = s.stuID where c.couseName = "XXXX";`



![总结]($resource/%E6%80%BB%E7%BB%93.png)

