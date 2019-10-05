# FrameDay05_3 Struts2



**上节内容**
- 在action获取表单提交数据
（1）使用ActionContext类获取
（2）使用ServletActionContext类获取
（3）接口注入

- 结果配置
（1）全局结果页面和局部结果页面
（2）result标签type属性
  - type属性值：
    - 默认值是dispatcher做转发
    - redirect做重定向
    - chain转发到action
    - redirectAction重定向到action

- struts2 提供获取表单数据方式
  - 属性封装
  - 模型驱动封装
  - 表达式封装

- struts2获取数据封装到集合中

- 使用模型驱动和属性封装注意问题：
不能同时使用对同一个表单进行数据操作

- 表达式封装和模型驱动比较
  - 相同点：可以把数据封装到实体类对象里面
  - 不同点：表达式封装可以封装到多个实体类对象里面



**今天内容**

- ognl概述
- ognl入门案例
- 什么是值栈
  - servlet和action区别
  - 值栈存储位置
    - 每个action对象里面有一个值栈对象
    - 值栈对象里面有action引用
- 如何获取值栈对象
- 值栈内部结构
  - root：list 集合
  - context：map 集合
- 向值栈放数据
  - s:debug 标签
  - 向值栈放字符串
  - 向值栈放对象
  - 向值栈放 list 集合
    - set 方法
    - push 方法
    - 定义变量，get 方法
- 从值栈获取数据
  * 从值栈获取字符串
  * 从值栈获取对象
  * 从值栈获取 list 集合
    - s:iterator 标签使用
- EL 表达式获取值栈数据
  - 增强 request 里面 getAttribute 方法
- ognl 表达式 #、% 使用

## 一、OGNL概述

- 之前web阶段，学习过EL表达式，**EL 表达式在 jsp 中获取域对象里面的值**；

- OGNL（Object-Graph Navigation Language）对象图导航语言，是一种表达式语言，这个表达式功能更加强大，
  - 一般用于在 struts2 里面操作值栈数据
  - 一般把 ognl 在struts2 操作：和struts2标签一起使用来操作值栈

-  OGNL 不是 struts2 的一部分，是一个单独的项目，经常和 struts2 一起使用
  - 使用ognl时候首先导入jar包，struts2里面已经提供jar包；

- Struts2 默认的表达式语言就是 OGNL, 它具有以下特点：
  - 支持对象 方法调用 。 例如： objN ame. methodName()。
  - 支持 类静态方法调用和值访问 ， 表达式的格式为＠［类全名（包括包路径）］＠［方法名|值名］。例如： @java.lang.String@format('foo %s','bar')。
  - 支持赋值操作和 表达式串联。
例如： price= 100, discount=O 8, calculatePrice(), 在方法中进行乘法计算会返回80。
  - 访问OGNL上下文(OGNL context)和ActionContext。
  - 操作集合对象 。

- OGNL 三大要素
  - 表达式：Expression：这是核心，所有 OGNL 操作都是针对表达式解析后进行的，表明要**做什么**；表达式就是一个带有语法含义的字符串，该字符串规定了操作的类型和操作的内容。
  - 根对象（Root）：Root 对象为 OGNL 的操作对象，它规定了 **对谁操作**，对象图的含义：以任意一个对象为根，通过 OGNL 可以访问与这个对象关联的其他对象；
  - Context 对象：设置了 Root 对象， OGNL 可以对 Root 对象进行取值或者写值操作，Root 对象所在的环境就是 OGNL 的上下文环境（Context）。上下文环境规定了 OGNL 的操作 **在哪里执行**，它是一个 Map 类型的对象。

## 二、OGNL入门案例

示例：使用ognl+struts2标签实现计算字符串长度

- 在java代码中，调用 `字符串.length();`实现；

-  使用struts2标签
  - 首先：之前使用jstl时候，导入jar包之外，在jsp页面中引入标签库，使用struts2标签时候，也需要在jsp中引入标签库；
`<%@taglib uri="/struts-tags" prefix="s"%>`
  - 然后使用struts2标签实现操作；
```ognlDemo_jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@taglib uri="/struts-tags" prefix="s"%>
<html>
<head>
    <title>This is a demo of ognl</title>
</head>
<body>
    <s:property value="'GJXAIOU'.length()"/>
</body>
</html>
```
- 最后在 web.xml 中向之前一样配置一样的过滤器即可；
使用 URL 访问，最后加上 ognlDemo.jsp 即可；页面显示结果为：`7`

