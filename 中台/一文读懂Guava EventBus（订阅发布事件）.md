# 一文读懂Guava EventBus（订阅\发布事件）

## 零、背景

本文主要分为五个部分：

- 1、简述：简单介绍EventBus及其组成部分。
- 2、原理解析：主要对listener注册流程及Event发布流程进行解析。
- 3、使用指导：EventBus简单的使用指导。
- 4、注意事项：在使用EventBus中需要注意的一些隐藏逻辑。
- 5、分享时提问的问题
- 6、项目中遇到的问题：上述问题进行详细描述并复现场景。

## 一、简述

### （一）概念

下文摘自EventBus源码注释，从注释中可以直观了解到他的**功能、特性、注意事项**。

【源码注释】

> Dispatches events to listeners, and provides ways for listeners to register themselves.
>
> The EventBus allows publish-subscribe-style communication between components without requiring the components to explicitly register with one another (and thus be aware of each other). **It is designed exclusively to replace traditional Java in-process event distribution using explicit registration**. It is not a general-purpose publish-subscribe system, nor is it intended for interprocess communication.
>
> **Receiving Events**
>
> To receive events, an object should:
>
> - Expose a public method, known as the event subscriber, which accepts a single argument of the type of event desired;
> - Mark it with a Subscribe annotation;
> - Pass itself to an EventBus instance's register(Object) method.
>
> **Posting Events**
>
> To post an event, simply provide the event object to the post(Object) method. The EventBus instance will determine the type of event and route it to all registered listeners.
>
> Events are routed based on their type — an event will be delivered to any subscriber for any type to which the event is assignable. This includes implemented interfaces, all superclasses, and all interfaces implemented by superclasses.
>
> When post is called, all registered subscribers for an event are run in sequence, so subscribers should be reasonably quick. If an event may trigger an extended process (such as a database load), spawn a thread or queue it for later. (For a convenient way to do this, use an AsyncEventBus.)
>
> Subscriber Methods
>
> Event subscriber methods must accept only one argument: the event.
>
> Subscribers should not, in general, throw. If they do, the EventBus will catch and log the exception. This is rarely the right solution for error handling and should not be relied upon; it is intended solely to help find problems during development.
>
> The EventBus guarantees that it will not call a subscriber method from multiple threads simultaneously, unless the method explicitly allows it by bearing the AllowConcurrentEvents annotation. If this annotation is not present, subscriber methods need not worry about being reentrant, unless also called from outside the EventBus.
>
> Dead Events
>
> If an event is posted, but no registered subscribers can accept it, it is considered "dead." To give the system a second chance to handle dead events, they are wrapped in an instance of DeadEvent and reposted.
>
> If a subscriber for a supertype of all events (such as Object) is registered, no event will ever be considered dead, and no DeadEvents will be generated. Accordingly, while DeadEvent extends Object, a subscriber registered to receive any Object will never receive a DeadEvent.
>
> This class is safe for concurrent use.
>
> See the Guava User Guide article on EventBus.
> Since:
> 10.0
> Author:
> Cliff Biffle

### （二）系统流程

![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-34hhFYGHy2tAeI3jh.png)

### （三）组成部分

![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-35CNd87spqws8i8Xi.png)

#### 1.调度器

EventBus、AsyncEventBus 都是一个调度的角色，区别是一个同步一个异步。

- EventBus

    源码注释：Dispatches events to listeners, and provides ways for listeners to register themselves.
    
    即 EventBus分发事件（Event）给listeners处理，并且提供listeners注册自己的方法。从这里我们可以看出EventBus主要是一个调度的角色。

- AsyncEventBus

    源码注释：An {@link EventBus} that takes the Executor of your choice and uses it to dispatch events, allowing dispatch to occur asynchronously.

    即 AsyncEventBus 就是 EventBus，只不过 AsyncEventBus 使用你指定的线程池（不指定使用默认线程池）去分发事件（Event），并且是异步进行的。

