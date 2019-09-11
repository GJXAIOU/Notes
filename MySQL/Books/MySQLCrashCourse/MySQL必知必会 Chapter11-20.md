---
tags: [MySQL]
---

# MySQL 必知必会 Chapter11-20

@toc

## 章十一：使用数据处理函数

减少使用函数，但是函数的可移植性却不强；



### 文本处理函数

函数 | 说明
---|---
LEFT()  |     返回串左边的字符 
length()  |  返回串的长度 
locate()   |  找出串的一个子串 
lower()  |    将串转换为小写
ltrim()   |   去掉串左边的空格
RIGHT()   |   返回串右边的字符 
rtrim()    | 去掉串右边的空格  
soundex() |返回串的soundex值，根据串的发音进行比较而不是根据字母比较
substring()  |   返回子串的字符 
upper()   |    将串转换为大写

实例：`SELECT cust_name FROM customers WHERE soundex(contact) = soundex('Y. Lie');` # 按发音搜索

### 日期和时间处理函数 
|函数| 说明
|---|---
 adddate()      |   增加一个日期（天，周等）
 addtime()     |    增加一个时间（时、分等）
curdate()     | 返回当前日期 
 curtime()     |    返回当前时间 
 date()         |    返回日期时间的日期部分 
 datediff()     |    计算两个日期之差 
 date_add()   |      高度灵活的日期运算函数 
 date_format()   |  返回一个格式化的日期或时间串 
 day()           |   返回一个日期的天数部分 
 dayofweek()    |     对于一个日期，返回对应的星期几 
 hour()          |    返回一个时间的小时部分 
 minute()      |    返回一个时间的分钟部分 
 month()     |         返回一个日期的月份部分 
 now()       |       返回当前日期和事件 
 second()   |       返回一个时间的秒部分 
 time()       |        返回一个日期时间的时间部分 
 year()       |       返回一个日期的年份部分 |
默认使用日期的格式为：`yyyy-mm-dd`


### 数值处理函数
仅仅用于处理数值数据
|函数| 说明|
|---|---
abs()         |   返回一个数的绝对值 
cos()         |   返回一个角度的余弦
exp()         |   返回一个数的指数值
mod()         |   返回除操作的余数 
pi()          |   返回圆周率 
sin()         |   返回一个角度的正弦 
sqrt()        |   返回一个数的平方根 
tan()         |   返回一个角度的正切



## 章十二：汇总数据
聚类函数：运行在行组上，计算和返回单个值的函数
|SQL 聚集函数 |
|---|--- 
avg()       |     返回某列的平均值 
COUNT()     |     返回某列的行数 
max()       |     返回某列的最大值 
min()       |     返回某列的最小值 
SUM()       |     返回某列值之和 

- avg()实例：
  - AVG()返回products表中所有产品的平均价格
`SELECT avg(prod_price) AS avg_price FROM products;`
  - avg()只能作用于单列，获取 多列的平均值，要使用多个avg()
`SELECT avg(item_price) AS avg_itemprice,avg(quantity) AS avg_quantity FROM orderitems;`
  - avg()函数忽略列值为 null 的行；

- COUNT()实例
  - COUNT(*)对表中行的数目进行计数，不忽略空值 
`SELECT COUNT(*) AS num_cust FROM customers; `
  - 使用COUNT(column)对特定列中具有值的行进行计数，忽略NULL值
`SELECT COUNT(cust_email) AS num_cust FROM customers;`  

- max() & min()
  - MAX()返回products表中最贵的物品的价格
`SELECT max(prod_price) AS max_price FROM products;`
  - 在用于文本数据时，如果数据按相应的列排序，则MAX()返回最后一行
`SELECT max(prod_name) FROM products; `
  - MIN()返回products表中最便宜物品的价格
`SELECT min(prod_price) AS min_price FROM products;`
  - 在用于文本数据时，如果数据按相应的列排序，则MIN()返回最前面一行
`SELECT min(prod_name) FROM products;` 
  - MAX()函数忽略列值为 null 的行；在用于文本数据时，如果数据按相应的列排序，在 max() 返回最后一行；



