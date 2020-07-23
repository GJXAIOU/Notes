# **从已知startup.bat/sh入手**

windows上启动是:startup.bat
linux/mac上启动是startup.sh

![1]($resource/1.png)

## **startup.sh**

重点在于最后一行:

```
# PRGDIR 是当前tomcat下的bin目录
PRGDIR=`dirname "$PRG"`

EXECUTABLE=catalina.sh

执行tomcat/bin/catalina.sh start 
$@ 是代表全部的参数,
exec "$PRGDIR"/"$EXECUTABLE" start "$@"
```

## catalina.sh

我们通过参数start进行追踪:

![2]($resource/2.png)

在这里看到了对start参数进行判断,然后走不同的分支逻辑
经过一大堆的判断,最后达到可启动的状态时,就开始执行启动的命令:

![3]($resource/3.png)

nohup: 是linux 系统中,可以后台运行程序的命令,窗口关掉也会存在

ENDORSED_PROP: 可以覆盖部分jvm bootstarp类加载器加载的类

org.apache.catalina.startup.Bootstrap: 最后会执行这个类

## Bootstrap

根据上文,我们可以找到Bootstrap这个类的main函数

![4]($resource/4.png)

如上图所示,只要执行了以下几步:
1\. 初始化bootstrap实例
2\. 调用bootstrap的init方法,去初始化类加载器,以及catalina实例
3\. 调用bootstrap的start方法,然后通过反射去调用catalina的start 方法

init():

```
public void init() throws Exception {
        // 初始化类加载器
        initClassLoaders();
        // 设置当前线程的类加载器为catalinaLoader
        // 当前线程:初始化Catalina的线程,初始化Servlet容器的这个线程
        Thread.currentThread().setContextClassLoader(catalinaLoader);

        SecurityClassLoad.securityClassLoad(catalinaLoader);

        // Load our startup class and call its process() method
        if (log.isDebugEnabled()){
            log.debug("Loading startup class");
        }
        // 寻找到 Catalina 类,然后用反射进行实例化
        Class<?> startupClass = catalinaLoader.loadClass("org.apache.catalina.startup.Catalina");
        Object startupInstance = startupClass.getConstructor().newInstance();

        // Set the shared extensions class loader
        if (log.isDebugEnabled()){
            log.debug("Setting startup class properties");
        }
        // 获取到Catalina.setParentClassLoader的方法
        String methodName = "setParentClassLoader";
        Class<?> paramTypes[] = new Class[1];
        paramTypes[0] = Class.forName("java.lang.ClassLoader");
        Object paramValues[] = new Object[1];
        paramValues[0] = sharedLoader;
        // 通过反射设置 Catalina实例使用的父类加载器为sharedLoader
        Method method =
            startupInstance.getClass().getMethod(methodName, paramTypes);
        method.invoke(startupInstance, paramValues);

        catalinaDaemon = startupInstance;

    }
```

start():

```
public void start()
        throws Exception {
        if( catalinaDaemon==null ) init();
        // 使用反射去调用Catalina 的 start 方法
        Method method = catalinaDaemon.getClass().getMethod("start", (Class [] )null);
        method.invoke(catalinaDaemon, (Object [])null);

    }
```

# 最后说两句

这样就完成了tomcat 从命令行到bootstrap以及catalina的初步初始化.后面还有server,service , Engine , host 等组件的加载