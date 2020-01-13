---
tags: 
- HTML
style: summer
flag: yellow
---

# JavaDay27 HTML

## 一、复习
- 多线程：
  - 进程是在操作系统中运行的一个应用程序，例如QQ，Word, Eclipse
  - 线程就是在应用程序中的某一个功能：QQ的视频功能和聊天功能，LOL游戏操作和即时聊天，Eclipse多个文件打开
  
  - 创建多线程的两种方式：
      1.继承Thread类，重写run方法，创建自定义线程类对象， 调用start方法，开启线程
      2.【遵从】Runnable接口，实现接口中要求完成的run方法，创建Thread类对象，构造使用的参数，是Runnable接口的实现类，利用Thread类对象的start方法，开启线程
  
  - synchronized 同步代码块 
```java
synchronized (锁对象) {
          同步的代码
      }
```
      
  - 同步函数(还没有讲)
  
- TCP/IP UDP:
  - UDP：
      1.面向无连接，数据是采用数据包的形式发送的
      2.数据包是有大小限制的，每一个数据的大小都是在64KB以内
      3.因为无连接，所以不可靠
      4.因为无连接，所以速度快
      5.UDP没有客户端和服务端，只有发送端和接收端
    - 常用方法： 
      DatagramSocket();
      DatagramPacket(byte[] buf, int length, InetAddress address, int port);
      sender(DatagramPacket packet);
      receive(DatagramPacket packet);
    - 使用场景：
      网络游戏，视频直播
  
  -  TCP：
      1. 面向连接，数据发送时必须经过三次握手
      2. 数据发送没有数据包限制大小
      3. 因为是面向连接，所以传输可靠
      4. 因为是面向连接，所以传世效率低
      5. TCP有客户端和服务端之分
      - 常用的方法：
      Socket(); //客户端创建TCP协议下的Socket传输
      ServerSocket(); 监听端口，捕获客户端发送的连接请求，捕获客户端的socket
      accept(); TCP的三次握手机制
      
      - TCP数据传输是基于IO流的，是通过Socket获取到对应输入和输出流即： getInputStream和getOutStream



## 二、前提知识：

### (一) C/S 和 B/S 架构

 **C/S：客户端和服务器架构**
 **B/S是浏览器和服务器架构**    
相比较而言：
    B/S架构开发，维护的成本要远远低于C/S架构，B/S架构是现在最流行的一种开发方式

## 三、 HTML初识
### （一）标签/属性/元素
- 标签:   
    就是以<>包含的特定字符串，通常用开始标签和结束标签
    - 双边标签：`<标签名>内容</标签名>`
    - 单边标签：`<标签名 />`
- 属性：
    `<标签名 属性1="值1" 属性2='值2' 属性3=值3>内容</标签名>`
        属性都是【键值对】形式
        建议:标签的属性值要使用双引号包含
- 元素:
    一个完整的标签称之为元素

### (二) 全局架构标签：

```java
<!DOCTYPE HTML>
<html>
    <head>
        <!--
            这里是注释
            Head里面的内容是不再页面内部展示的
        -->
    </head>
    <body>
        <!-- 所见即所得 -->
    </body>
</html>
```

- 注意事项：
    1.所有的内容，必须全部写在HTML标签内
    2.head标签里面的内容不会展示在页面当中
    3.所有需要展示的内容，都要放到body里面


- **常见的属性**：
  - <br />表示换行
  - vlink 表示点击后的链接颜色
  - alink 表示鼠标放在上面时链接颜色
  - link 表示没有点击过的链接颜色
  - 网页的超链接标签 
  - Https 安全连接
  - http 不是安全连接
  - text = "green" 文本颜色为绿色
  - bgcolor = " " 整个页面的背景为XXX
 
   - <a href = "地址"></a> 为超链接
```html
<!DOCTYPE HTML>
<html>
	<head>
		<!-- 这里是注释内容 -->
		<!-- 
		meta标签是一个多功能标签，目前使用meta标签来设置页面默认字符集 
		-->
		<meta charset="utf-8" /> 
	</head>
	<body text="green" vlink="gray" alink="red" 
	link="black"  bgcolor="yellowgreen">
		我们三是智障~~ <br />

		<a href="https://www.baidu.com">百度一下,你就知道</a> <br />
		<a href="http://www.163.com">网易</a> <br />
		<a href="http://www.vip.com">唯品会</a> <br />
		
	</body>
</html>
```
![显示样式]($resource/%E6%98%BE%E7%A4%BA%E6%A0%B7%E5%BC%8F.jpg)

### (三) 几乎所有的标签都有的一些属性(结合JS/CSS使用)
  class  name  id  style


### （四）字符实体
 `&nbsp;` 为空格，写几个就是空几个格  `&lt;hello &gt;`显示为<hello >
字符实体是为了解决一些本身具有特殊含义的字符展示，例如" '  ? 
全部的字符表示： www.w3school.com.cn