- SUM()
  - 检索所订购物品的总数（所有quantity值之和）
 `SELECT SUM(quantity) AS ordered FROM orderitems WHERE order_num = 20005;`
  - 订单20005的总订单金额
`SELECT SUM(quantity * item_price) AS total_price FROM orderitems WHERE order_num = 20005;`

- 聚类不同值DISTINCT
  - 使用了 DISTINCT 参数，因此平均值只考虑各个不同的价格
`SELECT avg(DISTINCT prod_price) AS avg_price FROM products WHERE vend_id = 1003;`
  - DISTINCT 只能作用于COUNT(),不能用于COUNT(*)
  - DISTINCT 同max(),min()的结合使用，没有意义 

- 组合聚类函数 
4个聚集计算:物品的数目，产品价格的最高、最低以及平均值 
```sql
SELECT 
    COUNT(*) AS num_items,
    MIN(prod_price) AS price_min,
    MAX(prod_price) AS price_max,
    AVG(prod_price) AS price_avg
FROM
    products;
```


## 章十三： 分组计算    

允许将数据分为多个逻辑组，以便能对每个组进行聚集计算；
主要有：GROUP BY 和 HAVING 子句；**WHERE过滤行，HAVING过滤分组 **, 
**WHERE在数据分组前进行过滤，HAVING在数据分组后进行过滤**

- GROUP BY 分组 
  - **如果分组列中具有 null 值，将 null 作为一个分组返回，如果有多组 null 值，则将他们分为一组**；
  - **GROUP BY 子句必须出现在 WHERE 子句之后，order by 子句之前**；
  - 示例：
按vend_id**排序并分组**数据
`SELECT vend_id, COUNT(*) AS num_prods FROM products GROUP BY vend_id;`
使用WITH ROLLUP关键字，可以得到每个分组以及每个分组汇总级别（针对每个分组）的值，下述语句得到所有分组COUNT(*)的和14 ，比上面显示结果在最后一栏加入一行所有的数据总数；
`SELECT vend_id, COUNT(*) AS num_prods FROM products GROUP BY vend_id with rollup;`

- HAVING子句 过滤分组 
  - **WHERE 过滤指定的是行而不是分组，HAVING 过滤的是分组**；同时 HAVING 支持所有的 WHERE 操作符；
  - 示例： COUNT(*) >=2（两个以上的订单）的那些分组
`SELECT cust_id, COUNT(*) AS orders FROM orders GROUP BY cust_id HAVING COUNT(*)>=2;`
或者使用：`SELECT cust_id, COUNT(*) AS orders FROM orders GROUP BY cust_id HAVING orders >= 2;`

**WHERE 在数据分组前进行过滤，HAVING 在数据分组之后进行过滤；**

- WHERE和HAVING组合使用 （进行递进式查询）
列出具有2个（含）以上、价格为10（含）以上的产品的供应商
`SELECT vend_id,COUNT(*) AS num_prods FROM products WHERE prod_price >=10 GROUP BY vend_id HAVING COUNT(*)>=2;`
不加WHERE条件，结果不同 
`SELECT vend_id,COUNT(*) AS num_prods FROM products GROUP BY vend_id HAVING COUNT(*) >=2;`



- 分组和排序 
检索总计订单价格大于等于50的订单的订单号和总计订单价格
`SELECT order_num,SUM(quantity * item_price) AS ordertotal FROM orderitems GROUP BY order_num HAVING SUM(quantity * item_price) >=50;`

按总计订单价格排序输出
```sql
SELECT 
    order_num, SUM(quantity * item_price) AS ordertotal
FROM
    orderitems
GROUP BY order_num
HAVING SUM(quantity * item_price) >= 50
ORDER BY ordertotal;
```

- order by 和 GROUP BY 区别

|order by |GROUP BY|
|---|---|
|排序产生的输出|分组行，但是输出可能不是分组的顺序|
|任意列都可以使用（甚至非选择的类也可以）|只能使用选择列或表达式列，而且必须使用每个选择列表达式   |
|不一定需要|如果和聚集函数一起使用列（或表达式），则必须使用|

**一般在使用 GROUP BY 子句时，同时给出 order by 子句，保证数据正确排序；**

