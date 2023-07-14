# 如何写出没有注释的代码？【dog】

## 阿里妹导读

每个程序员都会讨厌两件事情，阅读没有注释的代码，给代码写注释。那么如何一次解决两大难题，不用写注释，也不会被他人吐槽没有注释呢？﻿

> "If our programming languages were expressive enough, or if we had the talent to subtly wield those languages to express our intent, we would not need comments very much—perhaps not at all."

> -- Robert C.Martin 《Clean Code》

> 若编程语言足够有表达力，或者我们长于用这些语言来表达意图，那么我们就不那么需要注释，甚至根本不需要。

程序猿说要有代码，于是代码和注释就一起诞生了。有一句名言，每个程序员都会讨厌两件事情，阅读没有注释的代码，给代码写注释。

毕竟，人与人的悲欢并不相同，我只觉得你写的代码一团糟。如何一次解决程序猿的两大难题，不用写注释，也不会被他人吐槽没有注释呢？﻿

为什么要减少代码中的注释量呢？

- 注释的存在说明当前的这段代码逻辑并不是特别清晰，没有额外说明，他人就可能无法理解意图。
- 注释很难得到维护，一个任务开发结束后，分布在任务中的注释大概率就不会再被更新，随着时间的推移，代码越来越复杂，可能注释的信息早已不能准确反馈当前的逻辑了。
- 减少注释的过程也是一个重新审视代码结构，精简代码的过程。

那么糟糕的注释有哪些，又有什么办法可以干掉这些注释呢？

1.废话式注释

后端不能信任前端传入的任何信息，所以写代码的时候，也不能信任旁边同学的任何能力。哪怕最简单的操作，也要增加一段注释来说明操作的细节。

```java
public void addInfo(Info info){
  ...
  //向信息列表中追加信息内容
  this.infoList.add(info);
  ...
}
```

相信你的同学的能力，常见的操作，即使没有注释，也能够通过上下文理解出你的意图的。﻿

2.絮絮叨叨的注释

代码逻辑太绕，为了防止它变成上帝的小秘密，那就写上一段注释，这样就不会变成上帝的小秘密了。

```java
public void addInfos(List<Info> infos){
  ...
  //如果Infos没有，就直接返回，如果Infos只有一个，那就删除数据库的信息，再写入。。。
  ...
}
```

看似有了注释，再也不担心变成上帝的小秘密了。但是一旦离注释较远的少量代码被修改，则会导致上帝重新独享这段代码的秘密。

> "Comments Do Not Make Up for Bad Code"
>
> -- Robert C.Martin 《Clean Code》
>
> 注释并不能优化糟糕的代码

复杂的注释说明可能本身代码的逻辑是可能有问题的，整理逻辑图并重新梳理状态机的模型，可能比写出复杂的注释更有意义。从左边的七种可能的通路，多种执行的策略，到右边的清晰简单的二级判断+执行流程，状态及变化清晰简单。﻿

![图片](alibaba_%E5%A6%82%E4%BD%95%E5%86%99%E5%87%BA%E6%B2%A1%E6%9C%89%E6%B3%A8%E9%87%8A%E7%9A%84%E4%BB%A3%E7%A0%81%EF%BC%9F%E3%80%90dog%E3%80%91.resource/640.png)

3.代替代码分层的注释

开发框架讲究高内聚，松耦合。所以我的函数也要高内聚，能写在一起的函数，坚决不分开成多块的函数。为了避免有人理解不了这段代码的逻辑，那就加上注释好了。

```java
// 判断是否活跃用户
if((customer.getLastLoginTime().after(dateUtils.minusDays(new Date(),15)) && customer.getCommentsLast30Days() > 5) 
    || orderService.countRecentDaysByCustomer(customer,30) > 1)
```

不合理的分层一方面增加了多余的注释，而且也让整个代码维护性变高。如果判断活跃用户的逻辑发生变化，岂不是需要Ctrl+F去查找全文，一旦漏了一个地方，bug就由此产生了。

合理的函数抽提可以加强代码的逻辑性，与其看见复杂的布尔计算，isXXX才是我的最爱

```java
private boolean isActivate(){
  return getLastLoginTime().after(dateUtils.minusDays(new Date(),15)) && getCommentsLast30Days() > 5) 
    || orderService.countRecentDaysByCustomer(this, 30) > 1)
}

if (customer.isActive()){
  ...
}
```

4.不知所云的注释

为了让上帝更快的独享代码的秘密，所以连注释也快速投奔了上帝秘密的怀抱。

```java
public synchronized void setFormatter(Formatter newFormatter) {
    checkPermission();
    // 检查是否空指针
    newFormatter.getClass();
    formatter = newFormatter;
}
```

这儿的注释，看起来无论如何都不能和后面的代码所联系起来。但是java提供的函数 Objects.requireNonNull()能够比作者使用的的getClass方法更加清晰的说明检查空指针的意图，同时也能干掉这种不知所云的代码。与之相似的还有Collections、StringUtils下面的一系列函数。

虽然前面吐槽了这么多糟糕的注释，以及针对这些注释的无注释方法，但是正如ChatGPT的回答：

> 写出没有注释的代码是一项挑战，因为注释可以帮助读者理解代码的目的和工作原理。以下是一些编写没有注释的代码的建议：
>
> 1.保持代码简洁：简洁的代码易于理解，使其更容易阅读和理解。
>
> 2.使用描述性的名称：使用描述性的变量和函数名可以帮助读者理解代码的目的。
>
> 3.遵循一致的编码风格：使用一致的缩进、空格和命名约定可以使代码更加易于阅读。
>
> 4.将代码分解成小块：将代码分解成小块可以帮助读者理解代码的目的和功能。
>
> 5.编写自描述的代码：编写自描述的代码可以帮助读者理解代码的目的，例如，使用有意义的变量名和函数名。尽管这些技巧可能会帮助你编写更容易阅读的代码，但是注释仍然是编写高质量代码的重要组成部分。当您必须编写没有注释的代码时，请尽可能遵循这些建议。

虽然无注释的代码也许终不可得，但是通过减少注释，审视我们的代码结构，让代码晚一日成为上帝的小秘密，那也是非常值得的啦。





------

- “注释很难得到维护”这一段：注释并不是很难维护，注释如果很难维护，那你的代码也很难维护。修改代码的时候，注释也应该跟随一起调整。这是一个基本操守。 文章里其他部分写的还可以。当然为了做到不写注释，不只是写正确注释这么简单，涉及模型的设计，编码的整洁等。

- 对比《Clean Code》，我更推荐《REFACTORING: IMPROVING THE DESIGN OF EXISTING CODE, 2nd Edition》，实用性更高。

  文章里其他部分写的还可以。当然为了做到不写注释，不只是写正确注释这么简单，涉及模型的设计，编码的整洁等。