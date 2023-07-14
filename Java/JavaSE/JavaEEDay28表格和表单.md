---
tags:
- 表格
- 表单
- html
---

# JavaDay28 HTML 表格和表单
@toc

table仅仅只是用来布局用的,form则是用来封装数据的,通常是 form 里面包含 table；
## 一、复习
### 表单
- `<from></from>`
  - action:提交数据的地址，即 URL；当提交表单时，表单数据会提交到名为 "XXX"（action 后面的参数） 的页面
    - URL  ：向何处发送表单数据。
          可能的值：
      *   绝对 URL - 指向其他站点（比如 src="www.example.com/example.htm"）
      *   相对 URL - 指向站点内的文件（比如 src="example.htm"）

  - method: 提交数据的方式，有 GET 和 POST
     -  GET: 是通过 URL 明文传递；
      - POST：算是一个隐蔽传递，适合于传递需要加密的数据，但是实际上并没有给数据加密；



- `<input></input>`
  
  - type:
    - text:可视化文本，是缺省属性；
    - password: 密文
    - radio：**单选**，要求单选对话框的 name 属性要一致；
    - checkbox: **多选**，这里建议多选对话框的 name 属性用[]结尾；
    - date: 日期
    - file:文件上传
    - hidden:隐藏传递数据，适用于文件上传时候，先行告知服务器数据大小；
    - submit:提交数据
    - button：按钮，这里需要配合 JS 使用；
  - reset：重置；
  
- `<select></select>` **下拉菜单框**

- `<textarea rows = “” cols=“”></textarea>`

  单选框和多选框默认选择是使用 checked;
  下拉菜单默认选择是 selected;

- <p align = “ ”> </p> 创建一个段落；其中的“”值可以有：left、center、right 表示三种对齐方式；


## 二、表格
tr 表示行
th 表示表头 ：默认居中且加粗
td 表示一行里面的一个单元格

- 修饰：
一共三种修饰：表格整体修饰、行修饰、每个单元格修饰；

整体相对标准格式：
```html
<table border = " " width = " " bordercolor = " " style="border-collapse: collapse;"> 
<!--设置线宽和表格长度和线的颜色和线的样式-->
  <tr>   <!--表示一行-->
    <th colspan="2">这里是表头</th> <!--表示该单元格占两列-->
    <th rowspan="2">这里是表头</th> <!--表示该单元格占两行-->
  </tr>

  <tr align = " "> <!--该行所有内容居中显示-->
    <td width = " ">这里是相当于第一行第一格内容 </td>
    <td>这里是相当于第一行第二格内容</td>
  </tr>
</table>

```
显示效果：
![表格示例]($resource/%E8%A1%A8%E6%A0%BC%E7%A4%BA%E4%BE%8B.png)

- 常用属性设置练习
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
	</head>
	<body>
		<!--p标签为表示段落，align设置对齐方式-->
		<p align="center"> <font size="32" face="黑体">学员表</font> </p>
		
		<!-- 设置table整体属性，倒数第二个表示只是单线 -->
		<table border="1px" bordercolor="red" width="600"
			style="border-collapse: collapse;"
			align="center">
			<!-- 下面tr表示行，th表示表头，其中表头是默认加粗居中 -->
			<tr>
				<!-- 该行一种两大列，第一列占了3个单列，第二列占了2个单列 -->
				<th colspan="3">学员基本信息</th>
				<th colspan="2">成绩</th>
			</tr>
			<tr>
				<!-- 可以设置每个单元格的宽度 -->
				<th width="100px">姓名</th>
				<th>性别</th>
				<th>专业</th>
				<th>课程</th>
				<th>分数</th>
			</tr>
			<tr align="center">
				<td>小凯</td>
				<td>男</td>
				<td rowspan="2">计算机</td>
				<td rowspan="2">JavaEE</td>
				<td>86</td>
			</tr>
			<tr align="center">
				<td>小珊</td>
				<td>女</td>
				<td>98</td>
			</tr>
		</table>
	</body>