- SELECT子句总结及顺序 
子句      |      说明             |           是否必须使用 
|---|---|---
SELECT     |   要返回的列或表达式  |     是 
FROM       |     从中检索数据的表  |       仅在从表选择数据时使用 
WHERE      |      行级过滤        |         否 
GROUP BY   |    分组说明          |         仅在按组计算聚集时使用 
HAVING     |     组级过滤          |         否 
order by   |     输出排序顺序      |         否
limit      |    要检索的行数       |           否 





## 章十四：使用子查询     
**子查询（subquery）即嵌套在其他查询中的查询**；

- 利用子查询进行过滤
执行过程为从内往外执行，
列出订购物品TNT2的所有客户
```sql
SELECT cust_name, cust_contact
FROM customers
WHERE cust_id IN (SELECT cust_id
                  FROM orders
                  WHERE order_num IN (SELECT order_num
                                      FROM orderitems
                                      WHERE prod_id = 'TNT2'));
```

- 子查询不要嵌套太多；
- 子查询不仅可以与 IN 操作符连用，还可以和`=`以及 `<>`等其他操作符连用；
- 作为计算字段使用子查询
显示customers 表中每个客户的订单总数
`SELECT cust_name,cust_state, (SELECT COUNT(*) FROM orders WHERE orders.cust_id = customers.cust_id) AS orders FROM customers order by cust_name;`


## 章十五：联结表       

- 关系表的定义
关系表的设计就是要保证把信息分解成多个表，一类数据一个表，各表通过某些常用的值（即关系设计中的关系）互相连接；
- 外键(foreign key): 外键为某个表的一列，它包含另一个表的主键值，定义了两个表之间的关系；
- 联结：联结不是物理实体，在实际的数据库中不存在，联结仅仅存在于查询的执行中；

### 创建联结 
- 方式一： WHERE子句联结 （称为 等值联结，或者内部联结）
```sql
SELECT vend_name,prod_name,prod_price 
// SELECT 语句需要联结的两个表的名字
FROM vendors,products
// 这里的列名必须完全限定  表名.列名
WHERE vendors.vend_id = products.vend_id
order by vend_name,prod_name;
```
**在连接两个表时候，实际上是将第一个表中的每一行与第二个表中的每一行匹配，其中 WHERE 子句作为了过滤条件，只包含那些匹配给定条件的行**；

- 笛卡尔积 / 叉联结 
**由没有联结条件的表关系返回的结果为笛卡尔积。** 可以相当于不使用 WHERE 子句；
检索出的行的数目将是第一个表中的行数乘以第二个表的行数。


- 方式二：使用  INNER JOIN
内部联结 INNER JOIN ： 表间相等测试 。相比 WHERE 子句，应该首选 INNER JOIN 语句；
下面的语句实现的功能和上面的 WHERE 子句相同；
```sql
SELECT vend_name,prod_name,prod_price 
FROM vendors INNER JOIN products
on vendors.vend_id = products.vend_id;
```

- 连接多个表(只连接必要的表，否则性能会下降很多)
编号为20005的订单中的物品及对应情况 
```sql
SELECT prod_name,vend_name,prod_price,quantity
FROM orderitems,products,vendors
WHERE products.vend_id = vendors.vend_id
and orderitems.prod_id = products.prod_id
and order_num = 20005;
```

**章 14 中列出订购物品TNT2的所有客户的子查询可以使用联结代替**
```sql
SELECT cust_name, cust_contact
FROM customers, orders, orderitems
WHERE customers.id = orders.cust.id
and orderitems.order_num = orders.order_num
and prod_id = 'TNT2';
```


## 章十六： 创建高级联结      

一共三种联结方式：自联结、自然联结、外部联结；

- 使用表别名
给列名或计算字段起别名 
```sql
SELECT concat(rtrim(vend_name),' (',rtrim(vend_country),')') AS vend_title
 FROM vendors order by vend_name;
```

**注意：** **表的别名只能在查询执行中使用，表别名不返回客户机**；
 给表起别名 
```sql
 SELECT cust_name,cust_contact 
 FROM customers AS c,orders AS o,orderitems AS oi
 WHERE c.cust_id = o.cust_id
 and oi.order_num = o.order_num
 and prod_id = 'TNT2';
```

