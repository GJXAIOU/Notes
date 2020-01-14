---
pin: true
tags: 
- Git
style: summer
flag: red
date: '2019-7-31'
custom: GJXAIOU
note: Git&GitHub使用说明,2019-2-11号形成初稿，2019-7-31进行整体迭代，2019-8-6号形成终稿
---

# Git & GitHub 使用说明

==原则：不要在远程仓库（GitHub）中直接修改文件，应该在本地修改之后 push 到远程仓库。==

## 一、Git

### （一）Git 安装
[软件下载地址](https://git-scm.com/downloads)，选择对应系统版本正常安装即可，按照过程可参考[该博客](https://blog.csdn.net/qq_32786873/article/details/80570783)。

### （二）Git 简介
- Git 使用模式：
  - Git Bash   ：这是命令行模式（类似Linux）【推荐】
  - Git GUI      ：这是有界面的模式
  
- Git 整体结构：
  <img src="Git&amp;Github%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E.resource/Git%E7%BB%93%E6%9E%84%202.png" alt="Git结构 2" style="zoom:50%;" />

- 查看 Git 基本配置：`git config --list`  基本不使用；


### （三）Git 配置

下面是使用 Git 的最小配置
- 步骤一：首先在任意文件夹（路径尽量不要有中文）的空白处右击，选择 `git bash here`，即可打开 Git 命令行窗口并处于该文件夹路径下。
- 步骤二：输入命令 `git config --global user.name '任意用户名'`
- 步骤三：输入命令 `git config --global user.email '随便邮箱'`

【注】
- 上面两步骤是配置`user.name`和`user.email` 其中 global 表示对登录用户的所有仓库有效，是用于区分不同的开发人员身份；
- 以上信息都保存在 .git 文件夹中，存放的是本地库相关的子目录和文件，可以使用：`ll .git/`查看，同时，可以使用`cat .git/config`进行查看配置文件，这里看到所有的用户

```git
 git config --local         local 只对仓库有效。此也为缺省条件下的配置
 git config --global        global 对登录用户的所有仓库都有效
 git config --system        system 对系统的所有用户都有效
```
上面三个作用域是由一定的优先级的：local > global > system



### （三）在本地建立一个仓库

- 建立本地 Git 仓库的两种方式：
  *   如果用 Git 之前已经有项目代码，则使用以下两条命令建立本地 Git 仓库
```git
  cd 项目代码所在的文件夹
  git init
```
或者向上面那样，进入到项目路径所处的文件夹，然后右击：`git bash here`，在命令行窗口输入：`git init` 也可以。

  *   用Git之前还没有项目代码，使用下面三条命令创建本地 Git 仓库
```
  cd 路径
  git init your_project #会在当前路径下创建和项目名称同名的文件夹
  cd your_project
```
然后就是在自己的仓库中开始写代码或者更改代码了。


## 二、GitHub

### （一）将 GitHub 上的仓库克隆到本地

**演示说明：** 默认本地 文件地址为：`E:\Program\GitHub\`，GitHub上的库为：`Notes`

**步骤：**

- 进入GitHub 中所要同步的库页面，选择：`clone or download`，然后将URL复制下来，示例为：`https://github.com/GJXAIOU/Notes.git`
![image-20200114215859346](Git&Github%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E.resource/image-20200114215859346.png)

- 在 Git Bash 中首先将目录切换到想要保存该库的目录下，然后输入命令
`git clone https://github.com/GJXAIOU/Notes.git` (后面的网址就是刚才复制的)，这样就把 Github 在线仓库 clone 到了本地；
**至此就实现了将 GitHub 上仓库复制到本地的操作**，GitHub 下载较慢，耐心等待一会。

### （二）连接本地仓库与 GitHub
**作用**：实现 GitHub 账号和本地关联，使得下面你有权限在本地修改 GitHub 上面的仓库内容，所以只有更换机器的时候需要关联。

- 步骤一：注册 GitHub 账号，换个头像，折腾折腾。。。

- 步骤二：在本地生成 SSH Key :  

  在 Git Bash 中（如果不声明位置，默认任何位置右击选择 `git bash here` 打开命令行窗口即可）输入命令：`ssh-keygen -t rsa -C "GitHub 邮箱"`，接下来**全部回车**确认，这样在**电脑 C 盘本人用户名目录下**就会出现一个文件：`.ssh`

- 步骤三：将上面 `.ssh` 文件中的 `id_rsa` 使用记事本打开，然后全选复制内容；

- 步骤四：将上面复制的秘钥内容，黏贴到 GitHub 中（右上角头像旁边的下拉按钮点击 setting 就会到该页面）

  ![image-20200114220244937](Git&Github%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E.resource/image-20200114220244937.png)

  ![搜狗截图20190226204854](Git&Github%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E.resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20190226204854.png
  ![搜狗截图20190226205651](Git&Github%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E.resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20190226205651.png)

- 步骤六：将本地的 master 分支与 GitHub 仓库分支进行关联,仓库别名为 notes（可以自己制定，这个名字就是以后修改 GitHub 仓库时候，如果你有多个仓库，需要制定修改的是哪个仓库，相当于为 GitHub 仓库取个别名）
`git remote add notes https://github.com/GJXAIOU/Notes.git` # 后面为仓库的 SSH 地址，相当于以后使用 `notes` 指定自己操作的是名叫 `Notes` 的仓库 

- 步骤：这步**不需要**，这是移除关联，如果取错了或者想换个名字可以使用 `git remote remove 名字` 取消关联，然后使用步骤六重新关联。

【注】查看当前远程仓库地址别名命令：`git remote -v`


### （三）上传同步本地文件
**作用**主要用于自己上传文件到 GitHub。

**重要**：在 GitHub 上新创建仓库时候，一般都会自动创建 README.md 文件（可能没有，取决于自己选择），所以

- 步骤一：在想存放 GitHub 仓库的本地目录下，使用 Git Bash  命令`git init` 初始化本地 git 仓库；
- 步骤二：使用命令 `git pull GitHub仓库的SSH地址`，例如 `git pull https://github.com/GJXAIOU/Notes.git`，将仓库中已经有的文件先拉到本地。
- 步骤三：进入仓库目录下，输入：`git status` 可以查看多少文件未提交到仓库中（显示为红色的为未提交的），该命令用于查看工作区和暂存区的状态；
- 步骤四：将所在位置下的**所有文件**从工作区提交到缓存区：`git add . ` ， 将特定文件提交大缓冲区：`git add 具体的文件名（包括后缀）`
- 步骤五：再使用命令 `git status` 查看是不是想要提交的文件都提交到了缓冲区（显示为绿色的是的）
- 步骤六：为本次修改增加备注，使用命令： `git commit -m "本次提交备注描述"`
- 步骤七：推送到 GitHub，使用命令： `git push 仓库名 分支名`，仓库名就是上面起的别名，这里为：`notes`，分支名默认为 `master`。

==重要：== 
- 不要在 GitHub 上对仓库内文件做任何的修改，如果有修改在本地修改之后然后推送到 GitHub 上，否则如果本地和 GitHub 上都进行修改会造成文件冲突。
- 如果在 GitHub 上进行了修改，请使用上面的`git pull GitHub仓库的SSH地址`，例如 `git pull https://github.com/GJXAIOU/Notes.git`，将仓库中的文件先拉到本地。然后本地进行文件其它修改之后再使用上面命令传入 GitHub，否则会出现 rejected 错误。
- 如果想仅仅删除 GitHub 上文件，不删除本地该文件，使用以下命令：
 **删除缓冲区和 github 文件**，本地不删除
`git rm -r --cached 文件名或者文件夹名`
`git commit -m "本次修改备注内容"`
`git push 仓库名字 master`

### （四）平时使用时候上次文件步骤
- `git add .` 或者 `git add 文件名或文件夹`
- `git commit -m "提交的备注"`
- `git push 仓库别名 master`



--------下面可以忽略不看，有需求时候可以在学习---------



### （四）工作区和版本库区别

![工作区与版本库的区别](Git&Github%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E.resource/%E5%B7%A5%E4%BD%9C%E5%8C%BA%E4%B8%8E%E7%89%88%E6%9C%AC%E5%BA%93%E7%9A%84%E5%8C%BA%E5%88%AB.png)

注：示例文件名`test.txt`
- 当使用`git add  test.txt`将文件提交到暂存区之后，仍然在本地继续修改，如果产生了新的修改；
  - 使用：`git commit -m '说明信息' `提交只是提交第一个修改的，后面修改的不提交；
  - 使用：`git commit -m '说明信息' test.txt`提交的将是后面修改的版本；
  - 使用：`git diff head -- test.txt`可以显示工作区和分支的区别；


### （五）撤销修改
- 工作区（本地）修改之后，未提交到暂存区：`git checkout -- test.txt`,撤销工作区的修改；
- 工作区修改后，已提交到暂存区：`git reset head test.txt`,撤销暂存区的修改，然后再使用上面的命令撤销工作区的修改；


### （六）删除文件
- 删除工作区文件：`rm test.txt` 就是删除本地文件
- 删除暂存区文件：(删除的修改操作，要提交)：这里是本地和 github 都删除
`git rm test.txt`
`git commit -m "delete test.txt"`
`git push 名字`

==仅仅删除 Github 端文件==
- **删除缓冲区和 github 文件**，本地不删除
`git rm -r --cached 文件名或者文件夹名`  --cached 不会将本地文件删除
`git commit -m "XXXXX"`
`git push 名字 master`




### （七）Git文件重命名的命令

前提是需要修改git仓库中已经commit了的文件

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


### （八）查看git仓库的版本演变历史
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
提交记录可能会非常多，按 `J` 键往下翻，按 `K` 键往上翻，按 `Q` 键退出查看

### （九）版本回退

分别修改几次hello.txt，并分别提交。
版本1：hello
版本2：hello world
版本3：hello world third
版本4：hello  world  third last
以上版本都提交之后，如果想回到第二个版本：

- 方式一：【推荐】
1.首先输入：`git reflog` 会得到之后所有该文件的提交版本信息，其中最前面黄色字体为提交的版本号；
注：可以使用`git log`查看该文件每次提交的详细信息（包括：完整的版本号，作者，时间等），也可以使用：`git log --pretty=online`按行显示以上信息；
其中：Head 含义：HEAD@{移动到当前版本需要多少步}
2.输入：`git reset --hard 第二次提交版本号` （这里版本号填写 `git reflog`显示的即可，不必填写完整的版本号）即可回退到第二个版本；

![在这里插入图片描述](Git&Github%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E.resource/20181028233812515.png)

![在这里插入图片描述](Git&Github%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E.resource/20181028234106489.png)

![在这里插入图片描述](Git&Github%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E.resource/20181028234356533.png)

- 方式二：只能后退
`git reset --head HEAD^`其中一个^表示后退一步，n 个表示后退 n 步；
- 方式三：只能后退
`git reset --head HEAD~n`,表示后退 n 步

 **补充：reset 参数**
- 参数一:`--soft`  ：仅仅在本地库移动 HEAD 指针；
- 参数二:`--mixed` ：在本地库移动 HEAD 指针，同时重置暂存区；
- 参数三:`--hard` ：在本地库移动 HEAD 指针，同时重置暂存区和工作区；


### （十）删除文件并找回
前提：删除前，文件存在时的状态提交到了本地库；
操作：`git reset --head 指针位置`
- 删除操作已经提交到了本地库：指针位置指向历史记录
- 删除操作尚未提交到本地库：指针位置指向 HEAD

### （十一）比较文件的差异
- 将工作区文件和暂存区进行比较：`git diff 文件名`
- 将工作区文件和本地库历史记录比较：`git diff 本地库中历史版本 文件名`
- 以上不带文件名就是比较多个文件；





## 二、连接到GitHub

### （一）将GitHub上的库克隆到本地

**演示说明：** 默认本地 文件地址为：E:\Program\GitHub\
         GitHub上的库为：Notes
**步骤：**
1.将GitHub上已有的仓库下载到本地（这样可以在本地编辑）
- 进入GitHub中所要同步的库页面，选择：`clone or download`，然后将URL复制下来：本人为：`https://github.com/GJXAIOU/Notes.git`
![URL](Git&Github%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E.resource/URL.jpg)

- 在Git Bash 中首先将目录切换到想要保存该库的目录下：然后输入
`git clone https://github.com/GJXAIOU/Notes.git` (后面的网址就是刚才复制的)，这样就把 Github 在线仓库 clone 到了本地；


### （二）连接本地仓库与GitHub：

1.在本地生成SSH Key :  在Git Bash中输入：`ssh-keygen -t rsa -C "GitHub邮箱"`，接下来**全部回车**确认，这样在电脑C盘本人用户名目录下就会出现一个文件：`.ssh`
2.将上面文件中的`id_rsa`使用记事本打开，然后全选复制；
3.将上面复制的秘钥，黏贴到GitHub中
![搜狗截图20190226204854](Git&Github%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E.resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20190226204854.png)
![搜狗截图20190226205651](Git&Github%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E.resource/%E6%90%9C%E7%8B%97%E6%88%AA%E5%9B%BE20190226205651.png)



### （三）具体使用

#将本地的master分支与GitHub仓库分支进行关联,仓库别名为notes
`git remote add notes https://github.com/GJXAIOU/Notes.git` #后面为仓库的SSH地址

##这步不需要,这是移除关联
`git remote remove 名字`
#注：notes 是自己为远程GitHub仓库取的别名，主要是方便以后书写
#注：查看当前远程仓库地址别名：`git remote -v`

#下面的步骤只需要第一次的时候配置以下：
步骤一：配置上面的公钥和私钥
步骤二；输入：`ssh -T -v git@github.com`

#这一步是指从**本地直接生成库**，而不是克隆的上传使用（一般也没有）
`git pull --rebase origin master` #相当于将GitHub创建仓库时候多创建的README.md拖下来

#从GitHub往本地克隆，最好保证本地仓库是空的；同样本地上传到GitHub上时候最好保证GitHub新建的仓库是空的；

#将本地的master分支上传到GitHub;格式为：git push 仓库别名 分支名
`git push -u notes master`


### （三）上传同步本地文件
1.在刚才的本地目录下，使用Git Bash ,输入：`Git init` 初始化本地git仓库；
然后在改目录下会出现：`.git`这个文件  （只需要一次，以后不需要）

2.进入修改目录下；输入：`git status` :查看多少文件未提交到仓库中（显示为红色的为未提交的），用于查看工作区和暂存区的状态；

3.将上述所有文件从工作区提交到缓存区：`git add . `
将特定文件提交大缓冲区：`git add 文件名称.后缀`

4.再使用命令`git status`查看是不是都提交到了缓冲区（显示为绿色的是的）

5.将缓冲区的文件提交到GitHub仓库中：`git commit -m "想显示文件后面的的描述"`

6.最后一步：`git push 仓库名 分支名`


### （四）分支的使用

查看各个分支推动的内容： Github网站的项目首页，点击 `Branch:master` 下拉按钮，就会看到各个分支，选择查看即可；

**git branch**
创建、重命名、查看、删除项目分支，通过 `Git` 做项目开发时，一般都是在开发分支中进行，开发完成后合并分支到主干。
- `git branch daily/0.0.0`创建一个名为 `daily/0.0.0` 的日常开发分支，分支名只要不包括特殊字符即可。

- `git branch -m daily/0.0.0 daily/0.0.1`如果觉得之前的分支名不合适，可以为新建的分支重命名，重命名分支名为 `daily/0.0.1`

- `git branch`通过不带参数的branch命令可以查看当前项目分支列表

- `git branch -d daily/0.0.1`如果分支已经完成使命则可以通过 `-d` 参数将分支删除，这里为了继续下一步操作，暂不执行删除操作

**git checkout**切换分支
- `git checkout daily/0.0.1`切换到 `daily/0.0.1` 分支，后续的操作将在这个分支上进行

### （五）将 Github 上最新版拉到本地

**git pull**：将服务器上的最新代码拉取到本地

`git pull origin daily/0.0.1`

如果其它项目成员对项目做了改动并推送到服务器，我们需要将最新的改动更新到本地，这里我们来模拟一下这种情况。

进入Github网站的项目首页，再进入 `daily/0.0.1` 分支，在线对 `README.md` 文件做一些修改并保存，然后在命令中执行以上命令，它将把刚才在线修改的部分拉取到本地，用编辑器打开 `README.md` ，你会发现文件已经跟线上的内容同步了。

如果线上代码做了变动，而你本地的代码也有变动，拉取的代码就有可能会跟你本地的改动冲突，一般情况下 Git 会自动处理这种冲突合并，但如果改动的是同一行，那就需要手动来合并代码，编辑文件，保存最新的改动，再通过 `git add .` 和 `git commit -m 'xxx'` 来提交合并。


### （六）版本标记

 **git tag**：为项目标记里程碑

```
git tag publish/0.0.1
git  push origin publish/0.0.1
```

当我们完成某个功能需求准备发布上线时，应该将此次完整的项目代码做个标记，并将这个标记好的版本发布到线上，这里我们以 `publish/0.0.1` 为标记名并发布，当看到命令行返回如下内容则表示发布成功了
```
Total 0 (delta 0), reused 0 (delta 0)
To https://github.com/gafish/gafish.github.com.git
 * [new tag]         publish/0.0.1 -> publish/0.0.1
```

### （七）忽略推送文件

设置哪些内容不需要推送到服务器，使用配置文件`.gitignore`
```
touch .gitignore
```

`.gitignore` 不是 `Git` 命令，而在项目中的一个文件，通过设置 `.gitignore` 的内容告诉 `Git` 哪些文件应该被忽略不需要推送到服务器，通过以上命令可以创建一个 `.gitignore` 文件，并在编辑器中打开文件，每一行代表一个要忽略的文件或目录，如：

```
demo.html
build/
```

以上内容的意思是 `Git` 将忽略 `demo.html` 文件 和 `build/` 目录，这些内容不会被推送到服务器上



## 三、进阶知识点


### （一）基本概念

**工作区（Working Directory）**
就是你在电脑里能看到的目录，比如上文中的 文件夹就是一个工作区 。

**本地版本库（Local Repository）**
工作区有一个隐藏目录 `.git`，这个不算工作区，而是 `Git`的版本库。 
![版本库](Git&Github%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E.resource/%E7%89%88%E6%9C%AC%E5%BA%93.png)

 **暂存区（stage）**
本地版本库里存了很多东西，其中最重要的就是称为 `stage`（或者叫index）的暂存区，还有 `Git` 为我们自动创建的第一个分支 `master`，以及指向 `master` 的一个指针叫 `HEAD`。

**远程版本库（Remote Repository）**

一般指的是 `Git` 服务器上所对应的仓库，本文的示例所在的`github`仓库就是一个远程版本库 

**以上概念之间的关系**
`工作区`、`暂存区`、`本地版本库`、`远程版本库`之间几个常用的 `Git`操作流程如下图所示： 
![Git各区之间关系](Git&Github%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E.resource/Git%E5%90%84%E5%8C%BA%E4%B9%8B%E9%97%B4%E5%85%B3%E7%B3%BB.png)

**分支（Branch）**
分支是为了将修改记录的整个流程分开存储，让分开的分支不受其它分支的影响，所以在同一个数据库里可以同时进行多个不同的修改 
![分支](Git&Github%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E.resource/%E5%88%86%E6%94%AF.png)

**主分支（Master）**
前面提到过 `master` 是 `Git` 为我们自动创建的第一个分支，也叫主分支，其它分支开发完成后都要合并到 `master`
![分支和标签](Git&Github%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E.resource/%E5%88%86%E6%94%AF%E5%92%8C%E6%A0%87%E7%AD%BE.png)

**标签（Tag）**
标签是用于标记特定的点或提交的历史，通常会用来标记发布版本的名称或版本号（如：`publish/0.0.1`），虽然标签看起来有点像分支，但打上标签的提交是固定的，不能随意的改动，参见上图中的`1.0` / `2.0` / `3.0`

**HEAD**

`HEAD`指向的就是当前分支的最新提交 

![](Git&Github%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E.resource/640.webp)



### （二）操作文件

#### git add
添加文件到暂存区
`git add -i`
通过此命令将打开交互式子命令系统，你将看到如下子命令
```
***Commands***
  1: status      2: update      3: revert      4: add untracked
  5: patch      6: diff      7: quit      8: help
```

通过输入序列号或首字母可以选择相应的功能，具体的功能解释如下：
*   `status`：功能上和 `git add -i` 相似，没什么用
*   `update`：详见下方 `git add -u`
*   `revert`：把已经添加到暂存区的文件从暂存区剔除，其操作方式和 `update` 类似
*   `add untracked`：可以把新增的文件添加到暂存区，其操作方式和 `update` 类似
*   `patch`：详见下方 `git add -p`
*   `diff`：比较暂存区文件和本地版本库的差异，其操作方式和 `update` 类似
*   `quit`：退出 `git add -i` 命令系统
*   `help`：查看帮助信息


`git add -p`
直接进入交互命令中最有用的 `patch` 模式

这是交互命令中最有用的模式，其操作方式和 `update` 类似，选择后 `Git` 会显示这些文件的当前内容与本地版本库中的差异，然后您可以自己决定是否添加这些修改到暂存区，在命令行 `Stage deletion [y,n,q,a,d,/,?]?` 后输入 `y,n,q,a,d,/,?` 其中一项选择操作方式，具体功能解释如下：
*   y：接受修改
*   n：忽略修改
*   q：退出当前命令
*   a：添加修改
*   d：放弃修改
*   /：通过正则表达式匹配修改内容
*   ?：查看帮助信息


`git add -u`
直接进入交互命令中的 `update` 模式

它会先列出工作区 `修改` 或 `删除` 的文件列表，`新增` 的文件不会被显示，在命令行 `Update>>` 后输入相应的列表序列号表示选中该项，回车继续选择，如果已选好，直接回车回到命令主界面

`git add --ignore-removal .`
添加工作区 `修改` 或 `新增` 的文件列表， `删除` 的文件不会被添加

#### git commit
把暂存区的文件提交到本地版本库

`git commit -m '第一行提交原因'  -m '第二行提交原因'`
不打开编辑器，直接在命令行中输入多行提交原因

`git commit -am '提交原因'`
将工作区 `修改` 或 `删除` 的文件提交到本地版本库， `新增` 的文件不会被提交


`git commit --amend -m '提交原因'`
修改最新一条提交记录的提交原因


`git commit -C HEAD`
将当前文件改动提交到 `HEAD` 或当前分支的历史ID

#### git mv
移动或重命名文件、目录

`git mv a.md b.md -f`
将 `a.md` 重命名为 `b.md` ，同时添加变动到暂存区，加 `-f` 参数可以强制重命名，相比用 `mv a.md b.md` 命令省去了 `git add` 操作

#### git rm
从工作区和暂存区移除文件

`git rm b.md`
从工作区和暂存区移除文件 `b.md` ，同时添加变动到暂存区，相比用 `rm b.md` 命令省去了 `git add` 操作


`git rm src/ -r`
允许从工作区和暂存区移除目录

#### git status
`git status -s`
以简短方式查看工作区和暂存区文件状态

`git status --ignored`
查看工作区和暂存区文件状态，包括被忽略的文件

### **操作分支**

#### git branch

查看、创建、删除分支

- `git branch -a`
查看本地版本库和远程版本库上的分支列表

- `git branch -r`
查看远程版本库上的分支列表，加上 `-d` 参数可以删除远程版本库上的分支

- `git branch -D`
分支未提交到本地版本库前强制删除分支

- `git branch -vv`
查看带有最后提交id、最近提交原因等信息的本地版本库分支列表 


#### git merge 将其它分支合并到当前分支

`git merge --squash`
将待合并分支上的 `commit` 合并成一个新的 `commit`放入当前分支，适用于待合并分支的提交记录不需要保留的情况 

![合并分支](Git&Github%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E.resource/%E5%90%88%E5%B9%B6%E5%88%86%E6%94%AF.gif)

- `git merge --no-ff `
默认情况下，`Git` 执行"`快进式合并`"（fast-farward merge），会直接将 `Master` 分支指向 `Develop` 分支，使用 `--no-ff` 参数后，会执行正常合并，在 `Master`分支上生成一个新节点，保证版本演进更清晰。 

![分支合并](Git&Github%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E.resource/%E5%88%86%E6%94%AF%E5%90%88%E5%B9%B6.png)

- `git merge --no-edit`
在没有冲突的情况下合并，不想手动编辑提交原因，而是用 `Git` 自动生成的类似 `Merge branch 'test'`的文字直接提交

#### git checkout 切换分支

- `git checkout -b daily/0.0.1`
创建 `daily/0.0.1` 分支，同时切换到这个新创建的分支

- `git checkout HEAD demo.html`
从本地版本库的 `HEAD`（也可以是提交ID、分支名、Tag名） 历史中检出 `demo.html` 覆盖当前工作区的文件，如果省略 `HEAD` 则是从暂存区检出

- `git checkout --orphan new_branch`
这个命令会创建一个全新的，完全没有历史记录的新分支，但当前源分支上所有的最新文件都还在，真是强迫症患者的福音，但这个新分支必须做一次 `git commit` 操作后才会真正成为一个新分支。

- `git checkout -p other_branch`
这个命令主要用来比较两个分支间的差异内容，并提供交互式的界面来选择进一步的操作，这个命令不仅可以比较两个分支间的差异，还可以比较单个文件的差异。

#### git stash

在 `Git` 的栈中保存当前修改或删除的工作进度，当你在一个分支里做某项功能开发时，接到通知把昨天已经测试完没问题的代码发布到线上，但这时你已经在这个分支里加入了其它未提交的代码，这个时候就可以把这些未提交的代码存到栈里。

- `git stash`
将未提交的文件保存到Git栈中

* `git stash list`
查看栈中保存的列表

* `git stash show stash@{0}`
显示栈中其中一条记录

* `git stash drop stash@{0}`
移除栈中其中一条记录

* `git stash pop`
从Git栈中检出最新保存的一条记录，并将它从栈中移除

- `git stash apply stash@{0}`
从Git栈中检出其中一条记录，但不从栈中移除

* `git stash branch new_banch`
把当前栈中最近一次记录检出并创建一个新分支

* `git stash clear`
清空栈里的所有记录

* `git stash create`
 为当前修改或删除的文件创建一个自定义的栈并返回一个ID，此时并未真正存储到栈里

* `git stash store xxxxxx`
将 `create` 方法里返回的ID放到 `store` 后面，此时在栈里真正创建了一个记录，但当前修改或删除的文件并未从工作区移除

 

### 操作历史

#### git log 显示提交历史记录

* `git log -p`
显示带提交差异对比的历史记录

* `git log demo.html`
显示 `demo.html` 文件的历史记录

* `git log --since="2 weeks ago"`
显示2周前开始到现在的历史记录，其它时间可以类推

* `git log --before="2 weeks ago"`
显示截止到2周前的历史记录，其它时间可以类推

* `git log -10`
显示最近10条历史记录

* `git log f5f630a..HEAD`
显示从提交ID `f5f630a` 到 `HEAD` 之间的记录，`HEAD` 可以为空或其它提交ID

* `git log --pretty=oneline`
在一行中输出简短的历史记录

* `git log --pretty=format:"%h" `
格式化输出历史记录

* `Git` 用各种 `placeholder` 来决定各种显示内容，我挑几个常用的显示如下：
    *   %H: commit hash
    *   %h: 缩短的commit hash
    *   %T: tree hash
    *   %t: 缩短的 tree hash
    *   %P: parent hashes
    *   %p: 缩短的 parent hashes
    *   %an: 作者名字
    *   %aN: mailmap的作者名
    *   %ae: 作者邮箱
    *   %ad: 日期 (--date= 制定的格式)
    *   %ar: 日期, 相对格式(1 day ago)
    *   %cn: 提交者名字
    *   %ce: 提交者 email
    *   %cd: 提交日期 (--date= 制定的格式)
    *   %cr: 提交日期, 相对格式(1 day ago)
    *   %d: ref名称
    *   %s: commit信息标题
    *   %b: commit信息内容
    *   %n: 换行

#### git cherry-pick 合并分支的一条或几条提交记录到当前分支末梢

- `git cherry-pick 170a305`
合并提交ID `170a305` 到当前分支末梢

#### git reset

将当前的分支重设（reset）到指定的 `<commit>` 或者 `HEAD`

* `git reset --mixed <commit>`
`--mixed` 是不带参数时的默认参数，它退回到某个版本，保留文件内容，回退提交历史

* `git reset --soft <commit>`
暂存区和工作区中的内容不作任何改变，仅仅把 `HEAD` 指向 `<commit>`

* `git reset --hard <commit>`
自从 `<commit>` 以来在工作区中的任何改变都被丢弃，并把 `HEAD` 指向 `<commit>`

#### git rebase 重新定义分支的版本库状态

`git rebase branch_name`
合并分支，这跟 `merge`很像，但还是有本质区别，看下图： 

![合并分支过程](Git&Github%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E.resource/%E5%90%88%E5%B9%B6%E5%88%86%E6%94%AF%E8%BF%87%E7%A8%8B.png)

合并过程中可能需要先解决冲突，然后执行 `git rebase --continue`

`git rebase -i HEAD~~`
打开文本编辑器，将看到从 `HEAD` 到 `HEAD~~` 的提交如下
```git
pick 9a54fd4 添加commit的说明
pick 0d4a808 添加pull的说明
# Rebase 326fc9f..0d4a808 onto d286baa
#
# Commands:
#  p, pick = use commit
#  r, reword = use commit, but edit the commit message
#  e, edit = use commit, but stop for amending
#  s, squash = use commit, but meld into previous commit
#  f, fixup = like "squash", but discard this commit's log message
#  x, exec = run command (the rest of the line) using shell
#
```
将第一行的 `pick` 改成 `Commands` 中所列出来的命令，然后保存并退出，所对应的修改将会生效。

如果移动提交记录的顺序，将改变历史记录中的排序。

#### git revert

撤销某次操作，此次操作之前和之后的 `commit` 和 `history` 都会保留，并且把这次撤销作为一次最新的提交

`git revert HEAD`
撤销前一次提交操作

`git revert HEAD --no-edit`
撤销前一次提交操作，并以默认的 `Revert "xxx"` 为提交原因

`git revert -n HEAD`
需要撤销多次操作的时候加 `-n` 参数，这样不会每次撤销操作都提交，而是等所有撤销都完成后一起提交

#### git diff

查看工作区、暂存区、本地版本库之间的文件差异，用一张图来解释
![git各个区域文件差异](Git&Github%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E.resource/git%E5%90%84%E4%B8%AA%E5%8C%BA%E5%9F%9F%E6%96%87%E4%BB%B6%E5%B7%AE%E5%BC%82.png)

`git diff --stat`
通过 `--stat` 参数可以查看变更统计数据
```git
 test.md | 1 -
 1 file changed, 1 deletion(-)
```


#### git reflog

`reflog` 可以查看所有分支的所有操作记录（包括commit和reset的操作、已经被删除的commit记录，跟 `git log`的区别在于它不能查看已经删除了的commit记录 

![640](Git&Github%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E.resource/640.png)

### 远程版本库连接

如果在GitHub项目初始化之前，文件已经存在于本地目录中，那可以在本地初始化本地版本库，再将本地版本库跟远程版本库连接起来

**git init**在本地目录内部会生成.git文件夹

**git remote**

 `git remote -v`
不带参数，列出已经存在的远程分支，加上 `-v` 列出详细信息，在每一个名字后面列出其远程url


`git remote add origin https://github.com/gafish/gafish.github.com.git`
添加一个新的远程仓库，指定一个名字，以便引用后面带的URL

**git fetch**
将远程版本库的更新取回到本地版本库

`git fetch origin daily/0.0.1`
默认情况下，`git fetch` 取回所有分支的更新。如果只想取回特定分支的更新，可以指定分支名。

### **问题排查**

**git blame**查看文件每行代码块的历史信息

`git blame -L 1,10 demo.html`
截取 `demo.html` 文件1-10行历史信息

**git bisect**二分查找历史记录，排查BUG

- `git bisect start`
开始二分查找

- `git bisect bad`
标记当前二分提交ID为有问题的点

- `git bisect good`
标记当前二分提交ID为没问题的点

- `git bisect reset`
查到有问题的提交ID后回到原分支

### **更多操作**

**git submodule**
通过 Git 子模块可以跟踪外部版本库，它允许在某一版本库中再存储另一版本库，并且能够保持2个版本库完全独立

`git submodule add https://github.com/gafish/demo.git demo`
将 `demo` 仓库添加为子模块

`git submodule update demo`
更新子模块 `demo`

**git gc**
运行Git的垃圾回收功能，清理冗余的历史快照

**git archive**
将加了tag的某个版本打包提取

`git archive -v --format=zip v0.1 > v0.1.zip`
`--format` 表示打包的格式，如 `zip`，`-v` 表示对应的tag名，后面跟的是tag名，如 `v0.1`。




## 平常使用步骤
如果 Github 上已经有仓库，而且不是本地传上去的；
1.首先将 github 上文件 clone 下来，如果和本地文件夹名字一样则修改本地名字，防止覆盖；
2.将 clone 下来的文件夹里面内容，连同.git 文件一起移动，同时将本地文件夹的名字改成与仓库名字相同；就是保证一点：本地文件只能比 github 端的文件多，不能少；
3.查看目前的 git 用户：`git remote -v`,可以将目前的一些用户名删掉：`git remote remove 名字`，然后新建用户：`git remote add 名字 库url`, 这里的名字最好使用库名，便于记忆和区分；
4.添加新文件：`git add .`或者`git add 文件名`
5.然后 ：`git commit -m "说明"`
6.最后 push：`git push 名字`



# 使用 GitHub 搭建个人网站
使用 GitHub 搭建的网站是静态网站，搭建的操作分为两类：
- 完全基于新的仓库搭建：
  - 首先创建新的仓库，仓库名为：用户名.github.io
  - 然后创建 index.html 即可；
  - 相关的网页显示配置可以在 index.html 中进行配置；

- 基于已有的仓库进行搭建：(现在已经改变，在摸索着)
  - 首先进入仓库，在右上角选择：setting->Launch automatic gener....
  - 在 Tagline 中填写项目名称；


# 附录：


![github开源协议区分](Git&Github%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E.resource/github%E5%BC%80%E6%BA%90%E5%8D%8F%E8%AE%AE%E5%8C%BA%E5%88%86.png)

