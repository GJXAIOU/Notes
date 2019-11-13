# 第三章：UML 类图

## 一、UML 基本介绍

- UML——Unified  modeling  language  UML (统一建模语言)，是一种用于软件系统分析和设计的语言工具，它用于帮助软件开发人员进行思考和记录思路的结果。

- UML 本身是一套符号的规定，就像数学符号和化学符号一样，这些符号用于描述软件模型中的各个元素和他们之间的关系，比如类、接口、实现、泛化、依赖、组合、聚合等。

- 使用 UML 来建模，常用的工具有 Rational Rose , 也可以使用一些**插件**来建模

## 二、UML 图

![UML 各个组件功能]($resource/UML%20%E5%90%84%E4%B8%AA%E7%BB%84%E4%BB%B6%E5%8A%9F%E8%83%BD.png)

画 UML 图与写文章差不多，都是把自己的思想描述给别人看，关键在于思路和条理。

### （一）UML 图分类

- 用例图(use  case)
- 静态结构图：**类图**、对象图、包图、组件图、部署图
- 动态行为图：交互图（时序图与协作图）、状态图、活动图

**说明**

- **类图是描述类与类之间的关系的，是 UML 图中最核心的**；
- 在讲解设计模式时，我们必然会使用类图，为了让学员们能够把设计模式学到位，需要先给大家讲解类图；
- 温馨提示：如果已经掌握 UML 类图的学员，可以直接听设计模式的章节；

## 三、UML 类图

- 用于描述系统中的**类(对象)本身的组成和类(对象)之间的各种静态关系**。
- 类之间的关系：**依赖、泛化（继承）、实现、关联、聚合与组合**。

- 类图简单举例

```java
public class Person{ //代码形式->类图
    private Integer id;
    private String name;

    public void setName(String name){ 
         this.name=name;
    }

    public String  getName(){ 
        return  name;
    }
}
```

![对应的类图]($resource/%E5%AF%B9%E5%BA%94%E7%9A%84%E7%B1%BB%E5%9B%BE.png)

### （一）类图—依赖关系（Dependence）

只要是在**类中用到了对方，那么他们之间就存在依赖关系**。如果没有对方，连编绎都通过不了。

```java
package com.atguigu.uml.dependence;

public class PersonServiceBean {
	private PersonDao personDao;// 类

	public void save(Person person) {
	}

	public IDCard getIDCard(Integer personid) {
		return null;
	}

	public void modify() {
		Department department = new Department();
	}

}
// 下面依赖类中具体实现没有写出
public class PersonDao{} 
public class IDCard{} 
public class Person{} 
public class  Department{}
```

![PersonServiceBean 2]($resource/PersonServiceBean%202.png)

**小结**

- 类中用到了对方
- 如果是**类的成员属性**
- 如果是**方法的返回类**型
- 是方法**接收的参数类**型
- 方法中使用到

### （二）类图—泛化关系(generalization）

- **泛化关系实际上就是继承关系**，他是**依赖关系的特例**

```java
package com.atguigu.uml.generalization;

public abstract class DaoSupport{
	public void save(Object entity){
	}
	public void delete(Object id){
	}
}

// 下面类继承了上面的类
public class PersonServiceBean extends DaoSupport {

}
```

![DaoSupport]($resource/DaoSupport.png)

**小结**:

- 泛化关系实际上就是继承关系
- 如果 A 类继承了 B 类，我们就说 A 和 B 存在泛化关系

### （三） 类图—实现关系（Implementation）

实现关系实际上就**是A类实现B接口**，他是**依赖关系的特**例

```java
public interface PersonService {
	public void delete(Integer id);
}

// 该方法实现了上面的接口
public class PersonServiceBean implements PersonService{
   @Override
  public void delete(Integer id) {
      System.out.println("delete..");
   }
}
```

![implementation]($resource/implementation.png)

### （四）类图—关联关系（Association）

关联关系实际上就是类与类之间的联系，是**依赖关系的特例**。

关联具有**导航性**：即双向关系和单向关系；
关系具有多重性：如 `1`：表示有且只有一个，`0...`：表示 0 个或者多个，`0，1`：表示 0 个或者 1 个，`n，m` :表示 n 到 m 个都可以； `m...` ：表示至少 m 个。

例如单向一对一关系：

```java
public class Person{
    private ID id;
}

public class ID {
}
```

双向一对一关系：

```java
public class Person{
    private ID id;
}

public class ID{
    private Person person;
}
```

两者的 UML 图为：

![association1]($resource/association1.png)
![association2]($resource/association2.png)

### （五）类图—聚合关系（Aggregation）

**基本介绍**

聚合关系（Aggregation）表示的是**整体和部分的关系**，**整体与部分可以分开**。聚合关系是**关联关系的特例**， 所以他具有关联的**导航性与多重性**。
导航性：谁指向谁； 多重性：是否有多个；

如：一台电脑由键盘(keyboard)、显示器(monitor)，鼠标等组成；组成电脑的各个配件是可以从电脑上分离出来的，使用带空心菱形的实线来表示：

**应用实例** 
如果电脑和鼠标、显示器可以分开就是聚合关系，如果不可以分开就是组合关系

![aggregation]($resource/aggregation.png)

### （六）类图—组合关系（Composition）

==这里理论和最后的类图不符合==
**基本介绍**
这是代码中：因为使用 new ，就是组合关系，就是computer创建的时候，两个属性同时创建了

组合关系：也是整体与部分的关系，但是整体与部分不可以分开。

再看一个案例：在程序中我们定义实体：Person 与 IDCard、Head, 那么 Head 和 Person 就是 组合，IDCard 和 Person 就是聚合。

但是如果在程序中 Person 实体中定义了对 IDCard 进行级联删除，即删除 Person 时连同 IDCard 一起删除，那么 IDCard 和 Person 就是组合了.

```java
public class Person {
    //聚合关系
    private IDCard card;
    //组合关系
    private Head head = new Head();
}

public class Head {
}
public class IDCard {
}
```

```java
public class Computer {
	private Mouse mouse = new Mouse(); //鼠标可以和computer不能分离
	private Moniter moniter = new Moniter();//显示器可以和Computer不能分离
	public void setMouse(Mouse mouse) {
		this.mouse = mouse;
	}
	public void setMoniter(Moniter moniter) {
		this.moniter = moniter;
	}
}

public class Moniter {
}
public class Mouse {
}
```
