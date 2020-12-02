# 简介
### 官网
https://spring.io/projects/spring-cloud-sleuth
### 监控链路调用
这包括将跟踪数据（跨度）报告到的位置，要保留（跟踪）多少个跟踪，是否发送了远程字段（行李）以及要跟踪哪些库。微服务中节点过多，使用它能更好的做监控。
### 安装Zipkin
1. Sleuth 负责链路监控，Zipkin负责展现

2. https://dl.bintray.com/openzipkin/maven/io/zipkin/java/zipkin-server/

3. 下载 exec.jar

4. 使用
命令行打开到jar包所在目录
java -jar zipkin-server-2.12.9-exec.jar
成功后访问http://localhost:9411/
# 使用
### 改变最原始的模块80与8001
1. 依赖
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```
2. yml
```yml
spring:
  zipkin:
    # 放到 zipkin上
    base-url: http://localhost:9411
  sleuth:
    sampler:
      # 采样率介于0-1之间，1表示全部采集
      probability: 1
```
3. controller 80
```java
    @GetMapping("/consumer/payment/zipkin")
    public String paymentZipkin(){
        String result = restTemplate.getForObject(PAYMENT_URL+"/payment/zipkin",String.class);
        return result;
    }
```
4. controller 8001
```java
    @GetMapping("/payment/zipkin")
    public String paymentZipkin(){
        return "我是 zipkin";
    }
```
5. 测试
依次打开7001,8001,80
访问80
访问http://localhost:9411/可以查看到访问的链路
