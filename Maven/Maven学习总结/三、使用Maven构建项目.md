# 使用Maven构建项目

maven作为一个高度自动化构建工具，本身提供了构建项目的功能，下面就来体验一下使用maven构建项目的过程。

## 一、构建Java项目

### 1.1、创建Java Project

　　1、使用 `mvn archetype:generate`命令，如下所示：

```shell
mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=myapp -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

　　2、使用mvn archetype:create命令，如下所示：

```shell
mvn archetype:create -DgroupId=com.mycompany.app -DartifactId=myapp -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

　　使用"**mvn archetype:generate**"命令和"**mvn archetype:create**"都可以创建项目，目前没有发现这两者的区别，唯一区别的地方就是发现使用"**mvn archetype:generate**"命令创建项目时要特别长的时间才能够将项目创建好，而使用"**mvn archetype:create**"命令则可以很快将项目创建出来。

　　使用"**mvn archetype:create**"命令创建一个java项目的过程如下图所示：

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171322.png)

　　BUILD SUCCESS就表示项目构建成功，当在前用户目录下（即C:\Documents and Settings\Administrator）下构建了一个Java Project叫做myapp。

　　构建好的Java项目的目录结构如下：

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171325.png)

　　可以看到，Maven帮我们创建的项目是一个标准的Maven项目，不过目前Maven只是帮我们生成了src/main/java(存放项目的源代码)和src/test/java(存放测试源代码)这两个目录，但实际项目开发中我们一般都会有配置文件，例如log4j.properties，所以我们还需要手动创建src/main/resources(存放项目开发中用到的配置文件，如存放log4j.properties等)和src/test/resources(存放测试时用到的配置文件)，如下图所示：

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171327.png)

　　然后我们就可以将创建好的myapp项目导入到Eclipse中进行开发了，如下图所示：

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171329.png)

### 1.2、JavaProject的pom.xml文件说明

　　通过Maven构建的JavaProject，在项目的根目录下都会存在一个pom.xml文件，**进入myapp目录，可以看到有一个pom.xml文件，这个文件是Maven的核心。如下图所示：**

　　**![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171331.png)**

　　　　1、pom意思就是project object model。

　　　　2、pom.xml包含了项目构建的信息，包括项目的信息、项目的依赖等。

　　　　3、pom.xml文件是可以继承的，大型项目中，子模块的pom.xml一般都会继承于父模块的pom.xml

　　pom.xml文件的内容如下：

```xml
 <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>
 
   <groupId>com.mycompany.app</groupId>
   <artifactId>myapp</artifactId>
   <version>1.0-SNAPSHOT</version>
   <packaging>jar</packaging>
 
   <name>myapp</name>
   <url>http://maven.apache.org</url>
 
   <properties>
     <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
   </properties>
 
   <dependencies>
     <dependency>
       <groupId>junit</groupId>
       <artifactId>junit</artifactId>
       <version>3.8.1</version>
       <scope>test</scope>
     </dependency>
   </dependencies>
 </project>
```

　　pom.xml文件的节点元素说明：

-  `<project>`　　　　　　　pom文件的顶级节点
- `<modelVersion>`　　　　 object model版本，对Maven2和Maven3来说，只能是4.0.0　
- `<groupId>`　　　　　　　项目创建组织的标识符，一般是域名的倒写
- `<artifactId>`　　　　　 定义了项目在所属组织的标识符下的唯一标识，一个组织下可以有多个项目
- `<version>`　　　　　 　当前项目的版本，SNAPSHOT，表示是快照版本，在开发中
- `<packaging>`　　　　 打包的方式，有jar、war、ear等
- `<name>`　　　　　　　 项目的名称
- `<url>`　　　　　　　　 项目的地址
- `<properties>`　　　　属性配置，比如：<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
- `<dependencies>`　　  构建项目依赖的jar

　　其中**由groupId、artifactId和version唯一的确定了一个项目坐标**

### 1.3、使用Maven编译-测试-打包-安装项目

#### 1.3.1、编译

　　编译源程序，进入命令行，切换到myapp目录，执行命令：**mvn clean compile**，如下图所示：

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171337.png)

　　编译成功，在myapp目录下多出一个target目录，target\classes里面存放的就是编译后的class文件，如下图所示：

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171340.png)

#### 1.3.2、测试

　　进入命令行，切换到myapp目录，执行命令：**mvn clean test**，如下图所示：

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171342.png)

　　测试成功，在myapp\target目录下会有一个test-classes目录，存放的就是测试代码的class文件，如下图所示：

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171345.png)

#### 1.3.3、打包

　　进入命令行，切换到myapp目录，执行命令：**mvn clean package**，执行打包命令前，会先执行编译和测试命令，如下图所示：

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171348.png)

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171350.png)

　　构建成功后，会在target目录下生成myapp-1.0-SNAPSHOT.jar包，如下图所示：

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171352.png)

#### 1.3.4、安装

　　进入命令行，切换到my-app目录，执行命令：**mvn clean install** ，执行安装命令前，会先执行编译、测试、打包命令，如下图所示：

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171354.png)

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171357.png)

　　构建成功，就会将项目的jar包安装到本地仓库，如下图所示：

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171400.png)

#### 1.3.5、运行jar包

　　进入命令行，切换到myapp目录，执行命令：java -cp target\myapp-1.0-SNAPSHOT.jar com.mycompany.app.App，如下图所示：

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171401.png)

