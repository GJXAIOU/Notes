本例使用seata1.2
### 创建seata数据库
1. 找到 seata/conf 下的 README-zh.md
2. 进入 [server](https://github.com/seata/seata/tree/develop/script/server)
    1. 找到 db 下的 mysql.sql
    2. 创建数据库 seata ，后执行mysql.sql
    3. mysql.sql中的三张表为 seata配置必须的表 
### 创建业务必须数据库
用以做案例
```sql
-- 建数据库，订单
create database seata_order;
use seata_order;
-- 建订单表
CREATE TABLE `t_order` (
	`id` INT(11) NOT NULL AUTO_INCREMENT,
	`name` VARCHAR(50) NULL DEFAULT NULL,
	PRIMARY KEY (`id`)
)
COLLATE='utf8_general_ci'
ENGINE=InnoDB
AUTO_INCREMENT=8
;
-- 建数据库，库存
create database seata_storage;
-- 建库存表
CREATE TABLE `t_storage` (
	`id` INT(11) NOT NULL,
	`num` INT(11) NOT NULL,
	PRIMARY KEY (`id`)
)
COLLATE='utf8_general_ci'
ENGINE=InnoDB
;
-- 初始化库存数量
insert into t_storage values(1,20);
```
1. 找到 seata/conf 下的 README-zh.md
2. 进入 [client](https://github.com/seata/seata/tree/develop/script/client) 
3. 找到db下的mysql.sql[client](https://github.com/seata/seata/tree/develop/script/client) 
4. 其中为一建表sql
5. ==每一个分布式业务数据库都需要这张表，即在新建的数据库 seata_order 与 seata_storage中新建该表==
### 修改seata1.2
1. 找到 seata/conf/file.conf
将 store 下的 mode 改为 db ，代表采用数据库配置
更改 store下数据库的相关配置
2. 找到  seata/conf/registry.conf
将 type 改为 nacos 同时修改 nacos中的信息和config下nacos的信息
```conf
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  type = "nacos"

  nacos {
    application = "seata-server"
    serverAddr = "localhost:8848"
    namespace = ""
    cluster = "default"
    username = "nacos"
    password = "nacos"
  }
}

config {
  # file、nacos 、apollo、zk、consul、etcd3
  type = "nacos"

  nacos {
    serverAddr = "localhost:8848"
    namespace = ""
    group = "SEATA_GROUP"
    username = "nacos"
    password = "nacos"
  }
}

```
### 为nacos添加配置信息
1. 访问 [config-center](https://github.com/seata/seata/tree/develop/script/config-center)
2. 将 config.txt 拷贝到 seata/下
3. 修改config.txt内容为下列，因为其余配置为默认或无用配置
```
service.vgroupMapping.my_test_tx_group=default
store.mode=db
store.db.datasource=druid
store.db.dbType=mysql
store.db.driverClassName=com.mysql.jdbc.Driver
store.db.url=jdbc:mysql://127.0.0.1:3306/seata?useUnicode=true
store.db.user=root
store.db.password=root
store.db.minConn=5
store.db.maxConn=30
store.db.globalTable=global_table
store.db.branchTable=branch_table
store.db.queryLimit=100
store.db.lockTable=lock_table
store.db.maxWait=5000
```
4. 将 nacos 下的 nacos-config.sh 拷贝到 seata/conf/ 下
5. 启动nacos
6. 使用 Git Bash Here 切换到 seata/conf/ 下执行命令``sh nacos-config.sh``
7. 查看nacos中是否有 seata 相关的配置信息。
### 配置业务seata-order-service2001
1. pom
```xml
    <dependencies>
        <!--引入自己的公共api-->
        <dependency>
            <groupId>com.wxh.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--热部署-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <!-- 使用openfeign做微服务调用 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
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
    </dependencies>
```
2. yml
```yml
# 端口号
server:
  port: 2001
spring:
  application:
    name: seata-order-service
  cloud:
    nacos:
      discovery: #Nacos注册中心地址
        server-addr: localhost:8848
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource  #数据源类型
    driver-class-name: org.gjt.mm.mysql.Driver    #mysql驱动包
    url: jdbc:mysql://localhost:3306/seata_order?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: root
feign:
  hystrix:
    enabled: true

logging:
  level:
    io:
      seata: info
mybatis:
  mapper-locations: classpath:mapper/*.xml

seata:
  enabled: true
  # 应用 id 为唯一便于区分
  application-id: order
  # 事务分组，这个是默认分组
  tx-service-group: my_test_tx_group
  config:
    type: nacos
    nacos:
      namespace:
      serverAddr: 127.0.0.1:8848
      group: SEATA_GROUP
      userName: "nacos"
      password: "nacos"
  registry:
    type: nacos
    nacos:
      application: seata-server
      server-addr: 127.0.0.1:8848
      namespace:
      userName: "nacos"
      password: "nacos"

```
3. mapper
```java
@Mapper
public interface OrderMapper {
    // 插入一条订单
    @Insert("insert into t_order values(null,'test')")
    public void test();
}
```
4. service：接口类省略
```java
@Service
public class OrderServiceImpl implements OrderService {

    @Resource
    OrderMapper orderMapper;
    @Resource
    StroageService stroageService;

    @Override
    // name对应配置文件里的事务分组
    @GlobalTransactional(name = "my_test_tx_group",rollbackFor = Exception.class)
    public void test() {
        orderMapper.test();
        stroageService.test();
    }
}
```
5. StorageService.java
```java
@FeignClient(value = "seata-storage-service")
public interface StroageService {
    @RequestMapping("/test")
    public String test();
}
```
6. controller
```java
    @Resource
    private OrderService orderService;
    @RequestMapping("/test")
    public String test(){
        orderService.test();
        return "test";
    }
```
### 配置业务模块seata-storage-service2003
1. pom 与上一模块相同
2. yml 与上一模块相同
    1. 更改端口号
    2. 更改spring.application.name.
    3. 更改seata.application-id: storage
3. 为了方便省去service由controller直接调用mapper
4. mapper
```java
@Mapper
public interface StorageMapper {
    // 代表库存减1
    @Update("UPDATE t_storage SET num = num-1 WHERE id = 1")
    public void test();
}
```
5. controller
```java
@RestController
public class StorageController {
    @Resource
    StorageMapper storageMapper;

    @RequestMapping("/test")
    public String test(){

        // int a = 1/0; // 模拟异常

        storageMapper.test();
        return "storage";
    }
}
```
### 测试
1. 正常启动两个模块
2. 访问http://localhost:2001/test查看是否能成功访问
3. 启动模拟异常
4. 将模块 seata-order-service2001 中 OrderServiceImpl.class 下的 stroageService.test();打上断点
5. debug 启动seata-order-service2001
6. 访问 http://localhost:2001/test 
7. 此时进入断点
8. 查看 t_order 表中新增一条数据，数据库中 seata_order 下的 undo_log 表中增加信息，代表事务
9. 数据库 seata_order 的 undo_log 表中添加事务信息
10. 结束执行代码
11. t_order 表中数据消失
12. 数据库 seata_order 的 undo_log 表数据消失
13. 测试成功

### 修改事务分组
1. yml中
seata:
  tx-service-group: test
2. nacos中新增配置
service.vgroupMapping.test
内容为 default
3. service.vgroupMapping.test 的值为集群名称，test即事务分组 ，default代表无集群
