

# MySQL 必知必会

[TOC]

### 一、检索数据
- SQL 语句不区分大小写；

- 使用 `distinct` 进行去重的时候，作用范围是**所有列**，不仅仅是前置它的列；

    即保证 distinct 后的所有列组合的结果是唯一的，不保证每列的结果是唯一的；

- 使用 `select` 检索的结果中，第一行为行 0，即从 0 开始计数；

- 使用 `limit` 限制显示行数，如果结果中没有足够的行，则只返回能返回的所有结果；

- `desc` 关键字**只应用在直接位于其前面的列名**；

- 子句的位置：从前往后：from -> order by -> limit

### 二、过滤数据
- `order by` 必须位于`where`之后；

- where 子句操作符：`= `、`<>`、`!=`、`<`、`<=`、`>`、`>=`、`between A and B`；

    其中 `between A  and  B ` 是包括 A B的，即相当于 `>= A and <= B`。

- **MySQL 执行匹配时候默认不区分大小写；**

- 空值检测：使用：`is null`

  注：null 为空值，与字段包含 0、空字符串以及空格是不一样的；

- where 子句中使用 and 和 or 连接子句时候，and 在计算次序中的优先级更高，最好使用 `()` 进行分组操作；

- in 操作符：`where 列名 in(值 1,值 2,值 3...)`；

- in 操作符一般比 or 操作符清单执行更快；

- **not 操作符：允许针对：`in、between、exists` 子句进行取反；**

### 三、用通配符进行过滤

- `like` 是谓词不是操作符，其中谓词主要包括：IN、BETWEEN、LIKE，逻辑操作符主要包括：OR 和 AND。
- 注意尾空格，例如使用 `%hello`，注意文章中 hello 后面是有空格的，造成无法匹配，可以使用：`%hello%` 或者使用函数进行去除空格；
- **通配符不会匹配 `null`；**
- 不要在搜索模式的开始处使用通配符，会造成搜索变慢；


### 四、用正则表达式进行搜索

- 正则表达式是用来匹配文本的特殊的串（字符集合）；

    MySQL 仅仅支持部分的正则表达式；

