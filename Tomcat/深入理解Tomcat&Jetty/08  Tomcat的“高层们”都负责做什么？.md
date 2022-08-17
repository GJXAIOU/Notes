# 08 | Tomcat的“高层们”都负责做什么？

使用过 Tomcat 的同学都知道，我们可以通过 Tomcat 的 /bin 目录下的脚本 startup.sh 来启动 Tomcat，那你是否知道我们执行了这个脚本后发生了什么呢？你可以通过下面这张流程图来了解一下。

![image-20220814212814581](08%20%20Tomcat%E7%9A%84%E2%80%9C%E9%AB%98%E5%B1%82%E4%BB%AC%E2%80%9D%E9%83%BD%E8%B4%9F%E8%B4%A3%E5%81%9A%E4%BB%80%E4%B9%88%EF%BC%9F.resource/image-20220814212814581-16604836949535.png)

1.Tomcat 本质上是一个 Java 程序，因此 startup.sh 脚本会启动一个 JVM 来运行 Tomcat 的启动类 Bootstrap。

2.Bootstrap 的主要任务是初始化 Tomcat 的类加载器，并且创建 Catalina。关于 Tomcat 为什么需要自己的类加载器，我会在专栏后面详细介绍。

3.Catalina 是一个启动类，它通过解析 server.xml、创建相应的组件，并调用 Server 的 start 方法。

4.Server 组件的职责就是管理 Service 组件，它会负责调用 Service 的 start 方法。

5.Service 组件的职责就是管理连接器和顶层容器 Engine，因此它会调用连接器和 Engine 的 start 方法。

这样 Tomcat 的启动就算完成了。下面我来详细介绍一下上面这个启动过程中提到的几个非常关键的启动类和组件。

你可以把 Bootstrap 看作是上帝，它初始化了类加载器，也就是创造万物的工具。

如果我们把 Tomcat 比作是一家公司，那么 Catalina 应该是公司创始人，因为 Catalina 负责组建团队，也就是创建 Server 以及它的子组件。

Server 是公司的 CEO，负责管理多个事业群，每个事业群就是一个 Service。

Service 是事业群总经理，它管理两个职能部门：一个是对外的市场部，也就是连接器组件；另一个是对内的研发部，也就是容器组件。

Engine 则是研发部经理，因为 Engine 是最顶层的容器组件。

你可以看到这些启动类或者组件不处理具体请求，它们的任务主要是“管理”，管理下层组件的生命周期，并且给下层组件分配任务，也就是把请求路由到负责“干活儿”的组件。因此我把它们比作 Tomcat 的“高层”。

今天我们就来看看这些“高层”的实现细节，目的是让我们逐步理解 Tomcat 的工作原理。另一方面，软件系统中往往都有一些起管理作用的组件，你可以学习和借鉴 Tomcat 是如何实现这些组件的。

## Catalina

Catalina 的主要任务就是创建 Server，它不是直接 new 一个 Server 实例就完事了，而是需要解析 server.xml，把在 server.xml 里配置的各种组件一一创建出来，接着调用 Server 组件的 init 方法和 start 方法，这样整个 Tomcat 就启动起来了。作为“管理者”，Catalina 还需要处理各种“异常”情况，比如当我们通过“Ctrl + C”关闭 Tomcat 时，Tomcat 将如何优雅的停止并且清理资源呢？因此 Catalina 在 JVM 中注册一个“关闭钩子”。

```
public void start() {
    //1. 如果持有的 Server 实例为空，就解析 server.xml 创建出来
    if (getServer() == null) {
        load();
    }
    //2. 如果创建失败，报错退出
    if (getServer() == null) {
        log.fatal(sm.getString("catalina.noServer"));
        return;
    }
 
    //3. 启动 Server
    try {
        getServer().start();
    } catch (LifecycleException e) {
        return;
    }
 
    // 创建并注册关闭钩子
    if (useShutdownHook) {
        if (shutdownHook == null) {
            shutdownHook = new CatalinaShutdownHook();
        }
        Runtime.getRuntime().addShutdownHook(shutdownHook);
    }
 
    // 用 await 方法监听停止请求
    if (await) {
        await();
        stop();
    }
}
```

那什么是“关闭钩子”，它又是做什么的呢？如果我们需要在 JVM 关闭时做一些清理工作，比如将缓存数据刷到磁盘上，或者清理一些临时文件，可以向 JVM 注册一个“关闭钩子”。“关闭钩子”其实就是一个线程，JVM 在停止之前会尝试执行这个线程的 run 方法。下面我们来看看 Tomcat 的“关闭钩子”CatalinaShutdownHook 做了些什么。

