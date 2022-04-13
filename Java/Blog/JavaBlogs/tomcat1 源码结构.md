## [tomcat学习|tomcat源码结构 ]

## 学习背景

提到tomcat,相信大家都不陌生,只要是搞web开发的,老师总会让我们把应用打成war包,然后再丢到tomcat的webapp里面,然后就可以用ip:port来访问了
再后来工作之后,我们现在用springboot ,可以直接打成jar包,只用引一个web-starter就可以启动tomcat了,实际上里发生着什么,我们一点都不知道,那是不是就代表着我们可以不用学tomcat了? 当然不,我们要学tomcat , demo级别的工程不用深入去研究,但是当工程进入生产环境,会有各种各样的问题,我们就要深入其原理,然后进行调优
tomcat 小刀也是才刚刚开始学, 这个系列,小刀会和大家一起学习下去

## 学习准备

idea: 看源码,写代码的不二之选
tomcat 源码: 小刀fork 了一份,新建了一个分支,写一些注释之类的
[https://github.com/weixiaodexiaoxiaodao/tomcat](https://github.com/weixiaodexiaoxiaodao/tomcat)
分支是 study_8_5
笔,本子: 好记性不如烂笔头,tomcat做为一个web容器中大佬级别的存在,只用肉眼,很难看穿他


## 拉下代码

用idea把源代码拉到本地, 切换好分支,现在目录结构应该是这个样子的:
![image.png](https://img.hacpai.com/file/2019/08/image-db6f8557.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)
可以看到这是一个基于ant的工程,那我们就对build.xml点右键,然后
`Add as Ant Build File`
通过右侧的ant 任务列表,我们可以看到,有一个任务名为: ide-intellij,我们对应的在build.xml中找到这个target 可以看到相关说明:
![image.png](https://img.hacpai.com/file/2019/08/image-f46d1dc6.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)
我们按照说明去配置环境变量和建包
![image.png](https://img.hacpai.com/file/2019/08/image-31d8abc9.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)
然后运行ide-intellij这个任务,然后项目就会下载包之类的等等,等他运行完

## 目录结构

目前看代码目录,代码应该都在java目录,我们就把java目录标记为源代码目录
![image.png](https://img.hacpai.com/file/2019/08/image-43278fdc.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)
在这里,我们可以看到tomcat最上层的几大模块,这里先介绍下他们的作用,实现等后面我们再一起学习
![image.png](https://img.hacpai.com/file/2019/08/image-9e4c4b34.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)

### Catalina

Catalina是Tomcat提供的Servlet容器实现,负责处理来自客户端的请求并输出响应,
里面有Server ,Service ,Connector,Container, Engine,Host,Context,Wrapper,Executor ,这些概念,现在小刀也只是看了个大概,下次我们学习Catalina的时候再细看这些

### Coyote

Coyote是Tomcat链接器框架的名称,是Tomcat服务器提供的供客户端访问的外部接口,客户端通过Coyote 与Catalina容器进行通信. 我们比较熟悉的Request, Response 就是来自于Coyote模块
![image.png](https://img.hacpai.com/file/2019/08/image-3952bca8.png?imageView2/2/w/1280/format/jpg/interlace/1/q/100)
Coyota将Socket输入转换为Request对象交给Catalina, 然后Catalina处理完之后再转成Response返回Coyota

### el

Expression Language, java表达式语言, 这个对应的就是我们jsp中取值的那些,由于现在我们对页面要么是前后端分离,要么是使用模板语言如freemarker , thymeleaf , 所以这块倒可以不用怎么深入,到时候我们也会大致的看一看

### jasper

Tomcat的jsp引擎,我们可以在jsp中引入各种标签,在不重启服务器的情况下,检测jsp页面是否有更新,等等,还是上面那句话,现在前后端分离比较多,以后的学习,我们也以关注上面的Catalina和Coyota为主

### jui ,naming ,tomcat

这三个就并在一起说吧
jui是日志相关的
naming 是命名空间,JNDI,用于java目录服务的API,JAVA应用可以通过JNDI API 按照命名查找数据和对象,常用的有: 1.将应用连接到一个外部服务,如数据库. 2\. Servlet通过JNDI查找 WEB容器提供的配置信息
tomcat 是一些附加功能,如websocket等