## IntelliJ IDEA 17 创建maven项目

[家宝](https://yq.aliyun.com/users/apeutkgdkwjw2) 2017-06-26 18:10:34 浏览9446

*   [系统软件](https://yq.aliyun.com/tags/type_blog-tagid_4/)

*   [编程语言](https://yq.aliyun.com/tags/type_blog-tagid_5/)

*   [java](https://yq.aliyun.com/tags/type_blog-tagid_41/)

*   [http](https://yq.aliyun.com/tags/type_blog-tagid_557/)

*   [web](https://yq.aliyun.com/tags/type_blog-tagid_696/)

*   [Maven](https://yq.aliyun.com/tags/type_blog-tagid_1014/)

*   [xml](https://yq.aliyun.com/tags/type_blog-tagid_1518/)

*   [html](https://yq.aliyun.com/tags/type_blog-tagid_2236/)

*   [Servlet](https://yq.aliyun.com/tags/type_blog-tagid_2384/)
*   [IDEA](https://yq.aliyun.com/tags/type_blog-tagid_2509/)

*   [utf-8](https://yq.aliyun.com/tags/type_blog-tagid_2681/)

# 说明

*   创建Maven项目的方式：**手工创建**
*   好处：参考[IntelliJ IDEA 17创建maven项目二](https://yq.aliyun.com/go/articleRenderRedirect?url=http%3A%2F%2Fwww.cnblogs.com%2Fwql025%2Fp%2F5205716.html)（此文章描述了用此方式创建Maven项目的好处）及[idea17使用maven创建web工程](https://yq.aliyun.com/go/articleRenderRedirect?url=http%3A%2F%2Fwww.cnblogs.com%2Fwql025%2Fp%2F5015057.html)（此文章描述了用模板创建Maven的弊端。）

# 创建一个新Maven项目

*   new 一个project

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160224215012240-1065055961.png)

*   **不选择任何Maven模板**

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160224215039974-537020396.png)

*   起个GroupId、ArifactId

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160224215124583-1065420029.png)

*   起个项目名。注意：Idea_Project是存放此项目的工作区间，mavenDemo_idea15为存放此项目的子目录。

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160224215235005-1391385829.png)

*   建好项目后，打开，点击Auto-Import

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160224215419802-817948119.png)

*   下面为此项目的结构

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160224215502146-429875408.png)

# 项目部署

*   点击

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160224215545208-902358365.png)

## Project: 无需设置 （当然可点击Project complier output自定义编译目录）

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160224215607771-1984494060.png)

## Modules:**可看到此项目无任何适配服务组件（因为是手工创建Maven，没有选择任何Maven模板）--因此需要我们进行添加。**

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160224215919396-86983636.png)

*   选择Web（为此项目添加Web服务组件，这便是一个Web项目了）

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160224220134802-1514004998.png)

*   现在为Web设置资源目录。双击Web Resource Directory

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160224220305380-142824778.png)

*   选择scr/main目录

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160224220559458-128756380.png)

*   在后面加上webapp。好了，点OK，Web的资源目录便设置好了。

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160224220655130-1637259503.png)

*   现在设置Web的描述文件的目录

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160224220749411-825717204.png)

*   设置在webapp目录下

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160224220856536-1927192653.png)

## Facts: 表示当前项目的适配服务组件。可看到此项目已是一个Web项目了。

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160224221000458-1246044489.png)

## Aftifacts: 这个Aftifacts描述了当前项目发布的信息。现在进行添加，从Modeles中选择。

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160224221233786-1051320504.png)

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160224222957521-83778449.png)

**说明：A: 现在Artifacts已有了发布的项目了（idea中准确的说应是Modele） B：_output root目录描述了当前项目的编译目录及适配服务。_**

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160224223342396-417104738.png)

确定之后当前项目的结构：

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160224233751083-1977822404.png)

*   如有需要，添加lib包

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160225001330536-185190312.png)

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160225001337255-2034413311.png)

# 部署服务器

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160224233904818-1626127517.png)

*   添加服务器

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160224233918646-1203588879.png)

*   部署

注：很多童鞋在这里找不到Arifact，请参考部署项目中的Modules的配置。如果没有为项目配置Web服务组件，那么就没有Artifact。（当前项目连Web项目都不是，哪儿来的Artifact，又部署什么呢？）

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160224233953865-792827627.png)

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160224234005068-1029599599.png)

*   **注意下面的选择：**

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160224234103880-734957972.png)

# 编写代码测试

*   创建一个java类。**可看到继承HttpServlet出问题了--这是因为没有把Tomcat的依赖加入Module**

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160225002926099-94225678.png)

*   在Modules加入Tomcat依赖

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160225002941833-5215041.png)

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160225002953630-1950910896.png)

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160225003003427-157546013.png)

添加完毕

![](http://images2015.cnblogs.com/blog/797348/201602/797348-20160225003011318-295062355.png)