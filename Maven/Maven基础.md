# Maven基础

Maven 是一个自动化的构建工具，服务于 Java 平台的项目构建和 jar 包依赖管理；

其并不是一个框架，用于帮助开发者管理框架所需的 jar 包；



### 作用

- 添加第三方 jar 包
使用 Maven 后每个 jar 包本身只在本地仓库中保存一份，需要jar 包的工程只需要以坐标的方式简单的引用一下就可以了。不仅极大的节约了存储空间，更避免了重复文件太多而造成混乱。

- jar 包之间的依赖关系
Maven 可以替我们自动的将当前 jar 包所依赖的其他所有 jar 包全部导入进来，无需人工参与，节约了我们大量的时间和精力。示例：通过 Maven 导入 commons-fileupload-1.3 后 commons-io-2.01 jar 会被自动导入,程序员不必了解这个依赖关系。

- 获取第三方 Jar 包；
从中央仓库可以获取；


### Maven 约定的目录结构

- groupid：一般为公司域名
- artifactid：项目名称
- version：项目版本
- package：创建项目默认的包名

### POM

Project Object Model：项目对象模型
将 Java 工程的相关信息封装为对象作为便于操作和管理的模型，是 maven 的核心配置；
pom.xml 文件存在于项目的根节点下；


### 依赖的范围

使用 `<scope>` 显示 jar 包作用范围，常用值为：compile、test、provided。

- compile：编译依赖范围(默认)，使用此依赖范围对于编译、测试、运行三种 classpath 都有效，即在编译、测试和运行的时候都要使用该依赖 jar 包。

- test：测试依赖范围，此依赖范围只能用于测试 classpath，而在编译和运行项目时无法使用此类依赖，典型的是 Junit，它只用于编译测试代码和运行测试代码的时候才需要;

- provided：此依赖范围，对于编译和测试 classpath 有效,而对运行时无效;


 |过程 | compile | test | provide
 ---|---|----|---
主程序|可以| 否   |可以
测试程序|可以|可以|可以
参与部署，运行阶段|可以|否|否


### 依赖的传递性
A 依赖B,B依赖C,A能否使用C呢?那要看B依赖C的范围是不是 compile，如果是则可用,否则不可用。













