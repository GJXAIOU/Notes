## 徒手撸一个简单的IOC



Spring 框架中最经典的两个就是 IOC 和 AOP，其中 IOC(Inversion of Control)是什么呢？控制反转，简单来说就是将控制实体 Bean 的动作交给了 Spring 容器进行管理。

推荐相关阅读：[透彻理解Spring事务设计思想之手写实现](http://mp.weixin.qq.com/s?__biz=MzAxMjEwMzQ5MA==&mid=2448885967&idx=2&sn=a8634cafe753e1c6c18673917a5bfe92&chksm=8fb552e2b8c2dbf4223538769af6e40a363b3903849213cb67cef206d361e32aabbcecd58889&scene=21#wechat_redirect)

再简单点来说就是例如之前想用一个类，必须 new 一个，但是使用了 Spring 那么直接用@Autowired 注解或者用 xml 配置的方式就能直接获得此对象，而且你也不用管它的生命周期啊等等之类的。就不用自己 new 一个对象了。

![](https://mmbiz.qpic.cn/mmbiz/JfTPiahTHJhrHYNfLpkVoZyib4kPnZe9BC8y2wnb7NzERibH04jjnbh8jo1tU9WNz2NNibF4bGmEDZgPibanY60v5Qw/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

如果是之前没有使用 IOC 的话，那么这些对象的创建以及赋值都是由我们自己创建了，下面简单的演示了如果有上面四个对象依赖的话，那么没有 IOC 我们必须要创建对象并且赋值。仅仅四个对象就这么多，那么一旦项目大了，对象成百上千，如果还这样写的话，那么绝对是一场灾难。

```
对象A a = new 对象A();
对象B b = new 对象B();
对象C c = new 对象C();
对象D d = new 对象D();
a.setB(b);
a.setC(c);
b.setD(d);
c.setD(d);
```

因此在 Spring 中通过 IOC 将所有的对象统一放到 Spring 的容器中进行管理，所以就简单了很多。上面的实例化对象的代码也不需要我们写了。

![](https://mmbiz.qpic.cn/mmbiz/JfTPiahTHJhrHYNfLpkVoZyib4kPnZe9BCza5ewoesSTN8eXGib9iapSicf4zic5Isldzh7Qxefgzbbib6FnzWtR2UZUQ/640?wx_fmt=other&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

上面说了那么多，其实就是一句话 IOC 非常重要，但是如果直接看 Spring 源码的话会非常懵逼，所以就简单的写一个 IOC 的小例子来理解这种思想。

## 分析并编写代码

还是编写代码前的分析阶段，Spring 的 IOC 其实就是将所有的 Bean 放在统一容器中进行管理起来，然后在在获取的时候进行初始化，所以需要我们在程序启动的时候将被标记的类进行存储在自定义的容器中管理。

*   初始化阶段：将被@MyIoc 类似于 Spring 中@Service 标记的类放入到自定义的容器中。

*   使用：通过自定义的获取 Bean 的类进行统一获取。

现在我们就以上面两个步骤进行详细点的分析

### 数据准备阶段

首先初始化阶段我们要先建立两个注解类用于类的发现(@MyIoc 类似于@Service)。

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyIoc {

}
```

然后要初始化信息进自定义容器的话用什么类型的容器去存储这些信息呢？这里可以想到是用 Map 来存，用 key 为类名，value 用什么呢？value 就是要放在容器中进行管理的类的信息了，那么一个类有什么信息呢即类是由什么组成呢？有以下几个信息

*   类名
*   构造函数
*   属性值
*   父类

所以根据上面的分析我们可以建立一个实体类来存储这些信息，此时我们就不考虑复杂的构造函数了，就都是初始化的无参构造函数。然后父类的属性就不进行分析注入了。所以此时类实体类就简单了。

```
@Data
public class BeanDefinition {
    private String className;
    private String alias;
    private String superNames;
}
```

### 初始化阶段

有了存储类信息的类了，那么我们在程序启动的时候就应该将这些信息给加载到 Map 中，此时建立一个启动类用于初始化被@MyIoc 标记的类的信息。

```
@Component
@Order(value = 1)
public class IoCInitConifg implements CommandLineRunner{

    @Override
    public void run(String... args){
        ConcurrentHashMap<String,BeanDefinition> concurrentHashMap = new ConcurrentHashMap<>();
        Reflections reflections = new Reflections();
        //获得项目中所有被MyIoc标记得类
        Set<Class<?>> typesAnnotatedWith = reflections.getTypesAnnotatedWith(MyIoc.class);
        //将其信息初始进自定义容器MyBeanFactory中
        for (Class clazz : typesAnnotatedWith){
            BeanDefinition beanDefinition = new BeanDefinition();
            String className = clazz.getName();
            String superclassName = clazz.getSuperclass().getName();
            beanDefinition.setClassName(className);
            beanDefinition.setSuperNames(superclassName);
            beanDefinition.setAlias(getClassName(className));
            concurrentHashMap.put(className,beanDefinition);
        }
        MyBeanFactoryImpl.setBeanDineMap(concurrentHashMap);
    }

    private String getClassName(String beanClassName) {
        String className = beanClassName.substring(beanClassName.lastIndexOf(".") + 1);
        className = className.substring(0, 1).toLowerCase() + className.substring(1);
        return className;
    }
}
```

此时得说一下自定义的统一容器管理的类 MyBeanFactory 此类用作统一获得类的途径

```
public interface MyBeanFactory {
    Object getBeanByName(String name) throws Exception;
}
```

此时还有其实现类

```
@Log4j
public class MyBeanFactoryImpl implements MyBeanFactory{
    //存储对象名称和已经实例化的对象映射
    private static ConcurrentHashMap<String,Object> beanMap = new ConcurrentHashMap<>();
    //存储对象名称和对应对象信息的映射
    private static ConcurrentHashMap<String,BeanDefinition> beanDefineMap= new ConcurrentHashMap<>();
    //存储存储在容器中对象的名称
    private static Set<String> beanNameSet = Collections.synchronizedSet(new HashSet<>());

    @Override
    public Object getBeanByName(String name) throws Exception {
        //看有没有已经实例化的对象,有的话就直接返回
        Object object = beanMap.get(name);
        if (object != null){
            return object;
        }
        //没有的话就实例化一个对象
        object = getObject(beanDefineMap.get(name));
        if (object != null){
            //对实例化中对象的注入需要的参数
            setFild(object);
            //将实例化的对象放入Map中,便于下次使用
            beanMap.put(name,object);
        }
        return object;
    }

    public void setFild(Object bean) throws Exception {
        Field[] declaredFields = bean.getClass().getDeclaredFields();
        for (Field field: declaredFields){
            String filedAllName = field.getType().getName();
            if (beanNameSet.contains(filedAllName)){
                Object findBean = getBeanByName(filedAllName);
                //为对象中的属性赋值
                field.setAccessible(true);
                field.set(bean,findBean);
            }
        }
    }

    public Object getObject(BeanDefinition beanDefinition) throws Exception {
        String className = beanDefinition.getClassName();
        Class<?> clazz = null;
        try {
            clazz = Class.forName(className);
        } catch (ClassNotFoundException e) {
            log.info("can not find bean by beanName: "+className);
            throw new Exception("can not find bean by beanName: "+className);
        }
        return clazz;
    }

    public static void setBeanDineMap(ConcurrentHashMap<String,BeanDefinition> beanDefineMap){
        MyBeanFactoryImpl.beanDefineMap = beanDefineMap;
    }

    public static void setBeanNameSet(Set<String> beanNameSet){
        MyBeanFactoryImpl.beanNameSet = beanNameSet;
    }

}
```

此时初始化的阶段已经完成了，即已经将所有被@MyIoc 标记的类已经被全部存放在了自定义的容器中了。其实在这里我们已经能使用自己的自定义的容器进行获得 Bean 了。

```
@MyIoc
@Data
public class User {
    private Student student;
}
```

```
@MyIoc
public class Student {
    public String play(){
        return "student"+ this.toString();
    }
}

```

此时我们在启动类中写如下

```
   User user1 = (User)beanFactory.getBeanByName("com.example.ioc.domain.User");
        User user2 = (User)beanFactory.getBeanByName("com.example.ioc.domain.User");
        Student student1 = user1.getStudent();
        Student student2 = user1.getStudent();
        Student student3 = (Student)beanFactory.getBeanByName("com.example.ioc.domain.Student");
        System.out.println(user1);
        System.out.println(user2);
        System.out.println(student1);
        System.out.println(student2);
        System.out.println(student3);
```

发现控制台中输出的对象都是同一个对象，并且在 User 中也自动注入了 Student 对象。此时一个简单的 IOC 就完成了。

```
User(student=com.example.ioc.domain.Student@705e7b93)
User(student=com.example.ioc.domain.Student@705e7b93)
com.example.ioc.domain.Student@705e7b93
com.example.ioc.domain.Student@705e7b93
com.example.ioc.domain.Student@705e7b93
```

## 项目地址

 https://github.com/modouxiansheng/Doraemon

## 参考

 https://juejin.im/post/5a5875a4518825733a30a463
 https://juejin.im/entry/599f8ba6518825241f788ad1