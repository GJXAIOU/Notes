##  IDEA 配置



来源：blog.csdn.net/fly910905/article/details/77868300

* * *

**1.设置maven**

 `1.在File->settings->搜索maven`
 
 `2.Mavan home directory--设置maven安装包的bin文件夹所在的位置`
 
 `3.User settings file--设置setting文件所在的位置`
 
 `4.Local repository--设置本地仓库`

**2.IDEA 设置代码行宽度**

 `1.在File->settings->Editor->Code Style`
 
 `2.有人会问，如果输入的代码超出宽度界线时，如何让IDE自动将代码换行？``有两种方式！`
 
 `3.第一种，在上述的“Right margin (columns)”的下方，有“Wrap when typing reaches right margin”选项，选中它，是什么效果呢？`
 
 `4.随着输入的字符的增加，当代码宽度到达界线时，IDEA会自动将代码换行。`
 
 `5.第一种方式是在输入代码时触发，还有第二种方式，在File->settings->Code Style->Java中，选中“Wrapping and Braces”选项卡，`
 
 `6.在“Keep when reformatting”中有一个“Ensure rigth margin is not exceeded”，选中它，是什么效果呢？`
 
 `7.从配置项的字面意思很容易理解，在格式化Java代码时，确保代码没有超过宽度界线。`
 
 `8.即输入的代码超出界线后，`

**3.IDEA 提示不区分大小写**

 `1.首先打开File----->setting`
 
 `2.然后，输入：``sensitive`
 
 `3.将右侧的 case sensitive completion 修改为NONE`

# **4.IntelliJ强制更新Maven Dependencies**

 `1.Intellj 自动载入Mave依赖的功能很好用，但有时候会碰到问题，导致pom文2.件修改却没有触发自动重新载入的动作，此时需要手动强制更新依赖。`
 
 `如下：`
 
 `1.手动删除Project Settings里面的Libraries内容；`
 
 `2.在Maven Project的试图里clean一下，删除之前编译过的文件；`
 
 `3.项目右键-》Maven-》Reimport`
 
 `4.Ok， 此时发现依赖已经建立！`

**5.idea的环境配置默认保存位置**

 `1.idea的环境配置默认保存位置:C:\Users\xxxxxxxxx\.IntelliJIdea14 ,xxxxxx代表用户目录,`
 
 `2.可以对该目录进行备份,一但环境出问题恢复此配置即可.`
 
 `3.可以在%IDEA_HOME%/bin/idea.properties中修改该配置路径.`

**6.隐藏不想看到的文件或者文件夹（类似eclipse的filter功能）**

 `intellij idea 隐藏不想看到的文件或者文件夹（类似eclipse的filter功能）`
 
 `打开intellij -->:>File -->>Settings-->>搜索File Type`

**7.修改为Eclipse快捷键**

 `File -> Settings -> Keymap => Keymaps改为 Eclipse copy`

**8.修改默认设置--default setting**

 `修改默认设置--default setting`

**9.修改智能提示快捷键**

 `1.File -> Settings -> Keymap -> Main menu -> Code -> Completion -> Basic=>修改为Ctrl+Alt+Enter  `
 
 `2.保存时把冲突的Remove掉。`
 
 `3.File -> Settings -> Keymap -> Editor Actions -> Complete Current Statement=>修改为Ctrl+`

**10.查找快捷键冲突问题处理**

 `1.File -> Settings -> Keymap -> Main menu -> Edit ->Find =>修改Find...和Replace...分别改为Ctrl+F 和Ctrl+R`

**11.显示行号**

 `1.File -> Settings ->Editor ->General -> Appearance =>Show line numbers选中`

** 12.代码智能提示，忽略大小写**

 `File -> Settings -> Editor -> Code Completion里把Case sensitive completion设置为None就可以了`

**13.用*标识编辑过的文件 **

 `1.Editor–>General –> Editor Tabs`
 
 `2.在IDEA中，你需要做以下设置, 这样被修改的文件会以*号标识出来，你可以及时保存相关的文件。`
 
 `3.“Mark modifyied tabs with asterisk`

**14.关闭自动代码提示**

 `1.Preferences => IDE Settings => Editor => Code Completion => Autopopup documentation in (ms)`