**EventBus总结**

- 同步执行，事件发送方在发出事件之后，会等待所有的事件消费方执行完毕后，才会回来继续执行自己后面的代码。
- 事件发送方和事件消费方会在同一个线程中执行，消费方的执行线程取决于发送方。
- 同一个事件的多个订阅者，在接收到事件的顺序上面有不同。谁先注册到 EventBus 的，谁先执行，如果是在同一个类中的两个订阅者一起被注册到 EventBus 的情况，收到事件的顺序跟方法名有关。

**AsyncEventBus总结**
- 异步执行，事件发送方异步发出事件，不会等待事件消费方是否收到，直接执行自己后面的代码。
- 在定义 AsyncEventBus 时，构造函数中会传入一个线程池。事件消费方收到异步事件时，消费方会从线程池中获取一个新的线程来执行自己的任务。
- 同一个事件的多个订阅者，它们的注册顺序跟接收到事件的顺序上没有任何联系，都会同时收到事件，并且都是在新的线程中，**异步并发**的执行自己的任务。

#### 2.事件承载器

- Event

​	事件主体，用于承载消息。

- DeadEvent

    源码注释：Wraps an event that was posted, but which had no subscribers and thus could not be delivered, Registering a DeadEvent subscriber is useful for debugging or logging, as it can detect misconfigurations in a system's event distribution.

    即 DeadEvent 就是一个被包装的 event，只不过是一个没有订阅者无法被分发的 event。我们可以在开发时注册一个 DeadEvent，因为它可以检测系统事件分布中的错误配置。

#### 3.事件注册中心

SubscriberRegistry

源码注释：Registry of subscribers to a single event bus.
即 SubscriberRegistry 是单个事件总线（EventBus）的订阅者注册表。

#### 4.事件分发器

Dispatcher

源码注释：Handler for dispatching events to subscribers, providing different event ordering guarantees that make sense for different situations.

Note: The dispatcher is orthogonal to the subscriber's Executor. The dispatcher controls the order in which events are dispatched, while the executor controls how (i.e. on which thread) the subscriber is actually called when an event is dispatched to it.

即 Dispatcher 主要任务是将事件分发到订阅者，并且可以不同的情况，按不同的顺序分发。

Dispatcher 有三个子类，用以满足不同的分发情况

- PerThreadQueuedDispatcher

    源码注释：Returns a dispatcher that queues events that are posted reentrantly on a thread that is already dispatching an event, guaranteeing that all events posted on a single thread are dispatched to all subscribers in the order they are posted.

    When all subscribers are dispatched to using a direct executor (which dispatches on the same thread that posts the event), this yields a breadth-first dispatch order on each thread. That is, all subscribers to a single event A will be called before any subscribers to any events B and C that are posted to the event bus by the subscribers to A.

    意思是说一个线程在处理事件过程中又发布了一个事件，PerThreadQueuedDispatcher 会将后面这个事件放到最后，从而保证在单个线程上发布的所有事件都按其发布顺序分发给订阅者。**注意，每个线程都要自己存储事件的队列。**

    第二段是说PerThreadQueuedDispatcher按**广度优先**分发事件。并给了一个例子：
    代码中发布了事件A，订阅者收到后，在执行过程中又发布了事件B和事件C，PerThreadQueuedDispatcher会确保事件A分发给所有订阅者后，再分发B、C事件。

- LegacyAsyncDispatcher

    源码注释：Returns a dispatcher that queues events that are posted in a single global queue. This behavior matches the original behavior of AsyncEventBus exactly, but is otherwise not especially useful. For async dispatch, an immediate dispatcher should generally be preferable.

    意思是说LegacyAsyncDispatcher有一个全局队列用于存放所有事件，LegacyAsyncDispatcher特性与AsyncEventBus特性完全相符，除此之外没有其他什么特性。如果异步分发的话，最好用immediate dispatcher。

