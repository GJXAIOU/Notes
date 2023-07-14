# 浅析 MySQL 的隐式转换

**[作者简介]** 陈晓，信息部订单组研发工程师，目前主要负责小米订单中台业务。

## 前言

跟大家一块看下 MySQL 的隐式转换相关知识，主要是相等操作时，先看两个可能都遇到过的场景。

表

```mysql
CREATE TABLE `t1` (
  `c1` varchar(255) NOT NULL DEFAULT '',
  KEY `idx_c1` (`c1`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert into t1 values('1234567890123456789');
insert into t1 values('123456789012345678');
insert into t1 values('123456789012345677');
insert into t1 values('12345678901234567');
insert into t1 values('12345678901234568');
insert into t1 values('123456789012345');
```

数据

```mysql
+---------------------+
| c1                  |
+---------------------+
| 123456789012345     |
| 12345678901234567   |
| 12345678901234568   |
| 123456789012345677  |
| 123456789012345678  |
| 1234567890123456789 |
+---------------------+
```

- 场景一

```mysql
select * from t1 where c1=123456789012345678;

+--------------------+
| c1                 |
+--------------------+
| 123456789012345677 |
| 123456789012345678 |
+--------------------+
```

查询出两条数据，并且 123456789012345677 不等于查询条件

- 场景二

```mysql
explain select * from t1 where c1=12345678901234567;

+----+-------------+-------+------------+-------+---------------+--------+---------+------+------+----------+--------------------------+
| id | select_type | table | partitions | type  | possible_keys | key    | key_len | ref  | rows | filtered | Extra                    |
+----+-------------+-------+------------+-------+---------------+--------+---------+------+------+----------+--------------------------+
|  1 | SIMPLE      | t1    | NULL       | index | idx_c1        | idx_c1 | 767     | NULL |    6 |    16.67 | Using where; Using index |
+----+-------------+-------+------------+-------+---------------+--------+---------+------+------+----------+--------------------------+
```

全表扫描

## 一、什么是隐式类型转换

MySQL 中对隐式转换的定义：

> When an operator is used with operands of different types, type conversion occurs to make the operands compatible. Some conversions occur implicitly.
>
> 翻译：当操作符与不同类型的操作数一起使用时，会发生类型转换以使操作数兼容。

举个例子，当操作数是字符跟数字时， MySQL 会根据使用的操作符，转换字符到数字或转换数字成字符。

```mysql
mysql> SELECT 1+'1';
        -> 2
mysql> SELECT CONCAT(2,' test');
        -> '2 test'
```

### 二、比较操作时 MySQL 隐式类型转换规则

官方文档中有一段对比较操作时， MySQL 的类型转换做了说明：

> If one or both arguments are NULL, the result of the comparison is NULL, except for the NULL-safe <=> equality comparison operator. For NULL <=> NULL, the result is true. No conversion is needed.
>
> If both arguments in a comparison operation are strings, they are compared as strings.
>
> If both arguments are integers, they are compared as integers.
>
> Hexadecimal values are treated as binary strings if not compared to a number.
>
> If one of the arguments is a TIMESTAMP or DATETIME column and the other argument is a constant, the constant is converted to a timestamp before the comparison is performed. This is done to be more ODBC-friendly. This is not done for the arguments to IN(). To be safe, always use complete datetime, date, or time strings when doing comparisons. For example, to achieve best results when using BETWEEN with date or time values, use CAST() to explicitly convert the values to the desired data type.
>
> A single-row subquery from a table or tables is not considered a constant. For example, if a subquery returns an integer to be compared to a DATETIME value, the comparison is done as two integers. The integer is not converted to a temporal value. To compare the operands as DATETIME values, use CAST() to explicitly convert the subquery value to DATETIME.
>
> If one of the arguments is a decimal value, comparison depends on the other argument. The arguments are compared as decimal values if the other argument is a decimal or integer value, or as floating-point values if the other argument is a floating-point value.
>
> In all other cases, the arguments are compared as floating-point (real) numbers. For example, a comparison of string and numeric operands takes places as a comparison of floating-point numbers.

翻译为中文就是：

- 两个参数至少有一个是 NULL 时，比较的结果也是 NULL，例外是使用 `<=>` 对两个 NULL 做比较时会返回 1，这两种情况都不需要做类型转换
- 两个参数都是字符串，会按照字符串来比较，不做类型转换
- 两个参数都是整数，按照整数来比较，不做类型转换
- 十六进制的值和非数字做比较时，会被当做二进制串
- 有一个参数是 TIMESTAMP 或 DATETIME，并且另外一个参数是常量，常量会被转换为 timestamp
- 有一个参数是 decimal 类型，如果另外一个参数是 decimal 或者整数，会将整数转换为 decimal 后进行比较，如果另外一个参数是浮点数，则会把 decimal 转换为浮点数进行比较
- 所有其他情况下，两个参数都会被转换为浮点数再进行比较

