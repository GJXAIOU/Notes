# 33Kafka认证机制用哪家？

你好，我是胡夕。今天我要和你分享的主题是：Kafka 的认证机制。

## 什么是认证机制？

所谓认证，又称“验证”“鉴权”，英文是 authentication，是指通过一定的手段，完成对用户身份的确认。认证的主要目的是确认当前声称为某种身份的用户确实是所声称的用户。

在计算机领域，经常和认证搞混的一个术语就是授权，英文是 authorization。授权一般是指对信息安全或计算机安全相关的资源定义与授予相应的访问权限。

举个简单的例子来区分下两者：认证要解决的是你要证明你是谁的问题，授权要解决的则是你能做什么的问题。

在 Kafka 中，认证和授权是两套独立的安全配置。我们今天主要讨论 Kafka 的认证机制，在专栏的下一讲内容中，我们将讨论授权机制。

## Kafka 认证机制

自 0.9.0.0 版本开始，Kafka 正式引入了认证机制，用于实现基础的安全用户认证，这是将 Kafka 上云或进行多租户管理的必要步骤。截止到当前最新的 2.3 版本，Kafka 支持基于 SSL 和基于 SASL 的安全认证机制。

**基于 SSL 的认证主要是指 Broker 和客户端的双路认证**（2-way authentication）。通常来说，SSL 加密（Encryption）已经启用了单向认证，即客户端认证 Broker 的证书（Certificate）。如果要做 SSL 认证，那么我们要启用双路认证，也就是说 Broker 也要认证客户端的证书。

对了，你可能会说，SSL 不是已经过时了吗？现在都叫 TLS（Transport Layer Security）了吧？但是，Kafka 的源码中依然是使用 SSL 而不是 TLS 来表示这类东西的。不过，今天出现的所有 SSL 字眼，你都可以认为它们是和 TLS 等价的。

Kafka 还支持通过 SASL 做客户端认证。**SASL 是提供认证和数据安全服务的框架**。Kafka 支持的 SASL 机制有 5 种，它们分别是在不同版本中被引入的，你需要根据你自己使用的 Kafka 版本，来选择该版本所支持的认证机制。

1. GSSAPI：也就是 Kerberos 使用的安全接口，是在 0.9 版本中被引入的。
2. PLAIN：是使用简单的用户名 / 密码认证的机制，在 0.10 版本中被引入。
3. SCRAM：主要用于解决 PLAIN 机制安全问题的新机制，是在 0.10.2 版本中被引入的。
4. OAUTHBEARER：是基于 OAuth 2 认证框架的新机制，在 2.0 版本中被引进。
5. Delegation Token：补充现有 SASL 机制的轻量级认证机制，是在 1.1.0 版本被引入的。

## 认证机制的比较

Kafka 为我们提供了这么多种认证机制，在实际使用过程中，我们应该如何选择合适的认证框架呢？下面我们就来比较一下。

目前来看，使用 SSL 做信道加密的情况更多一些，但使用 SSL 实现认证不如使用 SASL。毕竟，SASL 能够支持你选择不同的实现机制，如 GSSAPI、SCRAM、PLAIN 等。因此，我的建议是**你可以使用 SSL 来做通信加密，使用 SASL 来做 Kafka 的认证实现**。

SASL 下又细分了很多种认证机制，我们应该如何选择呢？

SASL/GSSAPI 主要是给 Kerberos 使用的。如果你的公司已经做了 Kerberos 认证（比如使用 Active Directory），那么使用 GSSAPI 是最方便的了。因为你不需要额外地搭建 Kerberos，只要让你们的 Kerberos 管理员给每个 Broker 和要访问 Kafka 集群的操作系统用户申请 principal 就好了。总之，**GSSAPI 适用于本身已经做了 Kerberos 认证的场景，这样的话，SASL/GSSAPI 可以实现无缝集成**。

