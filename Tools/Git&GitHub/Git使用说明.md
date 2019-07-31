---
pin: true
tags: 
- Git
style: summer
flag: red
date: '2019-7-31'
custom: GJXAIOU
note: Git&GitHub使用说明,2019-2-11号形成初稿，2019-7-31做最后的总结
---


# Git&GitHub使用说明

@toc


## 一、下载安装

1.首先在本地安装git，[软件下载地址](https://git-scm.com/downloads)。

2.安装好Git 之后，会有两个 Git 形式：
- Git Bash   ：这是命令行模式（类似Linux）【推荐】
- Git GUI      ：这是有界面的模式

3.具体的安装过程详见[附录一：Git安装教程](#附录一：git-安装)

4.Git 整体结构
![Git结构 2]($resource/Git%E7%BB%93%E6%9E%84%202.png)

5.查看git配置（以下默认命令输入处为：GitBash）
`git config --list`



## 二、在本地建立一个仓库
- 例如在`E`盘下建立一个名为`practice`的仓库
```git
  cd e:
  mkdir practice
  cd practice
  ls -la #看现在里面有多少文件
  git init  #完成初始化，建立了一个空仓库
```

有两种方式可以建立Git仓库：

*   用Git之前已经有项目代码，则使用以下两条命令建立Git仓库
```git
  cd 项目代码所在的文件夹
  git init
```
*   用Git之前还没有项目代码，使用下面三条命令创建git仓库
```
  cd 某个文件夹
  git init your_project #会在当前路径下创建和项目名称同名的文件夹
  cd your_project
```
然后就是在自己的仓库中开始写代码或者更改代码了。

## 三、工作区和版本库区别：

![工作区与版本库的区别]($resource/%E5%B7%A5%E4%BD%9C%E5%8C%BA%E4%B8%8E%E7%89%88%E6%9C%AC%E5%BA%93%E7%9A%84%E5%8C%BA%E5%88%AB.png)

注：示例文件名`test.txt`
- 当使用`git add `将文件提交到暂存区之后，仍然在本地继续修改；
  - 使用：`git commit -m 'shuoming' `提交只是提交第一个修改的，后面修改的不提交
  - 使用：`git commit -m 'shuoming' test.txt`提交的将是后面修改的版本；
  - 使用：`git diff head -- test.txt`可以显示工作区和分支的区别



## 四、撤销修改

- 工作区修改之后，未提交到暂存区：`git checkout -- test.txt`,撤销工作区的修改；
- 工作区修改后，已提交到暂存区：`git reset head test.txt`,撤销暂存区的修改，然后再使用上面的命令撤销工作区的修改；


## 五、删除文件

- 删除工作区文件：`rm test.txt`
- 删除暂存区文件：(删除的修改操作，要提交)
`git rm test.txt`
`git commit -m &quot;delete test.txt&quot;`


## 六、配置Git

- 下面是使用Git的最小配置
1.打开Git Bash :切换到下面文件目录下；（或者在文件资源器中进入该目录，然后右键：Git Bash Here）
配置`user.name`和`user.email` 其中global表示对登录用户的所有仓库有效，是用于区分不同的开发人员身份；
注：以上信息都保存在.git 文件夹中，存放的是本地库相关的子目录和文件，可以使用：`ll .git/`查看，同时，可以使用`cat .git/config`进行查看配置文件，这里看到所有的用户
2.输入：`git config --global user.name '随便用户名'`
3.输入：`git config --global user.email '随便邮箱'`
```git
 git config --local         local只对仓库有效。此也为缺省条件下的配置
 git config --global        global对登录用户的所有仓库都有效
 git config --system        system对系统的所有用户都有效
```
上面三个作用域是由一定的优先级的：local > global > system




## 七、git文件重命名的命令

>前提是需要修改git仓库中已经commit了的文件

- 假如想使文件`readme`，改名为 `readme.cpp`。可以使用下面的三条命令：
```git
mv readme readme.cpp
git rm readme //注意与linux 的命令rm的区别
git add readme.cpp
```
最后可以使用git status查看修改后的仓库的状态
上述是三条命令，还可以使用下面的一条命令达到上述三条命令的效果
 `git mv readme readme.cpp`
 
- 如果想要撤销刚刚的修改，使用下面的命令：
`git reset --hard`


## 八、 如何使用git log系列命令查看git仓库的版本演变历史
```
git branch -v 看本地有多少版本分支
git log --all 查看所有分支的历史
git log --all --graph 查看图形化的 log 地址
git log --oneline 查看单行的简洁历史。
git log --oneline -n4 查看最近的四条简洁历史。
git log --oneline --all -n4 --graph 查看所有分支最近 4 条单行的图形化历史。
git help --web log 跳转到git log 的帮助文档网页
git log --oneline temp 只查看temp的分支的信息且oneline形式
```
- 注意：
  - git log 仅仅是查看当前分支的更改历史
  - git log --all 查看所有分支的信息
  - git log --all --graph 上面的查看不容易看各个分支的关系，加上图形化容易看出各个分支的关系
  - git checkout -b temp +从哪个commit开始新加一个分支(一大串数字,表示某一个commit中的数字 )
  - git checkout master // 切换到master分支

*   git命令的后面加的参数，有的时候是-，有的时候是- -
*   单字母的参数是 ‘-’，非单字母的参数是’- -’




## 九、本地.git 文件目录

*   COMMIT_EDITMSG
*   cconfig,当前仓库的配置信息,core,用户,远程,分支等信息.(命令操作其实就是修改当前config文件)
*   description （仓库的描述信息文件）
*   HEAD （指向当前所在的分支），例如当前在 develop 分支，实际指向地址是 refs/heads/develop
*   hooks [文件夹]
*   index
*   info [文件夹]
*   logs [文件夹]
*   objects [文件夹] （存放所有的 git 对象，对象哈希值前 2 位作为文件夹名称，后 38 位作为对象文件名, 可通过 git cat-file -p 命令，拼接文件夹名称+文件名查看）
*   ORIG_HEAD
*   refs
    - heads,其实就是分支,里面包含所有的分支文件,文件存储了分支指向的指纹信息
    - tags 叫做里程碑,或者版本发布用等记录重要版本.文件也存储了tag的指纹信息
    - remotes,远程仓库信息

## 十、版本回退

分别修改几次hello.txt，并分别提交。
**版本1：**
hello
**版本2：**
hello world
**版本3：**
hello world third
**版本4：**
hello  world  third last
以上版本都提交之后，如果想回到第二个版本：

- 方式一：【推荐】
1.首先输入：`git reflog` 会得到之后所有该文件的提交版本信息，其中最前面黄色字体为提交的版本号；
注：可以使用`git log`查看该文件每次提交的详细信息（包括：完整的版本号，作者，时间等），也可以使用：`git log --pretty=online`按行显示以上信息；
其中：Head 含义：HEAD@{移动到当前版本需要多少步}
2.输入：`git reset --hard 第二次提交版本号` （这里版本号填写 `git reflog`显示的即可，不必填写完整的版本号）即可回退到第二个版本；

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181028233812515.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NjbnVhY21oZHU=,size_27,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181028234106489.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NjbnVhY21oZHU=,size_27,color_FFFFFF,t_70)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181028234356533.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NjbnVhY21oZHU=,size_27,color_FFFFFF,t_70)