**15.常用快捷键**

 `1.Ø Top #10切来切去：``Ctrl+Tab`
 
 `2.Ø Top #9选你所想【选中上下文相关联代码】：``Ctrl+W`
 
 `3.Ø Top #8代码生成：``Template/Postfix +Tab`
 
 `4.Ø Top #7发号施令：``Ctrl+Shift+A`
 
 `5.Ø Top #6无处藏身：``Shift+Shift`
 
 `6.Ø Top #5自动完成：``Ctrl+Shift+Enter`
 
 `7.Ø Top #4创造万物：``Alt+Insert`
 
 `使用前三名！`
 
 `1.Ø Top #1智能补全：``Ctrl+Shift+Space`
 
 `2.Ø Top #1自我修复：``Alt+Enter`
 
 `3.Ø Top #1重构一切：``Ctrl+Shift+Alt+T`
 
 `其他辅助`
 
 `1.以上这些神键配上一些辅助快捷键，即可让你的双手90%以上的时间摆脱鼠2标，专注于键盘仿佛在进行钢琴表演。``这些不起眼却是至关重要的最后一块拼图有：`
 
 `2.Ø 命令：``Ctrl+Shift+A可以查找所有Intellij的命令，并且每个命令后面还有其快捷键。``所以它不仅是一大神键，也是查找学习快捷键的工具。`
 
 `3.Ø 新建：``Alt+Insert可以新建类、方法等任何东西。`
 
 `4.Ø 格式化代码：``格式化import列表Ctrl+Alt+O，格式化代码Ctrl+Alt+L。`
 
 `5.Ø 切换窗口：``Alt+Num，常用的有1-项目结构，3-搜索结果，4/5-运行调试。``Ctrl+Tab切换标签页，Ctrl+E/Ctrl+Shift+E打开最近打开过的或编辑过的文件。`
 
 `6.Ø 单元测试：``Ctrl+Alt+T创建单元测试用例。`
 
 `7.Ø 运行：``Alt+Shift+F10运行程序，Shift+F9启动调试，Ctrl+F2停止。`
 
 `8.Ø 调试：``F7/F8/F9分别对应Step into，Step over，Continue。`
 
 `此外还有些我自定义的，例如水平分屏Ctrl+|等，和一些神奇的小功能9.Ctrl+Shift+V粘贴 很早以前拷贝过的，Alt+Shift+Insert(块选)进入到列模式进行按列选中`

**16.svn 不能同步代码问题修正**

 `File -> Settings ->Subversion ->General => Use command line client 选中`
 
 `1.使用command line方式需要指定svn.exe的路径,例如:D:\tools\TortoiseSVN\bin\svn.exe`
 
 `2.注意,安装TortoiseSVN时路径中不要带空格,例如:C:\Program Files\TortoiseSVN\bin\svn.exe就会报错.`
 
 `3.安装TortoiseSVN选择全部安装组件,否则可能没有svn.exe`

**17.设置idea的SVN忽略掉*.iml文件**

 `1.Editor->File Types=>Ignore files and folders增加*.iml;`
 
 `2.在lgnore files and folesrs中输入.idea;注意要";"结尾。``你就可以隐藏.idea文件夹`

**18.改变编辑文本字体大小**

 `File -> settings -> EDITOR COLORS & FONTS -> FONT -> SIZ`

**19.IDEA编码设置**

 `1.FILE -> SETTINGS -> FILE ENCODINGS => IDE ENCODING`

 `2.FILE -> SETTINGS -> FILE ENCODINGS => Project Encoding`

`3.FILE -> SETTINGS -> FILE ENCODINGS => Default encoding for properties files`

`4.FILE -> SETTINGS -> FILE ENCODINGS => Transparent native-to-ascii conversion`

** 20.Live Templates**

 `System.out.println 快捷输出`
 
 `“abc”.sout => System.out.println("abc");`
 
 `在eclipse中使用方式为：``sysout=> System.out.println();`

 `for循环`
 
 `List<String> list = new ArrayList<String>();`
 
 `输入: list.for 即可输出`
 
 `for(String s:list){} `

**21.配置tomcat参数**

 `1.vm options: -Xms256m -Xmx512m -XX:PermSize=128m -XX:MaxPermSize=256m`

**22.idea安装插件的方法**

 `1.以IntelliJ IDEA 14.0.1安装findbugs插件为例：`
 
 `2.(1)在线方式:进入File->setting->plugins->browse repositorits 搜索你要下载的插件名称，`
 
 `3.右侧可以找到下载地址,完成后按提示重启即可.`
 
 `4.(2)离线安装: 下载findbugs插件地址：``5.http://plugins.jetbrains.com/plugin/3847,`
 
 `6.将下载的FindBugs-IDEA-0.9.994.zip,安装插件：``进入File->setting-7.>plugins=> Install plugin from disk...`
 
 `8.定位到到刚才下载的jar,点击ok,完成后按提示重启即可.`
 
 `9.插件安装的位置在C:\Users\xxxxxxxxx\.IntelliJIdea14\config\plugins\插件名下.`
 
 `10.安装iBATIS/MyBatis min-plugin插件`

**23.调整idea启动时的内存配置参数**

 `1.%IDEA_HOME%/bin/idea.exe.vmoptions`

**24.导入eclipse web项目发布到Tomcat如果找不到**

 `1.导入eclipse web项目发布到Tomcat如果找不到,可以在环境配置的Facets增加web支持,在Artifacts中增加项目部署模块`

**25.每次打开一个新jsp或java文件时,cpu都占用很高,去掉检验即可**

 `每次打开一个新jsp或java文件时,cpu都占用很高,去掉检验即可:`
 
 `file->settings->editor->inspection`

**26.idea增加spring/struts关联文件支持**

 `project Settings->Modules->选中项目右键可添加`

