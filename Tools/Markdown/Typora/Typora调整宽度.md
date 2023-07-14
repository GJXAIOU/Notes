# 调整Typora显示宽度 

默认展示渲染结果的宽度太窄了。`#write{    max-width: 860px;    margin: 0 auto;    padding: 20px 30px 40px 30px;    padding-top: 20px;    padding-bottom: 100px; }`github.css这种情况在你想贴一段代码， 或者加一个很多列的表格的时候尤为明显，导致本该一行显示的代码变成了两行显示或者表格区域下方多了一个x方向的滚动条，丑。 但是你翻遍Typora的配置项也找不到任何一个地方可以修改的地方。PS.缩放字体也不行， 该丑的， 它还是丑。不过好在Typora是用electron技术栈开发的软件，意味着你可以按下`F12`显示最熟悉的chrome控制台， 审查元素之后你就会发现（以github white主题为例）， css文件中指定了最大宽度， 而且， 还是860px。虽然不太懂为什么要这么做， 但是接下来就是我们要做的事情。搜索安装目录下的github.css文件linux下： ${TYPORA_HOME}/resources/app/style/themes/github.css修改`#white`下的`max-width`为你想要的宽度， 像素值或者百分比， 建议80%.重启Typora即可。



### **[linmingwei](https://github.com/linmingwei)** commented [on 16 Apr 2020](https://github.com/jwenjian/ghiblog/issues/18#issuecomment-614372923)

Windows下：C:\Users${user_name}\AppData\Roaming\Typora\themes 新建base.user.css(所有主题都生效) 或者github.user.css(仅GitHub主题生效)，将自己的css添加进去即可



### https://github.com/jwenjian/ghiblog/issues/18#issuecomment-617606771)

Windows下：C:\Users${user_name}\AppData\Roaming\Typora\themes 新建base.user.css(所有主题都生效) 或者github.user.css(仅GitHub主题生效)，将自己的css添加进去即可还可以这样 学习了因为自己的修改不生效就去翻文档，发现了可以这样改😁有文档链接不The built-in CSS will be replaced after update / reinstall, DO NOT MODIFY THEM.Reffer https://support.typora.io/Add-Custom-CSS/ when you want to modify those CSS. Reffer https://support.typora.io/About-Themes/ if you want to create / install new themes.

