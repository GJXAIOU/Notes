# 21理论七：重复的代码就一定违背DRY吗？如何提高代码的复用性？

DRY（Don’t Repeat Yourself）原则：不要重复自己。将它应用在编程中，可以理解为：不要写重复的代码。

实际上，重复的代码不一定违反 DRY 原则，而且有些看似不重复的代码也有可能违反 DRY 原则。

同时 DRY 原则与代码的复用性也有一些联系，因此如何写出可复用性好的代码很重要。

## 一、DRY 原则（Don’t Repeat Yourself）

**三种典型的代码重复情况包括：实现逻辑重复、功能语义重复和代码执行重复**。但这三种代码重复并不都是违反 DRY 原则。

### （一）实现逻辑重复

```java
public class UserAuthenticator {
    public void authenticate(String username, String password) {
        if (!isValidUsername(username)) {
            // ...throw InvalidUsernameException...
        }
        if (!isValidPassword(password)) {
            // ...throw InvalidPasswordException...
        }
        //... 省略其他代码...
    } 
    
    private boolean isValidUsername(String username) {
        // check not null, not empty
        if (StringUtils.isBlank(username)) {
            return false;
        }
        
        // check length: 4~64
        int length = username.length();
        if (length < 4 || length > 64) {
            return false;
        }
        
        // contains only lowcase characters
        if (!StringUtils.isAllLowerCase(username)) {
            return false;
        }
        
        // contains only a~z,0~9,dot
        for (int i = 0; i < length; ++i) {
            char c = username.charAt(i);
            if (!(c >= 'a' && c <= 'z') || (c >= '0' && c <= '9') || c == '.') {
                return false;
            }
        }
        return true;
    } 
    
    private boolean isValidPassword(String password) {
        // check not null, not empty
        if (StringUtils.isBlank(password)) {
            return false;
        }
        
        // check length: 4~64
        int length = password.length();
        if (length < 4 || length > 64) {
            return false;
        }
        
        // contains only lowcase characters
        if (!StringUtils.isAllLowerCase(password)) {
            return false;
        }
        
        // contains only a~z,0~9,dot
        for (int i = 0; i < length; ++i) {
            char c = password.charAt(i);
            if (!(c >= 'a' && c <= 'z') || (c >= '0' && c <= '9') || c == '.') {
                return false;
            }
        }
        return true;
    }
}
```

在代码中，有两处非常明显的重复的代码片段：`isValidUserName()` 函数和 `isValidPassword()` 函数。重复的代码被敲了两遍，看起来明显违反 DRY 原则。为了移除重复的代码，将两个函数重构合并为一个更通用的函数 `isValidUserNameOrPassword()`。重构后的代码如下所示：

```java
public class UserAuthenticatorV2 {
    public void authenticate(String userName, String password) {
        if (!isValidUsernameOrPassword(userName)) {
            // ...throw InvalidUsernameException...
        }
        if (!isValidUsernameOrPassword(password)) {
            // ...throw InvalidPasswordException...
        }
    }
    
    private boolean isValidUsernameOrPassword(String usernameOrPassword) {
        // 省略实现逻辑
        // 跟原来的 isValidUsername() 或 isValidPassword() 的实现逻辑一样...
        return true;
    }
}
```

经过重构之后，代码行数减少了，也没有重复的代码了，但是并不是更好了：

因为单从名字上看，合并之后的 `isValidUserNameOrPassword()` 函数，**负责两件事情：验证用户名和验证密码，违反了「单一职责原则」和「接口隔离原则」**。实际上，即便将两个函数合并成 `isValidUserNameOrPassword()`，代码仍然存在问题。

因为 `isValidUserName()` 和 `isValidPassword()` 两个函数，**虽然从代码实现逻辑上看起来是重复的，但是从语义上并不重复。所谓「语义不重复」指的是：从功能上来看，这两个函数干的是完全不重复的两件事情，一个是校验用户名，另一个是校验密码**。尽管在目前的设计中，两个校验逻辑是完全一样的，但如果按照第二种写法，将两个函数的合并，那就会存在潜在的问题。在未来的某一天，如果我们修改了密码的校验逻辑，比如，允许密码包含大写字符，允许密码的长度为 8 到 64 个字符，那这个时候，`isValidUserName()` 和 `isValidPassword()` 的实现逻辑就会不相同。我们就要把合并后的函数，重新拆成合并前的那两个函数。

