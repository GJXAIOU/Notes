# FrameDay05_1 Struts2

**Struts2 学习安排**
* struts2入门
* struts2数据操作
* struts2值栈
* struts2拦截器

**今天内容**
- struts2概述
应用在web层
- struts2入门案例
- struts2底层执行过程
- struts2相关配置
- struts.xml配置
  - package、action、result标签
  - 常量配置
  - 分模块开发

- struts2的action创建
有三种，一般使用继承类 ActionSupport实现

- struts2的action方法访问
  - 使用action标签method属性，method属性值写执行的方法名称
  - 使用通配符方式，使用`*`匹配任意内容，method里面写`*`值，写法固定 {1}

## 一、Struts2概述
是一个基于 MVC 设计模式的 Web 应用框架，本质上相当于一个 Servlet，Struts2 作为一个控制器（Controller）来建立模型与视图的数据交互；**使用拦截器的机制来处理用户请求**， WEB 层框架都是基于前端控制器模式实现；

- struts2框架应用javaee三层结构中web层框架
- struts2框架在struts1和webwork基础之上发展全新的框架
- struts2解决问题：

![01-struts2概述](FrameDay05_1%20Struts2.resource/01-struts2%E6%A6%82%E8%BF%B0.png)

- web层常见框架
  - struts2
  - springMVC

## 二、 Struts2框架入门

- 第一步 导入jar包
  - 在lib中有jar包，但是不必也不能把这些jar都导入到项目中；
  - 到apps目录里面，找到示例程序，从示例程序（打开压缩包里面的lib）复制jar包
```jar
asm-5.2.jar  //操作 Java 字节码的类库
asm-commons-5.2.jar // 提供了基于事件的表现形式
asm-tree-5.2.jar // 提供了基于事件的表现形式
commons-fileupload-1.4.jar // Struts2 文件上传组件依赖包
commons-io-2.6.jar // Struts2 输入输出以及文件依赖包
commons-lang3-3.8.1.jar // 包含一些数据类型工具，是对 java.lang 包的增强
freemarker-2.3.28.jar // Struts2 标签模板使用的类库
javassist-3.20.0-GA.jar // JavaScript 字节码解析器
log4j-api-2.11.1.jar // 日志管理组件依赖包
log4j-core-2.11.1.jar // 日志管理组件依赖包
ognl-3.1.21.jar // 对象视图导航语言，是 Struts2 框架通过其读写对象的属性
stax2-api-3.1.4.jar
struts2-config-browser-plugin-2.5.20.jar
struts2-convention-plugin-2.5.20.jar
struts2-core-2.5.20.jar   // struts2 核心类库
struts2-rest-plugin-2.5.20.jar
woodstox-core-5.0.3.jar
xwore-core.jar //  WebWork 核心库，是 Struts 构建基础；
xmlpull-1.1.3.1.jar
xpp3_min-1.1.4c.jar
xstream-1.4.11.1.jar
```


- 第二步：创建 action
```java
package com.gjxaiou.action;

/**
 * @author GJXAIOU
 * @create 2019-09-29-15:53
 */
public class HelloAction {
    /**
     * (1) 之前项目中，每次访问 servlet 的时候，都会执行 service 方法；
     *（2）在 Struts2 中，每次访问 action 都会默认执行 execute 方法；
     */
    public String execute(){
        return "OK";
    }
}
```
**execute 方法的返回值是一个 String 类型，该值是一个逻辑视图；**
逻辑视图：相当于对一个真实的页面取了一个别名；

- 第三步：配置 action 类访问路径
  - 首先创建 struts2 核心配置文件：struts.xml
  - 核心配置文件名称和位置是固定的
位置必须在src下面，名称 struts.xml
  - 首先引入 dtd 约束，然后进行 action 配置
  本地版的 dtd 在  structs2 - core.jar 下面的 struts-2.5.dtd 中的 28-30 行；
```struts_xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts PUBLIC
        "-//Apache Software Foundation//DTD Struts Configuration 2.5//EN"
        "http://struts.apache.org/dtds/struts-2.5.dtd">

<struts>
    <package name="helloDemo" extends="struts-default" namespace="/">
        <!--name：发布之后访问名称，这里通过 hello ，就可以访问到 HelloAction，然后根据 HelloAction 的返回值
            是 OK，对应下面的返回值，可以跳转到 hello.jsp 页面；-->
        <action name="hello" class="com.gjxaiou.action.HelloAction">
            <!--这里配置方法的返回值对应的页面-->
            <result name="OK">/hello.jsp</result>
        </action>
    </package>
</struts>
```