## 三、值栈含义
值栈（ValueStack）是 Struts 的一个接口，OgnlValueStack 是 ValueStack 的实现类；
**当客户端发起一个请求，Struts2 架构会创建一个 action 示例，同时创建一个 OgnlValueStack 值栈实例，OgnlValueStack 贯穿整个 action 的生命周期。struts2中使用 OGNL将请求 Action 的参数封装为对象存储到值栈中，并通过 OGNL 表达式读取值栈中的对象属性值。**

- 之前在 web 阶段，在 servlet 里面进行操作，通过把数据放到域对象里面，在页面中使用 EL 表达式获取到，**域对象主要作用：在一定范围内存值和取值**；

- 在 struts2 里面本身提供一种存储机制，类似于域对象，是值栈，可以存值和取值，因此可以这样做：在 action 里面把数据放到值栈里面，在页面中获取到值栈数据；

- servlet 和 action 区别
    - Servlet：默认在第一次访问时候创建，只创建一次，单实例对象；
    - Action：访问时候创建，每次访问 action 时候，都会创建 action 对象，创建多次，多实例对象；

- 值栈存储位置
  - 每次访问action时候，都会创建action对象，
  - 在每个action对象里面都会有一个值栈对象（只有一个）
![值栈存储位置]($resource/%E5%80%BC%E6%A0%88%E5%AD%98%E5%82%A8%E4%BD%8D%E7%BD%AE.png)


## 四、获取值栈对象（UserAction中代码）

获取值栈对象有多种方式
- 常用方式：使用 ActionContext 类里面的方法得到值栈对象
```UserAction_java
package com.gjxaiou.action;

import com.opensymphony.xwork2.ActionContext;
import com.opensymphony.xwork2.ActionSupport;
import com.opensymphony.xwork2.util.ValueStack;

/**
 * @author GJXAIOU
 * @create 2019-10-03-11:02
 */
public class UserAction extends ActionSupport {
    @Override
    public String execute() throws Exception {
        // 1.获取 ActionContext 类的对象
        ActionContext actionContext = ActionContext.getContext();
        // 2.调用方法获取值栈对象
        ValueStack valueStack = actionContext.getValueStack();

        // 验证每个 Action 对象只有一个值栈对象
        ValueStack valueStack1 = actionContext.getValueStack();
        System.out.println(valueStack == valueStack1);

        return NONE;
    }
}
```
配置 struts.xml 后，每次访问该 Action 结果都是 true；
上面代码中同时验证了每个action对象中只有一个值栈对象；

## 五、值栈内部结构

因为栈是一种先进后出的结构；

值栈内容：
这里以示例通过在 .getValueStack() 添加断点，可以看出值栈中具体内容：
* valueStack = {OgnlValueStack@4512} 
  * root = {CompoundRoot@4559}  size = 2
  * context = {OgnlContext@4474}  size = 22
  * defaultType = null
  * overrides = null
  * ognlUtil = {OgnlUtil@4430} 
  * securityMemberAccess = {SecurityMemberAccess@4560} 
  * converter = {XWorkConverter@4561} 
  * devMode = true
  * logMissingProperties = false
- 第一部分 root，结构是本质上是 list 集合
一般操作都是 root 里面数据；
```java
public class CompoundRoot extends CopyOnWriteArrayList<Object> {
// ........
}

public class CopyOnWriteArrayList<E> implements List<E>, RandomAccess, Cloneable, Serializable {
// .......
}
```
// 因为 root 对应于 CompoundRoot，其最终实现了 List 接口；

- 第二部分 context，结构本质上是map集合
`public class OgnlContext extends Object implements Map`

context 中结构为：
key | value|
---|---|
request|request 对象引用|
session|HttpSession 对象引用|
application|ServletContext 对象引用|
parameters|传递相关的参数|
attr|如果在三个域对象中同时使用 setAttribute(“name”, value)放入同样名称的值，使用 attr 操作获取域对象中值时，获取的是域对象范围最小的那一个（request）的值|

**注：** Context 中存储的是对象的引用，其中 key 值是固定只有这么多；

