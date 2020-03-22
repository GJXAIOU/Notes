## JVM的启动及java -version的执行过程

## tomcat有没有main函数

在学启动的时候, 我一直在想一个以前的java问题,就是Tomcat有没有Main函数, 答案肯定是有! 那么jvm做为一个C++应用程序, 他也肯定有man函数, 我们坚定这一点, 然后再去看代码

## JVM的main函数

main 函数位置为：`src/launcher/main.c`

完整代码为：

```c
/*
 * Copyright (c) 1995, 2012, Oracle and/or its affiliates. All rights reserved.
 * ORACLE PROPRIETARY/CONFIDENTIAL. Use is subject to license terms.
 *
 *
 *
 *
 *
 *
 *
 *
 *
 *
 *
 *
 *
 *
 *
 *
 *
 *
 *
 *
 */


/*
 * This file contains the main entry point into the launcher code
 * this is the only file which will be repeatedly compiled by other
 * tools. The rest of the files will be linked in.
 */

#include "defines.h"

#ifdef _MSC_VER
#if _MSC_VER > 1400 && _MSC_VER < 1600

/*
 * When building for Microsoft Windows, main has a dependency on msvcr??.dll.
 *
 * When using Visual Studio 2005 or 2008, that must be recorded in
 * the [java,javaw].exe.manifest file.
 *
 * As of VS2010 (ver=1600), the runtimes again no longer need manifests.
 *
 * Reference:
 *     C:/Program Files/Microsoft SDKs/Windows/v6.1/include/crtdefs.h
 */
#include <crtassem.h>
#ifdef _M_IX86

#pragma comment(linker,"/manifestdependency:\"type='win32' "            \
        "name='" __LIBRARIES_ASSEMBLY_NAME_PREFIX ".CRT' "              \
        "version='" _CRT_ASSEMBLY_VERSION "' "                          \
        "processorArchitecture='x86' "                                  \
        "publicKeyToken='" _VC_ASSEMBLY_PUBLICKEYTOKEN "'\"")

#endif /* _M_IX86 */

//This may not be necessary yet for the Windows 64-bit build, but it
//will be when that build environment is updated.  Need to test to see
//if it is harmless:
#ifdef _M_AMD64

#pragma comment(linker,"/manifestdependency:\"type='win32' "            \
        "name='" __LIBRARIES_ASSEMBLY_NAME_PREFIX ".CRT' "              \
        "version='" _CRT_ASSEMBLY_VERSION "' "                          \
        "processorArchitecture='amd64' "                                \
        "publicKeyToken='" _VC_ASSEMBLY_PUBLICKEYTOKEN "'\"")

#endif  /* _M_AMD64 */
#endif  /* _MSC_VER > 1400 && _MSC_VER < 1600 */
#endif  /* _MSC_VER */

/*
 * Entry point.
 */
#ifdef JAVAW

char **__initenv;

int WINAPI
WinMain(HINSTANCE inst, HINSTANCE previnst, LPSTR cmdline, int cmdshow)
{
    int margc;
    char** margv;
    const jboolean const_javaw = JNI_TRUE;

    __initenv = _environ;

#else /* JAVAW */
int
main(int argc, char **argv)
{
    int margc;
    char** margv;
    const jboolean const_javaw = JNI_FALSE;
#endif /* JAVAW */
#ifdef _WIN32
    {
        int i = 0;
        if (getenv(JLDEBUG_ENV_ENTRY) != NULL) {
            printf("Windows original main args:\n");
            for (i = 0 ; i < __argc ; i++) {
                printf("wwwd_args[%d] = %s\n", i, __argv[i]);
            }
        }
    }
    JLI_CmdToArgs(GetCommandLine());
    margc = JLI_GetStdArgc();
    // add one more to mark the end
    margv = (char **)JLI_MemAlloc((margc + 1) * (sizeof(char *)));
    {
        int i = 0;
        StdArg *stdargs = JLI_GetStdArgs();
        for (i = 0 ; i < margc ; i++) {
            margv[i] = stdargs[i].arg;
        }
        margv[i] = NULL;
    }
#else /* *NIXES */
    margc = argc;
    margv = argv;
#endif /* WIN32 */
    return JLI_Launch(margc, margv,
                   sizeof(const_jargs) / sizeof(char *), const_jargs,
                   sizeof(const_appclasspath) / sizeof(char *), const_appclasspath,
                   FULL_VERSION,
                   DOT_VERSION,
                   (const_progname != NULL) ? const_progname : *margv,
                   (const_launcher != NULL) ? const_launcher : *margv,
                   (const_jargs != NULL) ? JNI_TRUE : JNI_FALSE,
                   const_cpwildcard, const_javaw, const_ergo_class);
}

```



点进去文件可以看到jvm对不同的处理器32位/64位和不同的启动类型javaw/java做不了同的处理.

JAVAW 的启动头是

```c
/*
 * Entry point.
 */
#ifdef JAVAW

char **__initenv;

int WINAPI
WinMain(HINSTANCE inst, HINSTANCE previnst, LPSTR cmdline, int cmdshow)
{
    int margc;
    char** margv;
    const jboolean const_javaw = JNI_TRUE;

    __initenv = _environ;

#else /* JAVAW */
int
main(int argc, char **argv)
{
    int margc;
    char** margv;
    const jboolean const_javaw = JNI_FALSE;
#endif /* JAVAW */
```





java的启动头是

```
JNIEXPORT intmain(int argc, char **argv){    int margc;    char** margv;    int jargc;    char** jargv;    const jboolean const_javaw = JNI_FALSE;
```





