# [云原生](https://testerhome.com/topics/node176) Elasticsearch 与 Clickhouse 数据存储对比

## 1 背景

京喜达技术部在社区团购场景下采用 JDQ+Flink+Elasticsearch 架构来打造实时数据报表。随着业务的发展 Elasticsearch 开始暴露出一些弊端，不适合大批量的数据查询，高频次分页导出导致宕机、存储成本较高。

Elasticsearch 的查询语句维护成本较高、在聚合计算场景下出现数据不精确等问题。Clickhouse 是列式数据库，列式型数据库天然适合 OLAP 场景，类似 SQL 语法降低开发和学习成本，采用快速压缩算法节省存储成本，采用向量执行引擎技术大幅缩减计算耗时。所以做此对比，进行 Elasticsearch 切换至 Clickhouse 工作。

## 2 OLAP

OLAP 意思是 On-Line Analytical Processing 联机分析处理，Clickhouse 就是典型的 OLAP 联机分析型数据库管理系统 (DBMS)。OLAP 主要针对数据进行复杂分析汇总操作，比如我们业务系统每天都对当天所有运输团单做汇总统计，计算出每个省区的妥投率，这个操作就属于 OLAP 类数据处理。与 OLAP 类似的还有一个 OLTP 类数据处理，意思是 On-Line Transaction Processing 联机事务处理，在 OLTP 场景中用户并发操作量会很大，要求系统实时进行数据操作的响应，需要支持事务，Mysql、Oracle、SQLServer 等都是 OLTP 型数据库。

### 2.1 OLAP 场景的特征

- 宽表，即每个表包含着大量的列
- 对于读取，从数据库中提取相当多的行，但只提取列的一小部分。
- 查询相对较少 (通常每台服务器每秒查询数百次或更少)
- 查询结果明显小于源数据。换句话说，数据经过过滤或聚合，因此结果适合于单个服务器的 RAM 中
- 绝大多数是读请求
- 数据以相当大的批次 (> 1000 行) 更新，而不是单行更新;或者根本没有更新。
- 对于简单查询，允许延迟大约 50 毫秒
- 列中的数据相对较小：数字和短字符串 (例如，每个 URL 60 个字节)
- 处理单个查询时需要高吞吐量 (每台服务器每秒可达数十亿行)
- 事务不是必须的
- 对数据一致性要求低

## 3 特性

### 3.1 Elasticsearch

- **搜索：** 适用倒排索引，每个字段都可被索引且可用于搜索，海量数据下近实时实现秒级的响应，基于 Lucene 的开源搜索引擎，为全文检索、高亮、搜索推荐等提供了检索能力。百度搜索、淘宝商品搜索、日志搜索等
- **数据分析：** Elasticsearch 提供了大量数据分析的 API 和丰富的聚合能力，支持在海量数据的基础上进行数据分析处理。统计订单量、爬虫爬取不同电商的某个商品数据，通过 Elasticsearch 进行数据分析（各个平台的历史价格、购买力等等）

### 3.2 Clickhouse

- 列式存储
- 压缩算法：采用 lz4 和 zstd 算法数据压缩，高压缩比降低数据大小，降低磁盘 IO，降低 CPU 使用率。
- 索引：按照主键对数据进行排序，clickhouse 能在几十毫秒内完成在大量数据对特定数据或范围数据进行查找。
- 多核心并行处理：ClickHouse 会使用服务器上一切可用的资源，来全力完成一次查询。
- 支持 SQL：一种基于 SQL 的声明式查询语言，在许多情况下与 ANSI SQL 标准相同。支持 group by，order by，from, join，in 以及非相关子查询等。
- 向量引擎：为了高效的使用 CPU，数据不仅仅按列存储，同时还按向量 (列的一部分) 进行处理，这样可以更加高效地使用 CPU。
- 实时的数据更新：数据总是已增量的方式有序的存储在 MergeTree 中。数据可以持续不断高效的写入到表中，不会进行任何加锁等操作。写入流量在 50M-200M/s
- 适合在线查询：响应速度快延迟极低
- 丰富的聚合计算函数

## 4 我们的业务场景

1. 大宽表，**读大量行少量列**进行指标聚合计算查询，结果集比较小。数据表都是通过 Flink 加工出来的宽表，列比较多。在对数据进行查询或者分析时，经常选择其中的少数几列作为维度列、对其他少数几列作为指标列，对全表或者一定范围内的数据做聚合计算。这个过程会扫描大量的行数据，但是只用了少数几列。
2. 大量的列表**分页查询**与导出
3. Flink 中数据**大批量追加写入，不做更新**操作
4. 有时某个指标计算需要全表扫描做聚合计算
5. 很少进行全文搜索

**结论：数据报表、数据分析场景是典型的 OLAP 场景，在业务场景上列式存储数据库 Clickhouse 比 Elasticsearch 更有优势，Elasticsearch 在全文搜索上更占优势，但是我们这种全文搜索场景较少。**

