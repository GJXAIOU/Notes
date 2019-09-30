# FrameDay05_2 Struts2


**上节内容**
* struts2概述
* struts2入门案例
* struts2底层执行过程
* struts2相关配置
* struts2的action创建
* struts2的action方法访问

**今天内容**

- 结果页面配置
  - 全局结果页面
  - 局部结果页面
配置全局也配置局部，最终局部为准
  - result标签type属性
    - 默认值 dispatcher做转发
    - redirect做重定向
    - chain：转发到action
    - redirectAction：重定向到action

- 在action获取表单提交数据
  - 使用ActionContext类获取
  - 使用ServletActionContext类获取
  - 使用接口注入方式获取
使用ServletActionContext类操作域对象

-  struts2提供获取表单数据方式
    - 属性封装
定义变量，变量和表单输入项name属性值一样，生成get和set方法
    - 模型驱动封装（重点）
实现接口，实现接口里面的方法，表单输入项name属性值和实体类属性名称一样
    - 表达式封装
    - 表达式封装和模型驱动封装比较
      - 相同点：可以把数据封装到实体类对象里面
      - 不同点：表达式封装可以封装到不同的实体类里面

- struts2获取数据封装到集合中（会用）
  - 封装到list集合
  - 封装到map集合

- 案例-添加客户功能


## 一、结果页面配置

### （一）全局结果页面
 配置 result 标签，可以根据 action 方法的返回值到不同的路径里面；

示例：
创建两个action：StudentAction 和 TeacherAction，执行默认的方法 execute() 方法，让两个 action 的方法都返回 success，返回 success 之后，配置到同一个页面里面；

- 如果多个action，方法里面返回值相同的，到页面也是相同的，这个时候可以使用全局
结果页面配置，避免像下面的重复配置；

StudentAction.java  和 TeacherAction.java 代码如下：
```java
public class StudentAction extends ActionSupport {
    @Override
    public String execute(){
        return SUCCESS;
    }
}

public class TeacherAction extends ActionSupport{
    @Override
    public String execute(){
        return SUCCESS;
    }
}
```
struts 全局配置文件：
```struts_xml
<struts>
    <package name="global" extends="struts-default" namespace="/">
        <action name="student" class="com.gjxiou.action.StudentAction">
            <result name="success">/hello.jsp</result>
        </action>
        <action name="teacher" class="com.gjxiou.action.TeacherAction">
            <result name="success">/hello.jsp</result>
        </action>
    </package>
</struts>
```

- 改进方法：在package标签里面配置全局返回结果；
```struts_xml
<struts>
    <package name="global" extends="struts-default" namespace="/">
        <global-results>
            <result name = "success">/hello.jsp</result>
        </global-results>

        <action name="student" class="com.gjxiou.action.StudentAction"></action>
        <action name="teacher" class="com.gjxiou.action.TeacherAction"></action>
       
    </package>
</struts>
```


### （二）局部结果页面
 
为每个 action 单独配置 <result 标签就是局部结果返回页面；

- 配置全局页面，也配置了局部页面，最终**以局部配置为准**
```struts_xml
<struts>
    <package name="global" extends="struts-default" namespace="/">
        <global-results>
            <result name = "success">/hello.jsp</result>
        </global-results>

        <action name="student" class="com.gjxiou.action.StudentAction">
            <result name="success">/world.jsp</result>
        </action>
    </package>
</struts>
```
 上面使用 URL 访问 student 的时候，最终跳转到了 world.jsp 页面；

### （三）Result 标签的 type 属性
result标签里面除了name属性之外，还有一个属性 type属性

- type 属性：如何到路径里面（转发还是重定向）
- type 属性值
  - 默认值，做转发操作，值是 dispatcher
`<result name = "teacher" type = "dispatcher">/hello.jsp</result>` 
 访问的时候，页面 URL 不跳转到 hello.jsp，但是显示 hello.jsp 中的内容； 

  - 做重定向操作，值是 redirect
`<result name = "student" type = "redirect">/hello.jsp</result>`
 访问的时候，页面 URL 跳转到 hello.jsp， 同时显示 hello.jsp 中的内容； 

上面两个值dispatcher、redirect，这两个值一般针对到页面中配置；
- 配置到其他的action里面
  - chain：转发到其他 action，一般不用，因为缓存问题
`<result name = "success" type = "chain">class</result>`
这里的 class 是配置过的其他 action，它可以有其他操作，可以执行； 
  - redirectAction：重定向到其他 action
`<result name = "success" type = "redirectAction">class</result>` 
这里 Action 的访问名称就是上面 action 中name 属性配置的；


## 二、Action中获取表单提交数据
之前web阶段，提交表单到 servlet 里面，在 servlet 里面使用 request 对象里面的方法获取，例如： getParameter，getParameterMap；

在这里是提交表单到 action，但是 action 没有 request 对象，不能直接使用 request 对象；

- action 获取表单提交数据主要三种方式
  - 使用 ActionContext 类
  - 使用 ServletActionContext 类
  - 使用接口注入方式（不常用）

### （一）使用ActionContext类获取（推荐）
 获取当前线程的 ActionContext 对象方法：`static ActionContext getContext();`
- 因为方法不是静态的方法，需要创建ActionContext类的对象
- 这个ActionContext类对象不是new出来的；
 
**具体演示：**

- 首先创建表单，提交表单到action里面
```submit1_jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>This is submit.jsp</title>
</head>
<body>
    <form action = "${pageContext.request.contextPath}/form1.action" method="post">
        username: <input type="text" name="username"> <br>
        password: <input type="text" name="password"> <br>
        address:  <input type="text"  name="address"> <br>
        submit:   <input type="submit" name="提交">
    </form>
</body>
</html>
```
（2）在action使用ActionContext获取数据（来自文件 Form1Action），别忘了在 structs.xml中配置
 

