# 【Git、GitHub、GitLab】九 工作中非常重要的一些git用法

2019年01月23日 10:07:06 [杨柳_](https://me.csdn.net/qq_37375427) 阅读数：45更多

所属专栏： [Git、GitHub、GitLab学习专栏](https://blog.csdn.net/column/details/19366.html)

版权声明：本文为博主原创文章，未经博主允许不得转载，转载请加博主qq：1126137994或者微信：liu1126137994 https://lyy-0217.blog.csdn.net/article/details/86597260

> 上一篇文章学习了 如何修改commit的message，点击链接查看：[【Git、GitHub、GitLab】八 如何修改commit的message](https://blog.csdn.net/qq_37375427/article/details/86592438)

> 注意；下面的‘–’ 都是两个‘-’组成

*   本文介绍一些在使用git中非常常用的一些命令：

1.  怎么比较暂存区与HEAD所指向的文件的差异？

*   git diff --cached

1.  怎么比较工作区和暂存区所含文件的差异？

*   git diff
*   git diff – filename （中间为两个’-’）

1.  如何将暂存区所有文件恢复为和HEAD所存文件一样？

*   git reset HEAD

1.  如何将工作区内容恢复为暂存区内容？

*   git checkout
*   git checkout – filename

1.  怎样恢复暂存区部分文件为HEAD的一样？

*   git reset HEAD – filename1 filename2

1.  如何消除最近的几次提交？

*   git reset --hard commit_id

1.  看不同提交的指定文件的差异

*   git diff 分支1 分支2 – filename
*   git diff 分支1commit 分支2commit – filename

1.  正确删除git仓库文件的方法

*   git rm filename

1.  开发中临时加塞了紧急任务如何处理？

*   先将手头上的暂存区的文件内容放到一边存起来，先去做紧急任务
*   先 git stash 将正在做的暂存区的内容存到stash中
*   然后干其他紧急任务
*   紧急任务完成后使用:
*   git stash apply : stash列表中的信息还在，可以反复使用
*   git stash pop ： stash列表中的细信息不在了