

# MySQL 必知必会 Chapter1-10

### 章一：了解 SQL

- 数据库软件称为**DBMS（数据库管理系统**）；例如 MySQL
- 数据库是通过 DBMS 创建和操纵的容器；
- **模式：（schema）是关于数据库和表布局及特性的信息**；
- 主键：（primary key）可以是一列或者一组列，其值能够唯一区分表中每一行；
  - **==表中列成为主键的条件==**：
    - 任意两行都不具有相同的主键值；
    - 每个行都必须有一个主键值（即主键值不能为 null）；

### 章四：检索数据
- SQL 语句不区分大小写；
- 使用`distinct`进行去重的时候，作用范围是**所有列**，不仅仅是前置它的列；
- 使用`select`检索的结果中，第一行为行 0，即从 0 开始计数；
- 使用`limit`限制显示行数，如果结果中没有足够的行，则只返回能返回的所有结果；

### 章五：排序检索数据
- `desc`关键字**只应用在直接位于其前面的列名**；
- 子句的位置：从前往后：from -> order by -> limit

### 章六：过滤数据
- `order by` 必须位于`where`之后；
- where 子句操作符：`= `、`<>`、`!=`、`<`、`<=`、`>`、`>=`、`between A and B`；
- MySQL 执行匹配时候默认不区分大小写；
- 空值检测：使用：`is null`

  注：null 为空值，与字段包含 0、空字符串以及空格是不一样的；

### 章七：数据过滤
- where 子句中使用 and 和 or 连接子句时候，and 在计算次序中的优先级更高，最好使用 `()` 进行分组操作；
- in 操作符：`where 列名 in(值 1，值 2，值 3...)`；
- in 操作符一般比 or 操作符清单执行更快；
- not 操作符：允许针对：`in、between、exists` 子句进行取反；

### 章八：用通配符进行过滤

- `like` 是谓词不是操作符
- 注意尾空格，例如使用`%hello`，注意文章中 hello 后面是有空格的，造成无法匹配，可以使用：`%hello%`或者使用函数进行去除空格；
- 通配符不会匹配 `null`；
- 不要在搜索模式的开始处使用通配符，会造成搜索变慢；


### 章九：用正则表达式进行搜索

- 正则表达式是用来匹配文本的特殊的串（字符集合）；
- MySQL 仅仅支持部分的正则表达式；

#### （一）基本字符匹配
- `REGEXP`后跟的东西作为正则表达式处理；
- 使用：`binary`来区分大小写；
- 使用功能：`|`来进行 OR 操作；
- 使用`[`和 `]`匹配几个字符之一；
```sql
where name REGEXP '1000' // 等效于：like '%1000%'
where name REGEXP '.000' // 其中.为匹配任意一个字符的通配符
where name regexp binary 'Hello' // 区分大小写匹配
where name regexp '1000|2000' // 只要匹配其中之一就可以返回
where name regexp '[123] Ton' // 表示1 Ton 、2 Ton、3 Ton之一
// 等价于：[1|2|3] Ton
where name regexp '[0-9]' // 匹配所有的数字，[a-z]表示匹配任意字母字符
where name regexp '\\.' // 表示匹配.,其他转义符 - 以及 |以及[]
// 其他空白元字符 \\f 换页； \\n 换行；\\r 回车；\\t 制表；\\v 纵向制表 \\\ 反斜杠\本身；
```

#### （二）匹配字符串
|类| 说明|
| --- | --- |
| [:alnum:]  |  任意字母和数字（同[a-zA-Z0-9]）  |
| [:alpha:]   | 任意字符（同[a-zA-Z]） |
| [:blank:]   | 空格和制表（同[\\t]） |
| [:cntrl:]   | ASCII控制字符（ASCII 0到31和127） |
| [:digit:]   | 任意数字（同[0-9]） |
| [:graph:]  |  与[:print:]相同，但不包括空格 |
| [:lower:]   | 任意小写字母（同[a-z]） |
| [:print:]  |  任意可打印字符 |
| [:punct:]  |  既不在[:alnum:]又不在[:cntrl:]中的任意字符 |
| [:space:]  |  包括空格在内的任意空白字符（同[\\f\\n\\r\\t\\v]） |
| [:upper:]  |  任意大写字母（同[A-Z]） |
| [:xdigit:]   | 任意十六进制数字（同[a-fA-F0-9]）|

#### （三）匹配多个实例

|元字符| 说明 |
|---|---|
|`*`  | 0个或多个匹配 |
|`+`  |      1个或多个匹配（等于{1,}）|
|`?`   |   0个或1个匹配（等于{0,1}），是匹配？前面任意字符的 0 次或者 1 次出现|
|{n}  |     指定数目的匹配 |
|{n,} |   不少于指定数目的匹配|
|{n,m} |  匹配数目的范围（m不超过255）|

`select prod_name from products where prod_name regexp '\\([0-9]sticks?\\)' order by prod_name;`  # 返回了'TNT (1 stick)'和'TNT (5 sticks)'
`select prod_name from products where prod_name regexp '[[:digit:]]{4}'order by prod_name;`  # [[:digit:]]{4}匹配连在一起的任意4位数字

#### （四）定位符
|元字符 | 说明|
|---|---|
|^  | 文本的开始 |
|$    |   文本的结尾 |
|[[:<:]]     |   词的开始 |
|[[:>:]]      |  词的结尾|

`select prod_name from products where prod_name regexp '^[0-9\\.]' order by prod_name;` #找出以一个数（包括以小数点开始的数）开始的所有产品
`select prod_name from products where prod_name regexp '[0-9\\.]' order by prod_name;`  #找出包括小数点和数字的所有产品

- `^`用法：在集合中，即使用`[`和`]`定义的，用来否认该集合，否则就是指串的开始处； `[^123]`表示匹配除了 1,2,3 之外的字符；

### 章十：创建计算字段

#### 拼接字段
- 使用`concat()`函数来拼接两个列；
实例：`select concat(vend_name,' (',vend_country,')') as XXX from vendors order by vend_name;`，这里是四个元素进行拼接，分别为列 `vend_name `、`(` 、`vend_country`、`)`
最终是查询结果是按照列拼接的这种格式。

- 去除空格：
  - 删除数据左侧多余空格 `ltrim()`
  - 删除数据两侧多余空格 `trim()`
  - 删除数据右侧多余空格 `rtrim()`
`select concat(rtrim(vend_name),' (',rtrim(vend_country),')') from vendors order by vend_name; `
- `AS` 赋予别名；
- 计算实例：`select quantity * item_price as expanded_price from orderitems where order_num = 20005;`  # 计算总价 expanded_price
- 支持的算术操作符：`+`和`-`和`*`和`/`