```
protected class CatalinaShutdownHook extends Thread {
 
    @Override
    public void run() {
        try {
            if (getServer() != null) {
                Catalina.this.stop();
            }
        } catch (Throwable ex) {
           ...
        }
    }
}
```

从这段代码中你可以看到，Tomcat 的“关闭钩子”实际上就执行了 Server 的 stop 方法，Server 的 stop 方法会释放和清理所有的资源。

## Server 组件

Server 组件的具体实现类是 StandardServer，我们来看下 StandardServer 具体实现了哪些功能。Server 继承了 LifeCycleBase，它的生命周期被统一管理，并且它的子组件是 Service，因此它还需要管理 Service 的生命周期，也就是说在启动时调用 Service 组件的启动方法，在停止时调用它们的停止方法。Server 在内部维护了若干 Service 组件，它是以数组来保存的，那 Server 是如何添加一个 Service 到数组中的呢？

```
@Override
public void addService(Service service) {
 
    service.setServer(this);
 
    synchronized (servicesLock) {
        // 创建一个长度 +1 的新数组
        Service results[] = new Service[services.length + 1];
        
        // 将老的数据复制过去
        System.arraycopy(services, 0, results, 0, services.length);
        results[services.length] = service;
        services = results;
 
        // 启动 Service 组件
        if (getState().isAvailable()) {
            try {
                service.start();
            } catch (LifecycleException e) {
                // Ignore
            }
        }
 
        // 触发监听事件
        support.firePropertyChange("service", null, service);
    }
 
}
```

从上面的代码你能看到，它并没有一开始就分配一个很长的数组，而是在添加的过程中动态地扩展数组长度，当添加一个新的 Service 实例时，会创建一个新数组并把原来数组内容复制到新数组，这样做的目的其实是为了节省内存空间。

除此之外，Server 组件还有一个重要的任务是启动一个 Socket 来监听停止端口，这就是为什么你能通过 shutdown 命令来关闭 Tomcat。不知道你留意到没有，上面 Caralina 的启动方法的最后一行代码就是调用了 Server 的 await 方法。

在 await 方法里会创建一个 Socket 监听 8005 端口，并在一个死循环里接收 Socket 上的连接请求，如果有新的连接到来就建立连接，然后从 Socket 中读取数据；如果读到的数据是停止命令“SHUTDOWN”，就退出循环，进入 stop 流程。

## Service 组件

Service 组件的具体实现类是 StandardService，我们先来看看它的定义以及关键的成员变量。

```
public class StandardService extends LifecycleBase implements Service {
    // 名字
    private String name = null;
    
    //Server 实例
    private Server server = null;
 
    // 连接器数组
    protected Connector connectors[] = new Connector[0];
    private final Object connectorsLock = new Object();
 
    // 对应的 Engine 容器
    private Engine engine = null;
    
    // 映射器及其监听器
    protected final Mapper mapper = new Mapper();
    protected final MapperListener mapperListener = new MapperListener(this);
```

StandardService 继承了 LifecycleBase 抽象类，此外 StandardService 中还有一些我们熟悉的组件，比如 Server、Connector、Engine 和 Mapper。

那为什么还有一个 MapperListener？这是因为 Tomcat 支持热部署，当 Web 应用的部署发生变化时，Mapper 中的映射信息也要跟着变化，MapperListener 就是一个监听器，它监听容器的变化，并把信息更新到 Mapper 中，这是典型的观察者模式。

作为“管理”角色的组件，最重要的是维护其他组件的生命周期。此外在启动各种组件时，要注意它们的依赖关系，也就是说，要注意启动的顺序。我们来看看 Service 启动方法：

```
protected void startInternal() throws LifecycleException {
 
    //1. 触发启动监听器
    setState(LifecycleState.STARTING);
 
    //2. 先启动 Engine，Engine 会启动它子容器
    if (engine != null) {
        synchronized (engine) {
            engine.start();
        }
    }
    
    //3. 再启动 Mapper 监听器
    mapperListener.start();
 
    //4. 最后启动连接器，连接器会启动它子组件，比如 Endpoint
    synchronized (connectorsLock) {
        for (Connector connector: connectors) {
            if (connector.getState() != LifecycleState.FAILED) {
                connector.start();
            }
        }
    }
}
```

