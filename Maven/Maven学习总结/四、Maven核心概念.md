## Maven学习总结(四)——Maven核心概念

## 一、Maven坐标

### 1.1、什么是坐标？

　　在平面几何中坐标（x,y）可以标识平面中唯一的一点。

### 1.2、Maven坐标主要组成

*   groupId：组织标识（包名）
*   artifactId：项目名称
*   version：项目的当前版本
*   packaging：项目的打包方式，最为常见的jar和war两种

样例：

　　![](%E5%9B%9B%E3%80%81Maven%E6%A0%B8%E5%BF%83%E6%A6%82%E5%BF%B5.resource/Mon,%2018%20Nov%202019%20171718.png)

　　![](data:image/png;base64,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)

### 1.3、Maven为什么使用坐标？

*   Maven世界拥有大量构建，我们需要找一个用来唯一标识一个构建的统一规范。
*   拥有了统一规范，就可以把查找工作交给机器。

## 二、依赖管理

### 2.1、依赖配置

　　依赖配置主要包含如下元素：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

 1     <!--添加依赖配置-->
 2     <dependencies>
 3         <!--项目要使用到junit的jar包，所以在这里添加junit的jar包的依赖-->
 4         <dependency>
 5             <groupId>junit</groupId>
 6             <artifactId>junit</artifactId>
 7             <version>4.9</version>
 8             <scope>test</scope>
 9         </dependency>
10         <!--项目要使用到Hello的jar包，所以在这里添加Hello的jar包的依赖-->
11         <dependency>
12             <groupId>me.gacl.maven</groupId>
13             <artifactId>Hello</artifactId>
14             <version>0.0.1-SNAPSHOT</version>
15             <scope>compile</scope>
16         </dependency>    
17     </dependencies>

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0); "复制代码")

### 2.2、依赖范围

　　**依赖范围scope用来控制依赖和编译，测试，运行的classpath的关系**. 主要的是三种依赖关系如下：
　　　　1.**compile**：** 默认编译依赖范围**。对于编译，测试，运行三种classpath都有效
　　　　2.test：测试依赖范围。只对于测试classpath有效
　　　　3.provided：已提供依赖范围。对于编译，测试的classpath都有效，但对于运行无效。因为由容器已经提供，例如servlet-api
　　　　4.runtime:运行时提供。例如:jdbc驱动

### 2.3、传递性依赖

　　MakeFriends.jar直接依赖于HelloFriends.jar，而HelloFriends.jar又直接依赖于Hello.jar，那么MakeFriends.jar也依赖于Hello.jar，这就是传递性依赖，只不过这种依赖是间接依赖，如下图所示：

　　![](%E5%9B%9B%E3%80%81Maven%E6%A0%B8%E5%BF%83%E6%A6%82%E5%BF%B5.resource/Mon,%2018%20Nov%202019%20171725.png)

### 2.4、可选依赖

## 三、仓库管理

### 3.1、Maven仓库

　　用来统一存储所有Maven共享构建的位置就是仓库

### 3.2、Maven仓库布局

　　根据Maven坐标定义每个构建在仓库中唯一存储路径，大致为：groupId/artifactId/version/artifactId-version.packaging

### 3.3、仓库的分类

#### 3.3.1、本地仓库

　　每个用户只有一个本地仓库，默认是在~/.m2/repository/，~代表的是用户目录

#### 3.3.2、远程仓库

　　1、中央仓库：Maven默认的远程仓库，URL地址：http://search.maven.org/

![image-20230210094109848](%E5%9B%9B%E3%80%81Maven%E6%A0%B8%E5%BF%83%E6%A6%82%E5%BF%B5.resource/image-20230210094109848.png)

　　2、私服：是一种特殊的远程仓库，它是架设在局域网内的仓库

![image-20230210094119105](%E5%9B%9B%E3%80%81Maven%E6%A0%B8%E5%BF%83%E6%A6%82%E5%BF%B5.resource/image-20230210094119105.png)

