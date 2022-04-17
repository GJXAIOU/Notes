# import org.junit.Test 和@Test报错---解决方案大全

2018年01月09日 17:39:20 [cao_yanjie](https://me.csdn.net/cao_yanjie) 阅读数 28507更多

分类专栏： [Java](https://blog.csdn.net/cao_yanjie/article/category/7076325)

[](http://creativecommons.org/licenses/by-sa/4.0/)版权声明：本文为博主原创文章，遵循[ CC 4.0 BY-SA ](http://creativecommons.org/licenses/by-sa/4.0/)版权协议，转载请附上原文出处链接和本声明。

本文链接：[https://blog.csdn.net/cao_yanjie/article/details/79015517](https://blog.csdn.net/cao_yanjie/article/details/79015517) 

在最近用spring框架做项目的时候，发现不定时出现import org.junit.Test 和@Test报错，在包没有导入错误的前提下，该问题的解决方案如下：

![](https://img-blog.csdn.net/20180109174507949?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FvX3lhbmppZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)![](https://img-blog.csdn.net/20180109174547092?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FvX3lhbmppZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![](https://img-blog.csdn.net/20180109173900881?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2FvX3lhbmppZQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

解决方案一：

MyEclipse 在项目右击属性选择【build path】的【add Libararies】界面 ，选择JUnit导入，Eclipse也同样。

解决方案二：

原因是java文件所在的包为默认的匿名包。
新建个com.XXX.XXX有名字的包，把java文件移动到这个名字下就ok了

解决方案三：

写junit测试的java类名为Test.java  ，一定要用Test.java的话只能在方法上加@ org.junit.Test

解决方案四：

该测试类的类名尽量不要用Test，改为其他的类名，我遇到很多次该问题，都是类名用Test的缘故，更改后正常无报错