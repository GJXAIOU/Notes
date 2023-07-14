# Spring Boot 与检索

## 一、检索

我们的应用经常需要添加检索功能，开源的 ElasticSearch 是目前**全文搜索引擎**的首选。他可以快速的存储、搜索和分析海量数据。Spring Boot 通过整合 Spring Data ElasticSearch 为我们提供了非常便捷的检索功能支持；
Elasticsearch 是一个分布式搜索服务，提供 Restful API，底层基于 Lucene，采用多 shard（分片）的方式保证数据安全，并且提供自动 resharding 的功能，github 等大型的站点也是采用了 ElasticSearch 作为其搜索服务，

## 二、概念

- 以员工文档的形式存储为例：一个文档代表一个员工数据。**存储数据到 ElasticSearch 的行为叫做索引**，但在索引一个文档之前，需要确定将文档存储在哪里。
- 一个 ElasticSearch 集群可以包含多个索引，相应的每个索引可以包含多个类型。这些不同的类型存储着多个文档，每个文档又有多个属性。
- 和数据库类似关系：
    - 索引   ->   数据库
    - 类型   ->   表
    - 文档   ->   表中的记录
    - 属性   ->     列

![image-20201114123644778](FrameDay06_11%20SpringBoot%E4%B8%8E%E6%A3%80%E7%B4%A2.resource/image-20201114123644778.png)

**官方文档**中的[测试程序：](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_indexing_employee_documents.html)

通过 Postman 向 ES 中发送数据：

![image-20201114133504608](FrameDay06_11%20SpringBoot%E4%B8%8E%E6%A3%80%E7%B4%A2.resource/image-20201114133504608.png)

[检索、删除、检查数据详见](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_retrieving_a_document.html)

## 三、整合 ElasticSearch 测试

- 步骤一：虚拟机安装：

    `docker pull elasticsearch:7.9.3` 注意：这个镜像不支持 Lastest

    因为默认 ES 初始化占用堆内存为 2G，所以一般在启动的时候指定占用内存大小。

    `docker run -e ES_JAVA_OPTS="-Xms256m -Xmx256m" -d -p 9200:9200 -p 9300:9300 --name ES01 镜像ID`

    检查安装是否正确：浏览器输入 `IP:9200` 会出现一段 JSON 数据。

- 引入 `spring-boot-starter-data-elasticsearch`

    ```xml
    <!--SpringBoot默认使用SpringData ElasticSearch模块进行操作-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
    </dependency>
    ```

    

#### SpringBoot 默认支持两种技术来和 ES 交互；

导入相应依赖之后，可以在 Jar 包：`Maven: org.springframework.boot:spring-boot-autoconfigure:1.5.12.RELEASE`中看到两个自动配置类

![image-20201114135545767](FrameDay06_11%20SpringBoot%E4%B8%8E%E6%A3%80%E7%B4%A2.resource/image-20201114135545767.png)

- Jest（默认不生效）
     如果要生效，需要导入 jest 的工具包（io.searchbox.client.JestClient）

     ```xml
     <!-- https://mvnrepository.com/artifact/io.searchbox/jest -->
     <dependency>
         <groupId>io.searchbox</groupId>
         <artifactId>jest</artifactId>
         <version>5.3.3</version>
     </dependency>
     ```
     
- SpringData ElasticSearch
    注意：ES 版本和 SpringBoot 版本有可能不合适，可以选择升级 SpringBoot 版本或者安装对应版本的 ES。
    
    |                  Spring Data Release Train                   |                  Spring Data Elasticsearch                   | Elasticsearch |                         Spring Boot                          |
    | :----------------------------------------------------------: | :----------------------------------------------------------: | :-----------: | :----------------------------------------------------------: |
| 2020.0.0[[1](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_1)] | 4.1.x[[1](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_1)] |     7.9.3     | 2.3.x[[1](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_1)] |
    |                           Neumann                            |                            4.0.x                             |     7.6.2     |                            2.3.x                             |
    |                            Moore                             |                            3.2.x                             |    6.8.12     |                            2.2.x                             |
    |                           Lovelace                           |                            3.1.x                             |     6.2.2     |                            2.1.x                             |
    | Kay[[2](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_2)] | 3.0.x[[2](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_2)] |     5.5.0     | 2.0.x[[2](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_2)] |
    | Ingalls[[2](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_2)] | 2.1.x[[2](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_2)] |     2.4.0     | 1.5.x[[2](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_2)] |
    
    1）、Client 节点信息clusterNodes；clusterName
    2）、提供了 ElasticsearchTemplate 操作 es
    3）、需要编写一个提供的 ElasticsearchRepository 的子接口来操作ES；