我们用十六进制的来验证看下

### （一）十六进制

```mysql
CREATE TABLE `t5` (
  `c1` varchar(255) NOT NULL DEFAULT '',
  c2 int,
  KEY `idx_c1` (`c1`),
  KEY idx_c2(c2)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

alter table t5 add column c3 bigint;
alter table t5 add index idx_c3(c3);
alter table t5 drop index idx_c3;

insert into t5(c1, c2) values('$', 16);
insert into t5(c1, c2) values('!', 12);
insert into t5(c1, c2) values('1', 16);
insert into t5(c1, c2) values(' 1', 11);
insert into t5(c1, c2) values('1a', 10);

select * from t5 where c1 = 0x24;
+----+------+
| c1 | c2   |
+----+------+
| $  |   16 |
+----+------+

explain select * from t5 where c1 = 0x24;
+----+-------------+-------+------------+------+---------------+--------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key    | key_len | ref   | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+--------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | t5    | NULL       | ref  | idx_c1        | idx_c1 | 767     | const |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+------+---------------+--------+---------+-------+------+----------+-------+
```

通过查询

```mysql
select HEX('$');
+----------+
| HEX('$') |
+----------+
| 24       |
+----------+
```

![img](浅析 MySQL 的隐式转换.resource/asciifull.gif)

或者查 ASCII 码表可以看到，**$** 对应的 16 进制正是 0x24。

查询数字

```mysql
select * from t5 where c2 = 0x10;
+----+------+
| c1 | c2   |
+----+------+
| $  |   16 |
| 1  |   16 |
+----+------+

explain select * from t5 where c2 = 0x10;
+----+-------------+-------+------------+------+---------------+--------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key    | key_len | ref   | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+--------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | t5    | NULL       | ref  | idx_c2        | idx_c2 | 5       | const |    2 |   100.00 | NULL  |
+----+-------------+-------+------------+------+---------------+--------+---------+-------+------+----------+-------+
```

### 根据规则看下示例

根据上面的规则，分析开始的示例，c1 字段类型是 varchar，等号右边常量是数字，MySQL 会把 c1 字段的值和数字常量转换成浮点数比较。

当字段是字符串跟数字比较时，MySQL 不能使用索引来加快查询。比如下面的 SQL 语句，str_col 是 varchar 型的字段

```mysql
SELECT * FROM tbl_name WHERE str_col=1;
```

正如示例的执行计划显示的那样，进行了全表扫描。原因在于字符串转浮点型的时候所使用的算法，在 MySQL 里以下’1’, ‘ 1’, ‘1a’字符串转换成数字后都是 1。

我们根据 t5 表做下查询

```mysql
select * from t5 where c1 = 1;
+----+------+
| c1 | c2   |
+----+------+
| 1  |   16 |
|  1 |   11 |
| 1a |   10 |
+----+------+
```

看下官方文档

> The server includes dtoa, a conversion library that provides the basis for improved conversion > between string or DECIMAL values and approximate-value (FLOAT/DOUBLE) numbers:
>
> Consistent conversion results across platforms, which eliminates, for example, Unix versus > Windows conversion differences.
>
> Accurate representation of values in cases where results previously did not provide sufficient > precision, such as for values close to IEEE limits.
>
> Conversion of numbers to string format with the best possible precision. The precision of dtoa > is always the same or better than that of the standard C library functions.

MySQL 里使用了 dtoa 这个转换类库来实现字符跟浮点数的相互转换，一是提高了浮点数的精度（相比 c 标准库）, 二是保持平台一致性。

在 MySQL 里浮点数之间的比较是近似的。因为 MySQL 里浮点数是不精确的，它的精度位只有 53 比特，大于 53 比特的位被“四舍五入”。这会导致一些违反直观的结果。
比如下面两个 SQL

```mysql
mysql> SELECT '1801537632024345812' = 1801537632024345812;
        -> 1
mysql> SELECT '1801537632024345812' = 1801537632024345813;
        -> 1
```

比较结果都是相等，这里就发生了浮点数转换。
我们来看下到底发生了什么。

浮点数

```mysql
SELECT '1801537632024345812'+0.0;
+---------------------------+
| '1801537632024345812'+0.0 |
+---------------------------+
|     1.8015376320243459e18 |
+---------------------------+
```

转换成浮点数后对应的整数

