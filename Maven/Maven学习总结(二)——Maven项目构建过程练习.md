只为成功找方法，不为失败找借口！

## Maven学习总结(二)——Maven项目构建过程练习

　　上一篇只是简单介绍了一下maven入门的一些相关知识，这一篇主要是体验一下Maven高度自动化构建项目的过程

## 一、创建Maven项目

### 1.1、建立Hello项目

　　1、首先建立Hello项目，同时建立Maven约定的目录结构和pom.xml文件

　　　　Hello
　　　　　　| --src
　　　　　　| -----main
　　　　　　| ----------java
　　　　　　| ----------resources
　　　　　　| -----test
　　　　　　| ---------java
　　　　　　| ---------resources
　　　　　　| --pom.xml

　　![img](Maven%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93(%E4%BA%8C)%E2%80%94%E2%80%94Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20171040.png)

　　2、编辑项目Hello根目录下的pom.xml，添加如下的代码：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
 2 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
 3   <modelVersion>4.0.0</modelVersion>
 4   <groupId>me.gacl.maven</groupId>
 5   <artifactId>Hello</artifactId>
 6   <version>0.0.1-SNAPSHOT</version>
 7   <name>Hello</name>
 8   
 9     <!--添加依赖的jar包-->
10     <dependencies>
11         <!--项目要使用到junit的jar包，所以在这里添加junit的jar包的依赖-->
12         <dependency>
13             <groupId>junit</groupId>
14             <artifactId>junit</artifactId>
15             <version>4.9</version>
16             <scope>test</scope>
17         </dependency>        
18         
19     </dependencies>
20 </project>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　3、在src/main/java/me/gacl/maven目录下新建文件Hello.java

　　![img](Maven%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93(%E4%BA%8C)%E2%80%94%E2%80%94Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20170943.png)

