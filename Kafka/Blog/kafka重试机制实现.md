# kafka 重试机制实现

> 原文：https://cloud.tencent.com/developer/article/1863357

## 概述

Spring-Kafka 提供消费重试的机制。当消息消费失败的时候，Spring-Kafka 会通过消费重试机制，重新投递该消息给 Consumer ，让 Consumer 重新消费消息 。

默认情况下，Spring-Kafka 达到配置的重试次数时，【每条消息的失败重试时间，由配置的时间隔决定】Consumer 如果依然消费失败 ，那么该消息就会进入到死信队列。

Spring-Kafka 封装了消费重试和死信队列， 将正常情况下无法被消费的消息称为死信消息（`Dead-Letter Message`），将存储死信消息的特殊队列称为死信队列（`Dead-Letter Queue`）。

我们在应用中可以对死信队列中的消息进行监控重发，来使得消费者实例再次进行消费，消费端需要做幂等性的处理。

------

## 二、Code

### （一）POM依赖

```xml
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot<groupId>
			<artifactId>spring-boot-starter-web<artifactId>
		dependency>

		
		<dependency>
			<groupId>org.springframework.kafka<groupId>
			<artifactId>spring-kafka<artifactId>
		dependency>

		<dependency>
			<groupId>org.springframework.boot<groupId>
			<artifactId>spring-boot-starter-test<artifactId>
			<scope>testscope>
		dependency>
		<dependency>
			<groupId>junitgroupId>
			<artifactId>junit<artifactId>
			<scope>testscope>
		dependency>
	dependencies>
```

------

### （二）配置文件

```javascript
spring:
  # Kafka 配置项，对应 KafkaProperties 配置类
  kafka:
    bootstrap-servers: 192.168.126.140:9092 # 指定 Kafka Broker 地址，可以设置多个，以逗号分隔
    # Kafka Producer 配置项
    producer:
      acks: 1 # 0-不应答。1-leader 应答。all-所有 leader 和 follower 应答。
      retries: 3 # 发送失败时，重试发送的次数
      key-serializer: org.apache.kafka.common.serialization.StringSerializer # 消息的 key 的序列化
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer # 消息的 value 的序列化

    # Kafka Consumer 配置项
    consumer:
      auto-offset-reset: earliest # 设置消费者分组最初的消费进度为 earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      properties:
        spring:
          json:
            trusted:
              packages: com.artisan.springkafka.domain
    # Kafka Consumer Listener 监听器配置
    listener:
      missing-topics-fatal: false # 消费监听接口监听的主题不存在时，默认会报错。所以通过设置为 false ，解决报错

logging:
  level:
    org:
      springframework:
        kafka: ERROR # spring-kafka
      apache:
        kafka: ERROR # kafka
```

### （三）配置类

首先要写一个配置类，用于处理消费异常 ErrorHandler

```java
package com.artisan.springkafka.configuration;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.listener.*;
import org.springframework.util.backoff.BackOff;
import org.springframework.util.backoff.FixedBackOff;

/**
 * @author 小工匠
 * @version 1.0
 * @description: TODO
 * @date 2021/2/18 14:32
 * @mark: show me the code , change the world
 */

@Configuration
public class KafkaConfiguration {
    private Logger logger = LoggerFactory.getLogger(getClass());


    @Bean
    @Primary
    public ErrorHandler kafkaErrorHandler(KafkaTemplate<?, ?> template) {

        logger.warn("kafkaErrorHandler begin to Handle");

        // <1> 创建 DeadLetterPublishingRecoverer 对象
        ConsumerRecordRecoverer recoverer = new DeadLetterPublishingRecoverer(template);
        // <2> 创建 FixedBackOff 对象   设置重试间隔 10秒 次数为 3次
        BackOff backOff = new FixedBackOff(10 * 1000L, 3L);
        // <3> 创建 SeekToCurrentErrorHandler 对象
        return new SeekToCurrentErrorHandler(recoverer, backOff);
    }



//    @Bean
//    @Primary
//    public BatchErrorHandler kafkaBatchErrorHandler() {
//        // 创建 SeekToCurrentBatchErrorHandler 对象
//        SeekToCurrentBatchErrorHandler batchErrorHandler = new SeekToCurrentBatchErrorHandler();
//        // 创建 FixedBackOff 对象
//        BackOff backOff = new FixedBackOff(10 * 1000L, 3L);
//        batchErrorHandler.setBackOff(backOff);
//        // 返回
//        return batchErrorHandler;
//    }
}
```

Spring-Kafka 通过实现自定义的 SeekToCurrentErrorHandler ，当 Consumer 消费消息异常的时候，进行拦截处理：

- 重试小于最大次数时，重新投递该消息给 Consumer
- 重试到达最大次数时，如果Consumer 还是消费失败时，该消息就会发送到死信队列。 死信队列的 命名规则为： 原有 Topic + .DLT 后缀 = 其死信队列的 Topic

```javascript
 ConsumerRecordRecoverer recoverer = new DeadLetterPublishingRecoverer(template);
```

创建 DeadLetterPublishingRecoverer 对象，它负责实现，在重试到达最大次数时，Consumer 还是消费失败时，该消息就会发送到死信队列。

