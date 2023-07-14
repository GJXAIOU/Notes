# 一次访问Redis延时高问题排查与总结

## 背景

20230308 在某地域进行了线上压测, 发现接口RT频繁超时, 性能下降严重, P50 400ms+, P90 1200ms+, P99 2000ms+。

细致排查发现其中重要的原因是, **访问缓存rt竟然飙到了1.2s左右**。

作为高性能爱好者, 榨干CPU的每一分价值是我们的宗旨, 是可忍孰不可忍, 怎么能光空转, 不干活呢? 那就仔细分析下问题。

## 一、为啥Redis访问延时如此高?

我们简化下Redis访问流程如下:

![图片](alibaba_%E4%B8%80%E6%AC%A1%E8%AE%BF%E9%97%AERedis%E5%BB%B6%E6%97%B6%E9%AB%98%E9%97%AE%E9%A2%98%E6%8E%92%E6%9F%A5%E4%B8%8E%E6%80%BB%E7%BB%93%20.resource/640.png)

### （一）可能性1: 服务端问题?

- 我们Redis使用的是 redis_amber_master_4xlarge_multithread 16C32G+480G SSD 规格, 最大 QPS 参考值24w, 最大连接数3w, 配置还是非常豪华的。
- 如下, QPS以及Load在峰值请求阶段, 都仍然处于低位。

![图片](alibaba_%E4%B8%80%E6%AC%A1%E8%AE%BF%E9%97%AERedis%E5%BB%B6%E6%97%B6%E9%AB%98%E9%97%AE%E9%A2%98%E6%8E%92%E6%9F%A5%E4%B8%8E%E6%80%BB%E7%BB%93%20.resource/640-20230621172803290.png)

### （二）可能性2: 物理网络问题?

如下, 请求远远没有达到机器带宽, 不是瓶颈. 另外单独看了网卡重传率等指标, 也都正常。

![图片](alibaba_%E4%B8%80%E6%AC%A1%E8%AE%BF%E9%97%AERedis%E5%BB%B6%E6%97%B6%E9%AB%98%E9%97%AE%E9%A2%98%E6%8E%92%E6%9F%A5%E4%B8%8E%E6%80%BB%E7%BB%93%20.resource/640-20230621172803079.png)

### （三）可能性3: 客户端问题?

那么很大概率就是客户端自身问题了. 我们把客户端详细放大如下:

![图片](alibaba_%E4%B8%80%E6%AC%A1%E8%AE%BF%E9%97%AERedis%E5%BB%B6%E6%97%B6%E9%AB%98%E9%97%AE%E9%A2%98%E6%8E%92%E6%9F%A5%E4%B8%8E%E6%80%BB%E7%BB%93%20.resource/640-20230621172802034.jpeg)

#### **JVM FGC STW?**

根据当时ARMS监控结果如下, 虽然YGC次数与耗时有所上升, 但没有发生FGC:

![图片](alibaba_%E4%B8%80%E6%AC%A1%E8%AE%BF%E9%97%AERedis%E5%BB%B6%E6%97%B6%E9%AB%98%E9%97%AE%E9%A2%98%E6%8E%92%E6%9F%A5%E4%B8%8E%E6%80%BB%E7%BB%93%20.resource/640-20230621172803280.png)

#### **JedisPool问题?**

把内存Dump出来, 分析JedisConnectionFactory几个相关重要指标, 发现问题有如下2个: 

1. **maxBorrowWaitTimeMills 过大**: 即最大等待时间过久。在等待从连接池中获取连接, **最大等待了1200ms**。很大概率是因为block在连接池获取, 导致请求处理缓慢。
2. Redis连接创建销毁次数过多: **createdCount 11555次; destroyedCount: 11553次。**说明max-idle参数设置不合理(on return的时候检查idle是否大于maxIdle, 如果大于则直接销毁该连接)。每个对象的创建就是一次TCP连接的创建, 开销较大。导致脉冲式请求过来时引发频繁创建/销毁, 也会影响整体性能。

![图片](alibaba_%E4%B8%80%E6%AC%A1%E8%AE%BF%E9%97%AERedis%E5%BB%B6%E6%97%B6%E9%AB%98%E9%97%AE%E9%A2%98%E6%8E%92%E6%9F%A5%E4%B8%8E%E6%80%BB%E7%BB%93%20.resource/640-20230621172803620.png)

![图片](alibaba_%E4%B8%80%E6%AC%A1%E8%AE%BF%E9%97%AERedis%E5%BB%B6%E6%97%B6%E9%AB%98%E9%97%AE%E9%A2%98%E6%8E%92%E6%9F%A5%E4%B8%8E%E6%80%BB%E7%BB%93%20.resource/640-20230621172803313.png)

顺便说一句: maxBorrowWaitTimeMills, createdCount, destroyedCount **几个metrics信息是JedisPool对象持久维护的全局变量信息, 只要JVM不重启, 这个信息就会一直存在。**这也就是为啥不需要在压测峰值时获取内存dump, 而是事后dump也可以。

此外, 如果细致探索JedisPool参数工作机制, 就需要了解apache的ObjectPool2的机制。刚好笔者在之前研究过ObjectPool, 后续会出单独文章阐述&对比ObjectPool, ObjectPool2, JedisPool以及经常踩坑的DruidPool的实现原理与差异。

本文就不再赘述, 敬请期待~

至此, 定位问题是JedisPool行为异常导致。

## 如何解决问题?

### 线上JedisPool实际参数

