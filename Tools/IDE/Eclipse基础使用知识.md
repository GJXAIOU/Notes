# 基础使用知识

视频地址：[https://www.bilibili.com/video/av24042956/?p=12](https://www.bilibili.com/video/av24042956/?p=12)

其中第六讲：6.1和6.2 关于远程调试没有观看
第八讲：8.1和8.2关于JSP开发环境搭建没有看
以及第九讲没有看

## 一、基础知识

- 新建java Project
- jre是java库
- 新建文件：sourceFolder：用于存放java源代码，一般命名为src;通常在新建工程的时候eclipse会自动建好
  - Folder ：一般存放不需要编译的文件，例如第三方jar包以及其他文件；
- `run as` 运行：选择：java application ：使用java工程运行
- `debug as` 调试： 

### 第三方包的使用

`Project - properties - Java Build Path - Libraies`
右边的选项：
- Add JARs ：当已经将第三方包黏贴到当前工程文件夹下面的时候，使用这个将该包导入；
- Add External JARs ：直接链接到第三方包存放的位置进行导入；


### 工程的自动构建
Eclipse中工程默认保存之后就会自动的进行构建；
可以使用：`Project  - Build Automatically `， 将前面的选项勾选掉就不会自动的进行构建

使用：`Project - clean` 就会去掉之前所有的构建形成的`.class`文件，并且进行重新构建



## 二、Java调试


### （一）设置断点
 - 默认情况：在需要设置代码的该行最左边`双击`或者右击选择：`Toggle Breakpoint` ，或者使用快捷键：`ctrl + shift + B`；通过：`disable Breakpoint`可以暂停断点作用。但是不是取消断点
- 设置条件：通过设置条件，让系统决定遇到条件是跳过还是执行断点：
   方法：设置断点之后，在断点处右击，选择`Properties`,然后勾选：`conditional`,在下面填写条件即可；

设置断点之后，选择debug进行逐步调试，然后将鼠标放在变量上，就会显示该变量现在的值，或者选择该变量，右击选择：`watch`也可以显示，同时一步一步调试，选择：`resume`逐步调试。
如果想看其他的变量，选择watch之后，在窗口：Expression中右击，选择Add Watch Expression ,然后输入想要查看的变量名即可


### （二）智能纠错
 在错误处，右击，根据系统的提示进行纠错


### （三）自动生成

- 自动生成set，get方法：`source - generate Getter and Setter`,选择需要生成的即可
- 自动生成构造函数： `source - generate Constructor Using Field`
- 自动生成Javadoc ; 输入：`/**`然后回车即可




## 三、重构

### （一）将现有代码段抽取为方法
选取需要抽取的代码段，右击：`refactor - Extract Mothod`

### (二)重命名
选择需要重命名的类名、变量名或者文件名等，右击：`refactor - Rename`


### （三）内联
将很短的方法直接内联到调用该方法的地方，减少调用；
选择需要内联的方法，右击：`refactor - lnline`


### (四)常量抽取
将需要抽取为常量的字符串等选择，右击：`refactor - Extract Constant`


### (五)抽取为局部变量
当类对象实例化之后，当多次使用调用方法或者属性时候，可以将这种调用设置成为局部变量。选择需要设置的调用，右击：`refactor - Extract local variable`

### (六)包装字段
例如将类中的public修饰成员变量设置为private，同时生成set、get方法
选中需要设置的字段，然后右击：`refactor - Encapsulate field`

### (七)抽取为接口
选取代码段，右击：`refactor - Extract interface`




## 四、常用技巧

### （一）代码自动补齐
一般情况下使用`.`就会自动提示补齐，如果不提示，可以使用：`Alt + /`


### (二)代码格式化
首先选择需要格式化的代码块，然后按：`ctrl + shift + F`，会使用Eclipse默认代码格式进行格式化；
当然也可以将默认格式改为自己的格式：`Windows - Preference - java - Code Style - Formattor - New`



### (三)查找
- 找类：当文件中类很多的时候，用于查找：`Navirate - OpenType -输入要查找的类名`
- 查看父类的实现：如果子类与父类有同名方法，先看一个父类对于该方法的实现；此时，子类实现方法最左边有一个向上的三角箭头，点一下就会跳转到父类的实现方法；
- 查看父类有多少子类： 在父类名上，右击选择：`Open Type Hierarchy`
- 查看该方法在哪些地方被调用：在方法名上右击选择：`Open call Hierarchy`
- 在源代码中查找部分字段：`search - file - 在containing text 中输入内容 - 在file name pattern 中输入 *.java 即可，这里是支持正则表达式的`；以上搜索是在整个文件夹中搜索，包括库中代码，如果只想在自己写的源代码中搜，可以设置工作集，可以减少搜索花费的时间：`search - file - working set - choose - new - java - 选择文件即可（src）`



## 五、第三方插件安装

插件安装方式共有三种：在线安装；压缩包解压安装、links安装

- 在线安装：
  `help - software update - find and install - search for new feacture to install - new Remote site  - 输入名称（随便，好记）,和网址即可`

- 压缩包解压安装：
  首先下载压缩包并且解压，然后将解压后的`feacture`和`plugins`文件夹内的文件，复制黏贴到eclipse安装路径下面的同名文件夹中即可；
  然后重启Eclipse,如果没有加载，将eclipse安装路径下的`configuration`文件夹下的`com.eclipse.update`文件夹删除即可，然后再重启eclipse

- links安装方式：
    自己新建文件夹存放第三方插件，然后再该文件夹下最后一层新建文件夹：`eclipse`，例如：`3rd\JWt\eclipse`，然后将第三方插件解压的文件夹：`feacture`和`plugins`复制到该文件夹下；
    同时在eclipse安装目录之下，建立：`links`文件夹，下面新建文件：`第三方插件名（和上面文件名一样.link）`,文件中写入第三方插件路径，例如：`c:/XXX/JWt`,写到之前路径中eclipse前一个即可；最后重启eclipse

![插件安装]($resource/%E6%8F%92%E4%BB%B6%E5%AE%89%E8%A3%85.jpg)