### （一）自联结 
**自联结通常作为外部语句用来替代从相同表中检索数据时候使用的子查询语句；**

ID为DTNTR该物品的供应商生产的其他物品
  - 方法：子查询 
```sql
SELECT prod_id,prod_name FROM products
WHERE vend_id = (SELECT vend_id FROM products WHERE prod_id = 'DTNTR');
```
  - 方法：使用联结 
```sql
SELECT p1.prod_id,p1.prod_name
FROM products AS p1, products AS p2
WHERE p1.vend_id = p2.vend_id
and p2.prod_id = 'DTNTR';
```

### （二）自然联结
**自然联结排除多次出现，使每个列只返回一次**

方法：通过对表使用通配符*，对所有其他表的列使用明确的子集 
```sql
SELECT c.*,o.order_num,o.order_date,oi.prod_id,oi.quantity,oi.item_price
FROM customers AS c,orders AS o,orderitems AS oi
WHERE c.cust_id = o.cust_id
and oi.order_num = o.order_num
and prod_id = 'FB';
```
目前建立的所有内部联结都是自然联结；


### （三） 外部联结 
**联结包含了那些在相关表中没有关联行的行**，这种联结称为外部联结；

**注：**在使用 OUTER JOIN 语法时候，必须使用 RIGHT 或者 LEFT 关键字指定包括其所有行的表（RIGHT 指出的是 OUTER JOIN 右边的表，而 LEFT 指出的是 OUTER JOIN 左边的表）。

检索所有客户及其订单,包括那些没有订单的客户
01 ： 左外部联结
```sql
SELECT customers.cust_id,orders.order_num
FROM customers LEFT OUTER JOIN orders
on customers.cust_id = orders.cust_id;
```

02 ：若使用 右外部联结 结果不同 
```sql
SELECT customers.cust_id,orders.order_num
FROM customers RIGHT OUTER JOIN orders
on customers.cust_id = orders.cust_id;
```

 03： 若使用 右外部联结 调换两表位置 结果同01代码相同 
```sql
SELECT customers.cust_id,orders.order_num
FROM orders RIGHT OUTER JOIN customers
on customers.cust_id = orders.cust_id;
```


### 使用带聚集函数的联结 
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



## 第17章 组合查询      

**使用 union 将多个 SELECT 语句组合成一个结果集**；
同时组合查询称之为：并或者复合查询；

两种基本情况下，需要使用组合查询；
  - 在单个查询中从不同的表返回类似结构的数据；
  - 对单个表执行多个查询，按单个查询返回数据；

 多数情况下，组合相同表的两个查询完成的工作与具有多个 WHERE 子句条件的单个查询完成的工作相同，即任何具有多个 WHERE 子句的 SELECT 语句都可以作为一个组合查询给出；


### （一）创建组合查询

- 使用union 
下面两个单条语句，使用 union 进行组合查询；
价格小于等于5的所有物品
`SELECT vend_id,prod_id,prod_price FROM products WHERE prod_price <=5;`
供应商1001和1002生产的所有物品
`SELECT vend_id,prod_id,prod_price FROM products WHERE vend_id in (1001,1002);`

价格小于等于5的所有物品的列表，而且包括供应商1001和1002生产的所有物品（不考虑价格）
  - 方法1 使用union 
```sql
SELECT vend_id,prod_id,prod_price FROM products WHERE prod_price <=5
union
SELECT vend_id,prod_id,prod_price FROM products WHERE vend_id in (1001,1002);
```

  - 方法2 使用WHERE 
