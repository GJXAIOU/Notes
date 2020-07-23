# POJO是什么，javabean是什么，以及POJO与javabean的区别
[原文链接地址](https://blog.csdn.net/qq_27093465/article/details/52527270)

**POJO**（Plain Ordinary Java Object）简单的Java对象，实际就是普通JavaBeans，是为了避免和EJB混淆所创造的简称。
使用POJO名称是为了避免和EJB混淆起来, 而且简称比较直接. 其中**有一些属性及其getter setter方法的类,没有业务逻辑**，有时可以作为VO(value -object)或dto(Data Transform Object)来使用.当然,如果你有一个简单的运算属性也是可以的,**但不允许有业务方法**,也不能携带有connection之类的方法。
 **自身特点**
POJO是Plain OrdinaryJava Object的缩写不错，但是它通指没有使用Entity Beans的普通java对象，可以把POJO作为支持业务逻辑的协助类。
POJO实质上可以理解为简单的实体类，顾名思义POJO类的作用是方便程序员使用数据库中的数据表，对于广大的程序员，可以很方便的将POJO类当做对象来进行使用，当然也是可以方便的调用其get,set方法。POJO类也给我们在struts框架中的配置带来了很大的方便。

**实例**
POJO有一些private的参数作为对象的属性。然后针对每个参数定义了get和set方法作为访问的接口。例如：

```
public class User {
    private long id;
    private String name;
 
    public void setId(long id) {
        this.id = id;
    }
 
    public void setName(String name) {
        this.name = name;
    }
 
    public long getId() {
        return id;
    }
 
    public String getName() {
        return name;
    }
}
```

POJO对象有时也被称为Data对象，大量应用于表现现实中的对象。如果项目中使用了Hibernate框架，有一个关联的xml文件，使对象与数据库中的表对应，对象的属性与表中的字段相对应。
 **POJO与javabean的区别**
POJO 和JavaBean是我们常见的两个关键字，一般容易混淆，POJO全称是Plain Ordinary Java Object / Pure Old Java Object，中文可以翻译成：**普通Java类，具有一部分getter/setter方法的那种类就可以称作POJO**，但是JavaBean则比 POJO复杂很多， Java Bean 是可复用的组件，对 Java Bean 并没有严格的规范，理论上讲，任何一个 Java 类都可以是一个 Bean 。但通常情况下，由于 Java Bean 是被容器所创建（如 Tomcat) 的，所以 **Java Bean 应具有一个无参的构造器**，另外，通常 **Java Bean 还要实现 Serializable 接口**用于实现 Bean 的持久性。**Java Bean 是不能被跨进程访问的**。JavaBean是一种组件技术，就好像你做了一个扳子，而这个扳子会在很多地方被拿去用，这个扳子也提供多种功能(你可以拿这个扳子扳、锤、撬等等)，而这个扳子就是一个组件。一般在web应用程序中建立一个数据库的映射对象时，我们只能称它为POJO。POJO(Plain Old Java Object)这个名字用来强调它是一个普通java对象，而不是一个特殊的对象，其主要用来指代那些没有遵从特定的Java对象模型、约定或框架（如EJB）的Java对象。理想地讲，一个POJO是一个不受任何限制的Java对象（除了Java语言规范）。

**错误的认识**
POJO是这样的一种“纯粹的”JavaBean，在它里面除了JavaBean规范的方法和属性没有别的东西，即private属性以及对这个属性方法的public的get和set方法。我们会发现这样的JavaBean很“单纯”，它只能装载数据，作为数据存储的载体，而不具有业务逻辑处理的能力。
 **真正的意思**
POJO = "Plain Old Java Object"，是MartinFowler等发明的一个术语，用来表示普通的Java对象，不是JavaBean, EntityBean 或者 SessionBean。POJO不担当任何特殊的角色，也不实现任何特殊的Java框架的接口如，EJB，JDBC等等。
即**POJO是一个简单的普通的Java对象，它不包含业务逻辑或持久逻辑等，但不是JavaBean、EntityBean等，不具有任何特殊角色和不继承或不实现任何其它Java框架的类或接口。**

下面是摘自Martin Fowler个人网站的一句话：
"We wondered why people were so against using regular objects in their systems and concluded that it was because simple objects lacked a fancy name. So we gave them one, and it's caught on very nicely."－－Martin Fowler
我们疑惑为什么人们不喜欢在他们的系统中使用普通的对象，我们得到的结论是——普通的对象缺少一个响亮的名字，因此我们给它们起了一个，并且取得了很好的效果。——Martin Fowler