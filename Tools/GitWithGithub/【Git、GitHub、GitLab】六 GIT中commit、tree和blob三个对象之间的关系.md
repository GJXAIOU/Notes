# 【Git、GitHub、GitLab】六 GIT中commit、tree和blob三个对象之间的关系

2019年01月21日 23:22:03 [杨柳_](https://me.csdn.net/qq_37375427) 阅读数：748更多

所属专栏： [Git、GitHub、GitLab学习专栏](https://blog.csdn.net/column/details/19366.html)

版权声明：本文为博主原创文章，未经博主允许不得转载，转载请加博主qq：1126137994或者微信：liu1126137994 https://lyy-0217.blog.csdn.net/article/details/86586584

> 上一篇文章学习了git裸仓库.git中的内容，点击查看上一篇文章：[【Git、GitHub、GitLab】五 git中裸仓库.git下的内容](https://blog.csdn.net/qq_37375427/article/details/86584993)

本篇文章记录学习git中commit、tree和blob三个对象之间的关系。

首先需要会使用下面的命令：

*   cat 命令， 功能：用来显示文件。 例如 cat [text.md](http://text.md/) 显示 [text.md](http://text.md/) 文件的内容
*   ls -al 命令， 表示列出当前目录下的所有文件（包括隐藏文件）
*   git cat-file -t + 对象哈希值 命令 ， 查看 git 对象的类型
*   git cat-file -p + 对象哈希值 命令， 查看 git 对象的内容
*   git cat-file -s + 对象哈希值 命令， 查看 git 对象的大小

> 注意，在使用对象的哈希值的时候可以只取前几位数字，只要git不保存就行

*   commit我们很熟悉了
*   tree就类似于一棵树，这棵树下还存有其他的tree或者其他的blob文件
*   blob就是一个文件，是可以显示的文件。也就是可以使用git cat-file -p + 对象哈希值 来查看blob对象存的内容。

下面我们以之前我们做建立的git仓库为例，说明一下commit、tree、blob之间的关系，点击链接查看之前建立仓库的文章：[【Git、GitHub、GitLab】三 Git基本命令之创建仓库并向仓库中添加文件](https://blog.csdn.net/qq_37375427/article/details/86427150)

该仓库的根目录下的内容如下图;
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190121231516974.png#pic_center)

下图是针对上述仓库中，commit、tree以及blob三者之间的关系。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190121231732751.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3Mzc1NDI3,size_16,color_FFFFFF,t_70#pic_center)

> 其中注意一点：blob是根据文件的内容来区分的，只要文件内容一样，就只有一个blob，与文件名没有任何关系，大大的节约了存储空间

*   新建的Git仓库，有且仅有1个commit，仅仅包含 /doc/readme ，请问内含多少个tree，多少个blob？

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190121233428655.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3Mzc1NDI3,size_16,color_FFFFFF,t_70#pic_center)

*   一共包含两个tree，一个blob，一个commit。如上图所示。