从启动方法可以看到，Service 先启动了 Engine 组件，再启动 Mapper 监听器，最后才是启动连接器。这很好理解，因为内层组件启动好了才能对外提供服务，才能启动外层的连接器组件。而 Mapper 也依赖容器组件，容器组件启动好了才能监听它们的变化，因此 Mapper 和 MapperListener 在容器组件之后启动。组件停止的顺序跟启动顺序正好相反的，也是基于它们的依赖关系。

## Engine 组件

最后我们再来看看顶层的容器组件 Engine 具体是如何实现的。Engine 本质是一个容器，因此它继承了 ContainerBase 基类，并且实现了 Engine 接口。

```
public class StandardEngine extends ContainerBase implements Engine {
}
```

我们知道，Engine 的子容器是 Host，所以它持有了一个 Host 容器的数组，这些功能都被抽象到了 ContainerBase 中，ContainerBase 中有这样一个数据结构：

```
protected final HashMap<String, Container> children = new HashMap<>();
复制代码
```

ContainerBase 用 HashMap 保存了它的子容器，并且 ContainerBase 还实现了子容器的“增删改查”，甚至连子组件的启动和停止都提供了默认实现，比如 ContainerBase 会用专门的线程池来启动子容器。

```
for (int i = 0; i < children.length; i++) {
   results.add(startStopExecutor.submit(new StartChild(children[i])));
}
```

所以 Engine 在启动 Host 子容器时就直接重用了这个方法。

那 Engine 自己做了什么呢？我们知道容器组件最重要的功能是处理请求，而 Engine 容器对请求的“处理”，其实就是把请求转发给某一个 Host 子容器来处理，具体是通过 Valve 来实现的。

通过专栏前面的学习，我们知道每一个容器组件都有一个 Pipeline，而 Pipeline 中有一个基础阀（Basic Valve），而 Engine 容器的基础阀定义如下：

```
final class StandardEngineValve extends ValveBase {
 
    public final void invoke(Request request, Response response)
      throws IOException, ServletException {
  
      // 拿到请求中的 Host 容器
      Host host = request.getHost();
      if (host == null) {
          return;
      }
  
      // 调用 Host 容器中的 Pipeline 中的第一个 Valve
      host.getPipeline().getFirst().invoke(request, response);
  }
  
}
```

这个基础阀实现非常简单，就是把请求转发到 Host 容器。你可能好奇，从代码中可以看到，处理请求的 Host 容器对象是从请求中拿到的，请求对象中怎么会有 Host 容器呢？这是因为请求到达 Engine 容器中之前，Mapper 组件已经对请求进行了路由处理，Mapper 组件通过请求的 URL 定位了相应的容器，并且把容器对象保存到了请求对象中。

## 本期精华

今天我们学习了 Tomcat 启动过程，具体是由启动类和“高层”组件来完成的，它们都承担着“管理”的角色，负责将子组件创建出来，并把它们拼装在一起，同时也掌握子组件的“生杀大权”。

所以当我们在设计这样的组件时，需要考虑两个方面：

首先要选用合适的数据结构来保存子组件，比如 Server 用数组来保存 Service 组件，并且采取动态扩容的方式，这是因为数组结构简单，占用内存小；再比如 ContainerBase 用 HashMap 来保存子容器，虽然 Map 占用内存会多一点，但是可以通过 Map 来快速的查找子容器。因此在实际的工作中，我们也需要根据具体的场景和需求来选用合适的数据结构。

其次还需要根据子组件依赖关系来决定它们的启动和停止顺序，以及如何优雅的停止，防止异常情况下的资源泄漏。这正是“管理者”应该考虑的事情。

## 课后思考

Server 组件的在启动连接器和容器时，都分别加了锁，这是为什么呢？

不知道今天的内容你消化得如何？如果还有疑问，请大胆的在留言区提问，也欢迎你把你的课后思考和心得记录下来，与我和其他同学一起讨论。如果你觉得今天有所收获，欢迎你把它分享给你的朋友。

## 精选留言(22)

- 

  一路远行

  2019-05-28

  **7

  加锁通常的场景是存在多个线程并发操作不安全的数据结构。

  不安全的数据结构:
  Server本身包含多个Service，内部实现上用数组来存储services，数组的并发操作(包含缩容，扩容)是不安全的。所以，在并发操作(添加/修改/删除/遍历等)services数组时，需要进行加锁处理。

  可能存在并发操作的场景:
  Tomcat提供MBean的机制对管理的对象进行并发操作，如添加/删除某个service。

  展开**

