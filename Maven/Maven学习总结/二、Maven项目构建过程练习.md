# Maven项目构建过程练习

## 一、创建Maven项目

### （一）建立Hello项目

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

2、编辑项目Hello根目录下的pom.xml，添加如下的代码：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>me.gacl.maven</groupId>
    <artifactId>Hello</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>Hello</name>

    <!--添加依赖的jar包-->
    <dependencies>
        <!--项目要使用到junit的jar包，所以在这里添加junit的jar包的依赖-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.9</version>
            <scope>test</scope>
        </dependency>        

    </dependencies>
    project>
```

3、在src/main/java/me/gacl/maven目录下新建文件Hello.java

```java
package me.gacl.maven;

public class Hello {

    public String sayHello(String name){
        return "Hello "+name+"!";
    }
}
```

4、在/src/test/java/me/gacl/maven目录下新建测试文件HelloTest.java　　

```java
package me.gacl.maven;
//导入junit的包
import org.junit.Test;
import static junit.framework.Assert.*;

public class HelloTest {

    @Test
    public void testHello(){
        Hello hello = new Hello();
        String results = hello.sayHello("gacl");
        assertEquals("Hello gacl!",results);        
    }
}
```

### （二）使用Maven编译、清理、测试、打包项目

1、使用Maven编译项目，编译项目的命令是：`mvn compile`，进入Hello项目根目录执行 `mvn compile` 命令编译项目的 java 类

　　![img](%E4%BA%8C%E3%80%81Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20170950.png)

编译成功之后，可以看到hello项目的根目录下多了一个【target】文件夹，这个文件夹就是编译成功之后Maven帮我们生成的文件夹，打开【target】文件夹，可以看到里面有一个【classes】文件夹，文件夹中存放的就是Maven我们编译好的java类，即 Hello.class

这就是使用 Maven 自动编译项目的过程。

2、使用Maven清理项目，清理项目的命令是：`mvn clean`

　进入Hello项目根目录执行 `mvn clean` 命令清理项目，清理项目的过程就是把执行 `mvn compile` 命令编译项目时生成的 target 文件夹删掉，如下图所示：

　　![img](%E4%BA%8C%E3%80%81Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20170957.png)

3、使用Maven测试项目，测试项目的命令是：`mvn test`

　　进入Hello项目根目录执行 `mvn test` 命令测试项目，如下图所示：

　　![img](%E4%BA%8C%E3%80%81Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20170959.png)

　　测试成功之后，可以看到hello项目的根目录下多了一个【target】文件夹，这个文件夹就是测试成功之后Maven帮我们生成的文件夹，打开【target】文件夹，可以看到里面有一个【classes】和【test-classes】文件夹，即执行 `mvn test` 命令测试项目时，Maven 先帮我们编译项目，然后再执行测试代码。

**4、使用Maven打包项目，打包项目的命令是：`mvn package`

　　进入Hello项目根目录执行 `mvn package`  命令测试项目，如下图所示：

　　![img](%E4%BA%8C%E3%80%81Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20171005.png)

　　![img](%E4%BA%8C%E3%80%81Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20171007.png)

打包成功之后，可以看到 hello 项目的根目录下的【target】文件夹中多了一个 Hello-0.0.1-SNAPSHOT.jar，这个Hello-0.0.1-SNAPSHOT.jar就是打包成功之后Maven帮我们生成的jar文件，如下图所示：

5、使用Maven部署项目，部署项目的命令是：`mvn install`

　　进入 Hello项目根目录执行 `mvn install` 命令测试项目，如下图所示：

　　![img](%E4%BA%8C%E3%80%81Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20171010.png)

　　![img](%E4%BA%8C%E3%80%81Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20171012.png)

安装成功之后，首先会在hello项目的根目录下生成【target】文件夹，打开【target】文件夹，可以看到里面会有Hello-0.0.1-SNAPSHOT.jar，这个Hello-0.0.1-SNAPSHOT.jar就是安装成功之后Maven帮我们生成的jar文件，如下图所示：

除此之外，在我们存放Maven下载下来的jar包的仓库也会有一个Hello-0.0.1-SNAPSHOT.jar，所以Maven安装项目的过程，实际上就是把项目进行【清理】→【编译】→【测试】→【打包】，再把打包好的jar放到我们指定的存放jar包的Maven仓库中，如下图所示：

所以使用"mvn install"命令，就把maven构建项目的【清理】→【编译】→【测试】→【打包】的这几个过程都做了，同时将打包好的jar包发布到本地的Maven仓库中，所以maven最常用的命令还是 `mvn install`，这个命令能够做的事情最多。

### 二、组合使用Maven的命令

maven的编译，清理，测试，打包，部署命令是可以几个命令同时组合起来使用的，常用的命令组合如下：

　　1、先清理再编译：`mvn clean compile`，如下所示：

　　![img](%E4%BA%8C%E3%80%81Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20171018.png)

　　还有的就是"mvn clean test"，"mvn clean package"，"mvn clean install"，这些组合命令都比较常用。

　　以上就是关于Maven构建项目的各个个过程演示。

## 二、在别的项目中使用通过Maven安装生成的项目的jar包

在上面，我们使用 `mvn install` 命令将 hello 这个项目打包成了 Hello-0.0.1-SNAPSHOT.jar 包并且发布到本地的 maven 仓库E:\repository\me\gacl\maven\Hello\0.0.1-SNAPSHOT 中，下面我们来看看如何在别的项目中使用 Hello-0.0.1-SNAPSHOT.jar

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

　　2、编辑项目HelloFriend根目录下的pom.xml，添加如下的代码：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>me.gacl.maven</groupId>
    <artifactId>HelloFriend</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>HelloFriend</name>

    <!--添加依赖的jar包-->
    <dependencies>
        <!--项目要使用到junit的jar包，所以在这里添加junit的jar包的依赖-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.9</version>
            <scope>test</scope>
        </dependency>
        
        <!--项目要使用到Hello的jar包，所以在这里添加Hello的jar包的依赖-->
        <dependency>
            <groupId>me.gacl.maven</groupId>
            <artifactId>Hello</artifactId>
            <version>0.0.1-SNAPSHOT</version>
            <scope>compile</scope>
        </dependency>    
    </dependencies>
</project>
```

3、在src/main/java/me/gacl/maven目录下新建文件HelloFriend.java

```java
package me.gacl.maven;

import me.gacl.maven.Hello;

public class HelloFriend {

    public String sayHelloToFriend(String name){

        Hello hello = new Hello();
        String str = hello.sayHello(name)+" I am "+this.getMyName();
        System.out.println(str);
        return str;
    }

    public String getMyName(){
        return "John";
    }
}
```

4、在/src/test/java/me/gacl/maven目录下新建测试文件HelloFriendTest.java，如下图所示：

```java
package me.gacl.maven;

import static junit.framework.Assert.assertEquals;
import org.junit.Test;
import me.gacl.maven.Hello;

public class HelloFriendTest {

    @Test
    public void tesHelloFriend(){

        HelloFriend helloFriend = new HelloFriend();
        String results = helloFriend.sayHelloToFriend("gacl");
        assertEquals("Hello gacl! I am John",results);
    }
}
```

5、在HelloFriend目录下执行命令"mvn package"测试Hello-0.0.1-SNAPSHOT.jar里面的类是否引用成功，如下所示：

　　![img](%E4%BA%8C%E3%80%81Maven%E9%A1%B9%E7%9B%AE%E6%9E%84%E5%BB%BA%E8%BF%87%E7%A8%8B%E7%BB%83%E4%B9%A0.resource/Mon,%2018%20Nov%202019%20170929.png)