## 5 成本

- 学习成本：Clickhouse 是 SQL 语法比 Elasticsearch 的 DSL 更简单，几乎所有后端研发都有 Mysql 开发经验，比较相通学习成本更低。
- 开发、测试、维护成本：Clickhouse 是 SQL 语法，与 Mysql 开发模式相似，更好写单元测试。Elasticsearch 是使用 Java API 拼接查询语句，复杂度较高，不易读不易维护。
- 运维成本：未知，互联网上在日志场景下 Clickhouse 比 Elasticsearch 成本更低。
- 服务器成本：
- Clickhouse 的数据压缩比要高于 Elasticsearch，相同业务数据占用的磁盘空间 ES 占用磁盘空间是 Clickhouse 的 3-10 倍，平均在 6 倍。 见图 1
- Clickhouse 比 ES 占用更少的 CPU 和内存

![img](jingdong_%E4%BA%91%E5%8E%9F%E7%94%9F%20Elasticsearch%20%E4%B8%8E%20Clickhouse%20%E6%95%B0%E6%8D%AE%E5%AD%98%E5%82%A8%E5%AF%B9%E6%AF%94.resource/2bc57ae3502a4784a1b974e7d3471e97tplv-k3u1fbpfcp-zoom-1.jpeg)

**结论：同等数据量情况下，Elasticsearch 使用的存储空间是 Clickhouse 的 3-10 倍，平均在 6 倍。综合学习、开发、测试、维护方面，Clickhouse 比 Elasticsearch 更友好**

## 6 测试

### 6.1 服务器配置

以下均基于下图配置进行测试

![img](jingdong_%E4%BA%91%E5%8E%9F%E7%94%9F%20Elasticsearch%20%E4%B8%8E%20Clickhouse%20%E6%95%B0%E6%8D%AE%E5%AD%98%E5%82%A8%E5%AF%B9%E6%AF%94.resource/2dd7852b285b41f98022263377243d2atplv-k3u1fbpfcp-zoom-1.png)

### 6.2 写入压测

以下基于 wms_order_sku 表，通过 Flink 在业务平稳情况下双写 Elasticsearch 和 Clickhouse1000W+ 数据进行测试得到的结果

- 占用 CPU 情况：Elasticsearch CPU 一直占用很高，Clickhouse 占用很少 CPU。见 图 2

![img](jingdong_%E4%BA%91%E5%8E%9F%E7%94%9F%20Elasticsearch%20%E4%B8%8E%20Clickhouse%20%E6%95%B0%E6%8D%AE%E5%AD%98%E5%82%A8%E5%AF%B9%E6%AF%94.resource/9f2096cf11074c45b4db12f257e49626tplv-k3u1fbpfcp-zoom-1.jpeg)

- 占用内存情况：Elasticsearch 内存升高频繁 GC，Clickhouse 占用内存较低，比较平稳。见 图 3

![img](jingdong_%E4%BA%91%E5%8E%9F%E7%94%9F%20Elasticsearch%20%E4%B8%8E%20Clickhouse%20%E6%95%B0%E6%8D%AE%E5%AD%98%E5%82%A8%E5%AF%B9%E6%AF%94.resource/87c671c5e90845b9a392c34fea46ad26tplv-k3u1fbpfcp-zoom-1.jpeg)

- 写入吞吐量：CH 单机写入速度大约为 50~200MB/s，如果写入的数据每行为 1kb，写入速度为 5-20W/s，图 4(写入吞吐量) 为互联网上 Elasticsearch 与 Clickhouse 写入数据的对比图，同等数据样本的情况下 CH 写入性能是 Elasticsearch 的 5 倍。由于我们目前 Flink 任务为双写，考虑到会互相影响，后续补充压测结果。

![img](jingdong_%E4%BA%91%E5%8E%9F%E7%94%9F%20Elasticsearch%20%E4%B8%8E%20Clickhouse%20%E6%95%B0%E6%8D%AE%E5%AD%98%E5%82%A8%E5%AF%B9%E6%AF%94.resource/c0e25605826e422491fcf29c92414335tplv-k3u1fbpfcp-zoom-1.jpeg)

**结论：批量写入数据时 Elasticsearch 比 Clickhouse 更吃内存和 CPU，Elasticsearch 消耗的内存是 Clickhouse 的 5.3 倍，消耗的 CPU 是 Clickhouse 的 27.5 倍。吞吐量是 Elasticsearch 的 5 倍**

### 6.3 查询性能 (单并发测试)

以下场景是我们数据报表以及数据分析中出现的高频场景，所以基于此进行查询性能测试