**尽管代码的实现逻辑是相同的，但语义不同，我们判定它并不违反 DRY 原则。对于包含重复代码的问题，我们可以通过抽象成更细粒度函数的方式来解决。比如将校验只包含 `a~z、0~9、dot` 的逻辑封装成 `boolean onlyContains(String str, String charlist);` 函数。**

### （二）功能语义重复

在同一个项目代码中有下面两个函数：`isValidIp()` 和 `checkIfIpValid()`。**尽管两个函数的命名不同，实现逻辑不同，但功能是相同的**，都是用来判定 IP 地址是否合法的。

之所以在同一个项目中会有两个功能相同的函数，那是因为这两个函数是由两个不同的同事开发的，其中一个同事在不知道已经有了 `isValidIp()` 的情况下，自己又定义并实现了同样用来校验 IP 地址是否合法的 `checkIfIpValid()` 函数。

那在同一项目代码中，存在如下两个函数，是否违反 DRY 原则呢？

```java
public boolean isValidIp(String ipAddress) {
    if (StringUtils.isBlank(ipAddress)) return false;
    String regex = "^(1\\d{2}|2[0-4]\\d|25[0-5]|[1-9]\\d|[1-9])\\."
        + "(1\\d{2}|2[0-4]\\d|25[0-5]|[1-9]\\d|\\d)\\."
        + "(1\\d{2}|2[0-4]\\d|25[0-5]|[1-9]\\d|\\d)\\."
        + "(1\\d{2}|2[0-4]\\d|25[0-5]|[1-9]\\d|\\d)$";
    return ipAddress.matches(regex);
} 

public boolean checkIfIpValid(String ipAddress) {
    if (StringUtils.isBlank(ipAddress)) return false;
    String[] ipUnits = StringUtils.split(ipAddress, '.');
    if (ipUnits.length != 4) {
        return false;
    }
    for (int i = 0; i < 4; ++i) {
        int ipUnitIntValue;
        try {
            ipUnitIntValue = Integer.parseInt(ipUnits[i]);
        } catch (NumberFormatException e) {
            return false;
        }
        if (ipUnitIntValue < 0 || ipUnitIntValue > 255) {
            return false;
        }
        if (i == 0 && ipUnitIntValue == 0) {
            return false;
        }
    }
    return true;
}
```

这个例子跟上个例子正好相反。**上一个例子是代码实现逻辑重复，但语义不重复，我们并不认为它违反了 DRY 原则。而在这个例子中，尽管两段代码的实现逻辑不重复，但语义（功能）重复，我们认为它违反了 DRY 原则**。我们应该在项目中，统一一种实现思路，所有用到判断 IP 地址是否合法的地方，都统一调用同一个函数。

假设我们不统一实现思路，那有些地方调用了 `isValidIp()` 函数，有些地方又调用了 `checkIfIpValid()` 函数，这就会导致代码看起来很奇怪，相当于给代码「埋坑」，给不熟悉这部分代码的同事增加了阅读的难度。同事有可能研究了半天，觉得功能是一样的，但又有点疑惑，觉得是不是有更高深的考量，才定义了两个功能类似的函数，最终发现居然是代码设计的问题。

除此之外，如果哪天项目中 IP 地址是否合法的判定规则改变了，比如：255.255.255.255 不再被判定为合法的了，相应地，我们对 `isValidIp()` 的实现逻辑做了相应的修改，但却忘记了修改 `checkIfIpValid()` 函数。又或者，我们压根就不知道还存在一个功能相同的 `checkIfIpValid()` 函数，这样就会导致有些代码仍然使用老的 IP 地址判断逻辑，导致出现一些莫名其妙的 bug。

### （三）代码执行重复

前两个例子一个是实现逻辑重复，一个是语义重复，我们再来看第三个例子。其中， UserService 中 `login()` 函数用来校验用户登录是否成功。如果失败，就返回异常；如果成功，就返回用户信息。具体代码如下所示：

