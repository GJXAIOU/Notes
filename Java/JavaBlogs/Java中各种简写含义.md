---
tags: [DO, DTO, BO, AO, VO, POJO,Query,DAO POJO]
---

# Java中各种简写含义


层领域模型规约：

*   DO（ Data Object）：与数据库表结构一一对应，通过DAO层向上传输数据源对象。就是从现实世界中抽象出来的有形或无形的业务实体
*   DTO（ Data Transfer Object）：数据传输对象，Service或Manager向外传输的对象。泛指用于展示层与服务层之间的数据传输对象。
*   BO（ Business Object）：业务对象。 由Service层输出的封装业务逻辑的对象。从业务模型的角度看 , 见 UML 元件领域模型中的领域对象。封装业务逻辑的 java 对象 , 通过调用 DAO 方法 , 结合 PO,VO 进行业务操作。 business object: 业务对象 主要作用是把业务逻辑封装为一个对象。这个对象可以包括一个或多个其它的对象。 比如一个简历，有教育经历、工作经历、社会关系等等。 我们可以把教育经历对应一个 PO ，工作经历对应一个 PO ，社会关系对应一个 PO 。 建立一个对应简历的 BO 对象处理简历，每个 BO 包含这些 PO 。 这样处理业务逻辑时，我们就可以针对 BO 去处理。
*   AO（ Application Object）：应用对象。 在Web层与Service层之间抽象的复用对象模型，极为贴近展示层，复用度不高。
*   VO（ View Object）：显示层对象，通常是Web向模板渲染引擎层传输的对象。它的作用是把某个指定页面（或组件）的所有数据封装起来。
*   POJO（ Plain Ordinary Java Object）：在本手册中， POJO专指只有setter/getter/toString的简单类，包括DO/DTO/BO/VO等。
*   Query：数据查询对象，各层接收上层的查询请求。 注意超过2个参数的查询封装，禁止使用Map类来传输。

-  PO(persistant object) 持久对象
在 o/r 映射的时候出现的概念，如果没有 o/r 映射，没有这个概念存在了。通常对应数据模型 ( 数据库 ), 本身还有部分业务逻辑的处理。可以看成是与数据库中的表相映射的 java 对象。最简单的 PO 就是对应数据库中某个表中的一条记录，多个记录可以用 PO 的集合。 PO 中应该不包含任何对数据库的操作。

- TO(Transfer Object) ，数据传输对象
在应用程序不同 tie( 关系 ) 之间传输的对象

- DAO(data access object) 数据访问对象
是一个 sun 的一个标准 j2ee 设计模式， 这个模式中有个接口就是 DAO ，它负持久层的操作。为业务层提供接口。此对象用于访问数据库。通常和 PO 结合使用， DAO 中包含了各种数据库的操作方法。通过它的方法 , 结合 PO 对数据库进行相关的操作。夹在业务逻辑与数据库资源中间。配合 VO, 提供数据库的 CRUD 操作


领域模型命名规约：

*   数据对象：xxxDO，xxx即为数据表名。
*   数据传输对象：xxxDTO，xxx为业务领域相关的名称。
*   展示对象：xxxVO，xxx一般为网页名称。
*   POJO是DO/DTO/BO/VO的统称，禁止命名成xxxPOJO。