```javascript
BackOff backOff = new FixedBackOff(10 * 1000L, 3L);
```

![img](kafka%E9%87%8D%E8%AF%95%E6%9C%BA%E5%88%B6%E5%AE%9E%E7%8E%B0.resource/7000.png)

也可以选择 BackOff 的另一个子类 ExponentialBackOff 实现，提供指数递增的间隔时间

```javascript
new SeekToCurrentErrorHandler(recoverer, backOff);
```

创建 SeekToCurrentErrorHandler 对象，负责处理异常，串联整个消费重试的整个过程。

------

### （四）SeekToCurrentErrorHandler

在消息消费失败时，`SeekToCurrentErrorHandler` 会将 调用 Kafka Consumer 的 `seek(TopicPartition partition, long offset)` 方法，将 Consumer 对于该消息对应的 TopicPartition 分区的本地进度设置成该消息的位置。

这样，Consumer 在下次从 Kafka Broker 拉取消息的时候，又能重新拉取到这条消费失败的消息，并且是第一条。

同时，Spring-Kafka 使用 FailedRecordTracker 对每个 Topic 的每个 TopicPartition 消费失败次数进行计数，这样相当于对该 TopicPartition 的第一条消费失败的消息的消费失败次数进行计数。

另外，在 `FailedRecordTracker` 中，会调用 BackOff 来进行计算，该消息的下一次重新消费的时间，通过 `Thread#sleep(...)` 方法，实现重新消费的时间间隔。

注意：

> FailedRecordTracker 提供的计数是客户端级别的，重启 JVM 应用后，计数是会丢失的。所以，如果想要计数进行持久化，需要自己重新实现下 FailedRecordTracker 类，通过 ZooKeeper 存储计数。 

SeekToCurrentErrorHandler 是只针对消息的单条消费失败的消费重试处理。如果想要有消息的批量消费失败的消费重试处理，可以使用 SeekToCurrentBatchErrorHandler 。配置方式如下

```javascript
@Bean
@Primary
public BatchErrorHandler kafkaBatchErrorHandler() {
    // 创建 SeekToCurrentBatchErrorHandler 对象
    SeekToCurrentBatchErrorHandler batchErrorHandler = new SeekToCurrentBatchErrorHandler();
    // 创建 FixedBackOff 对象
    BackOff backOff = new FixedBackOff(10 * 1000L, 3L);
    batchErrorHandler.setBackOff(backOff);
    // 返回
    return batchErrorHandler;
}
```

复制

`SeekToCurrentBatchErrorHandler` 暂时不支持死信队列的机制。

------

### （五）自定义逻辑处理消费异常

支持自定义 ErrorHandler 或 BatchErrorHandler 实现类，实现对消费异常的自定义的逻辑

比如 https://github.com/spring-projects/spring-kafka/blob/master/spring-kafka/src/main/java/org/springframework/kafka/listener/LoggingErrorHandler.java

```javascript
public class LoggingErrorHandler implements ErrorHandler {

	private static final LogAccessor LOGGER = new LogAccessor(LogFactory.getLog(LoggingErrorHandler.class));

	@Override
	public void handle(Exception thrownException, ConsumerRecord<?, ?> record) {
		LOGGER.error(thrownException, () -> "Error while processing: " + ObjectUtils.nullSafeToString(record));
	}

}
```

配置方式同 `SeekToCurrentErrorHandler` 或 `SeekToCurrentBatchErrorHandler`。

------

### （六）生产者

```javascript
   package com.artisan.springkafka.producer;

import com.artisan.springkafka.constants.TOPIC;
import com.artisan.springkafka.domain.MessageMock;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.support.SendResult;
import org.springframework.stereotype.Component;
import org.springframework.util.concurrent.ListenableFuture;

import java.util.Random;
import java.util.concurrent.ExecutionException;

/**
 * @author 小工匠
 * @version 1.0
 * @description: TODO
 * @date 2021/2/17 22:25
 * @mark: show me the code , change the world
 */

@Component
public class ArtisanProducerMock {
    @Autowired
    private KafkaTemplate<Object,Object> kafkaTemplate ;

    public ListenableFuture<SendResult<Object, Object>> sendMsgASync()  {
        // 模拟发送的消息
        Integer id = new Random().nextInt(100);
        MessageMock messageMock = new MessageMock(id,"messageSendByAsync-" + id);
        // 异步发送消息
        ListenableFuture<SendResult<Object, Object>> result = kafkaTemplate.send(TOPIC.TOPIC, messageMock);
        return result ;

    }

}
```



------

### （七）消费者

```javascript
 package com.artisan.springkafka.consumer;

import com.artisan.springkafka.domain.MessageMock;
import com.artisan.springkafka.constants.TOPIC;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;

/**
 * @author 小工匠
 * @version 1.0
 * @description: TODO
 * @date 2021/2/17 22:33
 * @mark: show me the code , change the world
 */

@Component
public class ArtisanCosumerMock {


    private Logger logger = LoggerFactory.getLogger(getClass());
    private static final String CONSUMER_GROUP_PREFIX = "MOCK-A" ;

    @KafkaListener(topics = TOPIC.TOPIC ,groupId = CONSUMER_GROUP_PREFIX + TOPIC.TOPIC)
    public void onMessage(MessageMock messageMock){
        logger.info("【接受到消息][线程:{} 消息内容：{}]", Thread.currentThread().getName(), messageMock);

        // 模拟抛出一次一行
        throw new RuntimeException("MOCK Handle Exception Happened");
    }

}
```

