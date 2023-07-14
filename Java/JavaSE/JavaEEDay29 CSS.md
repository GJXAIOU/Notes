---
tags:
- css
style: summer
---

# JavaDay29 CSS
@toc

## 一、CSS 选择器
就是根据标记，确定哪些内容是用该修饰器修饰的，可以认为是一种格式化；
六种修饰器都是定义在`<style></style>`标签中；

选择器一： 标签选择器 通过标签名来确定要修饰的内容是什么，下面代码中只有P标签被修饰了；
选择器二：class选择器，根据标签里的class属性来确定需要哪一个标签；**使用点开头**；
选择器三：id选择器，通过标签的id属性来确定修饰哪一个标签，ID选择器用#开头；
选择器四：组合选择器 每一个不同的选择通过,逗号隔开，做一个组合选择；
选择器五：层级选择器 ，下面代码中修饰的是“北方吃甜粽“；
选择器六：属性选择器 ，根据标签属性选择哪一个；
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>CSS选择器</title>
		<style>
			/*选择器一： 标签选择器 通过标签名来确定要修饰的内容是什么，这里只有P标签被修饰了 */
			p {
				color: red;
			}
			
			/* 选择器二：class选择器，根据标签里面的class属性来确定需要哪一个标签
			   class选择器使用点开头 */
			.sdj {
				background: red;
				color: white;
				width: 300px;
				height: 50px;
				/* 注意：在CSS样式中，修饰长度的属性，都有带有数据单位 */
			}
			
			/* 选择器三：id选择器，通过标签的id属性来确定修饰哪一个标签，ID选择器用#开头*/
			#chunjie {
				background: #F21000;
				width: 300px;
				height: 100px;
				color: gold;
			}
			
			/* 选择器四：组合选择器 每一个不同的选择通过,逗号隔开，做一个组合选择*/
			p, .sdj, #chunjie {
				font-size: 32px;
			}
			
			/* 选择器五：层级选择器 ，这里修饰的是“北方吃甜粽”*/
			#box .one div {
				font-size: 20px;
				color: greenyellow;
			}
			
			/* 选择器六：属性选择器 ，根据标签属性选择哪一个*/
			input[type=text] {
				color: red;
			}
			/* 通配符，以及字体修饰过的不再改变，属性越确定优先级越高    */
			
			* {
				font-size: 24px;
			}
		</style>
		
	</head>
	<body>
		<p>我的家在东北，松花江上啊~~~哪里有漫山遍野，大豆高粱啊~~~</p>
		<div class="sdj">
			Merry 圣诞节
		</div>
		
		<div id="chunjie">
			大年初一中午团圆饭
		</div>
		
		<div id="box">
			<div class="one">
				<div> 北方吃甜粽 </div>
			</div>
			
			<div class="two">
				<div> 南方吃肉粽 </div>
			</div>
		</div>
		
		<input type="text" />
		<input type="password" />
		<input type="submit" />
	</body>
</html>

```
执行效果：
![选择器显示结果]($resource/%E9%80%89%E6%8B%A9%E5%99%A8%E6%98%BE%E7%A4%BA%E7%BB%93%E6%9E%9C.jpg)



## 二、伪类选择器
针对鼠标点击前、放置、点击、点击后的变化进行设置；
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>伪类选择器</title>
		<style type="text/css">
			/* 伪类选择器 */
			/* a:link 表示a标签的连接状态，未点击状态 */
			a:link {
				color: hotpink;
			}
			
			/* 点击之后的状态 */
			a:visited {
				color: purple;
			}
			
			/* 鼠标悬停的状态 */
			a:hover {
				color: red;
			}
			
			/* 鼠标点击之后的活动状态 */
			a:active {
				color: forestgreen;
			}
		</style>
	</head>
	<body>
		<a href="http://www.mogujie.com/">蘑菇街-我的买手街</a>
	</body>
</html>

```

### size 设置
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title>尺寸设置</title>
		<style>
			html {
				font-size: 30px;
			}
			
			body {
				font-size: 32px;
			}
			
			div {
				font-size: 2em;
				/*
				em相对于父元素的比例
				当前div的父元素是body标签
				当前字体大小是32px * 2 = 64px
				 */
			}
			
			p {
				font-size: 1.5rem;
				/* 相对于HTML标签(20)的比例 */	
			}
		</style>
	</head>
	<body>
		今天中午吃什么？？？
		<div style="color: red;">东哥今天中午大盘鸡~~~</div>
		<p style="color: gold;">照烧鸡米饭</p>
	</body>
