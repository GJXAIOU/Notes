# JavaEEDay49 Ajax & JSON

@toc

# Ajax

##  一、含义
- 全称：Asynchronous  JavaScript and XML
Ajax 通过与服务器进行**少量的数据交互**，可以使网页实现**异步局部的更新**，意味着可以在不重新加载这个网页的情况下，对网页的某部分进行更新。（传统的网页需要更新内容必须重载整个网页）

Ajax 需要：HTML、CSS、JavaScript、Servlet 做技术支撑；

## 二、传统的客户端与服务器交换数据流程

* 客户端发起一个get或者post请求并且携带参数
* 目前已知客户端发起请求的方式都会有页面跳转效果
* 服务器(jsp, servlet)接受到客户端参数执行操作(业务逻辑判断,数据库增删改查)
* 服务根据执行的结果转发或者重定向

## 三、常见使用 Ajax的地方

- 输入值校验的问题，申请用户的时候检查用户名是否重复
- 层级联动显示的问题。级联菜单,导航树等
- 请求结果只改变部分页面,数据录入和列表显示在同一个页面如,论坛的回复帖子和帖子列表在一个页面上的时候。
- 由于技术原因而使用 iframe的问题避免 iframe的嵌套引入的技术难题
- 翻页,下一页。不需要刷新的翻页


## 四、 Ajax 缺点 

- 对搜索引擎的支持不好(seo,使用 Ajax 会减少搜索引擎对页面的检索)；
- 编写复杂、容易出错(将来会有框架解决此问题)；
- 冗余代码更多了；
- 破坏了Web的原有标准,不在使用<a>；
- 没有back和 history的浏览器，用户界面不能回退；
- **一般使用 Ajax 传输尽量单一和简陋的数据,不要传输大量的数据**。

## 五、Ajax核心类: XMLHttpRequest

XMLHttpRequest 是 Javascript中的类,**执行在客户端**。中文可以解释为可扩展超文本传输请求,此类在js中使用。

但是该类在 IE5 和 IE6 中，必须使用特定于 IE 的 ActiveXObject()构造函数代替。

XMLHttpRequest对象提供了对HTTP协议的完全的访问,包括做出POST和HEAD请求以及普通的GET请求的能力。 XMLHttpRequest可以同步或异步返回Web服务器的响应,并且能以文本或者一个DOM文档形式返回内容。尽管名为 XMLHttpRequest,它并不限于和XML文档一起使用:它可以接收任何形式的文本文档。

- 使用步骤:
  - XMLHttpRequest创建
  - XMLHttpRequest请求
  - XMLHttpRequest响应


### XMLHttpRequest请求相关的方法

- `xhr.open("Get","test.html", true)`
规定请求的类型、URL以及是否异步处理请求。
  * method:请求的类型;GET或POST
  * url:文件在服务器上的位置
  * async:true(异步)或 false(同步)


- 针对于 GET 和 POST 在 send（作用是：将请求发送到服务器） 和 open 使用的区别：
GET 的请求数据可以放在 URL 中以键值对形式传递；
```java
xhr.open("GET", "checkusername?name=" + usernamestr, false);
xhr.send()
```
POST 请求放在 send 里面以键值对的形式传递到服务器;
```java
xhr.open("POST", "checkuser", false);
xhr. send("name=" + usernamestr);
```

**注：**在使用 POST 请求时候，必须在**open 后面**加上下面这句话，目的是告诉服务器，客户端此次请求的数据是一个表单提交，如果不加会造成后台 Servlet 无法通过 getRequestPrarmter 获取到数据
`xhr. setRequestHeader("Content-type", "application/x-www-form-urlencoded");`

### XMLHttpRequest请求过程

- `XMLHttpRequest.onreadystatechange`  存储函数(或函数名)，每当 readystate属性改变时,就会调用该函数。

- `XMLHttpRequest.readyState`  请求状态
存有 XMLHttpRequest的状态。从0到4发生变化。
  * 0:请求未初始化
  * 1:服务器连接已建立
  * 2:请求已接收
  * 3:请求处理中
  * 4:请求已完成,且响应已就绪

- `XMLHttpRequest.status` 响应码
  - 200:表示请求成功
  - 404:未找到页面


## 示例代码






# JSON

**前言：**
因为 Java 语言的对象在 js 中无法直接读取，因此通常使用 XML 作为中间传递数据；
过程为：后台创建 XML 格式的数据发送给客户端，客户端的 js 也有对 XMl 解析的 API，从而实现通过 XML 为媒介进行数据交互；


**JSON 作为 XML 的替代品，作为 Java 和 JS 的数据交换；**


## 概念
JSON（JavaScript Object Notation）是一种轻量级的数据交换方式，它是基于 JS 的一个子集；
- JSON 采用完全独立于语言的文本格式； java /c /object-c 等都对 json 支持；
- 易于阅读和编写，同时易于及其解析和生成；
- 比 xml 更轻量的数据结构；
- json 实质上是一个字符串，一个有特定格式的字符串；

## 语法规则
- json 语法是 JavaScript 对象表示语法的子集；
- 数据在键值对中：`"name":"value"`
- 数据由逗号分隔：`"name1":"value1", "name2":"value2"`
- 花括号表示对象：`{}`
- 方括号表示数组：`[]`
- **json 就是一个字符串，只不过有固定格式的字符串，并且使用这些固定格式可以来描述一个对象，用来表示一个对象或者数组；**

## 书写格式
书写的方式是键值对，例如：`"name" : "value"`

Demo:表示一个对象，属性有：name = zhangsan,age = 23,isman = true
`String jsonPerson = "{\"name\" : \"zhangsan\", \"age\" : 23, \"isman\" : true};"`,其中 int 类型和 Boolean 不需要使用“”


## JSON 允许的值
- 数字（整数或者浮点数）
- 字符串
- 逻辑值（true 或 false）
- 对象 （在花括号中）
- 数组 （在方括号中）
- null



## JSON 基础结构
json 中 map 是属于对象，list 属于数组；
### 对象
对象:对象在js中表示为“{}”括起来的内容,数据结构为{key: value,key: value,...}的键值对的结构,
在面向对象的语言中,key为对象的属性,value为对应的属性值,取值方法为`对象.key`取属性值,这个属性值的类型可以是数字、字符串、数组、对象几种；

### 数组
数组:数组在js 中是中括号“[]”括起来的内容,数据结构为`[["北京市”],[“上海市”],["合肥市”,"芜湖市","蚌埠市"]]`
或者`[{json对象},};},]`,取值方式和所有语言中一样,使用索引获取,字段值的类型可以是数字、字符串、数组、对象几种。

