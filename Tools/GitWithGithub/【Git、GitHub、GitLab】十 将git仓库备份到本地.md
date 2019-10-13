# 【Git、GitHub、GitLab】十 将git仓库备份到本地

2019年01月23日 19:29:30 [杨柳_](https://me.csdn.net/qq_37375427) 阅读数：54更多

所属专栏： [Git、GitHub、GitLab学习专栏](https://blog.csdn.net/column/details/19366.html)

版权声明：本文为博主原创文章，未经博主允许不得转载，转载请加博主qq：1126137994或者微信：liu1126137994 https://lyy-0217.blog.csdn.net/article/details/86616141

> 上一篇文章学习记录了工作中常用的一些git命令，点击链接查看：[【Git、GitHub、GitLab】九 工作中非常重要的一些git用法](https://blog.csdn.net/qq_37375427/article/details/86597260)

> ### 文章目录
> 
> *   [1 git的传输协议](https://lyy-0217.blog.csdn.net/article/details/86616141#1_git_4)
> *   [2 如何将git仓库备份到本地](https://lyy-0217.blog.csdn.net/article/details/86616141#2_git_15)
> 
> *   [2.1 使用哑协议备份](https://lyy-0217.blog.csdn.net/article/details/86616141#21__16)
> *   [2.2 使用智能协议备份](https://lyy-0217.blog.csdn.net/article/details/86616141#22__24)

# 1 git的传输协议

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190123185533638.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3Mzc1NDI3,size_16,color_FFFFFF,t_70#pic_center)

哑协议与智能协议的区别：

*   直观区别： 哑协议传输进度不可见；智能协议传输可见。
*   传输速度： 智能协议比哑协议传输速度快。

备份特点：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019012319074266.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3Mzc1NDI3,size_16,color_FFFFFF,t_70#pic_center)

# 2 如何将git仓库备份到本地

## 2.1 使用哑协议备份

使用以下命令：

*   git clone --bare /path/to/.git ya.git

*   上述命令中/path/to/.git 是git仓库的路径，备份到本地的ya.git文件中

*   参数 --bare的意思是不带工作区的裸仓库

## 2.2 使用智能协议备份

使用以下命令：

*   git clone --bare file:///path/to/.git ya.git zhineng.git
*   git仓库前面带上file://前缀就是使用智能协议传输，传输的时候有进度条