![img](jingdong_%E4%BA%91%E5%8E%9F%E7%94%9F%20Elasticsearch%20%E4%B8%8E%20Clickhouse%20%E6%95%B0%E6%8D%AE%E5%AD%98%E5%82%A8%E5%AF%B9%E6%AF%94.resource/d08a7cfb070d4043bb6f3d076f8b5336tplv-k3u1fbpfcp-zoom-1.jpeg)

**数据对比情况**

- Clickhouse 自身在集群配置差一倍的情况下查询性能差异不是很大，CH2(48C 182GB) 比 CH1(80C 320GB) 平均慢 14%。见图 5

![img](jingdong_%E4%BA%91%E5%8E%9F%E7%94%9F%20Elasticsearch%20%E4%B8%8E%20Clickhouse%20%E6%95%B0%E6%8D%AE%E5%AD%98%E5%82%A8%E5%AF%B9%E6%AF%94.resource/f6ab10012e214fd3bee9c703991cc4d4tplv-k3u1fbpfcp-zoom-1.jpeg)

- Elasticsearch 在集群配置差一倍的情况下查询性能受影响较大，ES2(46C 320GB) 比 ES1(78C 576GB) 平均慢 40%。见图 6

![img](jingdong_%E4%BA%91%E5%8E%9F%E7%94%9F%20Elasticsearch%20%E4%B8%8E%20Clickhouse%20%E6%95%B0%E6%8D%AE%E5%AD%98%E5%82%A8%E5%AF%B9%E6%AF%94.resource/ae0ca780b0bf4e04b34b5ee5d0e1bb58tplv-k3u1fbpfcp-zoom-1.jpeg)

- ES2(46C 320GB) 与 CH2(48C 182GB) 相比，ES2 与 CH2 CPU 核数相近，ES2 内存是 CH2 1.75 倍的情况下，CH2 的响应速度是 ES2 的响应速度的的 12.7 倍。见图 7

![img](jingdong_%E4%BA%91%E5%8E%9F%E7%94%9F%20Elasticsearch%20%E4%B8%8E%20Clickhouse%20%E6%95%B0%E6%8D%AE%E5%AD%98%E5%82%A8%E5%AF%B9%E6%AF%94.resource/7fe55069ccfb441ca38ae4c66f1f8ae6tplv-k3u1fbpfcp-zoom-1.jpeg)

**结论：查询数据时 Elasticsearch 比 Clickhouse 慢，在配置相近的情况下 Clickhouse 的响应速度是 Elasticsearch 的 12.7 倍，特别是基于时间的多字段进行聚合查询是 Clickhouse 比 Elasticsearch 快 32 倍。Clickhouse 的查询响应素速度受集群配置大小的影响较小。**

### 6.4 查询压测 (高并发测试，数据来源于互联网)

由于准备高并发测试比较复杂耗时多，后续会基于我们的业务数据以及业务场景进行查询压力测试。以下数据来源于互联网在用户画像场景（数据量 262933269）下进行的测试，与我们的场景非常类似。

![img](jingdong_%E4%BA%91%E5%8E%9F%E7%94%9F%20Elasticsearch%20%E4%B8%8E%20Clickhouse%20%E6%95%B0%E6%8D%AE%E5%AD%98%E5%82%A8%E5%AF%B9%E6%AF%94.resource/d176ec990490426cb5054e3340663696tplv-k3u1fbpfcp-zoom-1.png)

![img](jingdong_%E4%BA%91%E5%8E%9F%E7%94%9F%20Elasticsearch%20%E4%B8%8E%20Clickhouse%20%E6%95%B0%E6%8D%AE%E5%AD%98%E5%82%A8%E5%AF%B9%E6%AF%94.resource/c52ecf5e89194ce59cb27197dad535a7tplv-k3u1fbpfcp-zoom-1.png)

结论：Clickhouse 对于高并发支持的不够，官方建议最大 QPS 为 100。高并发情况下吞吐量不如 Elasticsearch 更友好

## 7 总结

Clickhouse 与 Elasticsearch 对比 Clickhouse 的优缺点。

**优点:**

- 硬件资源成本更低，同等场景下，Clickhouse 占用的资源更小。
- 人力成本更低，新人学习、开发单测以及测试方面都更加友好，更容易介入。
- OLAP 场景下 Clickhouse 比 Elasticsearch 更适合，聚合计算比 Elasticsearch 更精缺、更快，更节省服务器计算资源。
- 写入性能更高，同等情况下是 Elasticsearch 的 5 倍，写入时消耗的服务器资源更小。
- Elasticsearch 在大量导出情况下频繁 GC，严重可能导致宕机，不如 Clickhouse 稳定。
- 查询性能平均是 Elasticsearch 的 12.7 倍，Clickhouse 的查询性能受服务器配置影响较小
- 月服务器消费相同情况 Clickhouse 能够得到更好的性能。

**缺点：**

- 在全文搜索上不如 Elasticsearch 支持的更好，在高并发查询上支持的不如 Elasticsearch 支持的更好