- 

  allean

  2019-05-28

  **3

  老师的专栏思路清晰，讲解的深入但又通俗易懂，感谢大佬带我飞😁

  展开**

- 

  W.T

  2019-05-28

  **2

  老师，以上源码是基于tomcat的哪个版本？

  展开**

  作者回复: 最新9.x版

- 

  刘章周

  2019-06-04

  **1

  老师，1.catalina创建组件，是把所有的对象都new出来了吧，只是各个组件之间没有相互注入吧。
  2.为什么catalina直接调用server的start方法？不是先init吗？
  3.容器之间是什么时候注入进去的？还有listener是什么时候注入到组件中去的？

  展开**

  作者回复: 1，对的，直接new出来
  2，start方法里调了init方法
  3，父容器应该是在构造函数里new了子容器

- 

  why

  2019-06-02

  **1

  \- Tomcat 本质是 Java 程序, [startup.sh](http://startup.sh) 启动 JVM 运行 Tomcat 启动类 bootstrap
  \- Bootstrap 初始化类加载器, 创建 Catalina
  \- Catalina 解析 server.xml, 创建相应组件, 调用 Server start 方法
  \- Server 组件管理 Service 组件并调用其 start 方法
  \- Service 负责管理连接器和顶层容器 Engine, 其会调用 Engine start 方法
  \- 这些类不处理具体请求, 主要管理下层组件, 并分配请求
  \- Catalina 完成两个功能
    \- 解析 server.xml, 创建定义的各组件, 调用 server init 和 start 方法
    \- 处理异常情况, 例如 ctrl + c 关闭 Tomcat. 其会在 JVM 中注册"关闭钩子"
      \- 关闭钩子, 在关闭 JVM 时做清理工作, 例如刷新缓存到磁盘
      \- 关闭钩子是一个线程, JVM 停止前会执行器 run 方法, 该 run 方法调用 server stop 方法
  \- Server 组件, 实现类 StandServer
    \- 继承了 LifeCycleBase
    \- 子组件是 Service, 需要管理其生命周期(调用其 LifeCycle 的方法), 用数组保存多个 Service 组件, 动态扩容数组来添加组件
    \- 启动一个 socket Listen停止端口, Catalina 启动时, 调用 Server await 方法, 其创建 socket Listen 8005 端口, 并在死循环中等连接, 检查到 shutdown 命令, 调用 stop 方法
  \- Service 组件, 实现类 StandService
    \- 包含 Server, Connector, Engine 和 Mapper 组件的成员变量
    \- 还包含 MapperListener 成员变量, 以支持热部署, 其Listen容器变化, 并更新 Mapper, 是观察者模式
    \- 需注意各组件启动顺序, 根据其依赖关系确定
      \- 先启动 Engine, 再启动 Mapper Listener, 最后启动连接器, 而停止顺序相反.
  \- Engine 组件, 实现类 StandEngine 继承 ContainerBase
    \- ContainerBase 实现了维护子组件的逻辑, 用 HaspMap 保存子组件, 因此各层容器可重用逻辑
    \- ContainerBase 用专门线程池启动子容器, 并负责子组件启动/停止, "增删改查"
    \- 请求到达 Engine 之前, Mapper 通过 URL 定位了容器, 并存入 Request 中. Engine 从 Request 取出 Host 子容器, 并调用其 pipeline 的第一个 valve

  展开**

- 

  清风

  2019-05-29

  **1

  调试源码，会遇到在执行不到对应的断点的情况，看调用栈是被之前的断点拦住了，这个有什么好的断点调试方法吗，这样调试太费劲了

  作者回复: IDE一般有disable断点的功能

- 

  轩恒

  2019-05-29

  **1

  有个常见问题请教一下，在实际应用场景中，tomcat在shutdown的时候，无法杀死java进程，还得kill，这是为何呢？

  作者回复: Tomcat会调用Web应用的代码来处理请求，可能Web应用代码阻塞在某个地方。

- 

  Monday

  2019-05-29

  **1

  listener对应的中文（三个字）竟然是敏感词。。。

  展开**

- 

  Monday

  2019-05-29

  **1

  根据老师的给出的Github上Tomcat源码调试Tomcat的启动过程，遇到以下这个问题。
  经debug发现，运行完Catalina.load()方法的第566行digester.parse(inputSource)初始化了Server对象。但是我单步进入第566行，各种操作都没有跟踪到具体是哪一行初始化了Server对象。莫非有Listener？

  展开**

  作者回复: digester应该是通过反射的方式创建Server对象的。

- 

  allean

  2019-05-28

  **1

  如果映射关系不变，而是某个具体的Servlet的方法处理逻辑变了，热部署也可以解决重启tomcat的尴尬吗

  作者回复: 那不叫热部署，叫热加载。
  热部署和热加载都不需要重启Tomcat。

- 

  wwm

  2019-05-28

  **1

  老师，请教一个问题：
  在Bootstrap中，基于什么原因用反射的方式创建Catalina实例，之后继续基于反射方式调用load、init、start这些方法？为什么不是直接new Catalina实例后通过实例直接调用这些方法？

  作者回复: Tomcat有自己的类加载器体系，Catalina相关的类都是由
  专门的类加载器catalinaLoader来加载：

  Class<?> startupClass = catalinaLoader.loadClass("org.apache.catalina.startup.Catalina")

  Tomcat的类加载器体系会有专门的文章来详细解释。

- 

  大卫

  2019-06-05

  **

  老师好，tomcat一般生产环境线程数大小建议怎么设置呢

  展开**

  作者回复: 理论上：

  线程数=（(线程阻塞时间 + 线程忙绿时间) / 线程忙碌时间) * cpu核数

  如果线程始终不阻塞，一直忙碌，会一直占用一个CPU核，因此可以直接设置 线程数=CPU核数。

  但是现实中线程可能会被阻塞，比如等待IO。因此根据上面的公式确定线程数。

  那怎么确定线程的忙碌时间和阻塞时间？要经过压测，在代码中埋点统计，专栏后面的调优环节会涉及到。