- 方式二：只能后退
`git reset --head HEAD^`其中一个^表示后退一步，n 个表示后退 n 步；
- 方式三：只能后退
`git reset --head HEAD~n`,表示后退 n 步

### 补充：reset 参数
![搜狗截图20190731203134]($resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20190731203134.png)

- 参数一:`--soft`  ：仅仅在本地库移动 HEAD 指针；
- 参数二:`--mixed` ：在本地库移动 HEAD 指针，同时重置暂存区；
- 参数三:`--hard` ：在本地库移动 HEAD 指针，同时重置暂存区和工作区；


### 删除文件并找回
前提：删除前，文件存在时的状态提交到了本地库；
操作：`git reset --head 指针位置`
- 删除操作已经提交到了本地库：指针位置指向历史记录
- 删除操作尚未提交到本地库：指针位置指向 HEAD

### 比较文件的差异
- 将工作区文件和暂存区进行比较：`git diff 文件名`
- 将工作区文件和本地库历史记录比较：`git diff 本地库中历史版本 文件名`
- 以上不带文件名就是比较多个文件；







## GitHub

### 1.创建远程仓库别名
- 查看当前远程仓库地址别名：`git remote -v`
- 为远程仓库添加别名：`git remote add 别名 远程仓库地址`

### 2.推送
`git push 别名 分支名`