```java
public class UserService {
    // 通过依赖注入或者 IOC 框架注入
    private UserRepo userRepo;
    
    public User login(String email, String password) {
        boolean existed = userRepo.checkIfUserExisted(email, password);
        if (!existed) {
            // ... throw AuthenticationFailureException...
        }
        User user = userRepo.getUserByEmail(email);
        return user;
    }
} 

public class UserRepo {
    public boolean checkIfUserExisted(String email, String password) {
        if (!EmailValidation.validate(email)) {
            // ... throw InvalidEmailException...
        } 
        if (!PasswordValidation.validate(password)) {
            // ... throw InvalidPasswordException...
        } 
        //...query db to check if email&password exists...
    } 
    
    public User getUserByEmail(String email) {
        if (!EmailValidation.validate(email)) {
            // ... throw InvalidEmailException...
        }
        //...query db to get user by email...
    }
}
```

上面这段代码，既没有逻辑重复，也没有语义重复，但仍然违反了 DRY 原则。这是因为代码中存在「执行重复」。

重复执行最明显的一个地方，就是在 `login()` 函数中，email 的校验逻辑被执行了两次。一次是在调用 `checkIfUserExisted()` 函数的时候，另一次是调用 `getUserByEmail()` 函数的时候。这个问题解决起来比较简单，我们只需要将校验逻辑从 UserRepo 中移除，统一放到 UserService 中就可以了。

除此之外，代码中还有一处比较隐蔽的执行重复，不知道你发现了没有？实际上，`login()` 函数并不需要调用 `checkIfUserExisted()` 函数，只需要调用一次 `getUserByEmail()` 函数， 从数据库中获取到用户的 email、password 等信息，然后跟用户输入的 email、password 信息做对比，依次判断是否登录成功。

实际上，这样的优化是很有必要的。因为 `checkIfUserExisted()` 函数和 `getUserByEmail()` 函数都需要查询数据库，而数据库这类的 I/O 操作是比较耗时的。我们在写代码的时候， 应当尽量减少这类 I/O 操作。

按照刚刚的修改思路，我们把代码重构一下，移除「重复执行」的代码，只校验一次 email 和 password，并且只查询一次数据库。重构之后的代码如下所示：

```java
public class UserService {
    // 通过依赖注入或者 IOC 框架注入
    private UserRepo userRepo;
    
    public User login(String email, String password) {
        if (!EmailValidation.validate(email)) {
            // ... throw InvalidEmailException...
        }
        if (!PasswordValidation.validate(password)) {
            // ... throw InvalidPasswordException...
        }
        User user = userRepo.getUserByEmail(email);
        if (user == null || !password.equals(user.getPassword())){
            // ... throw AuthenticationFailureException...
        }
        return user;
    }
} 

public class UserRepo {
    public boolean checkIfUserExisted(String email, String password) {
        //...query db to check if email&password exists
    } 
    public User getUserByEmail(String email) {
        //...query db to get user by email...
    }
}
```

## 二、代码复用性（Code Reusability）

代码的复用性是评判代码质量的一个非常重要的标准，下面深入地学习一下这个知识点。

### （一）什么是代码的复用性？

首先区分三个概念：代码复用性（Code Reusability）、代码复用（Code Resue） 和 DRY 原则。

- 代码复用表示一种行为：我们在开发新功能的时候，尽量复用已经存在的代码。

- 代码的可复用性表示一段代码可被复用的特性或能力：我们在编写代码的时候，让代码尽量可复用。
- DRY 原则是一条原则：不要写重复的代码。

从定义描述上，它们好像有点类似，但深究起来，三者的区别还是蛮大的。

**首先，「不重复」并不代表「可复用」。**在一个项目代码中，可能不存在任何重复的代码， 但也并不表示里面有可复用的代码，不重复和可复用完全是两个概念。所以，从这个角度来说，DRY 原则跟代码的可复用性讲的是两回事。

**其次，「复用」和「可复用性」关注角度不同。**代码「可复用性」是从代码开发者的角度来讲的，「复用」是从代码使用者的角度来讲的。比如，A 同事编写了一个 UrlUtils 类，代码的「可复用性」很好。B 同事在开发新功能的时候，直接「复用」A 同事编写的 UrlUtils 类。

这三者从理解上有所区别，但实际上要达到的目的都是类似的，都是**为了减少代码量，提高代码的可读性、可维护性**。除此之外，复用已经经过测试的老代码，bug 会比从零重新开发要少。

