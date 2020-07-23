## tomcat学习|tomcat中的类加载器

原创： 微笑的小小刀 [程序员学习大本营](javascript:void(0);) _今天_

**开头说两句**

小刀博客: http://www.lixiang.red
小刀公众号: 程序员学习大本营

# **学习背景**

上期我们聊到了tomcat中各个组件在默认值,在其中,我们看到了有关类加载器的代码, 如context中初始化wabAppLoader
https://www.lixiang.red/articles/2019/08/09/1565361802994.html
今天我们一起学习类加载器相关知识

# **java里面的类加载器**

我们在写java代码时,源文件是 *.java , 然后经过编译之后,会变成 _.class 文件,类加载器加载的,实际上就是_.class文件, 在实际开发中,我们会把相关的 .class 文件,打成一个jar包, 然后直接加载jar包就可以了.
类加载器就是用来加载这些类到虚拟机里,供程序调用

## **Bootstrap Class Loader**

用来加载JVM提供的基础运行类,即位于%JAVA_HOME%jre/lib 这个目录下面的核心类库

![](https://mmbiz.qpic.cn/mmbiz_png/8sl8s4eiazL6fISdedMYOzxTmdO3gUUHrbffVvyfwibY0wBr7XZhPI0V6TzQy7BRib5BvWsjdewrAnVIDRatE7ERw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## **Extension Class Loader**

java提供的一个标准的扩展机制,用于加载除核心类库外的jar包.默认的扩展目录是 %JAVA_HOME%/jar/lib/ext

![](https://mmbiz.qpic.cn/mmbiz_png/8sl8s4eiazL6fISdedMYOzxTmdO3gUUHrLyJlVQgTjVeLeumibzcyN2FheH3fGgzvQzsKFDEeAgMhrO7V9FqpGaw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

该目录下的类库,对所有基于该JVM运行的程序都是可见的

## **System class loader**

用于加载环境变量 CLASSPATH 指定目录下的或者是用 -classpath运行参数指定的jar包.

![](https://mmbiz.qpic.cn/mmbiz_png/8sl8s4eiazL6fISdedMYOzxTmdO3gUUHrKDGrdGib5gbh3GXrV6L5Wo3hFLCDzpJjmyrV116uOlNNeBnDI3YiaLyw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

System Class Loader 通常用于加载应用程序jar包及其启动入口类(Tomcat Bootstrap类就是由System Class Loader 来加载的)

# **类加载器的双亲委派模式**

上面三种类加载器,实际上是有父子关系,Bootstrap 是 Extension的父加载器, Extension 是System的父加载器
当System ClassLoader 拿到一个class 文件之后, 会先问父加载器(Extension Class Loader)能不能加载,当(Extension )接收到请求时,会先问问他的父加载器(BootStrap类加载器能不能加载). 如果 Bootstrap可以加载,则由Bootstrap来加载,如不能,则由Extension来加载. 如果 Extension 也加载不了的话,最后由System 类加载器来加载

![](https://mmbiz.qpic.cn/mmbiz_png/8sl8s4eiazL6fISdedMYOzxTmdO3gUUHr6pVNXeeFQJ2GWD4pYiap9vDRZ76ricseKEDickrCSfwQPsTnUzN2vjwZg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

# **tomcat中的类加载器**

总共有四种类加载器: Common Class Loader, Catalina Class Loader , Shared Class Loader, Web AppClass Loader.
tomcat中各个类加载器初始化,声明的地方

```

/**
     * 初始化tomcat中的三大类加载器
     */
    private void initClassLoaders() {
        try {
            // CommonLoader 默认继承AppClassLoader
            commonLoader = createClassLoader("common", null);
            if( commonLoader == null ) {
                // no config file, default to this loader - we might be in a 'single' env.
                commonLoader=this.getClass().getClassLoader();
            }
            // 以CommonLoader 为父加载器
            catalinaLoader = createClassLoader("server", commonLoader);
            // 以CommonLoader 为父加载器
            sharedLoader = createClassLoader("shared", commonLoader);
        } catch (Throwable t) {
            handleThrowable(t);
            log.error("Class loader creation threw exception", t);
            System.exit(1);
        }
    }
```

![](https://mmbiz.qpic.cn/mmbiz_png/8sl8s4eiazL6fISdedMYOzxTmdO3gUUHr5XxqMwAymibOSv1saGtNqYZBc3RvykTcZYoylt9ibhJhTIJwkibOMiansA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![](https://mmbiz.qpic.cn/mmbiz_png/8sl8s4eiazL6fISdedMYOzxTmdO3gUUHrvZ4IqvicVp1baPicmo7headdL2JO6RB4AHJLsBoUB9hULUPm8tJWphxA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## **Common Class loader**

以System Class Loader 为父类加载器, 是位于Tomcat 应用服务器顶层的公用类加载器,默认是加载$CATALINE_HOME/lib 下的jar 

![](https://mmbiz.qpic.cn/mmbiz_png/8sl8s4eiazL6fISdedMYOzxTmdO3gUUHrEicSLGic5gtvQ5nich0t92kuspQt58o4c2eoIV6ico2o3cdtEFVkITjibMg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## **Catalina Class Loader**

以Common Class Loader 为父加载器.用于加载 Tomcat 应用服务器本身的.可以在下图中看到使用的位置
1.设置当前线程的类加载器为Catalina Class Loader , 在没设置之前,是由 shell 脚本用 System Class Loader 来加载的
2\. 用Catalina Class Loader 加载 Catalina.class 这个文件,并完成一系统组件的初始

![](https://mmbiz.qpic.cn/mmbiz_png/8sl8s4eiazL6fISdedMYOzxTmdO3gUUHr7FRlftePJwAlZ2mgBicmTtrRNKW8Ldua020uUhST5aNyggF25licgBPA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## **shared Class Loader**

以Common 为父加载器,是所有web应用的父加载器
使用位置如下

![](https://mmbiz.qpic.cn/mmbiz_png/8sl8s4eiazL6fISdedMYOzxTmdO3gUUHr6b0EjC8vRMvcaQulKa1GzNjqzxASiccAJBUlACyzRFNApypMkqpcqog/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

```

/**
     * Set the shared extensions class loader.
     *
     * @param parentClassLoader The shared extensions class loader.
     */
    public void setParentClassLoader(ClassLoader parentClassLoader) {
        this.parentClassLoader = parentClassLoader;
    }
```

在源码中声明parentClassLoader时有一个小坑,他默认是声明的加载Catalina的类载器            即:Catalina Class Loader,但实际上,在实例化后,我们会用反射调用其setParentClassLoader 方法,将parentClassLoader 更改为shared Class Loader

```

// XXX Should be moved to embedded
    /**
     * The shared extensions class loader for this server.
     */
    protected ClassLoader parentClassLoader =
        Catalina.class.getClassLoader();
```

使用地点、

![](https://mmbiz.qpic.cn/mmbiz_png/8sl8s4eiazL6fISdedMYOzxTmdO3gUUHrgvWT7SyjZY0DMF7OL5tel1IfROtrWlypPryUwPOVY1P4wKiccneqy9g/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**WebApp Class Loader**

初始化的地点有两处:
1.createStartDigester中

```
digester.addObjectCreate(prefix + "Context/Loader",
                            "org.apache.catalina.loader.WebappLoader",
                            "className");
```

1.  StandardContext.startInternal 方法中

```
   
if (getLoader() == null) {
            WebappLoader webappLoader = new WebappLoader(getParentClassLoader());
            webappLoader.setDelegate(getDelegate());
            setLoader(webappLoader);
        }
```

其作用是,每个独立的Context(web应用)都使用独立的ClassLoader,加载我们web应用中,WEB-INFO/libs 这个目录下的jar(如我们在应用中引用的spring , mybatis 这些包)
这个做的好处是,不同的web应用包不会冲突,如A应用用的是spring 4.X , B应用用的是spring 5.X , 他们可以在同一个tomcat中运行

## **最后说两句**

tomcat的类加载机制, 是开始的一个比较复杂的点,需要好好理一理,边看代码边做笔记,这个类加载器什么时候初始化的,做了什么,然后在哪里使用的. 大家在学习过种中,有什么问题,可以和小刀一起交流讨论: best396975802