### (五) 常用的标签(文本修饰)
```html
<h1>H1标签内容</h1>
<h2>H2标签内容</h2>
<h3>H3标签内容</h3>
<h4>H4标签内容</h4>
<h5>H5标签内容</h5>
<h6>H6标签内容</h6>
<b>加粗</b>
<strong>加粗</strong> 
<i>斜体</i>
<em>斜体</em>
<cite>斜体</cite> 
<b><i>加粗斜体</i></b>
<u>下划线</u>
<del>删除线</del>
<s>删除线</s>

<!-- 上标和下标 -->
X<sup>2</sup> 表示：x的平方  y<sub>2</sub> 表示y的下标2
```


### （六）格式控制
- 段落：`<p>  </p>`
- 换行：`<br>`或者 `<br />`
- 水平线（就是一条线）：`<hr />`
- 无序列表：默认是黑圆点
        ul可以加上type属性修改源点样式，属性值:circle（圆空心） square（方黑心） none （没有） desc（）
```html
<ul type = “circle” >
            <li>啦 1</li>
            <li>啦 2</li>
            <li>啦 3</li>
</ul>
```
  
- 有序列表：
        ol可以添加type属性，属性值可以是A/a/I/1 还可以有start 从哪一个开始
```html
  <ol type = “a” start = “2”>
    <li></li>
    <li></li>
    <li></li>
</ol>
```
- 自定义列表：
```html
<dl>
    <dt>标题</dt>
    <dd>内容一</dd>
    <dd>内容二</dd>
    <dd>内容三</dd>
</dl>
```
         
### （七）超链接标签：
超链接标签为：`<a></a>`
    href:目标地址/mailto/tel
    target:打开页面的方式， `_blank` 表示 新建空白页，重新打开
    title：鼠标悬停之后的提示
    name:本页面内 可以作为锚点，跳转
其他页面标签作为锚点：可以用id属性
注：
```html
<!--本页面内跳转-->
<a href = "#名字1"> 从这里跳 </a>
<a name = "名字1" href = "#"> 跳到这里,通过#可以调回去 </a>

<!--不同页面间跳转-->
<a href = "跳转去的页面名#名字2"> 从这里跳 </a>
<a href = "url#名字2"> 从这里跳 </a>
<a id = "名字2"> 跳到这里,通过#可以调回去 </a>
```

```html
<body>
	<a href="http://www.baidu.com" target="_blank">百度一下,你就知道</a> <br />
	<a href="tel:15565311790" title="打电话给杰哥">呼叫杰哥</a> <br />
	<a href="mailto:liuxiaolei@1000phone.com">发邮件给磊哥</a> <br />
	<a href="7.html#dst" target="_blank">看看你去哪了</a> <br/> <!----在另一个地方放入：<b id="dst">目标位置</b>>
	<a href="#C">跳到C位置</a>
	<p>记者： 台湾军方23日称，解放军轰-6K等各型军机开展出岛链远海训练，
		其中部分机种经巴士海峡飞往西太平洋后沿原航路返回基地，台湾军方全程依规派舰机</p>
				
	<!-- a标签的name属性作为锚点 -->
	<a name="C" href="#">C位置,返回</a>
		
</body>
```

上面链接中对应的 7.html 文件如下：
```html
<body>		
		我在这里强调一下，没有什么锁链能够锁住中国。	
	<!-- 其他标签利用id属性作为锚点 -->
	<b id="dst">目标位置</b>
	<p>记者： 台湾军方23日称，解放军轰-6K等各型军机开展出岛链远海训练，			
	</p>
	
</body>
```


### （八）图片
图片表示：`<img />`
    src: 文件路径:可以是本地的绝对路径或者网络地址
    width:宽度
    height:高度
    alt:图片显示不出来的时候，用该字段串代替；
```html
<body>
    <!--这里的alt为：如果无法显示图片就用该参数值代替显示，
    title含义：当鼠标放置在图片上时，会显示该参数值-->
	<img src="img/timg.jpg" width="50" height="50" alt="阿狸" title="测试阿狸"/>		
</body>
```


### （九）统一资源定位符 URL：

https://www.baidu.com/s? // ?前面表示地址，后面表示参数
wd=%E9%83%AD%E8%BE%BE%E6%96%AF%E5%9D%A6%E6%A3%AE
rsv_spt=1&

**格式**：协议://地址:端口/文件?键1=值1&键2=值2
例如: https://www.baidu.com:80/index.html?page=2&user=lxl
    

### （十）多媒体标签：
- audio属性：src(地址) controls（控制栏） autoplay （自动播放） loop（循环）
-  video多两个属性：width 和 height
```html
<body>
	<audio src="audio/Ryan.B,AY楊佬叁 - 再也没有.mp3" 
		autoplay controls></audio>
	<video src="video/搞笑视频.mp4" autoplay controls></video>
</body>
```
