# 记录 mysql 函数GROUP_CONCAT用法及踩坑点 

## concat()函数：

1、功能：

将多个字符串连接成一个字符串。

2、语法：

concat(str1, str2,...)

返回结果为连接参数产生的字符串，如果有任何一个参数为null，则返回值为null。

## concat_ws()函数：

1、功能：

和concat()一样，将多个字符串连接成一个字符串，但是可以一次性指定分隔符～（concat_ws就是concat with separator）

2、语法：

concat_ws(separator, str1, str2, ...)

说明：第一个参数指定分隔符。需要注意的是分隔符不能为null，如果为null，则返回结果为null。

## group_concat函数：

1、功能：

将group by产生的同一个分组中的值连接起来，返回一个字符串结果。

2、语法：

group_concat( [distinct] 要连接的字段 [order by 排序字段 asc/desc ] [separator '分隔符'] )

说明：通过使用distinct可以排除重复值；如果希望对结果中的值进行排序，可以使用order by子句；separator是一个字符串值，缺省为一个逗号。

下面主要看一下group_concat函数

表中数据如下：
![img](CONCAT%E8%AF%A6%E8%A7%A3.resource/703023-20210530231109910-758746603.png)*

如图可知，一个用户可以有多种角色，现在要把多种角色合并为一条数据展示。

```sql
SELECT
    user_no,
    GROUP_CONCAT( DISTINCT role_name ) AS role_name 
FROM
    report_user_role_info 
GROUP BY
    user_no;
```

查询结果如下：

![img](CONCAT%E8%AF%A6%E8%A7%A3.resource/703023-20210530231215069-164359357.png)

 

可以达到预期效果。

如果想要根据角色对应id对这三个角色进行排序，比如降序预期结果应该是“顺风车车主用户(3),乘客用户(2),注册用户(1)”来排序：

```sql
SELECT
    user_no,
    GROUP_CONCAT( DISTINCT role_name ORDER BY role_id desc ) AS role_name
FROM
    report_user_role_info 
GROUP BY
    user_no;
```

查询结果如下:

![img](CONCAT%E8%AF%A6%E8%A7%A3.resource/703023-20210530231313828-943656287.png)

 

 如果只想展示一个或者两个角色，可以使用SUBSTRING_INDEX函数，SUBSTRING_INDEX(a,b,c)，a参数代表要截取的字符串，b参数代表分隔符，c参数代表截取几个字符

sql(截取两个)：

```sql
SELECT
    user_no,
    SUBSTRING_INDEX(GROUP_CONCAT( DISTINCT role_name ORDER BY role_id desc ),',',2) AS role_name
FROM
    report_user_role_info 
GROUP BY
    user_no;
```

查询结果如下：

 ![img](CONCAT%E8%AF%A6%E8%A7%A3.resource/703023-20210530231413758-524574322.png)

### 踩坑记录：

当数据太大，group_concat超出了默认值1024，超过就会截断，group_concat查询出来的数据就是不全。

解决：

1.查看当前 mysql group_concat_max_len：
进入mysql状态，输入：

```
show variables like 'group_concat_max_len';
```

结果如下：

 ![img](CONCAT%E8%AF%A6%E8%A7%A3.resource/703023-20210530231526311-2029448382.png)

可以看到当前值为1024.

2.在MySQL配置文件中添加配置：group_concat_max_len = -1 （-1为最大值或根据实际需求设置长度），配置后需要重启MySQL服务
如果是生产环境下，不能擅自重启MySQL服务，则可以通过语句设置group_concat的作用范围，如：

```
set session group_concat_max_len = -1 
```

 