</html>
```
显示效果：
![属性练习结果]($resource/%E5%B1%9E%E6%80%A7%E7%BB%83%E4%B9%A0%E7%BB%93%E6%9E%9C.png)

## 三、表单

- 在前端页面中，通过input标签想要发送数据给后台，有两个必要条件
	1. 该标签必须有name属性
	2. 当前标签必须放到form表单里面

- form表单中属性解释：
  - action表示提交的目标地址
  - method表示提交方式 GET/POST DELETE    	
    - GET是通过URL传递参数的，传递参数之后，URL中会明文显示参数
    - POST是隐含传递参数，可以使用浏览器里面的开发者工具->netword->找到发送给哪一个文件->from data
  

整体标准格式为：
```html
<form action="2.html" method="post">
	<p><b>这里是文本输入框</b></p>
	 <!--参数含义：  输入类型   name属性   placeholder表示默认输入框中显示的值  可输入的最大长度-->
  用户名：<input type= "text" name = "usename" placeholder = "请输入用户名" maxlength="10"/> <br />  
  密码:<input type="password" name="password" placeholder="请输入密码"/> <br />
  
  <p><b>这里是单选框</b></p>
  <!--参数含义：radio表示单选  单选的两者的name值必须相同 value值为存储时候代替值
  	checked表示默认选择
	id和<label for="另一个标签的ID"></label>配合是为了点击：男 字也可以选择
	当前情况下是为了友好性-->
  性别：<input type="radio" id = "nan" name="sex" value="0" checked/>
		<label for="nan"> 男 </label>
   
		<input type="radio" id = "nv" name="sex" value="1" checked/>
		<label for="nv"> 女 </label> <br />
		
	<p><b>这里是时间选择框</b></p>
	出生年月：<input type="date" name = "birthday"> <br />
	
	<p><strong>这里是多选框</strong></p>
	爱好：<input type="checkbox" name = "like[]" value="football" checked/>足球
		  <input type="checkbox" name = "like[]" value="music" >音乐
		  <input type="checkbox" name = "like[]" value="swimming" > 游泳 <br />
		  
	<p><strong>这里是下拉选择框</strong></p>
	籍贯：<select name = "city">
			<option value="jiangsu">南京 </option>
			<option value="shanghai" selected>上海 </option>
			<option value="hangzhou">杭州 </option>
		  </select>   <br/>
		  
	<p><strong>这里是插入图片</strong></p>
	<input type="file" name = "icon"/> <br/>
	
	<p><strong>这里是输入文本框</strong></p>
	<textarea name="desc" cols="100" rows="10"></textarea> <br />
        	<input type="hidden" name="size" value="200"/>
        	
        	<input type="button" value="这个能不能用？？" /><br/>
        	<input type="reset" value="重置" /> <br/>
        	<button>看看这个是啥</button><br/>
        	<input type="submit" value="注册"/><br />
</form>

```

![表单示例结果]($resource/%E8%A1%A8%E5%8D%95%E7%A4%BA%E4%BE%8B%E7%BB%93%E6%9E%9C.png)



## 四、 页中页
```html
<body>
	<a href="http://www.163.com" target="iframe1">网易</a>
	<a href="http://www.taobao.com" target="iframe1">淘宝</a>
	<a href="http://www.jd.com" target="iframe1">京东</a>
	<iframe name="iframe1" src="http://www.baidu.com"
		width="800" height="400"></iframe>
</body>
```
![页中页]($resource/%E9%A1%B5%E4%B8%AD%E9%A1%B5.jpg)


### 超级页中页：将整个页面分割成几个页面
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>超级页中页</title>
	</head>
	<frameset rows="100, *" frameborder="1"> 
		<frame src="http://www.baidu.com"/>
		
		<frameset cols="30%, *">
			<frame src="http://www.taobao.com"/>
			<frame src="http://www.jd.com"/>
		</frameset>
	</frameset>
</html>

```
![3]($resource/3.jpg)

## div 知识：
一个标签如果单独成行，这表示这个标签是一个块标签
如果不是单独一行，表示非块标签