### （二）使用ServletActionContext类获取（常用）
 
* `static HttpServletRequest getReques();` //获取web应用的HttpServletRequest对象
* `static HttpServletResponse getResponse();` //获取web应用的HttpServletResponse对象
* `static ServletContext getServletContext();` //获取web'应用的ServletContext对象
* `static PageContext getPageContext();` //获取web应用的pageContext对象

- 首先调用类里面静态方法，得到request对象
```Form2Action_java
public class Form2Action extends ActionSupport {
    @Override
    public String execute() throws Exception {
        // 使用 ServletActionContext 类获取 request 对象
        HttpServletRequest request = ServletActionContext.getRequest();
        
        // 调用 request 中方法获取结果
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        String address = request.getParameter("address");
        System.out.println(username + ":" + password + ":" + address);

        return NONE;
    }
}
```
 jsp 页面和上面的一样，仅仅是将 form1.action 修改为 form2.action

### （三） 使用接口注入（了解）

- 让 action 实现接口，为了得到 request 对象
```form3Action_java
public class Form3Action extends ActionSupport implements ServletRequestAware {
    private HttpServletRequest httpServletRequest;
    @Override
    public void setServletRequest(HttpServletRequest httpServletRequest) {
        this.httpServletRequest = httpServletRequest;
    }

    @Override
    public String execute() throws Exception {
        httpServletRequest.getParameter("");
        return NONE;
    }
}
```
 

### （四）在 action 操作域对象

- servlet 中有 request、session、servletContext 域对象；

使用ServletActionContext类操作
```Form2Action_java
public class Form2Action extends ActionSupport {
    /**
     *  在 action 中操作域对象
     * @return
     * @throws Exception
     */
    @Override
    public String execute() throws Exception {
        // request 域
        HttpServletRequest request = ServletActionContext.getRequest();
        request.setAttribute("req", "reqValue");

        // session 域
        HttpSession session = request.getSession();
        session.setAttribute("sess", "sessValue");
        
        // ServletContext 域
        ServletContext servletContext = ServletActionContext.getServletContext();
        session.setAttribute("contextname", "contextValue");
        return NONE;
    }
}
```
 

## 三、Struts2 封装获取表单数据方式
**原始方式获取表单封装到实体类对象**

首先创建实体类：User，生成对应的 get、set方法，然后在 form4Demo...写下面的代码，同时在 struts.xml 中配置 form4Demo......（）
```User_java
package com.gjxaiou.entity;

import lombok.*;

/**
 * @author GJXAIOU
 * @create 2019-09-30-14:25
 */
@Getter
@Setter
@ToString
@NoArgsConstructor
@AllArgsConstructor
public class User {
    private String username;
    private String password;
    private String address;
}
```

```Form4Action_java
public class Form4Action extends ActionSupport {
    @Override
    public String execute() throws Exception{
        // 首先获取表单数据
        HttpServletRequest request = ServletActionContext.getRequest();
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        String address = request.getParameter("address");

        // 封装到实体类对象里面
        User user = new User();
        user.setUsername(username);
        user.setPassword(password);
        user.setAddress(address);
        System.out.println(user);
        return NONE;
    }
}
```
然后在 struts.xml 中配置上面即可；


属性封装（会用）
1 直接把表单提交属性封装到action的属性里面

2 实现步骤（见 DataDemo1Action.java,同样需要在配置文件中配置）
（1）在action成员变量位置定义变量
- 变量名称和表单输入项的name属性值一样
（2）生成变量的set方法（把set和get方法都写出来）
 

3 使用属性封装获取表单数据到属性里面，不能把数据直接封装到实体类对象里面，放入还是得写上面的新建对象，然后使用 setParm....

模型驱动封装（重点）
1 使用模型驱动方式，可以直接把表单数据封装到实体类对象里面 -> DataDemo2Action.java

2 实现步骤
（1）action实现接口 ModelDriven
 
（2）实现接口里面的方法 getModel方法
- 把创建对象返回

（3）在action里面创建实体类对象
 
 

3 使用模型驱动和属性封装注意问题：
（1）在一个action中，获取表单数据可以属性封装，使用模型驱动封装，
不能同时使用属性封装和模型驱动封装获取同一个表单数据
如果同时使用，只会执行模型驱动

表达式封装（会用）- dateDemo3
1 实现过程
（1）使用表达式封装可以把表单数据封装到实体类对象里面

第一步 在action里面声明实体类（也需要实体类）
第二步 生成实体类变量的set和get方法
 
第三步 在表单输入项的name属性值里面写表达式形式
 
2 把表达式封装归类到属性封装里面

比较表达式封装和模型驱动封装
1 使用表达式封装和模型驱动封装都可以把数据封装到实体类对象里面

2 不同点：
（1）使用模型驱动只能把数据封装到一个实体类对象里面
- 在一个action里面不能使用模型驱动把数据封装到不同的实体类对象里面

（2）使用表达式封装可以把数据封装到不同的实体类对象里面
 
 
获取表单数据封装到集合里面
封装数据到List集合
ListAction.java 和 list.jsp
第一步 在action声明List
第二步 生成list变量的set和get方法
 
第三步 在表单输入项里面写表达式
 

封装数据到Map集合
mapAction.java 和 map.jsp
第一步 声明map集合
第二步 生成get和set方法
 
第三步 在表单输入项的name属性值里面写表达式
 

案例-添加客户功能
1 模型驱动获取表单数据
 
2 在hibernate实现
 
3 添加之后到列表页面中，让列表的action执行一次
 

完成任务
1 客户列表功能

2 添加客户功能

3 修改客户

4 删除客户




