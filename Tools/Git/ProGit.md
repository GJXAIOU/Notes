

### 初始运行 Git 前的配置

每台计算机上只需要配置一次，程序升级时会保留配置信息。 你可以在任何时候再次通过运行命令来修改它们。

Git 自带一个 git config 的工具来帮助设置控制 Git 外观和行为的配置变量。 这些变量存储在三个不同的位
置：

- /etc/gitconfig 文件: 包含系统上每一个用户及他们仓库的通用配置（系统配置文件，需要管理员权限）。 **如果在执行 git config 时带上 `--system` 选项，那么它就会读写该文件中的配置变量**。 

- ~/.gitconfig 或 ~/.config/git/config 文件：只针对**当前用户**。 你可以传递 --global 选项让 Git
    读写此文件，这会对你系统上 **所有** 的仓库生效。

- 当前使用仓库的 Git 目录中的 config 文件（即 .git/config）：针对该仓库。 你可以传递 --local 选
    项让 Git 强制读写此文件，虽然默认情况下用的就是它。。 （当然，你需要进入某个 Git 仓库中才能让该选
    项生效。）
    每一个级别会覆盖上一级别的配置，所以 .git/config 的配置变量会覆盖 /etc/gitconfig 中的配置变量。
    在 Windows 系统中，Git 会查找 $HOME 目录下（一般情况下是 `C:\Users\$USER` ）的 .gitconfig 文件。
    Git 同样也会寻找 /etc/gitconfig 文件，但只限于 MSys 的根目录下，即安装 Git 时所选的目标位置。 如果
    你在 Windows 上使用 Git 2.x 以后的版本，那么还有一个系统级的配置文件，Windows XP 上在
    C:\Documents and Settings\All Users\Application Data\Git\config ，Windows Vista 及更新
    的版本在 C:\ProgramData\Git\config 。此文件只能以管理员权限通过 git config -f <file> 来修
    改。
    查看所有的配置以及它们所在的文件命令：`git config --list --show-origin`

    ```shell
    C:\Users\gjx16>git config --list --show-origin
    file:E:/Program/VersionControl/Git/etc/gitconfig        core.fscache=true
    .....
    file:C:/Users/gjx16/.gitconfig  user.name=GJXAIOU
    file:C:/Users/gjx16/.gitconfig  user.email=gjx1690048420@163.com
    ......
    file:.git/config        core.repositoryformatversion=0
     remote.leetcode.url=https://github.com/GJXAIOU/LeetCode.git
    file:.git/config        remote.leetcode.fetch=+refs/heads/*:refs/remotes/leetcode/*
    file:.git/config        remote.interviewexperience.url=https://github.com/GJXAIOU/InterviewExperience.git
    file:.git/config        remote.interviewexperience.fetch=+refs/heads/*:refs/remotes/interviewexperience/*
    ....
    ```

#### 设置用户信息

设置用户名和邮件地址，每一个 Git 提交都会使用这些信息，它们会写入到你的每一次提交中，不可更改：

```shell
git config --global user.name "GJXAIOU"
git config --global user.email gjxaiou@163.com
```

如果使用了 --global 选项，那么该命令只需要运行一次，因为之后无论你在该系统上做任何事情， Git 都会使用那些信息。 当你想针对特定项目使用不同的用户名称与邮件地址时，可以在那个项目目录下运
行没有 --global 选项的命令来配置。

#### 设置文本编辑器

配置默认文本编辑器了，当 Git 需要你输入信息时会调用它。 如果未配置，Git 会使用操作系统默认的文本编辑器。

`git config --global core.editor "完整的安装路径"`

#### 检查配置信息

如果想要检查你的配置，可以使用 git config --list 命令来列出所有 Git 当时能找到的配置。

你可能会看到重复的变量名，因为 Git 会从不同的文件中读取同一个配置（例如：/etc/gitconfig 与
~/.gitconfig）。 这种情况下，Git 会使用它找到的每一个变量的最后一个配置。如果想看是哪个文件修改了该值可以使用  `git config --show-origin <key>` 

```shell
$ git config --show-origin user.name
file:C:/Users/gjx16/.gitconfig  GJXAIOU
```



