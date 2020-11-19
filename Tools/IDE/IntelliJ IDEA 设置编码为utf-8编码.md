# IntelliJ IDEA 设置编码为utf-8编码


**问题一：**

File->Settings->Editor->File Encodings

![](https://img-blog.csdn.net/20180608220601158?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM4MTMyMzYx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**问题二：**

File->Other Settings->Default Settings ->Editor->File Encodings

#### ![](https://img-blog.csdn.net/20180608212648780)

**问题三：**

将项目中的**.idea文件夹**中的**encodings.xml**文件中的编码格式改为uft-8

**问题四：**

File->Settings->Build,Execution,Deployment -> Compiler -> Java Compiler

设置 **Additional command line parameters**选项为 **-encoding utf-8**

![](https://img-blog.csdn.net/20180608215004536)

**问题五：**

1)打开Run/Debug Configuration,选择你的tomcat

![](https://img-blog.csdn.net/20180608220138855?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM4MTMyMzYx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

2) 然后在  Server > VM options 设置为 -Dfile.encoding=UTF-8 ，重启tomcat

![](https://img-blog.csdn.net/20180608220247543?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM4MTMyMzYx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**问题六：**

清空浏览器缓存再试一次。