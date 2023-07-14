# Spring Boot如何远程调试？

## 前言

上周末一个朋友庆生，无意间听他说起了近况，说公司项目太多了，每天一堆BUG需要修复，项目来回切换启动，真是挺烦的。

随着项目越来越多，特别是身处外包公司的朋友，每天可能需要切换两三个项目，难道一有问题就本地启动项目调试？

今天这篇文章就来介绍一下什么是远程调试，`Spring Boot`如何开启远程调试？

## 什么是远程调试？

所谓的远程调试就是服务端程序运行在一台远程服务器上，我们可以在本地服务端的代码（**前提是本地的代码必须和远程服务器运行的代码一致**）中设置断点，每当有请求到远程服务器时时能够在本地知道远程服务端的此时的内部状态。

> 简单的意思：本地无需启动项目的状态下能够实时调试服务端的代码。

## 为什么要远程调试？

随着项目的体量越来越大，启动的时间的也是随之增长，何必为了调试一个BUG花费十分钟的时间去启动项目呢？你不怕老大骂你啊？

## 什么是JPDA？

`JPDA`(`Java Platform Debugger Architecture`)，即 Java 平台调试体系，具体结构图如下图所示：

![image-20230607113652212](IDEA%20%E8%BF%9C%E7%A8%8B%E8%B0%83%E8%AF%95.resource/image-20230607113652212.png)

其中实现调试功能的主要协议是`JDWP`协议，在 `Java SE 5` 以前版本，JVM 端的实现接口是 `JVMPI`(Java Virtual Machine Profiler Interface)，而在`Java SE 5`及以后版本，使用 `JVMTI`(Java Virtual Machine Tool Interface) 来替代 JVMPI。

因此，如果你使用的是`Java SE 5`之前的版本，则使用的调试命令格式如下：

```
java -Xdebug -Xrunjdwp:...
```

如果你使用的是`Java SE 5`之后的版本，则使用的命令格式如下：

```
java -agentlib:jdwp=...
```

## 如何开启调试？

由于现在使用的大多数都是`Java SE 5`之后的版本，则之前的就忽略了。

日常开发中最常见的开启远程调试的命令如下：

```
java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=9093 -jar xxx.jar
```

前面的`java -agentlib:jdwp=`是基础命令，后面的跟着的一串命令则是可选的参数，具体什么意思呢？下面详细介绍。

- transport

    指定运行的被调试应用和调试者之间的通信协议，有如下可选值：

    - `dt_socket`： 采用`socket`方式连接（常用）
    - `dt_shmem`：采用共享内存的方式连接，支持有限，仅仅支持windows平台

- server

    指定当前应用作为调试服务端还是客户端，默认的值为`n`（客户端）。

    如果你想将当前应用作为被调试应用，设置该值为`y`;如果你想将当前应用作为客户端，作为调试的发起者，设置该值为`n`。

- suspend

    当前应用启动后，是否阻塞应用直到被连接，默认值为`y`（阻塞）。

    大部分情况下这个值应该为`n`，即不需要阻塞等待连接。一个可能为`y`的应用场景是，你的程序在启动时出现了一个故障，为了调试，必须等到调试方连接上来后程序再启动。

- address

    对外暴露的端口，默认值是`8000`

    **注意**：此端口不能和项目同一个端口，且未被占用以及对外开放。

- onthrow

    这个参数的意思是当程序抛出指定异常时，则中断调试。

- onuncaught

    当程序抛出未捕获异常时，是否中断调试，默认值为`n`。

- launch

    当调试中断时，执行的程序。

- timeout

    超时时间，单位`ms`（毫秒）

    当 `suspend = y` 时，该值表示等待连接的超时；当 `suspend = n` 时，该值表示连接后的使用超时。

## 常用的命令

下面列举几个常用的参考命令，这样更加方便理解。

以`Socket` 方式监听 `8000` 端口，程序启动阻塞（`suspend` 的默认值为 `y`）直到被连接，命令如下：

```
-agentlib:jdwp=transport=dt_socket,server=y,address=8000
```

以 `Socket` 方式监听 `8000` 端口，当程序启动后 `5` 秒无调试者连接的话终止，程序启动阻塞（`suspend` 的默认值为 `y`）直到被连接。

```
-agentlib:jdwp=transport=dt_socket,server=y,address=localhost:8000,timeout=5000
```

选择可用的共享内存连接地址并使用 `stdout` 打印，程序启动不阻塞。

```
-agentlib:jdwp=transport=dt_shmem,server=y,suspend=n
```

以 `socket` 方式连接到 `myhost:8000`上的调试程序，在连接成功前启动阻塞。

```
-agentlib:jdwp=transport=dt_socket,address=myhost:8000
```

以 `Socket` 方式监听 `8000` 端口，程序启动阻塞（`suspend` 的默认值为 `y`）直到被连接。当抛出 `IOException` 时中断调试，转而执行 `usr/local/bin/debugstub`程序。

```
-agentlib:jdwp=transport=dt_socket,server=y,address=8000,onthrow=java.io.IOException,launch=/usr/local/bin/debugstub
```

## IDEA如何开启远程调试？

首先的将打包后的`Spring Boot`项目在服务器上运行，执行如下命令（各种参数根据实际情况自己配置）：

```
java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=9193 -jar debug-demo.jar
```

项目启动成功后，点击 `Edit Configurations`，在弹框中点击 `+` 号，然后选择`Remote`。

![image-20230607114443243](IDEA%20%E8%BF%9C%E7%A8%8B%E8%B0%83%E8%AF%95.resource/image-20230607114443243.png)

然后填写服务器的地址及端口，点击 `OK` 即可。

![image-20230607114453531](IDEA%20%E8%BF%9C%E7%A8%8B%E8%B0%83%E8%AF%95.resource/image-20230607114453531.png)

以上步骤配置完成后，点击DEBUG调试运行即可。

![image-20230607114503531](IDEA%20%E8%BF%9C%E7%A8%8B%E8%B0%83%E8%AF%95.resource/image-20230607114503531.png)

配置完毕后点击保存即可，因为我配置的 `suspend=n`，因此服务端程序无需阻塞等待我们的连接。我们点击 `IDEA` 调试按钮，当我访问某一接口时，能够正常调试。

![image-20230607114517310](IDEA%20%E8%BF%9C%E7%A8%8B%E8%B0%83%E8%AF%95.resource/image-20230607114517310.png)