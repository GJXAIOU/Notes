# JavaEEDay44 EL 表达式

## EL表达式

EL: Expression Language
​**原则：JSP 里面尽量减少，甚至不使用 Java 代码；**
EL表达式是用来替换JSP页面中的JSP脚本
​
EL表达式的基本语法规范: `${标识符}`
​
1 . 获取数据
 可以从当前JSP域对象中，找出对应的属性名，获取属性值
 可以获取域对象中的数据，同时可以获取Java对象数据

```jsp
<%
	//JSP脚本里面写的都是Java代码
	Student stu = new Student();
	stu.setId(10);
	stu.setName("Jack");
	
	//把一个类对象，放入了request域对象中
	request.setAttribute("s", stu);
%>

	<%-- 这里使用JSP表达式 --%>
	<%=((Student)request.getAttribute("s")).getName() %><br>

	<%-- EL表达式可以从JSP页面的域对象中（这里是request域对象），获取对应的属性名，并且通过属性名得到
  	 属性值，并且展示在页面当中
  	 	类似于下面这句：
  	 	<%
  	 		out.write((((Student)request.getAttribute("s")).toString)) 
  	 	%>
  	  --%>
	${s }
	<br />

	<%--
		pageContext.findAttribute()表示在当前页面中找出域对象里面对应的属性名
		查找顺序为：pageContext -> request -> session -> application
	 --%>
	<%
		request.setAttribute("name", "lily");
		pageContext.setAttribute("name", "Tom");
	 %>
	${name }  <%--	值为 Tom--%>
	<br>
	<%-- 如果就是想要从指定的域对象里面拿出我想要的数据
	 		requestScope
	 		sessionScope
	 		applicationScope
	 		pageScope
	 	--%>
	${requestScope.name }  <%--	值为 lily--%>
	<br>

	<%
	 	int i = 10;
	  %>
	<%-- 因为i是一个普通的Java变量，不是放到域对象里面的一个属性值，所以获取不到i值--%>
	${i }
	<br>
	
	<%-- 这里是通过setter和getter方法来修改域对象里面保存数据的内容 --%>
	${s.getName() } <br>   <%--	值为 Jack--%>
	${s.setName("Rose") }<br>
	${s } <br>   <%--	值为 Student[id = 10, name = Rose]--%>
	
	<%-- 上面的代码可以使用下面形式：
	Tomcat8可以这样使用，相对于调用了对象的setter和getter方法
	当时在Tomcat7不能这么使用 --%>
	
	${s.name = "David" } <br>
	${s } <br>
	
	${requestScope.name = "233333" } <br >
```


 获取集合中数据
```jsp
<%
        // 这里使用Java代码
    	List<String> list = new ArrayList<String>();
    	list.add("周日");
    	list.add("周一");
    	request.setAttribute("list", list);
    	
    	Map<String, String> map = new HashMap<String, String>();
    	map.put("name", "123");
    	map.put("age", "10");
    	request.setAttribute("map", map);
     %>
     ${list } <br>  <%--[周日，周一]--%>
     ${list[0] } <br> <%--[周日]--%>
     
     ${map["name"] } <br> <%--[123]--%>
     ${map.name } <br>   <%--[123]--%>
```


2 . EL表达式使用算术运算符，关系运算符，逻辑运算符和三目运算符
```jsp
	<%-- 数学运算 --%>
	${100 + 50 }
	<br>
	<%-- 关系运算符 --%>
	${10 > 3 }
	<br> ${10 < 3 }
	<br> ${10 == 3 }
	<br> ${10 != 3 }
	<br> ${10 >= 3 }
	<br> ${10 <= 3 }
	<br>
	<%-- &gt;大于 &lt;小于 &nbsp; --%>
	${10 gt 3 }
	<br> ${10 lt 3 }
	<br>
	<hr>
	<%-- 逻辑运算符 --%>
	${true && true }
	<br> ${true && false }
	<br> ${true || false }
	<br> ${false || false }
	<br> ${!true }
	<br> ${not false }
	<br> ${false and true }
	<br> ${false or true }
	<br>
	<hr>
	<%
		String str = "123";
		request.setAttribute("str", str);
	 %>
	<%-- 判断这个字符串的首地址是不是空，或者是不是一个空串 --%>
	${str == null || str == "" }<br> 
	${empty str } <br>
	${not empty str } <br>
	
	<%-- 三目运算符 --%>
	${empty str ? 100 : 1000 } <br>
```


3 . 内置对象
 EL表示可以使用内置(隐式)对象

