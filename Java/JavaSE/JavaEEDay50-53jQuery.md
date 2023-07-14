---
style: summer
tags : [jQuery]
---

# JavaEEDay50jQuery
避免重复的造轮子；

## 一、常识

- jQuery 是一个快速、简洁的 JavaScript 框架
- jQuery 是一个快速、小、功能丰富的 JavaScript 库，它使注入 HTML 文档遍历和操作，事件处理、动画和 Ajax 更简单和易于使用的 API，跨越多种浏览器。结合的通用性和可拓展性。
- 对 js 代码进行封装简化，提供 js 常用功能，比如 html 元素的遍历、事件的处理、动画等等；
- 效果：使用 js 的时候不需要书写原生的 js 代码，而是使用 jquery 

- `$()`使用：
  - `$(function)`相当于：`$(document).ready(function)`的缩写；
  - `$(selector)`是选择器；
  - `$(DOMselector)`将 DOM 对象转换为 jQuery 对象；示例：`var dom = document.getElementById(); $(dom).html()`;
  - `$(html)`例如：`$("<p></p>")`为创建新的节点对象；

## 二、jQuery 的延伸框架

### （一）jQuery Ul
- jQuery UI 是建立在 jQuery javaScript库上的一组用户界面交互、特效、小部件及主题。无论您是创建高度交互的Web应用程序还是仅仅向窗体控件添加一个日期选择器, jQuery U都是一个完美的选择。
- jQuery UI 包含了许多维持状态的小部件( Widget）

### （二）jQuery mobile
- jQuery mobile是创建移动web应用程序的框架
- jQuery mobile适用于所有流行的智能手机和平板电脑


## 三、 jQuery 主要功能

- HTML 元素选取 ：【原生 js 方法】`document.getElementByXXX()`
- HTML 元素操作 ：【原生 js 方法】`document.getElementByXXX().value = ""`
- CSS 操作 ：【原生 js 方法】`document.getElementById().style.color = ""`
- HTML 事件函数 ：【原生 js 方法】`document.getElementById().onclick`等
- JavaScript 特效和动画
- HTML DOM 遍历和修改 （即创建、修改或者删除标签对象）
- AJAX
- Utilities（实用工具）


## 四、jQuery 使用
### （一）$(document).ready()
学习 jQuery的第一件事是:如果你想要一个事件运行在你的页面上,你必须在`$(document).ready()` 里调用这个事件,所有包括在`$(document).ready()`里面的元素或事件都将会在DOM完成加载之后立即加载,并且在页面内容加载之前。
$(document).ready (function ()){
alert(“第一个 Jquery实例”)
}
比如:给 button设置一个点击事件；

**$(document).ready()与 window.onload 区别**
项目 | window.onload | $(document).ready()
---|---|---
执行时机|必须等待页面中所有的内容加载完毕之后（包括图片、flash、视频等）才能执行|网页中所有的 DOM 文档结构绘制完成之后即刻执行，可能与 DOM 元素关联的内容（图片、flash、视频等）并没有加载完成
编写个数 | 同一个页面不能同时编写多个 | 同一个页面能同时编写多个
简化写法 |无  | $(function(){ // 执行代码})   



### （二）jQuery 语法
- 基础语法：`$(selector).action()`
其中`$`其实就是 jQuery 的缩写，用于定义 jQuery；选择符`(selector)`用于查询和查找 HTML 元素；`action()`就是 jQuery 对元素的操作；


### （三）jQuery 选择器
- 基本选择器：元素（标签）、id、class、并集、交集、全局（通配*）；
- 层次选择器：后代选择器、子选择器、相邻选择器、同辈选择器；
- 属性选择器：含有属性、属性以什么开头、属性以什么结尾、属性值含有、属性值为某值、某属性不为某值；
- 基本过滤选择器：伪类选择器；
- 可见过滤选择器：伪元素选择器；

#### 1. 基本选择器
名称  | 语法构成  | 描述 |示例
---|---|---|---
标签选择器|element|根据给定的标签名匹配元素|`$("h1")` 选取所有标签为 h1 的元素
类选择器|.class|根据给定的 class 匹配元素|`$(".title")`选取所有的lass为title的元素 
id 选择器|#id|根据给定的 id 匹配元素|`$("#title")`选取 id 为 title 的元素
并集选择器 |selector1,selectot2,... （中间用逗号分隔）|将每个选择器匹配的元素合并后一起返回 |`$("div,p,.title")`所有的 div、p 和拥有 class 为 title 的元素
交集选择器 | element.class 或者 element#id （中间什么都没有）| 匹配指定 class 或者 id 的某个元素或者元素集合  |`$("h1.title")`选取所有拥有 class 为 title 的 h1 标签    
全局选择器| * |匹配所有元素|`$("*")`选取所有元素

#### 2. 层级选择器
层级选择器通过 DOM 元素之间的层次关系来获取元素
名称 | 语法构成 | 描述 | 示例 |
---|---|---|---|
后代选择器 | ancestor descendant | 选取 ancestor 元素里的所有 descendant（后代）元素 | `$("#menu span")`选取#menu 下的 <span> 元素|
子选择器 | parent>child | 选取 parent 元素下的 child（子）元素 | `$("#menu>span")`选取#menu 的子元素 <span> |
相邻元素选择器|prev+next|选取紧邻 prev 元素之后的 next 元素|`$("h1+dl")`选取紧邻 <h1> 元素之后的同辈元素 <dl>|
同辈元素选择器|prev~siblings|选取 prev 元素之后的所有 sublings 元素|`$("h1~dl")`选取 <h1> 元素之后所有的同辈元素 <dl>   |

#### 3. 属性选择器

名称 | 语法构成 | 描述 | 示例 
---|---|---|---
属性选择器|[attribute]|选取包含给定属性的元素|`$("[href]")`选取含有 href 属性的元素
属性选择器|[attribute=value]|选取等于给定属性是某个特定值的元素|`$("[href='#']")`选取属性值为“#的元素
属性选择器 | [attribute != value] | 选取不等于给定属性是某个特定值的元素 | `$("[href != '#']")`选取 href 属性值不为“#”的元素
属性选择器 | [attribute ^= value] | 选取给定属性是以某些特定值开始的元素 | `$("[href^=en]")`选取 href 属性值以 en 开头的元素
属性选择器 | [attribute $= value] | 选取给定属性是以某些特定值结尾的元素 | `$("[href $= '.jpg']")`选取href属性值以.jpg结尾的元素
属性选择器 |[attribute*=value] | 选取给定属性是以包含某些值的元素 | `$("[href *='txt']")`选取href属性值中含有 txt的元素
属性选择器 |[selector] [selector2] [selectorN] | 选取满足多个条件的复合属性的元素 | `$("li[id][title = 新闻要点]")`选取含有 Id 属性和 title 属性为 新闻要点 的<li>元素


### 测试代码
```jQuery
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8" />
		<title></title>
		<!-- 1.先通过 script src 引入 jquery，然后使用新的 script 标签书写自己的代码 -->
		<script type="text/javascript" src="js/jquery-3.4.1.js"></script>
		<script>
			// $可以认为是 jQuery 提供的函数,参数需要传入 document,ready 类似于Window 的 onload
			$(document).ready(function(){
				alert("bbb")
			})
			
			// 上面的也可以直接传入一个函数,此代码和上面的 $(document).ready 是等价的,表示 dom 绘制完成
			$(function(){
				// 功能一:隐藏网页中的图片
				// js 代码为:
				//  window.onload = function(){
				// 	// 这里因为 document.getElementbyTagName("img") 返回的是一个数组
				// 	document.getElementbyTagName("img")[0].style.display = "none";
				// }
				
				// 第一部分：$() 是一个函数； 第二部分:选择器也就是作为参数传入$()的字符串; 第三部分就是具体的要对标签对象做的动作,就是 action
				$("img").hide()
				
				alert("ccc")
			})
		
			alert("aaa")
		</script>
	</head>
	<body>
		<img src="https://ss1.baidu.com/9vo3dSag_xI4khGko9WTAnF6hhy/image/h%3D300/sign=05b297ad39fa828bce239be3cd1e41cd/0eb30f2442a7d9337119f7dba74bd11372f001e0.jpg" >
	</body>
</html>

```



```jQuery
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
		<!-- jq 选择器完全沿用了 CSS 选择器的语法，我们可以使用 CSS 选择器语法在 js 中 -->
		<script src="js/jquery-3.4.1.js" type="text/javascript" charset="utf-8"></script>
		<script type="text/javascript">
			// 功能二:将一个 div 中的文字变色
			// window.onload = function(){
			// 	document.getElementById("bbb").style.color = "red"
			// }
			
			$(function(){
				// CSS() 是一个 action，属性一为要修改的 CSS 样式属性,属性二为对应的值
				// 直接使用标签名表示标签选择器,这里下面标签为 div 的三行都会变色
				$("div").css("color", "green")
				alert("hello")
				// 使用#XXX表示id为XXX的选择器
				$("#bbb").css("color", "red")
				// 使用.XXX,表示类名为XXX的选择器
				$(".ccc").css("color", "blue")
				
				
				// 并集选择器
				// ccc 类的和 P 标签的都变色
				$(".ccc, p").css("color", "black")
				
				// 交集选择器
				// 一般由一个标签选择器和一个 Id 或者 class 选择器配合使用,两个选择器直接没有任何符号,包括空格
				$("p.ccc").css("color","yellow")
				
				
				// 后代选择器  (中间加上空格)
				// 将 div 中的所有 h1 颜色进行更改
				$("div h1").css("color", "gold")
				
				// 子选择器:表示选中某个选择器的下一层级元素,直接出现在下一个层级的元素
				$("div>h2").css("color", "orange")
				
				
				// 层级选择器:相邻选择器
				// 在同一层级下的,紧挨着某个选择器指定的元素下的第一个标签
				$("#ul+p").css("color", "red")
				// 同级选择器,挨个某个选择器下面的所有的标签
				$("#ul~p").css("color", "red")
				
			})
		</script>
		
	</head>
	<body>
		<div>第零行</div>
		<div id="bbb">第一行</div>
		<div class="ccc">第二行</div>
		<p>第四行P</p>
		<p class="ccc">第五行</p>
		
		<div>
			<h1>hhhhhh1</h1>
			<p>
				<h1>hhhhhh1</h1>
			</p>
			
			<h2>hhhhhhh2</h2>
			<p>
				<h2>hhhhhhhh2</h2>
			</p>
			
		</div>
		
		<h1>outhhhhhh1</h1>
		
		<p>cengjipppppp</p>
		<ul id="ul">
			<li>11111111</li>
		</ul>
		<p>pppppppp1</p>
		<p>pppppppp2</p>
			
	</body>
</html>

```

### 4. 过滤选择器

过滤选择器通过特定的过滤规则来筛选元素，语法特点是使用`:`，主要分为基本过滤选择器和可见性过滤选择器。

#### 基本过滤选择器
可以用于选取第一个元素、最后一个元素、索引为偶数或者奇数的元素；

名称 | 语法构成 | 描述 | 示例
---|---|---|---
基本过滤选择器|:first|选取第一个元素|`$("li:first")`选取所有<li>元素中的第一个<li>元素
基本过滤选择器|:last|选取最后一个元素|`$("li:last")`选取所有<li>元素中的最后一个<li>元素
基本过滤选择器|:even|选取索引是偶数的所有元素(index 从 0 开始)|`$("li:even")`选取索引是偶数的所有<li>元素   
基本过滤选择器|:odd|选取索引为奇数的所有元素(index 从 0 开始)|`$("li:odd")`选取索引是奇数的所有<li>元素   


**基本过滤选择器可以根据索引的值选取元素**
名称 | 语法构成 | 描述 | 示例
---|---|---|---
基本过滤选择器 | :eq(index) | 选取索引等于 Index 的元素（index 从 0 开始）| `$("li:eq(1)")`选取索引等于 1 的<li>元素
基本过滤选择器 | :gt(index) | 选取索引大于 index 的元素（index 从 0 开始）| `$("li:gt(1)")`选取索引大于 1 的<li>元素（不包括 1）
基本过滤选择器 | :lt(index) | 选取索引小于 index 的元素（index 从 0 开始）| `$("li:lt(1)")`选取索引小于 1 的<li>元素（不包括 1）  
 
**基本过滤选择器支持的特殊选择方式**
名称 | 语法构成 | 描述 | 示例
---|---|---|---
基本过滤选择器|:not(selector)| 选取去除所有与给定选择器匹配的元素 | `$("li:not(.three)")`选取 class 不是 three 的元素
基本过滤选择器 | :header | 选取所有标题元素，如 h1-h6 | `$(":header")`选取网页中所有标题元素
基本过滤选择器 | :focus | 选取当前获取焦点的元素 | `$(":focus")` 选取当前获取焦点的元素  


### 可见性过滤选择器
通过元素显示状态来选取元素
名称 | 语法构成 | 描述 | 示例 
---|---|---|---
可见性过滤选择器 | :visible | 选取所有可见的元素 | `$(":visible")`选取所有可见的元素
可见性过滤选择器  | :hidden | 选取所有隐藏的元素 | `$(":hidden")`选取所有隐藏的元素   


### 上面两项的代码示例
```java
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8" />
		<title></title>
		<script src="js/jquery-3.4.1.js" type="text/javascript" charset="utf-8"></script>
		<script type="text/javascript">
			$(function(){
				// 功能一:实现ul标签中的奇数行和偶数行颜色不同
				$("ul li:odd").css("background-color", "red")
				$("ul li:even").css("background-color", "green")
				
				// 功能二:实现以索引方式选取(从0开始),eq
				$("li:eq(3)").css("background-color", "blue")
				
				// 功能三:实现对当前焦点事件的操作(此为：将第一个输入框放大)
				$("input:first").focus(function(){
					// 第一个input获取焦点的时候,变化长和宽,同时jq中支持链式结构
					$("input:focus")
						.css("width", "200")
						.css("height", "50")				
				})
				
				// 功能四:实现对于所有的标题:h1-h6的操作
				$(":header").css("color", "blue")
				
				// 功能五:点击之后,显示相应的文本
				$("input").click(function(){
					// 方法一 : 使用标签选择器
					$("div").css("display", "block")
					// 方法二 : 使用可见性选择器
					$("div:hidden").css("display", "block")					
				})
			})
		</script>
	</head>
	<body>
		<ol>
			<li>hello</li>
			<li>hello</li>
			<li>hello</li>
			<li>hello</li>
		</ol>
		
		<ul>
			<li>hello</li>
			<li>hello</li>
			<li>hello</li>
			<li>hello</li>
		</ul>
		
		<input type="text" name="" id="text1" value="" />
		<input type="text" name="" id="text2" value="" />
		
		<h1>h1</h1>
		<h2>h2</h2>
		
		<div style="display:none;background-color: aqua;">
			点击之后，显示
		</div>
		<input type="button"  value="请点击" />
		
	</body>
</html>

```





## jQuery 中的事件
jQuery 事件是对 JavaScript 事件的封装，常用事件分类如下：
- 基础事件：
  - window 事件
  - 鼠标事件
  - 键盘事件
  - 表单事件
- 复合事件是多个事件的组合
  - 鼠标光标悬停
  - 鼠标连续点击

### 鼠标事件
鼠标事件是当用户在文档上移动或者单击鼠标时而产生的事件；
方法 | 描述 | 执行时机
---|---|---
click()|触发或将函数绑定到指定元素的 click 事件|单击鼠标时候
mouseover()|触发或将函数绑定到指定元素的 mouse over 事件|鼠标移过时
mouseout()|触发或将函数绑定到指定元素的 mouse out 事件|鼠标移除时

加上：hover

### 键盘事件
包括：keydown / keyup / keypress

### 复合事件
toggle


## 事件和动画类
### 表单事件
当元素获得焦点时候，会触发 focus 事件，失去焦点时，会触发 blur 事件；
方法 | 描述 | 执行时机
---|---|---|
focus()|触发或将函数绑定到指定元素的 focus 事件|获得焦点|
blur()|触发或将函数绑定到指定元素的 blur 事件|失去焦点|

### 事件的绑定
除了使用事件名绑定事件外，还可以使用 blind() 方法，示例如下
```html
$("button").blind("click",function(){
    $("p").css("background-color","#F30")
})
```


## 动画

jQuery 提供的动画效果包括：
- 控制元素显示与隐藏
- 控制元素淡入淡出
- 改变元素的高度

### 显示及隐藏元素
- show 在显示元素时候，能够定义显示元素时的效果，例如：显示速度；
`$(".tipsbox").show("slow")`, 显示参数可以选择：毫秒（如 1000）、slow、normal、fast；

###  淡入淡出效果
- `fadeIn()` 和 `fadeOut()` 可以通过改变元素的透明度实现淡入淡出效果；
```html
$("input").click(function(){
    $("img").fadeIn("slow");
});

$("input").click(function(){
    $("img").fadeOut(2000);
});
```


### 改变元素的高度
- slideDown()可以使元素逐步延伸展示，slideUp()则使元素逐步缩短直至隐藏；
```html
$("h2").click(function(){
    $(".txt").slideUp("slow");
    $(".txt").slideDown("slow");
});
```

代码示例：
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
		<script src="js/jquery-3.4.1.js" type="text/javascript" charset="utf-8"></script>
		<script type="text/javascript">
			$(function(){
				// 方法一:通过bind函数设置各种事件,参数一为时间名称,参数二为具体的执行函数
				$("input:first").bind("click",function(){
					// 可以设置时间,单位为ms,下面为2秒,show 和 hide 都是默认缩放
					$("img").show(2000)
					// fadeIn是淡入淡出效果
					$("img").fadeIn(2000)
				})
				
				$("input:last").bind("click",function(){
					// 可以设置时间,单位为ms,下面为2秒
					$("img").hide(2000)
					$("img").fadeOut(2000)
				})
			})
			
		</script>
	</head>
	<body>
		<img src="img/show.gif" >
		<input type="button" name="" id="" value="显示" />
		<input type="button" name="" id="" value="不显示" />
		
	</body>
</html>

```


## jQuery 中的 DOM 操作

jQuery 对 JavaScript 中的 DOM 操作进行了封装，使用起来也更加方便。
jQuery 中的 DOM 操作可以分为：
- 样式操作
- 内容及 value 属性值操作
- 节点操作
- 节点属性操作
- 节点遍历

### HTML 代码操作
HTML() 可以对 HTML 代码进行操作，类似于 js 中的 innerHTML；
两种用法：
```html
// 获取HTML内容
$("div.left").html(); 
// 设置标签内HTML内容
$("div.left").html("<div>class = 'content'</div>"); 
```
**HTML 和 Text（）的区别**
语法 | 参数 | 功能
---|----|---
html()|无参数 | 用于获取第一个匹配元素的 HTML 内容或者文本内容
html(content) | content 参数为元素的 HTML 内容 | 用于设置所有匹配元素的 HTML 内容或者文本内容
text() | 无参数 | 用于获取所有匹配元素的文本内容
text(content) | content 参数为元素的文本内容 | 用于设置匹配元素的文本内容   


代码示例：
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
		
		<script src="js/jquery-3.4.1.js" type="text/javascript" charset="utf-8"></script>
		<script type="text/javascript">
			$(function(){
				$("#btn1").click(function(){
					// 下面等价于:document.getElementbyId("div").innerHTML = ""
					// html()函数没有参数时候表示获取某个标签内部的内容
					$("#div").html("<a href=''> 重新设置值 </a>")
				})
				$("#btn1").click(function(){
					// 下面等价于:document.getElementbyId("div").innerText = ""
					// text()将里面的内容直接当做字符串,即使里面含有:HTML/js/css也不会解析,用于将源代码直接输出到页面中
					$("#div").text("<a href=''> 重新设置值 </a>")
				})
			})
		</script>
	</head>
	<body>
		<div id="div" >这是默认值</div>
		
		<input type="button" name="" id="btn1" value="点击HTML()" />
		<input type="button" name="" id="btn2" value="点击TEXT()" />
	</body>
</html>
```


### value 操作
```html
$(this).val();
$(this).val("");
```

YY 登录：
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
		<script src="js/jquery-3.4.1.js" type="text/javascript" charset="utf-8"></script>
		<script type="text/javascript">
			$(function(){
				// 设置获取焦点的监听
				$("input:first")
					.focus(function(){
						// 清空文字(即默认value)的值
						// jQuery中提供一个val函数，操作表单元素的value的属性值;相当于document.getElement().value = ""
						if($(this).val() == "通行账号/邮箱"){
							$(this).val("")
						}
						
					})
					// 
					.blur(function(){
						// 失去焦点的时候判断用户是否已经输入内容,没有则设为默认值,有则保留输入
						// val() 参数为空的时候表示获取值
						if($(this).val() == ""){
							$(this).val("通行账号/邮箱")
						}
					})
			})
		</script>
		
	</head>
	<body>
		<div id="login">
			<form action="" method="">
				<input type="text" name="username"  class="text username" value="通行账号/邮箱" />
				<input type="text" name="password" class="text password" value="密码" />
				<input type="submit" name="sub" class="btn" value="提交" />
			</form>
			
		</div>
	</body>
</html>

```



### 结点操作
jQuery 操作有：
- 查找节点
- 插入节点
- 删除节点
- 替换节点
- 创建节点
- 复制节点

#### 创建节点元素
工厂函数$()用于获取或者创建节点
- $(selector): 通过选择器获取节点
- $(element): 把 DOM 节点转换为 JQuery 节点
- $(html): 使用 HTML 字符串创建 jQuery 节点

DOM 节点和 jQuery 节点区别
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
		<script src="js/jquery-3.4.1.js" type="text/javascript" charset="utf-8"></script>
		<script type="text/javascript">
			// js和jQuery代码可以共存,但是jQuery函数只能由jQuery节点进行调用,同样dom对象只能由原生的方式或者属性进行操作
			$(function(){
				// 获取dom节点对象
				var div1 = document.getElementById("div")
				// 获取jq节点对象
				var div2 = $("#div")
				
				div1.innerHTML = "div1.dom"
				div2.html("div2.jq")
				
				// 如果dom对象先使用jq函数,需要进行类型转换
				$(div1).html("div1.jq")
			})
		</script>
	</head>
	<body>
		<div id="div">
			默认值
		</div>
	</body>
</html>

```

#### 添加子节点
即在元素内部插入子节点
 语法 | 功能
 ---|---
append(content)| $(A).append(B)表示将 B 追加到 A 中，如：`$("ul").append($newNode1);`
appendTo(content) |  $(A).appenTo(B)表示将 A 追加到 B 中，如：`$newNode1.append("ul");`  
prepend(content) | $(A).prepend(B)表示将 B 前置插入到 A 中，如：`$("ul").prepend($newNode1);`
prependTo(content) | $(A).prependTo(B)表示将 A 前置插入到 B 中，如：`$newNode1.prependTo("ul");`   
Demo：有一个 ul 和 button，每点击一个 Button 就在 ul 中添加一个 li
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
		
		<script src="js/jquery-3.4.1.js" type="text/javascript" charset="utf-8"></script>
		<script type="text/javascript">
			$(function(){
				$("#add").click(function(){
					// 1.新建一个li节点对象
					var li1 = $("<li>新添加元素1</li>");
					var li2 = $("<li>新添加元素2</li>");
					
					// 2.把HTML标签转换为一个Jq节点对象
					// append 是jq函数,因此追加的节点也应为jq节点,可以同时传入多个参数
					$("#ul").append(li1, li2)
				})
				
			})
		</script>
	</head>
	<body>
		<ul id="ul">
			原来的
		</ul>
		<input type="button" name="" id= "add" value="点击追加" />
	</body>
</html>


```

#### 插入同辈节点
元素外部插入同辈节点
语法 | 功能
---|---
after(content)| `$(A).after(B)`表示将 B 插入 A 之后，如：`$("ul").after($newNode1);`
insertAfter(content) | $(A).insertAfter(B)表示将 A 插到 B 之后，如：`$newNode1.insertAfter("ul");` 
before(content) | $(A).before(B)表示将 B 插入到 A 之前，如：`$("ul").before($newNode1);`
insertBefore(content) | $(A).insertBefore(B)表示将 A 插入到 B 之前，如：`$newNode1.insertBefore("ul");`  



#### 替换节点
使用：`replaceWith()`和`replaceAll()`用于替换某个节点，两者的关系类似于 append() 和 appendTo()
示例：
```html
$("ul li:eq(1)").replaceWith($newNode1);
$newNode1.repalceAll("ul li:eq(1)");
```
示例代码：
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
		<script src="js/jquery-3.4.1.js" type="text/javascript" charset="utf-8"></script>
		<script type="text/javascript">
			$(function(){
				$("#replace").click(function(){
					// 点击button替换ul的第二个li
					var newli = $("<li></li>").text("new")
					$("ul li:eq(1)").replaceWith(newli)
				})
			})
		</script>
	</head>
	<body>
		<input type="button" name="" id="replace" value="点击替换" />
		<ul>
			<li>111</li>
			<li>222</li>
			<li>333</li>
		</ul>
	</body>
</html>

```

#### 复制节点
- clone() 用于复制某个节点
示例：`$("ul li:eq(1)").clone(true).appendTo("ul");`其中 true 表示不仅处理复制节点的内容，还复制存在在节点上的事件；

- 可以使用 clone() 实现获取 DOM 元素本身的 HTML；
示例：`$(DOM元素).clone().html()`


#### 获取与设置节点属性
- attr() 用来获取与设置元素属性
示例：获取 alt 属性值
```html
$newNode1.attr("alt");
$newNode1.attr({width:"30", height:"50"});
```
 
- removeAttr() 用来删除元素的属性
示例：删除元素的 title 属性
```html
$newNode1.removeAttr("title");
```

代码示例：
```html

<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
		<script src="js/jquery-3.4.1.js" type="text/javascript" charset="utf-8"></script>
		<script type="text/javascript">
			$(function(){
				// 获取节点属性
				$("#but").click(function(){
					alert($("#img").attr("id"));
					alert($("#img").attr("title"));
				})
				
				// 设置节点属性,多属性设置有两种方法
				$("#but2").click(function(){
					$("#img")
					.attr("src","https://mmbiz.qpic.cn/webp&&wx_co=1")
					.attr("title","csdn")
					.attr({height:"200", width:"400"})
				})
			})
		</script>
	</head>
	<body>
		<img src="img/show.gif" id="img" title="diandian" alt="不显示">
		<input type="button" id="but" value="点击获取属性值" />
		<input type="button" id="but2" value="点击设置属性值" />
	</body>
</html>
```

#### 删除节点
jQuery 提供了三种删除节点的方法
- remove() 删除整个节点（参数为过滤条件）；
- detach() 删除整个节点，但是保留元素的绑定事件、附加的数据；
- enpty() 清空节点内容；

remove 和 detach 只是从节点中移除，并不是完全删除，还可以追加找回，如果是 remove 只能找回节点，但是数据事件的附加消息丢失，detach 附加数据不消失。例如点击事件
可以在运行时通过 F12，然后在浏览器的 Element 中查看

示例代码
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
		<script src="js/jquery-3.4.1.js" type="text/javascript" charset="utf-8"></script>
		<script type="text/javascript">
			$(function(){
				$("#empty").click(function(){
					// 功能一:点击之后清空div中的值
					// 下面等价于: $("#div").html("")
					$("#div").empty()
				})
				$("#remove").click(function(){
					// 功能二:从文档中移除节点,即删除标签,但是节点的对象还在内存中
					$("#div").remove()
				})
				$("#detach").click(function(){
					// 功能三:从文档中移除节点,即删除标签，但是节点的对象还在内存中
					$("#div").detach()
				})
								
				// 功能四:删除之后找回,remove之后只能找回样式,detach可以找回事件				
				// 为div设置一个事件,看删除之后能够找回
				$("div").click(function(){
					alert("事件还在")
				})
				// 使用remove之后恢复
				$("#removeRecovery").click(function(){
					// 功能二:从文档中移除节点,即删除标签
					$("#div").remove().appendTo("body")
				})
				// 使用detach之后恢复
				$("#detachRecovery").click(function(){
					// 功能二:从文档中移除节点,即删除标签
					$("#div").detach().appendTo("body")
				})
			})
		</script>
	</head>
	<body>
		<div id="div">
			<h1>nihao</h1>
			<p>nihao1</p>
			<p>nihao2</p>
			<p>nihao3</p>
			<p>nihao4</p>
		</div>
		<input type="button" name="" id="empty" value="点击empty清空节点内容" />
		<input type="button" name="" id="remove" value="点击remove移除节点" />
		<input type="button" name="" id="detach" value="点击detach移除节点" />
		<input type="button" name="" id="removeRecovery" value="点击removeRecovery移除后恢复节点" />
		<input type="button" name="" id="detachRecovery" value="点击detachRecovery移除后恢复节点" />
			
	</body>
</html>

```


### 遍历模块

#### 遍历子元素
- children() 方法
children() 方法可以用来获取元素的所有子元素，方法可以有参数，有参数表示过滤；
示例：`$("body").children();`表示获取<body>元素的子元素，但是不包含子元素的子元素；

- find() 方法
find() 方法返回被选元素的后代元素，一路向下直到最后一个后代
示例：返回属于<div>后代的所有<span>元素；
```html
$(document).ready(function(){
  $("div").find("span");
})
```

代码示例：
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
		<script src="js/jquery-3.4.1.js" type="text/javascript" charset="utf-8"></script>
		<script type="text/javascript">
			$(function(){
				// 功能 : 将ul下面子元素颜色改变
				// 原来方式: $("ul li").css("color", "red");
				$("ul").children().css("color","red")
				
				// children配合过滤选择器使用
				$("ul").children(":first").css("background-color","yellow")
				$("ul").children(":gt(0)").css("background-color","blue")
				
				// find 配合选择器使用
				$("ul").find("span").css("background-color","green")
			})
		</script>
	</head>
	<body>
		<ul>
			<li>li111</li>
			<li>li222</li>
			<li>li333</li>
			<li><span>li444</span></li>
		</ul>
	</body>
</html>
 

```

#### 遍历同辈元素
jQuery 可以获取紧邻其后、紧邻其前和位于该元素前与后的所有同辈元素；
语法 | 功能
---|---
next([expr]) | 用于获取紧邻匹配元素之后的元素，如：`$("li:eq(1)").next().css(XXXX)`
prev([expr]) | 用于获取紧邻匹配元素之前的元素，如：`$("li:eq(1)").prev().css(XXXX)`  
slibings([expr]) |用于获取位于匹配元素前面和后面的所有同辈元素，如：`$("li:eq(1)").siblings().css(XXXX)` 

其他常用方法：`siblings()`、`next()`、`nextAll()`、`nextUntil(4)`、`prev()`、`prevAll()`、`prevUntil()`。

####  遍历前辈元素
语法 | 功能
---|---
parent() | 获取元素的父级元素
parents() | 获取元素的祖先元素
parentsUnit() | 方法返回介于两个给定元素之间的所有祖先元素

点击复制该元素进行新增 Demo；
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>		
		<script src="js/jquery-3.4.1.js" type="text/javascript" charset="utf-8"></script>
		<script type="text/javascript">
			$(function(){
				$(".del").click(function(){
					// 获取当前点击的是哪一个删除按钮
					$(this).parent().parent().remove()
				})
				
				$(".copy").click(function(){
					// 复制一条新的数据加入当前表格
					// 注：table标签在被浏览器解析时候，浏览器会在table和tr之后加上标签 ：tbody
					var newLine = $("table").find("tr:eq(1)").clone()
					$("table").append(newLine)					
				})
				
				// 点击哪一个标签，在表格最顶端加上一条数据，数据为点击的当前行数据
				$(".add").click(function(){
					$(this).parent().parent().clone(true).insertAfter($("table tr:first"))
				})			
			})
		</script>
	</head>
	
	<body>
		<table border="0" cellspacing="0" cellpadding="">
			<tr>
				<th>姓名</th>
				<th>性别</th>
				<th>生日</th>
				<th>&nbsp;&nbsp;</th>
			</tr>
			<tr>
				<td>张三</td>
				<td>男</td>
				<td>123</td>
				<td>
					<img src="img/copy.png" class="copy" >
					<img src="img/del.png" class="del" >
				</td>
			</tr>
			<tr>
				<td>李四</td>
				<td>男</td>
				<td>1234</td>
				<td>
					<img src="img/copy.png" class="copy" >
					<img src="img/del.png" class="del" >
				</td>
			</tr>
			<tr>
				<td>王五</td>
				<td>男</td>
				<td>12345</td>	
				<td>
					<img src="img/copy.png" class="copy" >
					<img src="img/del.png" class="del" >
				</td>
			</tr>
		</table>
		
		<img src="img/add.png" class="add" >
	</body>
</html>

```


### 小结
- 样式操作：css()、addClass() 、removeClass() 、toggleClass(); 
- 内容操作：html()、text()、val()
- 节点操作：查找、创建、替换、复制和遍历
- 节点属性的操作：attr()、removeAttr()
- 遍历操作：遍历子元素、遍历同辈元素和遍历前辈元素；

购物页展开：
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
		
		<style type="text/css">
			#nav .navsBox ul{
				display: none;
				list-style: none;
			}
			
			#nav .navsBox ul li.onbg{
				background-color: antiquewhite;
			}
		</style>
		
		<script src="js/jquery-3.4.1.js" type="text/javascript" charset="utf-8"></script>
		<script type="text/javascript">
			$(function(){
				// 点击显示和隐藏列表
				$(".firstNav").click(function(){
					$("#nav .navsBox ul").toggle()
				})
				
				// 监听鼠标悬停
				$("#nav .navsBox ul li").hover(
					// 表示悬停在上面
					function(){
						// $(this)表示鼠标悬停在的li对象
						$(this).addClass("onbg")
					},
					// 表示移开之后
					function(){
						$(this).removeClass("onbg")
					}
				)
				
			})
		</script>	
	</head>
	

	<body>
		<div id="nav">
			<div class="navsBox">
				<div class="firstNav">购物特权</div>
				<ul>
					<li class="jedh"><a href="#">全额兑换</a></li>
					<li class="jlbbyk"><a href="#">俱乐部包邮卡</a></li>
					<li class="jwljb"><a href="#">购物领金币</a></li>
					<li class="mrljb"><a href="#">每日领金币</a></li>
					<li class="vipjtj"><a href="#">VIP 阶梯价</a></li>
				</ul>
				
			</div>
			
		</div>
	</body>
</html>

```


## jQuery 中的 Ajax 和 json

Ajax 是一种与服务器交换数据的技术，可以在不重新载入整个页面的情况下更新网页的一部分；

jQuery 中对 Ajax 进行了封装，使异步刷新更加简单； 


### $.ajax() 的使用
$.ajax()是 jq 提供的一个函数，这个函数实现了对 Ajax 的请求，并且帮助开发者完成了细节，例如兼容性问题，对于 POST 请求头部设置等等；开发者只需要将所有的参数组装成一个对象当做一个参数传入该方法中即可；
ajax（）方法用于执行AAX(异步HTTP)请求。格式为：`$.ajax({name: value, name: value,...})`
参数可以有：
参数值 | 含义
---|---  
async  | 请求是否异步处理。默认是true。
beforeSend  | 发送请求前运行的函数。
success | 当请求成功时运行的函数。
error | 当请求失败时运行的函数。
complete | 请求完成时运行的函数
data | 规定要发送到服务器的数据。
type | 规定请求的类型(GET或POST)。
url | 规定发送请求的URL。默认是当前页面

代码示例：
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title></title>
		<script src="js/jquery-3.4.1.js" type="text/javascript" charset="utf-8"></script>
		<script type="text/javascript">
			$(function(){
				$("#but").click(function(){
					// 使用jQuery发送一个Ajax请求
					// 参数只有一个,是一个js对象,对象中包裹着多个参数
					$.ajax({
						// 设置请求方式
						type:"post",
						// 设置请求路径
						url:"index1.html",
						// 设置是否异步
						async:true,
						// 对请求成功的监听,如果请求成功(相当于：readystate==4 && status ==200)
						// function()里面的参数表示请求成功的时候会将响应数据赋值给此参数(该参数也是一个对象)
						success: function(data){
							alert(data)
						},
						// 对请求失败的监听
						error: function(){
							// 请求失败的事件监听函数
							alert("请求失败")
						},
						// 如果是POST请求, 并且有参数时候,参数就放置到data键值对的值中
						data:"username = admin"
					});
				})
			})
		</script>
	</head>
	<body>
		<input type="button" name="" id="but" value="点击按钮返回一个Ajax请求"/>
	</body>
</html>

```

###    其他用法
一共四种用法：`$.ajax()`、`$.get()` 、`$.post()` 、`$.getJSON()`;
其中`$.ajax()`中可以使用 get 和 set 方法；

- $.get() 方法：`$.get("url", "username = admin", function(){})`
- $.post() 方法：`$.get("url", "username = admin", function(){})`

### Ajax 的跨域
不仅在 jQuery 中存在跨域，在原生 js 中也存在跨域；
发生 Ajax 请求的资源和指向 URL 的网络资源不再同一个服务器，虽然直接访问请求可以到达，但是客户端会报错；
即：jQuery 请求和 Servlet 不在一个路径下面，甚至不在一个项目；当 Servlet 的 Tomcat 服务器打开后怎么使用 jQuery 访问。

**▶解决方法：**是实现跨域需要的**服务端** 添加以下配置（放在 doGet 或者 doPOST 里面）；表示允许所有位置的请求，并且支持 GET 和 POST 方式；
```java
response.setHeader("Access-Control-Allow-Origin","*");
response.setHeader("Access-Control-Allow-Methods","POST,GET");
```
同时在 jQuery 中的 URL 填该 Servlet 的绝对路径；