这时候通过访问路径：http://localhost:8080/StandardStrutsProject_war_exploded/hello.action ， 这里的 action 可以省略
 但是这时候因为没有配置过滤器，因此会报错 404；

- 第四步：配置 struts2 过滤器（web.xml 中配置）
 **Struts2 如果想执行，所有的请求都需要进过这个前端控制器（核心控制器），因为这个过滤器完成了框架的部分功能；**
```web_xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>Archetype Created Web Application</display-name>

  <filter>
    <filter-name>struts2</filter-name>
    <filter-class>org.apache.struts2.dispatcher.filter.StrutsPrepareAndExecuteFilter</filter-class>
  </filter>

  <filter-mapping>
    <filter-name>struts2</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
</web-app>
```

上面 filter 里面的 StrutsPrepareAndExecutefilter 方法实现为：`public class StrutsPrepareAndExecuteFilter implements StrutsStatics, Filter {`，因为实现了 Filter，因此具有过滤器功能；


## 三、 Struts2 执行过程
从客户端发送请求过来之后，先进过前端控制器（核心过滤器为：StrutsPrepareAndExecuteFilter），过滤器中执行了一组拦截器（一组拦截器 就会完成部分功能代码），在 Struts2 中定义的默认栈中的拦截器会得到执行，执行完拦截器之后了就会执行目标 Action，在 Action 中返回一个结果视图，根据 Result 的配置进行页面跳转。
**默认的 Struts2 拦截器栈**
位置： struts-core.jar -> struts-default.xml 中 338 行；
```xml
<interceptor-stack name="defaultStack">
                <interceptor-ref name="exception"/>
                <interceptor-ref name="alias"/>
                <interceptor-ref name="servletConfig"/>
                <interceptor-ref name="i18n"/>
                <interceptor-ref name="prepare"/>
                <interceptor-ref name="chain"/>
                <interceptor-ref name="scopedModelDriven"/>
                <interceptor-ref name="modelDriven"/>
                <interceptor-ref name="fileUpload"/>
                <interceptor-ref name="checkbox"/>
                <interceptor-ref name="datetime"/>
                <interceptor-ref name="multiselect"/>
                <interceptor-ref name="staticParams"/>
                <interceptor-ref name="actionMappingParams"/>
                <interceptor-ref name="params"/>
                <interceptor-ref name="conversionError"/>
                <interceptor-ref name="validation">
                    <param name="excludeMethods">input,back,cancel,browse</param>
                </interceptor-ref>
                <interceptor-ref name="workflow">
                    <param name="excludeMethods">input,back,cancel,browse</param>
                </interceptor-ref>
                <interceptor-ref name="debugging"/>
            </interceptor-stack>
```

**Struts2 执行过程为：**
![02-struts2执行过程](FrameDay05_1%20Struts2.resource/02-struts2%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B.png)

**Struts2 配置文件的加载顺序**
每次从客户端发送请求到服务器都要先经过 Struts2 的核心过滤器StrutsPrepareAndExecuteFilter ,这个过滤器有两个功能： 预处理和执行。 **在预处理中主要就是来加载配置文件的。 对应的是过滤器中的 init 方法**，而执行是用来执行 一 组拦截器完成部分功能的，对应的是过滤器的 doFilter 方法。所以我们如果要去了解 Struts2 的配置文件的加载顺序， 那么我们需要查询过滤器的 init 方法。

点击上面的拦截器进入方法内部，找到 init 方法，然后进入 `init.initDispatcher(config)` 方法中（该方法用于加载配置文件），然后进入 dispatcher.init()中的 init() 方法，然后下面的代码就是用来加载配置文件的；
```java
this.init_FileManager();
// 加载org.apache.struts. default. properties配置的是struts2的所有常量
this.init_DefaultProperties();
// 加载struts-default.xml、struts-plugin.xml、struts.xml
this.init_TraditionalXmlConfigurations();
// 加载用户自定义 struts.properties 
this.init_LegacyStrutsProperties();
// 加载用户配置的提供对象
this.init_CustomConfigurationProviders();
// 加载web.xml
this.init_FilterInitParameters();
// 加载标准对象
this.init_AliasStandardObjects();
```