- ImmediateDispatcher

    源码注释：Returns a dispatcher that dispatches events to subscribers immediately as they're posted without using an intermediate queue to change the dispatch order. This is effectively a depth-first dispatch order, vs. breadth-first when using a queue.

    意思是说ImmediateDispatcher在发布事件时立即将事件分发给订阅者，而不使用中间队列更改分发顺序。这实际上是**深度优先**的调度顺序，而不是使用队列时的**广度优先**。

#### 5.订阅者

- Subscriber

    源码注释：A subscriber method on a specific object, plus the executor that should be used for dispatching events to it.

    Two subscribers are equivalent when they refer to the same method on the same object (not class). This property is used to ensure that no subscriber method is registered more than once.

    第一段意思是说，Subscriber是特定对象（Event）的订阅方法，用于执行被分发事件。第二段说当两个订阅者在同一对象 **（不是类）** 上引用相同的方法时，它们是等效的，此属性用于确保不会多次注册任何订阅者方法，主要说明会对订阅者进行判重，如果是同一个对象的同一个方法，则认为是同一个订阅者，不会进行重复注册。

- SynchronizedSubscriber

    源码注释：Subscriber that synchronizes invocations of a method to ensure that only one thread may enter the method at a time.

    意思是说同步方法调用以确保一次只有一个线程可以执行订阅者方法（线程安全）

## 二、原理解析

### （一）主体流程

- listener 通过EventBus进行注册。

- SubscriberRegister 会根据listener、listener中含有【@Subscribe】注解的方法及各方法参数创建Subscriber 对象，并将其维护在Subscribers（ConcurrentMap类型，key为event类对象，value为subscriber集合）中。

- publisher发布事件Event。

- 发布Event后，EventBus会从SubscriberRegister中查找出所有订阅此事件的Subscriber，然后让Dispatcher分发Event到每一个Subscriber。

流程如下：

![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-35ZoCIOMPMZ3XFjO35.png)

### （二）listener注册原理

#### 2.1listener注册流程

1. 缓存所有含有@Subscribe注解方法到subscriberMethodsCache（LoadingCache<Class<?>, ImmutableList>， key为listener，value为method集合）。
2. listener注册。

![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-35vFAQZ28dWGeyCzk.png)

#### 2.2原理分析

- 获取含有@Subscribe注释的方法进行缓存
    找到所有被【@Subscribe】修饰的方法，并进行缓存
    **注意！！！这两个方法被static修饰，类加载的时候就进行寻找**
    ![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-363632jcSA363HmIqDrO.png)
    ![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-36qD8hcLnttDbTKkH.png)

订阅者唯一标识是【方法名+入参】
![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-36sYr36D8wm728xLok.png)

- 注册订阅者
    1.注册方法
    ![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-37CKeAug118jTsL3HR.png)
    创建Subscriber时，如果method含有【@AllowConcurrentEvents】注释，则创建SynchronizedSubscriber，否则创建Subscriber
    ![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-37FTLB9ojhgA6ajJ37.png)
    2、获取所有订阅者
    ![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-37BNnJadaGDajnlv7.png)
    3、从缓存中获取所有订阅方法
    ![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-37uJjWQomNFoAXoOc.png)

### （三）Event发布原理

### 3.1发布主体流程

- publisher 发布事件Event。
- EventBus 根据Event 类对象从SubscriberRegistry中获取所有订阅者。
- 将Event 和 eventSubscribers 交由Dispatcher去分发。
- Dispatcher 将Event 分发给每个Subscribers。
- Subscriber 利用反射执行订阅者方法。

图中画出了三个Dispatcher的分发原理。
![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-38gUVcxWJdDxgOepS.png)

#### 3.2.原理分析

- 创建缓存
    缓存EventMsg所有超类
    **注意！！！此处是静态方法，因此在代码加载的时候就会缓存Event所有超类。**
    ![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-38jBbYGOy3KV38Psh25.png)
