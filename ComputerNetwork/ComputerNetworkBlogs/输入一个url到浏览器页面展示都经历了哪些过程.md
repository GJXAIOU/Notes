# 输入一个url到浏览器页面展示都经历了哪些过程





### 步骤

`→` **1- 输入网址**
`→` **2- 缓存解析**
`→` **3- 域名解析**
`→` **4- tcp连接，三次握手**
`→` **6- 页面渲染**

* * *

**一：输入网址**

那肯定是输入你要访问的网站网址了，俗称url；

**二：缓存解析**

浏览器获取了这个url，当然就去解析了，它先去缓存当中看看有没有，从 浏览器缓存-系统缓存-路由器缓存 当中查看，如果有从缓存当中显示页面，然后没有那就进行步骤三；
缓存就是把你之前访问的web资源，比如一些js，css，图片什么的保存在你本机的内存或者磁盘当中。

（1） 在chrome浏览器中输入网址： chrome://chrome-urls/ chrome-urls是一个看到所有的Chrome支持的伪RUL，找到其中的chrome://appcache-internals/ 可以看见chrome的本地缓存地址：Instances in: C:\Users\User\AppData\Local\Google\Chrome\User Data\Default (0)

（2）在chrome中访问www.baidu.com/，打开开发者模式，不勾选 Disable cache

![这里写图片描述](https://img-blog.csdn.net/20180719153559536?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI0MTQ3MDUx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

圈出来的部分显示了资源的来源： from disk cache ： 将资源缓存到磁盘中，等待下次访问时不需要重新下载资源，而直接从磁盘中获取；
from memory cache ：将资源缓存到内存中，等待下次访问时不需要重新下载资源，而直接从内存中获取；
可以看见资源的来源是缓存当中，从缓存当中获取了这些就可以直接显示在页面中，不需要发送http请求；

**三： 域名解析**

和步骤二一样，做一个访问新页面的操作juejin.im/timeline，同样打开开发者模式，，不勾选 Disable cache

![这里写图片描述](https://img-blog.csdn.net/20180719153750763?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI0MTQ3MDUx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

可以发现它的来源再也不是： from disk cache 或者from memory cache ，即发送http请求。
那么在发送http请求前，浏览器做了什么？

在发送http之前，需要进行DNS解析即域名解析。
DNS解析:域名到IP地址的转换过程。域名的解析工作由DNS服务器完成。解析后可以获取域名相应的IP地址

**四：tcp连接，三次握手**

在域名解析之后，浏览器向服务器发起了http请求，tcp连接，三次握手建立tcp连接。TCP协议是面向连接的，所以在传输数据前必须建立连接

![这里写图片描述](https://img-blog.csdn.net/20180719150732873?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzI0MTQ3MDUx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

（1）客户端向服务器发送连接请求报文；
（2）服务器端接受客户端发送的连接请求后后回复ACK报文，并为这次连接分配资源。
（3）客户端接收到ACK报文后也向服务器端发生ACK报文，并分配资源。

这样TCP连接就建立了。
在此之后，浏览器开始向服务器发送http请求，请求数据包。请求信息包含一个头部和一个请求体。

**五：服务器收到请求**

服务器收到浏览器发送的请求信息，返回一个响应头和一个响应体。

**六：页面渲染**

浏览器收到服务器发送的响应头和响应体，进行客户端渲染，生成Dom树、解析css样式、js交互。