# 常用Java设计模式系列（5）- 简单工厂、工厂方法模式和抽象工厂模式

。

本文链接：[https://blog.csdn.net/andamajing/article/details/71171328](https://blog.csdn.net/andamajing/article/details/71171328) 

讲到设计模式，我想大家都会想到工厂模式，在之前的几篇文章中，我们没有探讨，在这篇文章中，我们就来简单的看一下。

说道工厂模式，其实并不是指一种设计模式，从标题看就知道，其实有好几种，下面我们对这几种都简单的看看。

我们假设现在我们需要生产一些产品，这些产品我们委托给工厂进行生产。

（1）简单工厂模式

简单工厂模式，简单的说就是从前有个工厂，因为规模比较小，所以在一个工厂里面有多个生产线，生产不同的产品，客户提交订单时根据订单的要求去生产不同的产品并提供给客户。

简单工厂模式涉及的角色有：

工厂角色：具体生产产品的类

抽象产品角色：具体产品需要实现的接口；

具体产品角色：具体产品，由工厂生产出来的实体对象；

下面我们通过简单的 Java 代码来简单实现一下：

抽象产品角色示例代码：

```
package com.majing.test.designpattern.factory.simplefactory;
 
public interface IProduct {
	void display();
}
```

 具体产品角色示例代码：

```
package com.majing.test.designpattern.factory.simplefactory;
 
public class ProductA implements IProduct{
 
	@Override
	public void display() {
		System.out.println("我是ProductA.");
	}
	
}
```

```
package com.majing.test.designpattern.factory.simplefactory;
 
public class ProductB implements IProduct {
 
	@Override
	public void display() {
		System.out.println("我是ProductB.");
	}
 
}}
```

 工厂角色示例代码：

```
package com.majing.test.designpattern.factory.simplefactory;
 
public class SimpleFactory {
	public static IProduct produce(String type){
		IProduct product = null;
		if("A".equals(type)){
			product = new ProductA(); 
		}else if("B".equals(type)){
			product = new ProductB();
		}else{
			return null;
		}
		return product;
	}
}
```

 测试用例示例代码：

```
package com.majing.test.designpattern.factory.simplefactory;
 
public class Client {
	public static void main(String[] args) {
		IProduct product = SimpleFactory.produce("A");
		product.display();
		product = SimpleFactory.produce("B");
		product.display();
	}
}
```

 从上面的实现可以看到，工厂是一个集中生产产品的地方，根据客户的需求（这里是生产 A 产品，还是生产 B 产品），来生产出对应的产品。

这种设计模式的缺点是，如果需要这个工厂去生产新的产品，那么就需要修改工厂类，增加新产品生产逻辑，很明显，这违背了开闭原则。但是这种模式的优点是设计简单，实现方便。

但是随着科技的发展，原本的工厂需要生产的东西越来越多，这时候，工厂厂长决定将不同的产品生产线进行独立，同一类型产品生产线由单独一个工厂进行生产，这样不同的生产线之间就不会相互影响了，不用挤在一个拥挤的老工厂里了。于是就出现了工厂方法模式。

（2）工厂方法模式

工厂方法模式，是将不同的产品生产交由不同的工厂进行生产，这样的话，当我们把一个订单交给一个工厂生产时，我们就不需要去说给我生产什么产品了，毕竟这个工厂只会生产一种产品。

工厂方法模式涉及的角色大概有：

抽象产品角色：是具体产品角色的基类，包含所有具体产品共有的特性； 

具体产品角色：具体产品角色， 

抽象工厂角色：是具体工厂角色的基类，包含着所有具体工厂共有的特性，如生产产品； 

具体工厂角色：负责生产具体产品的工厂类；

和前面一样，我们也给出相应的 Java 代码示例：

抽象产品角色示例代码：

```
package com.majing.test.designpattern.factory.factorymethod;
 
public interface IProduct {
	void display();
}
```

 具体产品角色示例代码：

```
package com.majing.test.designpattern.factory.factorymethod;
 
public class ProductA implements IProduct{
 
	@Override
	public void display() {
		System.out.println("我是ProductA.");
	}
	
}
```

```
package com.majing.test.designpattern.factory.factorymethod;
 
public class ProductB implements IProduct {
 
	@Override
	public void display() {
		System.out.println("我是ProductB.");
	}
 
}
```

 抽象工厂角色示例代码：

```
package com.majing.test.designpattern.factory.factorymethod;
 
public interface IFactory {
	IProduct produce();
}
```

 具体工厂角色示例代码：

```
package com.majing.test.designpattern.factory.factorymethod;
 
public class ProductAFactory implements IFactory {
 
	@Override
	public IProduct produce() {
		return new ProductA();
	}
 
}
```

```
package com.majing.test.designpattern.factory.factorymethod;
 
public class ProductBFactory implements IFactory {
 
	@Override
	public IProduct produce() {
		return new ProductB();
	}
 
}
```

 从上面的示例代码中，我们看到，每个工厂都只生产一类产品。这种设计模式的好处是符合开闭原则，即使后面需要生产新的产品，也不需要修改已有的工厂类，而是新建一个工厂类和新的产品。当然也存在缺点，那就是，每次有新产品时都要实现相应的产品和工厂类，类的数量会越来越多。

此时的厂长还想继续增加效益，所以做出了决定，那就是每个工厂不仅仅生产一类产品，而是生产多个产品，所有的工厂都生产多个相同种类的产品，这就引出了我们下面要说的另一种设计模式----抽象工厂模式

（3）抽象工厂模式

抽象工厂模式，是对工厂方法模式的进一步抽象，它让每个工厂都生产同一个产品簇，产品簇内包含多个产品类型。这里的示例是让每个工厂都生产 ProductA 和 ProductB。

抽象工厂模式涉及的角色和工厂方法模式一样，这里就不赘述了。下面直接来看看抽象工厂模式的实现。

抽象产品角色示例代码：

```
package com.majing.test.designpattern.factory.abstractfactory; public interface IProduct {	void display();}
```

```
package com.majing.test.designpattern.factory.abstractfactory; import com.majing.test.designpattern.factory.simplefactory.IProduct; public abstract class AbstractProductA implements IProduct {	}
```

```
package com.majing.test.designpattern.factory.abstractfactory; import com.majing.test.designpattern.factory.simplefactory.IProduct; public abstract class AbstractProductB implements IProduct { }
```

具体产品角色示例代码： 

```
package com.majing.test.designpattern.factory.abstractfactory; public class ProductA1 extends AbstractProductA { 	@Override	public void display() {		System.out.println("我是工厂1生产的ProductA!");	} }
```

```
package com.majing.test.designpattern.factory.abstractfactory; public class ProductA2 extends AbstractProductA { 	@Override	public void display() {		System.out.println("我是工厂2生产的ProductA!");	} }
```

```
package com.majing.test.designpattern.factory.abstractfactory; public class ProductB1 extends AbstractProductB { 	@Override	public void display() {		System.out.println("我是工厂1生产的ProductB!");	} }
```

```
package com.majing.test.designpattern.factory.abstractfactory; public class ProductB2 extends AbstractProductB { 	@Override	public void display() {		System.out.println("我是工厂2生产的ProductB!");	} }
```

 抽象工厂角色示例代码：

```
package com.majing.test.designpattern.factory.abstractfactory; public interface IFactory {	AbstractProductA produceA();	AbstractProductB produceB();}
```

 具体工厂角色示例代码：

```
package com.majing.test.designpattern.factory.abstractfactory; public class Factory1 implements IFactory { 	@Override	public AbstractProductA produceA() {		return new ProductA1();	} 	@Override	public AbstractProductB produceB() {		return new ProductB1();	} }
```

```
package com.majing.test.designpattern.factory.abstractfactory;  public class Factory2 implements IFactory { 	@Override	public AbstractProductA produceA() {		return new ProductA2();	} 	@Override	public AbstractProductB produceB() {		return new ProductB2();	}	}
```

 测试用例代码：

```
package com.majing.test.designpattern.factory.abstractfactory; public class Client {	public static void main(String[] args) {		IFactory factory = new Factory1();		AbstractProductA productA = factory.produceA();		AbstractProductB productB = factory.produceB();		productA.display();		productB.display();	}}
```

 至此，一个简单的抽象工厂设计模式的演示代码便结束了。直观的感觉，那就是类的数量激增啊！！！大家看下每种模式的文件数（也就是类和接口数）：

![](https://img-blog.csdn.net/20170504170458163?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbWFqaW5nZ29nb2dv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

从上截图就可以看出来，一个模式比一个模式复杂，大家还是根据自己的需要选择合适的实现吧。