可以通过输入 git config <key>： 来检查 Git 的某一项配置

#### 获取帮助

`命令  help` 或者 `命令 -h`， 如 `git config -h`

### 二、Git 基础

#### 获取 Git 仓库

通常有两种获取 Git 项目仓库的方式：
- 将尚未进行版本控制的本地目录转换为 Git 仓库；（在已存在目录中初始化仓库）

    进入项目目录，然后执行 `git init`

    该命令将创建一个名为 .git 的子目录，这个子目录含有你初始化的 Git 仓库中所有的必须文件，这些文件是Git 仓库的骨干。 但是，在这个时候，我们仅仅是做了一个初始化的操作，你的项目里的文件还没有被跟踪。(参见 Git 内部原理 来了解更多关于到底 .git 文件夹中包含了哪些文件的信息。)

    如果在一个已存在文件的文件夹（而非空文件夹）中进行版本控制，你应该开始追踪这些文件并进行初始提交。
    可以通过 git add 命令来指定所需的文件来进行追踪，然后执行 git commit ：

    现在，你已经得到了一个存在被追踪文件与初始提交的 Git 仓库。

- 从其它服务器 克隆 一个已存在的 Git 仓库。

    使用 `git clone  <url>`命令，或者 `git clone <url> 新的名字` 设置保存在本地项目名称，Git 克隆的是该 Git 仓库服务器上的几乎**所有数据**，而不是仅仅复制完成你的工作所需要文件。 当你执行 git clone 命令的时候，默认配置下远程 Git 仓库中的**每一个文件的每一个版本**都将被拉取下来。

    这会在当前目录下创建一个名为 “libgit2” 的目录，并在这个目录下初始化一个 .git 文件夹， 从远程仓库拉取下所有数据放入 .git 文件夹，然后从中读取最新版本的文件的拷贝。 如果你进入到这个新建的 libgit2 文件夹，你会发现所有的项目文件已经在里面了，准备就绪等待后续的开发和使用。

    Git 支持多种数据传输协议。 上面的例子使用的是 https:// 协议，不过你也可以使用 git:// 协议或者使用
    SSH 传输协议，比如 user@server:path/to/repo.git 。

    两种方式都会在你的本地机器上得到一个工作就绪的 Git 仓库。

#### 记录每次更新到仓库

工作目录下的每一个文件只有两种状态：已跟踪 或 未跟踪。 已跟踪的文件是指那些被纳入了版本控制的文件，在上一次快照中有它们的记录，在工作一段时间后， 它们的状态可能是未修改，已修改或已放入暂存区。简而言之，已跟踪的文件就是 Git 已经知道的文件。

工作目录中除已跟踪文件外的其它所有文件都属于未跟踪文件，它们既不存在于上次快照的记录中，也没有被放入暂存区。 初次克隆某个仓库的时候，工作目录中的所有文件都属于已跟踪文件，并处于未修改状态，因为 Git 刚刚检出了它们， 而你尚未编辑过它们。

编辑过某些文件之后，由于自上次提交后你对它们做了修改，Git 将它们标记为已修改文件。 在工作时，你可以择性地将这些修改过的文件放入暂存区，然后提交所有已暂存的修改，如此反复。

![image-20210310151249594](Untitled.resource/image-20210310151249594.png)

#### 检查当前文件状态

使用命令 `git status` 会显示当前所在分支，「Untracked files」下面列出的文件就是未跟踪文件，即 Git 之前的快照（提交）中没有这些文件。「Changes to be committed」 下面表示为已暂存文件，如果此时提交，那么该文件在**你运行 git add 时的版本**将被留存在后续的历史记录中。

#### 跟踪新文件

使用 `git add 文件名称` 开始跟踪该文件，如果将文件名称改为文件路径，会递归跟踪该目录下的所有文件。

#### 暂存已修改文件

修改一个已被跟踪的文件，使用 `git status` 显示该文件处于 「Changes not staged for commit」 下面，即已追踪文件发生修改但是没有放到暂存区。如果要暂存该次更新，使用 git add。该命令可以用它开
始跟踪新文件，或者把已跟踪的文件放到暂存区，还能用于合并时把有冲突的文件标记为已解决状态等，总的来说就是：精确的将内容添加到下一次提交中。

