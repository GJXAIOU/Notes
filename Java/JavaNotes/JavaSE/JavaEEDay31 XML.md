---
tags:
- XML
---

# JavaEEDay31 XML

@toc

## 一、前言
- HTML：HyperText Markup Language 超文本标记语言 ，不经过任何的编译，浏览器通过标记进行对应的响应；

- CSS： 层级样式表；

- JavaScript： 让页面完成一些动态的特效；

HTML+CSS+JavaScript：用来制作静态网站

HTML 由标签组成，不区分大小写，是 W3C 组织制定的规范，所有的 HTML 标签都是确定的，固定的，不能自己创建，大约 100 多个

## 二、 XML 概念

- XML：Extend Markup Language 可拓展的标记语言
- XML 也是通过便签来组成语言，但是这些标签程序员可以自定义，但是要符合语法规定，同时标签是严格区分大小写的；

- 通常的使用方式：`<自定义标签>数据</自定义标签>`

- **使用场景**：
  - 1.**properties 文件**，采用键值对保存的（key - value），用于作为配置文件，例如：Tomcat 服务器配置文件和 Spring、SpringMVC、MyBatis 配置文件
    - 例如：
       name = root
       password = 12345
     对应的 XML 标签：
 
```xml
<User>
  <name>root</name>
  <password>12345</password>
</User>
```

2.**作为小型数据库**，是数据的载体；

## 三、XML 语法规范

### （一）文档声明
这是固定的格式：
`<?xml version = "1.0" encoding = "utf-8"？>`
其中：version： XML 使用的版本号
encoding: 解析当前 XML 文件使用的字符编码

### （二）标签语法
- 基本格式：`<自定义标签名>数据</自定义标签名>`
- 语法规范：
  - 结束标签必须有`/`进行标记；
  - `<student />` 为空标签，没有内容，一般用于占位；
  - XML 文件中使用的自定义标签是严格区分大小写的；
  - XML 文件中使用的标签必须一一匹配，不能交叉嵌套；
  - XML 文件中标签不能存在空格；
  - XML 文件中使用的自定义标签不能使用数字开头；
  - XML 文件中有且只能有一个根节点；☆☆☆




### （三）属性 
格式 示例：`<student name = "nnn"> </student>`
注意：
- 属性必须使用引号包含，尽量使用双引号；
- 一个标签内可以使用多个属性，但是属性的名字不能相同；

### （四）注释
格式：`<!-- 这里是注释-->`
- 其他注释汇总：
  - Java： `//` `/* */` `/** */ `单行注释、多行注释、文档注释
  - HTML: `<!-- -->`
  - CSS :`/* */`
  - JS: `//` `/* */`
  - JSP: `<%  %>`

### （五）转义字符
XML 中有很多特殊含义的字符，例如：<> ? “”
对应的转义字符：
|   字符|   转义字符   |  描述   |
|--------|-------|-----------   |
|  &     |`&amp;`  |  和         |
|   <    |`&lt;`   | 小于号      |
|   >    |`&gt;`   |  大于号     |
|   "    |`&quot;` |   双引号    |
|   '    |`&apos;` | 单引号      |



## 三、XML 解析
将 XML 文件解析到 Java 中

![DOM]($resource/DOM.jpg)

- XML 文件解析方式：
  - DOM 解析；
  - SAX 解析；
- XML 解析常用的工具：
  - DOM 解析：针对 XML 文件，可读可写可修改
    - JAXP（sun 公司官方，不好用）
    - JDOM（非官方，还行）
    - Dom4j(非官方，好用) ☆☆是三大框架默认使用功能 XML 解析方式；
  - SAX 解析：针对 XML 文件，只可读
    - SAX 解析工具（官方，常用于 Android 开发），了解即可； 
- 一般借助于 Dom4j 工具使用：
  

### （一）XML 解析示例
下面代码分为三个文件
- contact.xml  :需要解析的 xml 文件
- ParsingElementNode.java :Dom4j 中常见方法是的使用 Demo
- TrueUse.java :真正的用于解析 contact.xml 文件的代码，即实现 XML 文件解析到 Java 中；