而 SASL/PLAIN，就像前面说到的，它是一个简单的用户名 / 密码认证机制，通常与 SSL 加密搭配使用。注意，这里的 PLAIN 和 PLAINTEXT 是两回事。**PLAIN 在这里是一种认证机制，而 PLAINTEXT 说的是未使用 SSL 时的明文传输**。对于一些小公司而言，搭建公司级的 Kerberos 可能并没有什么必要，他们的用户系统也不复杂，特别是访问 Kafka 集群的用户可能不是很多。对于 SASL/PLAIN 而言，这就是一个非常合适的应用场景。**总体来说，SASL/PLAIN 的配置和运维成本相对较小，适合于小型公司中的 Kafka 集群**。

但是，SASL/PLAIN 有这样一个弊端：它不能动态地增减认证用户，你必须重启 Kafka 集群才能令变更生效。为什么呢？这是因为所有认证用户信息全部保存在静态文件中，所以只能重启 Broker，才能重新加载变更后的静态文件。

我们知道，重启集群在很多场景下都是令人不爽的，即使是轮替式升级（Rolling Upgrade）。SASL/SCRAM 就解决了这样的问题。它通过将认证用户信息保存在 ZooKeeper 的方式，避免了动态修改需要重启 Broker 的弊端。在实际使用过程中，你可以使用 Kafka 提供的命令动态地创建和删除用户，无需重启整个集群。因此，**如果你打算使用 SASL/PLAIN，不妨改用 SASL/SCRAM 试试。不过要注意的是，后者是 0.10.2 版本引入的。你至少要升级到这个版本后才能使用**。

SASL/OAUTHBEARER 是 2.0 版本引入的新认证机制，主要是为了实现与 OAuth 2 框架的集成。OAuth 是一个开发标准，允许用户授权第三方应用访问该用户在某网站上的资源，而无需将用户名和密码提供给第三方应用。Kafka 不提倡单纯使用 OAUTHBEARER，因为它生成的不安全的 JSON Web Token，必须配以 SSL 加密才能用在生产环境中。当然，鉴于它是 2.0 版本才推出来的，而且目前没有太多的实际使用案例，我们可以先观望一段时间，再酌情将其应用于生产环境中。

Delegation Token 是在 1.1 版本引入的，它是一种轻量级的认证机制，主要目的是补充现有的 SASL 或 SSL 认证。如果要使用 Delegation Token，你需要先配置好 SASL 认证，然后再利用 Kafka 提供的 API 去获取对应的 Delegation Token。这样，Broker 和客户端在做认证的时候，可以直接使用这个 token，不用每次都去 KDC 获取对应的 ticket（Kerberos 认证）或传输 Keystore 文件（SSL 认证）。

为了方便你更好地理解和记忆，我把这些认证机制汇总在下面的表格里了。你可以对照着表格，进行一下区分。

![image-20220812224632523](33Kafka%E8%AE%A4%E8%AF%81%E6%9C%BA%E5%88%B6%E7%94%A8%E5%93%AA%E5%AE%B6%EF%BC%9F.resource/image-20220812224632523.png)

## SASL/SCRAM-SHA-256 配置实例

接下来，我给出 SASL/SCRAM 的一个配置实例，来说明一下如何在 Kafka 集群中开启认证。其他认证机制的设置方法也是类似的，比如它们都涉及认证用户的创建、Broker 端以及 Client 端特定参数的配置等。

我的测试环境是本地 Mac 上的两个 Broker 组成的 Kafka 集群，连接端口分别是 9092 和 9093。

### 第 1 步：创建用户

配置 SASL/SCRAM 的第一步，是创建能否连接 Kafka 集群的用户。在本次测试中，我会创建 3 个用户，分别是 admin 用户、writer 用户和 reader 用户。admin 用户用于实现 Broker 间通信，writer 用户用于生产消息，reader 用户用于消费消息。

我们使用下面这 3 条命令，分别来创建它们。

```
$ cd kafka_2.12-2.3.0/
$ bin/kafka-configs.sh --zookeeper localhost:2181 --alter --add-config 'SCRAM-SHA-256=[password=admin],SCRAM-SHA-512=[password=admin]' --entity-type users --entity-name admin
Completed Updating config for entity: user-principal 'admin'.
$ bin/kafka-configs.sh --zookeeper localhost:2181 --alter --add-config 'SCRAM-SHA-256=[password=writer],SCRAM-SHA-512=[password=writer]' --entity-type users --entity-name writer
Completed Updating config for entity: user-principal 'writer'.
$ bin/kafka-configs.sh --zookeeper localhost:2181 --alter --add-config 'SCRAM-SHA-256=[password=reader],SCRAM-SHA-512=[password=reader]' --entity-type users --entity-name reader
Completed Updating config for entity: user-principal 'reader'.
```

