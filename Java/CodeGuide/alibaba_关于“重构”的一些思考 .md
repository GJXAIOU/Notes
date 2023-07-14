# 关于“重构”的一些思考

## 阿里妹导读

本文将从一个新人数次修改CR comments的角度探讨代码重构的定义、目的以及常见的重构方法，并以简单的代码案例来说明代码重构的具体实现。

> 任何一个傻瓜都可以写出计算机可以理解的代码，唯有写出人类容易理解的代码，才是优秀的程序员。
>
> ——Martin Fowler 《重构》

## 一、讲一个小故事

最近公司的电梯出了问题，平常从1层到7层都畅通无阻的电梯，偏偏在经过4楼时神秘的跳过了这一层。老板决定由你全盘负责电梯的修理和维护。你简单研究了下这部电梯，似乎初步找出了问题的关键所在。这部电梯的1层到4层是一个老程序员修建的，而后面的5到7层则来自于另一位跟你水平相仿的程序员。这也就是说，问题很大可能正是来自于4层到5层的接缝处，随着公司大楼的不断加高，原本生效的程序发生了某些错误的吻合，才导致4层被神秘的“跳过“。了解了问题所在，你立刻开始针对性的开展工作，在你的妙手回春下，你和你的老板一起眼睁睁的看着电梯不可逆转的上升，向上、向上，甚至还顶破了7层的天花板，在一众员工的众目睽睽下消失在了高空之中……

你开始意识到问题可能超出了你的想象，面对老板的不断催促，你只能赶鸭子上架般拿出了你的最终方案:

1.要求前往4楼的人站在电梯的后半部分。

2.在电梯下方安装滑梯。

3.将电梯后半部分的地板换成活板门，当检测到有人按4楼时，在5楼开启活板门。

4.最重要的一步，关闭电梯内部的光源，将电梯变成后来人看不明白的黑箱，也就没人会来找你的麻烦了。

虽然有些员工抱怨前往4楼时会遇到匪夷所思的失重感，但这无伤大雅的问题跟4楼的恢复使用又能算得上什么呢？你唯一需要知道的就是，你成功的解决了公司的问题，你的薪水翻了三番，而且你因为业务能力出色被调到了跟程序毫无关系的领导岗位，甚至还跟前一任修理电梯的员工当上了同事。

至于你问那个电梯？要相信后人的智慧。

## 前言

上文的小故事生动地向我们展现了重构在系统中的重要性，在软件开发领域，代码重构（Code Refactoring）是一项十分重要的技术。不仅可以提高代码的可读性和可维护性，同时也可以提高代码的质量和性能。本文将从一个新人数次修改CR comments的角度探讨代码重构的定义、目的以及常见的重构方法，并以简单的代码案例来说明代码重构的具体实现。

## 一、代码重构的定义

代码重构是指在不改变代码功能的前提下，通过修改代码的内部结构和外部表现形式，来提高代码的可读性、可维护性、性能和可扩展性的一种技术。代码重构通常包括以下几个方面：

- 改进代码的结构，使代码更加清晰简洁；
- 消除代码中的重复部分，减少代码冗余；
- 提高代码的可读性，使代码更加易于理解和维护；
- 提高代码的性能，减少代码的执行时间和内存占用；
- 改善代码的可扩展性，使代码更容易被扩展和修改。

## 二、代码重构的目的

代码重构的主要目的是提高代码的质量，使其更加易于理解、维护和扩展。具体来说，代码重构的目的包括以下几个方面：

1.提高代码的可读性

可读性是衡量代码质量的重要指标之一。鲁肃曾言：写下一行代码只要1分钟，但未来会被一代代工程师读很多次、改很多次。代码的可读性与可维护性，是我心目中好代码的第一标准。

良好的可读性可以使代码更加易于理解和维护，减少代码的错误和bug。代码重构可以通过改进代码的结构、消除代码中的冗余部分等方式来提高代码的可读性。

2.减少代码的冗余