## 二、构建JavaWeb项目

### 2.1、创建JavaWeb项目

　　1、使用**mvn archetype:generate**命令，如下所示：

```shell
mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=my-WebApp -DarchetypeArtifactId=maven-archetype-webapp -DinteractiveMode=false
```

　　使用"**mvn archetype:generate**"命令创建一个javaWeb项目的过程如下图所示：　　

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171403.png)

　　使用"**mvn archetype:generate**"命令创建一个javaWeb项目的时间非常长，要了40多秒，有时甚至会更久，不知道为啥。

　　2、使用**mvn archetype:create**命令，如下所示：

```shell
mvn archetype:create -DgroupId=com.mycompany.app -DartifactId=myWebApp -DarchetypeArtifactId=maven-archetype-webapp -DinteractiveMode=false
```

　　使用"**mvn archetype:create**"命令创建一个javaWeb项目的过程如下图所示：　　

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171407.png)

　　使用"**mvn archetype:create**"命令创建一个javaWeb项目的时间非常快，几秒钟就可以了。

　　创建好的JavaWeb项目的目录结构如下：

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171410.png)

　　创建好的JavaWeb项目中目前只有src/main/resources目录，因此还需要手动添加src/main/java、src/test/java、src/test/resources

　　如下图所示：

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171412.png)

　　接着我们就可以将创建好的JavaWeb导入Eclipse中进行开发了，如下图所示：

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171414.png)

### 2.2、使用Maven打包发布Web项目

　　Maven帮我们创建的JavaWeb项目是一个空的项目，只有一个index.jsp页面，我们使用Maven将Web项目打包发布运行。

　　在命令行切换到myWebApp目录，执行：**mvn package**，构建成功后，myWebApp目录目录下多了一个target目录，在这个目录下会打包成myWebApp目录.war，把这个war包拷贝到Tomcat的发布目录下就可以运行了。如下图所示：

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171415.png)

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171418.png)

　　打包成功，在myWebApp\target目录下生成了一个myWebApp.war文件，如下图所示：

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171421.png)

　　将myWebApp.war放到tomcat服务器中运行，如下图所示：

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171424.png)

　　运行效果如下：

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171426.png)

　　除了使用Tomcat服务器运行Web项目之外，我们还可以在Web项目中集成Jetty发布运行，首先在pom.xml文件中配置Jetty插件，如下：

```xml
 <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
   <modelVersion>4.0.0</modelVersion>
   <groupId>com.mycompany.app</groupId>
   <artifactId>myWebApp</artifactId>
   <packaging>war</packaging>
   <version>1.0-SNAPSHOT</version>
   <name>myWebApp Maven Webapp</name>
   <url>http://maven.apache.org</url>
   <dependencies>
     <dependency>
       <groupId>junit</groupId>
       <artifactId>junit</artifactId>
       <version>3.8.1</version>
       <scope>test</scope>
     </dependency>
   </dependencies>
   <build>
     <finalName>myWebApp</finalName>
      <pluginManagement>
         <!--配置Jetty-->
           <plugins>
             <plugin>
              <groupId>org.mortbay.jetty</groupId>   
              <artifactId>maven-jetty-plugin</artifactId>
             </plugin>
           </plugins>
     </pluginManagement>
   </build>
 </project>
```

　　打开命令行窗口，切换到myWebApp目录，然后执行：**mvn jetty:run**启动Jetty服务器，如下图所示：

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171430.png)

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171431.png)

　　接着就可以在8080端口上访问应用了。如下图所示：

　　![img](%E4%B8%89%E3%80%81%E4%BD%BF%E7%94%A8Maven%E6%9E%84%E5%BB%BA%E9%A1%B9%E7%9B%AE.resource/Mon,%2018%20Nov%202019%20171433.png)

## 三、Maven创建项目的命令说明

　　mvn archetype:create或者mvn archetype:generate　　固定写法

　　-DgroupId　　　　　　　　　　　　　　　　　　　　　　　组织标识（包名）

　　-DartifactId　　　　　　　　　　　　　　　　　　　　　　项目名称

　　-DarchetypeArtifactId　　 　　　　　　　　　　　　　　指定ArchetypeId，maven-archetype-quickstart，创建一个Java Project；maven-archetype-webapp，创建一个Web Project

　　-DinteractiveMode　　　　　　　　　　　　　　　　　　　　是否使用交互模式

　　archetype是mvn内置的一个插件，create任务可以创建一个java项目骨架，DgroupId是软件包的名称，DartifactId是项目名，DarchetypeArtifactId是可用的mvn项目骨架，目前可以使用的骨架有：

- maven-archetype-archetype
- maven-archetype-j2ee-simple
- maven-archetype-mojo
- maven-archetype-portlet
- maven-archetype-profiles (currently under development)
- **maven-archetype-quickstart**
- maven-archetype-simple (currently under development)
- maven-archetype-site
- maven-archetype-site-simple
- **maven-archetype-webapp**

　　每一个骨架都会建相应的目录结构和一些通用文件，最常用的是**maven-archetype-quickstart**和**maven-archetype-webapp**骨架。maven-archetype-quickstart骨架是用来创建一个Java Project，而maven-archetype-webapp骨架则是用来创建一个JavaWeb Project。

　　不得不说，Maven的确是一个很好的项目构建工具。掌握好Maven对于项目开发是非常有帮助的。