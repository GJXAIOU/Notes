解决方法：通过注册表修改Tomcat命令窗口的默认字符编码为UTF-8即可解决

第一步：Windows+R打开运行，输入regedit进入注册表编辑器

第二步：在HKEY_CURRENT_USER→Console→Tomcat中修改CodePage为十进制的65001

注意：如果没有Tomcat或者CodePage，直接新建一个，如下图所示

![](https://img-blog.csdnimg.cn/20190120194941772.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTcxMjA1OQ==,size_16,color_FFFFFF,t_70)

注：点击Console新建Tomcat，点击Tomcat，新建，选择DWPRD（32-位），然后在右边选中刚新建，重命名为CodePage，然再次选中，按下回车键，即可转换十进制编辑