### 使用 Jest

- 步骤一：导入 JestClient 依赖

- 步骤二：在 `application.properties` 配置相关属性

    ```properties
    spring.elasticsearch.jest.uris = http://192.168.238.151:9200
    ```

- 步骤三：测试

    ```java
    package com.atguigu.elastic;
    
    import com.atguigu.elastic.bean.Article;
    import com.atguigu.elastic.bean.Book;
    import com.atguigu.elastic.repository.BookRepository;
    import io.searchbox.client.JestClient;
    import io.searchbox.core.Index;
    import io.searchbox.core.Search;
    import io.searchbox.core.SearchResult;
    import org.junit.Test;
    import org.junit.runner.RunWith;
    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.boot.test.context.SpringBootTest;
    import org.springframework.test.context.junit4.SpringRunner;
    
    import java.io.IOException;
    import java.util.List;
    
    @RunWith(SpringRunner.class)
    @SpringBootTest
    public class Springboot03ElasticApplicationTests {
    
        @Autowired
        JestClient jestClient;
        
        /**
         * 测试 Jest 方式
         */
        @Test
        public void contextLoads() {
            //1、给Es中索引（保存）一个文档；
            Article article = new Article();
            article.setId(1);
            article.setTitle("好消息");
            article.setAuthor("zhangsan");
            article.setContent("Hello World");
    
            //构建一个索引功能
            Index index = new Index.Builder(article).index("atguigu").type("news").build();
            try {
                //执行
                jestClient.execute(index);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    
        //测试搜索
        @Test
        public void search() {
    
            //使用查询表达式
            String json = "{\n" +
                    "    \"query\" : {\n" +
                    "        \"match\" : {\n" +
                    "            \"content\" : \"hello\"\n" +
                    "        }\n" +
                    "    }\n" +
                    "}";
    
            //更多操作：https://github.com/searchbox-io/Jest/tree/master/jest
    
            //构建搜索功能
            Search search = new Search.Builder(json).addIndex("atguigu").addType("news").build();
    
            //执行
            try {
                SearchResult result = jestClient.execute(search);
                System.out.println(result.getJsonString());
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    
    
        @Autowired
        BookRepository bookRepository;
    
        /**
         * 测试 Spring Data 的自定义 ElasticsearchRepository 方式
         */
        @Test
        public void test02() {
            // 插入测试
            Book book = new Book();
            book.setId(1);
            book.setBookName("西游记");
            book.setAuthor("吴承恩");
            bookRepository.index(book);
    
            // 查找
            for (Book book1 : bookRepository.findByBookNameLike("游")) {
                System.out.println(book1);
            }
        }
    
    }
    ```
    
    

### 使用 Spring Data

- 步骤一：导入依赖

- 步骤二：在 `application.properties` 配置相关属性

    ```properties
    spring.data.elasticsearch.cluster-name=elasticsearch
    spring.data.elasticsearch.cluster-nodes=118.24.44.169:9300
    ```

使用方式：

- 方式一：自定义 `ElasticsearchRepository`

    ```java
    package com.atguigu.elastic.repository;
    
    import com.atguigu.elastic.bean.Book;
    import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;
    
    import java.util.List;
    
    public interface BookRepository extends ElasticsearchRepository<Book, Integer> {
        // 参照 https://docs.spring.io/spring-data/elasticsearch/docs/3.0.6.RELEASE/reference/html/
        public List<Book> findByBookNameLike(String bookName);
    }
    
    ```

    同时针对存储的类型上面要标注一下存储的索引名称和类型

    ```java
    package com.atguigu.elastic.bean;
    
    import org.springframework.data.elasticsearch.annotations.Document;
    
    @Document(indexName = "atguigu", type = "book")
    public class Book {
        private Integer id;
        private String bookName;
        private String author;
        
        // 省略 Getter、Setter 和 toString() 方法
    }
    ```

    测试程序见测试类的 `test02()`

- 方式二：ElasticsearchTemplate