## 十一、连接到GitHub

### （一）将GitHub上的库克隆到本地

说明：默认文件地址为：E:\Program\GitHub\
         GitHub上的库为：Notes
4.将GitHub上已有的仓库下载到本地（这样可以在本地编辑）
- 进入GitHub中所要同步的库页面，选择：`clone or download`，然后将URL复制下来：本人为：`https://github.com/GJXAIOU/Notes.git`
![URL]($resource/URL.jpg)

- 在Git Bash 中首先将目录切换到想要保存该库的目录下：然后输入
`git clone https://github.com/GJXAIOU/Notes.git` (后面的网址就是刚才复制的)


### （二）连接本地仓库与GitHub：

1.在本地生成SSH Key :  在Git Bash中输入：`ssh-keygen -t rsa -C "GitHub邮箱"`，接下来**全部回车**确认，这样在电脑C盘本人用户名目录下就会出现一个文件：`.ssh`
2.将上面文件中的`id_rsa`使用记事本打开，然后全选复制；
3.将上面复制的秘钥，黏贴到GitHub中
![搜狗截图20190226204854]($resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20190226204854.png)
![搜狗截图20190226205651]($resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20190226205651.png)

补充：github上新建仓库：`mimengfenxi`
使用：
```git
#将本地的master分支与GitHub仓库分支进行关联
git remote add origin https://github.com/GJXAIOU/mimengfenxi.git #后面为仓库的SSH地址

##这步不需要,这是移除关联
git remote rm origin https://github.com/GJXAIOU/mimengfenxi.git 

#注：origin是自己为远程GitHub仓库取的别名，主要是方便以后书写

#下面的步骤只需要第一次的时候配置以下：
步骤一：配置上面的公钥和私钥
步骤二；输入：ssh -T -v git@github.com

#这一步是指从本地直接生成库，而不是克隆的上传使用（一般也没有）
git pull --rebase origin master #相当于将GitHub创建仓库时候多创建的README.md拖下来

#从GitHub往本地克隆，最好保证本地仓库是空的；同样本地上传到GitHub上时候最好保证GitHub新建的仓库是空的；

#将本地的master分支上传到GitHub
git push -u origin master
```

### （三）上传同步本地文件（本地目录下文件有修改想同步到GitHub时候）
1.在刚才的本地目录下，使用Git Bash ,输入：`Git init` 初始化本地git仓库；
然后在改目录下会出现：`.git`这个文件  （只需要一次，以后不需要）

2.进入修改目录下；输入：`git status` :查看多少文件未提交到仓库中（显示为红色的为未提交的），用于查看工作区和暂存区的状态；

3.将上述所有文件从工作区提交到缓存区：`git add . `
将特定文件提交大缓冲区：`git add 文件名称.后缀`

4.再使用命令`git status`查看是不是都提交到了缓冲区（显示为绿色的是的）

5.将缓冲区的文件提交到GitHub仓库中：`git commit -m "想显示文件后面的的描述"`

6.最后一步：`git push`



# 附录：

## 附录一：Git 安装
TODO：Git 和 GitHub

