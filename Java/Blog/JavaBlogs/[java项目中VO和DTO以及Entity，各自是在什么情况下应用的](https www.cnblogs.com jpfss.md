# java项目中VO和DTO以及Entity，各自是在什么情况下应用的

j2ee中，经常提到几种对象(object)，理解他们的含义有助于我们更好的理解面向对象的设计思维。
    POJO(plain old java object)：普通的java对象，有别于特殊的java对象(含继承约束等)和EJB。POJO一般只有一系列的属性和相应的get、set方法。
    PO(persistant object):持久化对象，有别于POJO,必须对应数据库中的实体。一个PO对应数据库的一条记录。持久化对象的生命周期与数据库密切相关，只能存在于connection之中，连接关闭后，PO就消失了。
    PO相对于POJO有诸多不同，比如PO中会有保存数据库entity状态的属性和方法。但是ORM(object-relation mapping)追求的目标是PO和POJO的一致，所以在程序员的日常开发中，都是将POJO作为PO使用，而将POJO转化为PO的功能交给hibernate等框架来实现。
    DTO(data transfer object):数据传输对象，以前被称为值对象(VO,value object)，作用仅在于在应用程序的各个子系统间传输数据，在表现层展示。与POJO对应一个数据库实体不同，DTO并不对应一个实体，可能仅存储实体的部分属性或加入符合传输需求的其他的属性。
    DAO(data access object):数据访问对象。提供访问数据库的抽象接口，或者持久化机制，而不暴露数据库的内部详细信息。DAO提供从程序调用到持久层的匹配。
    BO(business object):业务对象。主要是将业务逻辑封装为一个对象，该对象可以包含一个或多个其他对象。如，"Principal"(委托人)，有"Name","Age"等属性，同时和"Employee"(雇员)有1对多的关系，这个"Principal"就可以作为一个与业务相关的PO。

# 实践小结

按照标准来说：
1、entity里的每一个字段，与数据库相对应，
2、VO里的每一个字段，是和你前台页面相对应，
3、DTO，这是用来转换从entity到dto，或者从dto到entity的中间的东西。
举个例子：

你的html页面上有三个字段，name，pass，age
你的数据库表里，有两个字段，name，pass(注意没有age哦)而你的dto里，就应该有下面三个(因为对应html页面上三个字段嘛)
private string name；
private string pass;
private string age;
这个时候，你的entity里，就应该有两个(因为对应数据库表中的2个字段嘛)
private string name；
private string pass;
到了这里，好了，业务经理让你做这样一个业务“年龄大于20的才能存入数据库”

这个时候，你就要用到vo了

你要先从页面上拿到VO，然后判断dto中的age是不是大于20，如果大于20，就把dto中的

name和pass拿出来，放到vo中，然后在把DTO中的name和pass原封不懂的给entity，然后根据

entity的值，在传入数据库，这就是他们三个的区别
PS，DTO和entity里面的字段应该是一样的，DTO只是entity到VO，或者VO到entity的中间过程，如果没有这个过程，你仍然可以做到增删改查。