「复用」这个概念不仅可以指导细粒度的模块、类、函数的设计开发，实际上，一些框架、类库、组件等的产生也都是为了达到复用的目的。比如，Spring 框架、Google Guava 类库、UI 组件等等。

### （二）怎么提高代码复用性？

实际上，我们前面已经讲到过很多提高代码可复用性的手段，总结如下：

- 减少代码耦合

  对于高度耦合的代码，当我们希望复用其中的一个功能，想把这个功能的代码抽取出来成为一个独立的模块、类或者函数的时候，往往会发现牵一发而动全身。移动一点代码，就要牵连到很多其他相关的代码。所以，高度耦合的代码会影响到代码的复用性，我们要尽量减少代码耦合。

- 满足单一职责原则

  我们前面讲过，如果职责不够单一，模块、类设计得大而全，那依赖它的代码或者它依赖的代码就会比较多，进而增加了代码的耦合。根据上一点，也就会影响到代码的复用性。相反，越细粒度的代码，代码的通用性会越好，越容易被复用。

- 模块化

  这里的「模块」，不单单指一组类构成的模块，还可以理解为单个类、函数。我们要善于将功能独立的代码，封装成模块。独立的模块就像一块一块的积木，更加容易复用，可以直接拿来搭建更加复杂的系统。

- 业务与非业务逻辑分离

  越是跟业务无关的代码越是容易复用，越是针对特定业务的代码越难复用。所以，为了复用跟业务无关的代码，我们将业务和非业务逻辑代码分离，抽取成一些通用的框架、类库、组件等。

- 通用代码下沉

  从分层的角度来看，越底层的代码越通用、会被越多的模块调用，越应该设计得足够可复用。一般情况下，在代码分层之后，为了避免交叉调用导致调用关系混乱，我们只允许上层代码调用下层代码及同层代码之间的调用，杜绝下层代码调用上层代码。所以，通用的代码我们尽量下沉到更下层。

- 继承、多态、抽象、封装

  在讲面向对象特性的时候，我们讲到，利用继承，可以将公共的代码抽取到父类，子类复用父类的属性和方法。利用多态，我们可以动态地替换一段代码的部分逻辑，让这段代码可复用。除此之外，抽象和封装，从更加广义的层面、而非狭义的面向对象特性的层面来理解的话，越抽象、越不依赖具体的实现，越容易复用。代码封装成模块，隐藏可变的细节、暴露不变的接口，就越容易复用。

- 应用模板等设计模式

  一些设计模式，也能提高代码的复用性。比如，模板模式利用了多态来实现，可以灵活地替换其中的部分代码，整个流程模板代码可复用。关于应用设计模式提高代码复用性这一部 分，我们留在后面慢慢来讲解。

除了刚刚我们讲到的几点，还有一些跟编程语言相关的特性，也能提高代码的复用性，比如泛型编程等。实际上，除了上面讲到的这些方法之外，复用意识也非常重要。在写代码的时候，我们要多去思考一下，这个部分代码是否可以抽取出来，作为一个独立的模块、类或者函数供多处使用。在设计每个模块、类、函数的时候，要像设计一个外部 API 那样，去思考它的复用性。

### （三）辩证思考和灵活应用

**实际上，编写可复用的代码并不简单。如果我们在编写代码的时候，已经有复用的需求场景，那根据复用的需求去开发可复用的代码，可能还不算难。但是，如果当下并没有复用的需求，我们只是希望现在编写的代码具有可复用的特点，能在未来某个同事开发某个新功能的时候复用得上。在这种没有具体复用需求的情况下，我们就需要去预测将来代码会如何复用，这就比较有挑战了。**

实际上，除非有非常明确的复用需求，否则，为了暂时用不到的复用需求，花费太多的时间、精力，投入太多的开发成本，并不是一个值得推荐的做法。这也违反我们之前讲到的 YAGNI 原则。

除此之外，有一个著名的原则，叫作「Rule of Three」。这条原则可以用在很多行业和场景中，你可以自己去研究一下。如果把这个原则用在这里，那就是说，我们在第一次写代码的时候，如果当下没有复用的需求，而未来的复用需求也不是特别明确，并且开发可复用代码的成本比较高，那我们就不需要考虑代码的复用性。在之后我们开发新的功能的时候，发现可以复用之前写的这段代码，那我们就重构这段代码，让其变得更加可复用。