　　Hello.java的代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
1 package me.gacl.maven;
2 
3 public class Hello {
4     
5     public String sayHello(String name){
6         return "Hello "+name+"!";
7     }
8 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　4、在/src/test/java/me/gacl/maven目录下新建测试文件HelloTest.java　　

　　![img](Maven%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93(%E4%BA%8C)%E2%80%94%E2%80%94Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20170946.png)

　　HelloTest.java的代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 package me.gacl.maven;
 2 //导入junit的包
 3 import org.junit.Test;
 4 import static junit.framework.Assert.*;
 5 
 6 public class HelloTest {
 7 
 8     @Test
 9     public void testHello(){
10         Hello hello = new Hello();
11         String results = hello.sayHello("gacl");
12         assertEquals("Hello gacl!",results);        
13     }
14 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

### 1.2、使用Maven编译、清理、测试、打包项目

**1、使用Maven编译项目，编译项目的命令是："\**mvn compile\**"**

　　打开cmd命令行，

　　![img](Maven%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93(%E4%BA%8C)%E2%80%94%E2%80%94Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20170948.png)

　　进入Hello项目根目录执行"**mvn compile**"命令编译项目的java类

　　![img](Maven%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93(%E4%BA%8C)%E2%80%94%E2%80%94Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20170950.png)

　　编译成功之后，可以看到hello项目的根目录下多了一个【target】文件夹，这个文件夹就是编译成功之后Maven帮我们生成的文件夹，如下图所示：

　　![img](Maven%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93(%E4%BA%8C)%E2%80%94%E2%80%94Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20170952.png)

　　打开【target】文件夹，可以看到里面有一个【classes】文件夹，如下图所示：

　　![img](Maven%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93(%E4%BA%8C)%E2%80%94%E2%80%94Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20170954.png)

　　【classes】文件夹中存放的就是Maven我们编译好的java类，如下图所示：

　　![img](Maven%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93(%E4%BA%8C)%E2%80%94%E2%80%94Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20170956.png)

　　这就是使用Maven自动编译项目的过程。

**2、使用Maven清理项目，清理项目的命令是："\**mvn clean\**"**

　　进入Hello项目根目录执行"**mvn clean**"命令清理项目，清理项目的过程就是把执行"**mvn compile**"命令编译项目时生成的target文件夹删掉，如下图所示：

　　![img](Maven%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93(%E4%BA%8C)%E2%80%94%E2%80%94Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20170957.png)

**3、使用Maven测试项目，测试项目的命令是："\**mvn test\**"**

　　进入Hello项目根目录执行"***\*mvn test\****"命令测试项目，如下图所示：

　　![img](Maven%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93(%E4%BA%8C)%E2%80%94%E2%80%94Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20170959.png)

　　测试成功之后，可以看到hello项目的根目录下多了一个【target】文件夹，这个文件夹就是测试成功之后Maven帮我们生成的文件夹，如下图所示：

　　![img](Maven%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93(%E4%BA%8C)%E2%80%94%E2%80%94Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20171002.png)

　　打开【target】文件夹，可以看到里面有一个【classes】和【test-classes】文件夹，如下图所示：

　　![img](Maven%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93(%E4%BA%8C)%E2%80%94%E2%80%94Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20171003.png)

　　也就是说，我们执行执行"***\*mvn test\****"命令测试项目时，Maven先帮我们编译项目，然后再执行测试代码。

**4、使用Maven打包项目，打包项目的命令是："mvn package"**

　　进入Hello项目根目录执行"***\**\*mvn\*\**\*** **package**"命令测试项目，如下图所示：

　　![img](Maven%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93(%E4%BA%8C)%E2%80%94%E2%80%94Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20171005.png)

　　![img](Maven%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93(%E4%BA%8C)%E2%80%94%E2%80%94Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20171007.png)

　　打包成功之后，可以看到hello项目的根目录下的【target】文件夹中多了一个Hello-0.0.1-SNAPSHOT.jar，这个Hello-0.0.1-SNAPSHOT.jar就是打包成功之后Maven帮我们生成的jar文件，如下图所示：

　　![img](Maven%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93(%E4%BA%8C)%E2%80%94%E2%80%94Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20171009.png)

**5****、使用Maven部署项目，部署项目的命令是："mvn install"**

　　进入Hello项目根目录执行"***\**\*\*\*mvn\*\*\*\*\**** **install**"命令测试项目，如下图所示：

　　![img](Maven%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93(%E4%BA%8C)%E2%80%94%E2%80%94Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20171010.png)

　　![img](Maven%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93(%E4%BA%8C)%E2%80%94%E2%80%94Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20171012.png)

　　安装成功之后，首先会在hello项目的根目录下生成【target】文件夹，打开【target】文件夹，可以看到里面会有Hello-0.0.1-SNAPSHOT.jar，这个Hello-0.0.1-SNAPSHOT.jar就是安装成功之后Maven帮我们生成的jar文件，如下图所示：

　　![img](Maven%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93(%E4%BA%8C)%E2%80%94%E2%80%94Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20171014.png)

　　除此之外，在我们存放Maven下载下来的jar包的仓库也会有一个Hello-0.0.1-SNAPSHOT.jar，所以Maven安装项目的过程，实际上就是把项目进行【清理】→【编译】→【测试】→【打包】，再把打包好的jar放到我们指定的存放jar包的Maven仓库中，如下图所示：

　　![img](Maven%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93(%E4%BA%8C)%E2%80%94%E2%80%94Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20171017.png)

　　所以使用"mvn install"命令，就把maven构建项目的【清理】→【编译】→【测试】→【打包】的这几个过程都做了，同时将打包好的jar包发布到本地的Maven仓库中，所以maven最常用的命令还是"mvn install"，这个命令能够做的事情最多。

### 1.2、组合使用Maven的命令

　　maven的编译，清理，测试，打包，部署命令是可以几个命令同时组合起来使用的，常用的命令组合如下：

　　1、先清理再编译："**mvn clean compile**"，如下所示：

　　![img](Maven%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93(%E4%BA%8C)%E2%80%94%E2%80%94Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20171018.png)

　　还有的就是"mvn clean test"，"mvn clean package"，"mvn clean install"，这些组合命令都比较常用。

　　以上就是关于Maven构建项目的各个个过程演示。

## 二、在别的项目中使用通过Maven安装生成的项目的jar包

　　在上面，我们使用mvn install命令将hello这个项目打包成了Hello-0.0.1-SNAPSHOT.jar包并且发布到本地的maven仓库E:\repository\me\gacl\maven\Hello\0.0.1-SNAPSHOT中，下面我们来看看如何在别的项目中使用Hello-0.0.1-SNAPSHOT.jar

　　1、新建HelloFriend项目，同时建立Maven约定的目录结构和pom.xml文件
　　　　HelloFriend
 　　　　 | --src
　　　　　　| -----main
　　　　　　| ----------java
　　　　　　| ----------resources
　　　　　　| -----test
　　　　　　| ---------java
　　　　　　| ---------resources
　　　　　　| --pom.xml

　　如下图所示：

　　![img](Maven%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93(%E4%BA%8C)%E2%80%94%E2%80%94Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20171021.png)

　　2、编辑项目HelloFriend根目录下的pom.xml，添加如下的代码：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
 2 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
 3   <modelVersion>4.0.0</modelVersion>
 4   <groupId>me.gacl.maven</groupId>
 5   <artifactId>HelloFriend</artifactId>
 6   <version>0.0.1-SNAPSHOT</version>
 7   <name>HelloFriend</name>
 8   
 9     <!--添加依赖的jar包-->
10     <dependencies>
11         <!--项目要使用到junit的jar包，所以在这里添加junit的jar包的依赖-->
12         <dependency>
13             <groupId>junit</groupId>
14             <artifactId>junit</artifactId>
15             <version>4.9</version>
16             <scope>test</scope>
17         </dependency>
18         <!--项目要使用到Hello的jar包，所以在这里添加Hello的jar包的依赖-->
19         <dependency>
20             <groupId>me.gacl.maven</groupId>
21             <artifactId>Hello</artifactId>
22             <version>0.0.1-SNAPSHOT</version>
23             <scope>compile</scope>
24         </dependency>    
25     </dependencies>
26 </project>
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　3、在src/main/java/me/gacl/maven目录下新建文件HelloFriend.java，如下图所示：

　　![img](Maven%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93(%E4%BA%8C)%E2%80%94%E2%80%94Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20171024.png)

　　HelloFriend.java的代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 package me.gacl.maven;
 2 
 3 import me.gacl.maven.Hello;
 4 
 5 public class HelloFriend {
 6 
 7     public String sayHelloToFriend(String name){
 8         
 9         Hello hello = new Hello();
10         String str = hello.sayHello(name)+" I am "+this.getMyName();
11         System.out.println(str);
12         return str;
13     }
14     
15     public String getMyName(){
16         return "John";
17     }
18 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　4、在/src/test/java/me/gacl/maven目录下新建测试文件HelloFriendTest.java，如下图所示：

　　![img](Maven%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93(%E4%BA%8C)%E2%80%94%E2%80%94Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20171026.png)

　　HelloFriendTest.java的代码如下：

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

```
 1 package me.gacl.maven;
 2 
 3 import static junit.framework.Assert.assertEquals;
 4 import org.junit.Test;
 5 import me.gacl.maven.Hello;
 6 
 7 public class HelloFriendTest {
 8 
 9     @Test
10     public void tesHelloFriend(){
11         
12         HelloFriend helloFriend = new HelloFriend();
13         String results = helloFriend.sayHelloToFriend("gacl");
14         assertEquals("Hello gacl! I am John",results);
15     }
16 }
```

[![复制代码](https://common.cnblogs.com/images/copycode.gif)](javascript:void(0);)

　　5、在HelloFriend目录下执行命令"mvn package"测试Hello-0.0.1-SNAPSHOT.jar里面的类是否引用成功，如下所示：

　　![img](Maven%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93(%E4%BA%8C)%E2%80%94%E2%80%94Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20170929.png)