因此 Struts 配置文件加载顺序为：
- default.properties
- struts-default.xml
- struts-plugin.xml
- struts.xml
- struts.properties
- web.xml

后面三个文件用户可以修改，且三个配置文件都可以修改 Struts 的常量的值，后加载配置文件中的常量值会覆盖先加载的配置文件中常量值；

- 过滤器在服务器启动时候创建，创建过滤器时候执行了 init 方法
  - 在init方法中主要作用：加载配置文件（使用 initDispatcher 方法进行加载，然后里面调用了 init 里面的 try 按照顺序加载不同的配置文件)；
  - 包含自己创建的配置文件和struts2自带配置文件
其中： struts.xml ：配置 action；以及 web.xml ： 配置过滤器

## 四、Struts2 的核心配置文件
 **用于配置 Action 和请求的对应关系；**
- 名称和位置固定的：文件名为：`struts.xml`，位置：放在 src 下面或者 resources 下面；
- 在配置文件中主要三个标签 package、action、result，标签里面的属性；

### （一）标签 package
因为 Struts2 框架的核心组件是 Action 和拦截器，它使用包来管理 Action 和拦截器；每个包就是多个 Action、多个拦截器、多个拦截器引用的集合；使用 package 标签定义包配置；
类似于代码中的包，用于区别不同的 action，要配置 action，必须首先写 package 标签，然后在 package 里面才能配置 action；

**package 标签属性**
只有 name 属性是必选属性；
- name 属性
  - name 属性值和功能本身没有关系的；
  - 在一个配置文件中可以写多个 package 标签，不同 package 标签的 name 属性值不能相同的；
  - name 属性值是该包被其他包引用的 key；

- extends 属性
  - 表示该包继承自其他包（值为其他包的 name 值）；如果继承其他包，可以继承其他包中的 Action 定义、拦截器定义等；一般值固定为：`struts-default`
写了这个属性之后，在 package 里面配置的类具有 action 功能（上面代码中就是 HelloAction 这个类就有了 action 的功能）；

- namespace属性
namespace 属性值和 action 标签里面的 name 属性值构成访问路径（这里两者就构成了 `/hello` 的访问路径，且 `/hello` 和 `/hello.action` 相等价），该属性不写默认就是 `/`；
  - 默认名称空间：值为：`""`；
  - 跟名称空间：值为：`"/"`；
  - 带名称的名称空间：值为：`"/demo1"`

- abstract 属性
 指定该包是否是一个抽象包，抽象包中不能包含 Action 定义；值为：true 或者 false；


### （二）标签 action
Action 映射就是将一个请求的 URL 映射到一个 Action 类；
action 标签用于配置 action 的访问路径；

**action标签属性**
只有 name 属性是必填标签；
- name 属性
  - namespace 属性值和 action 标签里面的 name 属性值构成访问路径；
  - 且在 package 标签里面可以写多个 action 标签，但是 action 的 name 属性值不能相同的；

- class 属性
表示对应 action 类的全路径；

- method 属性
比如在 action 里面默认执行的方法 execute 方法，但是在 action 里面可以写其他的方法，如果让 action 里面多个方法执行，使用 method 进行配置；

- converter 属性
指定类型转换器的类；

### （三）标签result

根据action的方法返回值，配置到不同的路径里面（不一定是页面，也可以是其他的 action）；

**result标签属性**

- name属性
和所使用的 action 方法返回值一样；

- type属性
  - 用于配置如何到路径中（转发或者重定向）
  - type 属性默认值是：做转发操作



## 四、Struts2常量配置

struts2框架，帮我们实现一部分功能，struts2里面有常量，在常量里面封装一部分功能；

- struts2默认的常量位置（记住）
 struts2-core.jar -> org.apache.struts2->default.properties

- 修改 struts2 默认常量值（一共三种方式）
  - 方式一：在 struts.xml 使用 `<constant>`中进行配置（常用的方式）
`<constant name = "struts.i18n.encoding" value = "UTF-8"></constant>` 
  - 方式二：在 src 下面创建 struts.properties，进行修改；
  将上面使用 name 和 value 属性配置的换成键值对格式就行，例如：
 `struts.i18n.encoding=UTF-8`； 
  - 方式三：在 web.xml 使用 `<init-param>`进行配置；该标签是 `<filter>`标签的子标签，例如：
