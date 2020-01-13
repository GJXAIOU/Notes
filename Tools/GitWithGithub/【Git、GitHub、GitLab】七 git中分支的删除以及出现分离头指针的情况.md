# 【Git、GitHub、GitLab】七 git中分支的删除以及出现分离头指针的情况

2019年01月22日 12:02:37 [杨柳_](https://me.csdn.net/qq_37375427) 阅读数：137更多

所属专栏： [Git、GitHub、GitLab学习专栏](https://blog.csdn.net/column/details/19366.html)

版权声明：本文为博主原创文章，未经博主允许不得转载，转载请加博主qq：1126137994或者微信：liu1126137994 https://lyy-0217.blog.csdn.net/article/details/86591584

> 上一篇文章学习了GIT中commit、tree和blob三个对象之间的关系，点击链接查看：[【Git、GitHub、GitLab】六 GIT中commit、tree和blob三个对象之间的关系](https://blog.csdn.net/qq_37375427/article/details/86586584)

> ### 文章目录
> 
> *   [1 git中如何删除分支](https://lyy-0217.blog.csdn.net/article/details/86591584#1_git_4)
> *   [2 分离头指针的情况需要注意什么](https://lyy-0217.blog.csdn.net/article/details/86591584#2__15)

# 1 git中如何删除分支

> 如何查看分支？
> 
> *   git branch -av

有时候会需要删除一些不需要的分支。

*   git branch -d branch_name/id：使用-d 在删除前Git会判断在该分支上开发的功能是否被merge到其它分支。如果没有，不能删除。如果merge到其它分支，但之后又在其上做了开发，使用-d还是不能删除。
*   git branch -D branch_name/id：-D会强制删除。

# 2 分离头指针的情况需要注意什么

*   git checkout commit_id：会出现分离头指针的情况，这种情况下比较危险，因为这个时候你提交的代码没有和分支对应起来，当切换到其他分支的时候(比如master分支)，容易丢失代码；

*   但是分离头指针也有它的应用场景，就是在自己做尝试或者测试的时候可以分离头指针，当尝试完毕没有用的时候可以随时丢弃，但是如果觉得尝试有用，那么可以新建一个分支，使用 git branch <新分支的名称> commit_id