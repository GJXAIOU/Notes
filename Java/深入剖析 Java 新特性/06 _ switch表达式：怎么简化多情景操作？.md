# 06 \| switch表达式：怎么简化多情景操作？

作者: 范学雷

完成时间:

总结时间:

![](<https://static001.geekbang.org/resource/image/2e/24/2ecb3bfe5e5630006f5f0aefe4533024.jpg>)

<audio><source src="https://static001.geekbang.org/resource/audio/ae/70/ae3ddf8c4d97a85cfeb049b72f47cc70.mp3" type="audio/mpeg"></audio>

你好，我是范学雷。今天，我们聊一聊Switch表达式。

switch表达式这个特性，首先在JDK 12中以预览版的形式发布。在JDK 13中，改进的switch表达式再次以预览版的形式发布。最后，switch表达式在JDK 14正式发布。

不论你学习什么样的编程语言，合理地分析、判断、处理不同的情况都是必备的基本功。比如我们使用的if-else语句，还有switch语句，都是用来处理种种不同的情况的。 我们都知道switch语句，那么switch表达式又是什么呢？switch语句和switch表达式又有什么不同呢？

如果你了解了Java的语句和表达式这两个基本概念，你的困扰也许会少一点。Java规范里，表达式完成对数据的操作。一个表达式的结果可以是一个数值（i \* 4）；或者是一个变量（i = 4）；或者什么都不是（void类型）。

Java语句是Java最基本的可执行单位，它本身不是一个数值，也不是一个变量。Java语句的标志性符号是分号（代码）和双引号（代码块），比如if-else语句，赋值语句等。这样再来看，就很简单了：switch表达式就是一个表达式，而switch语句就是一个语句。

switch表达式是什么样子的？为什么需要switch表达式？我们还是通过案例和代码，一点一点地来学习switch表达式吧。

<!-- [[[read_end]]] -->

## 阅读案例

在讲解或者学习switch语句时，每年的十二个月或者每周的七天，是我们经常使用的演示数据。在这个案例里，我们也使用这样的数据，来看看传统的switch语句有哪些需要改进的地方。

下面，我们要讨论的，也是一个传统的问题: 该怎么用代码计算一个月有多少天？生活中，我们熟悉这样的顺口溜，“一三五七八十腊，三十一天永不差，四六九冬三十整，平年二月二十八，闰年二月把一加”。

下面的这段代码，就是按照这个顺口溜的逻辑来计算了一下，今天所在的这个月，一共有多少天。

```java
package co.ivi.jus.swexpr.former;

import java.util.Calendar;

class DaysInMonth {
    public static void main(String[] args) {
        Calendar today = Calendar.getInstance();
        int month = today.get(Calendar.MONTH);
        int year = today.get(Calendar.YEAR);

        int daysInMonth;
        switch (month) {
            case Calendar.JANUARY:
            case Calendar.MARCH:
            case Calendar.MAY:
            case Calendar.JULY:
            case Calendar.AUGUST:
            case Calendar.OCTOBER:
            case Calendar.DECEMBER:
                daysInMonth = 31;
                break;
            case Calendar.APRIL:
            case Calendar.JUNE:
            case Calendar.SEPTEMBER:
            case Calendar.NOVEMBER:
                daysInMonth = 30;
                break;
            case Calendar.FEBRUARY:
                if (((year % 4 == 0) && !(year % 100 == 0))
                        || (year % 400 == 0)) {
                    daysInMonth = 29;
                } else {
                    daysInMonth = 28;
                }
                break;
            default:
                throw new RuntimeException(
                    "Calendar in JDK does not work");
        }

        System.out.println(
            "There are " + daysInMonth + " days in this month.");
    }
}
```

这段代码里，我们使用了switch语句。代码本身并没有什么错误，但是，至少有两个容易犯错误的地方。

第一个容易犯错的地方，就是在break关键字的使用上。上面的代码里，如果多使用一个break关键字，代码的逻辑就会发生变化；同样的，少使用一个break关键字也会出现问题。

```java
int daysInMonth;
switch (month) {
    case Calendar.JANUARY:
    case Calendar.MARCH:
    case Calendar.MAY:
        break;    // WRONG BREAK!!!
    case Calendar.JULY:
    case Calendar.AUGUST:
    case Calendar.OCTOBER:
    case Calendar.DECEMBER:
        daysInMonth = 31;
        break;
    // snipped
}
```

```java
int daysInMonth;
switch (month) {
    // snipped
    case Calendar.APRIL:
    case Calendar.JUNE:
    case Calendar.SEPTEMBER:
    case Calendar.NOVEMBER:
        daysInMonth = 30;
                         // WRONG, NO BREAK!!!
    case Calendar.FEBRUARY:
        if (((year % 4 == 0) && !(year % 100 == 0))
                || (year % 400 == 0)) {
            daysInMonth = 29;
        } else {
            daysInMonth = 28;
        }
        break;
    // snipped
}
```

break语句的遗漏或者冗余，这样的错误如此得常见，甚至于被单列成了一个[常见软件安全漏洞](<https://cwe.mitre.org/data/definitions/484.html>)。凡是使用switch语句的代码，都有可能成为黑客们重点关注的对象。由于逻辑的错误和黑客的特殊关照，我们在编写代码的时候，需要十二分的小心；阅读代码的时候，也需要反复地查验break语句的前后语境。毫无疑问，这增加了代码维护的成本，降低了生产效率。

为什么switch语句里需要使用break呢？最主要的原因，就是希望能够在不同的情况下，共享部分或者全部的代码片段。比如上面的例子中，四月、六月、九月、十一月这四种情景，可以共享每个月都是30天这样的代码片段。这个代码片段只需要写在十一月情景的后面，前面的四月、六月和九月这三个情景都会顺次执行下面的操作（fall-through），直到遇到下一个break语句或者switch语句终结。

现在我们都知道了，这样是一个弊大于利的设计。但很遗憾，Java初始的设计就是采用了这样的设计思想。如果要新设计一门现代的语言，我们需要更多地使用switch语句，但是就不要再使用break语句了。不过，不同的情景共享代码片段，仍然是一个真实的需求。在废弃掉break语句之前，我们要找到在不同的情景间共享代码片段的新规则。

第二个容易犯错的地方，是反复出现的赋值语句。 在上面的代码中，**daysInMonth**这个本地变量的变量声明和实际赋值是分开的。赋值语句需要反复出现，以适应不同的情景。如果在switch语句里，**daysInMonth**变量没有被赋值，编译器也不会报错，缺省的或者初始的变量值就会被使用。

```java
int daysInMonth = 0;
switch (month) {
    // snipped
    case Calendar.APRIL:
    case Calendar.JUNE:
    case Calendar.SEPTEMBER:
    case Calendar.NOVEMBER:
        break;   // WRONG, INITIAL daysInMonth value IS USED!!!
    case Calendar.FEBRUARY:
    // snipped
}
```

在上面的例子里，初始的变量值不是一个合适的数据；当然，在另外一个例子里，缺省的或者初始的变量值也可能就是一个合适的数据了。为了判断这个本地变量有没有合适的值，我们需要通览整个switch语句块，确保赋值没有遗漏，也没有多余。这增加了编码出错的几率，也增加了阅读代码的成本。

那么，能不能让多情景处理的代码块拥有一个数值呢？ 或者换个说法，多情景处理的代码块能不能变成一个表达式？这个想法，就催生了Java语言的新特性：“switch表达式”。

## switch表达式

switch表达式是什么样子的呢？下面的这段代码，使用的就是switch表达式，它改进了上面阅读案例里的代码。你可以带着上面遇到的问题，来阅读这段代码。这些问题包括：

- switch表达式是怎么表示一个数值，从而可以给变量赋值的？
- 在不同的情景间，switch表达式是怎么共享代码片段的？
- 使用switch表达式的代码，有没有变得更简单、更皮实、更容易理解？

<!-- -->

```java
package co.ivi.jus.swexpr.modern;

import java.util.Calendar;

class DaysInMonth {
    public static void main(String[] args) {
        Calendar today = Calendar.getInstance();
        int month = today.get(Calendar.MONTH);
        int year = today.get(Calendar.YEAR);

        int daysInMonth = switch (month) {
            case Calendar.JANUARY,
                 Calendar.MARCH,
                 Calendar.MAY,
                 Calendar.JULY,
                 Calendar.AUGUST,
                 Calendar.OCTOBER,
                 Calendar.DECEMBER -> 31;
            case Calendar.APRIL,
                 Calendar.JUNE,
                 Calendar.SEPTEMBER,
                 Calendar.NOVEMBER -> 30;
            case Calendar.FEBRUARY -> {
                if (((year % 4 == 0) && !(year % 100 == 0))
                        || (year % 400 == 0)) {
                    yield 29;
                } else {
                    yield 28;
                }
            }
            default -> throw new RuntimeException(
                    "Calendar in JDK does not work");
        };

        System.out.println(
                "There are " + daysInMonth + " days in this month.");
    }
}
```

**我们最先看到的变化，就是switch代码块出现在了赋值运算符的右侧。**这也就意味着，这个switch代码块表示的是一个数值，或者是一个变量。换句话说，这个switch代码块是一个表达式。

```java
int daysInMonth = switch (month) {
    // snipped
}
```

**我们看到的第二个变化，是多情景的合并。也就是说，一个case语句，可以处理多个情景。**这些情景，使用逗号分隔开来，共享一个代码块。而传统的switch代码，一个case语句只能处理一种情景。

```java
case Calendar.JANUARY,
     Calendar.MARCH,
     // snipped
```

多情景的合并的设计，满足了共享代码片段的需求。而且，由于只使用一个case语句，也就不再需要使用break语句来满足这个需求了。所以，break语句从switch表达式里消失了。

不同之处在于，传统的switch代码，不同的case语句之间可以共享部分的代码片段；而switch表达式里，需要共享全部的代码片段。这看似是一个损失，但其实，共享部分代码片段的能力给代码的编写者带来的困惑远远多于它带来的好处。如果需要共享部分的代码片段，我们总是可以找到替换的办法，比如把需要共享的代码封装成更小的方法。所以，我们没有必要担心switch表达式不支持共享部分代码片段。

**下一个变化，是一个新的情景操作符，“->”，****它是一个****箭头标识符。这个符号使用在case语句里，一般化的形式是“case L ->”**。这里的L，就是要匹配的一个或者多个情景。如果目标变量和情景匹配，那么就执行操作符右边的表达式或者代码块。如果要匹配的情景有两个或者两个以上，就要使用逗号“,”用分隔符把它们分割开来。

```java
case Calendar.JANUARY,
     // snipped
     Calendar.DECEMBER -> 31;
```

传统的switch代码，这个一般化的形式是“case L ：”，也就是使用冒号标识符。为什么不延续使用传统的情景操作符呢？这主要是出于简化代码的考虑。我们依然可以在switch表达式里使用冒号标识符，使用冒号标识符的一个case语句只能匹配一个情景，这种情况我们稍后再讨论。

**下一个我们看到的变化，是箭头标识符右侧的数值。这个数值，代表的就是该匹配情景下，switch表达式的数值。**需要注意的是，**箭头标识符右侧可以是表达式、代码块或者异常抛出语句，而不能是其他的形式。**如果只需要一个语句，这个语句也要以代码块的形式呈现出来。

```java
case Calendar.JANUARY,
     // snipped
     Calendar.DECEMBER -> {  // CORRECT, enclosed with braces.
    yield 31;
}
```

没有以代码块形式呈现的代码，编译的时候，就会报错。这是一个很棒的约束。代码块的形式，增强了视觉效果，减少了编码的失误。在[《代码精进之路》](<https://time.geekbang.org/column/intro/100019601>)这个专栏里，我们反复强调过这种形式的好处。

```java
case Calendar.JANUARY,
     // snipped
     Calendar.DECEMBER ->   // WRONG, not a block.
    yield 31;
```

另外，箭头标识符右侧需要一个表达switch表达式的数值，这是一个很强的约束。如果一个语句破坏了这个需要，它就不能出现在switch表达式里。比如，下面的代码里的return语句，意图退出该方法，而没有表达这个switch表达式的数值。这段代码就不能通过编译器的审查。

```java
int daysInMonth = switch (month) {
    // snipped
    case Calendar.APRIL,
         // snipped
         Calendar.NOVEMBER -> {
        // yield 30;
        return; // WRONG, return outside of enclosing switch expression.
    }
    // snipped
}
```

**最后一个我们能够看到的变化，是出现了一个新的关键字“yield”**。大多数情况下，switch 表达式箭头标识符的右侧是一个数值或者是一个表达式。 如果需要一个或者多个语句，我们就要使用代码块的形式。这时候，我们就需要引入一个新的 yield 语句来产生一个值，这个值就成为这个封闭代码块代表的数值。

为了便于理解，我们可以把yield语句产生的值看成是switch表达式的返回值。所以，yield只能用在switch 表达式里，而不能用在switch语句里。

```java
case Calendar.FEBRUARY -> {
    if (((year % 4 == 0) && !(year % 100 == 0))
            || (year % 400 == 0)) {
        yield 29;
    } else {
        yield 28;
    }
}
```

**其实，这里还有一个我们从上述的代码里看不到的变化。在switch表达式里，所有的情景都要列举出来，不能多、也不能少（这也就是我们常说的穷举）。**

比如说，在上面的例子里，如果没有最后的default情景分支，编译器就会报错。这是一个影响深远的改进，它会使得switch表达式的代码更加健壮，大幅度降低维护成本，如果未来需要增加一个情景分支的话，就更是如此了。

```java
int daysInMonth = switch (month) {
    case Calendar.JANUARY,
         // snipped
         Calendar.DECEMBER -> 31;
    case Calendar.APRIL,
         // snipped
         Calendar.NOVEMBER -> 30;
    case Calendar.FEBRUARY -> {
             // snipped
    }
    // WRONG to comment out the default branch, 'switch' expression
    // MUST cover all possible input values.
    //
    // default -> throw new RuntimeException(
    //        "Calendar in JDK does not work");
};
```

## 改进的switch语句

通过上面的解读，我们知道了switch表达式里有很多积极的变化。那这些变化有没有影响switch语句呢？比如说，我们能够在switch语句里使用箭头标识符吗？我们前面说过，yield语句是来产生一个switch表达式代表的数值的，因此yield语句只能用在switch表达式里，不能用在switch语句。

其他的变化呢？我们还是先来看下面一段代码。

```java
private static int daysInMonth(int year, int month) {
    int daysInMonth = 0;
    switch (month) {
        case Calendar.JANUARY,
             Calendar.MARCH,
             Calendar.MAY,
             Calendar.JULY,
             Calendar.AUGUST,
             Calendar.OCTOBER,
             Calendar.DECEMBER ->
            daysInMonth = 31;
        case Calendar.APRIL,
             Calendar.JUNE,
             Calendar.SEPTEMBER,
             Calendar.NOVEMBER ->
            daysInMonth = 30;
        case Calendar.FEBRUARY -> {
            if (((year % 4 == 0) && !(year % 100 == 0))
                    || (year % 400 == 0)) {
                daysInMonth = 29;
                break;
            }

            daysInMonth = 28;
        }
        // default -> throw new RuntimeException(
        //        "Calendar in JDK does not work");
    }

    return daysInMonth;
}
```

在这段代码里，我们看到了箭头标识符，看到了break语句，看到了注释掉的default语句。这是一段合法的、能够工作的代码。换个说法，switch语句可以使用箭头标识符，也可以使用break语句，也不需要列出所有的情景。表面上看起来，switch语句的改进不是那么显而易见。其实，switch语句的改进主要体现在break语句的使用上。

我们应该也看到了，break语句没有出现在下一个case语句之前。这也就意味着，使用箭头标识符的switch语句不再需要break语句来实现情景间的代码共享了。虽然我们还可以这样使用break语句，但是已经不再必要了。

```java
switch (month) {
    // snipped
    case Calendar.APRIL,
             // snipped
         Calendar.NOVEMBER -> {
            daysInMonth = 30;
            break;  // UNNECESSARY, could be removed safely.
        }
    // snipped
}
```

有没有break语句，使用箭头标识符的switch语句都不会顺次执行下面的操作（fall-through）。这样，我们前面谈到的break语句带来的烦恼也就消失不见了。

不过，使用箭头标识符的switch语句并没有禁止break语句，而是恢复了它本来的意义：从代码片段里抽身，就像它在循环语句里扮演的角色一样。

```java
switch (month) {
   // snipped
   case Calendar.FEBRUARY -> {
        if (((year % 4 == 0) && !(year % 100 == 0))
                || (year % 400 == 0)) {
            daysInMonth = 29;
            break;     // BREAK the switch statement
        }
    
        daysInMonth = 28;
    }
   // snipped
}
```

## 怪味的switch表达式

我们前面说过，switch表达式也可以使用冒号标识符。使用冒号标识符的一个case语句只能匹配一个情景，而且支持fall-through。和箭头标识符的switch表达式一样，使用冒号标识符switch表达式也不支持break语句，取而代之的是yield语句。

这是一个充满了怪味道的编码形式，我并不推荐使用这种形式，但我可以带你略作了解。下面的这段代码，就是我们试着把箭头标识符替换成冒号标识符的一个例子。你可以比较一下使用冒号标识符和箭头标识符的两段代码，想一想两种不同形式的优劣。毫无疑问，使用箭头标识符的代码更加简洁。

```java
package co.ivi.jus.swexpr.legacy;

import java.util.Calendar;

class DaysInMonth {
    public static void main(String[] args) {
        Calendar today = Calendar.getInstance();
        int month = today.get(Calendar.MONTH);
        int year = today.get(Calendar.YEAR);

    int daysInMonth = switch (month) {
        case Calendar.JANUARY:
        case Calendar.MARCH:
        case Calendar.MAY:
        case Calendar.JULY:
        case Calendar.AUGUST:
        case Calendar.OCTOBER:
        case Calendar.DECEMBER:
            yield 31;
        case Calendar.APRIL:
        case Calendar.JUNE:
        case Calendar.SEPTEMBER:
        case Calendar.NOVEMBER:
            yield 30;
        case Calendar.FEBRUARY:
            if (((year % 4 == 0) && !(year % 100 == 0))
                    || (year % 400 == 0)) {
                yield 29;
            } else {
                yield 28;
            }
        default:
            throw new RuntimeException(
                    "Calendar in JDK does not work");
        };

        System.out.println(
            "There are " + daysInMonth + " days in this month.");
    }
}
```

有了使用箭头标识符的switch语句和switch表达式之后，我们不再推荐使用冒号标识符的switch语句和switch表达式。学习并使用箭头标识符的switch语句和switch表达式，会使代码更简洁、更健壮。

## 总结

好，到这里，我来做个小结。从前面的讨论中，我们重点了解了switch表达式和改进的switch语句。我们还讨论了switch表达式带来的新概念和新的关键字，了解了这些基本概念以及它们的适用范围。

新的switch形式、语句和表达式，不同的使用范围，这些概念交织在一起，让switch的学习和使用都变成了一件有点挑战性的事情。箭头标识符的引入，简化了代码，提高了编码效率。可是，学习这么多种switch的表现形式，也增加了我们的学习负担。为了帮助你快速掌握这些形式，我把不同的switch表达形式，以及它们支持的特征，放在了下面这张表格里。

![图片](<https://static001.geekbang.org/resource/image/8e/09/8ea744fd05104e66703f4c24a72ddd09.jpg?wh=1920x991>)

或者，你也可以记住下面的总结：

- break语句只能出现在switch语句里，不能出现在switch表达式里；
- yield语句只能出现在switch表达式里，不能出现在switch语句里；
- switch表达式需要穷举出所有的情景，而switch语句不需要情景穷举；
- 使用冒号标识符的swtich形式，支持情景间的fall-through；而使用箭头标识符的swtich形式不支持fall-through；
- 使用箭头标识符的swtich形式，一个case语句支持多个情景；而使用冒号标识符的swtich形式不支持多情景的case语句。

<!-- -->

使用箭头标识符的swtich形式，废止了容易出问题的fall-through这个特征。因此，我们推荐使用箭头标识符的swtich形式，逐步废止使用冒号标识符的swtich形式。在switch表达式和switch语句之间，我们应该优先使用switch表达式。这些选择，都可以帮助我们简化代码逻辑，减少代码错误，提高生产效率。

如果你要丰富你的代码评审清单，学习完这一节内容后，你可以加入下面这一条：

> 使用冒号标识符的swtich形式，是不是可以更改为使用箭头标识符？<br>
> 
>  使用switch语句赋值的操作，是不是可以更改为使用switch表达式？

另外，我还拎出了几个今天讨论过的技术要点，这些都可能在你们面试中出现哦。通过这一次学习，你应该能够：

- 知道switch表达式，并且能够使用switch表达式； - 面试问题：你知道switch表达式吗？该怎么处理switch表达式里的语句？

    <!-- -->

- 了解switch表达式要解决的问题，并且知道解决掉这些问题的办法； - 面试问题：使用switch表达式有哪些好处？

    <!-- -->

- 了解不同的switch的表现形式，能够看得懂不同的表现形式，并且给出改进意见。 - 面试问题：你更喜欢使用箭头标识符还是冒号标识符？

    <!-- -->


<!-- -->

如果你能够有意识地使用箭头标识符的switch表达式，应该可以大幅度提高编码的效率和质量；如果你能够了解不同的switch表现形式，并且对每种形式都有自己的见解，你就能帮助你的同事提高编码的效率和质量。毫无疑问，在面试的时候，有意识地在代码里使用switch表达式，是一个能够展现你的学习能力、理解能力和对新知识的接受能力的一个好机会。

## 思考题

在前面的讨论里，我们说过情景穷举是一个影响深远的改进方向，它会使得switch表达式的代码更加健壮，大幅度降低维护成本，特别是在未来需要增加一个情景分支的情形下。但是，限于篇幅，我们并没有详细地展开讨论其中的细节。现在，我们把这个讨论当作一个稍微有点挑战的思考题。

假设有一天，地球和太阳的关系发生了变化，这种变化还没有大到毁灭人类的程度，但是也足以改变年月的关系了。于是，天文学家重新修订了日历，增加了一个新的月份，第十三个月。为了对应这种变化，JDK的设计者们也给Calendar类增加了第十三个月：Calendar.AFTERDEC。那么，我们的问题就来了。

第一个问题是，我们现在的代码能够检测到这个变化吗？如果不能，是不是只有系统崩溃的时候，我们才能够意识到问题的存在？

第二个问题是，有没有更健壮的设计，能够帮助我们在系统崩溃之前就能够检测到这个意想不到的变化？从而给我们留出时间更改我们的代码和系统？

稍微提示一个，解决这个问题的其中一个思路，就是要使用有穷举能力的表达式，然后设计出可以表达穷举情景的新形式，而不是使用泛泛的整数来表达十二个月。

我在下面的例子中写了一个代码小样。这个代码小样，实现的还是一年只有十二个月的逻辑。现在我们假设，一年还是十二个月，但是我们想让这段代码健壮到能够检测到未来一年变成十一个月或者十三个月的情景。

在这个代码小样里，我也试着加入了一些提示。当然，你也可以试着找找其他的解决方案。请试着将这段代码修改成你喜欢的样子，让我们一起看看怎么解决掉这个问题。

```java
package co.ivi.jus.swexpr.review.xuelei;

import java.util.Calendar;

class DaysInMonth {
    public static void main(String[] args) {
        Calendar today = Calendar.getInstance();
        int month = today.get(Calendar.MONTH);
        int year = today.get(Calendar.YEAR);

    // Hints: could we replace the integer month
    // with an exhaustive enumeration?
    int daysInMonth = switch (month) {
            case Calendar.JANUARY,
                 Calendar.MARCH,
                 Calendar.MAY,
                 Calendar.JULY,
                 Calendar.AUGUST,
                 Calendar.OCTOBER,
                 Calendar.DECEMBER -> 31;
            case Calendar.APRIL,
                 Calendar.JUNE,
                 Calendar.SEPTEMBER,
                 Calendar.NOVEMBER -> 30;
            case Calendar.FEBRUARY -> {
                if (((year % 4 == 0) && !(year % 100 == 0))
                        || (year % 400 == 0)) {
                    yield 29;
                } else {
                    yield 28;
                }
            }
            // Hints: Are we able to replace the default case by
            // enumerating all cases with case clause above?
            default -> throw new RuntimeException(
                    "Calendar in JDK does not work");
        };

        System.out.println(
            "There are " + daysInMonth + " days in this month.");
    }
}
```

欢迎你在留言区留言、讨论，分享你的阅读体验以及你对这个思考题的想法。

注：本文使用的完整的代码可以从[GitHub](<https://github.com/XueleiFan/java-up/tree/main/src/main/java/co/ivi/jus/swexpr>)下载，你可以通过修改[GitHub](<https://github.com/XueleiFan/java-up/tree/main/src/main/java/co/ivi/jus/instance>)上[review template](<https://github.com/XueleiFan/java-up/blob/main/src/main/java/co/ivi/jus/swexpr/review/xuelei/DaysInMonth.java>)代码，完成这次的思考题。如果你想要分享你的修改或者想听听评审的意见，请提交一个 GitHub的拉取请求（Pull Request），并把拉取请求的地址贴到留言里。这一小节的拉取请求代码，请在[实例匹配专用的代码评审目录](<https://github.com/XueleiFan/java-up/blob/main/src/main/java/co/ivi/jus/swexpr/review>)下，建一个以你的名字命名的子目录，代码放到你专有的子目录里。比如，我的代码，就放在swexpr/review/xuelei的目录下面。

