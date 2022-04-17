# [Python迭代器和生成器](https://www.cnblogs.com/wilber2013/p/4652531.html)

在Python中，很多对象都是可以通过for语句来直接遍历的，例如list、string、dict等等，这些对象都可以被称为可迭代对象。至于说哪些对象是可以被迭代访问的，就要了解一下迭代器相关的知识了。

## 迭代器

迭代器对象要求支持迭代器协议的对象，在Python中，支持迭代器协议就是实现对象的__iter__()和next()方法。其中__iter__()方法返回迭代器对象本身；next()方法返回容器的下一个元素，在结尾时引发StopIteration异常。

### __iter__()和next()方法

这两个方法是迭代器最基本的方法，一个用来获得迭代器对象，一个用来获取容器中的下一个元素。

对于可迭代对象，可以使用内建函数iter()来获取它的迭代器对象：

![](https://images0.cnblogs.com/blog/593627/201507/162113386417660.png)

例子中，通过iter()方法获得了list的迭代器对象，然后就可以通过next()方法来访问list中的元素了。当容器中没有可访问的元素后，next()方法将会抛出一个StopIteration异常终止迭代器。

其实，当我们使用for语句的时候，for语句就会自动的通过__iter__()方法来获得迭代器对象，并且通过next()方法来获取下一个元素。

### 自定义迭代器

了解了迭代器协议之后，就可以自定义迭代器了。

下面例子中实现了一个MyRange的类型，这个类型中实现了__iter__()方法，通过这个方法返回对象本身作为迭代器对象；同时，实现了next()方法用来获取容器中的下一个元素，当没有可访问元素后，就抛出StopIteration异常。
```language
class MyRange(object): 
  def __init__(self, n):
        self.idx = 0
        self.n = n 
  def __iter__(self): 
    return self 
  def next(self): 
    if self.idx < self.n:
        val = self.idx
        self.idx += 1
        return val 
    else: 
        raise StopIteration()
```


这个自定义类型跟内建函数xrange很类似，看一下运行结果：
```language
myRange = MyRange(3) 
for i in myRange: 
    print i   
```
![](https://images0.cnblogs.com/blog/593627/201507/162113392048788.png)

### 迭代器和可迭代对象

在上面的例子中，myRange这个对象就是一个可迭代对象，同时它本身也是一个迭代器对象。

看下面的代码，对于一个可迭代对象，如果它本身又是一个迭代器对象，就会有下面的 问题，就没有办法支持多次迭代。

![](https://images0.cnblogs.com/blog/593627/201507/162113401265874.png)

为了解决上面的问题，可以分别定义可迭代类型对象和迭代器类型对象；然后可迭代类型对象的__iter__()方法可以获得一个迭代器类型的对象。看下面的实现：
```python
class Zrange:
    def __init__(self, n):
        self.n = n

    def __iter__(self):
        return ZrangeIterator(self.n)

class ZrangeIterator:
    def __init__(self, n):
        self.i = 0
        self.n = n

    def __iter__(self):
        return self

    def next(self):
        if self.i < self.n:
            i = self.i
            self.i += 1
            return i
        else:
            raise StopIteration()    

            
zrange = Zrange(3)
print zrange is iter(zrange)         

print [i for i in zrange]
print [i for i in zrange]
```


代码的运行结果为：

![](https://images0.cnblogs.com/blog/593627/201507/162113404074088.png)

其实，通过下面代码可以看出，list类型也是按照上面的方式，list本身是一个可迭代对象，通过iter()方法可以获得list的迭代器对象：

![](https://images0.cnblogs.com/blog/593627/201507/162113418456316.png)

## 生成器

在Python中，使用生成器可以很方便的支持迭代器协议。生成器通过生成器函数产生，生成器函数可以通过常规的def语句来定义，但是不用return返回，而是用yield一次返回一个结果，在每个结果之间挂起和继续它们的状态，来自动实现迭代协议。

也就是说，yield是一个语法糖，内部实现支持了迭代器协议，同时yield内部是一个状态机，维护着挂起和继续的状态。

下面看看生成器的使用：

![](https://images0.cnblogs.com/blog/593627/201507/162113461573598.png)

在这个例子中，定义了一个生成器函数，函数返回一个生成器对象，然后就可以通过for语句进行迭代访问了。

其实，生成器函数返回生成器的迭代器。 "生成器的迭代器"这个术语通常被称作"生成器"。要注意的是生成器就是一类特殊的迭代器。作为一个迭代器，生成器必须要定义一些方法，其中一个就是next()。如同迭代器一样，我们可以使用next()函数来获取下一个值。

### 生成器执行流程

下面就仔细看看生成器是怎么工作的。

从上面的例子也可以看到，生成器函数跟普通的函数是有很大差别的。

结合上面的例子我们加入一些打印信息，进一步看看生成器的执行流程：

![](https://images0.cnblogs.com/blog/593627/201507/162113509381037.png)

通过结果可以看到：

*   当调用生成器函数的时候，函数只是返回了一个生成器对象，并没有 执行。
*   当next()方法第一次被调用的时候，生成器函数才开始执行，执行到yield语句处停止

    *   next()方法的返回值就是yield语句处的参数（yielded value）
*   当继续调用next()方法的时候，函数将接着上一次停止的yield语句处继续执行，并到下一个yield处停止；如果后面没有yield就抛出StopIteration异常

### 生成器表达式

在开始介绍生成器表达式之前，先看看我们比较熟悉的列表解析( List comprehensions)，列表解析一般都是下面的形式。

[expr for iter_var in iterable if cond_expr]

迭代iterable里所有内容，每一次迭代后，把iterable里满足cond_expr条件的内容放到iter_var中，再在表达式expr中应该iter_var的内容，最后用表达式的计算值生成一个列表。

例如，生成一个list来保护50以内的所以奇数：

[i for i in range(50) if i%2]

生成器表达式是在python2.4中引入的，当序列过长， 而每次只需要获取一个元素时，应当考虑使用生成器表达式而不是列表解析。生成器表达式的语法和列表解析一样，只不过生成器表达式是被()括起来的，而不是[]，如下：

(expr for iter_var in iterable if cond_expr)

看一个例子：

![](https://images0.cnblogs.com/blog/593627/201507/162113519859850.png)

生成器表达式并不是创建一个列表， 而是返回一个生成器，这个生成器在每次计算出一个条目后，把这个条目"产生"（yield）出来。 生成器表达式使用了"惰性计算"（lazy evaluation），只有在检索时才被赋值（evaluated），所以在列表比较长的情况下使用内存上更有效。

继续看一个例子：

![](https://images0.cnblogs.com/blog/593627/201507/162113540636248.png)

从这个例子中可以看到，生成器表达式产生的生成器，它自身是一个可迭代对象，同时也是迭代器本身。

### 递归生成器

生成器可以向函数一样进行递归使用的，下面看一个简单的例子，对一个序列进行全排列：
```python
def permutations(li):
    if len(li) == 0:
        yield li
    else:
        for i in range(len(li)):
            li[0], li[i] = li[i], li[0]
            for item in permutations(li[1:]):
                yield [li[0]] + item
    
for item in permutations(range(3)):
    print item
```


代码的结果为：

![](https://images0.cnblogs.com/blog/593627/201507/162113554389261.png)

### 生成器的send()和close()方法

生成器中还有两个很重要的方法：send()和close()。

*   send(value):

    从前面了解到，next()方法可以恢复生成器状态并继续执行，其实send()是除next()外另一个恢复生成器的方法。

    Python 2.5中，yield语句变成了yield表达式，也就是说yield可以有一个值，而这个值就是send()方法的参数，所以send(None)和next()是等效的。同样，next()和send()的返回值都是yield语句处的参数（yielded value）

    关于send()方法需要**注意**的是：调用send传入非None值前，生成器必须处于挂起状态，否则将抛出异常。也就是说，第一次调用时，要使用next()语句或send(None)，因为没有yield语句来接收这个值。

*   close():

    这个方法用于关闭生成器，对关闭的生成器后再次调用next或send将抛出StopIteration异常。

下面看看这两个方法的使用：

![](https://images0.cnblogs.com/blog2015/593627/201507/162152472511515.png)

## 总结

本文介绍了Python迭代器和生成器的相关内容。

*   通过实现迭代器协议对应的__iter__()和next()方法，可以自定义迭代器类型。对于可迭代对象，for语句可以通过iter()方法获取迭代器，并且通过next()方法获得容器的下一个元素。
*   像列表这种序列类型的对象，可迭代对象和迭代器对象是相互独立存在的，在迭代的过程中各个迭代器相互独立；但是，有的可迭代对象本身又是迭代器对象，那么迭代器就没法独立使用。
*   itertools模块提供了一系列迭代器，能够帮助用户轻松地使用排列、组合、笛卡尔积或其他组合结构。

*   生成器是一种特殊的迭代器，内部支持了生成器协议，不需要明确定义__iter__()和next()方法。
*   生成器通过生成器函数产生，生成器函数可以通过常规的def语句来定义，但是不用return返回，而是用yield一次返回一个结果。