### EL表达式对照JSP内置对象表

| EL表达式 | 对照JSP |
| --- | --- |
| pageContext | JSP页面中的pageContext对象，可以获取其他8大对象 |
| pageScope | 代表page域中保存的数据，Map对象 |
| requestScope | 代表request域中保存的数据，Map对象 |
| sessionScope | 代表session域中保存的数据，Map对象 |
| applicationScope | 代表application域中保存的数据，Map对象 |
| param | 表示保存所有页面请求参数的Map对象 |
| paramValues | 一般对照页面多选操作的Map对象，得到的其实是一个String[]数组 |
| header | 对应所有Http请求头数据 |
| headerValues | 对应所有Http请求头数据，对应的是String[]数组 |
| cookie | 对应cookie的Map对象 |
| initParam | 保存所有WEB应用的初始化参数Map对象 |

```jsp
<%--可以获取请求参数--%>
${param["id"] } <br>  <%--值为：12--%>
${param.name }<br> <%--值为：匿名君--%>
<%--
	XXX/04EL.jsp?id=12&name=匿名君&hobby[]=篮球&hobby[]=足球
 --%>
${paramValues["hobby[]"][0] }<br>
${paramValues["hobby[]"][1] }<br>

${cookie["JSESSIONID"] }<br>
${cookie["JSESSIONID"].name }<br>
${cookie["JSESSIONID"].value }<br>
${cookie.JSESSIONID.value }<br> <%-- 该句和上一句含义一样--%>

${pageContext.request.contextPath }<br>

<form action="<%=request.getContextPath() %>/01EL.jsp"></form>
<form action="${pageContext.request.contextPath }/01EL.jsp"></form>
```




## JSP标签

1 . 内置动作标签(了解)
```jsp
	<%-- 下面语句含义：
  		1.创建指定Class的类对象
  		2.命名为id对应的属性stu，对象名
  		3.放入到域对象中，因为不放入域对象中，不能通过EL表达式拿到
  	 --%>
    <jsp:useBean id="stu" class="com.qfedu.servlet.Student"></jsp:useBean>
  	
  	<%--下面语句含义：
  		设置对象的属性
  		name 表示对象的名字 
  		property 要设置的属性名
  		value 设置属性名需要的参数
  		这里相对于调用指定对象的setXXX方法
  	 --%>
  	<jsp:setProperty property="name" name="stu" value="Tom"/>

	${stu.name } <br>   <%--值为Tom--%>
	
  	<jsp:getProperty property="name" name="stu"/> <br>  <%--值为Tom--%>
  
  	
  	<%-- 
  		转发
  		因为在WEB-INF目录下的资源不能直接访问，需要通过转发的机制来完成
  	 --%>
  	<jsp:forward page="WEB-INF/test.jsp"></jsp:forward>
```

2 . JSTL标签
 JSP Standard Tag Library JSP标准标签库 
 这是一个不断完善的JSP标签库，是由apache组织完不断维护的

 a. 主要包含
 核心标签  `c:`
 JSTL函数  `fn:`
 格式化标签 `fmt:`
 数据库标签
 XML标签

 b. 使用过程
 在JSP文件中导入相关标签库
 <%@taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>

 c. 核心标签库 :用的不多
 c:set  c:out
```jsp
<%-- 
		下面语句作用：定义一个变量，并且把它放到域对象中
		var 相对于变量名
		value 变量的值
		scope 指定域
	 --%>
	<c:set var="name" value="Tom"></c:set>
	<c:set var="age" value="10" scope="request"></c:set>
	<c:set var="age" value="20" scope="session"></c:set>
	<%-- 设置null值，使用EL表达式表示null值 --%>
	<c:set var="name" value="${null }"></c:set>
	${name } <br>
	${requestScope.age } <br>
	
	<%
		Student stu = new Student();
		request.setAttribute("s", stu);
	 %>
	 <%-- 
	 	target: 需要设置的对象 ${s }是用EL表达式把对象拿出来
	 	property: 要设置的属性名
	 	value: 设置的数值
	  --%>
	 <c:set target="${s }" property="name" value="Jack"></c:set>
	 ${s.name } <br>
	 
	 <%--
	 	value 要输出的值
	 	default 如果值不存在，对应的展示效果
	 	escapeXml 是否让HTML标签起作用，false是起作用，true是不起作用
	  --%>
		<c:out value = "${name}" default = "没有该对象的时候默认显示值"></c:out>
	 <c:out value="${NDY }" default="<h3>真的没有啊~~~</h3>" escapeXml="false"></c:out>
```

 判断，循环 

 重定向

 URL参数设置

