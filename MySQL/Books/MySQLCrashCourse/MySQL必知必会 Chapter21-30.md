---
tags: [MySQL]
---

# MySQL 必知必会 Chapter21-30

@toc

==看到了这里==[从这里开始](#第23章-使用存储过程)

## 第21章 创建和操纵表  

### （一）新建表 create table
- 在新建表的同时可以设定主键`primary key(列名1，列名2，...)`，如果一个列名可以确定的情况下使用一个列名就行； 以及设定存储引擎：`engine = innodb`
- **NULL 值和 空串不同**，空串本质上为 NOT NULL，是一个有效值；
- 每个表中仅有一列可以设置为 `AUTO_INCREMENT`，同时它必须被索引；同时对该列的值可以采用 INSERT 进行制定一个值（值只要是唯一的即可，即之前没有使用过），该值将会替代自动生成的值，然后后续的值减从这里开始进行递增；对于 `AUTO_INCREMENT` 列的值，可以使用 `last_insert_id()` 可以获取该值；
    - 设置默认值：例如：`id int NOT NULL DEFAULT 1,`；同时 MySQL 的默认值仅仅支持常量，不支持函数；
    - 引擎【外键不能跨引擎】
      - InnoDB：是一个可靠的事务处理引擎，但是不支持全文本搜索；
      - MEMORY：功能上等同于 MyISAM，但是数据存储在内存中，速度较快，一般适用于临时表；
      - MyISAM：性能高，支持全文本搜索，不支持事务处理；


###  （二）更新表 alter table 

- 给vendors表增加一个名为vend_phone的列
`alter table vendors   add vend_phone char(20);`

- 删除刚刚添加的列
`alter table vendors drop column vend_phone;`


**ALTER TABLE的一种常见用途是定义外键**

- 以下为书本配套文件create.sql中定义外键的语句 
```sql
ALTER TABLE orderitems ADD CONSTRAINT fk_orderitems_orders FOREIGN KEY (order_num) REFERENCES orders (order_num);

ALTER TABLE orderitems ADD CONSTRAINT fk_orderitems_products FOREIGN KEY (prod_id) REFERENCES products (prod_id);

ALTER TABLE orders ADD CONSTRAINT fk_orders_customers FOREIGN KEY (cust_id) REFERENCES customers (cust_id);

ALTER TABLE products ADD CONSTRAINT fk_products_vendors FOREIGN KEY (vend_id) REFERENCES vendors (vend_id);
```


###  （三）删除表

- 删除customers2表（假设它存在）
`drop table customers2;`
删除表是无需撤销、不能撤销、该语句是永久删除该数据表；


- 重命名表 
使用RENAME TABLE语句可以重命名一个表 (假设存在下述表)
`rename table customers2 to customers;`

- 对多个表重命名(假设存在下述表)
```sql
rename table backup_customers to customer,
             backup_vendors to vendors,
             backup_products to products;
```


## 第22章 使用视图   
**视图主要用于数据检索**
### （一）基本概念

- 视图（需要 MySQL5.0 以上）提供了一种MySQL的SELECT语句层次的封装，可用来简化数据处理以及重新格式化基础数据或保护基础数据。 

- 视图是虚拟的表，它不包含表中应该有的任何列或者数据，视图只包含使用时动态检索数据的 SQL 查询；

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

delimiter //

create procedure productpricing()

begin

    select avg(prod_price) as priceaverage from products;

end //

delimiter ;

 调用上述存储过程 

call productpricing();



-- 删除存储过程,请注意:没有使用后面的()，只给出存储过程名。

drop procedure productpricing;



-- 使用参数 out

 重新定义存储过程productpricing

delimiter //

create procedure productpricing(out pl decimal(8,2), out ph decimal(8,2), out pa decimal(8,2))

begin

    select min(prod_price) into pl from products;

    select max(prod_price) into ph from products;

    select avg(prod_price) into pa from products;

end //

delimiter ;



为调用上述存储过程，必须指定3个变量名

call productpricing(@pricelow,@pricehigh,@priceaverage);

显示检索出的产品平均价格

select @priceaverage;

#获得3个值

select @pricehigh,@pricelow,@priceaverage;



-- 使用参数 in 和 out

 使用IN和OUT参数,存储过程ordertotal接受订单号并返回该订单的合计

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

给ordertotal传递两个参数；

 第一个参数为订单号，第二个参数为包含计算出来的合计的变量名

call ordertotal(20005,@total);

显示此合计

select @total;

 得到另一个订单的合计显示

call ordertotal(20009,@total);

select @total;



-- 建立智能存储过程 

 获得与以前一样的订单合计，但只针对某些顾客对合计增加营业税



-- Name:ordertotal

-- Parameters: onumber = order number

--                taxable = 0 if not taxable, 1 if taxable

--                ototal  = order total variable

delimiter //

create procedure ordertotal(

    in onumber int,

    in taxable boolean,

    out ototal decimal(8,2)

) comment 'obtain order total, optionally adding tax'

begin

    -- declare variable for total 定义局部变量total

    declare total decimal(8,2);

    -- declare tax percentage 定义局部变量税率 

    declare taxrate int default 6;

    -- get the order total 获得订单合计

    SELECT SUM(item_price * quantity)

    FROM orderitems

    WHERE order_num = onumber INTO total;

    -- is this taxable? 是否要增加营业税？ 

    if taxable then

        -- Yes,so add taxrate to the total 给订单合计增加税率

        select total+(total/100*taxrate) into total;

    end if;

    -- and finally,save to out variable 最后，传递给输出变量 

    SELECT total INTO ototal;

END //

delimiter ;

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


-- 创建、打开、关闭游标 

 定义名为ordernumbers的游标，检索所有订单

delimiter //

create procedure processorders()

begin

    -- decalre the cursor 声明游标 

    declare ordernumbers cursor

    for

    select order_num from orders;



    -- open the cursor 打开游标

    open ordernumbers;

    -- close the cursor 关闭游标

    close ordernumbers;

end //

delimiter ;



-- 使用游标数据 

 例1：检索 当前行 的order_num列，对数据不做实际处理

delimiter //

create procedure processorders()

begin



    -- declare local variables 声明局部变量

    declare o int;



    -- decalre the cursor 声明游标 
    declare ordernumbers cursor
    for
    select order_num from orders;
    -- open the cursor 打开游标
    open ordernumbers;
    -- get order number 获得订单号 
    fetch ordernumbers into o;
    /*fetch检索 当前行 的order_num列（将自动从第一行开始）到一个名为o的局部声明变量中。
    对检索出的数据不做任何处理。*/
    -- close the cursor 关闭游标
    close ordernumbers;

END //

delimiter ;



 例2：循环检索数据，从第一行到最后一行，对数据不做实际处理

delimiter //

create procedure processorders()

begin

    -- declare local variables 声明局部变量

    declare done boolean default 0;

    declare o int;



    -- decalre the cursor 声明游标 

    declare ordernumbers cursor

    for

    select order_num from orders;



    -- declare continue handler

    declare continue handler for sqlstate '02000' set done =1;

    -- SQLSTATE '02000'是一个未找到条件，当REPEAT由于没有更多的行供循环而不能继续时，出现这个条件。



    -- open the cursor 打开游标

    open ordernumbers;



    -- loop through all rows 遍历所有行 

    repeat



    -- get order number 获得订单号 

    fetch ordernumbers into o;

    -- FETCH在REPEAT内，因此它反复执行直到done为真



    -- end of loop

    until done end repeat;



    -- close the cursor 关闭游标

    close ordernumbers;



end //

delimiter ;



例3：循环检索数据，从第一行到最后一行，对取出的数据进行某种实际的处理

delimiter //

create procedure processorders()

begin

    -- declare local variables 声明局部变量 

    declare done boolean default 0;

    declare o int;

    declare t decimal(8,2);



    -- declare the cursor 声明游标

    declare ordernumbers cursor

    for

    select order_num from orders;



    -- declare continue handler

    declare continue handler for sqlstate '02000' set done = 1;



    -- create a table to store the results 新建表以保存数据

    create table if not exists ordertotals

    (order_num int,total decimal(8,2));



    -- open the cursor 打开游标

    open ordernumbers;



    -- loop through all rows 遍历所有行

    repeat



    -- get order number 获取订单号

    fetch ordernumbers into o;



    -- get the total for this order 计算订单金额

    call ordertotal(o,1,t);  # 参见23章代码，已创建可使用



    -- insert order and total into ordertotals 将订单号、金额插入表ordertotals内

    insert into ordertotals(order_num,total) values(o,t);



    -- end of loop

    until done end repeat;



    -- close the cursor 关闭游标

    close ordernumbers;



end // 

delimiter ;

调用存储过程 precessorders()

call processorders();

输出结果

select * from ordertotals;



#############################

 第25章 使用触发器      

#############################



-- 创建触发器 

create trigger newproduct after insert on products for each row select 'product added' into @new_pro;

 mysql 5.0以上版本在TRIGGER中不能返回结果集，定义了变量 @new_pro;

insert into products(prod_id,vend_id,prod_name,prod_price) values ('ANVNEW','1005','3 ton anvil','6.09'); # 插入一行 

select @new_pro;  # 显示Product added消息



-- 删除触发器 

drop trigger newproduct;



-- 使用触发器 

 insert触发器

create trigger neworder after insert on orders for each row select new.order_num into @order_num;

insert into orders(order_date,cust_id) values (now(),10001);

select @order_num;



 delete触发器

 使用OLD保存将要被删除的行到一个存档表中 

delimiter //

create trigger deleteorder before delete on orders for each row

begin

    insert into archive_orders(order_num,order_date,cust_id)

    values(old.order_num,old.order_date,old.cust_id); # 引用一个名为OLD的虚拟表，访问被删除的行

end //

delimiter ;



 update触发器

#在更新vendors表中的vend_state值时，插入前先修改为大写格式 

create trigger updatevendor before update on vendors 

for each row set new.vend_state = upper(new.vend_state);

 更新1001供应商的州为china

update vendors set vend_state = 'china' where vend_id =1001;

 查看update后数据，1001供应商对应的vend_state自动更新为大写的CHINA

select * from vendors;





## 第26章 管理事务处理 





-- 事务 transaction 指一组sql语句

-- 回退 rollback 指撤销指定sql语句的过程

-- 提交 commit 指将未存储的sql语句结果写入数据库表

-- 保留点 savepoint 指事务处理中设置的临时占位符，可以对它发布回退（与回退整个事务处理不同）



-- 控制事务处理

#开始事务及回退 

select * from ordertotals;   # 查看ordertotals表显示不为空

start transaction;           # 开始事务处理 

delete from ordertotals;     # 删除ordertotals表中所有行

select * from ordertotals;   # 查看ordertotals表显示 为空

rollback;                     # rollback语句回退 

select * from ordertotals;   # rollback后，再次查看ordertotals表显示不为空



#commit 提交 

start transaction;

delete from orderitems where order_num = 20010;

delete from orders where order_num = 20010;

commit;   # 仅在上述两条语句不出错时写出更改 



#savepoint 保留点 

 创建保留点

savepoint delete1;

 回退到保留点 

rollback to delete1;

 释放保留点 

release savepoint delete1;



-- 更改默认的提交行为 

set autocommit = 0;  # 设置autocommit为0（假）指示MySQL不自动提交更改




## 第27章 全球化和本地化





-- 字符集和校对顺序

 查看所支持的字符集完整列表

show character set;

 查看所支持校对的完整列表,以及它们适用的字符集

show collation;

 确定所用系统的字符集和校对

show variables like 'character%';

show variables like 'collation%';

 使用带子句的CREATE TABLE，给表指定字符集和校对

create table mytable

(

    column1 int,

    column2 varchar(10)

) default character set hebrew 

  collate hebrew_general_ci;

 除了能指定字符集和校对的表范围外，MySQL还允许对每个列设置它们

create table mytable

(

    column1 int,

    column2 varchar(10),

    column3 varchar(10) character set latin1 collate latin1_general_ci

)default character set hebrew 

 collate hebrew_general_ci;

 校对collate在对用ORDER BY子句排序时起重要的作用

 如果要用与创建表时不同的校对顺序排序,可在SELECT语句中说明 

select * from customers order by lastname,firstname collate latin1_general_cs;




## 第28章 安全管理

##



-- 管理用户

 需要获得所有用户账号列表时

 mysql数据库有一个名为user的表，它包含所有用户账号。user表有一个名为user的列

use mysql;

select user from user;



-- 创建用户账号 

 使用create user

create user ben identified by 'p@$$w0rd';

 重命名一个用户账号

rename user ben to bforta;

 删除用户账号 

drop user bforta;

 查看赋予用户账号的权限

show grants for bforta;

 允许用户在（crashcourse数据库的所有表）上使用SELECT，只读

grant select on crashcourse.* to bforta;

 重新查看赋予用户账号的权限，发生变化 

show grants for bforta;

 撤销特定的权限

revoke select on crashcourse.* from bforta;

 简化多次授权

grant select,insert on crashcourse.* to bforta;



-- 更改口令


 原来课本中使用的password()加密函数，在8.0版本中已经移除 

 password() :This function was removed in MySQL 8.0.11.

set password for bforta = 'n3w p@$$w0rd';  



-- 如果不指定用户名，直接修改当前登录用户的口令 

set password = 'n3w p@$$w0rd';



#############################

## 第29章 数据库维护 

#############################



 分析表 键状态是否正确

analyze table orders;

 检查表是否存在错误 

check table orders,orderitems;

check table orders,orderitems quick; # QUICK只进行快速扫描

 优化表OPTIMIZE TABLE，消除删除和更新造成的磁盘碎片，从而减少空间的浪费

optimize table orders;