<span> 标签被用来组合文档中的行内元素。
使用 <span> 来组合行内元素，以便通过样式来格式化它们。


- HTML <div> 元素 ：div是块级元素， 实际上就是一个区域， 主要用于容纳其他标签。 默认的display属性是block
  - HTML <div> 元素是块级元素，它是可用于组合其他 HTML 元素的容器。
  - <div> 元素没有特定的含义。除此之外，由于它属于块级元素，浏览器会在其前后显示折行。
  - 如果与 CSS 一同使用，<div> 元素可用于对大的内容块设置样式属性。
  - <div> 元素的另一个常见的用途是文档布局。它取代了使用表格定义布局的老式方法 ，使用<table> 元素进行文档布局不是表格的正确用法。<table> 元素的作用是显示表格化的数据。

- HTML <span> 元素 ：span是行内元素， 主要用于容纳文字。 默认的display属性是inline
HTML <span> 元素是内联元素，可用作文本的容器。
<span> 元素也没有特定的含义。
当与 CSS 一同使用时，<span> 元素可用于为部分文本设置样式属性。
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>块元素</title>
	</head>
	<body>
	<div style="background: deepskyblue; width: 100px; height: 100px;"></div>
	<div style="background: orangered; width: 100px; height: 100px;"></div>
	<div style="background: forestgreen; width: 100px; height: 100px;"></div>
	<span>你好</span><span>我不好</span>
	</body>
</html>

```
![块元素]($resource/%E5%9D%97%E5%85%83%E7%B4%A0.jpg)




## CSS :层叠样式表
英文全称：Cascading Style Sheet
HTML 为骨架，CSS 为润色，提高页面的观赏性； JavaScript 让页面有更好的展示方式，动态特效；

- CSS 的三种声明方式：
  - 行内样式表:直接在 style 标签中写；
		使用标签的style属性，直接在 style 标签中定义； CSS样式都是以键值对形式表示的，键值对用:连接；如果存在多个CSS样式，用分号隔开。
  - 内联样式表 ：在 style 标签中声明，然后文件中写 CSS 文件；
借助于在head标签内定义style标签声明CSS样式， 在style标签内就可以认为是一个CSS文件，这里的注释方式采用CSS文件的方式
   CSS修饰的样式是使用大括号包含的，在大括号之前有一个标记，这个标记表示修饰的是哪一部分内容
  - 外联样式表：☆☆☆ 在 style 标签中声明，然后在其他文件中写 CSS 文件；
是借助于head标签里面的link标签来连接在html 文件之外的样式表
rel="stylesheet" 样式表
type="text/css" 可视化的CSS文件
href="CSS文件所在路径" 可以是URL

外联样式表的好处：
	1. 多个页面可以同时使用一个CSS样式表：例如：淘宝的搜索详情页
	2. 提高加载效率，因为都是同一张CSS样式，加载一次之后，可以保存在
	本地缓存中，方便下一次使用
	3. 可以加快页面的加载，也可以节省用户的流量和服务器带宽
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>CSS的声明方式</title>
		<!--方式2:内联样式表
        		借助于在head标签内定义style标签声明CSS样式
        -->
        <style type="text/css">

        	p {
        		background-color: black;
        		color: white;
        	}
        </style>
        
          <!--方式3：外联样式表【重点】-->
          
        <link rel="stylesheet" type="text/css" href="01-usemethod.css"/>
	</head>
	
	<!--方式1：行内样式表-->
	
	<body style="background-color:#FFCE44;">
		<div style="background-color: #064B88; width: 200px; height: 200px;">
		</div>
		<p>
			最穷不过要饭，不死终会出头
		</p>
		<span>
			好好学习，天天向上
		</span>
	</body>
</html>

```

外联样式表：01-usemethod.css
```css
span {
	background-color: black;
	color: yellow;
}

```
最终显示结果：
![CSS样式表]($resource/CSS%E6%A0%B7%E5%BC%8F%E8%A1%A8.jpg)