**27\. IDEA开启类修改后自动编译**

 `1.File->setting->Buil,Execution,Deployment->compiler=>Make project automatically`
 
 `2.编译错误问题解决`
 
 `3.Error:java: Compilation failed: internal java compiler error`
 
 `4.set中Java complier 设置的问题 ，项目中有人用jdk1.6 有人用jdk1.7 版本不一样 会一起这个错`

**28.提示实现Serializable接口**

 `1.使用 Eclipse 或 MyEclipse 的同学可能知道，如果 implements Serializable 接口时，会提示你生成 serialVersionUID。`
 
 `2.但 Intellij IDEA 默认没启用这个功能。`
 
`3.Preferences->IEditor->nspections->Serialization issues->Serializable class without ’serialVersionUID’，`

>`4.选中以上后，在你的class中：``光标定位在类名前，按 Alt+Enter 就会提示自动创建 serialVersionUID`

**29.演出模式**

 `我们可以使用【Presentation Mode】，将IDEA弄到最大，可以让你只关注一个类里面的代码，进行毫无干扰的coding。`
 
 `可以使用Alt+V快捷键，谈出View视图，然后选择Enter Presentation Mode。``效果如下`

![](https://mmbiz.qpic.cn/mmbiz_png/WwPkUCFX4x4RTwr1N6WUcSFYRxqGduSgspQAxYCzSAU49BAj3tsaSl2kCeXicPU0BcA2fVm3m3IK64BKnaE39og/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

 `这个模式的好处就是，可以让你更加专注，因为你只能看到特定某个类的代码。``可能读者会问，进入这个模式后，我想看其他类的代码怎么办？``这个时候，就要考验你快捷键的熟练程度了。``你可以使用CTRL+E弹出最近使用的文件。``又或者使用CTRL+N和CTRL+SHIFT+N定位文件。`
 
 `如何退出这个模式呢？``很简单，使用ALT+V弹出view视图，然后选择Exit Presentation Mode 即可。`
 
 `但是我强烈建议你不要这么做，因为你是可以在Enter Presentation Mode模式下在IDEA里面做任何事情的。``当然前提是，你对IDEA足够熟练`

**30.神奇的Inject language**

 `如果你使用IDEA在编写JSON字符串的时候，然后要一个一个\去转义双引号的话，就实在太不应该了，又烦又容易出错。`
 
 `在IDEA可以使用Inject language帮我们自动转义双引号`

先将焦点定位到双引号里面，使用alt+enter快捷键弹出inject language视图，并选中Inject language or reference。

![](https://mmbiz.qpic.cn/mmbiz_png/WwPkUCFX4x4RTwr1N6WUcSFYRxqGduSgj7YcvEyuxCvNjApXV34nJe8IEegdhG5fLo7apGBVic6QibibqsxpWG6eQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

选择后,切记，要直接按下enter回车键，才能弹出inject language列表。在列表中选择 json组件。

![](https://mmbiz.qpic.cn/mmbiz_png/WwPkUCFX4x4RTwr1N6WUcSFYRxqGduSgMWlicgCH8kQXnoK6XCdR4YovuJG62QV1S06F6yh0s9gJL3vGO1xB2AA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

选择完后。鼠标焦点自动会定位在双引号里面，这个时候你再次使用alt+enter就可以看到

![](https://mmbiz.qpic.cn/mmbiz_png/WwPkUCFX4x4RTwr1N6WUcSFYRxqGduSgic4opmH2HcbM9lOZT2jzF4z51xnsdd2yDURqG2JQ5TYBajUTTjic6iclA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

选中Edit JSON Fragment并回车，就可以看到编辑JSON文件的视图了。

![](https://mmbiz.qpic.cn/mmbiz_png/WwPkUCFX4x4RTwr1N6WUcSFYRxqGduSgf1Wz67Odp7qU4E7PRHvAHJmRuFnGSOwDspricXWnCo8ln4b64srNzaQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

可以看到IDEA确实帮我们自动转义双引号了。如果要退出编辑JSON信息的视图，只需要使用ctrl+F4快捷键即可。

Inject language可以支持的语言和操作多到你难以想象，读者可以自行研究。

**31.强大的symbol**

 `如果你依稀记得某个方法名字几个字母，想在IDEA里面找出来，可以怎么做呢？`
 
 `直接使用ctrl+shift+alt+n，使用symbol来查找即可。`

![](https://mmbiz.qpic.cn/mmbiz_png/QCu849YTaIN8Qibq1rJDsIcoMnH8A2r6PRxRBjTC44CIfgt3pLjEZ8V3n998Dj4NOpzkGe2mnJEvjA2lTPmx2wQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**32.idea快捷键和Windows默认快捷键冲突解决（如：****Ctrl+Alt+↑或Ctrl+Alt+F12）**

 `解决方式：``在桌面右键 - 图形选项 - 快捷键 - 禁止 就可以`

如果觉得不错，转发给更多的小伙伴吧！