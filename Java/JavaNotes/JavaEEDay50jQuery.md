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
比如:给 button设置一个点击事件

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

#### 基本选择器
名称  | 语法构成  | 描述 |示例
---|---|---|---
标签选择器|element|根据给定的标签名匹配元素|`$("h1")` 选取所有标签为 h1 的元素
类选择器|.class|根据给定的 class 匹配元素|`$(".title")`选取所有的lass为title的元素 
id 选择器|#id|根据给定的 id 匹配元素|`$("#title")`选取 id 为 title 的元素
并集选择器 |selector1,selectot2,... （中间用逗号分隔）|将每个选择器匹配的元素合并后一起返回 |`$("div,p,.title")`所有的 div、p 和拥有 class 为 title 的元素
交集选择器 | element.class 或者 element#id （中间什么都没有）| 匹配指定 class 或者 id 的某个元素或者元素集合  |`$("h1.title")`选取所有拥有 class 为 title 的 h1 标签    
全局选择器| * |匹配所有元素|`$("*")`选取所有元素

#### 层级选择器
层架选择器通过 DOM 元素之间的层次关系来获取元素
名称 | 语法构成 | 描述 | 示例 |
---|---|---|---|
后代选择器 | ancestor descendant | 选取 ancestor 元素里的所有 descendant（后代）元素 | `$("#menu span")`选取#menu 下的 <span> 元素|
子选择器 | parent>child | 选取 parent 元素下的 child（子）元素 | `$("#menu>span")`选取#menu 的子元素 <span> |
相邻元素选择器|prev+next|选取紧邻 prev 元素之后的 next 元素|`$("h1+dl")`选取紧邻 <h1> 元素之后的同辈元素 <dl>|
同辈元素选择器|prev~siblings|选取 prev 元素之后的所有 sublings 元素|`$("h1~dl")`选取 <h1> 元素之后所有的同辈元素 <dl>   |

#### 属性选择器

名称 | 语法构成 | 描述 | 示例 
---|---|---|---
属性选择器|[attribute]|选取包含给定属性的元素|`$("[href]")`选取含有 href 属性的元素
属性选择器|[attribute=value]|选取等于给定属性是某个特定值的元素|`$("[href='#']")`选取属性值为“#的元素
属性选择器 | [attribute != value] | 选取不等于给定属性是某个特定值的元素 | `$("[href != '#']")`选取 href 属性值不为“#”的元素
属性选择器 | [attribute ^= value] | 选取给定属性是以某些特定值开始的元素 | `$("[href^=en]")`选取 href 属性值以 en 开头的元素
属性选择器 | [attribute$=value] | 选取给定属性是以某些特定值结尾的元素 | `$("[href$= '.jpg']")`选取href属性值以.jpg结尾的元素
属性选择器 |[attribute*=value] | 选取给定属性是以包含某些值的元素 | `$("[href*='txt']")`选取href属性值中含有 txt的元素
属性选择器 |[selector] [selector2] [selectorN] | 选取满足多个条件的复合属性的元素 | `$("li[id][title=新闻要点]")`选取含有 Id 属性和 title 属性为 新闻要点 的<li>元素


### 测试代码
```jQuery
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8" />
		<title></title>
		<!-- 1.先通过script src引入jquery，然后使用新的script标签书写自己的代码 -->
		<script type="text/javascript" src="js/jquery-3.4.1.js"></script>
		<script>
			// $可以认为是jQuery提供的函数,参数需要传入document,ready类似于Window的onload
			$(document).ready(function(){
				alert("bbb")
			})
			
			// 上面的也可以直接传入一个函数,此代码和上面的$(document).ready是等价的,表示dom绘制完成
			$(function(){
				// 功能一:隐藏网页中的图片
				// js代码为:
				//  window.onload = function(){
				// 	// 这里因为 document.getElementbyTagName("img") 返回的是一个数组
				// 	document.getElementbyTagName("img")[0].style.display = "none";
				// }
				
				// 第一部分：$()是一个函数； 第二部分:选择器也就是作为参数传入$()的字符串; 第三部分就是具体的要对标签对象做的动作,就是action
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
		<!-- jq选择器完全沿用了CSS选择器的语法，我们可以使用CSS选择器语法在js中 -->
		<script src="js/jquery-3.4.1.js" type="text/javascript" charset="utf-8"></script>
		<script type="text/javascript">
			// 功能二:将一个div 中的文字变色
			// window.onload = function(){
			// 	document.getElementById("bbb").style.color = "red"
			// }
			
			$(function(){
				// CSS()是一个action,属性一为要修改的CSS样式属性,属性二为对应的值
				// 直接使用标签名表示标签选择器,这里下面标签为div的三行都会变色
				$("div").css("color", "green")
				alert("hello")
				// 使用#XXX表示id为XXX的选择器
				$("#bbb").css("color", "red")
				// 使用.XXX,表示类名为XXX的选择器
				$(".ccc").css("color", "blue")
				
				
				// 并集选择器
				// ccc类的和P标签的都变色
				$(".ccc, p").css("color", "black")
				
				// 交集选择器
				// 一般由一个标签选择器和一个Id或者class选择器配合使用,两个选择器直接没有任何符号,包括空格
				$("p.ccc").css("color","yellow")
				
				
				// 后代选择器  (中间加上空格)
				// 将div中的所有h1颜色进行更改
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

### 过滤选择器

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
除了使用事件名绑定事件外，还可以使用 blind()方法，示例如下
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
- fadeIn() 和 fadeOut() 可以通过改变元素的透明度实现淡入淡出效果；
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






