*   [![](https://csdnimg.cn/cdn/content-toolbar/csdn-logo_.png?v=20190924.1)](https://www.csdn.net/ "CSDN首页")
*   [首页](https://www.csdn.net/ "首页")
*   [博客](https://blog.csdn.net/ "博客")
*   [学院](https://edu.csdn.net/ "学院")
*   [下载](https://download.csdn.net/ "下载")
*   [论坛](https://bbs.csdn.net/ "论坛")
*   [图文课](https://gitchat.csdn.net/?utm_source=csdn_toolbar "图文课")
*   [问答](https://ask.csdn.net/ "问答")
*   [商城](https://h5.youzan.com/v2/showcase/homepage?alias=BUj3rrGa2J&ps=760 "商城")
*   [活动](https://huiyi.csdn.net/ "活动")
*   [专题](https://spec.csdn.net/ "专题")
*   [招聘](http://job.csdn.net/ "招聘")
*   [ITeye](http://www.iteye.com/ "ITeye")
*   [GitChat](https://gitbook.cn/?ref=csdn "GitChat")
*   [APP](https://www.csdn.net/apps/download?code=pc_1555579859 "APP")
*   [VIP会员](https://mall.csdn.net/vip "VIP会员")续费8折

[![](https://csdnimg.cn/cdn/content-toolbar/csdn-sou.png?v=20190924.1)](https://so.csdn.net/so/)

*   [![](https://csdnimg.cn/cdn/content-toolbar/csdn-write.png)写博客](https://mp.csdn.net/postedit)
*   [![](https://csdnimg.cn/public/common/toolbar/images/messageIcon.png)](https://i.csdn.net/#/msg/index)
*   [![](https://profile.csdnimg.cn/7/A/6/2_gaojixu)](https://i.csdn.net/)

转载

# 使用Navicat for MySQL把本地数据库上传到服务器

2018-05-09 00:18:11 [kkfd1002](https://me.csdn.net/kkfd1002) 阅读数 7996

## 　　服务器系统基本都是基于linux的，这个数据库上传的方式适用于linux的各种版本，比如Ubuntu和Centos（尽管这两个版本各种大坑小坑，但至少在数据库传输上保持了一致性）

## 　　当然本地数据库上传到服务器的前提是，服务器也已经安装好了MySQL数据库

# 1.在服务器端：

## 1.linux如何查看mysql是否启动

service mysqld status

mysqld is stopped 那就说明mysql服务是停止状态

mysqld is running 那就说明mysql服务是启动状态 

## 2.重启mysql

service mysqld restart

## 3.登录mysql

#如果是刚刚安装了mysql，密码为空，直接按Enter键（回车）就进去了，如果已经设置了密码，就填写密码登录
mysql -uroot -p

## 4.给远程访问设置权限

![复制代码](https://common.cnblogs.com/images/copycode.gif)

#其中123456是用于连接的密码，读者可以将其设置得更加复杂一些
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;

FLUSH PRIVILEGES; #设置密码，如果是新安装的mysql需要在这里把密码设置了，如果已经有密码了就不用了
set password =password('123456');
flush privileges;

![复制代码](https://common.cnblogs.com/images/copycode.gif)

## 5.退出mysql

quit

# 2.在本地：

## 1.安装Navicat

　　虽然这个软件是收费的，但是给了20天试用期，所以可以放心大胆的用正版，毕竟，上传数据库这种事儿，只在项目上线部署的时候用一次，20天怎么也够用了。

## 2.建立连接

### 1.先新建连接，跟本地数据库连上，连接名随便起一个就可以，如图

图1

![](https://images2018.cnblogs.com/blog/1282071/201805/1282071-20180508232234684-1286027701.png)

图2

![](https://images2018.cnblogs.com/blog/1282071/201805/1282071-20180508232526306-1092687796.png)

图3

![](https://images2018.cnblogs.com/blog/1282071/201805/1282071-20180508232725823-684455099.png)

### 2.再新建连接，跟服务器数据库连上，连接名也随便起一个就可以，如图

图1

 ![](https://images2018.cnblogs.com/blog/1282071/201805/1282071-20180508233336230-1031165498.png)

图2

![](https://images2018.cnblogs.com/blog/1282071/201805/1282071-20180508233404808-663892391.png)

### 3.数据传输，如图

图1

![](https://images2018.cnblogs.com/blog/1282071/201805/1282071-20180508233604890-1805995381.png)

图2

![](https://images2018.cnblogs.com/blog/1282071/201805/1282071-20180508234118218-1284811614.png)

图3

![](https://images2018.cnblogs.com/blog/1282071/201805/1282071-20180508234339746-1370258325.png)

#  至此，完成了本地数据库传输到服务器的过程

## 可以到服务器端，进入mysql查看一下，是否已经上传成功：

![](https://images2018.cnblogs.com/blog/1282071/201805/1282071-20180508234559565-984976539.png)

http://www.xfz1235.cn/
http://news.koi7857.cn/
http://www.dsd3012.cn/
http://www.ife7579.cn/
http://www.gtt6107.top/
http://www.ewh1005.cn/
http://www.tzr5175.cn/
http://news.wqr1047.cn/
http://www.zps7191.cn/
http://news.taj7240.cn/
http://news.hud3144.cn/
http://www.bpa2365.cn/
http://news.kjx4882.cn/
http://www.ece1729.cn/
http://news.vii0197.cn/
http://www.evv5980.cn/
http://www.wsa2392.cn/
http://www.mzj8672.cn/
http://news.tzr5175.cn/
http://www.vdx0926.cn/
http://news.xjy3902.cn/
http://www.jfs6888.top/
http://news.ece1729.cn/
http://www.tqz4909.cn/
http://www.qeu2095.top/
http://news.dkk2480.cn/
http://news.oek0353.cn/
http://www.myo1179.cn/
http://www.iit3286.cn/
http://news.hyt6211.cn/
http://www.lhl7110.cn/
http://www.syh5891.cn/
http://www.mfu9569.cn/
http://www.tud8565.cn/
http://www.ewv7964.cn/
http://news.ube1531.cn/
http://www.hyt6211.cn/
http://www.dkk2480.cn/
http://news.ppr6189.cn/
http://www.vdf1425.cn/
http://www.xvn7640.cn/
http://www.bbe1708.cn/
http://news.nat5354.cn/
http://news.aht8537.cn/
http://news.eko5785.cn/
http://news.alj9141.cn/
http://news.xip7382.cn/
http://www.veo6686.cn/
http://news.dfr2203.cn/
http://www.nuq3623.top/
http://www.oiv1998.top/
http://www.gum4900.top/
http://www.xua4102.cn/
http://news.vtl3405.cn/
http://news.aua2439.cn/
http://news.yxp3496.cn/
http://news.grp2563.cn/
http://www.epv8502.cn/

文章最后发布于: 2018-05-09 00:18:11

展开阅读全文 

有 0 个人打赏

 [#### SQL从本地怎样上传_数据库_到_服务器_

阅读数 1万+](https://blog.csdn.net/skyfreedoms/article/details/6040618 "SQL从本地怎样上传数据库到服务器") 

[     许多网站制作者在本地把网站做好之后与调试好以后就要传到虚拟主机上去了,但MSSQL数据库有点不好传,用网上常用的先备份再还原的方法不行,提示没有操作权限出错时,不防试试这个方法.　　其实主要...](https://blog.csdn.net/skyfreedoms/article/details/6040618 "SQL从本地怎样上传数据库到服务器")博文[来自： skyfreedoms的专栏](https://blog.csdn.net/skyfreedoms)

[![](https://profile.csdnimg.cn/7/A/6/3_gaojixu)](https://me.csdn.net/gaojixu)

 [#### 本地导入大量数据至_服务器_端（_Mysql__数据库_）最快的方法

阅读数 1695](https://blog.csdn.net/qq_28137309/article/details/82431595 "本地导入大量数据至服务器端（Mysql数据库）最快的方法") 

[关于本地环境，phpstudy和wamp，相信大家都不陌生了。而Mysql是数据的存储地，直接上干货mysql的data目录下面就是我们的数据库，对应的数据库中的表又分为frm,MYD,MYI.分别代...](https://blog.csdn.net/qq_28137309/article/details/82431595 "本地导入大量数据至服务器端（Mysql数据库）最快的方法")博文[来自： Shelly Long的博客](https://blog.csdn.net/qq_28137309)

 [#### 【入门篇】篇四、将本地Web项目部署到_服务器_，迁移本地_数据库_到_服务器_

阅读数 5774](https://blog.csdn.net/qq_31772441/article/details/80637350 "【入门篇】篇四、将本地Web项目部署到服务器，迁移本地数据库到服务器") 

[前言1.远程连接服务器，执行命令行的SSH工具，使用PuTTY，点击下载2.将本地Web项目打包上传至服务器需要使用FTP工具，使用WinSCP，点击下载3.将本地数据库的数据迁移到服务器的数据库，使...](https://blog.csdn.net/qq_31772441/article/details/80637350 "【入门篇】篇四、将本地Web项目部署到服务器，迁移本地数据库到服务器")博文[来自： Markix的博客](https://blog.csdn.net/qq_31772441)

 [#### 怎样把_数据库_导入到_服务器_上

阅读数 2675](https://blog.csdn.net/iheyu/article/details/82784804 "怎样把数据库导入到服务器上") 

[1.打开phpstudy，点击“其他选项菜单”，点击phpMyAdmin 2.打开phpstorm，点击tiku，点击application，选择database.php文件，查看数据库名，如图： 3...](https://blog.csdn.net/iheyu/article/details/82784804 "怎样把数据库导入到服务器上")博文[来自： iheyu(一个新手小白女PHP初学者)](https://blog.csdn.net/iheyu)

 [#### 教新手如何把本地的msSQL_数据库_上_传到__服务器_

阅读数 1万+](https://blog.csdn.net/lishimin1012/article/details/16361913 "教新手如何把本地的msSQL数据库上传到服务器") 

[教新手如何把本地的msSQL数据库上传到服务器如何正确无误地把本地的msSQL数据库上传到服务器这2天帮2个客户上传MS SQL数据库到服务器，使用企业管理器导入数据，顺利地把表和数据导入到远程的SQ...](https://blog.csdn.net/lishimin1012/article/details/16361913 "教新手如何把本地的msSQL数据库上传到服务器")博文[来自： 资料汇总整理-仅供个人学习使用](https://blog.csdn.net/lishimin1012)

 [#### _使用__Navicat_ for _MySql_ 连接云_服务器_

阅读数 223](https://blog.csdn.net/qq1719448063/article/details/84140882 "使用Navicat for MySql 连接云服务器") 

[以下为连接步骤:1:配置ssh:注意:此处账号和密码填写服务器登录时的账号和密码,如果使用密匙登录,直接使用密匙即可2.配置完后连接:注意:此处账号和密码填写数据库的账户和密码.3.配置完成!......](https://blog.csdn.net/qq1719448063/article/details/84140882 "使用Navicat for MySql 连接云服务器")博文[来自： 子明的小博客](https://blog.csdn.net/qq1719448063)

 [#### 史上最详细的IDEA优雅整合Maven+SSM框架（详细思路+附带源码）

阅读数 1万+](https://blog.csdn.net/qq_44543508/article/details/100192558 "史上最详细的IDEA优雅整合Maven+SSM框架（详细思路+附带源码）") 

[网上很多整合SSM博客文章并不能让初探ssm的同学思路完全的清晰，可以试着关掉整合教程，摇两下头骨，哈一大口气，就在万事具备的时候，开整，这个时候你可能思路全无~中招了咩~，还有一些同学依旧在使用ec...](https://blog.csdn.net/qq_44543508/article/details/100192558 "史上最详细的IDEA优雅整合Maven+SSM框架（详细思路+附带源码）")博文[来自： 程序员宜春的博客](https://blog.csdn.net/qq_44543508)

 [#### 从入门到精通，Java学习路线导航

阅读数 5万+](https://blog.csdn.net/qq_42453117/article/details/100655512 "从入门到精通，Java学习路线导航") 

[引言最近也有很多人来向我"请教"，他们大都是一些刚入门的新手，还不了解这个行业，也不知道从何学起，开始的时候非常迷茫，实在是每天回复很多人也很麻烦，所以在这里统一作个回复吧。Java学习路线当然，这里...](https://blog.csdn.net/qq_42453117/article/details/100655512 "从入门到精通，Java学习路线导航")博文[来自： wangweijun](https://blog.csdn.net/qq_42453117)

 [#### _Navicat_ for _MySQL_如何实现_MYSQL_数据传输

阅读数 1226](https://blog.csdn.net/chenweihua556/article/details/80378562 "Navicat for MySQL如何实现MYSQL数据传输") 

[打开NavicatforMySQL，打开数据连接。打开数据连接后，我们会看到里面的数据库。打开织梦数据库dedecmsv57gbksp1，打开dede_admin表，我们会看到用户名和密码，大家在此处...](https://blog.csdn.net/chenweihua556/article/details/80378562 "Navicat for MySQL如何实现MYSQL数据传输")博文[来自： chenweihua556的专栏](https://blog.csdn.net/chenweihua556)

 [#### 阿里资深工程师教你如何优化 Java 代码！

阅读数 2万+](https://blog.csdn.net/csdnnews/article/details/100987866 "阿里资深工程师教你如何优化 Java 代码！") 

[作者|王超责编|伍杏玲明代王阳明先生在《传习录》谈为学之道时说：私欲日生，如地上尘，一日不扫，便又有一层。着实用功，便见道无终穷，愈探愈深，必使精白无一毫不彻方可。代码中的"坏味道"，如"私欲"如"灰...](https://blog.csdn.net/csdnnews/article/details/100987866 "阿里资深工程师教你如何优化 Java 代码！")博文[来自： CSDN资讯](https://blog.csdn.net/csdnnews)

 [#### 别再翻了，面试二叉树看这 11 个就够了~

阅读数 5万+](https://blog.csdn.net/qq_36903042/article/details/100798101 "别再翻了，面试二叉树看这 11 个就够了~") 

[写在前边数据结构与算法：不知道你有没有这种困惑，虽然刷了很多算法题，当我去面试的时候，面试官让你手写一个算法，可能你对此算法很熟悉，知道实现思路，但是总是不知道该在什么地方写，而且很多边界条件想不全面...](https://blog.csdn.net/qq_36903042/article/details/100798101 "别再翻了，面试二叉树看这 11 个就够了~")博文[来自： 一个不甘平凡的码农](https://blog.csdn.net/qq_36903042)

 [#### _Navicat_ For _MySQL_本地_数据库_同步结构和数据到_服务器__MySQL_

阅读数 161](https://blog.csdn.net/qq_24484085/article/details/88919845 "Navicat For MySQL本地数据库同步结构和数据到服务器MySQL") 

[一、需求场景和同学一块开发一个小APP，前端使用AppCan开发，后端使用SpringBoot，我主要负责后端接口的开发，因为异地吧，两个人都是男的，开发异地0..0。然后我俩都不一定什么时候有时间去...](https://blog.csdn.net/qq_24484085/article/details/88919845 "Navicat For MySQL本地数据库同步结构和数据到服务器MySQL")博文[来自： 小菜鸟的HelloWorld](https://blog.csdn.net/qq_24484085)

 [#### 将文件上_传到__数据库_ 和 从_数据库_下载文件到本地

阅读数 9780](https://blog.csdn.net/qq_33855133/article/details/73287238 "将文件上传到数据库 和 从数据库下载文件到本地") 

[有时候我们需要把图片、文档、dll文件、等等，上传的数据库，然后当需要的时候再从数据库中读取到本地，下面我以上传图片为例，讲解一下如何把本地的一张图片上传到数据库，然后再从数据库下载到本地。　　工具：...](https://blog.csdn.net/qq_33855133/article/details/73287238 "将文件上传到数据库 和 从数据库下载文件到本地")博文[来自： JohnApostle的博客](https://blog.csdn.net/qq_33855133)

 [#### GitHub开源的10个超棒后台管理面板

阅读数 4万+](https://blog.csdn.net/m0_38106923/article/details/101050788 "GitHub开源的10个超棒后台管理面板") 

[目录1、AdminLTE2、vue-Element-Admin3、tabler4、Gentelella5、ng2-admin6、ant-design-pro7、blur-admin8、iview-ad...](https://blog.csdn.net/m0_38106923/article/details/101050788 "GitHub开源的10个超棒后台管理面板")博文[来自： 不脱发的程序猿](https://blog.csdn.net/m0_38106923)

 [#### 我花了一夜用数据结构给女朋友写个H5走迷宫游戏

阅读数 12万+](https://blog.csdn.net/qq_40693171/article/details/100716766 "我花了一夜用数据结构给女朋友写个H5走迷宫游戏") 

[起因又到深夜了，我按照以往在csdn和公众号写着数据结构！这占用了我大量的时间！我的超越妹妹严重缺乏陪伴而怨气满满！而女朋友时常埋怨，认为数据结构这么抽象难懂的东西没啥作用，常会问道：天天写这玩意，有...](https://blog.csdn.net/qq_40693171/article/details/100716766 "我花了一夜用数据结构给女朋友写个H5走迷宫游戏")博文[来自： bigsai](https://blog.csdn.net/qq_40693171)

 [#### _MySQL__服务器_的安装与配置

阅读数 5245](https://blog.csdn.net/DFF1993/article/details/79492901 "MySQL服务器的安装与配置") 

[1、在浏览器的地址栏中输入地址：&quot;http://dev.mysql.com/downloads/&quot;，进入到MySQL官网的下载页面；    2、单击&quot;MySQLCommu...](https://blog.csdn.net/DFF1993/article/details/79492901 "MySQL服务器的安装与配置")博文[来自： 杜小白的博客](https://blog.csdn.net/DFF1993)

 [#### Windows下_使用__Navicat_同步连接_服务器_端_MySQL__数据库_

阅读数 3823](https://blog.csdn.net/godot06/article/details/81022682 "Windows下使用Navicat同步连接服务器端MySQL数据库") 

[在项目开发的过程中，我们通常会使用本地数据库测试，测试成功之后再通过数据传输的方式同步到服务器数据库，当然也有一些开发者直接同步服务器的数据库在本地电脑进行新建、修改、测试等操作。那么怎么用Windo...](https://blog.csdn.net/godot06/article/details/81022682 "Windows下使用Navicat同步连接服务器端MySQL数据库")博文[来自： godot06的博客](https://blog.csdn.net/godot06)

 [#### 让程序员崩溃的瞬间（非程序员勿入）

阅读数 17万+](https://blog.csdn.net/ybhuangfugui/article/details/100913641 "让程序员崩溃的瞬间（非程序员勿入）") 

[今天给大家带来点快乐，程序员才能看懂。来源：https://zhuanlan.zhihu.com/p/470665211.公司实习生找Bug2.在调试时，将断点设置在错误的位置3.当我有一个很棒的调试...](https://blog.csdn.net/ybhuangfugui/article/details/100913641 "让程序员崩溃的瞬间（非程序员勿入）")博文[来自： strongerHuang](https://blog.csdn.net/ybhuangfugui)

 [#### 技术一旦被用来作恶，究竟会有多可怕？

阅读数 1万+](https://blog.csdn.net/qq_43380549/article/details/101346556 "技术一旦被用来作恶，究竟会有多可怕？") 

[技术一直都在被用来作恶。作为与经常与黑客、攻击者打交道的我们，熟知各种用技术作恶的手段。这篇就作为简单的科普文来跟大家讲一讲。作恶之一：DDoS攻击用简单的一句话介绍DDoS攻击就是：黑客在短时间里发...](https://blog.csdn.net/qq_43380549/article/details/101346556 "技术一旦被用来作恶，究竟会有多可怕？")博文[来自： 知道创宇KCSC](https://blog.csdn.net/qq_43380549)

 [#### 学会了这些技术，你离BAT大厂不远了

阅读数 11万+](https://blog.csdn.net/z694644032/article/details/100084287 "学会了这些技术，你离BAT大厂不远了") 

[每一个程序员都有一个梦想，梦想着能够进入阿里、腾讯、字节跳动、百度等一线互联网公司，由于身边的环境等原因，不知道BAT等一线互联网公司使用哪些技术？或者该如何去学习这些技术？或者我该去哪些获取这些技术...](https://blog.csdn.net/z694644032/article/details/100084287 "学会了这些技术，你离BAT大厂不远了")博文[来自： 平头哥的技术博文](https://blog.csdn.net/z694644032)

 [#### 分享靠写代码赚钱的一些门路

阅读数 4万+](https://blog.csdn.net/lantian_123/article/details/101488841 "分享靠写代码赚钱的一些门路") 

[作者mezod，译者josephchang10如今，通过自己的代码去赚钱变得越来越简单，不过对很多人来说依然还是很难，因为他们不知道有哪些门路。今天给大家分享一个精彩......](https://blog.csdn.net/lantian_123/article/details/101488841 "分享靠写代码赚钱的一些门路")博文[来自： Python之禅的专栏](https://blog.csdn.net/lantian_123)

 [#### 如何_使用__navicat_等可视化工具连接到_服务器_上的_数据库_？

阅读数 3163](https://blog.csdn.net/qq_32958797/article/details/77870729 "如何使用navicat等可视化工具连接到服务器上的数据库？") 

[博主服务器是申请的腾讯云服务器，配置了SSL，但是远程连接mysql缺浪费蛮长时间的。其实无法连接到远程数据库就我目前为了解决所搜索到的原因和方法无非就那么几个，下面我根据错误提示来分析可能的原因和解...](https://blog.csdn.net/qq_32958797/article/details/77870729 "如何使用navicat等可视化工具连接到服务器上的数据库？")博文[来自： 蛋叔](https://blog.csdn.net/qq_32958797)

 [#### 德国 IT 薪酬大揭秘！

阅读数 5506](https://blog.csdn.net/csdnnews/article/details/102383573 "德国 IT 薪酬大揭秘！") 

[作者|德国IT那些事责编|伍杏玲“所有脱离工龄、级别、职位、经验、城市以及裙带关系来谈论工资，都是耍流氓！”——佛洛依德一般来说IT行业公司，资历是按等级划分的......](https://blog.csdn.net/csdnnews/article/details/102383573 "德国 IT 薪酬大揭秘！")博文[来自： CSDN资讯](https://blog.csdn.net/csdnnews)

 [#### JSP：上传图片到_数据库_并从_数据库_调用图片

阅读数 6028](https://blog.csdn.net/qq_42192693/article/details/81325436 "JSP：上传图片到数据库并从数据库调用图片") 

[实现将图片放入数据库，并从数据库调用图片准备：tomcat-9.0.01   jdk9  eclipse-ide  mysql8.0   mysql-connector-java-8.0.11.jar...](https://blog.csdn.net/qq_42192693/article/details/81325436 "JSP：上传图片到数据库并从数据库调用图片")博文[来自： 燕双嘤](https://blog.csdn.net/qq_42192693)

 [#### 世界上最好的学习法：费曼学习法

阅读数 4万+](https://blog.csdn.net/wo541075754/article/details/101554326 "世界上最好的学习法：费曼学习法") 

[你是否曾幻想读一遍书就记住所有的内容？是否想学习完一项技能就马上达到巅峰水平？除非你是天才，不然这是不可能的。对于大多数的普通人来说，可以通过笨办法（死记硬背）来达到学习的目的，但效率低下。当然，也可...](https://blog.csdn.net/wo541075754/article/details/101554326 "世界上最好的学习法：费曼学习法")博文[来自： 程序新视界](https://blog.csdn.net/wo541075754)

 [#### 据说中台凉了？唔，真香

阅读数 1万+](https://blog.csdn.net/u010459192/article/details/102548299 "据说中台凉了？唔，真香") 

[全文长度:2200字阅读时间:8分钟TL;DR(toolongdon'tread)1、业务中台就是流程模板+扩展点2、没法很好抽象就别做中台，没那么多需求和业务线就别做中台。很多同学都会问，啥叫中台，...](https://blog.csdn.net/u010459192/article/details/102548299 "据说中台凉了？唔，真香")博文[来自： u010459192的博客](https://blog.csdn.net/u010459192)

 [#### 500行代码，教你用python写个微信飞机大战

阅读数 4万+](https://blog.csdn.net/u012365828/article/details/102559913 "500行代码，教你用python写个微信飞机大战") 

[这几天在重温微信小游戏的飞机大战，玩着玩着就在思考人生了，这飞机大战怎么就可以做的那么好，操作简单，简单上手。帮助蹲厕族、YP族、饭圈女孩在无聊之余可以有一样东西让他们振作起来！让他们的左手/右手有节...](https://blog.csdn.net/u012365828/article/details/102559913 "500行代码，教你用python写个微信飞机大战")博文[来自： Python专栏](https://blog.csdn.net/u012365828)

 [#### 面试官，不要再问我三次握手和四次挥手

阅读数 12万+](https://blog.csdn.net/hyg0811/article/details/102366854 "面试官，不要再问我三次握手和四次挥手") 

[三次握手和四次挥手是各个公司常见的考点，也具有一定的水平区分度，也被一些面试官作为热身题。很多小伙伴说这个问题刚开始回答的挺好，但是后面越回答越冒冷汗，最后就歇菜了。见过比较典型的面试场景是这样的:面...](https://blog.csdn.net/hyg0811/article/details/102366854 "面试官，不要再问我三次握手和四次挥手")博文[来自： 猿人谷](https://blog.csdn.net/hyg0811)

 [#### _使用__Navicat_连接阿里云_服务器_上的_MySQL__数据库_

阅读数 2万+](https://blog.csdn.net/liuhailiuhai12/article/details/64124637 "使用Navicat连接阿里云服务器上的MySQL数据库") 

[1.首先打开Navicat，文件>新建连接>MySQL连接，其他的如一图所示。2.因为是连接服务器上的MySQL，所以我们使用SSH连接，操作如二图所示。3.最后连接测试，连接成功。...](https://blog.csdn.net/liuhailiuhai12/article/details/64124637 "使用Navicat连接阿里云服务器上的MySQL数据库")博文[来自： liuhai的博客](https://blog.csdn.net/liuhailiuhai12)

 [#### Docker技术( 容器虚拟化技术 )

阅读数 1万+](https://blog.csdn.net/qq_43371556/article/details/102631158 "Docker技术( 容器虚拟化技术 )") 

[Docker虚拟化容器技术第一章Docker简介诞生背景Docker介绍虚拟机技术容器虚拟化技术官方网址第二章Docker安装前提条件安装DockerDocker底层原理Docker结构图工作原理Do...](https://blog.csdn.net/qq_43371556/article/details/102631158 "Docker技术( 容器虚拟化技术 )")博文[来自： 时间静止](https://blog.csdn.net/qq_43371556)

 [#### _使用__navicat_ premium 将本地sqlserver_数据库_传输到远程_服务器__数据库_

阅读数 272](https://blog.csdn.net/haibo211314/article/details/84476830 "使用navicat premium 将本地sqlserver数据库传输到远程服务器数据库") 

[ 第一步.使用navicatpremium（V12.1.0）连接本地数据库和远程数据库 第二步.选择工具菜单中的数据传输，打开下图数据传输窗口，填写必要的远程sqlserver数据库服务器信息第...](https://blog.csdn.net/haibo211314/article/details/84476830 "使用navicat premium 将本地sqlserver数据库传输到远程服务器数据库")博文[来自： haibo211314的专栏](https://blog.csdn.net/haibo211314)

 [#### C语言实现推箱子游戏

阅读数 7万+](https://blog.csdn.net/ZackSock/article/details/101645494 "C语言实现推箱子游戏") 

[很早就想过做点小游戏了，但是一直没有机会动手。今天闲来无事，动起手来。过程还是蛮顺利的，代码也不是非常难。今天给大家分享一下~一、介绍开发语言：C语言开发工具：Dev-C++5.11日期：2019年9...](https://blog.csdn.net/ZackSock/article/details/101645494 "C语言实现推箱子游戏")博文[来自： ZackSock的博客](https://blog.csdn.net/ZackSock)

 [#### _Navicat_连接到_服务器_端_数据库_

阅读数 2万+](https://blog.csdn.net/javakklam/article/details/80060866 "Navicat连接到服务器端数据库") 

[Navicatformysql连接远程数据库（已经开启SSH带秘钥的服务器端数据库）本来没有开启秘钥的远程服务器端数据库连接非常方便，就在新建连接上填入数据就ok了，但是开启SSH秘钥后的服务器连接有...](https://blog.csdn.net/javakklam/article/details/80060866 "Navicat连接到服务器端数据库")博文[来自： SpiderFlame的博客](https://blog.csdn.net/javakklam)

 [#### java秀发入门到优雅秃头路线导航【教学视频+博客+书籍整理】

阅读数 4436](https://blog.csdn.net/qq_44543508/article/details/102651841 "java秀发入门到优雅秃头路线导航【教学视频+博客+书籍整理】") 

[在博主认为，学习java的最佳学习方法莫过于视频+博客+书籍+总结，前三者博主将淋漓尽致地挥毫于这篇博客文章中，至于总结在于个人，博主将为各位保驾护航，各位赶紧冲鸭！！！上天是公平的，只要不辜负时间，...](https://blog.csdn.net/qq_44543508/article/details/102651841 "java秀发入门到优雅秃头路线导航【教学视频+博客+书籍整理】")博文[来自： 程序员宜春的博客](https://blog.csdn.net/qq_44543508)

 [#### 史上最全的中高级JAVA工程师-面试题汇总

阅读数 3万+](https://blog.csdn.net/shengqianfeng/article/details/102572691 "史上最全的中高级JAVA工程师-面试题汇总") 

[史上最全的java工程师面试题汇总，纯个人总结，精准无误。适合中高级JAVA工程师。...](https://blog.csdn.net/shengqianfeng/article/details/102572691 "史上最全的中高级JAVA工程师-面试题汇总")博文[来自： 在广？No，在深? em,或许在专......](https://blog.csdn.net/shengqianfeng)

 [#### 【吐血整理】那些让你起飞的计算机基础知识：学什么，怎么学？

阅读数 334](https://blog.csdn.net/lc013/article/details/102548333 "【吐血整理】那些让你起飞的计算机基础知识：学什么，怎么学？") 

[作者：帅地来源公众号：苦逼的码农我公众号里的文章，写的大部分都是与计算机基础知识相关的，这些基础知识，就像我们的内功，如果在未来想要走的更远，这些内功是必须要修炼的。框架......](https://blog.csdn.net/lc013/article/details/102548333 "【吐血整理】那些让你起飞的计算机基础知识：学什么，怎么学？")博文[来自： 算法猿的成长](https://blog.csdn.net/lc013)

 [#### 不就是SELECT COUNT语句吗，竟然能被面试官虐的体无完肤

阅读数 2万+](https://blog.csdn.net/hollis_chuang/article/details/102657937 "不就是SELECT COUNT语句吗，竟然能被面试官虐的体无完肤") 

[数据库查询相信很多人都不陌生，所有经常有人调侃程序员就是CRUD专员，这所谓的CRUD指的就是数据库的增删改查。在数据库的增删改查操作中，使用最频繁的就是查询操作。而在所有查询操作中，统计数量操作更是...](https://blog.csdn.net/hollis_chuang/article/details/102657937 "不就是SELECT COUNT语句吗，竟然能被面试官虐的体无完肤")博文[来自： HollisChuang's Blog](https://blog.csdn.net/hollis_chuang)

 [#### 面试官：兄弟，说说基本类型和包装类型的区别吧

阅读数 2万+](https://blog.csdn.net/qing_gee/article/details/101670051 "面试官：兄弟，说说基本类型和包装类型的区别吧") 

[Java的每个基本类型都对应了一个包装类型，比如说int的包装类型为Integer，double的包装类型为Double。基本类型和包装类型的区别主要有以下4点。...](https://blog.csdn.net/qing_gee/article/details/101670051 "面试官：兄弟，说说基本类型和包装类型的区别吧")博文[来自： 沉默王二](https://blog.csdn.net/qing_gee)

 [#### 程序员实用工具网站

阅读数 16万+](https://blog.csdn.net/m0_38106923/article/details/100130354 "程序员实用工具网站") 

[目录 1、搜索引擎 2、PPT 3、图片操作 4、文件共享 5、应届生招聘 6、程序员面试题库 7、办公、开发软件 8、高清图片、视频素材网站 9、项目开源 10、在线工具宝典大全...](https://blog.csdn.net/m0_38106923/article/details/100130354 "程序员实用工具网站")博文

 [#### 程序员真是太太太太太有趣了！！！

阅读数 3万+](https://blog.csdn.net/j3T9Z7H/article/details/100179186 "程序员真是太太太太太有趣了！！！") 

[网络上虽然已经有了很多关于程序员的话题，但大部分人对这个群体还是很陌生。我们在谈论程序员的时候，究竟该聊些什么呢？各位程序员大佬们，请让我听到你们的声音！不管你是前端开发......](https://blog.csdn.net/j3T9Z7H/article/details/100179186 "程序员真是太太太太太有趣了！！！")博文

 [#### 我的 Input框 不可能这么可爱

阅读数 5497](https://blog.csdn.net/weixin_37615279/article/details/100516311 "我的 Input框 不可能这么可爱") 

[作者：陈大鱼头 github： KRISACHAN &lt;input /&gt; 标签是我们日常开发中非常常见的替换元素了，但是最近在刷 whattwg 跟 MDN 的时候发现 跟 &lt;in...](https://blog.csdn.net/weixin_37615279/article/details/100516311 "我的 Input框 不可能这么可爱")博文

 [#### 一声令下即可关灯，再也不用在寒冬深夜离开被窝~

阅读数 1248](https://blog.csdn.net/duxinshuxiaobian/article/details/100531797 "一声令下即可关灯，再也不用在寒冬深夜离开被窝~") 

[全文共2951字，预计学习时长6分钟 你有没有在大冬天里的深夜里，为了关灯睡觉而不得不离开温暖被窝的经历？ 本文将介绍如何为普通家庭照明开关构建自然语言接口，以便用户可以使用如“请打开所有灯”...](https://blog.csdn.net/duxinshuxiaobian/article/details/100531797 "一声令下即可关灯，再也不用在寒冬深夜离开被窝~")博文

 [#### 知乎上 40 个有趣回复，很精辟很提神

阅读数 4万+](https://blog.csdn.net/kexuanxiu1163/article/details/100613498 "知乎上 40 个有趣回复，很精辟很提神") 

[点击蓝色“五分钟学算法”关注我哟加个“星标”，天天中午 12:15，一起学算法作者 |佚名来源 |网络整理，版权归原作者所有，侵删。1交朋友的标准是什么？- Ques......](https://blog.csdn.net/kexuanxiu1163/article/details/100613498 "知乎上 40 个有趣回复，很精辟很提神")博文

 [#### 反转！物联网火爆，程序员开发技能却有待加强？

阅读数 1226](https://blog.csdn.net/csdnnews/article/details/100763959 "反转！物联网火爆，程序员开发技能却有待加强？") 

[近几年来，物联网发展迅速：据中商产业研究院《2016——2021年中国物联网产业市场研究报告》显示，预计到2020年，中国物联网的整体规模将达2.2万亿元，产业规模比互联......](https://blog.csdn.net/csdnnews/article/details/100763959 "反转！物联网火爆，程序员开发技能却有待加强？")博文

 [#### 推荐一位好朋友 | 机械转大数据，并拿到66个offer

阅读数 645](https://blog.csdn.net/u010459192/article/details/100912319 "推荐一位好朋友 | 机械转大数据，并拿到66个offer") 

[锦锋是我的好朋友，他自学编程从车辆工程转到了Java开发然后转大数据，在校招中拿了大小厂66个offer，其中有头条、腾讯等。除了学习之外，他还是国家高级健身教练，同时也......](https://blog.csdn.net/u010459192/article/details/100912319 "推荐一位好朋友 | 机械转大数据，并拿到66个offer")博文

 [#### 接私活必备的 10 个开源项目！

阅读数 4万+](https://blog.csdn.net/sinat_33224091/article/details/100980160 "接私活必备的 10 个开源项目！") 

[点击蓝色“GitHubDaily”关注我加个“星标”，每天下午 18:35，带你逛 GitHub！作者 | SevDot来源 | http://1t.click/VE8W......](https://blog.csdn.net/sinat_33224091/article/details/100980160 "接私活必备的 10 个开源项目！")博文

 [#### 不要在网站上无限滚动！

阅读数 1万+](https://blog.csdn.net/csdnnews/article/details/100987794 "不要在网站上无限滚动！") 

[人们在浏览网站的时候是喜欢用“无限滚动”，还是喜欢点击“阅读更多”或“查看更多”?无限滚动消除了分页的需要——分页是将数字内容分离到不同页面的过程。但这种方式真的好吗？ 作者|Monish re...](https://blog.csdn.net/csdnnews/article/details/100987794 "不要在网站上无限滚动！")博文

 [#### 为什么你的努力可能是没用的？

阅读数 1233](https://blog.csdn.net/wo541075754/article/details/101083036 "为什么你的努力可能是没用的？") 

[看到这个标题，不少朋友可能直观的以为后面的内容不是励志的鸡汤就是广告。那就错了，这篇文章只是满满的干货。内容来自最近一段时间探索学习外加亲身感悟汇集而成。 前段时间和一位朋友聊天，聊到如何通过自由职业...](https://blog.csdn.net/wo541075754/article/details/101083036 "为什么你的努力可能是没用的？")博文

 [#### 100 个网络基础知识普及，看完成半个网络高手

阅读数 11万+](https://blog.csdn.net/devcloud/article/details/101199255 "100 个网络基础知识普及，看完成半个网络高手") 

[1）什么是链接？ 链接是指两个设备之间的连接。它包括用于一个设备能够与另一个设备通信的电缆类型和协议。 2）OSI 参考模型的层次是什么？ 有 7 个 OSI 层：物理层，数据链路层，网络层，传...](https://blog.csdn.net/devcloud/article/details/101199255 "100 个网络基础知识普及，看完成半个网络高手")博文

 [#### 学Linux到底学什么

阅读数 2万+](https://blog.csdn.net/hyb612/article/details/101561520 "学Linux到底学什么") 

[来源：公众号【编程珠玑】 作者：守望先生 网站：https://www.yanbinghu.com/2019/09/25/14472.html 前言​我们常常听到很多人说要学学Linux或者...](https://blog.csdn.net/hyb612/article/details/101561520 "学Linux到底学什么")博文

 [#### C语言这么厉害，它自身又是用什么语言写的？

阅读数 3万+](https://blog.csdn.net/coderising/article/details/101731213 "C语言这么厉害，它自身又是用什么语言写的？") 

[这是来自我的星球的一个提问：“C语言本身用什么语言写的？”换个角度来问，其实是：C语言在运行之前，得编译才行，那C语言的编译器从哪里来？ 用什么语言来写的？如果是用C语......](https://blog.csdn.net/coderising/article/details/101731213 "C语言这么厉害，它自身又是用什么语言写的？")博文

 [#### win10电脑工具整理 - 常用工具！

阅读数 1893](https://blog.csdn.net/weixin_41599858/article/details/102084565 "win10电脑工具整理 - 常用工具！") 

[如题，本文主要为博主对电脑上安装的一些软件，所做的整理，当做备份用吧。 一、分类 系统工具 办公软件 编程开发 数据库相关 图片视频工具 网络及下载工具 解压缩工具 影音娱乐工具 二、软件工具 ...](https://blog.csdn.net/weixin_41599858/article/details/102084565 "win10电脑工具整理 - 常用工具！")博文

 [#### 第二弹！python爬虫批量下载高清大图

阅读数 2万+](https://blog.csdn.net/qq_40693171/article/details/102220448 "第二弹！python爬虫批量下载高清大图") 

[文章目录前言下载免费高清大图下载带水印的精选图代码与总结 前言 在上一篇写文章没高质量配图？python爬虫绕过限制一键搜索下载图虫创意图片！中，我们在未登录的情况下实现了图虫创意无水印高清小图的...](https://blog.csdn.net/qq_40693171/article/details/102220448 "第二弹！python爬虫批量下载高清大图")博文

 [#### 为什么说 Web 开发永远不会退出历史舞台？

阅读数 6575](https://blog.csdn.net/csdnnews/article/details/102425414 "为什么说 Web 开发永远不会退出历史舞台？") 

[早在 PC 崛起之际，Web 从蹒跚学步一路走到了主导市场的地位，但是随着移动互联网时代的来临，业界曾有不少人猜测，“Web 应该被杀死，App 才是未来”。不过时间是检......](https://blog.csdn.net/csdnnews/article/details/102425414 "为什么说 Web 开发永远不会退出历史舞台？")博文

 [#### 我所经历的三次裁员

阅读数 1万+](https://blog.csdn.net/mogoweb/article/details/102493424 "我所经历的三次裁员") 

[先从一则新闻说起：人民网旧金山9月19日电(邓圩 宫欣)当地时间9月19日，位于旧金山湾区Menlo Park的Facebook总部内，一名男子从园区内的一栋办公楼4楼跳......](https://blog.csdn.net/mogoweb/article/details/102493424 "我所经历的三次裁员")博文

 [#### div+css实现水平/垂直/水平垂直居中详解

阅读数 726](https://blog.csdn.net/qq_42532128/article/details/102526334 "div+css实现水平/垂直/水平垂直居中详解") 

[单个元素 水平居中 1.margin:0 auto方法 wrapper相对屏幕居中 &lt;div class="wrapper"&gt;&lt;/div&gt; body{ width: 100...](https://blog.csdn.net/qq_42532128/article/details/102526334 "div+css实现水平/垂直/水平垂直居中详解")博文

 [#### 为什么程序员在学习编程的时候什么都记不住？

阅读数 1万+](https://blog.csdn.net/csdnnews/article/details/102548475 "为什么程序员在学习编程的时候什么都记不住？") 

[在程序员的职业生涯中，记住所有你接触过的代码是一件不可能的事情！那么我们该如何解决这一问题？作者 |Dylan Mestyanek译者 | 弯月，责编 | 屠敏出品 |......](https://blog.csdn.net/csdnnews/article/details/102548475 "为什么程序员在学习编程的时候什么都记不住？")博文

 [#### 唐僧团队要裁员，你会裁谁？

阅读数 3万+](https://blog.csdn.net/wangxueming/article/details/102561475 "唐僧团队要裁员，你会裁谁？") 

[提问： 西游记取经团为了节约成本，唐太宗需要在这个团队里裁掉一名队员，该裁掉哪一位呢，为什么? 为了完成西天取经任务，组成取经团队，成员有唐僧、孙悟空、猪八戒、沙和尚、白龙马。 高层领导： 观音...](https://blog.csdn.net/wangxueming/article/details/102561475 "唐僧团队要裁员，你会裁谁？")博文

 [#### 2019诺贝尔经济学奖得主：贫穷的本质是什么？

阅读数 2642](https://blog.csdn.net/zhongyangzhong/article/details/102578044 "2019诺贝尔经济学奖得主：贫穷的本质是什么？") 

[2019年诺贝尔经济学奖，颁给了来自麻省理工学院的 阿巴希·巴纳吉（Abhijit Vinayak Banerjee）、艾丝特·杜芙若（Esther Duflo）夫妇和哈......](https://blog.csdn.net/zhongyangzhong/article/details/102578044 "2019诺贝尔经济学奖得主：贫穷的本质是什么？")博文

 [#### IntelliJ IDEA 超实用使用技巧分享

阅读数 1万+](https://blog.csdn.net/weixin_38405253/article/details/102583954 "IntelliJ IDEA 超实用使用技巧分享") 

[前言 工欲善其事 必先利其器 最近受部门的邀请，给入职新人统一培训IDEA，发现有很多新人虽然日常开发使用的是IDEA，但是还是很多好用的技巧没有用到，只是用到一些基本的功能，蛮浪费IDEA这...](https://blog.csdn.net/weixin_38405253/article/details/102583954 "IntelliJ IDEA 超实用使用技巧分享")博文

 [#### linux：最常见的linux命令（centOS 7.6）

阅读数 3000](https://blog.csdn.net/Dakshesh/article/details/102588364 "linux：最常见的linux命令（centOS 7.6）") 

[最常见，最频繁使用的20个基础命令如下： 皮一下，这都是干货偶，大佬轻喷 一、linux关机命令： 1.shutdown命令安全地将系统关机（推荐）参数说明: [-r] 重启计算器。 [-h] 关机后...](https://blog.csdn.net/Dakshesh/article/details/102588364 "linux：最常见的linux命令（centOS 7.6）")博文

 [#### 只因写了一段爬虫，公司200多人被抓！

阅读数 9万+](https://blog.csdn.net/ityouknow/article/details/102597598 "只因写了一段爬虫，公司200多人被抓！") 

[“一个程序员写了个爬虫程序，整个公司200多人被端了。” “不可能吧！” 刚从朋友听到这个消息的时候，我有点不太相信，做为一名程序员来讲，谁还没有写过几段爬虫呢？只因写爬虫程序就被端有点夸张了吧。...](https://blog.csdn.net/ityouknow/article/details/102597598 "只因写了一段爬虫，公司200多人被抓！")博文

 [#### 三年一跳槽、拒绝“唯学历”，火速 Get 这份程序员求生指南！

阅读数 1万+](https://blog.csdn.net/csdnnews/article/details/102617768 "三年一跳槽、拒绝“唯学历”，火速 Get 这份程序员求生指南！") 

[根据埃文斯数据公司（Evans Data Corporation）2019 最新统计的数据显示，2018 年全球共有 2300 万软件开发人员，预计到 2019 年底这个数字将达到 2640 万。但在...](https://blog.csdn.net/csdnnews/article/details/102617768 "三年一跳槽、拒绝“唯学历”，火速 Get 这份程序员求生指南！")博文

 [#### Docker 大势已去，Podman 万岁

阅读数 2万+](https://blog.csdn.net/alex_yangchuansheng/article/details/102618128 "Docker 大势已去，Podman 万岁") 

[前言郑重声明：本文不是 Podman 的入门篇，入门请阅读这篇文章：再见 Docker，是时候拥抱下一代容器工具了Podman 原来是 CRI-O 项目的一部分，后来被分......](https://blog.csdn.net/alex_yangchuansheng/article/details/102618128 "Docker 大势已去，Podman 万岁")博文

 [#### 别在学习框架了，那些让你起飞的计算机基础知识。

阅读数 3万+](https://blog.csdn.net/m0_37907797/article/details/102618796 "别在学习框架了，那些让你起飞的计算机基础知识。") 

[我之前里的文章，写的大部分都是与计算机基础知识相关的，这些基础知识，就像我们的内功，如果在未来想要走的更远，这些内功是必须要修炼的。框架千变万化，而这些通用的底层知识，却是几乎不变的，了解了这些知识，...](https://blog.csdn.net/m0_37907797/article/details/102618796 "别在学习框架了，那些让你起飞的计算机基础知识。")博文

 [#### “来我公司写爬虫吗？会坐牢的那种！”

阅读数 1万+](https://blog.csdn.net/yellowzf3/article/details/102634078 "“来我公司写爬虫吗？会坐牢的那种！”") 

[欢迎关注“技术领导力”博客，每天早上8:30推送 “你交代一下，总共抓了多少数据，在哪些网站抓的，数据干什么用了？看看够在里面呆几年。。。”警察语气凝重地对张强说。 程序员张强（化名），回...](https://blog.csdn.net/yellowzf3/article/details/102634078 "“来我公司写爬虫吗？会坐牢的那种！”")博文

 [#### 几道经典逻辑推理题，提高你的逻辑思考能力

阅读数 1万+](https://blog.csdn.net/qq_23853743/article/details/102650927 "几道经典逻辑推理题，提高你的逻辑思考能力") 

[整理了一些逻辑推理题，这些逻辑推理题能够提高大家的逻辑思考能力，同时也能给大家的学习带来一定的趣味性。希望大家看到题之后，不要着急看答案，要先独立思考解决。答案的获取可以关注我的公众号：[Albert...](https://blog.csdn.net/qq_23853743/article/details/102650927 "几道经典逻辑推理题，提高你的逻辑思考能力")博文

 [#### 五款高效率黑科技神器工具，炸裂好用，省时间

阅读数 1万+](https://blog.csdn.net/loongggdroid/article/details/102656177 "五款高效率黑科技神器工具，炸裂好用，省时间") 

[loonggg读完需要4分钟速读仅需2分钟感觉我好久好久没有给大家分享高质量的软件和插件了。今天周末，难得在家休息一下，痛下决心，分享一些我认为的高效率工具软件给大家。废......](https://blog.csdn.net/loongggdroid/article/details/102656177 "五款高效率黑科技神器工具，炸裂好用，省时间")博文

 [#### 动画：用动画给女朋友讲解 TCP 四次分手过程

阅读数 1万+](https://blog.csdn.net/qq_36903042/article/details/102656641 "动画：用动画给女朋友讲解 TCP 四次分手过程") 

[作者 | 小鹿 来源 | 公众号：小鹿动画学编程 写在前边 大家好，我们又见面了，做为一个业余的动画师，上次的用动画的形式讲解 TCP 三次握手过程再各大平台收到了广大读者的喜爱，说文章有趣、有...](https://blog.csdn.net/qq_36903042/article/details/102656641 "动画：用动画给女朋友讲解 TCP 四次分手过程")博文

 [#### 程序员必须掌握的核心算法有哪些？

阅读数 5万+](https://blog.csdn.net/m0_37907797/article/details/102661778 "程序员必须掌握的核心算法有哪些？") 

[由于我之前一直强调数据结构以及算法学习的重要性，所以就有一些读者经常问我，数据结构与算法应该要学习到哪个程度呢？，说实话，这个问题我不知道要怎么回答你，主要取决于你想学习到哪些程度，不过针对这个问题，...](https://blog.csdn.net/m0_37907797/article/details/102661778 "程序员必须掌握的核心算法有哪些？")博文

 [#### 如何优化MySQL千万级大表，我写了6000字的解读

阅读数 2万+](https://blog.csdn.net/yangjianrong1985/article/details/102675334 "如何优化MySQL千万级大表，我写了6000字的解读") 

[这是学习笔记的第2138篇文章 千万级大表如何优化，这是一个很有技术含量的问题，通常我们的直觉思维都会跳转到拆分或者数据分区，在此我想做一些补充和梳理，想和大家做一些这方面的经验总结，也欢迎大家...](https://blog.csdn.net/yangjianrong1985/article/details/102675334 "如何优化MySQL千万级大表，我写了6000字的解读")博文

 [#### 面试最后一问：你有什么问题想问我吗？

阅读数 2万+](https://blog.csdn.net/dyc87112/article/details/102676581 "面试最后一问：你有什么问题想问我吗？") 

[尽管，我们之前分享了这么多关于面试的主题： 高薪必备的一些Spring Boot高级面试题 面试必问：设计模式遵循的面向对象设计原则！ 面试必问：怎么保证缓存与数据库的双写一致性？ 27道高频Spr...](https://blog.csdn.net/dyc87112/article/details/102676581 "面试最后一问：你有什么问题想问我吗？")博文

 [#### python 程序员进阶之路：从新手到高手的100个模块

阅读数 3万+](https://blog.csdn.net/xufive/article/details/102676755 "python 程序员进阶之路：从新手到高手的100个模块") 

[在知乎和CSDN的圈子里，经常看到、听到一些 python 初学者说，学完基础语法后，不知道该学什么，学了也不知道怎么用，一脸的茫然。近日，CSDN的公众号推送了一篇博客，题目叫做《迷思：Python...](https://blog.csdn.net/xufive/article/details/102676755 "python 程序员进阶之路：从新手到高手的100个模块")博文

 [#### 大学四年，看过的优质书籍推荐

阅读数 3万+](https://blog.csdn.net/m0_37907797/article/details/102685204 "大学四年，看过的优质书籍推荐") 

[有时有些读者问我，数据结构与算法该怎么学？有书籍推荐的吗？Java 初学者该怎么学等等。今天我就给大家介绍一些我这几年看过的一些自认为优秀的书籍，由于我看的大部分书籍可以说都是通用的，所以如果你有时间...](https://blog.csdn.net/m0_37907797/article/details/102685204 "大学四年，看过的优质书籍推荐")博文

 [#### Python——画一棵漂亮的樱花树（不同种樱花+玫瑰+圣诞树喔）

阅读数 1万+](https://blog.csdn.net/weixin_43943977/article/details/102691392 "Python——画一棵漂亮的樱花树（不同种樱花+玫瑰+圣诞树喔）") 

[最近翻到一篇知乎，上面有不少用Python（大多是turtle库）绘制的树图，感觉很漂亮，我整理了一下，挑了一些我觉得不错的代码分享给大家（这些我都测试过，确实可以生成） one 樱花树 动...](https://blog.csdn.net/weixin_43943977/article/details/102691392 "Python——画一棵漂亮的樱花树（不同种樱花+玫瑰+圣诞树喔）")博文

 [#### 还在收集资料？我这里有个github汇总

阅读数 1万+](https://blog.csdn.net/lycyingO/article/details/102693626 "还在收集资料？我这里有个github汇总") 

[原创：小姐姐味道（微信公众号ID：xjjdog），欢迎分享，转载请保留出处。国内程序员都喜欢收集资料，但是又不看，github是重灾区。更有莫名其妙fork的，让人不得要......](https://blog.csdn.net/lycyingO/article/details/102693626 "还在收集资料？我这里有个github汇总")博文

 [#### 程序员不懂浪漫？胡扯！

阅读数 1万+](https://blog.csdn.net/csdnnews/article/details/102693777 "程序员不懂浪漫？胡扯！") 

[程序员男朋友你的程序员男朋友为你做过什么暖心的事情呢？我的男朋友是一个程序员，他有很多大家在网络上吐槽的程序员的缺点，比如加班很多，没空陪我吃饭逛街看电影，比如说他有的时......](https://blog.csdn.net/csdnnews/article/details/102693777 "程序员不懂浪漫？胡扯！")博文

 [#### 程序员成长的四个简单技巧，你 get 了吗？

阅读数 1万+](https://blog.csdn.net/z694644032/article/details/102695164 "程序员成长的四个简单技巧，你 get 了吗？") 

[最近拜读了“阿里工程师的自我修养”手册，12 位技术专家分享生涯感悟来帮助我们这些菜鸡更好的成长，度过中年危机，我收获颇多，其中有不少的方法技巧和我正在使用的，这让我觉得我做的这些事情是对的，我走在了...](https://blog.csdn.net/z694644032/article/details/102695164 "程序员成长的四个简单技巧，你 get 了吗？")博文

[c#连接扫描枪](https://www.csdn.net/gather_15/NtzaMgxsLWRvd25sb2Fk.html) [c# 跟c++ 进程同步](https://www.csdn.net/gather_16/NtzaMgzsLWRvd25sb2Fk.html) [c#中的arrylist](https://www.csdn.net/gather_1e/NtzaMg0sLWRvd25sb2Fk.html) [c#子窗体重复打开](https://www.csdn.net/gather_16/NtzaMg1sLWRvd25sb2Fk.html) [c# 源码 网站监控](https://www.csdn.net/gather_16/NtzaMg2sLWRvd25sb2Fk.html) [python和c#的区别](https://www.csdn.net/gather_2a/NtzaMg3sLWJsb2cO0O0O.html) [c# 时间比天数](https://www.csdn.net/gather_12/NtzaMg4sLWRvd25sb2Fk.html) [c# oracle查询](https://www.csdn.net/gather_1d/NtzaMg5sLWRvd25sb2Fk.html) [c# 主动推送 事件](https://www.csdn.net/gather_1e/NtzaQgwsLWRvd25sb2Fk.html) [c# java 属性](https://www.csdn.net/gather_12/NtzaQgxsLWRvd25sb2Fk.html)

©️2019 CSDN 皮肤主题: 大白 设计师: CSDN官方博客

[![](https://profile.csdnimg.cn/D/A/9/3_kkfd1002)![](https://g.csdnimg.cn/static/user-reg-year/1x/1.png)](https://blog.csdn.net/kkfd1002)

[kkfd1002](https://blog.csdn.net/kkfd1002 "kkfd1002")

[TA的个人主页 >](https://me.csdn.net/kkfd1002)

[私信](https://blog.csdn.net/kkfd1002/article/details/80247882)

关注

原创

0

粉丝

26

获赞

20

评论

8

访问

10万+

等级:

[](https://blog.csdn.net/home/help.html#level "4级,点击查看等级说明")

周排名:

[105万+](https://blog.csdn.net/rank/writing_rank)

积分:

1059

总排名:

[7万+](https://blog.csdn.net/rank/writing_rank_total)

### 最新文章

*   [Android软键盘弹出，覆盖h5页面输入框问题](https://blog.csdn.net/kkfd1002/article/details/80401604)
*   [安装scrapy出错Failed building wheel for Twisted](https://blog.csdn.net/kkfd1002/article/details/80400216)
*   [五大经典算法之回溯法](https://blog.csdn.net/kkfd1002/article/details/80400214)
*   [codeforces 982D Shark](https://blog.csdn.net/kkfd1002/article/details/80379456)
*   [js算法初窥（排序算法-归并、快速以及堆排序）](https://blog.csdn.net/kkfd1002/article/details/80379453)

### 归档

*   [2018年5月48篇](https://blog.csdn.net/kkfd1002/article/month/2018/05)
*   [2018年4月50篇](https://blog.csdn.net/kkfd1002/article/month/2018/04)
*   [2018年3月8篇](https://blog.csdn.net/kkfd1002/article/month/2018/03)

### 最新评论

*   [使用Navicat for MyS...](https://blog.csdn.net/kkfd1002/article/details/80247882#comments)

    [weixin_45109662：](https://my.csdn.net/weixin_45109662)膜拜大佬

*   [nodejs和vue的那些事](https://blog.csdn.net/kkfd1002/article/details/80013168#comments)

    [Mr_carry：](https://my.csdn.net/Mr_carry)这看完还不得断气

*   [Python3 urllib.re...](https://blog.csdn.net/kkfd1002/article/details/80306398#comments)

    [qq_33161208：](https://my.csdn.net/qq_33161208)例子3没有加上headers信息，应该是response = urllib.request.urlopen(request,headers=headers)

*   [使用Navicat for MyS...](https://blog.csdn.net/kkfd1002/article/details/80247882#comments)

    [ZHL8488：](https://my.csdn.net/ZHL8488)非常有用，谢谢

*   [nodejs和vue的那些事](https://blog.csdn.net/kkfd1002/article/details/80013168#comments)

    [qq_36762677：](https://my.csdn.net/qq_36762677)看这太难受了