#### （一）基本字符匹配
- `REGEXP`后跟的部分作为正则表达式处理；
- 使用：`binary`来区分大小写；
- 使用功能：`|`来进行 OR 操作；
- 使用`[`和 `]`匹配几个字符之一；
```sql
where name REGEXP '1000' // 等效于：like '%1000%'
where name REGEXP '.000' // 其中.为匹配任意一个字符的通配符
where name REGEXP binary 'Hello' // 区分大小写匹配
where name REGEXP '1000|2000' // 只要匹配其中之一就可以返回
where name REGEXP '[123] Ton' // 表示1 Ton 、2 Ton、3 Ton之一
// 等价于：[1|2|3] Ton
where name REGEXP '[0-9]' // 匹配所有的数字，[a-z]表示匹配任意字母字符
where name REGEXP '\\.' // 表示匹配.,其他转义符 - 以及 |以及[]
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
|`*`  | 0 个或多个匹配 |
|`+`  | 1 个或多个匹配（等于 `{1,}`） |
|`?`   | 0 个或 1 个匹配（等于{0,1}），是匹配？前面任意字符的 0 次或者 1 次出现 |
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

### 五、创建计算字段

- 使用 `concat()` 函数来拼接两个列
  `SELECT concat(vend_name,' (',vend_country,')') as XXX FROM vendors ORDER BY vend_name;`

  对四个元素进行拼接，分别为列 `vend_name `、`(` 、`vend_country`、`)`。最终是查询结果是按照列拼接的这种格式。

- 去除空格：

  删除数据左/右/两侧多余空格 `ltrim()`、`trim()`、`rtrim()`；
  
  示例：`select concat(rtrim(vend_name),' (',rtrim(vend_country),')') from vendors order by vend_name; `

- `AS` 赋予别名；

    `SELECT quantity * item_price AS expanded_price FROM orderitems WHERE order_num = 20005;`  
    
- 支持的算术操作符：`+` 和 `-` 和 `*` 和 `/`

    注：MySQL 中的 `+` 只有运算符作用，没有连接符的作用：

    - 如果两个操作符都是数值型，则做加法运算；
    
    - 如果其中一方为字符型数值（如 ’124’/'00.2'/‘002’ 会被转换为 124/0.2/2），则将字符型转换为数值型，如果转换成功则做加法运算，失败则将字符型数值转换为 0；
    
        注意：`select 1 + '0.03.2';`结果为 1.03，所以这里的转换算法可以再看看；
    
    - 如果任何一方为 null 则结果为 null；

###  六、使用数据处理函数

#### （一）文本字符串处理函数

| 函数                 | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| LEFT(str,length)     | 返回指定长度的字段串的左边部分；str 是要提取子字符串的字符串。`length`是一个正整数，指定将从左边返回的字符数（就是个数）。`LEFT()`函数返回`str`字符串中最左边的长度字符。如果`str`或`length`参数为`NULL`，则返回`NULL`值。 如果`length`为`0`或为负，则`LEFT`函数返回一个空字符串。如果`length`大于`str`字符串的长度，则`LEFT`函数返回整个`str`字符串。  </br> 功能等同于 substring() |
| RIGHT()              | 同上                                                         |
| length(str)          | 返回串的字节长度，数字、字母算 1 个，汉字算 3 个；英文标点算 1 个，中文标点算 3 个； |
| locate(subStr,str)   | 返回 subStr 在 str 中出现的第一个位置（如果存在，最小从 1 开始），即如果返回值大于 0 则表示 str 包含 subStr，反之不包含就是等于 0； |
| lower()              | 将串转换为小写                                               |
| ltrim()              | 去掉串左边的空格                                             |
| rtrim()              | 去掉串右边的空格                                             |
| soundex()            | 返回串的soundex值，根据串的发音进行比较而不是根据字母比较    |
| substring()/substr() | 返回子串的字符                                               |
| upper()              | 将串转换为大写                                               |

示例：`SELECT cust_name FROM customers WHERE soundex(contact) = soundex('Y. Lie');` # 按发音搜索

#### （二）日期和时间处理函数 

| 函数          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| adddate()     | 增加一个日期（天，周等）                                     |
| addtime()     | 增加一个时间（时、分等）                                     |
| `curdate()`   | 返回当前日期，示例：`2022-04-21`                             |
| `curtime()`   | 返回当前时间，示例：`21:06:47`                               |
| `date()`      | 返回日期时间的日期部分，示例：`date("2020-02-02 00:25:21")` 和 `date("2020-02-02")` 结果是：`2020-02-02` |
| datediff()    | 计算两个日期之差                                             |
| date_add()    | 高度灵活的日期运算函数                                       |
| date_format() | 返回一个格式化的日期或时间串                                 |
| day()         | 返回一个日期的天数部分                                       |
| dayofweek()   | 对于一个日期，返回对应的星期几                               |
| hour()        | 返回一个时间的小时部分                                       |
| minute()      | 返回一个时间的分钟部分                                       |
| month()       | 返回一个日期的月份部分                                       |
| now()         | 返回当前日期和时间，示例：`2022-04-21 21:03:34`              |
| second()      | 返回一个时间的秒部分                                         |
| `time()`      | 返回一个日期时间的时间部分，示例：`time("2004-11-14 21:14:13")`  和 `time("21:14:13")` 返回 `21:14:13`，特例： `time("2104-21-34")` 返回 `00:21:04`，同时当年月日不合法时候，不会返回任何值，如：`time("2021-23-12 21:14:13")` 不会返回任何值； |
| `year()`      | 返回一个日期的年份部分                                       |

默认使用日期的格式为：`yyyy-mm-dd`

注意：`now()` 和 `sysdate()` 区别：

```sql
SELECT now() AS now1,sysdate() AS date1,   sleep(4),    now() AS now2,sysdate() AS  date2
```

结果为：

| now1                | date1               | sleep(4) | now2                | date2               |
| :------------------ | :------------------ | :------- | :------------------ | :------------------ |
| 2022-04-22 14:16:48 | 2022-04-22 14:16:48 | 0        | 2022-04-22 14:16:48 | 2022-04-22 14:16:52 |

结论：`now()` 在进行休眠 4 秒之后，再次执行还是和开始的时间是一样的，对于 `sysdate()` 函数，在同一个语句，执行了两次，第二次就是休眠 2 秒之后的真正的时间。

`now()` 返回的时间是 SQL 语句执行的时间,无论在一次 SQL 语句中 `now()`函数被执行多少次，即 SQL 开始执行的时间。

`sysdate()` 返回的时间是函数执行的时间，比如以上的一条 SQL 语句中执行了 2 次，第二次就是 `sysdate()` 执行的时间，即 `sysdate()` 执行的时间。

#### （三）数值处理函数

仅仅用于处理数值数据

| 函数    | 说明                                                         |
| ------- | ------------------------------------------------------------ |
| abs()   | 返回一个数的绝对值                                           |
| round() | 把数值字段舍入为指定的小数位数。ROUND(X)：返回参数X的四舍五入的一个整数。**ROUND(X,D)：** 返回参数X的四舍五入的有 D 位小数的一个数字。如果D为0，结果将没有小数点或小数部分。 |
| cos()   | 返回一个角度的余弦                                           |
| exp()   | 返回一个数的指数值                                           |
| mod()   | 返回除操作的余数                                             |
| pi()    | 返回圆周率                                                   |
| sin()   | 返回一个角度的正弦                                           |
| sqrt()  | 返回一个数的平方根                                           |
| tan()   | 返回一个角度的正切                                           |

### 七、汇总数据

聚类函数：运行在行组上，计算和返回单个值的函数

- `avg()`函数：**返回某列的平均值，只能作用于单列，如果作用于多列值需要使用多个 AVG 函数；且该方法会忽略列值为 NULL 的行**；

    - `SELECT avg(prod_price) AS avg_price FROM products;`
    - `SELECT avg(item_price) AS avg_price,avg(quantity) AS avg_quantity FROM orderitems;`

- `COUNT()` 函数：返回某列的行数；

    - **`COUNT(*)` 对表中行的数目进行计数，不忽略空值** 
        `SELECT COUNT(*) AS num_cust FROM customers; `
    - **使用 `COUNT(column)` 对特定列中具有值的行进行计数，忽略 NULL 值**
        `SELECT COUNT(cust_email) AS num_cust FROM customers;`  

- `max()`与`min()`：返回某列的最大值/最小值；

    - MAX() 返回 products 表中最贵的物品的价格
        `SELECT max(prod_price) AS max_price FROM products;`
    - 在用于文本数据时，如果数据按相应的列排序，则 `MAX()` 返回最后一行，`MIN()`返回最前一行；
        `SELECT max(prod_name) FROM products; `
    - `MAX()` 函数忽略列值为 null 的行；在用于文本数据时，如果数据按相应的列排序，在 max() 返回最后一行；
    
- `SUM()`：返回某列值之和；

    - 检索所订购物品的总数（所有 quantity 值之和）
        `SELECT SUM(quantity) AS ordered FROM orderitems WHERE order_num = 20005;`
    - 订单 20005 的总订单金额
        `SELECT SUM(quantity * item_price) AS total_price FROM orderitems WHERE order_num = 20005;`

- 聚类不同值 DISTINCT

    - 使用了 DISTINCT 参数，因此平均值只考虑各个不同的价格
        `SELECT avg(DISTINCT prod_price) AS avg_price FROM products WHERE vend_id = 1003;`
    - DISTINCT 只能作用于 `COUNT()`，不能用于 `COUNT(*)`。
    - DISTINCT 同 `max()`, `min()` 的结合使用，没有意义 。

- COALESCE 函数使用方式：

    用途：将空值替换成其他值，同时返回第一个非空值

    表达式
    `COALESCE(expression_1, expression_2, ...,expression_n)` 依次参考各参数表达式，遇到非 null 值即停止并返回该值。如果所有的表达式都是空值，最终将返回一个空值。

    示例：`select coalesce(success_cnt, 1) from tableA`
    当 success_cnt 为 null 值的时候，将返回 1，否则将返回 success_cnt 的真实值。

    `select coalesce(success_cnt,period,1) from tableA`
    当 success_cnt 不为 null，那么无论 period 是否为 null，都将返回 success_cnt 的真实值（因为success_cnt 是第一个参数），当 success_cnt 为 null，而 period 不为 null 的时候，返回 period 的真实值。只有当 success_cnt 和 period 均为 null 的时候，将返回1。

- 组合聚类函数 
    4 个聚集计算:物品的数目，产品价格的最高、最低以及平均值 

    ```sql
    SELECT 
        COUNT(*) AS num_items,
        MIN(prod_price) AS price_min,
        MAX(prod_price) AS price_max,
        AVG(prod_price) AS price_avg
    FROM
        products;
    ```

### MySQL 高级函数

| 函数名                                                       | 描述                                                         | 实例                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| BIN(x)                                                       | 返回 x 的二进制编码                                          | 15 的 2 进制编码:`SELECT BIN(15); -- 1111`                   |
| BINARY(s)                                                    | 将字符串 s 转换为二进制字符串                                | `SELECT BINARY "RUNOOB"; -> RUNOOB`                          |
| `CASE expression              WHEN condition1 THEN result1    WHEN condition2 THEN result2   ...                           WHEN conditionN THEN resultN    ELSE result                         END` | CASE 表示函数开始，END 表示函数结束。如果 condition1 成立，则返回 result1, 如果 condition2 成立，则返回 result2，当全部不成立则返回 result，而当有一个成立之后，后面的就不执行了。 | `SELECT CASE  　WHEN 1 > 0 　THEN '1 > 0' 　WHEN 2 > 0 　THEN '2 > 0' 　ELSE '3 > 0' 　END ->1 > 0` |
| CAST(x AS type)                                              | 转换数据类型                                                 | 字符串日期转换为日期：`SELECT CAST("2017-08-29" AS DATE); -> 2017-08-29` |
| COALESCE(expr1, expr2, ...., expr_n)                         | 返回参数中的第一个非空表达式（从左向右）                     | `SELECT COALESCE(NULL, NULL, NULL, 'runoob.com', NULL, 'google.com'); -> runoob.com` |
| CONNECTION_ID()                                              | 返回唯一的连接 ID                                            | `SELECT CONNECTION_ID(); -> 4292835`                         |
| CONV(x,f1,f2)                                                | 返回 f1 进制数变成 f2 进制数                                 | `SELECT CONV(15, 10, 2); -> 1111`                            |
| CONVERT(s USING cs)                                          | 函数将字符串 s 的字符集变成 cs                               | `SELECT CHARSET('ABC') ->utf-8     SELECT CHARSET(CONVERT('ABC' USING gbk)) ->gbk` |
| CURRENT_USER()                                               | 返回当前用户                                                 | `SELECT CURRENT_USER(); -> guest@%`                          |
| DATABASE()                                                   | 返回当前数据库名                                             | `SELECT DATABASE();    -> runoob`                            |
| IF(expr,v1,v2)                                               | 如果表达式 expr 成立，返回结果 v1；否则，返回结果 v2。       | `SELECT IF(1 > 0,'正确','错误')     ->正确`                  |
| [IFNULL(v1,v2)](https://www.runoob.com/mysql/mysql-func-ifnull.html) | 如果 v1 的值不为 NULL，则返回 v1，否则返回 v2。              | `SELECT IFNULL(null,'Hello Word') ->Hello Word`              |
| ISNULL(expression)                                           | 判断表达式是否为 NULL                                        | `SELECT ISNULL(NULL); ->1`                                   |
| LAST_INSERT_ID()                                             | 返回最近生成的 AUTO_INCREMENT 值                             | `SELECT LAST_INSERT_ID(); ->6`                               |
| NULLIF(expr1, expr2)                                         | 比较两个字符串，如果字符串 expr1 与 expr2 相等 返回 NULL，否则返回 expr1 | `SELECT NULLIF(25, 25); ->`                                  |
| SESSION_USER()                                               | 返回当前用户                                                 | `SELECT SESSION_USER(); -> guest@%`                          |
| SYSTEM_USER()                                                | 返回当前用户                                                 | `SELECT SYSTEM_USER(); -> guest@%`                           |
| USER()                                                       | 返回当前用户                                                 | `SELECT USER(); -> guest@%`                                  |
| VERSION()                                                    | 返回数据库的版本号                                           | `SELECT VERSION() -> 5.6.34`                                 |

### 八、分组计算

允许将数据分为多个逻辑组，以便能对每个组进行聚集计算；
主要有：GROUP BY 和 HAVING 子句；**WHERE 过滤行，HAVING 过滤分组 **, **WHERE 在数据分组前进行过滤，HAVING 在数据分组后进行过滤**。

- GROUP BY 分组 
    - **如果分组列中具有 null 值，将 null 作为一个分组返回，如果有多组 null 值，则将他们分为一组**；
    - **==GROUP BY 子句必须出现在 WHERE 子句之后，ORDER BY 子句之前==**；
    - 示例：
        按 vend_id **排序并分组**数据
        `SELECT vend_id, COUNT(*) AS num_prods FROM products GROUP BY vend_id;`
        使用 WITH ROLLUP 关键字，可以得到每个分组以及每个分组汇总级别（针对每个分组）的值，下述语句得到所有分组COUNT(*)的和14 ，比上面显示结果在最后一栏加入一行所有的数据总数；
        `SELECT vend_id, COUNT(*) AS num_prods FROM products GROUP BY vend_id WITH ROLLUP;`

- HAVING 子句 过滤分组 
    - **WHERE 过滤指定的是行而不是分组，HAVING 过滤的是分组**；同时 HAVING 支持所有的 WHERE 操作符；
    - 示例： COUNT(*) >=2（两个以上的订单）的那些分组
        `SELECT cust_id, COUNT(*) AS orders FROM orders GROUP BY cust_id HAVING COUNT(*)>=2;`
        或者使用：`SELECT cust_id, COUNT(*) AS orders FROM orders GROUP BY cust_id HAVING orders >= 2;`

- WHERE 和 HAVING 组合使用 （进行递进式查询）
    列出具有 2 个（含）以上、价格为 10（含）以上的产品的供应商
    `SELECT vend_id,COUNT(*) AS num_prods FROM products WHERE prod_price >=10 GROUP BY vend_id HAVING COUNT(*)>=2;`

- 分组和排序 
    检索总计订单价格大于等于50的订单的订单号和总计订单价格
    `SELECT order_num,SUM(quantity * item_price) AS ordertotal FROM orderitems GROUP BY order_num HAVING SUM(quantity * item_price) >=50;`

    按总计订单价格排序输出
    
    ```mysql
    SELECT 
        order_num, SUM(quantity * item_price) AS ordertotal
    FROM
        orderitems
    GROUP BY order_num
    HAVING SUM(quantity * item_price) >= 50
    ORDER BY ordertotal;
    ```
    
- ORDER BY 和 GROUP BY 区别

    **一般在使用 GROUP BY 子句时，同时给出 ORDER BY 子句，保证数据正确排序；**

    | ORDER BY                                 | GROUP BY                                               |
    | ---------------------------------------- | ------------------------------------------------------ |
    | 排序产生的输出                           | 分组行，但是输出可能不是分组的顺序                     |
    | 任意列都可以使用（甚至非选择的类也可以） | 只能使用选择列或表达式列，而且必须使用每个选择列表达式 |
    | 不一定需要                               | 如果和聚集函数一起使用列（或表达式），则必须使用       |

- SELECT 子句总结及顺序 

    | 子句     | 说明               | 是否必须使用           |
    | -------- | ------------------ | ---------------------- |
    | SELECT   | 要返回的列或表达式 | 是                     |
    | FROM     | 从中检索数据的表   | 仅在从表选择数据时使用 |
    | WHERE    | 行级过滤           | 否                     |
    | GROUP BY | 分组说明           | 仅在按组计算聚集时使用 |
    | HAVING   | 组级过滤           | 否                     |
    | ORDER BY | 输出排序顺序       | 否                     |
    | LIMIT    | 要检索的行数       | 否                     |

## 九、使用子查询     

**子查询（subquery）即嵌套在其他查询中的查询**；

- 利用子查询进行过滤，执行过程「**通常**」为从内往外执行，
  
    ```mysql
    -- 列出订购物品 TNT2 的所有客户
    SELECT cust_name, cust_contact FROM customers
    WHERE cust_id IN (SELECT cust_id  FROM orders
                      WHERE order_num IN (SELECT order_num
                                  FROM orderitems
                                  WHERE prod_id ='TNT2'));
    ```
    
- 子查询不要嵌套太多；

- 子查询不仅可以与 IN 操作符连用，还可以和`=`以及 `<>`等其他操作符连用；

- 作为计算字段使用子查询

    ```mysql
    --显示 customers 表中每个客户的订单总数
    SELECT cust_name,cust_state, 
    	(SELECT COUNT(*) 
         	FROM orders 
         	WHERE orders.cust_id = customers.cust_id) AS orders 
    FROM customers order by cust_name;
    ```


### 十、联结表       

- 关系表的设计就是要保证把信息分解成多个表，一类数据一个表，各表通过某些常用的值（即关系设计中的关系）互相连接；
- **外键(foreign key): 外键为某个表的一列，它包含另一个表的主键值，定义了两个表之间的关系**；
- 联结：联结不是物理实体，在实际的数据库中不存在，联结仅仅存在于查询的执行中；

#### （一）创建联结 

- 方式一： WHERE子句联结 （等值/内部联结）

    ```mysql
    SELECT vend_name,prod_name,prod_price 
    -- SELECT 语句需要联结的两个表的名字
    FROM vendors,products
    -- 这里的列名必须完全限定  表名.列名
    WHERE vendors.vend_id = products.vend_id
    ORDER BY vend_name,prod_name;
    ```

    **在连接两个表时候，实际上是将第一个表中的每一行与第二个表中的每一行匹配，其中 WHERE 子句作为了过滤条件，只包含那些匹配给定条件的行**；

- 笛卡尔积 / 叉联结 
    **由没有联结条件的表关系返回的结果为笛卡尔积。** 即相当于不使用 WHERE 子句，其检索出的行的数目将是第一个表中的行数乘以第二个表的行数。


- 方式二：INNER JOIN（内部联结）
  
    ```mysql
    -- 等效于上面 WHERE 联结
    SELECT vend_name,prod_name,prod_price 
    FROM vendors INNER JOIN products
    ON vendors.vend_id = products.vend_id;
    ```
    
- 连接多个表(只连接必要的表，否则性能会下降很多)

    ```mysql
    -- 编号为20005的订单中的物品及对应情况
    SELECT prod_name,vend_name,prod_price,quantity
    FROM orderitems,products,vendors
    WHERE products.vend_id = vendors.vend_id
    AND orderitems.prod_id = products.prod_id
    AND order_num = 20005;
    ```


### 十一、创建高级联结      

一共三种联结方式：自联结、自然联结、外部联结；

- 可以使用表别名
    **注意：表的别名只能在查询执行中使用，表别名不返回客户机**；
    
    ```mysql
     SELECT cust_name,cust_contact 
     FROM customers AS c,orders AS o,orderitems AS oi
     WHERE c.cust_id = o.cust_id
     and oi.order_num = o.order_num
     and prod_id = 'TNT2';
    ```

#### （一）自联结 

**自联结通常作为外部语句用来替代从相同表中检索数据时候使用的子查询语句；**

```sql
-- 使用子查询实现「ID 为 DTNTR 该物品的供应商生产的其他物品」
SELECT prod_id,prod_name FROM products
WHERE vend_id = (SELECT vend_id FROM products WHERE prod_id = 'DTNTR');
```

```sql
-- 使用联结实现上述功能
SELECT p1.prod_id,p1.prod_name
FROM products AS p1, products AS p2
WHERE p1.vend_id = p2.vend_id
and p2.prod_id = 'DTNTR';
```

#### （二）自然联结

**自然联结排除多次出现，使每个列只返回一次**，内部联结都是自然联结；

方法：通过对表使用通配符`*`，对所有其他表的列使用明确的子集 

```sql
SELECT c.*,o.num,o.date,oi.id,oi.quantity,oi.price
FROM customers AS c, orders AS o, orderitems AS oi
WHERE c.id = o.id
AND oi.num = o.num
AND prod_id = 'FB';
```

#### （三） 外部联结 

**联结包含了那些在相关表中没有关联行的行**，这种联结称为外部联结；

**注：**==在使用 OUTER JOIN 语法时候，必须使用 RIGHT 或者 LEFT 关键字指定包括其所有行的表==（RIGHT 指出的是 OUTER JOIN 右边的表，而 LEFT 指出的是 OUTER JOIN 左边的表）。

检索所有客户及其订单，包括那些没有订单的客户

```sql
-- 使用左外部联结
SELECT customers.cust_id,orders.order_num
FROM customers LEFT OUTER JOIN orders
on customers.cust_id = orders.cust_id;