### （一）通过调试查看值栈内部结构 

- struts2里面标签 `<s:debug></s:debug>`，使用这个标签可以查看值栈结构和存储值；

过程：访问action，执行action的方法有返回值，配置返回值到jsp页面中，在jsp页面中使用这个标签

代码过程：
```ValueStackAction_java
public class ValueStackAction extends ActionSupport {
    @Override
    public String execute() throws Exception {

        ActionContext actionContext = ActionContext.getContext();
        ValueStack valueStack = actionContext.getValueStack();
        return SUCCESS;
    }
}
```
配置 struts.xml
```xml
<struts>
    <constant name="struts.devMode" value="true"></constant>
    <package name="valueStack" extends="struts-default" namespace="/"> 
        <action name="valueStackAction" class="com.gjxaiou.action.ValueStackAction">
            <result name="success">/debug.jsp</result>
        </action>
    </package>
</struts>
```
 调试页面：debug.jsp
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@taglib uri="/struts-tags" prefix="s"%>
<html>
<head>
    <title>This is debug page</title>
</head>
<body>
    <s:debug></s:debug>
</body>
</html>
```

访问 URL：http://localhost:8080/StandardStrutsProject3_war_exploded/valueStackAction 点击超链接 `[Debug]` 可以看到结构，下面是 root 结构,Context 结构忽略

![Root结构]($resource/Root%E7%BB%93%E6%9E%84.png)

如图所示：在action没有做任何操作，栈顶元素就是 action引用；
- action对象里面有值栈对象
- 这里面看出值栈对象里面有 action，这仅仅是 action 引用，不是真正的 action对象；


## 六、向值栈放数据
### （一）放入普通数据
- 第一种：首先获取值栈对象，调用值栈对象里面的 set 方法；
```putValueAction_xml
public class PutValueAction extends ActionSupport {
    @Override
    public String execute() throws Exception {
        ActionContext context = ActionContext.getContext();
        ValueStack valueStack = context.getValueStack();

        // 方式一：使用值栈对象中的 set 方法
        valueStack.set("userName", "GJXAIOU");

        return SUCCESS;
    }
}
```
配置 struts.xml 文件和 debug.jsp 文件（和上面一样）；
![使用 set 放入数据 2]($resource/%E4%BD%BF%E7%94%A8%20set%20%E6%94%BE%E5%85%A5%E6%95%B0%E6%8D%AE%202.png)
这是栈顶元素了；root是 list结构，但是 list中可以放入一个 map集合，用来存储使用 set方法设置的值；

- 第二种：获取值栈对象，调用值栈对象里面的 push 方法
```putValueAction_xml
public class PutValueAction extends ActionSupport {
    @Override
    public String execute() throws Exception {
        ActionContext context = ActionContext.getContext();
        ValueStack valueStack = context.getValueStack();
        
        // 方式二：调用 push 方法
        valueStack.push("gjxaiou");

        return SUCCESS;
    }
}
```
配置 struts.xml 文件和 debug.jsp 文件（和上面一样）；
![调用 push 方法]($resource/%E8%B0%83%E7%94%A8%20push%20%E6%96%B9%E6%B3%95.png)
这是栈顶元素，上面的set是上一部分的；

- **第三种：在 action 定义变量，生成变量的 get 方法（最常用）**
**因为直接在值栈中放数据，不需要像上面的需要分配空间。**
```putValueStackAction_java
public class PutValueAction extends ActionSupport {

    // 方式三：定义变量然后生成对应的 get 方法
    private String userName;

    public String getUserName() {
        return userName;
    }

    @Override
    public String execute() throws Exception {
        // 在执行的方法中面向变量设置值
        userName = "GJXaiou";
        return SUCCESS;
    }
}
```

![第三种方式]($resource/%E7%AC%AC%E4%B8%89%E7%A7%8D%E6%96%B9%E5%BC%8F.png)


### （二）向值栈放对象

**实现步骤**
* 第一步  定义对象变量
* 第二步  生成变量的get方法
* 第三步  在执行的方法里面向对象中设置值

```User_java
@Setter
public class User {
    private String username;
    private String password;
    private String address;
}
```

```putObjectAction_java
public class putObjectAction extends ActionSupport {
    // 首先定义对象变量
    private User user = new User();
    // 然后生成对应的 get 方法
    public User getUser() {
        return user;
    }

