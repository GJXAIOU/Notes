# CentOS 安装 Redis

==配置主从的时候：默认就绑定了 IP，一定要去掉：bind 127.0.0.1 ==

**目录结构**
Redis
    redis-5.0.5
    redis-5.0.5-rc2.tar.gz

- 首先上传源码包`redis-5.0.5-rc2.tar.gz`
- 解压源码包：`tar -zxvf redis.*****`
- 然后进入解压之后得到的文件 redis-XXX 目录中
- `make`
如果报错： **yum -y install gcc-c++**，然后重新解压再从头执行。
- 进入 src 目录中，执行 `sudo make install`
如果错误了，执行 `make clean`，然后重新执行上面命令

**安装结束**



**配置文件**
- 在解压的目录下面新建文件夹：log
- 在解压之后的根目录中，`vim redis.conf`
- 使用命令： `:/daemonize` 找到 daemonize，然后将其值从 no 换成 yes 之后保存。
作用是使  redis 后台启动
- 然后使用命令：`:/dir` 找到 dir 名称，将后面的 `./` 设置为：`/home/GJXAIOU/Redis/redis-5.0.5/log/` 就是修改日志输入位置。然后保存退出即可。


**启动**
在解压之后的路径之下：执行命令：`src/redis-server redis.conf` 即可；