-- 或者使用右外部联结（调换两表位置）
SELECT customers.cust_id,orders.order_num
FROM orders RIGHT OUTER JOIN customers
on customers.cust_id = orders.cust_id;
```


#### （四）使用带聚集函数的联结 

聚集函数用来汇总数据，可以和联结一起使用；
检索所有客户分别对应的订单数，INNER JOIN 

```sql
SELECT customers.cust_name,
       customers.cust_id,
       COUNT(orders.order_num) AS num_ord
FROM customers INNER JOIN orders 
on customers.cust_id = orders.cust_id
GROUP BY customers.cust_id; 
```

检索所有客户分别对应的订单数，包括没有订单的客户，LEFT OUTER JOIN 

```sql
SELECT customers.cust_name,
       customers.cust_id,
       COUNT(orders.order_num) AS num_ord
FROM customers LEFT OUTER JOIN orders 
on customers.cust_id = orders.cust_id
GROUP BY customers.cust_id; 
```

### 十二、组合/复合/并查询      

**使用 union 将多个 SELECT 语句组合成一个结果集**；

两种基本情况下，需要使用组合查询；

  - 在单个查询中从不同的表返回类似结构的数据；
  - 对单个表执行多个查询，按单个查询返回数据；

 多数情况下，组合相同表的两个查询完成的工作与具有多个 WHERE 子句条件的单个查询完成的工作相同，即任何具有多个 WHERE 子句的 SELECT 语句都可以作为一个组合查询给出；


### （一）创建组合查询

- 使用 union 
    下面两个单条语句，使用 union 进行组合查询；
    价格小于等于5的所有物品
    `SELECT vend_id,prod_id,prod_price FROM products WHERE prod_price <=5;`
    供应商1001和1002生产的所有物品
    `SELECT vend_id,prod_id,prod_price FROM products WHERE vend_id in (1001,1002);`

价格小于等于5的所有物品的列表，而且包括供应商1001和1002生产的所有物品（不考虑价格）

```sql
--  方式一：使用 union
SELECT vend_id,prod_id,prod_price FROM products WHERE prod_price <=5
union
SELECT vend_id,prod_id,prod_price FROM products WHERE vend_id in (1001,1002);