冗余代码是指在代码中重复出现的部分。最单纯的重复代码就是“同一个类的两个函数含有相同的表达式”，冗余代码会使代码量增大，影响代码的可读性和可维护性。代码重构可以通过消除代码中的冗余部分来减少代码量，提高代码的可维护性。﻿

3.提高代码的性能

代码重构可以通过优化代码结构和算法来提高代码的性能。具体来说，可以通过减少代码的执行时间和内存占用来提高代码的性能。虽然重构也有可能导致软件的运行速度下降，但重构之后也会使软件的性能优化更加容易，长时间看，最终的效果还是好的。

4.提高代码的可扩展性

对于一段代码的好坏，另一个重要的评价指标就是可扩展性，可扩展性是指代码在未来可以被容易地修改和扩展。代码重构可以通过改进代码结构和使用设计模式等方式来提高代码的可扩展性。一段好的代码一定是高可扩展的，这个就是代码设计方面的问题了。

## 三、代码重构的方法

代码重构的方法有很多种，从顶层设计到底层逻辑均可以实现重构。然而，若是所有的人力都投入到技术改造上，可能距离拥抱变化也就不远了。我们返璞归真，这里不谈多么高大上的设计方式，仅讲述笔者在开发过程中用到的几种最为常见的方法，代码较为简单，主要是体会重构的思路。

### **方法提取**

这种重构方法是我在开发过程中最常用的一个方法，因为我经常由于一个方法过长被提了若干个CR comments。后来，团队内的一个前辈告诉我：一个方法不宜超过50行，超过50行的代码，就充斥着“代码坏味道”。

方法提取是指将一段代码抽象出来形成一个方法。这样做的好处是可以减少代码的重复，提高代码的可读性和可维护性。下面用一个案例来说明提取方法的具体实现，重构前的代码：

```java
public void printInvoice(Invoice invoice) {
    System.out.println("Invoice Number: " + invoice.getNumber());
    System.out.println("Customer Name: " + invoice.getCustomer().getName());
    System.out.println("Invoice Date: " + invoice.getDate());
    System.out.println("Total Amount: " + invoice.getTotalAmount());
    System.out.println("Items:");
    for (InvoiceItem item : invoice.getItems()) {
        System.out.println(item.getName() + " - " + item.getPrice() + " - " + item.getQuantity());
    }
}
```

上述代码的printInvoice方法输出的内容五花八门，打印发票的过程包括了打印头部信息和打印详细项目信息，没有一个清晰的定义，对上述代码根据方法抽取的原则进行重构，将打印头部信息和打印详细项目信息分别抽象成两个方法，使代码更加清晰简洁。重构后的代码：

```java
public void printInvoice(Invoice invoice) {
    printInvoiceHeader(invoice);
    printInvoiceItems(invoice.getItems());
}
private void printInvoiceHeader(Invoice invoice) {
    System.out.println("Invoice Number: " + invoice.getNumber());
    System.out.println("Customer Name: " + invoice.getCustomer().getName());
    System.out.println("Invoice Date: " + invoice.getDate());
    System.out.println("Total Amount: " + invoice.getTotalAmount());
}
private void printInvoiceItems(List<InvoiceItem> items) {
    System.out.println("Items:");
    for (InvoiceItem item : items) {
        System.out.println(item.getName() + " - " + item.getPrice() + " - " + item.getQuantity());
    }
}
```

重构后的代码更加清晰，可读性明显提升不少。在搞懂这个方法之前，我曾经有过一个疑惑：会不会因为短函数的原因造成大量函数调用，从而影响系统的性能。经过我对长函数以及短函数的大量运行测试发现：代码运行时编译器对那些短函数更加容易的进行缓存，也就是说，短函数可以更好的调动编译器的优化功能。

### **提取变量**

变量在代码中有着各种用途，其中有些时候存在一些临时变量被多次赋值的情况，还有很多变量会用于保存一段冗长代码的运算结果。这些变量很显然在代码中不止被赋值一次，每一次赋值对于这些变量来说就是承担了一次新的责任，**同一个变量承担多个责任**，很显然，代码的可读性极其低下。所以，需要对这些变量进行提取。提取变量是指将一段表达式抽象出来形成一个变量。这样做的好处是可以减少代码的重复，提高代码的可读性和可维护性。下面用一个案例来说明提取变量的具体实现，重构前的代码：