```xml

<web-app>
  <display-name>Archetype Created Web Application</display-name>

  <filter>
    <filter-name>struts2</filter-name>
    <filter-class>org.apache.struts2.dispatcher.filter.StrutsPrepareAndExecuteFilter</filter-class>
    <init-param>
        <param-name>struts.i18n.encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
  </filter>

  <filter-mapping>
    <filter-name>struts2</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
</web-app>

```

- 介绍最常用常量
`struts.i18n.encoding = UTF-8`
（i18 是国际化的简写，18的含义是 i 和 n 直接还有 18个单词）

（1）表单提交数据到 action 里面，在 action 可以获取表单提交的数据，
（2）表单提交数据有中文，有乱码问题，解决：
    post提交直接设置编码
    get提交做编码转换
（3）**如果在 action 获取表单或者通过 post 方式提交中文，中文乱码问题 struts2 帮我们会解决了，不需要自己处理问题**



## 五、分模块开发
单独写配置文件，把配置文件引入到核心配置文件中
就是将上面的核心配置文件内容作为 helloAction.xml 单独配置文件，然后在 struts.xml 中在根标签中使用下面语句引入 helloAction.xml 文件；
```struts_xml
<struts>
    <!--引入 helloAction.xml-->
    <include file="helloAction.xml"></include>
</struts>
```


## 六、Action 编写方式
action 编写有三种方式

- 第一种：创建普通类，这个不继承任何类，不实现任何接口；
 就是上面实现的方法，类中的方法可以自己制定，只要配置 method 标签即可；

- 第二种：创建类，并且实现接口 Action
该接口仅仅定义了 execute() 方法和五个字符串常量；
  * SUCCESS : success, 代表成功
  * NONE : none, 代表页面不跳转
  * ERROR : error, 代表跳转到错误页面
  * INPUT : input, 数据校验的时候跳转的
  * LOGIN：login，用来跳转到登录页面
```java
package com.gjxaiou.action;
import com.opensymphony.xwork2.Action;

public class UserAction implements Action {
    @Override
    public String execute() throws Exception {
    // 这里的返回值一共三种：null ->对应 result 中 name 值为 NONE; ok 是自定义值，对应于 "ok"; SUCCESS -> 对应于 "SUCCESS",是 action 中提供的常量值； 
        // return null;
        // return "ok";
           return SUCCESS;
    }
}
```
下面为 action 中的方法
```java
package com.opensymphony.xwork2;

public interface Action {
    String SUCCESS = "success";
    String NONE = "none";
    String ERROR = "error";
    String INPUT = "input";
    String LOGIN = "login";

    String execute() throws Exception;
}
```

- 第三种：创建类，并且继承类 ActionSupport（一般使用）
```java
package com.gjxaiou.action;
import com.opensymphony.xwork2.ActionSupport;

public class PersonAction extends ActionSupport {
    @Override
    public String execute() throws Exception {
        return SUCCESS;
    }
}
```
进入 ActionSupport 可以看出，该类也是继承 Action：`public class ActionSupport implements Action, Validateable, ValidationAware, TextProvider, LocaleProvider, Serializable {`

 


## 七、访问 action 的方法（重点）
Action 作为框架的核心类，实现对用户请求的处理，Action 类称为业务逻辑控制器；一个 Action 类代表一次请求或者调用，每个请求的动作都对应于一个相应的 Action 类，一个 Action 类就是一个独立的工作单元； 有三种方式实现访问 action

- 第一种：使用 action 标签的 method 属性，在这个属性里面写要执行的 action 的方法；
- 第二种：使用通配符方式实现
- 第三种：动态访问实现（不用）

**演示错误**
如果 action 方法有返回值，在配置文件中没有配置（就是 result 的 name 属性没有对应的返回值），就会出现错误 404；

- 在 action 里面的方法有返回值，如果有返回值时候类型必须是String
- action 里面的方法可以没有返回值，没有返回值时候，在result标签不需要配置
  - 方法一：把 action 方法写成void
  - 方法二：设置返回值类型为 String，但是设置返回值为：`"none"`（建议使用）
```java
public String execute(){
    // return "none";
    return NONE;
}
```


### （一）使用 action 标签 method 属性

