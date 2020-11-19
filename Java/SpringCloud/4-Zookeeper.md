# 配置虚拟机和zookeeper
1. 虚拟机终端输入ifconfig查看 ens33 下的端口号
2. 查看主机与虚拟机之间通信是否畅通
    1. 虚拟机端口号：ifconfig查看 ens33 下的端口号
    2. 主机端口号：网络连接下的 VMnet8 的端口号
    3. 使用虚拟机ping主机，使用主机ping虚拟机确保都可以ping通
3. 新建项目
    1. pom
        ```xml
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
            </dependency>
        ```
4. 解压zookeeper
#tar -zxvf zookeeper-3.4.10.tar.gz
5. 启动 zookeeper
==注意此时应该修改conf里的zoo_sample.cfg名字为zoo.cfg==
[root@localhost ~]# cd /usr/local/zookeeper/zookeeper-3.4.14/bin
[root@localhost bin]# ./zkServer.sh start
6. 启用端口
[root@localhost bin]# ./zkCli.sh
Connecting to localhost:2181
# 新建项目
### pom文件
==注意：==
1. 排除zookeeper-discovery中自带的 zookeeper，同时引入与linux相同版本的 zookeeper
2. 排除引入 zookeeper 的日志，因为日志会会冲突
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
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
            <!-- 排除自身的zookeeper        -->
            <exclusions>
                <exclusion>
                    <groupId>org.apache.zookeeper</groupId>
                    <artifactId>zookeeper</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <!-- 添加zookeeper,与linux上的版本一致     -->
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.14</version>
            <exclusions>
                <exclusion>
                    <groupId>log4j</groupId>
                    <artifactId>log4j</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>
```

### yml 文件
```yml
server:
  port: 8004

spring:
  application:
    name: cloud-provider-payment
  cloud:
    zookeeper:
      # ip地址为linux中的网络接口，2181为zookeeper的默认端口
      connect-string: 192.168.150.66:2181
```
### linux测试
```
[zk: localhost:2181(CONNECTED) 3] ls /services
[cloud-provider-payment]
```
能看到服务名即为配置成功
### 测试2
1. http://localhost:8004/payment/zk
可以看到信息
2. linux 
```
# 获取服务名
[zk: localhost:2181(CONNECTED) 0] ls /services
[cloud-provider-payment]
# 获取流水号
[zk: localhost:2181(CONNECTED) 1] ls /services/cloud-provider-payment
[efc76371-522d-4d5d-8f56-f8fe4deb7a47]
# 获取详细信息
[zk: localhost:2181(CONNECTED) 2] get /services/cloud-provider-payment/efc76371-522d-4d5d-8f56-f8fe4deb7a47
{"name":"cloud-provider-payment","id":"efc76371-522d-4d5d-8f56-f8fe4deb7a47","address":"WINDOWS-N0GUAG7","port":8004,"sslPort":null,"payload":{"@class":"org.springframework.cloud.zookeeper.discovery.ZookeeperInstance","id":"application-1","name":"cloud-provider-payment","metadata":{}},"registrationTimeUTC":1590232919360,"serviceType":"DYNAMIC","uriSpec":{"parts":[{"value":"scheme","variable":true},{"value":"://","variable":false},{"value":"address","variable":true},{"value":":","variable":false},{"value":"port","variable":true}]}}
```
### 服务节点是临时还是持久
关闭8004后在linux终端中,一段时间后失去连接
```
[zk: localhost:2181(CONNECTED) 18] ls /services/cloud-provider-payment
[efc76371-522d-4d5d-8f56-f8fe4deb7a47]
[zk: localhost:2181(CONNECTED) 19] ls /services/cloud-provider-payment
[efc76371-522d-4d5d-8f56-f8fe4deb7a47]
[zk: localhost:2181(CONNECTED) 20] ls /services/cloud-provider-payment
[efc76371-522d-4d5d-8f56-f8fe4deb7a47]
[zk: localhost:2181(CONNECTED) 21] ls /services/cloud-provider-payment
[]
[zk: localhost:2181(CONNECTED) 22] 
```
再开启8004，再次查看流水号可以发现流水号跟之前的不一样 ，所以服务节点是临时的，在关闭服务后完全删除。
# 订单服务入住zookeeper
1. 新建订单 moudle
2. 该pom
3. 建yml
```yml
server:
  port: 80
spring:
  application:
    name: cloud-consumerzk-order80
  cloud:
    zookeeper:
      connect-string: 192.168.150.66:2181
```
4. 写主类
5. 配置类生成 RestTemplate
6. Controller 调用 8004
7. linux 输入，查看节点是否注册上
```
[zk: localhost:2181(CONNECTED) 1] ls /services
[cloud-provider-payment, cloud-consumerzk-order80]
```
8. 网址登陆查看
http://localhost:8004/payment/zk
http://localhost/consumer/payment/zk

# 存在问题
zookeeper集群?????????