## 四、生命周期

### 4.1、何为生命周期？

　　Maven生命周期就是为了对所有的构建过程进行抽象和统一，包括项目清理，初始化，编译，打包，测试，部署等几乎所有构建步骤

### 4.2、Maven三大生命周期

　　Maven有三套相互独立的生命周期，请注意这里说的是"三套"，而且"相互独立"，这三套生命周期分别是：

1.  Clean Lifecycle 在进行真正的构建之前进行一些清理工作。
2.  Default Lifecycle 构建的核心部分，编译，测试，打包，部署等等。
3.  Site Lifecycle 生成项目报告，站点，发布站点。

　　再次强调一下它们是相互独立的，你可以仅仅调用clean来清理工作目录，仅仅调用site来生成站点。当然你也可以直接运行 mvn clean install site 运行所有这三套生命周期。
　　clean生命周期每套生命周期都由一组阶段(Phase)组成，我们平时在命令行输入的命令总会对应于一个特定的阶段。比如，运行mvn clean ，这个的clean是Clean生命周期的一个阶段。有Clean生命周期，也有clean阶段。Clean生命周期一共包含了三个阶段：

1.  pre-clean 执行一些需要在clean之前完成的工作
2.  clean 移除所有上一次构建生成的文件
3.  post-clean 执行一些需要在clean之后立刻完成的工作

　　"mvn clean" 中的clean就是上面的clean，在一个生命周期中，运行某个阶段的时候，它之前的所有阶段都会被运行，也就是说，"mvn clean"等同于 mvn pre-clean clean ，如果我们运行 mvn post-clean ，那么 pre-clean，clean 都会被运行。这是Maven很重要的一个规则，可以大大简化命令行的输入。
　　Site生命周期pre-site 执行一些需要在生成站点文档之前完成的工作

1.  site 生成项目的站点文档
2.  post-site 执行一些需要在生成站点文档之后完成的工作，并且为部署做准备
3.  site-deploy 将生成的站点文档部署到特定的服务器上

　　这里经常用到的是site阶段和site-deploy阶段，用以生成和发布Maven站点，这可是Maven相当强大的功能，Manager比较喜欢，文档及统计数据自动生成，很好看。
　　Default生命周期Default生命周期是Maven生命周期中最重要的一个，绝大部分工作都发生在这个生命周期中。这里，只解释一些比较重要和常用的阶段：

*   validate
*   generate-sources
*   process-sources
*   generate-resources
*   process-resources 复制并处理资源文件，至目标目录，准备打包。
*   compile 编译项目的源代码。
*   process-classes
*   generate-test-sources
*   process-test-sources
*   generate-test-resources
*   process-test-resources 复制并处理资源文件，至目标测试目录。
*   test-compile 编译测试源代码。
*   process-test-classes
*   test 使用合适的单元测试框架运行测试。这些测试代码不会被打包或部署。
*   prepare-package
*   package 接受编译好的代码，打包成可发布的格式，如 JAR 。
*   pre-integration-test
*   integration-test
*   post-integration-test
*   verify
*   install 将包安装至本地仓库，以让其它项目依赖。
*   deploy 将最终的包复制到远程的仓库，以让其它开发人员与项目共享。

　　运行任何一个阶段的时候，它前面的所有阶段都会被运行，这也就是为什么我们运行mvn install 的时候，代码会被编译，测试，打包。此外，Maven的插件机制是完全依赖Maven的生命周期的，因此理解生命周期至关重要。

## 五、Maven插件

1.  Maven的核心仅仅定义了抽象的生命周期，具体的任务都是交由插件完成的。
2.  每个插件都能实现多个功能，每个功能就是一个插件目标。
3.  Maven的生命周期与插件目标相互绑定，以完成某个具体的构建任务，例如compile就是插件maven-compiler-plugin的一个插件目标。