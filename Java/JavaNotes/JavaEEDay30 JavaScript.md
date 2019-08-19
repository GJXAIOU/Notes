---
tags:
- JavaScript
---

# JavaEEDay30 JavaScript
@toc

JavaScript 是 Web 的编程语言。
1.HTML定义了网页的内容
2.CSS描述了网页的布局
3.JavaScript 网页的行为


# JavaScipt 基本语法
## 一、 JS 函数定义
```js
<<body>
	<script type = "text/javascript">
		//定义函数
		function method(){
			alert("这是以页面顶部弹窗的形式形式");
		}
		function method2(name, id){
			alert("name + " + name + "  id +  " + id );
		}
		
		function method3(num1, num2){
			return num1 + num2;
		}
		
		
		//调用函数
		method();
		method2(23,"hello");
		 var i = method3(23,12); //定义变量接收返回值
		console.log("i : " + i);  //在页面的console控制台中输出
		
		$j = method3(1, 2); //另一种定义变量的方式
		console.log("j : " + j);
</body>>
```


具体的 Demo 示例：

```js
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>第一运行JS</title>
		<!--
			这里在head标签里面借助于script标签引入外部的JS文件
			type="text/javascript" 可视化的JS文件
			src="JS文件路径"，表示这个JS文件在什么位置，要引入的是哪一个
			【注意】
			这里script标签是一个双边标签，但是在使用Script标签引入外部的JS文件
			时，里面不能有任何的内容！！！
		-->
		<script type="text/javascript" src="hello.js" ></script>
		
		<!--
			【注意】
			如果说不需要JavaScript对于页面进行绘制，则不需要把script标签
			定义在head标签里面，只需要放到body里面就可以了
		-->
		<script type="text/javascript">
			//这里是在Script标签里面来书写JS代码
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
			console.log("ret : " + ret);  //在页面中的控制台中输出
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
var 定义变量，则该变量 为局部变量， 只能在方法的内部使用；
不使用 var 定义变量，则该变量为全局变量，可以在任何地方使用；
- 在函数（方法）外部:
无论使用什么形式定义变量，该变量均为全局变量，可以在任何地方使用；
```js
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>定义变量</title>
	</head>
	<body>
		<script type="text/javascript">
		/* 
		定义变量
		 var是一个关键字，只是用来声明这是一个变量 ，不带有任何数据类型约束
		 变量的数据类型有：number/string/boolean/undefined(未赋值)/object类型(值为Null时候）
		*/
			var i = 23;  
			console.log(typeof i);//number 
			
			var s;
			console.log(typeof s); //undefined在定义时未赋初值，无法确定数据类型
			
			n = null;//这里可以这样定义变量，但是有一些限制
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
</html>

```

## 三、循环
for循环和 switch-case 使用和 Java 几乎相同：
- 不同点：
在JavaScript当中，for循环的大括号不能作为【变量作用域】的约束
只有JavaScript里面方法大括号才可以作为【变量作用域的约束】
即 for 循环参数中定义的变量在 for 循环外部也可以使用；
```html
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>分支循环</title>
	</head>
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
</html>

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
				
				var urlStr = "wd=电脑装机&rsv_spt=1&rsv_iqid=0xfc9f6b8300001a58&issp=1&f=3&rsv_bp=0&rsv_idx=2";
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

## 八、 Dom 和事件
		
getElementById();
通过ID属性获取到对应的元素对象

getElementsByClassName();
通过Class属性，获取到对应的元素对象数组

getElementsByName();
通过Name属性，获取到对应的元素对象数组

getElementsByTagName();
通过标签名，获取到对应的元素对象数组
			 			
```js
<!DOCTYPE html>
<html>
	<head>
		<meta charset="UTF-8">
		<title>DOM和事件</title>
		<!--
			DOM :Document Object Model
			在JS中可以把页面里面的所有标签看做是一个对象
		-->
		<style>
			#ydn { //id标签使用#
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