- 发布Event事件
    此方法是发布事件时调用的方法。
    ![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-38SqAi2TbgQlQ83Vp.png)
- 获取所有订阅者
    1、从缓存中获取所有订阅者
    ![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-40kEMWecEv7Jwl3KQ.png)
    2、获取Event超类
    ![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-40mTBBcuaNTePiD40q.png)
- 事件分发
    1、分发入口
    ![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-41ZIdeqdF2iBYzcOz.png)
    2、分发器分发
    2.1、ImmediateDispatcher
    来了一个事件则通知对这个事件感兴趣的订阅者。
    ![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-42cygy0wLltrwgoU2.png)
    2.2、PerThreadQueuedDispatcher（EventBus默认选项）
    在同一个线程post的Event执行顺序是有序的。用ThreadLocal<Queue> queue来实现每个线程的Event有序性，在把事件添加到queue后会有一个ThreadLocal dispatching来判断当前线程是否正在分发，如果正在分发，则这次添加的event不会马上进行分发而是等到dispatching的值为false（分发完成）才进行。
    源码如下：
    ![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-42vEYYeGWyVsd48Od0.png)
    ![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-43duAQsxYAM0iBwGS.png)
    2.3、LegacyAsyncDispatcher（AsyncEventBus默认选项）
    会有一个全局的队列ConcurrentLinkedQueue queue保存EventWithSubscriber(事件和subscriber),如果被不同的线程poll，不能保证在queue队列中的event是有序发布的。源码如下：
    ![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-43biIRe7ZsA3ZHUon.png)
    ![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-43pXjnKoCL9qJxTQp.png)
- 执行订阅者方法
    方法入口是dispatchEvent，源码如下：
    ![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-43BjSoAj3fl7ylHSY.png)
    由于Subscriber有两种，因此执行方法也有两种：
    1.Subscriber（非线程安全）
    ![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-447yKMtBuUO6S7uVS.png)
    2.SynchronizedSubscriber（线程安全）
    注意！！！执行方法会加同步锁
    ![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-44iWcrHTzrSIVhvw44.png)

# 3、使用指导

## 3.1、主要流程

![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-44EIcwn73n2fAw9m3.png)

## 3.2、流程详解

- 1、创建EventBus、AsyncEventBus Bean
    在项目中统一配置全局单例Bean（如特殊需求，可配置多例）
    ![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-44h0B7YvpUoluhSYt.png)
- 2、定义EventMsg
    设置消息载体。
    ![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-45WsFZ0rd456wCYQF8.png)
- 3、注册Listener
    注册Listener，处理事件
    ![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-45GWEuo09o7vFxTJ3.png)
    **注意! 在使用 PostConstruct注释进行注册时，需要注意子类会执行父类含有PostConstruct 注释的方法。**
- 3、事件发布
    封装统一发布事件的Bean，然后通过Bean注入到需要发布的Bean里面进行事件发布。
    ![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-45jydFrk8sCcCenv3.png)
    ![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-45j3wc7Z3l453cn9ai.png)

**此处对EventBus进行了统一封装收口操作，主要考虑的是如果做一些操作，直接改这一处就可以。如果不需要封装，可以在使用的地方直接注入EventBus即可。**

# 4、注意事项

## 4.1、循环分发事件

如果业务流程较长，切记梳理好业务流程，不要让事件循环分发。
目前EventBus没有对循环事件进行处理。

## 4.2、使用 @PostConstrucrt 注册listener

**子类在执行实例化时，会执行父类@PostConstrucrt 注释**。 如果listenerSon继承listenerFather，当两者都使用@PostConstrucrt注册订阅方法时，子类也会调用父类的注册方法进行注册订阅方法。 **由于EventBus机制，子类注册订阅方法时，也会注册父类的监听方法**
![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-46XM3CmgeO9xjpTfV.png)
Subscriber唯一标志是（listener+method），因此在对同一方法注册时，由于不是同一个listener，所以对于EventBus是两个订阅方法。
![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-46f9JVHDmpmPprhGS.png)
![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-46FWvUbRg6r9U469og.png)
![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-47KJ2EDHX0zcNMVqW.png)