-- 使用 WHERE 实现
SELECT vend_id,prod_id,prod_price FROM products 
WHERE prod_price <=5 or vend_id in (1001,1002);
```

* **union默认从查询结果中自动去除重复的行；**
* **union 连接的每个查询之间必须包含相同的列、表达式或者聚集函数（不过各个列不需要以相同的次序出现）；**
* **列数据类型必须兼容（即可以隐式转换）**
* **union all，返回的是匹配的所有行 ，不取消重复行**

```sql
SELECT vend_id,prod_id,prod_price FROM products WHERE prod_price <=5
union all
SELECT vend_id,prod_id,prod_price FROM products WHERE vend_id in (1001,1002);  # 结果中有一行出现2次 
```

- 对 union 组合结果进行排序
    **union 组合完只能使用一条 order by 语句，放在最后一个SELECT语句后面 **，但是实际上 MySQL 将用它排序**所有 SELECT 语句**返回的所有结果；不能以不同的方式分别排序结果集，也不允许出现多条 ORDER BY 子句；

```sql
SELECT vend_id,prod_id,prod_price FROM products WHERE prod_price <=5
UNION
SELECT vend_id,prod_id,prod_price FROM products WHERE vend_id in (1001,1002)
ORDER BY vend_id,prod_price;
```


## 十三、全文本搜索       

**MyISAM 引擎支持全文本搜索，InnoDB 不支持；**

### （一）全文本搜索·

全文本搜索必须索引被搜索的列，而且随着数据的改变不断的重新索引，在对表进行适当设计之后，MySQL 会自动进行所有的索引和重新索引；

**默认搜索也是不区分大小写，除非使用：binary 方式；**

- 开启全文本搜索：
    一般在创建表时启动全文搜索，（放在()里面）语句为：`fulltext(字段名)` 对该字段名的列进行全文本搜索，当然可以搜索多个指定列；

**不要在导入数据的时候使用 fulltext**

- 进行全文本搜索  **默认不区分大小写**
    **Match() 指定被搜索的列，against()指定要使用的搜索表达式**，传递给 match()的值必须与 fulltext 中定义的相同，如果指定多个列，则必须列出它们且次序正确；
    `SELECT note_text FROM productnotes WHERE match(note_text) against('rabbit');`
    如果上面方法用like语句 
    `SELECT note_text FROM productnotes WHERE note_text like '%rabbit%';`

**使用全文搜索返回以文本匹配的良好程度排序的数据，其对结果进行排序，较高优先级的行先返回；** 优先级较高：搜索的关键字在文本中的位置靠前，或者出现的次数较多；等级是由 MySQL 根据行中词的数目、唯一词的数据、整个索引中词的总数以及包含该词的行的数目进行计算；如果有多个搜索项，则匹配包含多数匹配词的搜索项等级值较高；

**注意**：RANK (R)在mysql 8.0.2 (reserved)版本中为keyword保留字
当字段名与MySQL保留字冲突时,可以用字符`‘’`将字段名括起来或者改为其他名字，比如AS rank1等
即`SELECT note_text, match(note_text) against('rabbit') AS 'rank' FROM productnotes; `

- 使用查询扩展 

含义：**搜索所有可能与搜索词相关的数据，即使不含有搜索词**；
在进行查询拓展时，MySQL 对数据和索引进行**两遍**扫描来完成搜索；

  - 首先，进行一个基本的全文本搜索，找出与搜索条件匹配的所有行；
  - 其次，MySQL 检查这些匹配行并选择所有有用的词；
  - 在其次，MySQL 会再次进行全文本搜索，这次不仅使用原来的条件，还使用所有有用的词；
    进行一个简单的全文本搜索，没有查询扩展
     `SELECT note_text FROM productnotes WHERE match(note_text) against('anvils');`
    相同的搜索，这次使用查询扩展
     `SELECT note_text FROM productnotes WHERE match(note_text) against('anvils' with query expansion);`

- 布尔文本搜索:**MySQL 支持全文本搜索的另外一种形式**
    **布尔形式可以不需要使用：fulltext 索引，但是速度慢；**

**布尔方式中，不按照等级值降序排序返回的行，即排列而不排序；**


- 全文本布尔操作符 

    | 布尔操作符 | 说明                                                         |
    | ---------- | ------------------------------------------------------------ |
    | `+`        | 包含，词必须存在                                             |
    | `-`        | 排除，词必须不出现                                           |
    | `> `       | 包含，而且增加等级值                                         |
    | `<`        | 包含，且减少等级值                                           |
    | `()`       | 把词组成子表达式（允许这些表达式作为一个组被包含、排除、排列等） |
    | `~`        | 取消一个词的排序值                                           |
    | `*`        | 词尾的通配符(截断操作符)                                     |
    | `“ ” `     | 定义一个短语（与单个词的列表不一样，它匹配整个短语一边包含或排除这个短语） |

- 示例：全文本搜索检索包含词heavy的所有行
    使用了关键字IN BOOLEAN MODE，但是实际上没有指定布尔操作符，其结果与没有指定布尔方式的结果相同
    `SELECT note_text FROM productnotes WHERE match(note_text) against('heavy' in boolean mode);`

- 排除包含 `rope*`（任何以rope开始的词，包括ropes）的行【* 理解为截断操作符，即是一个用于词尾的通配符】
    `SELECT note_text FROM productnotes WHERE match(note_text) against('heavy -rope*' in boolean mode);`

- 匹配包含词rabbit和bait的行
    `SELECT note_text FROM productnotes WHERE match(note_text) against('+rabbit +bait' in boolean mode);`

- 不指定操作符，搜索匹配包含rabbit和bait中的至少一个词的行
    `SELECT note_text FROM productnotes WHERE match(note_text) against('rabbit bait' in boolean mode);`

- 搜索匹配短语rabbit bait而不是匹配两个词rabbit和bait。 
    `SELECT note_text FROM productnotes WHERE match(note_text) against('"rabbit bait"' in boolean mode);`

- 匹配rabbit和carrot，增加前者的等级，降低后者的等级
    `SELECT note_text FROM productnotes WHERE match(note_text) against('>rabbit <carrot' in boolean mode);`

- 必须匹配词safe和combination，降低后者的等级
    `SELECT note_text FROM productnotes WHERE match(note_text) against('+safe +(<combination)' in boolean mode);`


### 总结

- 索引全文本数据时候，短词（3 个或者 3 个以下字符）的词会被忽略，当然这个数目可以更改；
- MySQL 带有一个内建的非用词列表，索引的时候会被忽略（当然列表可以被覆盖）；
- 如果一个词在 50%以上的行中出现，作为非用词忽略，**但是在 In boolean mode 中无效**；
- 表中行数少于 3 行则不返回结果；
- 会忽略词中的单引号，例：`don't`索引为：`dont`