contact.xml
```xml
<?xml version = "1.0" encoding = "utf-8" ?>

	
<ContactList>
		<contact id = "1" test = "1" tag = "2">
			<name>张三</name>
			<gender>男</gender>
			<tel>666666</tel>
			<age>18</age>
			<qq>1111111</qq>
			<email>1111111@qq.com</email>
		</contact>
		<contact id = "2">
			<name>李四</name>
			<gender>女</gender>
			<tel>8888888</tel>
			<age>17</age>
			<qq>2222222</qq>
			<email>2222222@qq.com</email>
	    </contact>
	<haha></haha>
	
</ContactList>
```

ParsingElementNode.java
```java
import org.dom4j.*;
import org.dom4j.io.SAXReader;
import org.junit.jupiter.api.Test;

import java.io.File;
import java.util.Iterator;
import java.util.List;

/**获取XML文件的结点信息，包括：
 * Node结点；
 * Element结点
 * Attribute结点
 * text文本结点
 *
 * @author GJXAIOU
 * @create 2019-07-27-16:19
 */
public class ParsingElementNode {
    //获取节点信息，最原始的方式
    @Test
    public void XMLNode() throws DocumentException {

        /*
        //1.创建一个XML文件的解析器
        SAXReader saxReader = new SAXReader();

        //2.读取XML文件，得到XML文件的Document对象
        Document read = saxReader.read(new File("E:\\Program\\Java\\study\\code\\Day30\\Day30\\contact.xml"));
         */

        //1.创建XML文档的解析器，返回Document对象(将上面代码写成一行如下)
        Document document = new SAXReader().read(new File("E:\\Program\\Java\\study\\code\\Day30\\Day30\\contact.xml"));

        //示例：nodeIterator // 得到当前结点下的所有子节点，不能跨界（不能往里读）
        Iterator<Node> nodeIterator = document.nodeIterator();

        //之前使用的Iterator迭代器中方法都可以使用，因为它是一个接口： hasNext()   next() remove()
        while (nodeIterator.hasNext()){
            Node node = nodeIterator.next(); //得到一个Node类型的结点，所有的XML文件结点中都是Node结点
            String nodeName = node.getName(); //获取节点的名字
            System.out.println(nodeName); //☆☆☆得到的是根节点名称


            /*
            以上的输出显示：在XML文件中，有一些标签是没有子节点的，这些标签也不是Element标签
            这里需要进行过滤，如果是Element标签就继续解析
            使用：Instanceof  作用是判断当前对象是不是指定类的对象；

             */

            //如果是一个标签结点，我们就继续解析；
            if (node instanceof Element){
                Element ele = (Element) node;

                Iterator<Node> it2 = ele.nodeIterator();

                while (it2.hasNext()){
                    Node node2 = it2.next();
                    System.out.println(node2.getName());
                }
            }
        }
        System.out.println("***********************************");
    }



//--------------------------------------------------

    //方法二：使用递归，遍历所有的XML文件的结点

    public void XMLNode2() throws DocumentException {
        //1.创建XML文档的解析器，返回Document对象
        Document document = new SAXReader().read(new File("E:\\Program\\Java\\study\\code\\Day30\\Day30\\contact.xml"));

        //2.获取根节点
        Element rootElement = document.getRootElement();

        //3.调用递归方法,遍历整个XML文件
        getChildNode(rootElement);
        System.out.println("***********************************");
    }



    private void getChildNode(Element element){
        System.out.println(element.getName()); //看当前解析的什么对象

        Iterator<Node> nodeIterator = element.nodeIterator();
        while (nodeIterator.hasNext()) {
            Node node = nodeIterator.next();

            if (node instanceof Element){
                Element node1 = (Element) node;
                getChildNode(node1);
            }
        }

    }



//--------------------------------------------------

    /**
     * 获取标签
     */
    @Test
    public void XMLElement() throws DocumentException {

        //1.创建XML解析器，获取到Document对象
        Document document = new SAXReader().read(new File("E:\\Program\\Java\\study\\code\\Day30\\Day30\\contact.xml"));

        //2.获取根节点
        Element rootElement = document.getRootElement();

        //3.获取当前节点下的指定名字的结点，如果有多个名字相同，拿到的是第一个节点
        Element contact = rootElement.element("contact");
        System.out.println(contact.attributeValue("id"));

        //4.获取当前结点下指定名字节点的所有子节点迭代器,即能把两个ContactList都拿到，并且用迭代器操作
        //这里相当于获取两个
        Iterator<Element> elementIterator = rootElement.elementIterator("contact");

        while (elementIterator.hasNext()) {
            Element element = elementIterator.next();
            System.out.println(element.attributeValue("id"));
        }

        //5.获取当前结点下的所有子节点
        List<Element> elements = rootElement.elements();
        for (Element element : elements) {
            System.out.println(element.getName());
        }
    }


    /**
     * 获取属性
     */
    @Test
    public void XMLAttribute() throws DocumentException {
        //1.读取XML文件，获取Document对象
        Document document = new SAXReader().read(new File("E:\\Program\\Java\\study\\code\\Day30\\Day30\\contact.xml"));

        //获取属性值方式一：
        //2.获取属性前提：必须获取到属性所在标签的节点（当前只有contact标签有属性）
        Element element = document.getRootElement().element("contact");
        String value = element.attributeValue("id"); //所有属性的值都是String类型
        System.out.println(element.getName() + ":" + value);

        //获取属性值方式二：
        Attribute idAttr = element.attribute("id");
        System.out.println(idAttr.getName() + ":" + idAttr.getValue());

        //获取属性值方式三：获取指定节点里面所有属性节点的List集合
        List<Attribute> listAttr = element.attributes();
        for (Attribute attribute : listAttr) {
            System.out.println(attribute.getName() + ":" + attribute.getValue());
        }

        //获取属性值方式四：获取指定节点里面所有属性节点的迭代器
        Iterator<Attribute> attributeIterator = element.attributeIterator();
        while (attributeIterator.hasNext()){
            Attribute next = attributeIterator.next();
            System.out.println(next.getName() + ":" + next.getValue());
        }

    }


    /**
     * 获取文本结点
     */
    @Test
    public void XMLText() throws DocumentException {
        //1.读取XML文件，获取Document对象
        Document document = new SAXReader().read(new File("E:\\Program\\Java\\study\\code\\Day30\\Day30\\contact.xml"));

        //2.获取根节点
        Element rootElement = document.getRootElement();

        Element element = rootElement.element("contact").element("name");
        System.out.println(element.getName() + ":" + element.getText());

    }
}

```
上面方法测试 Demo 结果：
```java
ContactList
null
contact
null
contact
null
haha
null
***********************************
ContactList
contact
name
gender
tel
age
qq
email
contact
name
gender
tel
age
qq
email
haha
***********************************
1
1
2
contact
contact
haha
***********************************
contact:1
id:1
id:1
test:1
tag:2
id:1
test:1
tag:2
***********************************
name:张三
```