```mysql
SELECT cast('1801537632024345812' + 0.0 as unsigned);
+-----------------------------------------------+
| cast('1801537632024345812' + 0.0 as unsigned) |
+-----------------------------------------------+
|                           1801537632024345856 |
+-----------------------------------------------+

SELECT cast('1801537632024345813' + 0.0 as unsigned);
+-----------------------------------------------+
| cast('1801537632024345813' + 0.0 as unsigned) |
+-----------------------------------------------+
|                           1801537632024345856 |
+-----------------------------------------------+
```

转换成浮点数后在转换成整数，对应的都是
**1801537632024345856**

分别对应的二进制

```mysql
1801537632024345812 -> 11001000000000101100011101110011011100100111100111100（53）11010100
1801537632024345856 -> 11001000000000101100011101110011011100100111100111101（53）00000000
```

对比可以看到 53 位后的数据被丢掉，因为第 54 位是 1，所以有一个进位。

再看下我们的示例数字转换成浮点数后分别对应的数字

```mysql
select c1, cast(c1 + 0.0 as unsigned) from t1;
+---------------------+----------------------------+
| c1                  | cast(c1 + 0.0 as unsigned) |
+---------------------+----------------------------+
| $                   |                          0 |
| 123456789012345     |            123456789012345 |
| 12345678901234567   |          12345678901234568 |
| 12345678901234568   |          12345678901234568 |
| 123456789012345677  |         123456789012345680 |
| 123456789012345678  |         123456789012345680 |
| 1234567890123456789 |        1234567890123456768 |
+---------------------+----------------------------+
```

## 其他情况

```mysql
CREATE TABLE `t3` (
  `c1` bigint NOT NULL DEFAULT '0'
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert into t3 values(1234567890123456789);
insert into t3 values(123456789012345678);
insert into t3 values(123456789012345677);
insert into t3 values(12345678901234567);
insert into t3 values(12345678901234568);
insert into t3 values(123456789012345);

select * from t3 where c1='123456789012345678';
+--------------------+
| c1                 |
+--------------------+
| 123456789012345678 |
+--------------------+
```

此处结果跟前面的规则貌似不一致？
在 MySQL 执行的 prepare 阶段，在设置比较方法之前，满足一些调教的情况下会做一次类型转换

mysql server 处理调用栈

```mysql
Item_bool_func2::convert_constant_arg item_cmpfunc.cc:636
Item_bool_func2::fix_length_and_dec item_cmpfunc.cc:700
Item_func::fix_fields item_func.cc:253
st_select_lex::setup_conds sql_resolver.cc:1190
st_select_lex::prepare sql_resolver.cc:212
handle_query sql_select.cc:139
execute_sqlcom_select sql_parse.cc:5155
mysql_execute_command sql_parse.cc:2826
mysql_parse sql_parse.cc:5584
dispatch_command sql_parse.cc:1491
do_command sql_parse.cc:1032
handle_connection connection_handler_per_thread.cc:313
pfs_spawn_thread pfs.cc:2197
start_thread 0x00007f1bb48ce6ba
clone 0x00007f1bb3d6341d
```

转换方法

```c
bool Item_bool_func2::convert_constant_arg(THD *thd, Item *field, Item **item)
{
  if (field->real_item()->type() != FIELD_ITEM)
    return false;

  Item_field *field_item= (Item_field*) (field->real_item());
  if (field_item->field->can_be_compared_as_longlong() &&
      !(field_item->is_temporal_with_date() &&
      (*item)->result_type() == STRING_RESULT))
  {
    if (convert_constant_item(thd, field_item, item))
    {
      cmp.set_cmp_func(this, tmp_arg, tmp_arg + 1, INT_RESULT);
      field->cmp_context= (*item)->cmp_context= INT_RESULT;
      return true;
    }
  }
  return false;
}
```

item 和 field 分别对应的数据

```c
item = {Item ** | 0x7f1b34006f88} 0x7f1b34006f88
 *item = {PTI_text_literal_text_string * | 0x7f1b34006538} 0x7f1b34006538

 
field = {Field_longlong * | 0x7f1b34abfb20} 0x7f1b34abfb20
 Field_num = {Field_num}
```

转换后 item 对应的数据

```c
a2 = {Item ** | 0x7f2b3c006f88} 0x7f2b3c006f88
 *a2 = {Item_int_with_ref * | 0x7f2b3c007320} 0x7f2b3c007320
  Item_int = {Item_int} 
  cached_field_type = {enum_field_types} MYSQL_TYPE_LONGLONG
  ref = {PTI_text_literal_text_string * | 0x7f2b3c006538} 0x7f2b3c006538
```