```java
public double calculateTotalAmount(List<InvoiceItem> items) {
    double totalAmount = 0;
    for (InvoiceItem item : items) {
        totalAmount += item.getPrice() * item.getQuantity();
    }
    if (totalAmount > 100) {
        totalAmount *= 0.9;
    }
    return totalAmount;
}
```

可以看到，在上述代码中，计算每个项目的金额是通过 item.getPrice() * item.getQuantity() 表达式来实现的。通过提取变量的方式，对上述代码进行重构。重构后的代码：

```java
public double calculateTotalAmount(List<InvoiceItem> items) {
    double totalAmount = 0;
    for (InvoiceItem item : items) {
        double itemAmount = item.getPrice() * item.getQuantity();
        totalAmount += itemAmount;
    }
    if (totalAmount > 100) {
        totalAmount *= 0.9;
    }
    return totalAmount;
}

```

重构后的代码将这个表达式抽象成一个变量 itemAmount，使代码更变得加易于理解和维护。

### **重构条件语句**

写过代码的人都明白一个定理：代码的大部分功能都来自于条件判断，然而，程序的复杂度也大量来自于逻辑判断。重构的一个万年不变的话题就是条件语句的重构。条件逻辑的重构有很多方法，例如：分解条件表达式、合并条件表达式、以多态取代条件表达式等。然而他们的核心思想都是一致的：通过简化、合并或提取条件语句，使代码更加清晰和易于理解。下面用一个案例来说明**重构条件**语句的具体实现，重构前的代码：

```java

public boolean canCreateAccount(Customer customer) {
    boolean canCreate = true;
    if (customer.getAge() < 18) {
        canCreate = false;
    }
    if (customer.getAccountNumber() != null && customer.getAccountNumber().length() != 0) {
        canCreate = false;
    }
    if (customer.getCreditScore() < 500) {
        canCreate = false;
    }
    return canCreate;
}
```

可以看到，代码中，判断客户是否有资格创建账户的过程是通过多个条件语句实现的，如果还有其他情况，只能通过添加if/else的方法实现。显然，这不符合一段优秀代码的定义。根据重构条件语句的方式对其重构，重构后的代码：

```java

public boolean canCreateAccount(Customer customer) {
    boolean canCreate = true;
    if (!isCustomerEligible(customer)) {
        canCreate = false;
    }
    return canCreate;
}
private boolean isCustomerEligible(Customer customer) {
    if (customer.getAge() < 18) {
        return false;
    }
    if (customer.getAccountNumber() != null && customer.getAccountNumber().length() != 0) {
        return false;
    }
    if (customer.getCreditScore() < 500) {
        return false;
    }
    return true;
}

```

可以看出，重构后的代码将多个条件语句合并成一个方法 isCustomerEligible()，使代码更加清晰易读。当然，对于这种多分支的逻辑语句，有各种不同的重构方法，在我的日常开发工作中，对于多条件判断的重构，首先想到的就是设计模式中的策略模式，大部分判断逻辑，都可用策略模式进行重构。本文只是列举了一种最为简单的方法，不意味着这就是最好的。

### **提取抽象类**

**提取抽象类**是指将多个类中的公共方法抽象出来形成一个抽象类，使得这些类可以继承这个抽象类来继承公共方法。这样做的好处是可以减少重复代码，提高代码的复用性和可维护性。下面用一个案例来说明提取抽象类的具体实现，重构前的代码：

