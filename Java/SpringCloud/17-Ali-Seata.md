# 简介
### 解决问题
1. 分布式前Java服务与数据库1->1
2. 分布式后 1->1,1>多,多->多
保证多个服务之间的数据一致性
### 是什么
Seata 是一款开源的分布式事务解决方案，致力于提供高性能和简单易用的分布式事务服务。Seata 将为用户提供了 AT、TCC、SAGA 和 XA 事务模式，为用户打造一站式的分布式解决方案。
### 官网
http://seata.io/zh-cn/
### 处理过程
<img src="imgs/seata.png">

###### 一ID+三组件
1. id
全局唯一的事务ID
2. 3组件
    1. TC - 事务协调者
    维护全局和分支事务的状态，驱动全局事务提交或回滚。
    2. TM - 事务管理器
    定义全局事务的范围：开始全局事务、提交或回滚全局事务。
    3. RM - 资源管理器
    管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。
###### 处理过程
1. TM向TC申请开启一个全局事务，全局事务创建成功并生成一个全局唯一的XID
2. XID在微服务调用链路的上下文中传播
3. RM向TC注册分支事务，将其纳入XID对应全局事务的管辖
4. TM 向 TC 发起针对 XID 的全局提交或回滚请求
5. TC 调度 XID 下管辖的全部分支事务完成提交或回滚请求

# 安装
1. 下载
https://github.com/seata/seata/releases
2. 找到conf下的 file.conf 
将 mode 改为 db代表将日志存储到数据库
修改数据库账号密码端口
找到 register.conf
将 registry 与 config 里的 type均改为nacos
同时修改两者下面的 nacos信息
3. 创建数据库 seata

4. 数据库加载文件
    查看RANDME.MD server 对应网址即可
    1. https://github.com/seata/seata/tree/develop/script/client下db中的mysql

# 实验
### 数据库
1. 创建数据库
    1. create database seata_order;订单
    2. create database seata_storage;库存
    3. create database seata_account;账户信息
2. 建表
    1. seata_order下建t_order
    2. seata_storage下建 t_storage;
    3. seata_account下建 t_account
3. 建表sql
```sql
create table t_order(
    id bigint(11) not null auto_increment primary key,
    user_id bigint(11) default null comment '用户id',
    product_id bigint(11) default null comment '产品id',
    count  int(11) default null comment '数量',
    money decimal(11,0) default null comment '余额',
    status int(1) default null comment '订单状态'
)
```
### 建模块
###### seata-order-service2001
1. pom
```xml
<!-- seata -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-alibaba-seata</artifactId>
    <exclusions>
        <exclusion>
            <groupId>io.seata</groupId>
            <artifactId>seata-all</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<!-- 引入与自己版本相同的 -->
<dependency>
    <groupId>io.seata</groupId>
    <artifactId>seata-all</artifactId>
    <version>1.2.0</version>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```


########################## 
1. 依赖
```xml
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
            <exclusions>
                <exclusion>
                    <groupId>io.seata</groupId>
                    <artifactId>seata-spring-boot-starter</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <!-- 引入与自己版本相同的 -->
        <dependency>
            <groupId>io.seata</groupId>
            <artifactId>seata-spring-boot-starter</artifactId>
            <version>1.2.0</version>
        </dependency>
```
2. yml

3. config.txt nacos-config.sh 上传配置