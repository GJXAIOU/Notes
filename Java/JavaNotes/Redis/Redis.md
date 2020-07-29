---
tags:[redis]
style: summer
---



# Redis（Remote Dictionary Server） 

[TOC]

## Redis 简介

**Redis 是一个高性能非关系型（NoSQL）键值对的内存数据库**，可以用在缓存、数据库、消息中间件等。**可以存储键和五种不同类型的值之间的映射。其中键的类型只能为字符串，值支持五种数据类型：字符串、列表、散列表、集合、有序集合**。Redis 支持很多特性，例如将内存中的数据持久化到硬盘中，使用复制来扩展读性能，使用分片来扩展写性能。

==使用缓存 Redis 作用：==

- 提高性能：缓存查询速度比数据库查询速度快（内存 VS 硬盘）
- 提高并发能力：缓存分担了部分请求支持更高的并发

## Redis 是单进程单线程的原因

https://juejin.im/post/5ebcb20d5188256d4c558a65

注意：redis 单线程指的是**网络请求模块使用了一个线程，即一个线程处理所有网络请求**，其他模块仍用了多个线程。

==**Redis 是单进程单线程的，Redis 利用队列技术将并发访问变为串行访问，消除了传统数据库串行控制的开销。**==

多线程处理会涉及到锁，而且多线程处理会涉及到线程切换而消耗CPU。因为 CPU不是Redis的瓶颈，Redis的瓶颈最有可能是机器内存或者网络带宽。既然单线程容易实现，而且CPU不会成为瓶颈，那就顺理成章地采用单线程的方案了。单线程无法发挥多核CPU性能，不过可以通过在单机开多个Redis实例来解决。

## Redis 为什么这么快

- **完全基于内存，绝大部分请求是纯粹的内存操作，内存的读取速度非常快速**。数据存在内存中，类似于HashMap，HashMap的优势就是查找和操作的时间复杂度都是O(1)；

- 数据结构简单，对数据操作也简单，Redis中的数据结构是专门进行设计的；

- **采用单线程，避免了不必要的上下文切换和竞争条件，也不存在多进程或者多线程导致的切换而消耗 CPU**，不用去考虑各种锁的问题，不存在加锁释放锁操作，没有因为可能出现死锁而导致的性能消耗；

- 使用多路I/O复用模型，可以处理并发的连接；保证在多连接的时候， 系统的高吞吐量。

    https://mp.weixin.qq.com/s?__biz=MzI4Njc5NjM1NQ==&mid=2247488320&idx=1&sn=2b8f1ffc06553d1e43419bc1958c61d1&chksm=ebd62c6cdca1a57af0dfbbe4ac41da68d894ac2913028929b0b65329811d76786c9113308046&scene=21#wechat_redirect

    **多路指的是多个socket连接，复用指的是复用一个线程**。多路复用主要有三种技术：select，poll，epoll。epoll是最新的也是目前最好的多路复用技术。

    这里“多路”指的是多个网络连接，“复用”指的是复用同一个线程。采用多路 I/O 复用技术可以让单个线程高效的处理多个连接请求（尽量减少网络IO的时间消耗），且Redis在内存中操作数据的速度非常快（内存内的操作不会成为这里的性能瓶颈），主要以上两点造就了Redis具有很高的吞吐量。

    >非阻塞IO 内部实现采用epoll，采用了epoll+自己实现的简单的事件框架。epoll中的读、写、关闭、连接都转化成了事件，然后利用epoll的多路复用特性，绝不在io上浪费一点时间。

- 使用底层模型不同，它们之间底层实现方式以及与客户端之间通信的应用协议不一样，Redis直接自己构建了VM 机制 ，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求；




## Redis 数据类型

**包括字符串 String、列表 List、哈希 Hash、集合 Set、有序集合 Zset**。 Redis 后续又丰富了几种数据类型Bitmaps、HyperLogLogs、GEO。

- String 类型是二进制安全的，及其 String 类型可以包含任何数据，比如 JPG 图片或者序列化的对象。String 类型的值最大能存储 512M。常用命令：`get、set、incr、decr、mget` 等
- List 列表是字符串列表，按照插入顺序排序，**可以添加一个元素到列表的头部或尾部**。常用命令：`lpush`, `rpush`, `lpop`, `rpop`, `lrange` 等。
    应用场景：**关注列表、粉丝列表等**；消息队列；**最新消息排行**。
- Hash 是一个键值(字段-值)的集合。Hash 适合存储对象。常用命令：`hset`, `hget`, `hgetall` 等。
- Set 是 String 类型的无序集合。
    应用场景：Set 是自动去重的，课题判断某个成员是否在一个 set 集合中。**共同好友**；利用唯一性，**统计访问网站的所有 ip**。
- Zset 是 String 类型元素的集合，不允许出现重复的元素。==**将 set 中的元素增加一个权重参数 score，元素按 score 进行有序排列**。== **默认是从小到大排序，如果想从大到小排序：`zrevrange myzset3 0 –1 withscores`，返回名称为 myzset 的 zset 中的 index 从 start 到 end 元素的反转。如果 score 相同，默认安装字典序排序**。
    应用场景：排行榜
- **BitMap 就是通过一个 bit 位来表示某个元素对应的值或者状态**, 其中的 key 就是对应元素本身，**实际上底层也是通过对字符串的操作来实现**。Redis 从 2.2 版本之后新增了setbit, getbit, bitcount 等几个 bitmap 相关命令。虽然是新命令，但是本身都是对字符串的操作。
- Redis 的 GEO 特性在 Redis 3.2 版本中推出， 这个功能可以将用户给定的地理位置信息储存起来， 并对这些信息进行操作。

### zset 底层存储结构

- zset 底层的存储结构包括 `ziplist` 或 `skiplist`，在同时满足以下两个条件的时候使用 `ziplist`，其他时候使用 `skiplist`，两个条件如下：

    - 有序集合保存的元素数量小于 128 个
    - 有序集合保存的所有元素的长度小于 64 字节

- 当`ziplist` 作为 zset  的底层存储结构时候，**每个集合元素使用两个紧挨在一起的压缩列表节点来保存，第一个节点保存元素的成员值（value)，第二个元素保存元素的分值（score）。**

    格式如下图，紧挨着的是元素 memeber 和分值 score，整体数据是有序格式。

    ![zipList](../../../../../Yu%2520Writer%2520Libraries/Interview/InterviewExperience/Redis.resource/zipList.png)

- 当 `skiplist` 作为 zset 的底层存储结构的时候，==**使用 skiplist 按序保存元素及分值，使用 dict 来保存元素和分值的映射关系。**==

    `skiplist` 作为 zset 的存储结构，整体存储结构如下图，**核心点主要是包括一个 dict（字典） 对象和一个 skiplist （跳跃表）对象**。dict 保存 key/value，key为元素（member），value为分值（score）；skiplist 保存的有序的元素列表，每个元素包括元素和分值。两种数据结构下的元素指向相同的位置即通过指针共享相同元素的 member和score**。

![image-20200406221829133](../../../../../Yu%2520Writer%2520Libraries/Interview/InterviewExperience/Redis.resource/image-20200406221829133.png)



Redis底层的数据结构包括：**简单动态数组SDS、链表、字典、跳跃链表、整数集合、压缩列表、对象**。使用它们实现了五种数据类型。

Redis为了平衡空间和时间效率，针对value的具体数据类型在底层会采用不同的数据结构来实现（如图）：

![image-20200406221939106](../../../../../Yu%2520Writer%2520Libraries/Interview/InterviewExperience/Redis.resource/image-20200406221939106.png)

## 数据结构

### Redis 的字典如何实现，简述渐进式rehash过程

![image-20200502173821640](../../../../../Yu%2520Writer%2520Libraries/Interview/InterviewExperience/Redis.resource/image-20200502173821640.png)

**dictEntry 和 dictht 可以看成一个 HashMap 来理解（虽然不完全一样）。**

- 关于 dictEntry

dictEntry 是哈希表节点，也就是我们存储数据地方，其包括三个指针：`key` 保存着键值对中的键，`v` 保存着键值对中的值（值可以是一个指针或者是 `uint64_t` 或者是 `int64_t`）。`next` 是指向另一个哈希表节点的指针，这个指针可以将多个哈希值相同的键值对连接在一次，以此来解决哈希冲突的问题。

- 关于dictht

哈希表包括的成员有 `table`、`size`、`used`、`sizemask`。`table` 是一个数组，数组中的每个元素都是一个指向 `dictEntry` 结构的指针， 每个 `dictEntry` 结构保存着一个键值对；`size` 属性记录了哈希表 `table` 的大小，而 `used`属性则记录了哈希表目前已有节点的数量。`sizemask` 等于 `size-1`，和哈希值计算一个键在`table`数组的索引。

- 关于dict

dict 结构体就是字典的定义，包含的成员有 `type`，`privdata`、`ht`、`rehashidx`。其中 `dictType` 指针类型的 `type` 指向了操作字典的 api，理解为函数指针即可，`ht`是包含 2 个 `dictht` 的数组，也就是字典包含了 2 个哈希表，`rehashidx` 是进行 `rehash` 时使用的变量，`privdata` 配合`dictType` 指向的函数作为参数使用。

![image-20200502173957083](../../../../../Yu%2520Writer%2520Libraries/Interview/InterviewExperience/Redis.resource/image-20200502173957083.png)

哈希表保存的键值对数量是动态变化的，为了让哈希表的负载因子维持在一个合理的范围之内，就需要对哈希表进行扩缩容。有两种方式：

- 普通 Rehash 重新散列

    扩缩容是通过执行 rehash 重新散列来完成，对字典的哈希表执行普通 rehash 的基本步骤为**分配空间->逐个迁移->交换哈希表**，详细过程如下：

    - 为字典的 ht[1] 哈希表分配空间，分配的空间大小取决于要执行的操作以及 ht[0] 当前包含的键值对数量：
        - 扩展操作时 ht[1]  的大小为第一个大于等于 ht[0].used*2 的 2^n；**扩容为原来的两倍**。
        - 收缩操作时 ht[1] 的大小为第一个大于等于 ht[0].used 的 2^n ；
    - 将保存在ht[0]中的所有键值对重新计算键的哈希值和索引值rehash到ht[1]上；
    - 重复rehash直到ht[0]包含的所有键值对全部迁移到了ht[1]之后释放 ht[0]， 将ht[1]设置为 ht[0]，并在ht[1]新创建一个空白哈希表，为下一次rehash做准备。

- 渐进式rehash过程

    Redis的rehash动作并不是一次性完成的，而是分多次、渐进式地完成的，原因在于当哈希表里保存的键值对数量很大时， 一次性将这些键值对全部rehash到ht[1]可能会导致服务器在一段时间内停止服务，这个是无法接受的。**针对这种情况Redis采用了渐进式rehash，过程的详细步骤**：

    - **为ht[1]分配空间，这个过程和普通Rehash没有区别**；
    - **将rehashidx设置为0，表示rehash工作正式开始，同时这个rehashidx是递增的，从0开始表示从数组第一个元素开始rehash**。
    - **在rehash进行期间，每次对字典执行增删改查操作时，顺带将ht[0]哈希表在rehashidx索引上的键值对rehash到 ht[1]，完成后将rehashidx加1，指向下一个需要rehash的键值对**。

    - **随着字典操作的不断执行，最终ht[0]的所有键值对都会被rehash至ht[1]，再将rehashidx属性的值设为-1来表示 rehash操作已完成**。

    **渐进式 rehash的思想在于将rehash键值对所需的计算工作分散到对字典的每个添加、删除、查找和更新操作上，从而避免了集中式rehash而带来的阻塞问题。**



### 跳跃表

是有序集合的底层实现之一。

跳跃表是基于多指针有序链表实现的，可以看成多个有序链表。

![image-20200401174300330](../../../../../Yu%2520Writer%2520Libraries/Interview/InterviewExperience/Redis.resource/image-20200401174300330.png)

在查找时，从上层指针开始查找，找到对应的区间之后再到下一层去查找。下图演示了查找 22 的过程。

![image-20200401174320549](../../../../../Yu%2520Writer%2520Libraries/Interview/InterviewExperience/Redis.resource/image-20200401174320549.png)

与红黑树等平衡树相比，跳跃表具有以下优点：

- 插入速度非常快速，因为不需要进行旋转等操作来维护平衡性；
- 更容易实现；
- 支持无锁操作。



### skiplist 的源码格式

 zset包括dict和zskiplist两个数据结构，其中dict的保存key/value，便于通过key(元素)获取score(分值)。zskiplist保存有序的元素列表，便于执行range之类的命令。

```cpp
/*
 * 有序集合
 */
typedef struct zset {

    // 字典，键为成员，值为分值
    // 用于支持 O(1) 复杂度的按成员取分值操作
    dict *dict;

    // 跳跃表，按分值排序成员
    // 用于支持平均复杂度为 O(log N) 的按分值定位成员操作
    // 以及范围操作
    zskiplist *zsl;

} zset;
```

 zskiplist作为skiplist的数据结构，包括指向头尾的header和tail指针，其中level保存的是skiplist的最大的层数。

```cpp
/*
 * 跳跃表
 */
typedef struct zskiplist {

    // 表头节点和表尾节点
    struct zskiplistNode *header, *tail;

    // 表中节点的数量
    unsigned long length;

    // 表中层数最大的节点的层数
    int level;

} zskiplist;
```

 skiplist跳跃列表中每个节点的数据格式，每个节点有保存数据的robj指针，分值score字段，后退指针backward便于回溯，zskiplistLevel的数组保存跳跃列表每层的指针。



```cpp
/*
 * 跳跃表节点
 */
typedef struct zskiplistNode {

    // 成员对象
    robj *obj;

    // 分值
    double score;

    // 后退指针
    struct zskiplistNode *backward;

    // 层
    struct zskiplistLevel {

        // 前进指针
        struct zskiplistNode *forward;

        // 跨度
        unsigned int span;

    } level[];

} zskiplistNode;
```

