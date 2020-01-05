---
tags:
- JavaScript
---

# JavaEEDay30 JavaScript
@toc

JavaScript 是 Web 的编程语言。
1.HTML 定义了网页的内容
2.CSS 描述了网页的布局
3.JavaScript 网页的行为


# JavaScipt 基本语法
## 一、 JS 函数定义
```js
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>第一运行JS</title>
		<!--
    		这里在 head 标签里面借助于 script 标签引入外部的 JS 文件
    		type="text/javascript" 可视化的 JS 文件
    		src=" JS 文件路径"，表示这个 JS 文件在什么位置;
    		【注意】这里 script 标签是一个双边标签，
    		但是在使用 Script 标签引入外部的 JS文件时，里面不能有任何的内容！
		-->
		<script type="text/javascript" src="hello.js" ></script>
		
		<!--
			【注意】如果说不需要 JavaScript 对于页面进行绘制，
			则不需要把script标签定义在head标签里面，
			只需要放到body里面就可以了
		-->
		<script type="text/javascript">
			//这里是在 Script 标签里面来书写JS代码
			alert("这里是JavaScript");
			
			//这里是在JavaScript里面定义函数
			function method() {
				alert("这里是一个JavaScript的方法");
			}
			
			function method(age, name, size) {
				alert("age:" + age + " name:" + name + " size:" + size);
			}
			
			function method(num1, num2) {
				return num1 + num2;
			}
			
			method(18, "貂蝉", "167");			
			var ret = method(5, 10);
			//在页面中的控制台中输出
			console.log("ret : " + ret); 
			$s = method(10, 15);
			console.log("s:" + $s);
		</script>
	</head>
	<body>
	</body>
</html>
```

## 二、定义变量
- 在函数（方法）内部：
var 定义变量，则该变量为局部变量， 只能在方法的内部使用；
不使用 var 定义变量，则该变量为全局变量，可以在任何地方使用；
- 在函数（方法）外部:
无论使用什么形式定义变量，该变量均为全局变量，可以在任何地方使用；
```js
<body>
	<script type="text/javascript">
	/* 
	定义变量
	 var是一个关键字，只是用来声明这是一个变量 ，不带有任何数据类型约束
	 变量的数据类型有：number/string/boolean/undefined(未赋值)/object类型(值为Null时候）
	*/
		var i = 23;  
		//number 
		console.log(typeof i);
		
		var s;
		//undefined在定义时未赋初值，无法确定数据类型
		console.log(typeof s); 
		
		//这里可以这样定义变量，但是有一些限制
		n = null;
		console.log(typeof n); //object
		
		function varTest() {
			m = 10;
			var t = 20;
			console.log("--->" + i); //---->true
			console.log("--->" + n); //---->null
		}
		
		varTest();
		
		console.log("--->" + m); //--->10
		console.log("--->" + t); //报错；方法内使用var定义变量为局部变量
	</script>
</body>
```

## 三、循环
for循环和 switch-case 使用和 Java 几乎相同：
- 不同点：
在 JavaScript 当中，for循环的大括号不能作为【变量作用域】的约束
只有 JavaScript 里面方法大括号才可以作为【变量作用域的约束】
即 for 循环参数中定义的变量在 for 循环外部也可以使用；
```html
<body>
	<script type="text/javascript">
	
  //switch循环
		var i = 2;
		switch (i){
			case 1:
				console.log("这里是switch-case 1");
				break;
			case 2:
				console.log("这里是switch-case 2");
				break;
			default:
				console.log("搞事情！！！");
				break;
		}

          //for循环
		for (var j = 0; j < 10; j++) {
			console.log("--->" + j);
		}
		console.log("???" + j); //？？？10

          //while循环
		var k = 10
		while (k-- > 0) {
			if (0 == k % 2) {
				console.log("JS就是这么灵活");
			} else {
				console.log("你觉得好玩吗?");
			}
		}
	</script>
</body>
```

## 四、字符串
整体的处理思路相似，