```sql
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

- 对union组合结果进行排序
**union组合完只能使用一条order by语句，放在最后一个SELECT语句后面 **，但是实际上 MySQL 将用它排序**所有 SELECT 语句**返回的所有结果；不能以不同的方式分别排序结果集，也不允许出现多条 ORDER BY 子句；

```sql
SELECT vend_id,prod_id,prod_price FROM products WHERE prod_price <=5
UNION
SELECT vend_id,prod_id,prod_price FROM products WHERE vend_id in (1001,1002)
ORDER BY vend_id,prod_price;
```


## 第18章 全文本搜索       

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

布尔操作符   |        说明
|---|---|
`+`         |  包含，词必须存在 
`-`         |  排除，词必须不出现
`> `        |  包含，而且增加等级值 
 `<`        |  包含，且减少等级值 
 `()`       |  把词组成子表达式（允许这些表达式作为一个组被包含、排除、排列等）
 `~`        |  取消一个词的排序值
 `*`       |   词尾的通配符(截断操作符)
`“ ” `   | 定义一个短语（与单个词的列表不一样，它匹配整个短语一边包含或排除这个短语）

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



## 第19章 插入数据 

### （一）插入完整的行 
**简单但不安全，如果原来表列结构调整，会有问题** 
**自动增量的列不进行赋值的话，可以指定值为：NULL**

- 插入一个新客户到customers表
`insert into customers values (null,'Pep E. LaPew','100 Main Street','LosAngeles','CA','90046','USA',NULL,NULL);`


 **表明括号内明确列名，更安全，稍繁琐** 
在插入的同时明确列名：
`insert into customers (cust_name,cust_address,cust_city,cust_state,cust_zip,cust_country,cust_contact,cust_email)values ('Pep E. LaPew','100 Main Street','Los Angeles','CA','90046','USA',NULL,NULL);`

**部分列可以在插入的时候进行省略**，条件是：
- 该列定义为允许 NULL 值；
- 在表的定义中给出默认值；


### （二）插入多个行 

- 方法1： 提交多个insert 语句
```sql
insert into customers(cust_name,cust_address,cust_city,cust_state,cust_zip,cust_country) values('Pep E. LaPew','100 Main Street','LosAngeles','CA','90046','USA');
insert into customers(cust_name,cust_address,cust_city,cust_state,cust_zip,cust_country)
values('M. Martian','42 Galaxy Way','New York','NY','11213','USA');
```

- 方法2： 只要每条INSERT语句中的列名（和次序）相同，可以如下组合各语句
单条INSERT语句有多组值，每组值用一对圆括号括起来，用逗号分隔
```sql
insert into customers(cust_name,cust_address,cust_city,cust_state,cust_zip,cust_country)
values('Pep E. LaPew','100 Main Street','Los Angeles','CA','90046','USA'),('M. Martian','42 Galaxy Way','New York','NY','11213','USA');
```

- 插入检索出来的值 
**INSERT 和 SELECT 语句中的列名不要求一定匹配**，MySQL 不会关系 SELECT 返回的列名，仅仅使用对应的列进行对应填充即可；
新建custnew表（非书本内容）
```sql
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
```sql
insert into custnew (cust_id,cust_contact,cust_email,cust_name,cust_address,cust_city,cust_state,cust_zip,cust_country)
values(null,null,'mysql carsh course@learning.com','Y.CARY','BAKE WAY','NEW YORK','NY','112103','USA');
```


- 将custnew中内容插入到customers表中 
同书本代码不同，这里省略了custs_id,这样MySQL就会生成新值。
```sql
insert into customers (cust_contact,cust_email,cust_name,cust_address,cust_city,cust_state,cust_zip,cust_country)
SELECT cust_contact,cust_email,cust_name,cust_address,cust_city,cust_state,cust_zip,cust_country FROM custnew;
```


## 第20章 更新和删除数据 

### （一）更新数据
- update语句 : 删除或更新指定列 
  - 更新单列： 
客户10005现在有了电子邮件地址
`update customers set cust_email = 'elmer@fudd.com' WHERE cust_id = 10005;`
  - 更新多列： 
```sql
UPDATE customers 
SET cust_name = 'The Fudds',
      cust_email = 'elmer@fudd.com'
WHERE cust_id = 10005;
```

### （二）删除数据

- 删除： 某个列的值，可设置它为NULL（假如表定义允许NULL值）
`update customers set cust_email = null WHERE cust_id = 10005;`

- delete 语句： **删除整行而不是某列** 
从customers表中删除一行
`delete FROM customers WHERE cust_id = 10006;`

**使用 delete 仅仅是从表中删除行，即使删除所有行也不会删除表本身；**

- truncate table语句 
**如果想从表中删除 所有行，不要使用DELETE，可使用TRUNCATE TABLE语句**
TRUNCATE实际是删除原来的表并重新创建一个表，而不是逐行删除表中的数据

MySQL 没有撤销按钮，注意！