**即第一次编写代码的时候，我们不考虑复用性；第二次遇到复用场景的时候，再进行重构使其复用**。`Three` 并不是真的就指确切的「三」，这里就是指「二」。

## 重点回顾

### DRY 原则

我们今天讲了三种代码重复的情况：实现逻辑重复、功能语义重复、代码执行重复。**实现逻辑重复，但功能语义不重复的代码，并不违反 DRY 原则。实现逻辑不重复，但功能语义重复的代码，也算是违反 DRY 原则。除此之外，代码执行重复也算是违反 DRY 原则。**

### 代码复用性

今天，我们讲到提高代码可复用性的一些方法，有以下 7 点。

- 减少代码耦合

- 满足单一职责原则模块化

- 业务与非业务逻辑分离通用代码下沉

- 继承、多态、抽象、封装应用模板等设计模式

实际上，除了上面讲到的这些方法之外，复用意识也非常重要。在设计每个模块、类、函数的时候，要像设计一个外部 API 一样去思考它的复用性。

我们在第一次写代码的时候，如果当下没有复用的需求，而未来的复用需求也不是特别明 确，并且开发可复用代码的成本比较高，那我们就不需要考虑代码的复用性。在之后开发新的功能的时候，发现可以复用之前写的这段代码，那我们就重构这段代码，让其变得更加可复用。

相比于代码的可复用性，DRY 原则适用性更强一些。我们可以不写可复用的代码，但一定不能写重复的代码。

## 课堂讨论

除了实现逻辑重复、功能语义重复、代码执行重复，你还知道有哪些其他类型的代码重复？ 这些代码重复是否违反 DRY 原则？

欢迎在留言区写下你的想法，和同学一起交流和分享。如果有收获，也欢迎你把这篇文章分享给你的朋友。

### 精选留言

- 1、注释或者文档违反 DRY

  2、数据对象违反 DRY

  对于 1，例如一个方法。写了好多的注释解释代码的执行逻辑，后续修改的这个方法的时候可能，忘记修改注释，造成对代码理解的困难。实际应用应该使用 KISS 原则，将方法写的简单。 

- 重复执行最明显的一个地方，就是在 login() 函数中，email 的校验逻辑被执行了两次。一次是在调用 checkIfUserExisted() 函数的时候，另一次是调用 getUserByEmail() 函数的时候。这个问题解决起来比较简单，我们只需要将校验逻辑从 UserRepo 中移除，统一放到 UserService 中就可以了。

- 整体来说我们要做的是不写"重复"代码，同时考虑代码的复用性，但要避免过度设计。 这几点说起来简单其实做起来还是有些难度的，在平常写代码的时候需要多思考，写完之后要反复审视自己的代码看看有没有可以优化的地方。说起来我感觉我还算是对代码有些追求的……但是真的需求来了为了赶需求基本就一遍过了……😂，对于一些脚本代码更是过程编程，惭愧啊

 

- 看完最后这个 Rlue of three , 我感觉把可扩展填进去也是有道理的, 一开始不一定写得出扩展性很好的代码, 所以可以先简单来, 后面需求明确了再慢慢重构把代码变得更加可以扩展?

- 关键点在于语义不重复，即使里面的执行逻辑重复，也并不违反 DRY 原则，而是 SRP 的体现。单一职责不仅体现在模块级，还体现在类级别，甚至函数级别，而很多人就错误的认为可复用就是没有重复的代码执行逻辑。细粒度的 DRY 指函数功能不重复，宏观的 DRY 指层与层之间职责不重复。

- 注释和 model 违反了 DRY 原则 注释写重复了, 或者 逻辑改了, 注释没改, model 则是 属性命名多余等等

- 之前只是觉得 DRY 就仅仅是代码上的重复，现在终于厘清了。还有语义，功能和执行上的重复。

  我们团队约定在写代码的时候，每层都需要检验参数，为了防止 NPE 和别人调用时出错， 就会造成很多重复的校验，但是由于每层的职责不一样，很多校验也算不上完全重复，不知道这个算不算的上是代码执行重复。