    @Override
    public String execute() throws Exception {
        // 最后向值栈的 user 中放入数据
        user.setUsername("GJXAIOU");
        user.setPassword("gjxaiou");
        user.setAddress("南京");

        return SUCCESS;
    }
}
```
然后同上配置 struts.xml 和 debug.jsp
![值栈中放入对象]($resource/%E5%80%BC%E6%A0%88%E4%B8%AD%E6%94%BE%E5%85%A5%E5%AF%B9%E8%B1%A1.png)


### （三）向值栈放list集合

第一步  定义list集合变量
第二步  生成变量的get方法
第三步  在执行的方法里面向list集合设置值

```putListAction_java
public class putListAction extends ActionSupport{
    // 首先定义 List 变量
    private List<User> userList = new ArrayList<User>();
    // 同时实现 get 方法
    public List<User> getUserList() {
        return userList;
    }

    @Override
    public String execute() throws Exception {
        // 向 list 中设置值
        User user1 = new User();
        user1.setUsername("GJXAIOU");
        user1.setPassword("GJXAIOU");
        user1.setAddress("江苏");

        User user2 = new User();
        user2.setUsername("gjxaiou");
        user2.setPassword("gjxaiou");
        user2.setAddress("南京");

        userList.add(user1);
        userList.add(user2);
        return SUCCESS;
    }
}
```
其他配置参照上面；

![堆栈中放入 List]($resource/%E5%A0%86%E6%A0%88%E4%B8%AD%E6%94%BE%E5%85%A5%20List.png)


## 七、从值栈获取数据

使用struts2的标签 和 ognl表达式组合获取值栈数据；格式为：`<s:property value=”ognl表达式”/>`

### （一）获取字符串

首先使用上面的方法在值栈中放入字符串：
```java
public class PutValueAction extends ActionSupport {
    private String userName;

    public String getUserName() {
        return userName;
    }

    @Override
    public String execute() throws Exception {
        // 在执行的方法中面向变量设置值
        userName = "GJXaiou";
        return SUCCESS;
    }
}
```
然后在 debug.jsp 中使用struts2标签+ognl表达式获取
```debug_jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@taglib uri="/struts-tags" prefix="s"%>
<html>
<head>
    <title>This is debug page</title>
</head>
<body>

    获取 userName: <s:property value="userName"></s:property>

</body>
</html>
```

结果：`获取 userName: GJXaiou`

### （二）获取对象

**对象：User 必须有 get 方法**

- 向值栈放对象
```putObjectAction_java
public class putObjectAction extends ActionSupport {
    // 首先定义对象变量
    private User user = new User();
    // 然后生成对应的 get 方法
    public User getUser() {
        return user;
    }

    @Override
    public String execute() throws Exception {
        // 最后向值栈的 user 中放入数据
        user.setUsername("GJXAIOU");
        user.setPassword("gjxaiou");
        user.setAddress("南京");

        return SUCCESS;
    }
}
```

- 在页面中获取值
```debug_jsp
<body>
    获取对象的属性值：
    <br>
    username：<s:property value="user.username"/> <br>
    password：<s:property value="user.password"/><br>
    address： <s:property value="user.address"/> <br>

</body>
```

结果：
```java
获取对象的属性值：
username：GJXAIOU
password：gjxaiou
address： 南京
```


### （三）获取list集合

放置 List 代码（同上）
```java
public class putListAction extends ActionSupport{
    // 首先定义 List 变量
    private List<User> userList = new ArrayList<User>();
    // 同时实现 get 方法
    public List<User> getUserList() {
        return userList;
    }

    @Override
    public String execute() throws Exception {
        // 向 list 中设置值
        User user1 = new User();
        user1.setUsername("GJXAIOU");
        user1.setPassword("GJXAIOU");
        user1.setAddress("江苏");

        User user2 = new User();
        user2.setUsername("gjxaiou");
        user2.setPassword("gjxaiou");
        user2.setAddress("南京");

        userList.add(user1);
        userList.add(user2);
        return SUCCESS;
    }
}
```

- 第一种方式：
```debug_jsp
 获取 List 集合：方式一：<br>
    <s:property value="userList[0].username"></s:property>
    <s:property value="userList[0].password"></s:property>
    <s:property value="userList[0].address"></s:property>
    <br>
    <s:property value="userList[1].username"></s:property>
    <s:property value="userList[1].password"></s:property>
    <s:property value="userList[1].address"></s:property>