在消费消息时候，抛出一个 RuntimeException 异常，模拟消费失败

------

### （八）单元测试

```javascript
 package com.artisan.springkafka.produceTest;

import com.artisan.springkafka.SpringkafkaApplication;
import com.artisan.springkafka.producer.ArtisanProducerMock;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.kafka.support.SendResult;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.util.concurrent.ListenableFuture;
import org.springframework.util.concurrent.ListenableFutureCallback;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.TimeUnit;

/**
 * @author 小工匠
 * * @version 1.0
 * @description: TODO
 * @date 2021/2/17 22:40
 * @mark: show me the code , change the world
 */

@RunWith(SpringRunner.class)
@SpringBootTest(classes = SpringkafkaApplication.class)
public class ProduceMockTest {

    private Logger logger = LoggerFactory.getLogger(getClass());


    @Autowired
    private ArtisanProducerMock artisanProducerMock;


    @Test
    public void testAsynSend() throws ExecutionException, InterruptedException {
        logger.info("开始发送");

        artisanProducerMock.sendMsgASync().addCallback(new ListenableFutureCallback<SendResult<Object, Object>>() {
            @Override
            public void onFailure(Throwable throwable) {
                logger.info(" 发送异常{}]]", throwable);
            }

            @Override
            public void onSuccess(SendResult<Object, Object> objectObjectSendResult) {
                logger.info("回调结果 Result =  topic:[{}] , partition:[{}], offset:[{}]",
                        objectObjectSendResult.getRecordMetadata().topic(),
                        objectObjectSendResult.getRecordMetadata().partition(),
                        objectObjectSendResult.getRecordMetadata().offset());
            }
        });


        // 阻塞等待，保证消费
        new CountDownLatch(1).await();

    }

}
```

复制

------

## 测速结果

我们把这个日志来梳理一下

```javascript
2021-02-18 16:18:08.032  INFO 25940 --- [           main] c.a.s.produceTest.ProduceMockTest        : 开始发送
2021-02-18 16:18:08.332  INFO 25940 --- [ad | producer-1] c.a.s.produceTest.ProduceMockTest        : 回调结果 Result =  topic:[C_RT_TOPIC] , partition:[0], offset:[0]
2021-02-18 16:18:08.371  INFO 25940 --- [ntainer#0-0-C-1] c.a.s.consumer.ArtisanCosumerMock        : 【接受到消息][线程:org.springframework.kafka.KafkaListenerEndpointContainer#0-0-C-1 消息内容：MessageMock{id=15, name='messageSendByAsync-15'}]
2021-02-18 16:18:18.384 ERROR 25940 --- [ntainer#0-0-C-1] essageListenerContainer$ListenerConsumer : Error handler threw an exception

......
......
......

2021-02-18 16:18:18.388  INFO 25940 --- [ntainer#0-0-C-1] c.a.s.consumer.ArtisanCosumerMock        : 【接受到消息][线程:org.springframework.kafka.KafkaListenerEndpointContainer#0-0-C-1 消息内容：MessageMock{id=15, name='messageSendByAsync-15'}]
2021-02-18 16:18:28.390 ERROR 25940 --- [ntainer#0-0-C-1] essageListenerContainer$ListenerConsumer : Error handler threw an exception

......
......
......

2021-02-18 16:18:28.394  INFO 25940 --- [ntainer#0-0-C-1] c.a.s.consumer.ArtisanCosumerMock        : 【接受到消息][线程:org.springframework.kafka.KafkaListenerEndpointContainer#0-0-C-1 消息内容：MessageMock{id=15, name='messageSendByAsync-15'}]
2021-02-18 16:18:38.395 ERROR 25940 --- [ntainer#0-0-C-1] essageListenerContainer$ListenerConsumer : Error handler threw an exception

......
......
......

2021-02-18 16:18:38.399  INFO 25940 --- [ntainer#0-0-C-1] c.a.s.consumer.ArtisanCosumerMock        : 【接受到消息][线程:org.springframework.kafka.KafkaListenerEndpointContainer#0-0-C-1 消息内容：MessageMock{id=15, name='messageSendByAsync-15'}]
```

复制

清晰了么 老兄？

是不是和我们设置的消费重试

```javascript
 BackOff backOff = new FixedBackOff(10 * 1000L, 3L);
```

复制

10秒 重试3次

3次处理后依然失败，转入死信队列

看看数据

![img](kafka%E9%87%8D%E8%AF%95%E6%9C%BA%E5%88%B6%E5%AE%9E%E7%8E%B0.resource/7000-1679412687089-1.png)

------

# 源码地址

https://github.com/yangshangwei/boot2/tree/master/springkafkaRetries