# 期中测试 \| 这些Redis源码知识，你都掌握了吗？

作者: 蒋德钧

完成时间:

总结时间:

![](<https://static001.geekbang.org/resource/image/2b/84/2bcc961a69d4c7980956b8b157386884.jpg>)

<audio><source src="https://static001.geekbang.org/resource/audio/25/f2/2590e51444cf27171f106971152d77f2.mp3" type="audio/mpeg"></audio>

你好，我是蒋德钧。

时间过得真快，从 7 月 26 日上线到现在，我们已经走过了一个半月的学习之旅，不知道你的收获如何呢？

前面我其实也说过，阅读和学习 Redis 源码确实是一个比较烧脑的任务，需要你多花些时间钻研。而从我的经验来看，阶段性的验证和总结是非常重要的。所以在这里，我特别设置了期中考试周，从 9 月 13 日开始到 9 月 19 日结束，这期间我们会暂停更新正文内容，你可以好好利用这一周的时间，去回顾一下前 20 讲的知识，做一个巩固。

有标准才有追求，有追求才有动力，有动力才有进步。一起来挑战一下吧，开启你的期中考试之旅。

我给你出了一套测试题，包括一套选择题和一套问答题。

- 选择题：满分共 100 分，包含 4 道单选题和 6 道多选题。提交试卷之后，系统会自动评分。
- 问答题：包括 2 道题目，不计入分数，但我希望你能认真回答这些问题，可以把你的答案写在留言区。在 9 月 16 日这一天，我会公布答案。

<!-- -->

### 选择题

[![](<https://static001.geekbang.org/resource/image/28/a4/28d1be62669b4f3cc01c36466bf811a4.png>)](<http://time.geekbang.org/quiz/intro?act_id=926&exam_id=2699>)

### 问答题

**第一题**

Redis 源码中实现的哈希表在 rehash 时，会调用 dictRehash 函数。dictRehash 函数的原型如下，它的参数 n 表示本次 rehash 要搬移 n 个哈希桶（bucket）中的数据。假设 dictRehash 被调用，并且 n 的传入值为 10。但是，在 dictRehash 查找的 10 个 bucket 中，前 5 个 bucket 有数据，而后 5 个 bucket 没有数据，那么，本次调用 dictRehash 是否就只搬移了前 5 个 bucket 中的数据？

<!-- [[[read_end]]] -->

```
int dictRehash(dict *d, int n)
```

**第二题**

Redis 的事件驱动框架是基于操作系统 IO 多路复用机制进行了封装，以 Linux 的 epoll 机制为例，该机制调用 epoll\_create 函数创建 epoll 实例，再调用 epoll\_ctl 将监听的套接字加入监听列表，最后调用 epoll\_wait 获取就绪的套接字再进行处理。请简述 Redis 事件驱动框架中哪些函数和 epoll 机制各主要函数有对应的调用关系。

好了，这节课就到这里。希望你能抓住期中周的机会，查漏补缺，快速提升 Redis 源码的阅读和学习能力。我们下节课再见！