```java

public class SavingsAccount {
    private double balance;
    private double interestRate;
    public SavingsAccount(double balance, double interestRate) {
        this.balance = balance;
        this.interestRate = interestRate;
    }
    public double getBalance() {
        return balance;
    }
    public double getInterestRate() {
        return interestRate;
    }
    public double calculateInterest() {
        return balance * interestRate;
    }
}
public class CheckingAccount {
    private double balance;
    private double transactionFee;
    public CheckingAccount(double balance, double transactionFee) {
        this.balance = balance;
        this.transactionFee = transactionFee;
    }
    public double getBalance() {
        return balance;
    }
    public double getTransactionFee() {
        return transactionFee;
    }
    public double calculateTransactionFee() {
        return transactionFee;
    }
}
```

可以看到，代码中，SavingsAccount 和 CheckingAccount 类有很多相同的方法，如 getBalance() 方法。通过提取抽象类的方式对代码进行重构，重构后的代码：

```java
public abstract class Account {
    protected double balance;
    public Account(double balance) {
        this.balance = balance;
    }
    public double getBalance() {
        return balance;
    }
    public abstract double calculateInterest();
}
public class SavingsAccount extends Account {
    private double interestRate;
    public SavingsAccount(double balance, double interestRate) {
        super(balance);
        this.interestRate = interestRate;
    }
    public double getInterestRate() {
        return interestRate;
    }
    public double calculateInterest() {
        return balance * interestRate;
    }
}
public class CheckingAccount extends Account {
    private double transactionFee;
    public CheckingAccount(double balance, double transactionFee) {
        super(balance);
        this.transactionFee = transactionFee;
    }
    public double getTransactionFee() {
        return transactionFee;
    }
    public double calculateTransactionFee() {
        return transactionFee;
    }
}
```

经过上述重构，将这些公共方法抽象成一个抽象类 Account，使得 SavingsAccount 和 CheckingAccount 类可以继承这个抽象类来继承公共方法，代码的可读性有着显著的上升。

## Q&A

**1.重构与设计之间的关系是什么？**

我认为重构和设计可以理解成全局与局部的关系。因此重构并不是设计的改正措施，不能希望重构能把一个糟糕的设计变成优秀的设计。所以，在软件开发之前设计出优秀的系统就显得尤为重要。﻿

**2.什么时候选择重构？什么时候选择重写？**

每当有功能要接入N年前的老代码的时候，重新来做一个新的系统完全替代这个老项目吧，我们可以用最新的框架，更好的实现方式去完成这个系统，这种天真的想法很多人的脑海里会无数次出现，然而，旧的系统业务很复杂，新的系统在兼容旧系统逻辑的同时，旧的系统也在更新需求，增加功能，在新系统完全可以抗衡旧系统之前，旧的系统会一直运行。如果新系统开发的时间过长，等完成的时候，可能开发者都已经不知道换了几批了，代码又乱成了一锅粥，周而复始，最后只能作罢。

至于这个问题，我相信这个问题就算是世界上最牛的架构师都无法给出一个确定性的答案。说到底，这个问题只能个人判断，很难给出一个有共通性的同类情况。不过，一些需要重写的代码肯定有迹象，例如：某个项目由于编程语言太老或者平台环境太老导致推进速度较慢，不适用于当前情况。此时，这种情况下，重构代码的作用微乎其微，就需要选择重写了。﻿

**3.如何保证重构过程中不会引入新的bug？**

既然想要重构，就意味着要修改代码，修改代码就可能引入新的bug。所以，重构只能保证设计的改进，而不能保证程序没有bug。目前集团内发现bug的有效手段就是充分有效的测试，而保证充分有效的测试的关键是单元测试的高覆盖率。﻿

## 结语

重构的核心不仅仅是一份“修代码”的指南，更为重要的是它所传达的理念：如何将一个大变化抽象为若干个细微的小变化，又在尽可能多进行细微变化的同时，不改变系统的整体表现。其实，“重构”代码并不是一项多么高深的工作，甚至很多方法显得略微基础。然而，很多人就是忽略了这些基础，才会写出大量充满着“坏味道”的代码。所以，要想减少重构所消耗的资源与精力，那就从基础开始写好每一行代码。

其实，写这篇文章之前，我回顾看过和写过的很多代码，让自己满意、有成就感的不多。未来路还长，以下一段代码才是我写的最好的代码的态度继续努力吧！