因此，如果存在listenerSon、listenerFather两个listener，且listenerSon继承listenerFather。当都使用@PostConstrucrt注册时，会导致listenerFather里面的订阅方法注册两次。

## 4.3、含有继承关系的listener

当注册listener含有继承关系时，listener处理Event消息时，listener的父类也会处理该消息。

### 4.3.1、继承关系的订阅者

![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-16-14F089qFJtgwn8Xud.png)

### 4.3.2、原理

子类listener注册，父类listener也会注册
![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-5050vdLEu23zk8GfrZ.png)

## 4.4、含有继承关系的Event

如果作为参数的Event有继承关系，使用EventBus发布Event时，Event父类的监听者也会对Event进行处理。

### 4.4.1、执行结果

![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-51WxPqOJjtAW93Nxz.png)
![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-51W7dZaJqQSckySbQ.png)

### 4.4.2、原理

在分发消息的时候，会获取所有订阅者数据（Event订阅者和Event超类的订阅者），然后进行分发数据。
获取订阅者数据如下图：
![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-52iFTw6TT52nU9w52og.png)

![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-5252c6jhLokxQybOMG.png)

缓存Event及其超类的类对象，key为Event类对象。
![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-520xLNddrPzK3hnKd.png)

# 5、分享提问问题

## 问题1：PerThreadQueuedDispatcherd 里面的队列，是否是有界队列？

有界队列，最大值为 int 的最大值 （2147483647），源码如下图：

![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-528QBIATdvsKU8cfm.png)

![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-52RnFOuRGdDIzkizF.png)

![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-52IQqykEEJO7AoDK57.png)

![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-53QeG3LYWhrKdGo2b.png)

![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-53MhCR2MFFuTtTBxU.png)

![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-53JTkKEqwnalveOjz.png)

## 问题2：dispatcher 分发给订阅者是否有序？

**EventBus:同步事件总线**：
同一个事件的多个订阅者，在接收到事件的顺序上面有不同。谁先注册到EventBus的，谁先执行（由于base使用的是PostConstruct进行注册，因此跟不同Bean之间的初始化顺序有关系）。如果是在同一个类中的两个订阅者一起被注册到EventBus的情况，收到事件的顺序跟方法名有关。

**AsyncEventBus:异步事件总线**：同一个事件的多个订阅者，它们的注册顺序跟接收到事件的顺序上没有任何联系，都会同时收到事件，并且都是在新的线程中，异步并发的执行自己的任务。

## 问题3：EventBus与SpringEvent的对比？

- 使用方式比较

| 项目        | 事件     | 发布者                    | 发布方法                               | 是否异步     | 监听者                | 注册方式                  |
| ----------- | -------- | ------------------------- | -------------------------------------- | ------------ | --------------------- | ------------------------- |
| EventBus    | 任意对象 | EventBus                  | EventBus#post                          | 支持同步异步 | 注解Subscribe方法     | 手动注册EventBus#register |
| SpringEvent | 任意对象 | ApplicationEventPublisher | ApplicationEventPublisher#publishEvent | 支持同步异步 | 注解EventListener方法 | 系统注册                  |

- 使用场景比较

| 项目         | 事件区分 | 是否支持事件簇 | 是否支持自定义event | 是否支持过滤 | 是否支持事件隔离 | 是否支持事务 | 是否支持设置订阅者消费顺序 | 复杂程度 |
| ------------ | -------- | -------------- | ------------------- | ------------ | ---------------- | ------------ | -------------------------- | -------- |
| EventBus     | Class    | 是             | 是                  | 否           | 是               | 否           | 否                         | 简单     |
| Spring Event | Class    | 是             | 是                  | 是           | 否               | 是           | 是                         | 复杂     |

