# 理解和正确使用Java中的断言(assert)

##  一、语法形式：
    Java2在1.4中新增了一个关键字：assert。在程序开发过程中使用它创建一个断言(assertion)，它的
语法形式有如下所示的两种形式：
- assert condition;
    这里condition是一个必须为真(true)的表达式。如果表达式的结果为true，那么断言为真，并且无任何行动
如果表达式为false，则断言失败，则会抛出一个AssertionError对象。这个AssertionError继承于Error对象，
而Error继承于Throwable，Error是和Exception并列的一个错误对象，通常用于表达系统级运行错误。
- asser condition:expr;
    这里condition是和上面一样的，这个冒号后跟的是一个表达式，通常用于断言失败后的提示信息，说白了，它是一个传到AssertionError构造函数的值，如果断言失败，该值被转化为它对应的字符串，并显示出来。

## 二、使用示例：
    下面是一个使用assert的例子：
```java
public class TestAssert{
     public static void main(String[] args){
         String name = "abner chai";
         //String name = null;
         assert (name!=null):"变量name为空null";
         System.out.println(name);
     }
}
```

上面程序中，当变量name为null时，将会抛出一个AssertionError，并输出错误信息。
要想让上面的程序中的断言有效并且正确编译，在编译程序时，必须使用-source 1.4选项。如：

javac -source 1.4 TestAssert.java

在Eclipse(3.0M9)开发环境中，必须在window->preferences 中，左边选中"Java->Compiler"，右边选择
“Compliance and ClassFiles”页面下的将"Compiler Compliance Level"选择为1.4；同时，将
"Use Default Compiler Settings"前的勾去掉。并将下面的
"Generated .class file compatibility"和"Source compatibility"均选择为1.4，才能正确编译。

同时，要想让断言起效用，即让断言语句在运行时确实检查，在运行含有assert的程序时，必须指定-ea选项
如：为了能够让上面的程序运行，我们执行下面代码：

java -ea TestAssert

在在Eclipse(3.0M9)开发环境中，运行时，我们必须配置运行时的选项"Run"，在Arguments页面中的
"VM Arguments" 中填入-ea选项。才能让断言在运行时起作用。

## 三、注意事项：
    理解断言最重要的一点是必须不依赖它们完成任何程序实际所需的行为。理由是正常发布的代码都是断言无效的，即正常发布的代码中断言语句都不不执行的（或不起作用的），如果一不小心，我们可以错误地使用断言，如：

public class TestPerson{
    private String name = null;
    public TestPerson(String name){
        this.name = name;
    }
    public void setName(String nameStr){
        this.name = nameStr;
    }
    public String getName(){
         return this.name;
    }
    public static void main(String[] args){
        TestPerson personObj = new TestPerson("Abner Chai");
        String personName = null;
        assert (personName=personObj.getName())!=null;
        System.out.println(personName.length());
    }
}

这个程序中，对personName的赋值被转移到assert6语句中，尽管断言有效时它可以很好地运行（即使用-ea运行
时可以有效地运行）但如果断言失效，则它会运行时报空指针错误。因为断言无效时，
personName=personObj.getName()一句永远不会执行！
    断言对Java来说是一个好的条件，因为它们使开发过程中错误类型检查流线化，例如，在没有assert之前，
上面的程序要想确认personName 不空，则必须：

if(personName!=null){
    System.out.println(personName.length());
}
才行。有了assert后，使用assert，只需一行代码，并且不必从发布的代码中删除assert语句。
于是，上面的那个程序，经改正后，我们可以这么样来正确的使用assert，如下：

public class TestPerson{
    private String name = null;
    public TestPerson(String name){
        this.name = name;
    }
    public void setName(String nameStr){
        this.name = nameStr;
    }
    public String getName(){
         return this.name;
    }
    public static void main(String[] args){
        TestPerson personObj = new TestPerson("Abner Chai");
        String personName = null;
        personName=personObj.getName();
        assert personName!=null;
        System.out.println(personName.length());
    }
}

## 四、其它选项：
    当执行代码时，使用-ea选项使断言有效，也可以使用-da选项使断言无效（默认为无效）
同样，也可以通过在-ea或-da后面指定包名来使一个包的断言有效或无效。例如，要使一个com.test包中的断言
无效，可以使用：
-da:com.test
要使一个包中的所有子包中的断言能够有效或无效，在包名后加上三个点。例如：
-ea:com.test...

即可使com.test包及其子包中的断言无效。