> 部分参数是由 redis.clients.jedis.JedisPoolConfig 继承而来

```
spring.redis.jedis.pool.max-active=100spring.redis.jedis.pool.max-idle=16
spring.redis.jedis.pool.time-between-eviction-runs-millis=30000
spring.redis.jedis.pool.min-idle=0spring.redis.jedis.pool.test-while-idle=truespring.redis.jedis.pool.num-tests-per-eviction-run=-1spring.redis.jedis.pool.min-evictable-idle-time-millis=60000
```

### 参数行为解析

1. max-active: 连接池的最大数量为100, 包括 idle + active. 注意, **这里spring.redis.jedis.pool.max-active被映射为了ObjectPool的maxTotal参数上。**
2. 连接池的最大空闲数量为16, **即如果return时, idleObject>=16, 则该对象直接被销毁。**
3. 启动后台线程, 每30s执行一次, 定时心跳保活与检测。
4. 连接池最小空闲的连接数量为0. 即corePoolSize为0, 不会长期maintain一个固定的容量。

### 脉冲式请求引发的问题

我们把问题简化为如下序列, 即可发现问题所在. 在T2~T3内, 84个对象创建, 84个对象销毁. 造成了极大的损耗。

![图片](alibaba_%E4%B8%80%E6%AC%A1%E8%AE%BF%E9%97%AERedis%E5%BB%B6%E6%97%B6%E9%AB%98%E9%97%AE%E9%A2%98%E6%8E%92%E6%9F%A5%E4%B8%8E%E6%80%BB%E7%BB%93%20.resource/640-20230621172802573.png)

### 期望的行为模式

由于线上环境, Redis服务器配置较高, 为了能充分压榨性能, 同时应对容器场景下典型的突发峰值, 因此如下行为: 

1. 连接池的最大数量=连接池的最小数量=连接池的稳定数量. **即不要临时去创建连接, 防止等待过久。**
2. 需要定时心跳保活与检测, 及时删除掉超时/无效的连接。
3. 不要因为idle时间过久而重建连接(只因为连接失效而重建)。防止无意义的大规模连接重建。

```
spring.redis.jedis.pool.max-active=500 // 线上稳定保有4台, 4*500=2000, 仍然远小于Redis规格支持的3wspring.redis.jedis.pool.max-idle=500
spring.redis.jedis.pool.time-between-eviction-runs-millis=30000 // 定时心跳保活与检测
spring.redis.jedis.pool.min-idle=500 // 连接池的稳定数量spring.redis.jedis.pool.test-while-idle=true //定时心跳保活与检测spring.redis.jedis.pool.num-tests-per-eviction-run=-1 // 每次保活检测, 都需要把500个连接都检测一遍. 如果设置为-2, 则每次检测1/2比例的的连接.spring.redis.jedis.pool.min-evictable-idle-time-millis=-1 // 不要因为idleTime大于某个阈值从而把连接给删除掉. 这样可以防止无意义的大规模连接重建。
```

## 效果验证

终于在20230413重新迎来了一波压测, 流量模型与上次相同。结果如下: 

- maxBorrowWaitTimeMills 下降比例接近 80%
- createdCount 也从之前的 11555次 下降到了 500次(即池子初始化的size)
- 业务侧整体性能也大幅提升, P50与P90均下降了将近60%, P99更是夸张地下降了70%。简直是amazing, 完结撒花!~

![图片](alibaba_%E4%B8%80%E6%AC%A1%E8%AE%BF%E9%97%AERedis%E5%BB%B6%E6%97%B6%E9%AB%98%E9%97%AE%E9%A2%98%E6%8E%92%E6%9F%A5%E4%B8%8E%E6%80%BB%E7%BB%93%20.resource/640-20230621172803057.png)

![图片](alibaba_%E4%B8%80%E6%AC%A1%E8%AE%BF%E9%97%AERedis%E5%BB%B6%E6%97%B6%E9%AB%98%E9%97%AE%E9%A2%98%E6%8E%92%E6%9F%A5%E4%B8%8E%E6%80%BB%E7%BB%93%20.resource/640-20230621172803330.png)

![图片](alibaba_%E4%B8%80%E6%AC%A1%E8%AE%BF%E9%97%AERedis%E5%BB%B6%E6%97%B6%E9%AB%98%E9%97%AE%E9%A2%98%E6%8E%92%E6%9F%A5%E4%B8%8E%E6%80%BB%E7%BB%93%20.resource/640-20230621172803169.png)

------





## 问答

- 大牛，"可能性2: 物理网络问题?”的网络监控工具是什么啊？

    sar 或tsar

- 这种配置比较适合脉冲式流量。如果比较业务流量波动比较平滑，还是推荐之前的配置![[奸笑]](alibaba_%E4%B8%80%E6%AC%A1%E8%AE%BF%E9%97%AERedis%E5%BB%B6%E6%97%B6%E9%AB%98%E9%97%AE%E9%A2%98%E6%8E%92%E6%9F%A5%E4%B8%8E%E6%80%BB%E7%BB%93%20.resource/pic_blank.gif)
- 最后的参数值是怎么得来的，经验还是逐步试验？
- max-idle=min-idle=max-active 没有缓冲空间吗？流量徒增的时候还是容易出现block问题，建议参考线程池的做法，以及可以模仿美团做成动态参数配置的就更好了
- 严格说latency和delay是不一样的，文章在讲latency问题，所以应该叫时延，而不是延时。是不是这样？
- 