```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>字符串方法</title>
	</head>
	<body>
		<script type="text/javascript">
			function stringTest() {
			
			    //新建字符串
				var str = "字符串是一个让人又爱又恨的东西";
		        //截取字符串
				var subString = str.substring(2, 4);				
				console.log("subString:" + subString);  //subString:串是
				
				var urlStr = "wd=电脑&rsv_spt=1&rs1&f=3&rsv_bp=0&rsv_idx=2";
			    //分割字符串
				var arrays = urlStr.split("&");
				
				console.log(arrays);
				
				//寻找rsv_bp后面的值
				for (var i = 0; i < arrays.length; i++) {
					if (arrays[i].indexOf("rsv_bp") == 0) {
						console.log(arrays[i].substring(arrays[i].indexOf("=") + 1));
					}
				}
				
//在forEach里参数是一个Callback 回调函数，给当前forEach提供处理数据的方式和方法
				arrays.forEach(function(item) {
					console.log("forEach:" + item);
				});
			}
			
			stringTest();
		</script>
	</body>
</html>

```

## 五、date 方法
```js
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>日期使用</title>
	</head>
	<body>
		<script type="text/javascript">
			
			//获取当前系统使用 new Date();
			var date = new Date();
			
			console.log(date);
			
			var day = date.getDate();
			var month = date.getMonth() + 1; //因为从0开始
			var year = date.getFullYear();
			var seconds = date.getSeconds();
			var minutes = date.getMinutes();
			var hours = date.getHours();
			
			var time = year + "年" + month + "月" + day + "日" 
				+ hours + ":" + minutes + ":" + seconds ;
				
			document.writeln(time);//在页面中打印
			document.writeln("<br />");
			document.writeln("<hr />");
			document.write("测试");
			document.write("在测试");
			
		</script>
	</body>
</html>

```

## 六、 Math
```js
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>Math</title>
	</head>
	<body>
		<script type="text/javascript">
			var i = Math.random();
			document.writeln(i);			
			document.writeln("<hr />");
			
			var xiangshangquzheng = Math.round(5.1); //向上取整
			document.writeln(xiangshangquzheng);	
			document.writeln("<hr />");
			
			var xiangxiaquzheng = Math.floor(5.1); //这三个里面参数为字符串数字也行
			document.writeln(xiangxiaquzheng);	
			document.writeln("<hr />");
			
			var n = Math.round(5.5); //四舍五入
			document.writeln(n);
		</script>
	</body>
</html>

```


## 七、数组的使用
```js
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>数组的定义</title>
	</head>
	<body>
		<script type="text/javascript">
			var arrays = new Array();//new ArrayList(); 类似于Java里面的ArrayList
			arrays[0] = "云姨";
			arrays[1] = "云婶儿";
			arrays[2] = "云大娘";
			
			for (var i = 0; i < arrays.length; i++) {
				console.log(arrays[i]);
			}
			
			arrays.forEach(function(item) {
				console.log(item);
			});
			
			var doubleArray = [["a", "b", "c"],["d", "e", "f"]];
			
			for (var i = 0; i < 2; i++) {
				for (var j = 0; j < 3; j++) {
					console.log(doubleArray[i][j]);
				}
			}
			
			//高大上
			doubleArray.forEach(function(arr) {
				arr.forEach(function(item) {
					console.log("--->" + item);
				});
			});
		</script>
	</body>
</html>

```

## 八、 Dom 和事件（最重要）
		
