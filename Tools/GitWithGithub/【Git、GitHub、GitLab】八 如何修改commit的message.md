# 【Git、GitHub、GitLab】八 如何修改commit的message

2019年01月22日 15:21:36 [杨柳_](https://me.csdn.net/qq_37375427) 阅读数：221更多

所属专栏： [Git、GitHub、GitLab学习专栏](https://blog.csdn.net/column/details/19366.html)

版权声明：本文为博主原创文章，未经博主允许不得转载，转载请加博主qq：1126137994或者微信：liu1126137994 https://lyy-0217.blog.csdn.net/article/details/86592438

> 上一篇文章记录了git中分支的删除以及出现分离头指针的情况，点击查看:[【Git、GitHub、GitLab】七 git中分支的删除以及出现分离头指针的情况](https://blog.csdn.net/qq_37375427/article/details/86591584)

> ### 文章目录
> 
> *   [1 如何修改最新的commit的message](https://lyy-0217.blog.csdn.net/article/details/86592438#1_commitmessage_4)
> *   [2 如何修改老旧的commit的message](https://lyy-0217.blog.csdn.net/article/details/86592438#2_commitmessage_11)
> *   [3 如何将连续的多个commit整理成一个](https://lyy-0217.blog.csdn.net/article/details/86592438#3_commit_25)
> *   [4 如何将间隔的多个的commit合并成一个commit](https://lyy-0217.blog.csdn.net/article/details/86592438#4_commitcommit_38)

## 1 如何修改最新的commit的message

使用下面的命令即可

*   git commit --amend

> –amend 不只是修改最新commit的message 而是会创建一个将暂存区的内容生成一个commit，再将当前最新的commit替换成新生成的那一个

## 2 如何修改老旧的commit的message

使用命令：

*   git rebase -i parent_commit_id

然后对要修改的message前的cmd改为`reword`，保存退出后修改message

> 其中， rebase -i 会产生分离头指针的状态。
> 因为：
> 
> *   git rebase工作的过程中，就是用了分离头指针。rebase意味着基于新base的commit来变更部分commits。它处理的时候，把HEAD指向base的commit，此时如果该commit没有对应branch，就处于分离头指针的状态，然后重新一个一个生成新的commit，当rebase创建完最后一个commit后，结束分离头状态，Git让变完基的分支名指向HEAD。

> 这里有一个问题就是rebase后面跟的是父commit的哈希值，那么第一个commit的message如何修改？

## 3 如何将连续的多个commit整理成一个

使用如下命令：

*   git rebase -i 开始commit [结束commit]

然后对要修改的message前的cmd改为`squash`，保存退出后可以在new commit下加一句总体的commit message

> 在执行这个命令时
> 
> *   如果没有指定 结束commit,那么结束commit 默认为当前分支最新的 commit，那么rebase 结束后会自动更新当前分支指向的 commit, 如果指定了结束 commit，而且结束 commit不是当前分支最新的
>     commit，那么rebase 后会有生成一个 游离的 head,，而且当前分支指向的commit 不会更新

## 4 如何将间隔的多个的commit合并成一个commit

上面是将连续的多个commit 合并成一个，那么如何将不连续的多个间隔的commit合并成一个？

使用下面的命令：

*   git rebase -i 最靠前需要合并的commit的父commit_id，如果是第一个commit，那么直接就是第一个commit_id

然后将要合并的几条commit的message放到一起（紧挨着最靠前的需要合并的commit），且相关commit前的cmd改为`squash`

然后保存退出。

> 后面解决冲突后，先 git add，在git rebase --continue