</html>

```
### front 设置
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>字体设置</title>
		<style type="text/css">
			#fonttest {
				/* 设置斜体 */
				font-style: italic; 
				/* 设置加粗 */
				font-weight: bolder;
				/* 设置字体大小 */
				font-size: 32px;
				/* 设置字体样式 */
				font-family: 幼圆;
			}
		</style>
	</head>
	<body>
		<div id="fonttest">
			中午我定的炒米饭，爱马仕米饭
		</div>
	</body>
</html>

```

### text
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>文本设置</title>
		<style>
			.taobao {
				width: 500px;
				height: 300px;
				background-color: #FF5000;
				color: white; 
				/* 首行缩进两个字符 */
				text-indent: 2em;
				/* 设置文字居中 */
				text-align: center;
			}
			
			#jd {
				width: 380px;
				height: 50px;
				background-color: #F10215;
				color: white;
				/* 超出部分隐藏 */
				overflow: hidden;
				
				/*
				 ellipsis:超出部分用...替换
				 clip: 截断
				*/
				text-overflow: ellipsis;
				
				/* 行高 */
				line-height: 50px;
				
				/* 强制不换行 */
				white-space: nowrap;
			}
			
			#shadowtest {
				font: 32px 幼圆;
				/* 
				 text-shadow：文本阴影，
				 第一个参数X方向偏移，
				 第二个参数Y轴方向偏移
			 	 第三参数虚化程度
			 	 第四个参数阴影颜色 
				 * */
				text-shadow: 5px 5px 1px red;
			}
			
		</style>
	</head>
	<body>
		<p class="taobao">
			淘宝网 - 亚洲较大的网上交易平台，提供各类服饰、美容、家居、数码、话费/点卡充值… 
			数亿优质商品，同时提供担保交易(先收货后付款)等安全交易保障服务，
			并由商家提供退货承诺、破损补寄等消费者保障服务，让你安心享受网上购物乐趣！
		</p>
		
		<div id="jd">
京东JD.COM-专业的综合网上购物商城,销售家电、数码通讯、电脑、家居百货、服装服饰
			、母婴、图书、食品等数万个品牌优质商品.便捷、诚信的服务，为您提供愉悦的网上购物体验!
			京东JD.COM-专业的综合网上购物商城,销售家电、数码通讯、电脑、家居百货、服装服饰
			、母婴、图书、食品等数万个品牌优质商品.便捷、诚信的服务，为您提供愉悦的网上购物体验!
			京东JD.COM-专业的综合网上购物商城,销售家电、数码通讯、电脑、家居百货、服装服饰
			、母婴、图书、食品等数万个品牌优质商品.便捷、诚信的服务，为您提供愉悦的网上购物体验!
			</div>
		<span id="shadowtest">
			我的米饭已经到了，多半是凉了~~~
		</span>
	</body>
</html>