```

- 第二种方式：value 值为 List 对象名；
```debug_jsp
获取 List 集合：方式二：<br>
    <s:iterator value="userList">
        <!--遍历 list 得到 list 里面每一个 user 对象-->
        <s:property value="username"></s:property>
        <s:property value="password"></s:property>
        <s:property value="address"></s:property>
        <br>
    </s:iterator>
```

- 第三种方式：最常用
```debug_jsp
 获取 List 集合：方式三：<br>
    <s:iterator value="userList" var="user">
        <%--遍历值栈 List 集合，得到每一个 user 对象；
            机制：把每次遍历得到的 user 对象放在 context 里面(因为值是从 context 中取到的)，
            获取 context 里面数据使用 # 和 ognl 表达式；
        --%>
        <s:property value="#user.username"></s:property>
        <s:property value="#user.password"></s:property>
        <s:property value="#user.address"></s:property>
        <br>
    </s:iterator>
```


结果：
```java
获取 List 集合：方式一：
GJXAIOU GJXAIOU 江苏
gjxaiou gjxaiou 南京

获取 List 集合：方式二：
GJXAIOU GJXAIOU 江苏
gjxaiou gjxaiou 南京

获取 List 集合：方式三：
GJXAIOU GJXAIOU 江苏
gjxaiou gjxaiou 南京
```


### （四）其他操作

即使用 set 和 push 方法存入数据怎么取出；

- 使用set方法向值栈放数据，获取方法：
```java
// 放数据
stack.set("username", "GJXAIOU");
```

```jsp
<!--通过 set 放置的数据，根据名称进行获取-->
<s:property value = "username"/>
```


- 使用 push 方法向值栈放数据，获取方法：
```java
// 使用 push 方法放数据
stack.push("GJXAIOU");
```

（1）使用 push 方法设置值，没有名称，只有设置的值
（2）向值栈放数据，会把向值栈放数据存到数组里面，数组名称 top，根据数组获取值
```jsp
<!--获取 push 方法放入的值-->
<s:property value = "[0].top"/>
```


## 八、 EL表达式可以获取值栈数据的原因

- EL 表达式获取域对象值
首先加上标签头：`<%@ taglib uri="http://java.sun.com/jsp/jstl/core"prefix="c"%>`
然后使用 foreach
```jsp
<!-- 使用foreach标签+el表达式获取值栈list集合数据 -->
	<c:forEach items="${list }" var="user">
		${user.username }
		${user.password }
		${user.address }
		<br/>
	</c:forEach>
```

- 向域对象里面放值使用setAttribute方法，获取值使用getAttribute方法

- 底层增强 request 对象里面的方法：getAttribute 方法

（1）首先从request域获取值，如果获取到，直接返回
（2）如果从request域获取不到值，到值栈中把值获取出来，把值放到域对象里面


## 九、OGNL的#、%使用

### （一）#使用

- 可以使用#获取 context 里面数据
```debug_jsp
<s:iterator value="userList" var="user">
    <s:property value="#user.username"></s:property>
    <s:property value="#user.password"></s:property>
    <s:property value="#user.address"></s:property>
</s:iterator>
```

- 当是向 request 域中放值和取值的时候：
  - 向request域放值（见ContextAction.java）
```ContextAction_java
public class ContextAction extends ActionSupport {
    @override
	public String execute() throws Exception {	
		HttpServletRequest request = ServletActionContext.getRequest();
		request.setAttribute("req", "reqValue");
		return "success";
	}
}
```

  - 在页面中使用ognl获取
```jsp
<!--获取 context 里面的数据，在写 ognl 的时候，首先添加 #
    格式为：#context 的 key 名称.域对象名称；  （key 名称都是固定的，见上面）
    <s:property value = "#request.req"/>
-->
```

### （二）%使用

-  在struts2标签中表单标签
在struts2标签里面使用ognl表达式，如果直接在struts2表单标签里面使用ognl表达式不识别，只有%之后才会识别。
`<s:textfield name="username" value="%{#request.req}"></s:textfield>`相当于以前的：`<input type="text" name="name" value="${req }"/>`