看下 `convert_constant_item` 的注释

```c
/**
  Convert a constant item to an int and replace the original item.

    The function converts a constant expression or string to an integer.
    On successful conversion the original item is substituted for the
    result of the item evaluation.
    This is done when comparing DATE/TIME of different formats and
    also when comparing bigint to strings (in which case strings
    are converted to bigints).

  @param  thd             thread handle
  @param  field           item will be converted using the type of this field
  @param[in,out] item     reference to the item to convert

  @note
    This function is called only at prepare stage.
    As all derived tables are filled only after all derived tables
    are prepared we do not evaluate items with subselects here because
    they can contain derived tables and thus we may attempt to use a
    table that has not been populated yet.

  @retval
    0  Can't convert item
  @retval
    1  Item was replaced with an integer version of the item
*/

static bool convert_constant_item(THD *thd, Item_field *field_item,
                                  Item **item)
```

大体意思是把常量对象（表达式或者字符串）转换成整数并替换，此方法只有在比较不同的日期格式和比较 bigint 跟字符串时才有效。注意上面括号中的这句 `in which case strings are converted to bigints`

上面的转换规则并没有覆盖所有的情况，或者说一些情况 MySQL 在比较之前做了优化，比如此处的查询，已经不在是数字跟字符串比较了。

上面规则里面还有句 `This is not done for the arguments to IN()` ，此处不再分析，留下大家有兴趣自行看下， MySQL 又做了哪些优化。

## 总结

避免发生隐式类型转换，隐式转换的类型主要有字段类型不一致、in 参数包含多个类型、字符集类型或校对规则不一致等

隐式类型转换可能导致无法使用索引、查询结果不准确等，因此在使用时必须仔细甄别

数字类型的建议在字段定义时就定义为 int 或者 bigint，表关联时关联字段必须保持类型、字符集、校对规则都一致

由于历史原因，需要兼容旧的设计，可以使用 MySQL 的类型转换函数 cast 和 convert，来明确的进行转换。

如有错误之处，还望大家指正。

## 附录

### 关于浮点数的问题

[Problems with Floating-Point Values](https://dev.mysql.com/doc/refman/5.7/en/problems-with-float.html)

### 关于进制问题

Round-to-Even for Floating Point

Round-To-Even 在于 To-Up , To-Down, To-towards-Zero 对比中，在一定数据量基础上，更加精准。To-Up 的平均值比真实数值偏大，To-Down 偏小。

例如有效数字超出规定数位的多余数字是 1001，它大于超出规定最低位的一半，故最低位进 1。
如果多余数字是 0111，它小于最低位的一半，则舍掉多余数字（截断尾数、截尾）即可。
对于多余数字是 1000、正好是最低位一半的特殊情况，最低位为 0 则舍掉多余位，最低位为 1 则进位 1、使得最低位仍为 0（偶数）。

注意这里说明的数位都是指二进制数。

对于第三种情况，来看下两个例子：

```mysql
SELECT cast('1801537632024345728' + 0.0 as unsigned);
+-----------------------------------------------+
| cast('1801537632024345728' + 0.0 as unsigned) |
+-----------------------------------------------+
|                           1801537632024345600 |
+-----------------------------------------------+

1801537632024345728 -> 1100100000000010110001110111001101110010011110011110010000000
1801537632024345600 -> 1100100000000010110001110111001101110010011110011110000000000

SELECT cast('1801537632024345984' + 0.0 as unsigned);
+-----------------------------------------------+
| cast('1801537632024345984' + 0.0 as unsigned) |
+-----------------------------------------------+
|                           1801537632024346112 |
+-----------------------------------------------+

1801537632024345984 -> 1100100000000010110001110111001101110010011110011110110000000
1801537632024346112 -> 1100100000000010110001110111001101110010011110011111000000000
```

------

**作者**

陈晓，小米信息技术部订单组

**招聘**

信息部是小米公司整体系统规划建设的核心部门，支撑公司国内外的线上线下销售服务体系、供应链体系、ERP 体系、内网 OA 体系、数据决策体系等精细化管控的执行落地工作，服务小米内部所有的业务部门以及 40 家生态链公司。

同时部门承担大数据基础平台研发和微服务体系建设落，语言涉及 Java、Go，长年虚位以待对大数据处理、大型电商后端系统、微服务落地有深入理解和实践的各路英雄。

欢迎投递简历：jin.zhang(a)xiaomi.com（武汉）

Tags: [MySQL ](https://xiaomi-info.github.io/tags#MySQL)[隐式转换](https://xiaomi-info.github.io/tags#隐式转换)