在专栏前面，我们提到过，kafka-configs 脚本是用来设置主题级别参数的。其实，它的功能还有很多。比如在这个例子中，我们使用它来创建 SASL/SCRAM 认证中的用户信息。我们可以使用下列命令来查看刚才创建的用户数据。

```
$ bin/kafka-configs.sh --zookeeper localhost:2181 --describe --entity-type users  --entity-name writer
Configs for user-principal 'writer' are SCRAM-SHA-512=salt=MWt6OGplZHF6YnF5bmEyam9jamRwdWlqZWQ=,stored_key=hR7+vgeCEz61OmnMezsqKQkJwMCAoTTxw2jftYiXCHxDfaaQU7+9/dYBq8bFuTio832mTHk89B4Yh9frj/ampw==,server_key=C0k6J+9/InYRohogXb3HOlG7s84EXAs/iw0jGOnnQAt4jxQODRzeGxNm+18HZFyPn7qF9JmAqgtcU7hgA74zfA==,iterations=4096,SCRAM-SHA-256=salt=MWV0cDFtbXY5Nm5icWloajdnbjljZ3JqeGs=,stored_key=sKjmeZe4sXTAnUTL1CQC7DkMtC+mqKtRY0heEHvRyPk=,server_key=kW7CC3PBj+JRGtCOtIbAMefL8aiL8ZrUgF5tfomsWVA=,iterations=4096
```

这段命令包含了 writer 用户加密算法 SCRAM-SHA-256 以及 SCRAM-SHA-512 对应的盐值 (Salt)、ServerKey 和 StoreKey。这些都是 SCRAM 机制的术语，我们不需要了解它们的含义，因为它们并不影响我们接下来的配置。

### 第 2 步：创建 JAAS 文件

配置了用户之后，我们需要为每个 Broker 创建一个对应的 JAAS 文件。因为本例中的两个 Broker 实例是在一台机器上，所以我只创建了一份 JAAS 文件。但是你要切记，在实际场景中，你需要为每台单独的物理 Broker 机器都创建一份 JAAS 文件。

JAAS 的文件内容如下：

```
KafkaServer {
org.apache.kafka.common.security.scram.ScramLoginModule required
username="admin"
password="admin";
};
```

关于这个文件内容，你需要注意以下两点：

- 不要忘记最后一行和倒数第二行结尾处的分号；
- JAAS 文件中不需要任何空格键。

这里，我们使用 admin 用户实现 Broker 之间的通信。接下来，我们来配置 Broker 的 server.properties 文件，下面这些内容，是需要单独配置的：

```
sasl.enabled.mechanisms=SCRAM-SHA-256
复制代码
sasl.mechanism.inter.broker.protocol=SCRAM-SHA-256
复制代码
security.inter.broker.protocol=SASL_PLAINTEXT
复制代码
listeners=SASL_PLAINTEXT://localhost:9092
复制代码
```

第 1 项内容表明开启 SCRAM 认证机制，并启用 SHA-256 算法；第 2 项的意思是为 Broker 间通信也开启 SCRAM 认证，同样使用 SHA-256 算法；第 3 项表示 Broker 间通信不配置 SSL，本例中我们不演示 SSL 的配置；最后 1 项是设置 listeners 使用 SASL_PLAINTEXT，依然是不使用 SSL。

另一台 Broker 的配置基本和它类似，只是要使用不同的端口，在这个例子中，端口是 9093。

### 第 3 步：启动 Broker

现在我们分别启动这两个 Broker。在启动时，你需要指定 JAAS 文件的位置，如下所示：

