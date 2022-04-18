# java项目中VO和DTO以及Entity，各自是在什么情况下应用的

j2ee 中，经常提到几种对象(object)，理解他们的含义有助于我们更好的理解面向对象的设计思维。
    POJO(plain old java object)：普通的 java 对象，有别于特殊的 java 对象(含继承约束等)和 EJB。POJO 一般只有一系列的属性和相应的 get、set 方法。
    PO(persistant object):持久化对象，有别于 POJO,必须对应数据库中的实体。一个 PO 对应数据库的一条记录。持久化对象的生命周期与数据库密切相关，只能存在于 connection 之中，连接关闭后，PO 就消失了。
    PO 相对于 POJO 有诸多不同，比如 PO 中会有保存数据库 entity 状态的属性和方法。但是 ORM(object-relation mapping)追求的目标是 PO 和 POJO 的一致，所以在程序员的日常开发中，都是将 POJO 作为 PO 使用，而将 POJO 转化为 PO 的功能交给 hibernate 等框架来实现。
    DTO(data transfer object):数据传输对象，以前被称为值对象(VO,value object)，作用仅在于在应用程序的各个子系统间传输数据，在表现层展示。与 POJO 对应一个数据库实体不同，DTO 并不对应一个实体，可能仅存储实体的部分属性或加入符合传输需求的其他的属性。
    DAO(data access object):数据访问对象。提供访问数据库的抽象接口，或者持久化机制，而不暴露数据库的内部详细信息。DAO 提供从程序调用到持久层的匹配。
    BO(business object):业务对象。主要是将业务逻辑封装为一个对象，该对象可以包含一个或多个其他对象。如，"Principal"(委托人)，有"Name","Age"等属性，同时和"Employee"(雇员)有 1 对多的关系，这个"Principal"就可以作为一个与业务相关的 PO。

# 实践小结

按照标准来说：
1、entity 里的每一个字段，与数据库相对应，
2、VO 里的每一个字段，是和你前台页面相对应，
3、DTO，这是用来转换从 entity 到 dto，或者从 dto 到 entity 的中间的东西。
举个例子：

你的 html 页面上有三个字段，name，pass，age
你的数据库表里，有两个字段，name，pass(注意没有 age 哦)而你的 dto 里，就应该有下面三个(因为对应 html 页面上三个字段嘛)
private string name；
private string pass;
private string age;
这个时候，你的 entity 里，就应该有两个(因为对应数据库表中的 2 个字段嘛)
private string name；
private string pass;
到了这里，好了，业务经理让你做这样一个业务“年龄大于 20 的才能存入数据库”

这个时候，你就要用到 vo 了

你要先从页面上拿到 VO，然后判断 dto 中的 age 是不是大于 20，如果大于 20，就把 dto 中的

name 和 pass 拿出来，放到 vo 中，然后在把 DTO 中的 name 和 pass 原封不懂的给 entity，然后根据

entity 的值，在传入数据库，这就是他们三个的区别
PS，DTO 和 entity 里面的字段应该是一样的，DTO 只是 entity 到 VO，或者 VO 到 entity 的中间过程，如果没有这个过程，你仍然可以做到增删改查。