TrueUse.java
```java
import org.dom4j.Document;
import org.dom4j.DocumentException;
import org.dom4j.Element;
import org.dom4j.io.SAXReader;

import java.io.File;
import java.util.ArrayList;
import java.util.Iterator;

/**将XML里面的数据读取到Contact对象里面
 * @author GJXAIOU
 * @create 2019-07-27-17:45
 */
public class TrueUse {
    public static void main(String[] args) throws DocumentException {
        ArrayList<Contact> contacts = new ArrayList<>();

        //获取到XML对应的Document对象
        Document doc = new SAXReader().read(new File("E:\\Program\\Java\\study\\code\\Day30\\Day30\\contact.xml"));
        Iterator<Element> it = doc.getRootElement().elementIterator("contact"); //拿到根节点下面contact的迭代器

        while (it.hasNext()){
            Element contactElem = it.next();
            Integer id = Integer.valueOf(contactElem.attributeValue("id")); //拿到id的值，并将string类型强转为int类型
            String name = contactElem.elementText("name"); //这里直接拿文本就行
            char gender = contactElem.elementText("gender").charAt(0);
            String tel = contactElem.elementText("tel");
            Integer age = Integer.valueOf(contactElem.elementText("age"));
            String qq = contactElem.elementText("qq");
            String email = contactElem.elementText("email");

            Contact contact = new Contact(id, age, gender, tel, qq, name, email);
            contacts.add(contact);

        }

        //展示数据
        for (Contact contact : contacts) {
            System.out.println(contact);
        }

    }
}

```
以上程序结果：
```java
Contact{id=1, age=18, gender=男, tel='666666', qq='1111111', name='张三', email='1111111@qq.com'}
Contact{id=2, age=17, gender=女, tel='8888888', qq='2222222', name='李四', email='2222222@qq.com'}
```