```

### background
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>背景</title>
		<style type="text/css">
			#hlm {
				width: 800px;
				height: 500px;
				background-color: mediumturquoise;
				overflow: auto;
				/*
				 url(图片的地址) 这里可以是服务器本地的地址，也可以是一个网络端的图片地址
				 */
				background-image: url(img/u=168663427,492053552&fm=27&gp=0.jpg);
				/*
				 是否平铺图片：no-repeat
				 repeat-x X轴方向平铺
				 repeat-y Y轴方向平铺
				 */
				background-repeat:no-repeat;
				
				/*background-position: right bottom*/;
				background-attachment: local;
				/*
				fixed:相对于整个页面窗口固定
				scroll:相对于当前元素固定，如果是整个页面，随着元素的移动而且移动
				local:固定于当前元素，随着元素的滚动而滚动
				 */
			}
			
			body {
				background-image: url(img/u=168663427,492053552&fm=27&gp=0.jpg);
				background-repeat: no-repeat;
				background-position: right bottom;
				background-attachment: fixed; 
			}
		</style>
	</head>
	<body>
		<div id="hlm">
			话说平儿听迎春之言，正自好笑，忽见宝玉也来了。原来管厨房柳家媳妇之妹，
			也因放头开赌得了不是。这园中有素与柳家不睦的，便又告出柳家的来，说她和
			她妹子是伙计，虽然她妹子出名，其实赚了钱，两个人平分。因此凤姐要治柳家
			之罪。那柳家的因得此信，便慌了手脚，因思素与怡红院人最为深厚，故走来悄
			悄的央求晴雯、金星玻璃等人。金星玻璃告诉了宝玉。宝玉因思内中迎春之乳母
			也现有此罪，不若来约同迎春讨情，比自己独去，单为柳家说情，又更妥当，故
			此前来。忽见许多人在此，见他来时，都问：“你的病可好了？跑来作什么？”宝
			玉不便说出讨情一事，只说：“来看二姐姐。”当下众人也不在意，且说些闲话。	
			平儿便出去办累丝金凤一事。那王住儿媳妇紧跟在后，口内百般央求，只说：“姑
			娘好歹口内超生，我横竖去赎了来。”平儿笑道：“你迟也赎，早也赎，既有今日，
			何必当初。你的意思得过去就过去了。既是这样，我也不好意思告人，趁早去赎了
			来，交与我送去，我一字不提。”王住儿媳妇听说，方放下心来，就拜谢，又说：“
			姑娘自去贵干，我赶晚拿了来，先回了姑娘，再送去，如何？”平儿道：“赶晚不来，
			
			娘好歹口内超生，我横竖去赎了来。”平儿笑道：“你迟也赎，早也赎，既有今日，
			何必当初。你的意思得过去就过去了。既是这样，我也不好意思告人，趁早去赎了
			来，交与我送去，我一字不提。”王住儿媳妇听说，方放下心来，就拜谢，又说：“
			姑娘自去贵干，我赶晚拿了来，先回了姑娘，再送去，如何？”平儿道：“赶晚不来，
			可别怨我。”说毕，二人方分路各自散了。
		</div>
		<br />
		<br />
		<br />
		<br />
		<br />
		<br />
		<br />
		<br />
		<br />
		<br />
		<br />
		<br />
		<br />
		<br />
		<br />
		<br />
		<br />
	</body>
</html>

```

### position
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>定位</title>
		<style type="text/css">
			div {
				width: 300px;
				height: 200px;
			}
			
			#div1 {
				background-color: greenyellow;
				/* 绝对absolute: 是和body标签的位置
				 * 会脱离文档流，和原本的标签顺序无关 */
				position: absolute;
				left: 358px;
				top: 258px;
			}
			
			#div2 {
				background-color: pink;
				
				position: fixed;
				right: 0px;
				bottom: 0px;
			}
			
			#div3 {
				background-color: deepskyblue;
				position: absolute;
				top: 50px;
				left: 50px;
				z-index: 5;
			}
			
			.d {
				width: 300px;
				height: 200px;
				background-color: hotpink;
				position: relative; /* 相对，相对于当前父元素的位置 */
				top: 20px;
				left: 20px;
			}
		</style>
	</head>
	<body>
		<div id="div1">
			河南省郑州市二七区航海中路与京广路交叉口向西100米海为科技园C区10/12层
		</div>
		<div id="div2">
			河南省郑州市金水区农业路*********
		</div>
		<div id="div3">
			河南省郑州市金水区纬五路21号河南教育学院
			<div class="d">
				???
			</div>
		</div>
		<p>占位置</p>
		<p>占位置</p>
		<p>占位置</p>
		<p>占位置</p>
		<p>占位置</p>
		
	</body>
</html>

```


### list
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title></title>
		<style type="text/css">
		
			ul li {
				font-size: 20px;
				font-weight: bolder;
				color: #555;
				list-style-type: none;
				/*list-style-image: url(img/u=168663427,492053552&fm=27&gp=0.jpg);
				list-style-position: inside;*/
				display: inline-block;
				margin-left: 10px;
			}
			
			a {
				text-decoration: none;
			}
			
			a:link {
				color: #555555;
			}
			
			a:visited {
				color: #555555;
			}
			
			a:hover {
				color: #C81623;
			}
		</style>
	</head>
	<body>
		<ul>
			<li><a href="#">秒杀</a></li>
			<li><a href="#">优惠券</a></li>
			<li><a href="#">闪购</a></li>
			<li><a href="#">拍卖</a></li>
			<li>|</li>
			<li><a href="#">京东服饰</a></li>
		</ul>
	</body>
</html>

```



