**注意** ：提交的时候，提交文件的版本是最后一次运行 `git add` 时的版本，而不是运行 `git commit` 时当前工作目录中的当前版本，所以运行 `git add` 之后又修改了该文件，需要重新运行 `git add` 将最新版本文件重新暂存起来才行。

可以使用 `git status -s` 或者使用 `git status --short` 来简化输出结果。

新添加的未跟踪文件前面有 ?? 标记，新添加到暂存区中的文件前面有 A 标记，修改过的文件前面有 M 标记。 输出中有两栏，左栏指明了暂存区的状态，右栏指明了工作区的状态。

```shell
$ git status -s
M README
MM Rakefile
A lib/git.rb
M lib/simplegit.rb
?? LICENSE.txt
上面的状态报告显示： README 文件
在工作区已修改但尚未暂存，而 lib/simplegit.rb 文件已修改且已暂存。 Rakefile 文件已修，暂存后又
作了修改，因此该文件的修改中既有已暂存的部分，又有未暂存的部分。
```

#### 忽略文件

无需纳入 Git 管理并且不希望其出现在未跟踪文件列表中。

通过创建一个  `.gitignore` 文件，格式规划如下：

• 所有空行或者以 # 开头的行都会被 Git 忽略。
• 可以使用标准的 glob 模式匹配，它会递归地应用在整个工作区中。
• 匹配模式可以以（/）开头防止递归。
• 匹配模式可以以（/）结尾指定目录。
• 要忽略指定模式以外的文件或目录，可以在模式前加上叹号（!）取反。
所谓的 glob 模式是指 shell 所使用的简化了的正则表达式。 星号（*）匹配零个或多个任意字符；[abc] 匹配
任何一个列在方括号中的字符 （这个例子要么匹配一个 a，要么匹配一个 b，要么匹配一个 c）； 问号（?）只匹配一个任意字符；如果在方括号中使用短划线分隔两个字符， 表示所有在这两个字符范围内的都可以匹配（比如 [0-9] 表示匹配所有 0 到 9 的数字）。 使用两个星号（**）表示匹配任意中间目录，比如 a/**/z 可以匹配 a/z 、 a/b/z 或 a/b/c/z 等。

```shell
# 忽略所有的 .a 文件
*.a
# 但跟踪所有的 lib.a，即便你在前面忽略了 .a 文件
!lib.a
# 只忽略当前目录下的 TODO 文件，而不忽略 subdir/TODO
/TODO
# 忽略任何目录下名为 build 的文件夹
build/
# 忽略 doc/notes.txt，但不忽略 doc/server/arch.txt
doc/*.txt
# 忽略 doc/ 目录及其所有子目录下的 .pdf 文件
doc/**/*.pdf
```

各种语言推荐的 `.gitignore` 文件写法见 [Github](https://github.com/github/gitignore)。

#### 查看已暂存和未暂存的修改

git diff 命令用来解决这两个问题：当前做的哪些更新尚未暂存？ 有哪些更新已暂存并准备好下次提交？ 虽然 git status 已经通过在相应栏下列出文件名的方式回答了这个问题，但 git diff 能通过文件补丁的格式更加具体地**显示哪些行发生了改变**。

分别修改两个文件，一个使用`git add`暂存，一个不暂存。使用 status 查看就是一个在 「Changes to be committed」，一个在 「Changes not staged for commit」。

使用 `git diff` 比较工作目录中的当前文件和暂存区域快照之间的差异，即修改后还未暂存的变化内容。若要查看已暂存的将要添加到下次提交里的内容，可以用 git diff --staged 或者 `git diff --cached`命令。 这条命令将比对已暂存文件与最后一次提交的文件差异，**git diff 本身只显示尚未暂存的改动，而不是自上次提交以来所做的所有改动。**所以暂存之后在用就看不到差异了。

#### 提交更新

在每次提交之前使用 `git status` 查看是不是需要的文件都已经暂存了，然后在使用 `git commit`进行提交，这里会启动选择的文本编辑器进行提交说明。

提交之后会显示分支和本次提交的完整 SHA-1 完整校验和，有多少文件修改过，多少行添加和删除。

