```jsp
  	<%--
  		这里是一个条件判断
  		test 中给出条件 这里用到了EL表达式
  	 --%>
    <c:if test="${10 > 5 }">
    	<h2>YES</h2>
    </c:if>
    
    <c:set var="sex" value="女"></c:set>
    <c:if test="${sex == '男' }">
    	<input type="radio" checked="checked"> 男 
    	<input type="radio"> 女
    </c:if>
    <c:if test="${sex == '女' }">
    	<input type="radio"> 男 
    	<input type="radio" checked="checked"> 女
    </c:if>
    
    
	
	<c:set var="score" value="80"></c:set>
    <%-- 多条件判断 --%>
    <c:choose>
    	<c:when test="${score >= 90 && score <= 100}">
    		<h3>优秀</h3>
    	</c:when>
    	<c:when test="${score >= 80}">
    		<h3>良好</h3>
    	</c:when>
    	<c:when test="${score >= 70}">
    		<h3>中等</h3>
    	</c:when>
    	<c:when test="${score >= 60}">
    		<h3>及格</h3>
    	</c:when>
    	<c:when test="${score < 60}">
    		<h3>叫家长</h3>
    	</c:when>
    </c:choose>
    <%
    	List<String> list = new ArrayList<String>();
    	for (int i = 0; i < 10; i++) {
    		list.add("Test" + i);
    	}
    	request.setAttribute("list", list);
     %>
     <%-- 
     	                       遍历
     	items 表示要遍历的对象
     	var 表示遍历出来的每一个对象的临时名字，这个相当于在pageContaxt域对象中，创建了一个s变量
     	begin / end 从哪里开始到哪里结束
     	varStatus:表示遍历的状态：
     			count：表示遍历出来的数据个数
     			index：表示遍历出来的数据对应的下标号
     			begin：从哪个数据开始
     			end：  结束下标是哪一个
     			first：是否是开始
     			last： 是否是结尾
      --%>
     <c:forEach items="${requestScope.list }" var="s" varStatus="vs" 
     begin="2" end="8" step="2">
     	${s } &nbsp; ${vs.count } &nbsp; ${vs.index } &nbsp; 
     	${vs.first } &nbsp; ${vs.last } &nbsp; <br />
     </c:forEach>
     
     <%-- 
     	遍历一些特殊写法的字符串
     	delims 用来拆分字符串的分隔符
      --%>
     <c:forTokens items="java-c-php-ios-python" delims="-" var="str">
     	${str } <br>
     </c:forTokens>
     
     <%-- 
     	重定向
      --%>
     <%-- <c:redirect url="http://www.baidu.com"></c:redirect>--%>
     
     <%-- 
     	设置URL
     	最终效果：*****/04EL.jsp?id=20&name=Tom	
      --%>
     <c:url value="/04EL.jsp" var="url">
     	<c:param name="id" value="20"></c:param>
     	<c:param name="name" value="Tom"></c:param>
     </c:url>
     
     <a href="${url }">04EL.jsp</a>
```


格式化
```jsp
 <%--
    	数字的格式化
    		type 数字类型
    			number 数字
    			currency 货币
    			percent 百分比
    		pattern 指定格式
    		currencySymbol 货币符号
     --%>
     <fmt:formatNumber value="12345.12345" type="number"
     	pattern="###,###.##"
     ></fmt:formatNumber> <br>
     <fmt:formatNumber value="12345.12" type="currency" currencySymbol="￥"
     ></fmt:formatNumber> <br>
     <fmt:formatNumber value="0.96689" type="percent"
     minFractionDigits="2"></fmt:formatNumber> <br>
     
      <%
      	Date date = new Date();
      	request.setAttribute("date", date);
       %>
       
     <%--
     	日期格式化
     	type
     		date 年月日
     		time 时分秒
     		both 两者都要
     	dateStyle 日期的风格
     	timeStyle 时间的风格
     	pattern 样式
      --%>
       <fmt:formatDate value="${date }" type="date"/> <br>
       <fmt:formatDate value="${date }" type="time"/> <br>
       <fmt:formatDate value="${date }" type="both"/> <br>
       <fmt:formatDate value="${date }" type="both" dateStyle="long"
       timeStyle="long"/> <br>
       <fmt:formatDate value="${date }" type="both" 
       pattern="YYYY/MM/dd HH:mm:ss.SSS"/> <br>
```