- `getElementById();` 通过 ID 属性获取到对应的元素对象
- `getElementsByClassName();` 通过 Class 属性，获取到对应的元素对象数组
- `getElementsByName();` 通过 Name 属性，获取到对应的元素对象数组
- `getElementsByTagName();` 通过标签名，获取到对应的元素对象数组			 			
```js
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title> DOM 和事件</title>
		<!--
			DOM :Document Object Model
			在 JS 中可以把页面里面的所有标签看做是一个对象
		-->
		<style>
			#ydn { // id 标签使用#
				color: yellow;
				font-size: 35px;
			}
			
			.yy {
				color: green;
				font-size: 20px;
			}
			
			p[name=ys] {
				color: red;
				font-size: 27px;
				display: none;
			}
		</style>
	</head>
	<body>
		<p id="ydn" class="yy">
			云大娘准备考驾照，（报名三年了，科目一还没考呢~~）
		</p>
		<p class="yy">
			云姨准备在众意西路买一套房子，也就4W一平米
		</p>
		<p name="ys">
			云婶准备买辆五菱荣光~~~
		</p>
		<!--
			这里是button标签，是一个按钮
			click点击
			使用button的onclick属性，属性的值是一个函数名()
		-->
		<button onclick="modify()">云大娘考驾照</button> <br />
		<button onclick="buyHouse()">云姨买房子</button> <br />
		<button onclick="buyCar()">云婶车</button> <br />
		<script type="text/javascript">
			function modify() {
				/*
				 * 通过ID属性获取到指定的标签，因为在HTML页面中，ID属性不可以重复
				 * getElementById(id属性)，返回一个确定的标签
				 */
				var aPtag = document.getElementById("ydn");
				aPtag.style.color = "purple"; //实际上就是在后面加上殊勋:style color = "purple"
			}
			
			function buyHouse() {
				/*
				 * getElementsByClassName();
				 * Elements 返回的是一个标签的数组，因为Class选择器可以用到多个标签上
				 */
				var pArray = document.getElementsByClassName("yy");
				pArray[1].innerHTML = "房子也不大，也就200平~~~";
			}
			
			function buyCar() {
				var pName = document.getElementsByName("ys");
				pName[0].style.display = "block"; //因为原来的值隐藏了,这里加上一个块就会显示出来
			}	
		</script>
	</body>
</html>

```

##  Js代码用 script 调用时，放在 head 与 body 的区别
**以下默认 head 标签在 body 标签上面**，因为 HTML 的加载顺序是从上到下的，如果先写<body></body>再写<head></head>，就会先执行body的内容。
只是浏览器加载顺序是从上到下，放在head中时，会在页面加载之前加载到浏览器里，放在body中时，会在页面加载完成之后读取。

**如果把 javascript 放在 head 里的话，则先被解析,但这时候 body 还没有解析。（常规 html 结构都是 head 在前，body 在后）如果 head 的 js 代码是需要传入一个参数（在 body 中调用该方法时，才会传入参数），并需调用该参数进行一系列的操作，那么这时候肯定就会报错，因为函数该参数未定义（undefined）。**

因此定义一个全局对象，而这个对象与页面中的某个按钮（等等）有关时， 我们必须将其放入 body 中，道理很明显：如果放入 head，那当页面加载 head 部分的时候，那个按钮（等等）都还没有被定义（也可以说是还没有被加载，因为加载的过程就是执行代码的过程，包括了定义），你能得到的只可能是一个 undefind。

什么时候应该将其写在body主体里呢？为了实现某些部分动态地创建文档。 这里比如制作鼠标跟随事件，肯定只有当页面加载后再进行对鼠标坐标的计算。或者是filter滤镜与javascript的联合使用产生的图片淡入淡出效果等。这个是在页面加载的时候加载。 

 而为什么我们经常看到有很多的人把js脚本放到head里面都不担心出问题？**因为通常把javascript放在head里的话，一般都会绑定一个监听，当全部的html文档解析完之后，再执行代码**：`$(document).ready(function(){//执行代码})`

除此之外，**从JavaScript对页面下载性能方向考虑：**由于脚本会阻塞其他资源的下载（如图片等）和页面渲染，直到脚本全部下载并执行完成后，页面的渲染才会继续，因此推荐将所有的<script>标签尽可能放到<body>标签的底部，以尽量减少对整个页面下载的影响。例如：

```
<html>
​<head>
   <title>Example</title>
   <link rel="stylesheet" type="text/css" href="style.css">
​</head>
​<body>
   <p>Change the world by simple products!</p>
   <script type="text/javascript" src="test1.js"></script>
</body>
​</html>
```
