# git）分支

2018年10月29日 16:14:18 [ccnuacmhdu](https://me.csdn.net/ccnuacmhdu) 阅读数：31更多

个人分类： [git](https://blog.csdn.net/ccnuacmhdu/article/category/8274629)

版权声明：本文为博主原创文章，未经博主允许不得转载。 https://blog.csdn.net/ccnuacmhdu/article/details/83504058

前面学习了git的基本知识，通过git命令可以方便地对仓库进行操作。这里想一下，如果是一个团队共同开发一个项目，这个项目有一个master分支，如果每个人写好的代码都往这个分支里面传，会造成无法修正的混乱局面。为此，再创建一个分支copy，所有人写好的代码都传到copy分支里面，由管理员负责把copy里的代码传到master分支，确保mater分支的稳定性和正确性。

同样地，对于个人来说，每个人在本地也是创建两个分支master分支和copy分支，这样，每个人只需要在copy里反复修改代码，写好了传到自己的master分支，确保自己的master分支正确性和稳定性。

# 创建、合并分支

[参考资料链接](https://www.cnblogs.com/zhoug2020/p/5917021.html)
当我们用git把文件传入仓库的时候，git保存的是每个文件版本的快照，并用指针把他们串起来，当只有一个分支（Master）的时候，Master指向当前最新版本，HEAD指针指向Master。而建立新的分支的时候，仅仅是建立指针，指向该文件的某个版本，同时HEAD指针指向当前正在使用的分支。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181029150813929.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NjbnVhY21oZHU=,size_27,color_FFFFFF,t_70)

创建新分支branch2。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181029151104645.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NjbnVhY21oZHU=,size_27,color_FFFFFF,t_70)

合并分支：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181029151338347.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NjbnVhY21oZHU=,size_27,color_FFFFFF,t_70)

上面仅仅是很简单的分支合并情况，如果branch2是从版本2开始修改的，或者不止两个分支，都会出现很多的合并问题，具体参见[参考资料链接](https://www.cnblogs.com/zhoug2020/p/5917021.html)。

#### 演示创建分支、合并分支

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181029152323482.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NjbnVhY21oZHU=,size_27,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181029152453381.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NjbnVhY21oZHU=,size_27,color_FFFFFF,t_70)
切换到Master分支：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2018102915253776.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NjbnVhY21oZHU=,size_27,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181029152615318.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NjbnVhY21oZHU=,size_27,color_FFFFFF,t_70)
现在master分支下面就有文件hello.txt了！！现在可以删除new_branch分支了：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181029152742231.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NjbnVhY21oZHU=,size_27,color_FFFFFF,t_70)

注意：在上述合并过程中，可能会提示Fast-forward模式（就是上面合并分支示意图的那种情况），这种模式合并之后，被合并的那个分支的信息就丢了，如果想保存整个合并的历史信息的话，可以禁用fast forward模式，在合并的时候加上–no–off参数即可`git merge --no-ff -m "merge with --no-ff" new_branch`。

# 合并冲突问题的解决

下面将要演示合并冲突的问题，比如对于文件hello.txt，内容是：
hello

（1）master分支修改hello.txt的内容为（修改后提交到master分支）：
hello

master write here
（2）new_branch分支修改hello.txt的内容为（修改后提交到new_branch分支）：
hello

new_branch write here

然后，把new_branch分支合并到master分支里面去，显然这两个文件的同一行内容不同，冲突！！这种情况只能手动解决，由管理员来修改hello.txt，然后传到master分支。具体演示如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181029160436124.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NjbnVhY21oZHU=,size_27,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181029160527663.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NjbnVhY21oZHU=,size_27,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181029160556965.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NjbnVhY21oZHU=,size_27,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181029160623585.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NjbnVhY21oZHU=,size_27,color_FFFFFF,t_70)
接下来需要自行打开hello.txt文件，修改后，上传到master，把分支new_branch删除。