### 十四、插入数据 

#### （一）插入完整的行 

**简单但不安全，如果原来表列结构调整，会有问题** 
**自动增量的列不进行赋值的话，可以指定值为：NULL**

- 插入一个新客户到customers表
  
    ```mysql
    -- 方式一：明确指定列名，推荐使用
    INSERT INTO customer(name,address,city,state,zip,country,contact,email)VALUES ('Pep E. LaPew','100 Main Street','Los Angeles','CA','90046','USA',NULL,NULL);
    
    -- 方式二：不指定列名，不推荐使用
    INSERT INTO customers VALUES (null,'Pep E. LaPew','100 Main Street','LosAngeles','CA','90046','USA',NULL,NULL);
    ```

**部分列可以在插入的时候进行省略**，条件是：

- 该列定义为允许 NULL 值；
- 在表的定义中给出默认值；


### （二）插入多个行 

- 方法1： 提交多个insert 语句

    ```mysql
    insert into customers(cust_name,cust_address,cust_city,cust_state,cust_zip,cust_country) values('Pep E. LaPew','100 Main Street','LosAngeles','CA','90046','USA');
    insert into customers(cust_name,cust_address,cust_city,cust_state,cust_zip,cust_country)
    values('M. Martian','42 Galaxy Way','New York','NY','11213','USA');
    ```

- 方法2： 只要每条INSERT语句中的列名（和次序）相同，可以如下组合各语句
    单条INSERT语句有多组值，每组值用一对圆括号括起来，用逗号分隔

    ```mysql
    insert into customers(cust_name,cust_address,cust_city,cust_state,cust_zip,cust_country)
    values('Pep E. LaPew','100 Main Street','Los Angeles','CA','90046','USA'),('M. Martian','42 Galaxy Way','New York','NY','11213','USA');
    ```

- 插入检索出来的值 
    **INSERT 和 SELECT 语句中的列名不要求一定匹配**，MySQL 不会关系 SELECT 返回的列名，仅仅使用对应的列进行对应填充即可；
    新建custnew表（非书本内容）

    ```mysql
    CREATE TABLE custnew (
      cust_id int(11) NOT NULL AUTO_INCREMENT,
      cust_name char(50) NOT NULL,
      cust_address char(50) DEFAULT NULL,
      cust_city char(50) DEFAULT NULL,
      cust_state char(5) DEFAULT NULL,
      cust_zip char(10) DEFAULT NULL,
      cust_country char(50) DEFAULT NULL,
      cust_contact char(50) DEFAULT NULL,
      cust_email char(255) DEFAULT NULL,
      PRIMARY KEY (cust_id)
    ) ENGINE=InnoDB;
    ```

- 在表custnew中插入一行数据 （非书本内容）

    ```mysql
    insert into custnew (cust_id,cust_contact,cust_email,cust_name,cust_address,cust_city,cust_state,cust_zip,cust_country)
    values(null,null,'mysql carsh course@learning.com','Y.CARY','BAKE WAY','NEW YORK','NY','112103','USA');
    ```