往下的方法体中可以看到大都是对参数进行的处理, 直到这个文件的最后,开始Launch启动



```
return JLI_Launch(margc, margv,               jargc, (const char**) jargv,               0, NULL,               VERSION_STRING,               DOT_VERSION,               (const_progname != NULL) ? const_progname : *margv,               (const_launcher != NULL) ? const_launcher : *margv,               jargc > 0,               const_cpwildcard, const_javaw, 0)
```



该方法的实现是在`src/java.base/share/native/libjli/java.c` 这个目录下

在这个方法中, 我们可以看到, 主要是以下几步:

调用方法`CreateExecutionEnvironment` 创建运行环境

调用方法`LoadJavaVM` 去加载libjvm

调用方法`ParseArguments` 去解析参数

最后调用`JVMInit`  去启动JVM

## JVMInit

这个方法就跟操作系统有关了,如下图所示不同的系统会去执行不同的文件的中代码

![img](https://mmbiz.qpic.cn/mmbiz_png/8sl8s4eiazL5MBJdYLMjIvpsCaPChyWDic12R9ibW8HZhGcquyiayEbNpGVLpzhiaicJm6eBTonDGlMXK48SmicCvGEaA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

在对应系统的`JVMInit`的方法中, 会调用`java.c` 中的`ContinueInNewThread`方法,并在方法`ContinueInNewThread` 中调用不同操作系统的`ContinueInNewThread0`方法,如下图所示:

![img](https://mmbiz.qpic.cn/mmbiz_png/8sl8s4eiazL5MBJdYLMjIvpsCaPChyWDicnlYQsfdYr2PT178vGZPm7UFkkclc7tYxLe2bmlk30D1DAae0QN7PFw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这样我们就来到了第一天,用Clion调试JDK源码时的JavaMain方法



## JavaMain 方法

这里还是以打印版本号为例,可以看到在444行,有如下代码:

- 
- 
- 
- 
- 
- 
- 

```
if (printVersion || showVersion) {    PrintJavaVersion(env, showVersion);    CHECK_EXCEPTION_LEAVE(0);    if (printVersion) {        LEAVE();    }}
```



可知实际调用的是`PrintJavaVersion(env,showVersion)`这个方法.

其实现如下:

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
static voidPrintJavaVersion(JNIEnv *env, jboolean extraLF){    jclass ver;    jmethodID print;    // 找到指定的类    NULL_CHECK(ver = FindBootStrapClass(env, "java/lang/VersionProps"));    // 找到指定的方法    NULL_CHECK(print = (*env)->GetStaticMethodID(env,                                                 ver,                                                 (extraLF == JNI_TRUE) ? "println" : "print",                                                 "(Z)V"                                                 )              );    // 调用这个方法    (*env)->CallStaticVoidMethod(env, ver, print, printTo);}
```



我们在Idea中,引入11的JDK , 然后就可以看到对应的输出

![img](https://mmbiz.qpic.cn/mmbiz_png/8sl8s4eiazL5MBJdYLMjIvpsCaPChyWDicpVPwa6uL2rBP58AnHXCphEPFicRgeAW8yQbITF9UqGvFPEtGr4DaBkg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



## 总结

至此,java -version的执行过程,我们是已经了解了, 而且借着`java -version` 我们还了解到了jvm虚拟机启动的过程. 这部分是后面的基础, 希望小伙伴们能跟着小刀一起,落实这方面的知识,加油!





# jvm启动分析

原创[freud.wy](https://me.csdn.net/wuyu6394232) 最后发布于2019-01-18 14:12:21 阅读数 25 收藏

展开

最近比较闲，一直想学习下jvm是怎么运行的，所以搭建了一套环境分析分析。

目前环境为mac+xcode+jdk9，使用clang编译，clang版本clang-900.0.39.2。

## 总体流程

jdk总体分为lancher和jvm，jvm以动态链接库的形式被lancher加载。

lancher入口在jdk/src/java.base/share/native/launcher/main.c:94中

启动线程会调用JLI_Launch（jdk/src/java.base/share/native/launcher/main.c:166）其中会启动java的主线程，java主线程入口在

jdk/src/java.base/share/native/libjli/java.c:386的JavaMain函数。

## lancher线程的关键流程

1.获取jrepath和jvmpath：jdk/src/java.base/share/native/libjli/java.c:269

2.加载jvm：jdk/src/java.base/share/native/libjli/java.c:285

3.判断java主线程和lancher线程是否共用一个线程：jdk/src/java.base/macosx/native/libjli/java_md_macosx.c:1043

4.启动java主线程（jdk/src/java.base/macosx/native/libjli/java_md_macosx.c:1071） 后续流程交付给java主线程。

## java主线程关键流程

1.初始化jvm：jdk/src/java.base/share/native/libjli/java.c:408

->jdk/src/java.base/share/native/libjli/java.c:1481

->hotspot/src/share/vm/prims/jni.cpp:4032

->hotspot/src/share/vm/prims/jni.cpp:3937

其中流程进一步拆分

  1）把java主线程设置成当前os线程：hotspot/src/share/vm/runtime/thread.cpp:3607-3612

  2)  初始化堆：hotspot/src/share/vm/memory/universe.cpp:672

  3)  初始化vm线程hotspot/src/share/vm/runtime/thread.cpp:3658

  4）初始化所有java.lang的class:hotspot/src/share/vm/runtime/thread.cpp:3700

 2.加载待执行的main class：jdk/src/java.base/share/native/libjli/java.c:505

 3.获取main函数的methodid：jdk/src/java.base/share/native/libjli/java.c:541

 4.调用main函数：jdk/src/java.base/share/native/libjli/java.c:546

 