### redis 存储结构

- redis的存储结构从外层往内层依次是redisDb、dict、dictht、dictEntry。
- redis的Db默认情况下有15个，每个redisDb内部包含一个dict的数据结构。
- redis的dict内部包含dictht的数组，数组个数为2，主要用于hash扩容使用。
- dictht内部包含dictEntry的数组，可以理解就是hash的桶，然后如果冲突通过挂链法解决。

![img](https://upload-images.jianshu.io/upload_images/6302559-b314eba4298b8ded.png?imageMogr2/auto-orient/strip|imageView2/2/w/897/format/webp)

## ==zset 存储过程==

 zset 的添加过程我们以 zadd 的操作作为例子进行分析，整个过程如下：

- 解析参数得到每个元素及其对应的分值
- 查找 key 对应的 zset 是否存在不存在则创建
- 如果存储格式是 ziplist，那么在执行添加的过程中我们需要区分元素存在和不存在两种情况，**存在情况下先删除后添加**；不存在情况下则添加并且需要考虑元素的长度是否超出限制或实际已有的元素个数是否超过最大限制进而决定是否转为 skiplist 对象。
- 如果存储格式是 skiplist，那么在执行添加的过程中我们需要区分元素存在和不存在两种情况，存在的情况下先删除后添加，不存在情况下那么就直接添加，在 skiplist 当中添加完以后我们同时需要更新 dict 的对象。

```php
void zaddGenericCommand(redisClient *c, int incr) {

    static char *nanerr = "resulting score is not a number (NaN)";

    robj *key = c->argv[1];
    robj *ele;
    robj *zobj;
    robj *curobj;
    double score = 0, *scores = NULL, curscore = 0.0;
    int j, elements = (c->argc-2)/2;
    int added = 0, updated = 0;

    // 输入的 score - member 参数必须是成对出现的
    if (c->argc % 2) {
        addReply(c,shared.syntaxerr);
        return;
    }

    // 取出所有输入的 score 分值
    scores = zmalloc(sizeof(double)*elements);
    for (j = 0; j < elements; j++) {
        if (getDoubleFromObjectOrReply(c,c->argv[2+j*2],&scores[j],NULL)
            != REDIS_OK) goto cleanup;
    }

    // 取出有序集合对象
    zobj = lookupKeyWrite(c->db,key);
    if (zobj == NULL) {
        // 有序集合不存在，创建新有序集合
        if (server.zset_max_ziplist_entries == 0 ||
            server.zset_max_ziplist_value < sdslen(c->argv[3]->ptr))
        {
            zobj = createZsetObject();
        } else {
            zobj = createZsetZiplistObject();
        }
        // 关联对象到数据库
        dbAdd(c->db,key,zobj);
    } else {
        // 对象存在，检查类型
        if (zobj->type != REDIS_ZSET) {
            addReply(c,shared.wrongtypeerr);
            goto cleanup;
        }
    }

    // 处理所有元素
    for (j = 0; j < elements; j++) {
        score = scores[j];

        // 有序集合为 ziplist 编码
        if (zobj->encoding == REDIS_ENCODING_ZIPLIST) {
            unsigned char *eptr;

            // 查找成员
            ele = c->argv[3+j*2];
            if ((eptr = zzlFind(zobj->ptr,ele,&curscore)) != NULL) {

                // 成员已存在

                // ZINCRYBY 命令时使用
                if (incr) {
                    score += curscore;
                    if (isnan(score)) {
                        addReplyError(c,nanerr);
                        goto cleanup;
                    }
                }

                // 执行 ZINCRYBY 命令时，
                // 或者用户通过 ZADD 修改成员的分值时执行
                if (score != curscore) {
                    // 删除已有元素
                    zobj->ptr = zzlDelete(zobj->ptr,eptr);
                    // 重新插入元素
                    zobj->ptr = zzlInsert(zobj->ptr,ele,score);
                    // 计数器
                    server.dirty++;
                    updated++;
                }
            } else {
                // 元素不存在，直接添加
                zobj->ptr = zzlInsert(zobj->ptr,ele,score);

                // 查看元素的数量，
                // 看是否需要将 ZIPLIST 编码转换为有序集合
                if (zzlLength(zobj->ptr) > server.zset_max_ziplist_entries)
                    zsetConvert(zobj,REDIS_ENCODING_SKIPLIST);

                // 查看新添加元素的长度
                // 看是否需要将 ZIPLIST 编码转换为有序集合
                if (sdslen(ele->ptr) > server.zset_max_ziplist_value)
                    zsetConvert(zobj,REDIS_ENCODING_SKIPLIST);

                server.dirty++;
                added++;
            }

        // 有序集合为 SKIPLIST 编码
        } else if (zobj->encoding == REDIS_ENCODING_SKIPLIST) {
            zset *zs = zobj->ptr;
            zskiplistNode *znode;
            dictEntry *de;

            // 编码对象
            ele = c->argv[3+j*2] = tryObjectEncoding(c->argv[3+j*2]);

            // 查看成员是否存在
            de = dictFind(zs->dict,ele);
            if (de != NULL) {

                // 成员存在

                // 取出成员
                curobj = dictGetKey(de);
                // 取出分值
                curscore = *(double*)dictGetVal(de);

                // ZINCRYBY 时执行
                if (incr) {
                    score += curscore;
                    if (isnan(score)) {
                        addReplyError(c,nanerr);

                        goto cleanup;
                    }
                }

                // 执行 ZINCRYBY 命令时，
                // 或者用户通过 ZADD 修改成员的分值时执行
                if (score != curscore) {
                    // 删除原有元素
                    redisAssertWithInfo(c,curobj,zslDelete(zs->zsl,curscore,curobj));

                    // 重新插入元素
                    znode = zslInsert(zs->zsl,score,curobj);
                    incrRefCount(curobj); /* Re-inserted in skiplist. */

                    // 更新字典的分值指针
                    dictGetVal(de) = &znode->score; /* Update score ptr. */

                    server.dirty++;
                    updated++;
                }
            } else {

                // 元素不存在，直接添加到跳跃表
                znode = zslInsert(zs->zsl,score,ele);
                incrRefCount(ele); /* Inserted in skiplist. */

                // 将元素关联到字典
                redisAssertWithInfo(c,NULL,dictAdd(zs->dict,ele,&znode->score) == DICT_OK);
                incrRefCount(ele); /* Added to dictionary. */

                server.dirty++;
                added++;
            }
        } else {
            redisPanic("Unknown sorted set encoding");
        }
    }

    if (incr) /* ZINCRBY */
        addReplyDouble(c,score);
    else /* ZADD */
        addReplyLongLong(c,added);

cleanup:
    zfree(scores);
    if (added || updated) {
        signalModifiedKey(c->db,key);
        notifyKeyspaceEvent(REDIS_NOTIFY_ZSET,
            incr ? "zincr" : "zadd", key, c->db->id);
    }
}
```

## Redis 与 Memcached 对比

两者都是非关系型内存键值数据库，主要有以下不同：

- 数据类型

    **Memcached 仅支持字符串和二进制类型**，而 Redis 支持五种不同的数据类型，可以更灵活地解决问题。

- 数据持久化

    Redis 支持两种持久化策略，可以将内存的数据保存到磁盘中，重启的时候再次加载使用：RDB 快照和 AOF 日志，**而 Memcached 不支持持久化**。

- 分布式

    Memcached 不支持分布式，只能通过在客户端使用一致性哈希来实现分布式存储，这种方式在存储和查询时都需要先在客户端计算一次数据所在的节点。

    Redis Cluster 实现了分布式的支持。

- **Memcached 是多线程，非阻塞IO复用的网络模型；Redis 使用单线程的多路 IO 复用模型**。

- 内存管理机制

    - 在 Redis 中，并不是所有数据都一直存储在内存中，可以将一些很久没用的 value 交换到磁盘，而 Memcached 的数据则会一直在内存中。
    - **Memcached 将内存分割成特定长度的块来存储数据，以完全解决内存碎片的问题。但是这种方式会使得内存的利用率不高**，例如块的大小为 128 bytes，只存储 100 bytes 的数据，那么剩下的 28 bytes 就浪费掉了。

## Redis 持久化

Redis 是内存型数据库，为了解决数据在断电后以及因进程退出造成数据丢失问题，需要将内存中的数据持久化到硬盘上，当下次重启时利用之前持久化文件即可实现数据恢复。

#### （一）RDB（Redis Database） 持久化（默认）

==**RDB 持久化是把当前进程数据生成快照保存到硬盘的过程**，触发 RDB 持久化过程分为手动触发和自动触发。==

- 触发机制

    - ==手动触发分别对应 save 和 bgsave 命令==：

        - **save 命令：阻塞当前 Redis 服务器，直到 RDB 过程完成为止**，对于内存比较大的实例会造成长时间阻塞，线上环境不建议使用。运行 save 命令对应 Redis 日志如下：
            `DB saved on disk`

        - bgsave 命令：**bgsave 命令是针对 save 阻塞问题做的优化（save 相对废弃了）。Redis 进程执行 fork 操作创建子进程，RDB 持久化过程由子进程负责，完成后自动结束。阻塞只发生在 fork 阶段，因此时间很短。**运行 bgsave 名字对应的 Redis 日志如下：

            ```java
            Background saving started by pid 3152
            DB saved on disk
            RDB: 0MB of memory userd by copy-on-write
            Background saving terminated with success
            ```

            ==**bgsave 流程说明：**==

            - 执行 bgsave 命令，**Redis 父进程判断当前是否存在正在执行的子进程**，如只 RDB/AOF 子进程，如果存在 bgsave 命令直接返回。
            - 父进程执行 fork 操作创建子进程，fork 操作过程中父进程会阻塞，通过 `info stats` 命令查看`latest_fork_usec` 选项，可以获取最近一个 `fork` 操作的耗时，单位为微秒。
            - 父进程 fork 完成后，bgsave 命令返回 `“Background saving started”`信息并不再阻塞父进程，可以继续响应其他命令。
            - 子进程创建 RDB 文件，**根据父进程内存生成临时快照文件，完成后对原有文件进行原子替换**。执行 `lastsave` 命令可以获取最后一次生成 RDB 的时间，对应 info 统计的 `rdb_last_save_time` 选项。
            - 进程发送信号给父进程表示完成，父进程更新统计信息，具体见 `info Persistence` 下的 `rdb_*` 相关选项。

    - Redis 内部还存在自动触发 RDB 的持久化机制，例如以下场景:

        - 使用 save 相关配置，如 `save m n` 表示 m 秒之内数据集存在 n 次修改时，自动触发 bgsave。
        - 如果从节点执行全量复制操作，主节点自动执行bgsave生成RDB文件并发送给从节点。
        - 执行 `debug reload` 命令重新加载 Redis 时，也会自动触发 save 操作。

    - 默认情况下执行 `shutdown` 命令时，如果没有开启 AOF 持久化功能则自动执行 bgsave。

- RDB 文件处理

    - 保存：**RDB 文件保存在 dir 配置指定的目录下**，文件名通过 `dbfilename` 配置指定。可以通过执行 `config set dir {newDir}`  和 `config set dbfilename {newFileName}` 运行期动态执行，当下次运行时 RDB 文件会保存到新目录。

    - ==压缩：Redis 默认采用 LZF 算法对生成的 RDB 文件做压缩处理，压缩后的文件远远小于内存大小==，默认开启，可以通过参数 `config set rdbcompression {yes|no}` 动态修改。

    - 校验：如果 Redis 加载损坏的 RDB 文件时拒绝启动，并打印如下日志：

        `Short read or OOM loading DB. Unrecoverable error , aborting now.`这时可以使用Redis提供的redis-check-dump工具检测RDB文件并获取对应的错误报告

- RDB 的优缺点

    - RDB的优点:

        - ==RDB 是一个紧凑压缩的二进制文件，代表  Redis  在某一个时间点上的数据快照==。非常适合用于备份，全量复制等场景。比如每6小时执行 bgsave 备份，并把 RDB 文件拷贝到远程机器或者文件系统中（如 hdfs），用于灾难恢复。
        - ==Redis 加载 RDB 恢复数据远远快于 AOF 方式==。

    - RDB 的缺点

        - ==RDB 方式数据没办法做到实时持久化/秒级持久化。因为 bgsave 每次运行都要执行 fork 操作创建子进程，属于重量级操作，频繁执行成本过高。==
        - RDB文件使用特定二进制格式保存，Redis版本演进过程中有多个格式的RDB 版本，存在老版本Redis服务无法兼容新版RDB格式的问题。

**针对 RDB 不适合实时持久化的问题，Redis 提供了 AOF 持久化方式来解决**

#### （二）AOF 持久化

**AOF(append only file) 持久化：以独立日志的方式记录每次写命令，重启时再重新执行 AOF 文件中命令达到恢复数据的目的。AOF 的主要作用是解决了数据持久化的实时性，目前已经是 Redis 持久化的主流方式。**

- 使用 AOF

    开启 AOF 功能需要设置配置：`appendonly yes` 默认不开启。AOF 文件通过 `append filename` 配置设置，默认文件名是 `appendonly.aof`。保存路径同 RDB 持久化方式一致。通过 dir 配置指定。

- AOF 的工作流程操作：**命令写入（append）、文件同步（sync）、文件重写（rewrite）、重启加载（load）**。工作流程如下：

    - ==**所有的写入命令会追加到 `aof_buf`（缓冲区）中。**==

        AOF 命令写入的内容直接是文本协议格式。例如 `set hello world` 这条命令，在 AOF 缓冲区会追加如下文本：`\r\n$3\r\nset\r\n$5\r\nhello\r\n$5\r\nworld\r\n`。

        - 为什么使用文本协议

            文本协议具有很好的兼容性。
            开启AOF后，所有写入命令都包含追加操作，直接采用协议格式，避免二次处理开销。
            文本协议具有可读性，方便直接修改和处理。

        - AOF为什么把命令追加到 `aof_buf` 中

            **Redis 使用单线程响应命令，如果每次写 AOF 文件命令都直接追加到硬盘，那么性能完全取决于当前硬盘负载。写入缓冲区 `aof_buf` 中，还有另一个好处，Redis 可以提供多种缓冲区同步硬盘的策略，在性能和安全性方面做出平衡。**

    - ==**AOF 缓冲区根据对应的策略向硬盘做同步操作。**==

        Redis 提供了多种 AOF 缓冲区同步文件策略，由参数 `appendfsync` 控制，不同值的含义如表所示

        - 配置为 `no`,由于操作系统每次同步 AOF 文件的周期不可控，而且会极大每次同步硬盘的数据量，虽然提升了性能，但数据安全性无法保证。

        - `write` 操作会触发延迟写（delayed write）机制，Linux在内核提供页缓冲区用来提高硬盘IO性能。write操作在写入系统缓冲区后直接返回。同步硬盘操作依赖于系统调度机制，列如：缓冲区页空间写满或达到特定时间周期。同步文件之前，如果此时系统故障宕机，缓冲区内数据将丢失。
        - **`fsync` 针对单个文件操作（比如AOF文件），做强制硬盘同步**，`fsync` 将阻塞知道写入硬盘完成后返回，保证了数据持久化。
        - 配置为 `always` 时，每次写入都要同步 AOF 文件，在一般的 STAT 硬盘上，Redis 只能支持大约几百TPS 写入，显然跟Redis高性能特性背道而驰，不建议配置。
        - 配置为 `everysec` ,是建议的同步策略，也是默认配置，做到兼顾性能和数据安全性，理论上只有在系统突然宕机的情况下丢失 1s 的数据。（严格来说最多丢失1s数据是不准确）

    - ==**随着 AOF 文件越来越大，需要定期对 AOF 文件进行重写，达到压缩的目的。**==

        **AOF 文件重写就是把 Redis 进程内的数据转化为写命令同步到新 AOF 文件的过程**。所以重写后的AOF文件为什么可以变小，原因如下

        - **进程内已经超时的数据不再写文件**。

        - **旧的 AOF 文件含有无效命令**，如del key1、 hdel key2、srem keys、set a 111、set a 222等。重写使用进程内数据直接生成，这样新的AOF文件只保留最终数据的写入命令。

        - **多条写命令可以合并为一个**，如lpush list a、lpush list b、 lpush list c 可以转化为：lpush list a b c。为了防止但挑明了过大造成客户端缓冲区溢出，对于list、set、hash、zset等类型曹组，以64个元素为界拆分为多条。

    > AOF重写过程可以手动触发和自动触发：
    >
    > - 手动触发：直接调用 bgrewriteaof 命令
    > - 自动触发：根据 `auto-aof-rewrite-min-size` 和 `auto-aof-rewrite-percentage` 参数确定自动触发时机
    >     - `auto-aof-rewrite-min-size`：表示运行 AOF 重写时文件最小体积，默认为 64MB
    >     - auto-aof-rewrite-percentage:代表当前AOF文件空间（aof_current_size）和上一次重写后AOF文件空间（aof_base_size）的值
    >
    >  **当触发AOF重写时的运行流程：**
    >
    > - 执行AOF重写请求。
    >     如果当前进程正在执行AOF重写，请求不执行并返回如下响应:`ERR Background append only file rewriting already in progress`。如果当前进程正在执行bgsave操作，重写命令延迟到bgsave完成后再执行，返回如下响应：`Background append only file rewriting scheduled`。
    >  - 父进程执行fork创建子进程，开销等同于bgsave过程。
    > - 主进程fork操作完成后，继续响应其他命令。所有修改命令依然写入AOF缓冲区并更具appendfsync策略同步到硬盘，保证原有AOF机制正确性。
    > - 由于fork操作运用写时复制技术，子进程只能共享fork操作时的内存数据。由于父进程依然响应命令，Redis使用"AOF重写缓冲区"保存这部分新数据，防止新AOF文件生成期间丢失这部分数据。
    >     - 子进程根据内存快照，按照命令合并规则写入到新的AOF文件。每次批量写入硬盘数据量由配置aof-rewrite-incremental-fsync控制，默认为32MB，防止单次刷盘数据过多造成硬盘阻塞。
    > - 新AOF文件写入完成后，子进程发送信号给父进程，父进程更新统计信息，具体见info persistence下的aof_*相关统计。
    > - 父进程把AOF重写缓冲区的数据写入到新的AOF文件。
    > - 使用新AOF文件替换老文件，完成AOF重写。

- ==**当 Redis 服务重启时，可以加载 AOF 文件进行数据恢复**==。

    AOF和RDB文件都可以用于服务器重启时的数据恢复。Redis持久化文件加载流程：

    -  AOF持久化开启且存在AOF文件时，优先加载AOF文件，打印如下日志：`DB loaded from append only file: 5.841 seconds`

    -  AOF关闭或者AOF文件不存在时，加载RDB文件，打印如下日志：
    -  加载AOF/RDB文件完成后，Redis启动成功。
    -  AOF/RDB文件存在错误时，Redis启动失败并打印错误信息

**RDB 以及 AOF 优缺点**

- RDB

- 优点：

    - 只有一个文件dump.rdb，方便持久化；
    - **fork一个子进程来完成写操作，将数据写到磁盘上一个临时RDB文件中，主进程可以继续处理命令，保证了redis的高性能**；当子进程完成写临时文件后，将原来的rdb文件替换掉，**这样的好处是可以copy-on-write**。
    - 数据集较大时，比AOF的启动效率更高。

    缺点：

    - 数据不安全，RDB持久化是周期性的保存数据，如果在未触发下一次存储时服务宕机，就会丢失增量数据。

- AOF(Append Only File)：**将redis执行的每条写命令追加到单独的日志文件appendonly.aof中，可以做到全程持久化**.当开启AOF后，服务端每执行一次写操作就会把该条命令追加到一个单独的AOF缓冲区的末尾，然后把AOF缓冲区的内容写入AOF文件里，但是AOF存储的是指令序列，恢复时要花费很长时间且文件更大。

    优点：

    - 数据安全，aof持久化每进行一此命令操作就记录到aof文件一次。
    - 通过append模式写文件，即使中途服务器宕机，也可以通过redis-check-aof工具解决数据一致性问题；
    - rewrite模式，aof重写可以把内存中的数据，逆化成命令，写入到aof日志中，解决aof日志过大的问题。

    >appendfsync always   #每1个命令都立即同步到aof
    >appendfsync everysec   #每秒写一次

    缺点：

    - AOF存储的是指令序列，恢复时要花费很长时间且文件更大



## 九、事务

一个事务包含了多个命令，服务器在执行事务期间，不会改去执行其它客户端的命令请求。

==事务中的多个命令被一次性发送给服务器，而不是一条一条发送，这种方式被称为流水线，它可以减少客户端与服务器之间的网络通信次数从而提升性能。==

Redis 最简单的事务实现方式是使用 MULTI 和 EXEC 命令将事务操作包围起来。

### 相关命令

| 命令        | 格式                    | 作用                                                         | 返回结果                                                     |
| ----------- | ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **WATCH**   | **WATCH key [key ...]** | 将给出的`Keys`标记为`监测态`，作为事务执行的条件             | always OK.                                                   |
| **UNWATCH** | **UNWATCH**             | 清除事务中`Keys`的 `监测态`，如果调用了**EXEC** or **DISCARD**，则没有必要再手动调用**UNWATCH** | always OK.                                                   |
| **MULTI**   | **MULTI**               | `显式`开启`redis事务`，后续`commands`将排队，等候使用**EXEC**进行原子执行 | always OK.                                                   |
| **EXEC**    | **EXEC**                | 执行事务中的`commands`队列，恢复连接状态。如果**WATCH**在之前被调用，只有`监测`中的`Keys`没有被修改，命令才会被执行，否则停止执行（详见下文，`CAS机制`） | **成功：** 返回数组 —— 每个元素对应着原子事务中一个 `command`的返回结果; **失败：** 返回`NULL`（`Ruby` 返回``nil``）; |
| **DISCARD** | **DISCARD**             | 清除事务中的`commands`队列，恢复连接状态。如果**WATCH**在之前被调用，`释放` `监测`中的`Keys` | always OK.                                                   |

**注意：**

- `MULTI`,`EXEC`,`DISCARD`才是`显式`开启并控制事务的常用命令，可类比`关系型数据库`中的  `BEGAIN`,`COMMIT`,`ROLLBACK`（事实上，差距很大）；

- `WATCH`命令的使用是为了解决 `事务并发` 产生的`不可重复读`和`幻读`的问题（简单理解为给`Key加锁`）；

------

### Redis 事务

MULTI, EXEC, DISCARD and WATCH 是Redis事务的基础。`用来显式开启并控制一个事务，它们允许在一个步骤中执行一组命令`。并提供两个重要的保证：

- **事务中的所有命令都会被序列化并按顺序执行**。在执行Redis事务的过程中，不会出现由另一个客户端发出的请求。这保证 `命令队列` 作为一个单独的原子操作被执行。
- **队列中的命令要么全部被处理，要么全部被忽略**。EXEC命令触发事务中所有命令的执行，因此，当客户端在事务上下文中失去与服务器的连接，
- 如果发生在调用MULTI命令之前，则不执行任何`commands`；
- 如果在此之前EXEC命令被调用，则所有的`commands`都被执行。

同时，redis使用AOF([append-only file](https://redis.io/topics/persistence#append-only-file))，使用一个额外的`write操作`将事务写入磁盘。如果发生宕机，进程奔溃等情况，可以使用redis-check-aof tool 修复append-only file，使服务正常启动，并恢复部分操作。

------

### 用法

使用`MULTI`命令`显式开启`Redis事务。 该命令总是以OK回应。`此时用户可以发出多个命令，Redis不会执行这些命令，而是将它们排队`。`EXEC`被调用后，所有的命令都会被执行。而调用`DISCARD`可以`清除`事务中的`commands队列`并`退出事务`。

- 以下示例以原子方式，递增键foo和bar。

```
>MULTI
OK
>INCR foo
QUEUED
>INCR bar
QUEUED
>EXEC
1）（整数）1
2）（整数）1
复制代码
```

从上面的命令执行中可以看出，`EXEC`返回一个`数组`，`其中每个元素都是事务中单个命令的返回结果，而且顺序与命令的发出顺序相同`。 当Redis连接处于`MULTI`请求的上下文中时，所有命令将以`字符串QUEUED`（**从Redis协议的角度作为状态回复发送**）作为回复，并在`命令队列`中排队。只有EXEC被调用时，排队的命令才会被执行，此时才会有`真正的返回结果`。

------

## 事务中的错误

**事务期间，可能会遇到两种命令错误：**

- **在调用`EXEC`命令之前出现错误（`COMMAND`排队失败）。**
- 例如，命令可能存在`语法错误`（参数数量错误，错误的命令名称...）；
- 或者可能存在`某些关键条件`，如内存不足的情况（如果服务器使用`maxmemory`指令做了`内存限制`）。

*客户端会在`EXEC`调用之前检测第一种错误*。 通过检查排队命令的`状态回复`（***注意：这里是指`排队`的`状态回复`，而不是`执行结果`***），如果命令使用`QUEUED`进行响应，则它已正确排队，否则Redis将返回错误。如果排队命令时发生错误，大多数客户端将`中止该事务并清除命令队列`。然而：

- 在`Redis 2.6.5之前`，这种情况下，在`EXEC`命令调用后，客户端会执行命令的子集（成功排队的命令）而忽略之前的错误。
- 从`Redis 2.6.5开始`，服务端会记住在累积命令期间发生的错误，当`EXEC`命令调用时，`将拒绝执行事务，并返回这些错误，同时自动清除命令队列`。
- 示例如下：

```
>MULTI
+OK
>INCR a b c
-ERR wrong number of arguments for 'incr' command
复制代码
```

> 这是由于`INCR`命令的语法错误，将在调用`EXEC`之前被检测出来，并终止事务（version2.6.5+）。

- **在调用`EXEC`命令之后出现错误。**
- 例如，使用`错误的值`对某个`key`执行操作（如针对`String`值调用`List`操作）

*`EXEC`命令执行之后发生的错误并不会被特殊对待*：`即使事务中的某些命令执行失败，其他命令仍会被正常执行`。

- 示例如下：

```
>MULTI
+OK
>SET a 3
+QUEUED
>LPOP a
+QUEUED
>EXEC
*2
+OK
-ERR Operation against a key holding the wrong kind of value
复制代码
```

> - `EXEC`返回一个包含两个元素的字符串数组，一个元素是`OK`，另一个是`-ERR……`。
> - **能否将错误合理的反馈给用户这取决于`客户端library`(如：`Spring-data-redis.redisTemplate`)的自身实现。**
> - 需要注意的是，**即使命令失败，队列中的所有其他命令也会被处理----Redis不会停止命令的处理**。

------

### Redis事务不支持Rollback（`重点`）

**事实上 Redis 命令在事务执行时可能会失败，但仍会继续执行剩余命令而不是`Rollback`（事务回滚）**。如果你使用过`关系数据库`，这种情况可能会让你感到很奇怪。然而针对这种情况具备很好的解释：

- Redis命令可能会执行失败，仅仅是由于错误的语法被调用（命令排队时检测不出来的错误），或者使用错误的数据类型操作某个`Key`： 这意味着，实际上失败的命令都是编程错误造成的，都是开发中能够被检测出来的，生产环境中不应该存在。（这番话，彻底甩锅，“都是你们自己编程错误，与我们无关”。）
- 由于不必支持`Rollback`,`Redis`内部简洁并且更加高效。

“**如果错误就是发生了呢？**”这是一个`反对Redis`观点的`争论`。然而应该指出的是，通常情况下，回滚并不能挽救编程错误。**鉴于没有人能够挽救程序员的错误**，并且`Redis命令`失败所需的错误类型不太可能进入生产环境，所以我们选择了不支持错误回滚（Rollback）这种更简单快捷的方法。

------

### 清除命令队列

`DISCARD`被用来中止事务。事务中的所有命令将不会被执行,连接将恢复正常状态。

```
> SET foo 1
OK
> MULTI
OK
> INCR foo
QUEUED
> DISCARD
OK
> GET foo
"1"
复制代码
```

## 事件

Redis 服务器是一个事件驱动程序。

### 文件事件

服务器通过套接字与客户端或者其它服务器进行通信，文件事件就是对套接字操作的抽象。

Redis 基于 Reactor 模式开发了自己的网络事件处理器，使用 I/O 多路复用程序来同时监听多个套接字，并将到达的事件传送给文件事件分派器，分派器会根据套接字产生的事件类型调用相应的事件处理器。

![image-20200401181022525](../../../../../Yu%2520Writer%2520Libraries/Interview/InterviewExperience/Redis.resource/image-20200401181022525.png)

### 时间事件

服务器有一些操作需要在给定的时间点执行，时间事件是对这类定时操作的抽象。

时间事件又分为：

- 定时事件：是让一段程序在指定的时间之内执行一次；
- 周期性事件：是让一段程序每隔指定时间就执行一次。

Redis 将所有时间事件都放在一个无序链表中，通过遍历整个链表查找出已到达的时间事件，并调用相应的事件处理器。

#### 事件的调度与执行

服务器需要不断监听文件事件的套接字才能得到待处理的文件事件，但是不能一直监听，否则时间事件无法在规定的时间内执行，因此监听时间应该根据距离现在最近的时间事件来决定。

事件调度与执行由 aeProcessEvents 函数负责，伪代码如下：

```python
def aeProcessEvents():
    # 获取到达时间离当前时间最接近的时间事件
    time_event = aeSearchNearestTimer()
    # 计算最接近的时间事件距离到达还有多少毫秒
    remaind_ms = time_event.when - unix_ts_now()
    # 如果事件已到达，那么 remaind_ms 的值可能为负数，将它设为 0
    if remaind_ms < 0:
        remaind_ms = 0
    # 根据 remaind_ms 的值，创建 timeval
    timeval = create_timeval_with_ms(remaind_ms)
    # 阻塞并等待文件事件产生，最大阻塞时间由传入的 timeval 决定
    aeApiPoll(timeval)
    # 处理所有已产生的文件事件
    procesFileEvents()
    # 处理所有已到达的时间事件
    processTimeEvents()
```

将 aeProcessEvents 函数置于一个循环里面，加上初始化和清理函数，就构成了 Redis 服务器的主函数，伪代码如下：

```python
def main():
    # 初始化服务器
    init_server()
    # 一直处理事件，直到服务器关闭为止
    while server_is_not_shutdown():
        aeProcessEvents()
    # 服务器关闭，执行清理操作
    clean_server()
```

从事件处理的角度来看，服务器运行流程如下：

<img src="../../../../../Yu%2520Writer%2520Libraries/Interview/InterviewExperience/Redis.resource/image-20200401181105659.png" alt="image-20200401181105659" style="zoom: 50%;" />

### 十四、一个简单的论坛系统分析

该论坛系统功能如下：

- 可以发布文章；
- 可以对文章进行点赞；
- 在首页可以按文章的发布时间或者文章的点赞数进行排序显示。

#### 文章信息

文章包括标题、作者、赞数等信息，在关系型数据库中很容易构建一张表来存储这些信息，在 Redis 中可以使用 HASH 来存储每种信息以及其对应的值的映射。

Redis 没有关系型数据库中的表这一概念来将同种类型的数据存放在一起，而是使用命名空间的方式来实现这一功能。键名的前面部分存储命名空间，后面部分的内容存储 ID，通常使用 : 来进行分隔。例如下面的 HASH 的键名为 article:92617，其中 article 为命名空间，ID 为 92617。

<img src="../../../../../Yu%2520Writer%2520Libraries/Interview/InterviewExperience/Redis.resource/image-20200406222204197.png" alt="image-20200406222204197" style="zoom:50%;" />

#### 点赞功能

当有用户为一篇文章点赞时，除了要对该文章的 votes 字段进行加 1 操作，还必须记录该用户已经对该文章进行了点赞，防止用户点赞次数超过 1。可以建立文章的已投票用户集合来进行记录。

为了节约内存，规定一篇文章发布满一周之后，就不能再对它进行投票，而文章的已投票集合也会被删除，可以为文章的已投票集合设置一个一周的过期时间就能实现这个规定。

<img src="../../../../../Yu%2520Writer%2520Libraries/Interview/InterviewExperience/Redis.resource/image-20200502203607696.png" alt="image-20200502203607696" style="zoom:50%;" />

#### 对文章进行排序

为了按发布时间和点赞数进行排序，可以建立一个文章发布时间的有序集合和一个文章点赞数的有序集合。（下图中的 score 就是这里所说的点赞数；下面所示的有序集合分值并不直接是时间和点赞数，而是根据时间和点赞数间接计算出来的）



### ==缓存雪崩==

#### （一）概念

如果缓存数据**设置的过期时间是相同**的，并且 Redis 恰好将这部分数据全部删光了。这就会导致在这段时间内，这些缓存**同时失效**，全部请求到数据库中。

#### （二）如何解决缓存雪崩

对于“对缓存数据设置相同的过期时间，导致某段时间内缓存失效，请求全部走数据库。”这种情况，非常好解决：

- 解决方法：在缓存的时候给过期时间加上一个**随机值**，这样就会大幅度的**减少缓存在同一时间过期**。

对于“Redis挂掉了，请求全部走数据库”这种情况，我们可以有以下的思路：

- 事发前：实现 Redis 的**高可用**(主从架构+ Sentinel 或者 Redis Cluster )，尽量避免 Redis 挂掉这种情况发生。

- 事发中：万一 Redis 真的挂了，我们可以设置**本地缓存(ehcache)+限流(hystrix)**，尽量避免我们的数据库被干掉(起码能保证我们的服务还是能正常工作的)

    https://www.jianshu.com/p/5a0669d6305e

- 事发后：redis 持久化，重启后自动从磁盘上加载数据，**快速恢复缓存数据**。

**其他**：

- 在缓存失效后，通过加锁或者队列来控制读数据库写缓存的线程数量。比如对某个key只允许一个线程查询数据和写缓存，其他线程等待。
- 做二级缓存，A1为原始缓存，A2为拷贝缓存，A1失效时，可以访问A2，A1缓存失效时间设置为短期，A2设置为长期
- 不同的key，设置不同的过期时间，让缓存失效的时间点尽量均匀。

### ==缓存穿透==

#### （一）缓存穿透概念

缓存穿透是指查询一个一定**不存在的数据**。由于缓存不命中，并且出于容错考虑，如果从**数据库查不到数据则不写入缓存**，这将导致这个不存在的数据**每次请求都要到数据库去查询**，失去了缓存的意义。

#### （二）如何解决缓存穿透

- 方案一：由于请求的参数是不合法的(每次都请求不存在的参数)，于是我们可以使用布隆过滤器(BloomFilter)（在查询的时候先去 BloomFilter 去查询 key 是否存在，如果不存在就直接返回，存在再走查缓存 -> 查 DB。）或者压缩 filter **提前拦截**，不合法就不让这个请求到数据库层！【针对这种key异常多、请求重复率比较低的数据】

- 方案二：当我们从数据库找不到的时候，我们也将这个空对象设置到缓存里边去【对于空数据的key有限的，重复率比较高的】

    下次再请求的时候，就可以从缓存里边获取了。这种情况我们一般会将空对象设置一个**较短的过期时间**。

### ==缓存击穿==

指缓存中的一个热点 key，不停地扛着大并发，在这个 key 失效的瞬间，持续的大并发就会击穿缓存，直接请求数据库。
**处理缓存击穿**：设置热点数据永不过期；或者使用互斥锁（上面的现象是多个线程同时去查询数据库的这条数据，那么我们可以在第一个查询数据的请求上使用一个 互斥锁来锁住它。其他的线程走到这一步拿不到锁就等着，等第一个线程查询到了数据，然后做缓存。后面的线程进来发现已经有缓存了，就直接走缓存。）。



## ==缓存与数据库双写一致==

==**从理论上来说，给缓存设置过期时间，是保证最终一致性的解决方案**==。这种方案下，我们可以对存入缓存的数据设置过期时间，所有的写操作以数据库为准，对缓存操作只是尽最大努力即可。也就是说如果数据库写成功，缓存更新失败，那么只要到达过期时间，则后面的读请求自然会从数据库中读取新值然后回填缓存。因此，接下来讨论的思路不依赖于给缓存设置过期时间这个方案。
在这里，我们讨论**三种**更新策略：

- 先更新数据库，再更新缓存
- 先删除缓存，再更新数据库
- 先更新数据库，再删除缓存

### 先更新数据库，再更新缓存（反对）

- **线程安全角度原因**
    同时有请求A和请求B进行更新操作，那么会出现

    - 线程 A 更新了数据库
    - 线程 B 更新了数据库
    - 线程 B 更新了缓存
    - 线程 A 更新了缓存

    > 这就出现请求A更新缓存应该比请求B更新缓存早才对，但是因为网络等原因，B却比A更早更新了缓存。这就导致了**脏数据**，因此不考虑。

- **业务场景角度原因：**
    （1）如果你是一个写数据库场景比较多，而读数据场景比较少的业务需求，采用这种方案就会导致，**数据压根还没读到，缓存就被频繁的更新，浪费性能。**
    （2）如果你写入数据库的值，并不是直接写入缓存的，而是要经过一系列复杂的计算再写入缓存。那么，每次写入数据库后，都再次计算写入缓存的值，无疑是浪费性能的。显然，删除缓存更为适合。

### 先删缓存，再更新数据库

该方案会导致不一致。例如同时有一个请求 A 进行更新操作，另一个请求 B 进行查询操作。那么会出现如下情形:

- 请求 A 进行写操作，删除缓存
- 请求 B 查询发现缓存不存在
- 请求 B 去数据库查询得到旧值
- 请求 B 将旧值写入缓存
- 请求 A 将新值写入数据库

而且如果不采用给缓存设置过期时间策略，该数据永远都是脏数据。**可以采用延时双删策略解决，具体步骤为：**

- 先淘汰缓存：`redis.delKey(key);`
- 再写数据库（这两步和原来一样）：`db.updateData(data);`
- 休眠一段时间（例如 1 秒），再次淘汰缓存：`Thread.sleep(1000);` 和 `redis.delKey(key);`

这么做，可以将 1 秒内所造成的缓存脏数据，再次删除。**具体休眠的时间根据自己项目读业务数据逻辑的耗时来设置，然后写数据的休眠时间则在读数据业务逻辑的耗时基础上，加几百 ms 即可。这么做的目的，就是确保读请求结束，写请求可以删除读请求造成的缓存脏数据。**
**==如果你用了mysql 的读写分离架构怎么办？==**
在这种情况下，造成数据不一致的原因如下，还是两个请求，一个请求A进行更新操作，另一个请求B进行查询操作。
（1）请求A进行写操作，删除缓存
（2）请求A将数据写入数据库了，
（3）请求B查询缓存发现，缓存没有值
（4）请求B去从库查询，这时，还没有完成主从同步，因此查询到的是旧值
（5）请求B将旧值写入缓存
（6）数据库完成主从同步，从库变为新值
上述情形，就是数据不一致的原因。还是使用双删延时策略。只是，**睡眠时间修改为在主从同步的延时时间基础上，加几百ms。**
**采用这种同步淘汰策略，吞吐量降低怎么办？**
ok**，那就将第二次删除作为异步的。自己起一个线程，异步删除。**这样，写的请求就不用沉睡一段时间后了，再返回。这么做，加大吞吐量。
**第二次删除失败仍然会造成缓存和数据库不一致的问题**

因为第二次删除失败，就会出现如下情形。还是有两个请求，一个请求A进行更新操作，另一个请求B进行查询操作，为了方便，假设是单库：
（1）请求A进行写操作，删除缓存
（2）请求B查询发现缓存不存在
（3）请求B去数据库查询得到旧值
（4）请求B将旧值写入缓存
（5）请求A将新值写入数据库
（6）请求A试图去删除请求B写入对缓存值，结果失败了。
**如何解决呢？**
具体解决方案，且看博主对第(3)种更新策略的解析。

### 先更新数据库，再删缓存

- **失效**：应用程序先从 cache 取数据，没有得到，则从数据库中取数据，成功后，放到缓存中。
- **命中**：应用程序从 cache 中取数据，取到后返回。
- **更新**：先把数据存到数据库中，成功后，再让缓存失效。

这种方式也会产生并发问题。假设这会有两个请求，一个请求 A 做查询操作，一个请求 B 做更新操作，那么会有如下情形产生，从而发生脏数据。
（1）缓存刚好失效
（2）请求 A 查询数据库，得一个旧值
（3）请求 B 将新值写入数据库
（4）请求 B 删除缓存
（5）请求 A 将查到的旧值写入缓存

但是发生上述情况有一个先天性条件，就是步骤（3）的写数据库操作比步骤（2）的读数据库操作耗时更短，才有可能使得步骤（4）先于步骤（5）。可是，大家想想，数据库的读操作的速度远快于写操作的（不然做读写分离干嘛，做读写分离的意义就是因为读操作比较快，耗资源少），因此步骤（3）耗时比步骤（2）更短，这一情形出现的概率很低。

**如何一定要解决上述并发问题？**
首先，给缓存设有效时间是一种方案。其次，采用策略（2）里给出的异步延时删除策略，保证读请求完成以后，再进行删除操作。
**还有其他造成不一致的原因么？**
有的，这也是缓存更新策略（2）和缓存更新策略（3）都存在的一个问题，如果删缓存失败了怎么办，那不是会有不一致的情况出现么。比如一个写数据请求，然后写入数据库了，删缓存失败了，这会就出现不一致的情况了。这也是缓存更新策略（2）里留下的最后一个疑问。
**如何解决？**
提供一个保障的重试机制即可，这里给出两套方案。
**方案一**：
如下图所示
![image](https://images.cnblogs.com/cnblogs_com/rjzheng/1202350/o_update1.png)
流程如下所示
（1）更新数据库数据；
（2）缓存因为种种问题删除失败
（3）将需要删除的key发送至消息队列
（4）自己消费消息，获得需要删除的key
（5）继续重试删除操作，直到成功
然而，该方案有一个缺点，对业务线代码造成大量的侵入。于是有了方案二，在方案二中，启动一个订阅程序去订阅数据库的binlog，获得需要操作的数据。在应用程序中，另起一段程序，获得这个订阅程序传来的信息，进行删除缓存操作。
**方案二**：
![image](https://images.cnblogs.com/cnblogs_com/rjzheng/1202350/o_update2.png)
流程如下图所示：
（1）更新数据库数据
（2）数据库会将操作信息写入binlog日志当中
（3）订阅程序提取出所需要的数据以及key
（4）另起一段非业务代码，获得该信息
（5）尝试删除缓存操作，发现删除失败
（6）将这些信息发送至消息队列
（7）重新从消息队列中获得该数据，重试操作。

**备注说明：**上述的订阅binlog程序在mysql中有现成的中间件叫canal，可以完成订阅binlog日志的功能。至于oracle中，博主目前不知道有没有现成中间件可以使用。另外，重试机制，博主是采用的是消息队列的方式。如果对一致性要求不是很高，直接在程序中另起一个线程，每隔一段时间去重试即可，这些大家可以灵活自由发挥，只是提供一个思路。

## Redis 的内存回收机制(过期键值对的删除策略、内存淘汰机制)

### 设置与移除键的生存时间或过期时间

redis 一共有 4 个命令来设置键的生存时间（可以存活多久）或过期时间（什么时候被删除）

- expire ：将 key 的生存时间设置为 ttl 秒
- pexpire ：将 key 的生存时间设置为 ttl 毫秒
- expireat ：将 key 的过期时间设置为 timestamp 所指定的秒数时间戳
- pexpireat ：将 key 的过期时间设置为 timestamp 所指定的毫秒数时间戳

上述四种命令本质上都是通过 pexpireat 命令来实现的。

```shell
127.0.0.1:6379> set a test
OK
127.0.0.1:6379> EXPIRE a 5
(integer) 1
127.0.0.1:6379> get a // 距离设置生存时间命令的 5 秒内执行
"test"
127.0.0.1:6379> get a // 距离设置生存时间命令的 5 秒后执行
(nil)
```

如果自己不小心设置错了过期时间，那么我们可以删除先前的过期时间

可以使用下面命令来返回键剩余的生存时间： ttl 是以秒为单位，返回键的剩余生存时间；同理还有 pttl 命令是以毫秒为单位，返回键的剩余生存时间

#### 移除过期时间

persist 命令可以移除一个键的过期时间，举个栗子：

```shell
127.0.0.1:6379> EXPIRE c 1000
(integer) 1
127.0.0.1:6379> ttl c   // 有过期时间
(integer) 9996
127.0.0.1:6379> PERSIST c
(integer) 1
127.0.0.1:6379> ttl c  // 无过期时间
(integer) -1
```

此时，如果我们没有移除过期时间，那么如果一个键过期了，那它什么时候会

### Redis 过期键的删除策略（即什么时候删除）

Redis 可以为每个键设置过期时间，当键过期时，会自动删除该键。**对于散列表这种容器，只能为整个键设置过期时间（整个散列表），而不能为键里面的单个元素设置过期时间。**

- **惰性过期**：**只有当访问一个 key 时，才会判断 key 是否过期，过期则清除**。该策略可以最大化的节省CPU资源，但是如果 某些键值对一直不使用，会造成一定量的内存浪费，对内存不友好。除非手动执行 flushdb 操来于清空当前数据库中的所有 key

    执行**数据写入**过程中，首先通过expireIfNeeded函数对写入的key进行过期判断。

    ```cpp
    /*
     * 为执行写入操作而取出键 key 在数据库 db 中的值。
     *
     * 和 lookupKeyRead 不同，这个函数不会更新服务器的命中/不命中信息。
     *
     * 找到时返回值对象，没找到返回 NULL 。
     */
    robj *lookupKeyWrite(redisDb *db, robj *key) {
    
        // 删除过期键
        expireIfNeeded(db,key);
    
        // 查找并返回 key 的值对象
        return lookupKey(db,key);
    }
    ```

     执行**数据读取**过程中，首先通过expireIfNeeded函数对写入的key进行过期判断。

    ```kotlin
    /*
     * 为执行读取操作而取出键 key 在数据库 db 中的值。
     *
     * 并根据是否成功找到值，更新服务器的命中/不命中信息。
     *
     * 找到时返回值对象，没找到返回 NULL 。
     */
    robj *lookupKeyRead(redisDb *db, robj *key) {
        robj *val;
    
        // 检查 key 释放已经过期
        expireIfNeeded(db,key);
    
        // 从数据库中取出键的值
        val = lookupKey(db,key);
    
        // 更新命中/不命中信息
        if (val == NULL)
            server.stat_keyspace_misses++;
        else
            server.stat_keyspace_hits++;
    
        // 返回值
        return val;
    }
    ```

     执行**过期动作expireIfNeeded**其实内部做了三件事情，分别是：

    - 查看key判断是否过期
    - 向slave节点传播执行过期key的动作并发送事件通知
    - 删除过期key

    ```kotlin
    /*
     * 检查 key 是否已经过期，如果是的话，将它从数据库中删除。
     *
     * 返回 0 表示键没有过期时间，或者键未过期。
     *
     * 返回 1 表示键已经因为过期而被删除了。
     */
    int expireIfNeeded(redisDb *db, robj *key) {
    
        // 取出键的过期时间
        mstime_t when = getExpire(db,key);
        mstime_t now;
    
        // 没有过期时间
        if (when < 0) return 0; /* No expire for this key */
    
        /* Don't expire anything while loading. It will be done later. */
        // 如果服务器正在进行载入，那么不进行任何过期检查
        if (server.loading) return 0;
    
        // 当服务器运行在 replication 模式时
        // 附属节点并不主动删除 key
        // 它只返回一个逻辑上正确的返回值
        // 真正的删除操作要等待主节点发来删除命令时才执行
        // 从而保证数据的同步
        if (server.masterhost != NULL) return now > when;
    
        // 运行到这里，表示键带有过期时间，并且服务器为主节点
    
        /* Return when this key has not expired */
        // 如果未过期，返回 0
        if (now <= when) return 0;
    
        /* Delete the key */
        server.stat_expiredkeys++;
    
        // 向 AOF 文件和附属节点传播过期信息
        propagateExpire(db,key);
    
        // 发送事件通知
        notifyKeyspaceEvent(REDIS_NOTIFY_EXPIRED,
            "expired",key,db->id);
    
        // 将过期键从数据库中删除
        return dbDelete(db,key);
    }
    ```

     判断key是否过期的数据结构是db->expires，也就是通过expires的数据结构判断数据是否过期。
    内部获取过期时间并返回。

    ```php
    /* Return the expire time of the specified key, or -1 if no expire
     * is associated with this key (i.e. the key is non volatile) 
     *
     * 返回给定 key 的过期时间。
     *
     * 如果键没有设置过期时间，那么返回 -1 。
     */
    long long getExpire(redisDb *db, robj *key) {
        dictEntry *de;
    
        /* No expire? return ASAP */
        // 获取键的过期时间
        // 如果过期时间不存在，那么直接返回
        if (dictSize(db->expires) == 0 ||
           (de = dictFind(db->expires,key->ptr)) == NULL) return -1;
    
        /* The entry was found in the expire dict, this means it should also
         * be present in the main dict (safety check). */
        redisAssertWithInfo(NULL,key,dictFind(db->dict,key->ptr) != NULL);
    
        // 返回过期时间,#define dictGetSignedIntegerVal(he) ((he)->v.s64)
        return dictGetSignedIntegerVal(de);
    }
    ```

     整个数据查找过程类比hashtab的查找过程，首先定位hash桶，然后遍历hash桶下挂的链查找对应的节点。

    ```php
    /*
     * 返回字典中包含键 key 的节点
     *
     * 找到返回节点，找不到返回 NULL
     *
     * T = O(1)
     */
    dictEntry *dictFind(dict *d, const void *key)
    {
        dictEntry *he;
        unsigned int h, idx, table;
    
        // 字典（的哈希表）为空
        if (d->ht[0].size == 0) return NULL; /* We don't have a table at all */
    
        // 如果条件允许的话，进行单步 rehash
        if (dictIsRehashing(d)) _dictRehashStep(d);
    
        // 计算键的哈希值
        h = dictHashKey(d, key);
        // 在字典的哈希表中查找这个键
        // T = O(1)
        for (table = 0; table <= 1; table++) {
    
            // 计算索引值
            idx = h & d->ht[table].sizemask;
    
            // 遍历给定索引上的链表的所有节点，查找 key
            he = d->ht[table].table[idx];
            // T = O(1)
            while(he) {
    
                if (dictCompareKeys(d, key, he->key))
                    return he;
    
                he = he->next;
            }
    
            // 如果程序遍历完 0 号哈希表，仍然没找到指定的键的节点
            // 那么程序会检查字典是否在进行 rehash ，
            // 然后才决定是直接返回 NULL ，还是继续查找 1 号哈希表
            if (!dictIsRehashing(d)) return NULL;
        }
    
        // 进行到这里时，说明两个哈希表都没找到
        return NULL;
    }
    ```

- **定时删除:** **在设置键的过期时间的同时，创建一个定时器（timer），让定时器在键的过期时间来临时，立即执行对键的删除操作.** 即从设置key的Expire开始，就启动一个定时器，到时间就删除该key；这样会**对内存比较友好，但浪费CPU资源**

- **定期删除:**每隔一段时间，程序就对数据库进行一次检查，删除里面的过期键。至于要删除多少过期键，以及要检查多少个数据库，则由算法决定。 即设置一个定时任务，比如10分钟删除一次过期的key；**间隔小则占用CPU,间隔大则浪费内存**

      

    redis 的定期删除是通过定时任务实现的，也就是定时任务会循环调用 serverCron() 方法。然后定时检查过期数据的方法是 databasesCron() 。
    定期删除的一大特点就是考虑了定时删除过期数据会占用 cpu 时间，所以每次执行 databasesCron() 的时候会限制cpu的占用不超过25%。

     **activeExpireCycle() 执行具体过期数据的删除**，其他的动作不在该部分讨论当中。

     该方法中删除过期数据的整个过程主要按照下面的逻辑进行：

    - 遍历指定个数的db（如16）进行删除操作
    - 针对每个db 随机获取过期数据每次遍历不超过指定数量（如20），发现过期数据并进行删除。
    - 每个db的次数累积到16次的时候会进行判断时间是否超过25%，超过就停止删除数据过程。
    - 最后如果删除的过期数据耗时（通过开始结束时间统计）超过待过期时间数量的25%的时候就停止删除过期数据过程。
    - timelimit = 1000000*ACTIVE_EXPIRE_CYCLE_SLOW_TIME_PERC/server.hz/100的解释是：server.hz代表每秒调用的次数,所以上面这个公司就是每次执行占用的时候的25%用于过期数据删除。

    

    ```cpp
    // 对数据库执行删除过期键，调整大小，以及主动和渐进式 rehash
    void databasesCron(void) {
    
        // 如果服务器不是从服务器，那么执行主动过期键清除
        if (server.active_expire_enabled && server.masterhost == NULL)
            // 清除模式为 CYCLE_SLOW ，这个模式会尽量多清除过期键
            activeExpireCycle(ACTIVE_EXPIRE_CYCLE_SLOW);
    
        // 在没有 BGSAVE 或者 BGREWRITEAOF 执行时，对哈希表进行 rehash
        if (server.rdb_child_pid == -1 && server.aof_child_pid == -1) {
            static unsigned int resize_db = 0;
            static unsigned int rehash_db = 0;
            unsigned int dbs_per_call = REDIS_DBCRON_DBS_PER_CALL;
            unsigned int j;
    
            /* Don't test more DBs than we have. */
            // 设定要测试的数据库数量
            if (dbs_per_call > server.dbnum) dbs_per_call = server.dbnum;
    
            /* Resize */
            // 调整字典的大小
            for (j = 0; j < dbs_per_call; j++) {
                tryResizeHashTables(resize_db % server.dbnum);
                resize_db++;
            }
    
            /* Rehash */
            // 对字典进行渐进式 rehash
            if (server.activerehashing) {
                for (j = 0; j < dbs_per_call; j++) {
                    int work_done = incrementallyRehash(rehash_db % server.dbnum);
                    rehash_db++;
                    if (work_done) {
                        /* If the function did some work, stop here, we'll do
                         * more at the next cron loop. */
                        break;
                    }
                }
            }
        }
    }
    ```

    

    ```cpp
    void activeExpireCycle(int type) {
        // 静态变量，用来累积函数连续执行时的数据
        static unsigned int current_db = 0; /* Last DB tested. */
        static int timelimit_exit = 0;      /* Time limit hit in previous call? */
        static long long last_fast_cycle = 0; /* When last fast cycle ran. */
    
        unsigned int j, iteration = 0;
        // 默认每次处理的数据库数量
        unsigned int dbs_per_call = REDIS_DBCRON_DBS_PER_CALL;
        // 函数开始的时间
        long long start = ustime(), timelimit;
    
        // 快速模式
        if (type == ACTIVE_EXPIRE_CYCLE_FAST) {
            // 如果上次函数没有触发 timelimit_exit ，那么不执行处理
            if (!timelimit_exit) return;
            // 如果距离上次执行未够一定时间，那么不执行处理
            if (start < last_fast_cycle + ACTIVE_EXPIRE_CYCLE_FAST_DURATION*2) return;
            // 运行到这里，说明执行快速处理，记录当前时间
            last_fast_cycle = start;
        }
    
        /* 
         * 一般情况下，函数只处理 REDIS_DBCRON_DBS_PER_CALL 个数据库，
         * 除非：
         *
         * 1) 当前数据库的数量小于 REDIS_DBCRON_DBS_PER_CALL
         * 2) 如果上次处理遇到了时间上限，那么这次需要对所有数据库进行扫描，
         *     这可以避免过多的过期键占用空间
         */
        if (dbs_per_call > server.dbnum || timelimit_exit)
            dbs_per_call = server.dbnum;
    
        // 函数处理的微秒时间上限
        // ACTIVE_EXPIRE_CYCLE_SLOW_TIME_PERC 默认为 25 ，也即是 25 % 的 CPU 时间
        timelimit = 1000000*ACTIVE_EXPIRE_CYCLE_SLOW_TIME_PERC/server.hz/100;
        timelimit_exit = 0;
        if (timelimit <= 0) timelimit = 1;
    
        // 如果是运行在快速模式之下
        // 那么最多只能运行 FAST_DURATION 微秒 
        // 默认值为 1000 （微秒）
        if (type == ACTIVE_EXPIRE_CYCLE_FAST)
            timelimit = ACTIVE_EXPIRE_CYCLE_FAST_DURATION; /* in microseconds. */
    
        // 遍历数据库
        for (j = 0; j < dbs_per_call; j++) {
            int expired;
            // 指向要处理的数据库
            redisDb *db = server.db+(current_db % server.dbnum);
    
            // 为 DB 计数器加一，如果进入 do 循环之后因为超时而跳出
            // 那么下次会直接从下个 DB 开始处理
            current_db++;
    
            do {
                unsigned long num, slots;
                long long now, ttl_sum;
                int ttl_samples;
    
                /* If there is nothing to expire try next DB ASAP. */
                // 获取数据库中带过期时间的键的数量
                // 如果该数量为 0 ，直接跳过这个数据库
                if ((num = dictSize(db->expires)) == 0) {
                    db->avg_ttl = 0;
                    break;
                }
                // 获取数据库中键值对的数量
                slots = dictSlots(db->expires);
                // 当前时间
                now = mstime();
    
                // 这个数据库的使用率低于 1% ，扫描起来太费力了（大部分都会 MISS）
                // 跳过，等待字典收缩程序运行
                if (num && slots > DICT_HT_INITIAL_SIZE &&
                    (num*100/slots < 1)) break;
    
                /* 
                 * 样本计数器
                 */
                // 已处理过期键计数器
                expired = 0;
                // 键的总 TTL 计数器
                ttl_sum = 0;
                // 总共处理的键计数器
                ttl_samples = 0;
    
                // 每次最多只能检查 LOOKUPS_PER_LOOP 个键
                if (num > ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP)
                    num = ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP;
    
                // 开始遍历数据库
                while (num--) {
                    dictEntry *de;
                    long long ttl;
    
                    // 从 expires 中随机取出一个带过期时间的键
                    if ((de = dictGetRandomKey(db->expires)) == NULL) break;
                    // 计算 TTL
                    ttl = dictGetSignedIntegerVal(de)-now;
                    // 如果键已经过期，那么删除它，并将 expired 计数器增一
                    if (activeExpireCycleTryExpire(db,de,now)) expired++;
                    if (ttl < 0) ttl = 0;
                    // 累积键的 TTL
                    ttl_sum += ttl;
                    // 累积处理键的个数
                    ttl_samples++;
                }
    
                /* Update the average TTL stats for this database. */
                // 为这个数据库更新平均 TTL 统计数据
                if (ttl_samples) {
                    // 计算当前平均值
                    long long avg_ttl = ttl_sum/ttl_samples;
                    
                    // 如果这是第一次设置数据库平均 TTL ，那么进行初始化
                    if (db->avg_ttl == 0) db->avg_ttl = avg_ttl;
                    /* Smooth the value averaging with the previous one. */
                    // 取数据库的上次平均 TTL 和今次平均 TTL 的平均值
                    db->avg_ttl = (db->avg_ttl+avg_ttl)/2;
                }
    
                // 我们不能用太长时间处理过期键，
                // 所以这个函数执行一定时间之后就要返回
    
                // 更新遍历次数
                iteration++;
    
                // 每遍历 16 次执行一次
                if ((iteration & 0xf) == 0 && /* check once every 16 iterations. */
                    (ustime()-start) > timelimit)
                {
                    // 如果遍历次数正好是 16 的倍数
                    // 并且遍历的时间超过了 timelimit
                    // 那么断开 timelimit_exit
                    timelimit_exit = 1;
                }
    
                // 已经超时了，返回
                if (timelimit_exit) return;
    
                // 如果已删除的过期键占当前总数据库带过期时间的键数量的 25 %
                // 那么不再遍历
            } while (expired > ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP/4);
        }
    }
    ```

    reids 同时使用了惰性过期和定期过期两种策略。通过配合使用，服务器可以很好的平衡 CPU 和内存。

其中惰性删除为 redis 服务器内置策略。而定期删除可以通过以下两种方式设置： 1. 配置 redis.conf 的 hz 选项，默认为10 （即 1 秒执行 10 次，值越大说明刷新频率越快，对 Redis 性能损耗也越大） 2. 配置 redis.conf 的 maxmemory 最大值，当已用内存超过 maxmemory 限定时，就会触发主动清理策略

### RDB 和 AOF 对过期键的处理

- 生成 RDB 文件

    程序会被数据库中的键进行检查，过期的键不会被保存到新创建的 RDB 文件中。因此**数据库中的过期键不会对生成新的 RDB 文件造成影响**

- 载入 RDB 文件

    这里需要分情况说明： 1. 如果服务器以主服务器模式运行，则在载入 RDB 文件时，程序会对文件中保存的键进行检查，过期键不会被载入到数据库中。**所以过期键不会对载入 RDB 文件的主服务器造成影响**。 2. 如果服务器以从服务器模式运行，则在载入 RDB 文件时，不论键是否过期都会被载入到数据库中。但由于主从服务器在进行数据同步时，从服务器的数据会被清空。所以一般来说，**过期键对载入 RDB 文件的从服务器也不会造成影响**。

- AOF 文件写入

    当服务器以 AOF 持久化模式运行时，如果数据库某个过期键还没被删除，那么 AOF 文件不会因为这个过期键而产生任何影响，依旧保留。

    而当过期键被删除后，那么程序会向 AOF 文件追加一条 DEL 命令来显式地记录该键被删除。

- AOF 重写

    执行 AOF 重写过程中，也会被数据库的键进行检查，已过期的键不会被保存到重写后的 AOF 文件中。因此**不会对 AOF 重写造成影响**

- 复制对过期键的处理

    当服务器运行在复制模式下，由主服务器来控制从服务器的删除过期键动作，目的是保证主从服务器数据的一致性。

    那到底是怎么控制的呢？ 1. 主服务器删除一个过期键后，会向所有从服务器发送一个 DEL 命令，告诉从服务器删除这个过期键 2. 从服务器接受到命令后，删除过期键

    PS:从服务器在接收到客户端对过期键的读命令时，依旧会返回该键对应的值给客户端，而不会将其删除。



### ==内存淘汰机制==

为了保证 Redis 的安全稳定运行，设置了一个 `max-memory` 的阈值，当内存用量到达阈值，新写入的键值对无法写入，此时就需要内存淘汰机制，在 Redis 的配置中有几种淘汰策略可以选择，详细如下：

- `noeviction`: 当内存不足以容纳新写入数据时，新写入操作会**报错**；
- `allkeys-lru`：当内存不足以容纳新写入数据时，在键空间中**移除最近最少使用的 key**；
- `allkeys-random`：当内存不足以容纳新写入数据时，在键空间中**随机移除某个 key**；
- `volatile-lru`：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，移除最近最少使用的 key；
- `volatile-random`：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，随机移除某个 key；
- `volatile-ttl`：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，有更早过期时间的 key 优先移除；

## Redis 主从复制

Redis单节点存在单点故障问题，为了解决单点问题，一般需要对 Redis 配置从节点，然后使用哨兵来监听主节点的存活状态，如果主节点挂掉，从节点能继续提供缓存功能。

**主从复制结合哨兵模式能解决单点故障问题，提高 Redis 可用性。主节点提供写操作，从节点仅提供读操作**。

**主从复制的作用：**

- **数据冗余**：实现数据冗余备份，这样一台节点挂了之后，其上的数据不至于丢失。
- **故障恢复**：当主节点出现问题时，其从节点可以被提升为主节点继续提供服务，实现快速的故障恢复；
- **负载均衡**：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务，分担服务器负载；尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高Redis服务器的并发量。

### 主从复制的过程和原理

**Redis主从复制默认是异步实现的**，避免客户端的等待。
<img src="../../../../../Yu%2520Writer%2520Libraries/Interview/InterviewExperience/Redis.resource/image-20200602094035427.png" alt="image-20200602094035427" style="zoom:80%;" />

**复制过程：**

- 从节点执行`slavaof[masterIP][masterPort]`，保存主节点信息。
- 从节点的定时任务发现主节点信息，建立和主节点的Socket连接；从节点发送Ping信号，主节点返回Pong，连接建立后，主节点将所有数据发送给从节点(**全量复制**)。
- 完成了复制的建立过程后，主节点持续的把写命令发送给从节点(**增量复制**)，保证主从数据一致性。

全量复制是从结点因为故障恢复或者新添加从结点时出现的初始化阶段的数据复制，这种复制是将主节点的数据全部同步到从结点来完成的，所以成本大但又不可避免。

增量复制是主从结点正常工作之后的每个时刻进行的数据复制方式。

**主从复制存在的问题：**

一旦主节点宕机，从节点晋升为主节点，同时需要修改应用方的主节点地址、命令所有从节点去复制新的主节点，整个过程需要人工干预。

解决方案：**哨兵**

### 11.2 哨兵模式

Redis Sentinel(哨兵)的主要功能包括主节点存活检测、主从运行情况检测、自动故障转移、主从切换。哨兵模式的最小配置是一主一从。

<img src="../../../../../Yu%2520Writer%2520Libraries/Interview/InterviewExperience/Redis.resource/image-20200602094252997.png" alt="image-20200602094252997" style="zoom:80%;" />

**哨兵功能**：

- 监控（Monitoring）： Sentinel 会不断地检查你的主服务器和从服务器是否运作正常。
- 提醒（Notification）： 当被监控的某个 Redis 服务器出现问题时， Sentinel 可以通过 API 向管理员或者其他应用程序发送通知。
- 自动故障迁移（Automatic failover）： 当一个主服务器不能正常工作时， Sentinel 会开始一次自动故障迁移操作， 它会进行选举，将其中一个从服务器升级为新的主服务器， 并让失效主服务器的其他从服务器改为复制新的主服务器； 当客户端试图连接失效的主服务器时， 集群也会向客户端返回新主服务器的地址， 使得集群可以使用新主服务器代替失效服务器。


**哨兵的工作原理：**

1. 每个哨兵节点定期执行以下任务：每个哨兵节点以每秒一次的频率，向它所知的主服务器、从服务器以及其他的哨兵实例发送一个PING命令。
2. 如果一个服务器距离上次回复PING命令的时间超过`down-after-milliseconds`所指定的值，那么这个实例会被哨兵标记为**主观下线**。
3. 如果主服务器被标记为主观下线，那么正在监视这个服务器的所有哨兵节点，以每秒一次的频率确认主服务器的确进入了主观下线状态。
4. 如果一个主服务器被标记为主观下线，并且有足够数量的哨兵(达到配置文件指定的数量)在指定的时间范围内同意这一判断，那么这个主服务器被标记为**客观下线**。
5. 一般情况下，每个哨兵节点会以每10秒一次的频率向它已知的所有服务器发送INFO命令；当一个主服务器被标记为客观下线时，哨兵节点以每秒1次的频率向该主服务器的所有的从服务器发送INFO命令。
6. 哨兵节点和其他哨兵节点协商主节点状态，如果处于SDOWN状态，则投票选出新的主节点，将剩余从节点指向新的主节点进行数据复制。**当原主服务器重新上线后，自动转为当前Master的从节点**。
7. 当没有足够数量的哨兵节点同意主服务器下线时，主服务器的客观下线状态就会被移除；当主服务器重新向哨兵的PING命令返回有效回复时，主服务器的主观下线状态被移除。

**哨兵模式的缺陷**：
在哨兵模式中，仍然只有一个Master节点。当并发写请求较大时，哨兵模式并不能缓解写压力。

只有主节点才具有写能力，如果在一个集群中，能够配置多个主节点，就可以缓解写压力了。**这个就是redis-cluster集群模式。**

### 11.3 Redis-Cluster

Redis Sentinal着眼于高可用，在master宕机时会自动将slave提升为master，继续提供服务。

Redis Cluster着眼于扩展性，在单个redis内存不足时，使用Cluster进行分片存储。

**集群模式**
（1）由多个Redis服务器组成的分布式网络服务集群；
（2）集群之中有多个Master主节点，每一个主节点都可读可写；可以给每一个主节点添加从节点，主节点和从节点遵循主从模型的特性
（3）节点之间会互相通信，两两相连；
（4）Redis集群无中心节点。

**集群分片策略**
Redis-cluster分片策略用来解决key存储位置。

![image-20200602095139785](../../../../../Yu%2520Writer%2520Libraries/Interview/InterviewExperience/Redis.resource/image-20200602095139785.png)

**集群将整个数据库分为16384个槽位slot，所有key-value数据都存储在这些slot中的某一个上**。一个slot槽位可以存放多个数据，key的槽位计算公式为：slot_number=crc16(key)%16384，其中crc16为16位的循环冗余校验和函数。

集群中的每个主节点都可以处理0个至16383个槽，当16384个槽都有节点在负责处理时，集群进入上线状态，并开始处理客户端发送的数据命令请求。

**集群redirect转向**：

由于Redis集群无中心节点，请求会随机发给任意主节点；

**主节点只会处理自己负责槽位的命令请求，其它槽位的命令请求，该主节点会返回客户端一个转向错误。
客户端根据错误中包含的地址和端口重新向正确的负责的主节点发起命令请求。**

![image-20200602095231276](../../../../../Yu%2520Writer%2520Libraries/Interview/InterviewExperience/Redis.resource/image-20200602095231276.png)

### 四、使用场景

- 计数器

    可以对 String 进行自增自减运算，从而实现计数器功能。

    Redis 这种内存型数据库的读写性能非常高，很适合存储频繁读写的计数量。

- 缓存

    将热点数据放到内存中，设置内存的最大使用量以及淘汰策略来保证缓存的命中率。

- 查找表

    例如 DNS 记录就很适合使用 Redis 进行存储。

    查找表和缓存类似，也是利用了 Redis 快速的查找特性。但是查找表的内容不能失效，而缓存的内容可以失效，因为缓存不作为可靠的数据来源。

- 消息队列

    List 是一个双向链表，可以通过 lpush 和 rpop 写入和读取消息

    不过最好使用 Kafka、RabbitMQ 等消息中间件。

- 会话缓存

    可以使用 Redis 来统一存储多台应用服务器的会话信息。

    当应用服务器不再存储用户的会话信息，也就不再具有状态，一个用户可以请求任意一个应用服务器，从而更容易实现高可用性以及可伸缩性。

- 分布式锁实现

    在分布式场景下，无法使用单机环境下的锁来对多个节点上的进程进行同步。

    可以使用 Redis 自带的 SETNX 命令实现分布式锁，除此之外，还可以使用官方提供的 RedLock 分布式锁实现。

- 其它

    Set 可以实现交集、并集等操作，从而实现共同好友等功能。

    ZSet 可以实现有序性操作，从而实现排行榜等功能。



**使用过Redis分布式锁么，它是什么回事？**

先拿setnx来争抢锁，抢到之后，再用expire给锁加一个过期时间防止锁忘记了释放。

这时候对方会告诉你说你回答得不错，然后接着问如果在setnx之后执行expire之前进程意外crash或者要重启维护了，那会怎么样？

这时候你要给予惊讶的反馈：唉，是喔，这个锁就永远得不到释放了。紧接着你需要抓一抓自己得脑袋，故作思考片刻，好像接下来的结果是你主动思考出来的，然后回答：我记得set指令有非常复杂的参数，这个应该是可以同时把setnx和expire合成一条指令来用的！对方这时会显露笑容，心里开始默念：摁，这小子还不错。

### 假如Redis里面有1亿个key，其中有10w个key是以某个固定的已知的前缀开头的，如果将它们全部找出来？

使用keys指令可以扫出指定模式的key列表。

对方接着追问：如果这个redis正在给线上的业务提供服务，那使用keys指令会有什么问题？

这个时候你要回答redis关键的一个特性：redis的单线程的。keys指令会导致线程阻塞一段时间，线上服务会停顿，直到指令执行完毕，服务才能恢复。这个时候可以使用scan指令，scan指令可以无阻塞的提取出指定模式的key列表，但是会有一定的重复概率，在客户端做一次去重就可以了，但是整体所花费的时间会比直接用keys指令长。

**使用过Redis做异步队列么，你是怎么用的？**

一般使用 list结构作为队列，rpush生产消息，lpop消费消息。当lpop没有消息的时候，要适当sleep一会再重试。

如果对方追问可不可以不用sleep呢？list还有个指令叫blpop，在没有消息的时候，它会阻塞住直到消息到来。

如果对方追问能不能生产一次消费多次呢？使用pub/sub主题订阅者模式，可以实现1:N的消息队列。

如果对方追问pub/sub有什么缺点？在消费者下线的情况下，生产的消息会丢失，得使用专业的消息队列如rabbitmq等。

如果对方追问redis如何实现延时队列？我估计现在你很想把面试官一棒打死如果你手上有一根棒球棍的话，怎么问的这么详细。但是你很克制，然后神态自若的回答道：使用sortedset，拿时间戳作为score，消息内容作为key调用zadd来生产消息，消费者用zrangebyscore指令获取N秒之前的数据轮询进行处理。

到这里，面试官暗地里已经对你竖起了大拇指。但是他不知道的是此刻你却竖起了中指，在椅子背后。

**Pipeline有什么好处，为什么要用pipeline？**

可以将多次IO往返的时间缩减为一次，前提是pipeline执行的指令之间没有因果相关性。使用redis-benchmark进行压测的时候可以发现影响redis的QPS峰值的一个重要因素是pipeline批次指令的数目。

### Redis的同步机制

Redis可以使用主从同步，从从同步。第一次同步时，主节点做一次bgsave，并同时将后续修改操作记录到内存buffer，待完成后将rdb文件全量同步到复制节点，复制节点接受完成后将rdb镜像加载到内存。加载完成后，再通知主节点将期间修改的操作记录同步到复制节点进行重放就完成了同步过程。











# Redis

## 一、非关系型数据库
### （一）NOSQL
关系型数据库：MySQL 、Oracle、SQLServer
NoSQL,泛指**非关系型的数据库**, NOSQL数据库的四大分类

- **键值 (Key-value)存储数据库**:这一类数据库主要会使用到一个哈希表,这个表中有一个特定的键和一个指针指向特定的数据。如 Redis, oldemort,Oracle BDB;

- 列存储数据库:这部分数据库通常是用来应对分布式存储的海量数据。键仍然存在,但是它们的特点是指向了多个列。如 HBase,Rak

- 文档型数据库:该类型的数据模型是版本化的文档,半结构化的文档以特定的格式存储,比如JsoN。文档型数据库可以看作是键值数据库的升级版,允许之间嵌套键值。而且文档型数据库比键值数据库的查询效率更高。如：CouchDB, MongoDb

- 图形 (Graph)数据库:图形结构的数据库同其他行列以及刚性结构的SQL数据库不同,它是使用灵活的图形模型,并且能够扩展到多个服务器上。NoSQ 数据库没有标准的查询语言(SQL),因此进行数据库查询需要制定数据模型

许多 NOSQL数据库都有REST式的数据接口或者查询AP。如:Neo4J,InfoGrid, Infinite Graph

### （二）非关系型数据库特点

- 数据模型比较简单
因为采用 key - value 格式；
- 需要灵活性更强的 IT 系统
- 对数据库性能要求较高
- 不需要高度的数据一致性
即不一定具备完善的 ACID；
- 对于给定 key，比较容易映射复杂值的环境

## 二、Redis 简介

是以 key-value形式存储,和传统的关系型数据库不一样,不一定遵循传统数据库的一些基本要求(非关系型的、分布式的、开源的。水平可扩展的)

- 优点:
  - 对数据高并发读写
  - 对海量数据的高效率存储和访问
因为是基于内存级别的读取操作
  - 对数据的可扩展性和高可用性

- 缺点: 
  - redis(ACID处理非常简单）无法做到太复杂的关系数据库模型

Redis是以 key-value 存储数据结构服务器。键可以包含:( string)字符串,哈希,（List）链表,（Set）集合，(zset)有序集合。这些数据集合都支持 push/pop、 add/ remove及取交集和并集以及更丰富的操作，redis 支持各种不同的方式排序。
为了保证效率数据都是缓存在内存中它也可以周期性的把更新的数据写入磁盘或者把修改操作写入追加到文件中。上述持久化主要方式是：RDB 和 AOF、RDB 是周期性将操作写入硬盘；AOF 是在执行数据写入操作的同时将操作写入日志，如果发生掉电可以从日志中查询到之前的操作。


### （一）Redis 安装
 见 Linux -> LinuxBlogs -> CentOS 安装 Redis；

**基本命令：**
路径按照上面文件进行安排

- 启动服务：`src/redis-server redis.conf`
- 退出服务：`pkill redis-server`/ `kill 进程号` / `src/redis-cli shutdown`
- 启动客户端：`src/redis-cli`
- 退出客户端：`quit`
- 查看是否有 Redis 服务或者查看端口：`ps -ef | grep redis` 或者 `netstat -tunpl | grep 6379`


## 三、Redis 数据类型
redis 一共分为五种基本数据类型： String、Hash、List（类型 Java 的 Queue）、Set（Java 中 List 增强版）、ZSet；

### （一）String 类型

String 类型是包含很多种类型的特殊类型，并且**是二进制安全的**。比如序列化的对象进行存储，比如一张图片进行二进制存储，比如一个简单的字符串。数值等等。


- Set 方法：
  - 设置值：`set key值 value值`，重复为同一个 key 值设置 value，新的会覆盖旧的值；
  - 设置值：`setnx key值 value值`，如果该 key 存在则不再设置，返回 0，否则进行设置；
  - 设置值有效期：`setex key值 过期时间 value值`，设置 key 值有效期（秒）。过期则返回 nil（相当于 null）；示例：`setex key 10 value`设置该值的有效期为 10 秒；
  - 取值：`get key值`
  - 删除值：`del key值`
  - 使用 setrange 替换字符串：比如之前的值：`set key1 value123`；然后通过 `setrange key1 4 ww`表示将 key1 对应的 value 值，从第 4 位开始（从 0 开始）逐个替换为 ww，上面示例的结果值为：`valuww23`；

- 批量设置多个值或者批量取出多个值：
  - 批量设置值：`mset key1 value1 key2 value2 key3 value3`；
  - 批量获取值：`mget key1 key2 key3`
  - 同样也有 msetnx 和 mget 方法；
  - 一次性设置和取值：getset 方法，示例，已经设置值：`set key1 value1`，然后：`getset key1 value2`，首先会返回原来的值 value1，同时将 key1 值设置为：value2；
- 其他方法
  - incr 和 decr ：对 key 对应的 value 值 +/- 1（因为默认步长为 1）；`incr key1`，key1 对应的值的大小 + 1；
  - incrby 和 decr 方法：对某个值进行制定长度的递增和递减，可以使用：`incrby key1 2`，将 key1 对应的值 + 2，这里 2 表示步长。步长可正可负。
  - append 方法：在 key 对应值的后面追加；`append key1 lalala`，在 key1 对应的 value 值后面追加 lalala。
  - strlen 方法：获取 key 对应的值的长度；


### （二）Hash 类型
 Hash 类型是 String 类型的 field 和 value 的映射表，或者说一个 String 集合，**特别适合存储对象**，相比，将一个对象类型存储在 Hash 类型中要比存储在 String 类型中占用内存空间更小同时更方便存取整个对象。

一般之前数据表中数据在 redis 中有两种方式存储

id | name | age
---|---|---
1|zhang|21
2|lisi|22

- 方式一：使用默认的 Map 形式（一般适用于经常存取部分字段时候使用）
user1 中 ：id:1，name:zhang，age:21；使用键值对存放
然后将 user1 和 user2 整合为 user 表；

- 方式二：使用 JSON 格式（一般适用于经常取出整条数据）
这里一般假设 id 即是 UUID
所以：key=uuid，value:{id=1，name=lisi，age=22}

**语法示例：**
- 存储内容：`hset hash集合名字 key值（字段名） value值`
- 批量存取内容：`hset hash集合名字 key1 value1 key2 value2`
- 获取内容：`hget hash集合名字 key值`
- 批量获取内容：`hmget hash集合名字 key1 key2`
- hsetnx 和 setnx 相似
- hincrby 和 hdecrby 集合的递增和递减
- hexists 是否存在 key，如果存在就返回，不存在返回 0
- hlen：返回 hash 集合中所有的键的数量
- hdel ：删除指定 hash 中的 key
- hkeys：返回 hash 中多有字段（key）
- hvals：返回 hash 中所有的 value
- hgetall：返回 hash 中所有的 key 和 value；


### （三）List 类型

List 类型是链表结构的集合，主要功能有：push、pop、获取元素等等。**List 类型是一个双端链表的结构，可以通过相关操作进行集合的头部或者尾部添加删除元素**，List 可以作为栈也可以作为队列。
**这也导致 Redis 可以充当 消息队列（MQ）使用**
- lpush 方法：从头部加入元素，先进后出（相当于栈）
```redis
127.0.0.1:6379> lpush list1 "hello"
(integer) 1
127.0.0.1:6379> lpush list1 "world"
(integer) 2
127.0.0.1:6379> lrange list1 0 -1 // 表示从头取到末尾，就是查询栈中所有元素 
1) "world"
2) "hello"
```

- rpush 方法：从尾部添加元素，先进先出（相当于队列）
```redis
127.0.0.1:6379> rpush list2 "hello"
(integer) 1
127.0.0.1:6379> rpush list2 "world"
(integer) 2
127.0.0.1:6379> lrange list2 0 -1
1) "hello"
2) "world"
```

- linsert 方法：插入元素
`linsert hash集合名 before 集合已有的元素 插入的元素`
```redis
127.0.0.1:6379> linsert list2 before "world" "your"
(integer) 3
127.0.0.1:6379> lrange list2 0 -1
1) "hello"
2) "your"
3) "world"

```

- lset 方法：将指定下标的元素替换掉
`lset 集合名 要替换元素在集合中的位置 新的值`
```redis
127.0.0.1:6379> lset list2 1 "my"
OK
127.0.0.1:6379> lrange list2 0 -1
1) "hello"
2) "my"
3) "world"
```

- lrem 方法：删除元素，返回删除元素的个数
`lrem 集合名 删除几个该名称元素  要删除的元素名称`
```redis
127.0.0.1:6379> lrange list2 0 -1
1) "hello"
2) "my"
3) "my"
4) "my"
5) "world"
127.0.0.1:6379> lrem list2 2 "my"
(integer) 2
127.0.0.1:6379> lrange list2 0 -1
1) "hello"
2) "my"
3) "world"
// 如果再想删除两个 "my"，最后实际上仅仅删除了唯一的一个
127.0.0.1:6379> lrem list2 2 "my"
(integer) 1
127.0.0.1:6379> lrange list2 0 -1
1) "hello"
2) "world"

```

- ltrim 方法：保留指定 key 的值范围内的数据
`ltrim 集合名 开始位置  结尾位置`：保留集合中 [开始位置，结尾位置] 元素，其他元素删除；
```redis
127.0.0.1:6379> lrange list2 0 -1
1) "hello"
2) "my"
3) "your"
4) "his"
5) "her"
6) "world"
127.0.0.1:6379> ltrim list2 2 4
OK
127.0.0.1:6379> lrange list2 0 -1
1) "your"
2) "his"
3) "her"
```

- lpop 方法：从 list 的头部删除元素，并返回删除元素
- rpop 方法：从 list 的尾部删除元素，并返回删除元素
```redis
127.0.0.1:6379> lrange list2 0 -1
1) "your"
2) "his"
3) "her"
127.0.0.1:6379> lpop list2
"your"
127.0.0.1:6379> rpop list2
"her"
127.0.0.1:6379> lrange list2 0 -1
1) "his"
```

- rpoplpush 方法：第一步从尾部删除元素，然后从头部加入元素
- lindex 方法：返回指定 list 中 Index 位置的元素：`lindex list2 3`
- llen 方法：返回元素的个数：`lindex list2`
```redis
127.0.0.1:6379> lrange list2 0 -1
1) "her"
2) "my"
3) "your"
4) "its"
5) "his"
// 将 list2 中尾部的元素删除，并将该元素加入 list1 的头部
127.0.0.1:6379> rpoplpush list2 list1
"his"
127.0.0.1:6379> lrange list2 0 -1
1) "her"
2) "my"
3) "your"
4) "its"
127.0.0.1:6379> lrange list1 0 -1
1) "his"
2) "world"
3) "hello"
```


### （四）Set 类型和 ZSet 类型

Set 集合是 String 类型的无序集合，set 是通过 hashTable 实现的，对集合我们可以取交集、并集、差集。

- sadd 方法：向 set 集合中添加元素（set 集合中不允许存在重复元素）
- smembers 方法：查看 set 集合的元素
- srem 方法：删除 set 集合元素
- spop 方法：随机返回删除的 key `spop set集合名 数量` 随机删除 set 集合中的一部分元素；
- sdiff 方法：返回后一个集合中不在前一个集合中的元素（哪个集合在前就以那个集合为标准）
- sdiffstore 方法：将上面方法返回的元素存储到一个新的集合中
```redis
127.0.0.1:6379> sadd set2 "f"
(integer) 1
127.0.0.1:6379> smembers set1
1) "d"
2) "c"
3) "b"
4) "a"
127.0.0.1:6379> smembers set2
1) "f"
2) "c"
3) "e"
4) "a"
127.0.0.1:6379> sdiffstore set3 set1 set2
(integer) 2
127.0.0.1:6379> smembers set1
1) "d"
2) "c"
3) "b"
4) "a"
127.0.0.1:6379> smembers set2
1) "f"
2) "c"
3) "e"
4) "a"
127.0.0.1:6379> smembers set3
1) "d"
2) "b"

```

- sinter 方法：返回集合的交集 `sinter set1 set2`
- sinterstore 方法：返回集合的交集结果数量，并将交集存入新的集合；`sinterstore set3 set1 set2`。
- sunion 方法：取两个集合的并集；`sunion set1 set2`
- sunionstore 方法：将两个集合的并集存到第三个集合中；`sunionstore set3 set1 set2`

- smove 方法：将某个集合中的元素移动到另一个集合中 `smove 从这个集合名  移动到这个集合名  要移动的元素`
- scard 方法：查看集合中元素的个数 `scard 集合名`
- sismember 方法：判断某元素是否为集合中的元素（返回 1 代表是的，返回 0 代表不是）`sismember 集合名 元素名`
- srandmember 方法：随机返回集合中一个元素 `srandmember 集合名`



**ZSet 方法**
- zadd 方法：向有序集合中添加一个元素，该**元素如果存在则更新顺序**
```redis
127.0.0.1:6379> zrange zset1 0 -1
1) "a"
2) "b"
3) "c"
// 插入同样的元素，会更新其位置
127.0.0.1:6379> zadd zset1 4 "b"
(integer) 0
127.0.0.1:6379> zrange zset1 0 -1
1) "a"
2) "c"
3) "b"
127.0.0.1:6379> zrange zset1 0 -1 withscores
1) "a"
2) "1"
3) "c"
4) "3"
5) "b"
6) "4"
```

- zrem：删除集合中的指定元素：`zrem zset1 "c"`
- zincrby：以指定的值去自动递增或者减少，用法和之前的 incrby 类似
- zrangebyscore：找到指定区间范围的数据进行返回；
- zremrangebyrank：删除 1 到 1 （只删除索引 1）；
- zremrangebyscore：删除指定序号；
- zrank ：返回排序索引，从小到大排序（升序之后再找索引）
顺序号和索引号不一样；
- zrevrank：返回排序索引，从大到小排序（降序排序之后再找索引）；
- zrangebyscore：找到指定区间范围的数据进行返回；`zrangebyscore zset1 2 3 withscores`
- zcard：返回集合中所有元素的个数；
- zcount：返回集合中 score 在给定区间中的数量；
- zremrangebyrank：删除索引 ；`zremrangerank 集合名 从这里  到这里`
- zremrangebyscore：删除指定序号；`zremrangerank 集合名 从这里 到这里`

### （五）Redis 高级命令及特性
- 返回满足所有的键：`keys *`
- 是否存在指定 key：`exists key1`
- 设置某个 key 的过期时间：`expire key名 时间`，可以使用 `ttl key名` 查看剩余的时间；
- 取消某个 key 的过期时间：`persist key名`
- 选择数据库（0-15 个库，默认是 0 库）`select 2`
- 将当前数据中的 key 转移到其他数据库中：`move key值 数据库编号`
- 随机返回数据库中的一个 key ：`randomkey`
- 重命名 key 值 ：`rename 原key值 新key值`
- 查看数据中的 key 数量：`dbsize`
- 获取数据库信息：`info`
- 清空当前数据库：`flushdb`
- 清空所有数据库：`flushall`
- 返回所有配置：`config get *`


## 四、Redis 实现主从复制

==配置主从的时候：默认就绑定了 IP，一定要去掉：bind 127.0.0.1 ==
### （一）主从复制
- Maste可以拥有多个slave
- 多个 slave可以连接同一个 master外,还可以连接到其他的slave
- 主从复制不会阻塞 master，在同步数据时 maste可以继续处理 client请求
- 提供系统的伸缩性

**主从复制过程**
- save与 master建立连接,发送sync同步命令
- maste会开启一个后台进程,将数据库快照保存到文件中,同时 mastel主进程会开始收集新的写命令并缓存
- 后台完成保存后,就将文件发送给slave
- slave将此文件保存到硬盘上


**主从复制配置过程**
这里以主服务器：192.168.238.143 ；从服务器：192.168.238.144 为例；
配置从服务器的 redis.conf ，添加：`slaveof 192.168.238.143 6379`


### （二）哨兵
主要是在 Redis 2.0 中的，用于对主服务器进行监控
- 功能一：监控主数据库和从数据库是否正常运行；
- 功能二：主数据库出现故障时候，可以自动将从数据库转换为主数据库，实现自动切换；

**实现步骤：**
- 修改从服务器的 sentinel.conf 
  - `sentinel monitor GJXAIOU 192.168.238.143 6379 1` # 主数据库名称、IP、端口、投票选举次数
  - `sentinel down-after-milliseconds GJXAIOU 5000`#默认 1 秒检查一次，这里配置超时 5000 毫秒为宕机；
  - `sentinel failover-timeout GJXAIOU 900000`
  - `sentinel parallel-syncs GJXAIOU 2`
  - `sentinel can-failover GJXAIOU yes`
- 启动 sentinel 哨兵
  `src/redis-server src/sentinel.conf --sentinel &`
  
### （三）事务
**事务一般不使用**
使用 `multi`开启事务，使用 `exec` 执行，使用 `discard`取消事务；


### 五、Redis 持久化
**配置是修改：** redis.conf 里面设置
redis是一个**支持持久化的内存数据**库,也就是说 redis需要经常将内存中的数据同步到硬盘来保证持久化。

**redis持久化的两种方式：RDB 方式和 AOF**
- snapshotting (快 照 ）默认方式,将内存中以快照的方式写入到二进制文件中,默认为 `dump.rdb` 可以通过配置设置自动做快照持久化的方式。我们可以配置redis 在n秒内如果超过m个key则修改就自动做快照。
  - snapshotting设置
    - save  900  1  #900秒内如果超过1个key被修改则发起快照保存
    - save  300  10  #300秒内如果超过10个key被修改,则发起快照保存
    - save  60   10000
  - append- only file(缩写aof)的方式(有点类似于 oracle日志)由于快照方式是在一定时间间隔做一次,所以可能发生redis意外dowm的情况就会丢失最后一次快照后的所有修改的数据、aof比快照方式有更好的持久化性,是由于在使用aof时, redis会将每一个收到的写命令都通过 write函数追加到命令中,当redis重新启动时会重新执行文件中保存的写命令来在内存中重建这个数据库的内容,这个文件在bin目录下 `appendonly.aof`。aof不是立即写到硬盘上,可以通过配置文件修改强制写到硬盘中。
    - aof设置:
      - appendonly yes   #启动ao持久化方式有三种修改方式:
      - appendfsync always  #收到写命令就立即写入到磁盘,效率最慢,但是保证完全的持久化；
      3.0 之后使用集群，可以多个主机同时执行写的操作，效率会提升；
      - appendfsync everysec #每秒钟写入磁盘一次,在性能和持久化方面做了很好的折中；
      默认是这种；
      - appendfsync no  #完全依赖os，性能最好，持久化没保证；

## 六、发布和订阅消息

- 使用 `subscribe 频道名` 进行订阅监听
- 使用 `publish 频道名 发布的内容` 进行发布消息广播；


## 七、Redis 集群

**文件目录说明**
Redis
  redis-5.0.0（默认操作：输入命令的文件夹）
    redis-cluster
        1
        2
        3
        4
        5
        6
        7

虚拟机：CentOS7MinClone2 ，IP 地址：192.168.238.147

- 创建上述目录结构（从第三层往下需要手动创建）；
- 将原来的配置文件：redis.conf 分别拷贝到 7 个文件下面
- 修改配置文件
```redis
daemonize yes
port 700* #针对每台机器设置端口号：本实验设置顺序为：6371-6376 代替 7001 - 7006
bind 192.168.238.147 #绑定当前机器的 IP
dir /home/GJXAIOU/Redis/redis-5.0.5/redis-cluster/*（这里是 1- 6）/ # 设置数据文件保存位置
cluster -enabled yes  # 启动集群模式
cluster-config-file nodes700*.conf #这里和端口名字相同，本实验顺序为：6371-6376代替 7001-7006
cluster-node-timeout 5000
appendonly yes
```

- 安装 ruby
```redis
yum install ruby
yum install rubygems

gem sources --remove https://rubygems.org/
gem sources -a https://gems.ruby-china.com/
gem install redis
```
如果报错：`redis requires Ruby version >= 2.3.0.`

```redis
yum install curl
curl -sSL https://rvm.io/mpapis.asc | gpg2 --import -
curl -sSL https://rvm.io/pkuczynski.asc | gpg2 --import -
gpg2 --keyserver hkp://keys.gnupg.net --recv-keys D39DC0E3
curl -L get.rvm.io | bash -s stable
source /usr/local/rvm/scripts/rvm
rvm install 2.6.3
rvm use 2.6.3 --default
rvm remove 2.0.0

gem install redis
```

- 然后分别启动这 6 个实例，并判断是否启动成功；
`src/redis-server redis-cluster/1/redis.conf`，然后将 1 依次换成 2- 6 挨个启动；
`ps -el | grep redis` 查看是否启动成功

- 然后执行 redis-trib.rb 命令
`src/redis-trib.rb create --replicas 1 192.168.238.147:6371 192.168.238.147:6372 192.168.238.147:6373 192.168.238.147:6374 192.168.238.147:6375 192.168.238.147:6376`

![搜狗截图20191108210120](Redis.resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20191108210120.png)
![搜狗截图20191108210205](Redis.resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20191108210205.png)
![集群1](Redis.resource/%E9%9B%86%E7%BE%A41.png)
![集群2](Redis.resource/%E9%9B%86%E7%BE%A42.png)
![集群3](Redis.resource/%E9%9B%86%E7%BE%A43.png)
![集群4](Redis.resource/%E9%9B%86%E7%BE%A44.png)
![111](Redis.resource/111.png)

