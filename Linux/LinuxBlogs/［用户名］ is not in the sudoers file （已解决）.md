# "［用户名］ is not in the sudoers file"（已解决）

2017-10-14 10:59:10 [小白狐狸](https://me.csdn.net/github_38236333) 阅读数 3670更多

分类专栏： [踩坑进行时之服务器](https://blog.csdn.net/github_38236333/article/category/6883757)

[](http://creativecommons.org/licenses/by-sa/4.0/)版权声明：本文为博主原创文章，遵循[ CC 4.0 BY-SA ](http://creativecommons.org/licenses/by-sa/4.0/)版权协议，转载请附上原文出处链接和本声明。

本文链接：[https://blog.csdn.net/github_38236333/article/details/78232737](https://blog.csdn.net/github_38236333/article/details/78232737) 

解决方法：
1.切换到root用户

```
su －
```

2.添加文件读写权限

```
chmod u+w /etc/sudoers
```

3.打开文件

```
vim /etc/sudoers
```

注：如果没有vim，请通过`apt-get install vim`安装。

4.找到`root ALL=(ALL) ALL`并在此行下方添加：

```
[用户名] ALL=(ALL) ALL
```

5.删除文件读写命令

```
chmod u-w /etc/sudoers
```