```
$KAFKA_OPTS=-Djava.security.auth.login.config=<your_path>/kafka-broker.jaas bin/kafka-server-start.sh config/server1.properties
......
[2019-07-02 13:30:34,822] INFO Kafka commitId: fc1aaa116b661c8a (org.apache.kafka.common.utils.AppInfoParser)
[2019-07-02 13:30:34,822] INFO Kafka startTimeMs: 1562045434820 (org.apache.kafka.common.utils.AppInfoParser)
[2019-07-02 13:30:34,823] INFO [KafkaServer id=0] started (kafka.server.KafkaServer)
$KAFKA_OPTS=-Djava.security.auth.login.config=<your_path>/kafka-broker.jaas bin/kafka-server-start.sh config/server2.properties
......
[2019-07-02 13:32:31,976] INFO Kafka commitId: fc1aaa116b661c8a (org.apache.kafka.common.utils.AppInfoParser)
[2019-07-02 13:32:31,976] INFO Kafka startTimeMs: 1562045551973 (org.apache.kafka.common.utils.AppInfoParser)
[2019-07-02 13:32:31,978] INFO [KafkaServer id=1] started (kafka.server.KafkaServer)
```

此时，两台 Broker 都已经成功启动了。

### 第 4 步：发送消息

在创建好测试主题之后，我们使用 kafka-console-producer 脚本来尝试发送消息。由于启用了认证，客户端需要做一些相应的配置。我们创建一个名为 producer.conf 的配置文件，内容如下：

```
security.protocol=SASL_PLAINTEXT
sasl.mechanism=SCRAM-SHA-256
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="writer" password="writer";
```

之后运行 Console Producer 程序：

```
$ bin/kafka-console-producer.sh --broker-list localhost:9092,localhost:9093 --topic test  --producer.config <your_path>/producer.conf
>hello, world
>   
```

可以看到，Console Producer 程序发送消息成功。

### 第 5 步：消费消息

接下来，我们使用 Console Consumer 程序来消费一下刚刚生产的消息。同样地，我们需要为 kafka-console-consumer 脚本创建一个名为 consumer.conf 的脚本，内容如下：

```
security.protocol=SASL_PLAINTEXT
sasl.mechanism=SCRAM-SHA-256
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="reader" password="reader";
```

之后运行 Console Consumer 程序：

```
$ bin/kafka-console-consumer.sh --bootstrap-server localhost:9092,localhost:9093 --topic test --from-beginning --consumer.config <your_path>/consumer.conf 
hello, world
```

很显然，我们是可以正常消费的。

### 第 6 步：动态增减用户

最后，我们来演示 SASL/SCRAM 动态增减用户的场景。假设我删除了 writer 用户，同时又添加了一个新用户：new_writer，那么，我们需要执行的命令如下：

```
$ bin/kafka-configs.sh --zookeeper localhost:2181 --alter --delete-config 'SCRAM-SHA-256' --entity-type users --entity-name writer
Completed Updating config for entity: user-principal 'writer'.
 
$ bin/kafka-configs.sh --zookeeper localhost:2181 --alter --delete-config 'SCRAM-SHA-512' --entity-type users --entity-name writer
Completed Updating config for entity: user-principal 'writer'.
 
$ bin/kafka-configs.sh --zookeeper localhost:2181 --alter --add-config 'SCRAM-SHA-256=[iterations=8192,password=new_writer]' --entity-type users --entity-name new_writer
Completed Updating config for entity: user-principal 'new_writer'.
```

现在，我们依然使用刚才的 producer.conf 来验证，以确认 Console Producer 程序不能发送消息。

```
$ bin/kafka-console-producer.sh --broker-list localhost:9092,localhost:9093 --topic test  --producer.config /Users/huxi/testenv/producer.conf
>[2019-07-02 13:54:29,695] ERROR [Producer clientId=console-producer] Connection to node -1 (localhost/127.0.0.1:9092) failed authentication due to: Authentication failed during authentication due to invalid credentials with SASL mechanism SCRAM-SHA-256 (org.apache.kafka.clients.NetworkClient)
......
```

很显然，此时 Console Producer 已经不能发送消息了。因为它使用的 producer.conf 文件指定的是已经被删除的 writer 用户。如果我们修改 producer.conf 的内容，改为指定新创建的 new_writer 用户，结果如下：