参考链接 https://www.cnblogs.com/shoren/p/eventBus_springEvent.html

## 问题4：EventBus的使用场景，结合现有使用场景考虑是否合适？

EventBus暂时不适用，主要有一下几个点:

- EventBus不支持事务，项目在更新、创建商品时，最好等事务提交成功后，再发送MQ消息（主要问题点）
- EventBus不支持设置同一消息的订阅者消费顺序。
- EventBus不支持消息过滤。SpringEvent支持消息过滤

# 6.项目中遇到的问题

## 6.1、问题描述

商品上架时会触发渠道分发功能，会有两步操作

- 1、创建一条分发记录，并对外发送一条**未分发状态**的商品变更消息（通过eventBus 事件发送消息）。
- 2、将分发记录改为审核中（需要审核）或审核通过（不需要审核），并对外发送一条**已分发状态**的商品变更消息（通过eventBus 事件发送消息）。

所以分发会触发两条分发状态不同的商品变更消息，**一条是未分发，另一条是已分发**。实际发送了两条分发状态相同的商品变更消息，**状态都是已分发**。

## 6.2、原因

我们先来回顾下EventBus 监听者处理事件时有三种策略，这是根本原因：

- ImmediateDispatcher：来一个事件马上进行处理。
- **PerThreadQueuedDispatcher（eventBus默认选项，项目中使用此策略）**：在同一个线程post的Event,执行的顺序是有序的。用ThreadLocal<Queue> queue来实现每个线程post的Event是有序的，在把事件添加到queue后会有一个ThreadLocal dispatching来判断当前线程是否正在分发，如果正在分发，则这次添加的event不会马上进行分发而是等到dispatching的值为false才进行。
- LegacyAsyncDispatcher（AsyncEventBus默认选项）：会有一个全局的队列ConcurrentLinkedQueue queue保存EventWithSubscriber(事件和subscriber),如果被不同的线程poll 不能保证在queue队列中的event是有序发布的。

详情可见上文中的【2.3.4、事件分发】

再看下项目中的逻辑：

```
商品自动分发在商品变更的Listener里操作。

由于当前分发操作处于商品上架事件处理过程中，因此对于添加分发记录事件不会立马处理，而是将其放入队列。

上架操作完成，分发状态变为已分发。

等上架操作完成后，商品变更Listener处理分发事件(此时有两条EventMsg，一个是添加分发记录另一个是修改分发状态)，分发状态实时查询，对于第一个分发事件，查询到的分发记录是已分发状态。

最终导致两条消息都是已分发状态。
```

## 6.3、场景复现

在handler中对静态变量进行两次+1 操作，每操作一步发送一条事件，此处假设静态变量为分发状态。
![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-540RxH6GrJonnQumZ.png)
![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-540y52wuOzKzDrnHD0.png)

## 6.4、解决办法

目前 Dispatcher 包用default 修饰，使用者无法指定Dispatcher 策略。并且 ImmediateDispatcher 使用private修饰。
![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-55bN2K2k29TXyBO6qi.png)
![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-553Neic8TdTNhiaYV.png)
![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-55RSoMbmTslSC87bm.png)

**因此目前暂无解决非同步问题，只能在业务逻辑上进行规避。**

其实可以修改源码并发布一个包自己使用，但是公司安全规定不允许这样做，只能通过业务逻辑上进行规避，下图是github上对此问题的讨论。
![image.png](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82Guava%20EventBus%EF%BC%88%E8%AE%A2%E9%98%85%E5%8F%91%E5%B8%83%E4%BA%8B%E4%BB%B6%EF%BC%89.resource/2023-02-03-15-55n3PfPRrhHoYg8Xj.png)

# 7、总结

如果项目中需要使用异步解耦处理一些事项，使用EventBus还是比较方便的。