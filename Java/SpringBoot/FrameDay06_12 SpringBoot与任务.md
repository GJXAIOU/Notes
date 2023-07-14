# 四、Spring Boot与任务

## 一、异步任务

在 Java 应用中，绝大多数情况下都是通过同步的方式来实现交互处理的；但是在处理与第三方系统交互的时候，容易造成响应迟缓的情况，之前大部分都是使用多线程来完成此类任务，其实，在 Spring 3.x 之后，就已经内置了`@Async` 来完美解决这个问题。

两个注解：`@EnableAysnc`、`@Aysnc`

即如果想让一个方法异步执行，需要在该方法上加上 `@Aysnc`，并且在主类上加上 `@EnableAysnc`

提供一个 Service 和 Controller 类

```java
package com.gjxaiou.service;

import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

@Service
public class AsyncService {
    // 告诉 Spring 这是一个异步方法，不加这个注解则访问页面之后 5s 之后才会响应。
    @Async
    public void hello() {
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("处理数据中...");
    }
}


// -------------------------------
package com.gjxaiou.controller;

import com.gjxaiou.service.AsyncService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class AsyncController {
    @Autowired
    AsyncService asyncService;

    @GetMapping("/hello")
    public String getHello() {
        asyncService.hello();
        return "success";
    }
}
```

同时在主类上加上   `@EnableAysnc` 注解。

## 二、定时任务 

项目开发中经常需要执行一些定时任务，比如需要在每天凌晨时候，分析一次前一天的日志信息。Spring为我们提供了异步执行任务调度的方式，提供TaskExecutor 、TaskScheduler 接口。

**两个注解：**`@EnableScheduling`、`@Scheduled`

**cron**表达式：

| **字段** | **允许值**             | **允许的特殊字符** |
| -------- | ---------------------- | ------------------ |
| 秒       | 0-59                   | , -  * /           |
| 分       | 0-59                   | , -  * /           |
| 小时     | 0-23                   | , -  * /           |
| 日期     | 1-31                   | , -  * ? / L W C   |
| 月份     | 1-12                   | , -  * /           |
| 星期     | 0-7或SUN-SAT  0,7是SUN | , -  * ? / L C #   |

| **特殊字符** | **代表含义**               |
| ------------ | -------------------------- |
| ,            | 枚举                       |
| -            | 区间                       |
| *            | 任意                       |
| /            | 步长                       |
| ?            | 日/星期冲突匹配            |
| L            | 最后                       |
| W            | 工作日                     |
| C            | 和calendar联系后计算过的值 |
| #            | 星期，4#2，第2个星期四     |

具体使用见：`ScheduledService`

```java
package com.gjxaiou.service;

import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Service;

/**
 * 定时任务测试
 */
@Service
public class ScheduledService {

    /**
     * second(秒), minute（分）, hour（时）, day of month（日）, month（月）, day of week（周几）.
     * 0 * * * * MON-FRI 周一到周五整分钟（秒数为 0）时候启动
     * 【0 0/5 14,18 * * ?】 每天14点整，和18点整，每隔5分钟执行一次
     * 【0 15 10 ? * 1-6】 每个月的周一至周六10:15分执行一次
     * 【0 0 2 ? * 6L】每个月的最后一个周六凌晨2点执行一次
     * 【0 0 2 LW * ?】每个月的最后一个工作日凌晨2点执行一次
     * 【0 0 2-4 ? * 1#1】每个月的第一个周一凌晨2点到4点期间，每个整点都执行一次；
     */
    // @Scheduled(cron = "0 * * * * MON-SAT")
    //@Scheduled(cron = "0,1,2,3,4 * * * * MON-SAT") // 周一到周四 0,1,2，3,4 秒都会启动
    // @Scheduled(cron = "0-4 * * * * MON-SAT")// 等价于上面
    @Scheduled(cron = "0/4 * * * * MON-SAT")  //从 0 秒启动，每4秒执行一次
    public void hello() {
        System.out.println("hello ... ");
    }
}
```

同时在主类上加上   `@EnableScheduling` 注解。

## 三、邮件任务 

邮件发送需要引入 `spring-boot-starter-mail`

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```

•Spring Boot 自动配置MailSenderAutoConfiguration

- 定义MailProperties内容，配置在 `application.yml/ application.properties`中

```properties
spring.mail.username=534096094@qq.com
# 首先开启 POP3 和 IMAP 等服务，同时在邮箱设置中生成「授权码」，填写的是授权码
spring.mail.password=gtstkoszjelabijb
spring.mail.host=smtp.qq.com
spring.mail.properties.mail.smtp.ssl.enable=true
```

测试类：

```java
package com.gjxaiou;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.mail.SimpleMailMessage;
import org.springframework.mail.javamail.JavaMailSenderImpl;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.test.context.junit4.SpringRunner;

import javax.mail.internet.MimeMessage;
import java.io.File;

@RunWith(SpringRunner.class)
@SpringBootTest
public class Springboot04TaskApplicationTests {

    @Autowired
    JavaMailSenderImpl mailSender;

    /**
     * 测试发送一个简单邮件
     */
    @Test
    public void contextLoads() {
        SimpleMailMessage message = new SimpleMailMessage();
        //邮件设置
        message.setSubject("通知-今晚开会");
        message.setText("今晚7:30开会");

        message.setTo("17512080612@163.com");
        message.setFrom("534096094@qq.com");

        mailSender.send(message);
    }

    /**
     * 测试发送复杂邮件信息
     *
     * @throws Exception
     */
    @Test
    public void test02() throws Exception {
        //1、创建一个复杂的消息邮件
        MimeMessage mimeMessage = mailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, true);

        //邮件设置
        helper.setSubject("通知-今晚开会");
        helper.setText("<b style='color:red'>今天 7:30 开会</b>", true);

        helper.setTo("17512080612@163.com");
        helper.setFrom("534096094@qq.com");

        //上传文件
        helper.addAttachment("1.jpg", new File("C:\\Users\\lfy\\Pictures\\Saved Pictures\\1.jpg"));
        helper.addAttachment("2.jpg", new File("C:\\Users\\lfy\\Pictures\\Saved Pictures\\2.jpg"));

        mailSender.send(mimeMessage);
    }
}

```