- 步骤一：创建 action，创建多个方法
```java
public class BookAction extends ActionSupport {
    public String add(){
        System.out.println("add......");
        return NONE;
    }    
    
    public String update(){
        System.out.println("update.......");
        return NONE;
    }
}
```

- 使用method配置
```xml
<struts>
    <package name="bookDemo" extends="struts-default" namespace="/">
        <action name="book" class="com.gjxaiou.action.BookAction" method="add"></action>
        <action name="book" class="com.gjxaiou.action.BookAction" method="update"></action>
    </package>
</struts>
```
缺陷：action 每个方法都需要配置，如果 action 里面有多个方法，配置很多的 action；可以采用下面的通配符实现；


### （二）使用通配符实现（重点）

==一定要配置下面这个==
`<global-allowed-methods>你想允许访问的方法，方法中间使用逗号隔开</global-allowed-methods>`
因为在 struts-default.xml (struts2-core.jar 下面)中的配置为：
`<global-allowed-methods>execute,input,back,cancel,browse,save,delete,list,index</global-allowed-methods>`

在 action 标签里面 name 属性，name 属性值里面写符号 `*`，这里写 `*` 是指不同的部分使用 `*` 进行通配；
`*` 理解： 表示匹配任意内容
  - 比如访问hello，* 可以匹配到
  - 比如访问add，* 可以匹配到
StudyAction.java 
```java
public class StudyAction extends ActionSupport {
    public String add(){
        System.out.println("add....");
        return "add";
    }

    public String update(){
        System.out.println("update....");
        return "update";
    }
}
```
studyAction.xml
```studyAction_xml
<struts>
    <package name="studyDemo" extends="struts-default" namespace="/">
        <action name="*" class="com.gjxaiou.action.StudyAction" method="{1}">
            <result name="add" >/{1}.jsp</result>
            <result name="update">/{1}.jsp</result>
            <allowed-methods>add,update</allowed-methods>
        </action>
    </package>
</struts>
```
然后在 struts.xml 中配置 studyAction.xml 即可；`<include file="studyAction.xml"></include>`

启动之后在 URL 后面加上 add，就会访问 add 方法，其余类似；
<action name 的值如果为：study*，可以在 URL 中输入：studyadd，就会匹配到 add 方法，后面的`{1} `的值就是 add；同样这里可以使用 多个 `*`，例如第二个 `*` 可以作为返回值匹配；

 增补示例：
```HelloWorldAction_java
public class HelloWorldAction extends ActionSupport {
    public String add() {
        return "add";
    }

    public String update() {
        return "update";
    }
```

```struts_xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts PUBLIC
    "-//Apache Software Foundation//DTD Struts Configuration 2.3//EN"
    "http://struts.apache.org/dtds/struts-2.3.dtd">
<struts>

    <package name="default" namespace="/" extends="struts-default">
        <action name="helloworld_*" method="{1}" class="com.imooc.action.HelloWorldAction">
            <result>/result.jsp</result>
            <result name="add">/{1}.jsp</result>
            <result name="update">/{1}.jsp</result>
        </action>
    </package>
</struts>
```

![执行机制](FrameDay05_1%20Struts2.resource/%E6%89%A7%E8%A1%8C%E6%9C%BA%E5%88%B6.png)

访问地址http://localhost:8888/HelloWorld/HelloWorld_update.action
name中的通配符*即代表update的值，也是{1}的值
通过传过来的参数表明method方法是class类中的update方法
在执行HelloWorldAction类中的update方法中，传出String的值为update代表result标签中的name属性，即知道用哪一个result方法返回update.jsp


**增补示例：**
假如我们有多个模块，每个模块使用一个Action，每个模块内有不同方法，假设有一个模块A，A中有方法B、C，我们希望访问A_B即可调用B方法的功能，使一种定义具有普适性。
对应的 struts.xml 为：
```struts_xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE struts PUBLIC
    "-//Apache Software Foundation//DTD Struts Configuration 2.5//EN"
    "http://struts.apache.org/dtds/struts-2.5.dtd">
    
    <struts>
	<package name="lf.blank.action" extends="struts-default" namespace="/">
	   <action name="*_*" class="lf.blank.action.{1}Action" method="{2}">
		  <result name="{2}">/{1}/{2}.jsp</result>
	   </action>
	</package>
   </struts>

```
**其中*是struts中的通配符 {数字}代表第几个通配的元素**

 

 