- 将custnew中内容插入到customers表中 
    同书本代码不同，这里省略了custs_id,这样MySQL就会生成新值。

    ```mysql
    insert into customers (cust_contact,cust_email,cust_name,cust_address,cust_city,cust_state,cust_zip,cust_country)
    SELECT cust_contact,cust_email,cust_name,cust_address,cust_city,cust_state,cust_zip,cust_country FROM custnew;
    ```


## 第20章 更新和删除数据 

### （一）更新数据

update语句 : 删除或更新指定列 
- 更新单列： 
    客户10005现在有了电子邮件地址
    `update customers set cust_email = 'elmer@fudd.com' WHERE cust_id = 10005;`
    
- 更新多列： 

    ```mysql
    UPDATE customers 
    SET cust_name = 'The Fudds',
          cust_email = 'elmer@fudd.com'
    WHERE cust_id = 10005;
    ```

### （二）删除数据

- 删除： 某个列的值，可设置它为 NULL（假如表定义允许 NULL 值）
    `update customers set cust_email = null WHERE cust_id = 10005;`

- delete 语句： **删除整行而不是某列** 
    从 customers 表中删除一行
    `delete FROM customers WHERE cust_id = 10006;`

**使用 delete 仅仅是从表中删除行，即使删除所有行也不会删除表本身；**

- truncate table语句 
    **如果想从表中删除所有行，不要使用DELETE，可使用TRUNCATE TABLE语句，TRUNCATE 实际是删除原来的表并重新创建一个表，而不是逐行删除表中的数据**。

## 第 21 章 创建和操纵表  

### （一）新建表 create table

- 在新建表的同时可以设定主键`primary key(列名1，列名2，...)`，如果一个列名可以确定的情况下使用一个列名就行； 以及设定存储引擎：`engine = innodb`
- **NULL 值和空串不同**，空串本质上为 NOT NULL，是一个有效值；
- 每个表中仅有一列可以设置为 `AUTO_INCREMENT`，同时它**必须被索引**；同时对该列的值可以采用 INSERT 进行制定一个值（值只要是唯一的即可，即之前没有使用过），该值将会替代自动生成的值，然后后续的值将从这里开始进行递增；对于 `AUTO_INCREMENT` 列的值，可以使用 `last_insert_id()` 可以获取该值；
    - 设置默认值：例如：`id int NOT NULL DEFAULT 1,`，同时 MySQL 的默认值仅仅支持常量，不支持函数；
    - 引擎【外键不能跨引擎】
        - InnoDB：是一个可靠的事务处理引擎，但是不支持全文本搜索；
        - MEMORY：功能上等同于 MyISAM，但是数据存储在内存中，速度较快，一般适用于临时表；
        - MyISAM：性能高，支持全文本搜索，不支持事务处理；


###  （二）更新表 alter table 

- 给 vendors 表增加一个名为 vend_phone 的列
    `ALTER table vendors ADD vend_phone char(20);`

- 删除刚刚添加的列
    `ALTER table vendors DROP COLUMN vend_phone;`

**ALTER TABLE 的一种常见用途是定义外键**

- 以下为书本配套文件 create.sql 中定义外键的语句 

    ```sql
    ALTER TABLE orderitems ADD CONSTRAINT fk_orderitems_orders FOREIGN KEY (order_num) REFERENCES orders (order_num);
    
    ALTER TABLE orderitems ADD CONSTRAINT fk_orderitems_products FOREIGN KEY (prod_id) REFERENCES products (prod_id);
    
    ALTER TABLE orders ADD CONSTRAINT fk_orders_customers FOREIGN KEY (cust_id) REFERENCES customers (cust_id);
    
    ALTER TABLE products ADD CONSTRAINT fk_products_vendors FOREIGN KEY (vend_id) REFERENCES vendors (vend_id);
    ```


###  （三）删除表

- 删除 customers2 表
    `DROP TABLE customers2;`
    删除表是无需撤销、不能撤销、该语句是永久删除该数据表；


- 重命名表 
    使用 RENAME TABLE 语句可以重命名一个表；
    `RENAME TABLE customers2 TO customers;`

- 对多个表重命名；

    ```sql
    rename table backup_customers to customer,
                 backup_vendors to vendors,
                 backup_products to products;
    ```


## 第 22 章 使用视图   

**视图主要用于数据检索**

### （一）基本概念

- 视图（需要 MySQL5.0 以上）提供了一种MySQL的SELECT语句层次的封装，可用来简化数据处理以及重新格式化基础数据或保护基础数据。 

- **视图是虚拟的表，它不包含表中应该有的任何列或者数据，视图只包含使用时动态检索数据的 SQL 查询**；

- 使用视图的作用：
    - 重用 SQL 语句；
    - 简化复杂的 SQL 操作，在编写查询后，方便的重用它而不必知道它的基本查询细节；
    - 使用表的组成部分而不是整个表；
    - 保护数据，可以给用户授予表的特定部分的访问权限而不是整个表的访问权限；
    - 更改数据格式和表示；视图可以返回与底层表的表示和格式不同的数据；

- 视图仅仅是用来查看存储在别处的数据的一种设施，视图的本身不包含数据；

- 因为视图不包含数据，所以每次使用视图时，都必须处理查询执行时需要的所有检索。如果你用多个联结和过滤创建了复杂的视图或者嵌 套了视图，性能可能会下降得很厉害。因此，在部署使用了大量视图的应用前，应该进行测试。

- 视图的规则和限制
    *   与表一样，视图必须唯一命名
    *   对于可以创建的视图数目没有限制
    *   创建视图，必须具有足够的访问权限。这些权限通常由数据库管理人员授予
    *   视图可以嵌套，即可以利用从其他视图中检索数据的查询来构造视图。所允许的嵌套层数在不同的DBMS 中有所不同（嵌套视图可能会严重降低查询的性能，因此在产品环境中使用之前，应该对其进行全面测试）
    *   视图不能索引，也不能有关联的触发器或默认值

### （二）使用

- 创建视图 `create view`
- 创建视图的语句 `show create view viewname`
- 删除视图 `drop view viewname`
- 更新视图 先drop后create 或者直接用`create or repalce view`

#### 可以使用视图简化复杂的联结

**一次编写基础的 SQL，多次使用；**

- 创建一个名为productcustomers的视图，联结了三张表，返回订购任意产品的所有用户列表；

```sql
create view productcustomers as
select cust_name,cust_contact,prod_id
from customers,orders,orderitems
where customers.cust_id = orders.cust_id
and orders.order_num = orderitems.order_num;
```

- 检索订购了产品TNT2的客户
    `select cust_name,cust_contact from productcustomers where prod_id = 'TNT2';`
    **上面分析：** MySQL 处理这个查询的时候会将指定的 where 子句添加到视图查询中的已有的 where 子句中，从而正确的过滤数据；


#### 用视图重新格式化检索出的数据