- 

  佑儿

  2019-06-04

  **

  Mapping和MapperListener会重点讲嘛？

  展开**

  作者回复: 在Tomcat系统架构下篇有Mapper的原理介绍，怎么将一个请求映射到具体容器来处理。

- 

  802.11

  2019-06-02

  **

  abel614: {
                label613: {
                  try {
  老师，在tomcat出现的，这是什么语法呀

  作者回复: 可能是你的IDE没有把源码下载下来，这是反编译的代码：）

- 

  Geek_00d56...

  2019-05-30

  **

  今天我们学习了 Tomcat 启动过程，具体是由启动类和“高层”组件来完成的，它们都承担着“管理”的角色，负责将子组件创建出来，并把它们拼装在一起，同时也掌握子组件的“生杀大权”。

  加锁🔒，是因为有并发。

  

  展开**

- 

  锦

  2019-05-29

  **

  加锁可能会有多个连接器和容器并发创建。

  展开**

- 

  王智

  2019-05-29

  **

  老师,具体的注入是个什么样的概念,上节课说的子组件注入到父组件,内层组件注入到外层组件,这是个什么样的操作,在上面的代码中并没有看到具体的操作呢?

  展开**

  作者回复: 其实就是父组件持有子组件的引用，new出来就好了。

- 

  -W.LI-

  2019-05-29

  **

  老师好!catalina的start()方法末尾那部分不太理解能帮忙讲解下么。
  // 用 await 方法监听停止请求
    if (await) {
      await();
      stop();
    }
  if里面那个await我理解是一个属性值是否启用await。然后进入await()方法，出了await()就直接stop()了。下文老师说await()是调用了Server的await()方法，然后Server的await()方法会死循环监听8005端口。读取到停止命令就会退出死循环。回到catalina执行stop方法。我这边的问题是。调用catalina.start方法的线程一直阻塞着，处理监听事件么，监听到关闭事件就去stop()?。感觉好怪啊!请老师看下哪里理解错了。
  课后问题:文中说部分通过线程池实现并发加载，加同步方法就是为了保证线程安全。

  展开**

  作者回复: 不怪把，这里的考虑是怎么让Tomcat这个程序不退出，否则程序执行完了就终止了。

- 

  发条橙子 ...

  2019-05-29

  **

  老师，对于你的问题，实际上我也不理解为何要加锁 。
  首先，按理说server对每一个service开一个线程去初始化 。 应该不会多个线程对一个service同时初始化吧。
  再者，这块同步如果是要防止重复初始化，那应该在start()方法中做，否则等释放锁后，下一个线程获得锁还是会执行start()方法。

  所以这块加锁具体的作用我也看不懂，难道是起到多线程同步阻塞的作用？？

  展开**

  作者回复: 不管外面怎么调，加了锁这个方法就是线程安全的了

- 

  QQ怪

  2019-05-29

  **

  因为还用了线程不安全的hashmap

  展开**