```
$ bin/kafka-console-producer.sh --broker-list localhost:9092,localhost:9093 --topic test  --producer.config <your_path>/producer.conf
>Good!  
```

现在，Console Producer 可以正常发送消息了。

这个过程完整地展示了 SASL/SCRAM 是如何在不重启 Broker 的情况下增减用户的。

至此，SASL/SCRAM 配置就完成了。在专栏下一讲中，我会详细介绍一下如何赋予 writer 和 reader 用户不同的权限。

## 小结

好了，我们来小结一下。今天，我们讨论了 Kafka 目前提供的几种认证机制，我给出了它们各自的优劣势以及推荐使用建议。其实，在真实的使用场景中，认证和授权往往是结合在一起使用的。在专栏下一讲中，我会详细向你介绍 Kafka 的授权机制，即 ACL 机制，敬请期待。

![image-20220812224710915](33Kafka%E8%AE%A4%E8%AF%81%E6%9C%BA%E5%88%B6%E7%94%A8%E5%93%AA%E5%AE%B6%EF%BC%9F.resource/image-20220812224710915.png)

## 开放讨论

请谈一谈你的 Kafka 集群上的用户认证机制，并分享一个你遇到过的“坑”。

欢迎写下你的思考和答案，我们一起讨论。如果你觉得有所收获，也欢迎把文章分享给你的朋友。

## 1716143665 拼课微信(5)

- 

  lmtoo

  2019-08-17

  我跟着试验做了一下，发现一个漏洞，就是原有的 writer 生产者只要是没有停，即使是删除了 writer 的配置，原有的 writer 还是能发消息，而且 consumer 还能收到

  展开**

  **

  **3

- 

  lmtoo

  2019-08-17

  No JAAS configuration section named 'Client' was found in specified JAAS configuration file: '/usr/local/kafka/config/kafka-broker.jaas'. Will continue connection to Zookeeper server without SASL authentication, if Zookeeper server allows it.

  展开**

  **1

  **1

- 

  明翼

  2019-08-17

  老师说的这 SCRAM 认证用户名和密码直接保存在 zookeeper 上的，如果 zookeeper 不做安全控制，岂不是失去意义了？目前我们没有做认证的，研究过一段时间的 ssl 认证，很麻烦，还影响性能

  作者回复: 不是明文保存的。当然做 ZooKeeper 的认证也是很有必要的

  **

  **1

- 

  Geek_edc612

  2019-08-19

  （1）之前做 kafak/ Sasl-Plain 认证，几经转折才发现，这个认证用户跟 linux 用户名没关系，而且不能动态添加减少用户，最重要的是租户可以自己修改 acl 权限，目前也只是把客户端的 kafka-topics.sh 给禁用了，一叶障目吧，=。=；
  （2）还有就是 sasl-plain 这个 acl 权限感觉肯定，明明给认证用户 a 赋予了所有 topic 的在所有 host 的读写权限，但重启时发现有部分 topic 突然无法消费写入了，提示没权限，再重启就好了；
  （3）接（2）情况，还有就是用 kafka-acls.sh 去查看 topic 的所有 acl 权限时，有的 acl 完全为空，但是用户 a 还能写入消费数据，这块完全不懂
  （4）目前 kafa-acls.sh 只是用的基础的 Write 和 Read 权限，像 Cluster 这个权限不知道干啥用的，其他的了解也不深入
  （5）最后就是做 kafka sasl plain 认证的时候给 zk 也加了认证，具体如下：
  zkserver.sh 加入这个
  "-Djava.security.auth.login.config=/opt/beh/core/zookeeper/conf/kafka_zoo_jaas.conf" \
  zoo.cfg 加入这个：
  authProvider.1=org.apache.zookeeper.server.auth.SASLAuthenticationProvider
  requireClientAuthScheme=sasl
  jaasLoginRenew=3600000
  但是有点疑惑的就是不知道 zk 这个认证是用在那块的？我发现加不加 kafka sasl plain 都能正常用

  展开**

  **

  **

- 

  玉剑冰锋

  2019-08-19

  胡老师，kafka 平滑升级后面会讲吗？

  展开**

  作者回复: 可能涉及不是很多。有什么问题只管问吧

  