- (来自第10章）在单个组合计算列中返回供应商名和位置
    `select concat(rtrim(vend_name),' (',rtrim(vend_country),')') as vend_title from vendors order by vend_name;`
    若经常使用上述格式组合，可以创建视图 
    `create view vendorlocations as select concat(rtrim(vend_name),' (',rtrim(vend_country),')') as vend_title from vendors order by vend_name;`

 然后可以检索出以创建所有邮件标签的数据
`select * from vendorlocations;`

#### 用视图过滤不想要的数据

- 定义customeremaillist视图，它过滤没有电子邮件地址的客户

```sql
create view customeremaillist as 
select cust_id,cust_name,cust_email from customers
where cust_email is not null;
select * from customeremaillist;
```


#### 使用视图与计算字段

(来自第10章）检索某个特定订单中的物品，计算每种物品的总价格
`select prod_id,quantity,item_price,quantity*item_price as expanded_price from orderitems where order_num = 20005;`
**使用视图的做法为：**
将其转换为一个视图
`create view orderitemsexpanded as select order_num,prod_id,quantity,item_price,quantity*item_price as expanded_price from orderitems;`
创建视图的时候select添加了列名order_num,否则无法按照order_num进行过滤查找 
`select * from orderitemsexpanded where order_num = 20005;`



#### 更新视图 

视图中虽然可以更新数据，但是有很多的限制。
一般情况下，最好将视图作为查询数据的虚拟表，而不要通过视图更新数据
**如果可以更新视图，本质上是更新其对应的基表；**
下面情况不能进行视图的更新：（即 MySQL 不能正确的确定更新的基数据，则不允许更新）

- 分组（使用 GROUP BY 和 HAVING）
- 联结
- 子查询
- 并
- 聚集函数（min()、max()、sum()等）
- DISTINCT；
- 导出（计算机）列



## 第23章 使用存储过程    

针对于 MySQL 5.0 以上版本，且一般 DBA 不允许用户创建存储过程，只允许其使用；
**存储过程就是为以后使用而保存的一条或多条SQL 语句。可将其视为批文件，虽然它们的作用不仅限于批处理**

**存储过程的优点：**
使用存储过程有三个主要的好处，即简单、安全、高性能:

*   通过把处理封装在一个易用的单元中，可以**简化**复杂的操作
*   由于不要求反复建立一系列处理步骤，因而保证了**数据的一致性**。可以防止错误。需要执行的步骤越多，出错的可能性就越大。
*   简化对变动的管理。提高**安全**性。通过存储过程限制对基础数据的访问，减少了数据讹误（无意识的或别的原因所导致的数据讹误）的机会
*   存储过程通常以编译过的形式存储，所以DBMS处理命令的工作较少，提高了**性能**
*   存在一些只能用在单个请求中的SQL元素和特性，存储过程可以使用它们来编写功能更强更**灵活**的代码

**存储过程的缺点**

*   不同DBMS中的存储过程语法有所不同。可移植性差
*   编写存储过程比编写基本SQL语句复杂，需要更高的技能，更丰富的经验


- 创建存储过程 
    返回产品平均价格的存储过程
    
    ```mysql
    delimiter //
    
    create procedure productpricing()
    
    begin
    
        select avg(prod_price) as priceaverage from products;
    
    end //
    
    delimiter ;
    ```

- 调用上述存储过程 

    `call productpricing();`

- 删除存储过程，请注意：没有使用后面的()，只给出存储过程名。

    `drop procedure productpricing;`

- 使用参数 out

     重新定义存储过程 productpricing

    ```mysql
    delimiter //
    
    create procedure productpricing(out pl decimal(8,2), out ph decimal(8,2), out pa decimal(8,2))
    
    begin
    
        select min(prod_price) into pl from products;
        
        select max(prod_price) into ph from products;
        
        select avg(prod_price) into pa from products;
    
    end //
    
    delimiter ;
    
    
    ```

    为调用上述存储过程，必须指定3个变量名

    `call productpricing(@pricelow,@pricehigh,@priceaverage);`

    显示检索出的产品平均价格

    `select @priceaverage;`

    获得 3 个值

    `select @pricehigh,@pricelow,@priceaverage;`

- 使用参数 in 和 out

     使用IN和OUT参数,存储过程ordertotal接受订单号并返回该订单的合计

    ```mysql
    delimiter //
    
    create procedure ordertotal(
        in onumber int,                   # onumber定义为IN，因为订单号被传入存储过程
        out ototal decimal(8,2)            # ototal为OUT，因为要从存储过程返回合计
    )
    
    begin
        select sum(item_price*quantity) from orderitems 
        where order_num = onumber
        into ototal;
    end //
    
    delimiter ;
    ```

    给ordertotal传递两个参数；

     第一个参数为订单号，第二个参数为包含计算出来的合计的变量名

    `call ordertotal(20005,@total);`

    显示此合计

    `select @total;`

     得到另一个订单的合计显示

    ```mysql
    call ordertotal(20009,@total);
    
    select @total;
    ```

- 建立智能存储过程 

     获得与以前一样的订单合计，但只针对某些顾客对合计增加营业税

    ```mysql
    -- Name:ordertotal
    
    -- Parameters: onumber = order number
    
    --                taxable = 0 if not taxable, 1 if taxable
    
    --                ototal  = order total variable
    
    delimiter //
    
    create procedure ordertotal(
    
        in onumber int,in taxable boolean,out ototal decimal(8,2)
    
    ) comment 'obtain order total, optionally adding tax'
    
    begin
    
        -- declare variable for total 定义局部变量totaldeclare total decimal(8,2);-- declare tax percentage 定义局部变量税率 declare taxrate int default 6;-- get the order total 获得订单合计SELECT SUM(item_price * quantity)FROM orderitemsWHERE order_num = onumber INTO total;-- is this taxable? 是否要增加营业税？ if taxable then    -- Yes,so add taxrate to the total 给订单合计增加税率    select total+(total/100*taxrate) into total;end if;-- and finally,save to out variable 最后，传递给输出变量 SELECT total INTO ototal;
    
    END //
    
    delimiter ;
    ```

    



调用上述存储过程，不加税 

call ordertotal(20005,0,@total);

select @total;

调用上述存储过程，加税 

call ordertotal(20005,1,@total);

select @total;


 显示用来创建一个存储过程的CREATE语句

show create procedure ordertotal;



获得包括何时、由谁创建等详细信息的存储过程列表

该语句列出所有存储过程

show procedure status;

#过滤模式 

show procedure status like 'ordertotal';


## 第24章 使用游标     

- 创建、打开、关闭游标 

     定义名为ordernumbers的游标，检索所有订单

    ```mysql
    delimiter //
    
    create procedure processorders()
    
    begin
    
        -- decalre the cursor 声明游标 declare ordernumbers cursorforselect order_num from orders;
    
    
    
        -- open the cursor 打开游标open ordernumbers;-- close the cursor 关闭游标close ordernumbers;
    
    end //
    
    delimiter ;
    ```

-  使用游标数据 

 例1：检索 当前行 的order_num列，对数据不做实际处理

```mysql
delimiter //

create procedure processorders()

begin

    -- declare local variables 声明局部变量declare o int;

    -- decalre the cursor 声明游标 declare ordernumbers cursorforselect order_num from orders;-- open the cursor 打开游标open ordernumbers;-- get order number 获得订单号 fetch ordernumbers into o;/*fetch检索 当前行 的order_num列（将自动从第一行开始）到一个名为o的局部声明变量中。对检索出的数据不做任何处理。*/-- close the cursor 关闭游标close ordernumbers;

END //

delimiter ;
```

 例2：循环检索数据，从第一行到最后一行，对数据不做实际处理

```mysql
delimiter //

create procedure processorders()

begin

    -- declare local variables 声明局部变量declare done boolean default 0;declare o int;

    -- decalre the cursor 声明游标 declare ordernumbers cursorforselect order_num from orders;

    -- declare continue handlerdeclare continue handler for sqlstate '02000' set done =1;-- SQLSTATE '02000'是一个未找到条件，当REPEAT由于没有更多的行供循环而不能继续时，出现这个条件。

    -- open the cursor 打开游标open ordernumbers;

    -- loop through all rows 遍历所有行 repeat

    -- get order number 获得订单号 fetch ordernumbers into o;-- FETCH在REPEAT内，因此它反复执行直到done为真

    -- end of loopuntil done end repeat;

    -- close the cursor 关闭游标close ordernumbers;

end //

delimiter ;
```



例3：循环检索数据，从第一行到最后一行，对取出的数据进行某种实际的处理

```mysql
delimiter //

create procedure processorders()

begin

    -- declare local variables 声明局部变量 declare done boolean default 0;declare o int;declare t decimal(8,2);

    -- declare the cursor 声明游标declare ordernumbers cursorforselect order_num from orders;

    -- declare continue handlerdeclare continue handler for sqlstate '02000' set done = 1;

    -- create a table to store the results 新建表以保存数据create table if not exists ordertotals(order_num int,total decimal(8,2));

    -- open the cursor 打开游标open ordernumbers;

    -- loop through all rows 遍历所有行repeat

    -- get order number 获取订单号fetch ordernumbers into o;

    -- get the total for this order 计算订单金额call ordertotal(o,1,t);  # 参见23章代码，已创建可使用

    -- insert order and total into ordertotals 将订单号、金额插入表ordertotals内insert into ordertotals(order_num,total) values(o,t);

    -- end of loopuntil done end repeat;

    -- close the cursor 关闭游标close ordernumbers;

end // 

delimiter ;
```

调用存储过程 precessorders()

call processorders();

输出结果

select * from ordertotals;

##  第 25 章 使用触发器      

- 创建触发器 

    `create trigger newproduct after insert on products for each row select 'product added' into @new_pro;`

    mysql 5.0以上版本在TRIGGER中不能返回结果集，定义了变量 @new_pro;

    ```mysql
    - insert into products(prod_id,vend_id,prod_name,prod_price) values ('ANVNEW','1005','3 ton anvil','6.09'); # 插入一行 
    
    select @new_pro;  # 显示Product added消息
    ```

- 删除触发器 

    `drop trigger newproduct;`

- 使用触发器 
    -  insert触发器

        ```mysql
        create trigger neworder after insert on orders for each row select new.order_num into @order_num;
        
        insert into orders(order_date,cust_id) values (now(),10001);
        
        select @order_num;
        ```

    -  delete触发器

         使用OLD保存将要被删除的行到一个存档表中 

        ```mysql
        delimiter //
        
        create trigger deleteorder before delete on orders for each row
        
        begin
            insert into archive_orders(order_num,order_date,cust_id)values(old.order_num,old.order_date,old.cust_id); # 引用一个名为OLD的虚拟表，访问被删除的行
        
        end //
        
        delimiter ;
        ```

    - update触发器

        ```mysql
        #在更新vendors表中的vend_state值时，插入前先修改为大写格式 
        
        create trigger updatevendor before update on vendors 
        
        for each row set new.vend_state = upper(new.vend_state);
        
        # 更新1001供应商的州为china
        
        update vendors set vend_state = 'china' where vend_id =1001;
        
        # 查看update后数据，1001供应商对应的vend_state自动更新为大写的CHINA
        
        select * from vendors;
        ```

## 第 26 章 管理事务处理 

- 事务 transaction 指一组 sql 语句

- 回退 rollback 指撤销指定 sql 语句的过程

- 提交 commit 指将未存储的sql语句结果写入数据库表

- 保留点 savepoint 指事务处理中设置的临时占位符，可以对它发布回退（与回退整个事务处理不同）

- 控制事务处理
    - 开始事务及回退 

        ```mysql
        select * from ordertotals;   # 查看ordertotals表显示不为空
        
        start transaction;           # 开始事务处理 
        
        delete from ordertotals;     # 删除ordertotals表中所有行
        
        select * from ordertotals;   # 查看ordertotals表显示 为空
        
        rollback;                     # rollback语句回退 
        
        select * from ordertotals;   # rollback后，再次查看ordertotals表显示不为空

    - commit 提交 

        ```mysql
        start transaction;
        
        delete from orderitems where order_num = 20010;
        
        delete from orders where order_num = 20010;
        
        commit;   # 仅在上述两条语句不出错时写出更改 
        ```

    - savepoint 保留点 

         ```mysql
         # 创建保留点
         savepoint delete1;
         
         # 回退到保留点 
         rollback to delete1;
         
         # 释放保留点 
         release savepoint delete1;
         ```

- 更改默认的提交行为 

    `set autocommit = 0;`  # 设置autocommit为0（假）指示MySQL不自动提交更改


## 第 27 章 全球化和本地化

### 字符集和校对顺序

- 查看所支持的字符集完整列表：`show character set;`

- 查看所支持校对的完整列表，以及它们适用的字符集：`show collation;`

- 确定所用系统的字符集和校对

    ```mysql
    show variables like 'character%';
    
    show variables like 'collation%';
    ```

- 使用带子句的 CREATE TABLE，给表指定字符集和校对

    ```mysql
    create table mytable(
        column1 int,
        column2 varchar(10)
    ) default character set hebrew  collate hebrew_general_ci;
    ```

- 除了能指定字符集和校对的表范围外，MySQL还允许对每个列设置它们

    ```mysql
    create table mytable(
        column1 int,
        column2 varchar(10),
        column3 varchar(10) character set latin1 collate latin1_general_ci
    )default character set hebrew  collate hebrew_general_ci;
    ```

- 校对 collate 在对用 ORDER BY 子句排序时起重要的作用

     如果要用与创建表时不同的校对顺序排序，可在 SELECT语句中说明 

    `select * from customers order by lastname,firstname collate latin1_general_cs;`


## 第28章 安全管理

- 管理用户

     需要获得所有用户账号列表时，mysql 数据库有一个名为 user 的表，它包含所有用户账号。user 表有一个名为 user 的列；

    ```mysql
    use mysql;
    select user from user;
    ```

- 创建用户账号，使用 create user

    `create user ben identified by 'p@$$w0rd';`

- 重命名一个用户账号

    `rename user ben to bforta;`

- 删除用户账号 

    `drop user bforta;`

- 查看赋予用户账号的权限

    `show grants for bforta;`

- 允许用户在（crashcourse 数据库的所有表）上使用 SELECT，只读

    `grant select on crashcourse.* to bforta;`

- 重新查看赋予用户账号的权限，发生变化 

    `show grants for bforta;`

- 撤销特定的权限

    `revoke select on crashcourse.* from bforta;`

- 简化多次授权

    `grant select,insert on crashcourse.* to bforta;`

- 更改口令


     原来课本中使用的 `password()` 加密函数，在 8.0 版本中已经移除 ，password() :This function was removed in MySQL 8.0.11.
    
    `set password for bforta = 'n3w p@$$w0rd';`

- 如果不指定用户名，直接修改当前登录用户的口令 

    `set password = 'n3w p@$$w0rd';`

## 第29章 数据库维护 

- 分析表/键状态是否正确：`analyze table orders;`

- 检查表是否存在错误 

    ```mysql
    check table orders,orderitems;
    
    check table orders,orderitems quick; # QUICK只进行快速扫描
    ```

- 优化表 OPTIMIZE TABLE，消除删除和更新造成的磁盘碎片，从而减少空间的浪费：`optimize table orders;`
