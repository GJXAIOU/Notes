# python's super() considered super

[sources](https://rhettinger.wordpress.com/2011/05/26/super-considered-super/)

## Python’s super() considered super!

If you aren’t wowed by Python’s super() builtin, chances are you don’t really know what it is capable of doing or how to use it effectively.

Much has been written about super() and much of that writing has been a failure. This article seeks to improve on the situation by:

*   providing practical use cases
*   giving a clear mental model of how it works
*   showing the tradecraft for getting it to work every time
*   concrete advice for building classes that use super()
*   favoring real examples over abstract ABCD [diamond diagrams](http://en.wikipedia.org/wiki/Diamond_problem).

The examples for this post are available in both [Python 2 syntax](http://code.activestate.com/recipes/577721-how-to-use-super-effectively-python-27-version/) and [Python 3 syntax](http://code.activestate.com/recipes/577720-how-to-use-super-effectively/).

Using Python 3 syntax, let’s start with a basic use case, a subclass for extending a method from one of the builtin classes:

class LoggingDict(dict):
    def __setitem__(self, key, value):
        logging.info('Setting %r to %r' % (key, value))
        super().__setitem__(key, value)

This class has all the same capabilities as its parent, _dict_, but it extends the __setitem__ method to make log entries whenever a key is updated. After making a log entry, the method uses super() to delegate the work for actually updating the dictionary with the key/value pair.

Before super() was introduced, we would have hardwired the call with _dict.__setitem__(self, key, value)_. However, super() is better because it is a computed indirect reference.

One benefit of indirection is that we don’t have to specify the delegate class by name. If you edit the source code to switch the base class to some other mapping, the super() reference will automatically follow. You have a single source of truth:

class LoggingDict(SomeOtherMapping):            # new base class
    def __setitem__(self, key, value):
        logging.info('Setting %r to %r' % (key, value))
        super().__setitem__(key, value)         # no change needed

In addition to isolating changes, there is another major benefit to computed indirection, one that may not be familiar to people coming from static languages. Since the indirection is computed at runtime, we have the freedom to influence the calculation so that the indirection will point to some other class.

The calculation depends on both the class where super is called and on the instance’s tree of ancestors. The first component, the class where super is called, is determined by the source code for that class. In our example, super() is called in the _LoggingDict.__setitem___method. That component is fixed. The second and more interesting component is variable (we can create new subclasses with a rich tree of ancestors).

Let’s use this to our advantage to construct a logging ordered dictionary without modifying our existing classes:

class LoggingOD(LoggingDict, collections.OrderedDict):
    pass

The ancestor tree for our new class is: _LoggingOD_,_ LoggingDict_,_OrderedDict_,_ dict_,_ object_. For our purposes, the important result is that _OrderedDict_ was inserted after _LoggingDict_ and before _dict_! This means that the super() call in _LoggingDict.__setitem___ now dispatches the key/value update to _OrderedDict_ instead of _dict_.

Think about that for a moment. We did not alter the source code for _LoggingDict_. Instead we built a subclass whose only logic is to compose two existing classes and control their search order.

__________________________________________________________________________________________________________________

**Search Order**

What I’ve been calling the search order or ancestor tree is officially known as the Method Resolution Order or MRO. It’s easy to view the MRO by printing the __mro__ attribute:

>>> pprint(LoggingOD.__mro__)
(<class '__main__.LoggingOD'>,
 <class '__main__.LoggingDict'>,
 <class 'collections.OrderedDict'>,
 <class 'dict'>,
 <class 'object'>)

If our goal is to create a subclass with an MRO to our liking, we need to know how it is calculated. The basics are simple. The sequence includes the class, its base classes, and the base classes of those bases and so on until reaching _object_ which is the root class of all classes. The sequence is ordered so that a class always appears before its parents, and if there are multiple parents, they keep the same order as the tuple of base classes.

The MRO shown above is the one order that follows from those constraints:

*   LoggingOD precedes its parents, LoggingDict and OrderedDict
*   LoggingDict precedes OrderedDict because LoggingOD.__bases__ is (LoggingDict, OrderedDict)
*   LoggingDict precedes its parent which is dict
*   OrderedDict precedes its parent which is dict
*   dict precedes its parent which is object

The process of solving those constraints is known as linearization. There are a number of good papers on the subject, but to create subclasses with an MRO to our liking, we only need to know the two constraints: children precede their parents and the order of appearance in ___bases___ is respected.

__________________________________________________________________________________________________________________

**Practical Advice**

super() is in the business of delegating method calls to some class in the instance’s ancestor tree. For reorderable method calls to work, the classes need to be designed cooperatively. This presents three easily solved practical issues:

*   the method being called by super() needs to exist
*   the caller and callee need to have a matching argument signature
*   and every occurrence of the method needs to use super()

1) Let’s first look at strategies for getting the caller’s arguments to match the signature of the called method. This is a little more challenging than traditional method calls where the callee is known in advance. With super(), the callee is not known at the time a class is written (because a subclass written later may introduce new classes into the MRO).

One approach is to stick with a fixed signature using positional arguments. This works well with methods like __setitem__ which have a fixed signature of two arguments, a key and a value. This technique is shown in the _LoggingDict_ example where __setitem__ has the same signature in both _LoggingDict_ and _dict_.

A more flexible approach is to have every method in the ancestor tree cooperatively designed to accept keyword arguments and a keyword-arguments dictionary, to remove any arguments that it needs, and to forward the remaining arguments using **kwds, eventually leaving the dictionary empty for the final call in the chain.

Each level strips-off the keyword arguments that it needs so that the final empty dict can be sent to a method that expects no arguments at all (for example, _object.__init___ expects zero arguments):

class Shape:
    def __init__(self, shapename, **kwds):
        self.shapename = shapename
        super().__init__(**kwds)        

class ColoredShape(Shape):
    def __init__(self, color, **kwds):
        self.color = color
        super().__init__(**kwds)

cs = ColoredShape(color='red', shapename='circle')

2) Having looked at strategies for getting the caller/callee argument patterns to match, let’s now look at how to make sure the target method exists.

The above example shows the simplest case. We know that _object_ has an __init__ method and that _object_ is always the last class in the MRO chain, so any sequence of calls to _super().__init___ is guaranteed to end with a call to _object.__init___ method. In other words, we’re guaranteed that the target of the super() call is guaranteed to exist and won’t fail with an _AttributeError_.

For cases where _object_ doesn’t have the method of interest (a draw() method for example), we need to write a root class that is guaranteed to be called before _object_. The responsibility of the root class is simply to eat the method call without making a forwarding call using super().

_Root.draw_ can also employ [defensive programming](http://en.wikipedia.org/wiki/Defensive_programming) using an assertion to ensure it isn’t masking some other draw() method later in the chain.  This could happen if a subclass erroneously incorporates a class that has a draw() method but doesn’t inherit from _Root_.:

class Root:
    def draw(self):
        # the delegation chain stops here
        assert not hasattr(super(), 'draw')

class Shape(Root):
    def __init__(self, shapename, **kwds):
        self.shapename = shapename
        super().__init__(**kwds)
    def draw(self):
        print('Drawing.  Setting shape to:', self.shapename)
        super().draw()

class ColoredShape(Shape):
    def __init__(self, color, **kwds):
        self.color = color
        super().__init__(**kwds)
    def draw(self):
        print('Drawing.  Setting color to:', self.color)
        super().draw()

cs = ColoredShape(color='blue', shapename='square')
cs.draw()

If subclasses want to inject other classes into the MRO, those other classes also need to inherit from _Root_ so that no path for calling draw() can reach _object_ without having been stopped by _Root.draw_. This should be clearly documented so that someone writing new cooperating classes will know to subclass from _Root_. This restriction is not much different than Python’s own requirement that all new exceptions must inherit from _BaseException_.

3) The techniques shown above assure that super() calls a method that is known to exist and that the signature will be correct; however, we’re still relying on super() being called at each step so that the chain of delegation continues unbroken. This is easy to achieve if we’re designing the classes cooperatively – just add a super() call to every method in the chain.

The three techniques listed above provide the means to design cooperative classes that can be composed or reordered by subclasses.

__________________________________________________________________________________________________________________

**How to Incorporate a Non-cooperative Class**

Occasionally, a subclass may want to use cooperative multiple inheritance techniques with a third-party class that wasn’t designed for it (perhaps its method of interest doesn’t use super() or perhaps the class doesn’t inherit from the root class). This situation is easily remedied by creating an [adapter class](http://en.wikipedia.org/wiki/Adapter_pattern) that plays by the rules.

For example, the following _Moveable_ class does not make super() calls, and it has an __init__() signature that is incompatible with _object.__init___, and it does not inherit from _Root_:

class Moveable:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    def draw(self):
        print('Drawing at position:', self.x, self.y)

If we want to use this class with our cooperatively designed _ColoredShape_ hierarchy, we need to make an adapter with the requisite super() calls:

class MoveableAdapter(Root):
    def __init__(self, x, y, **kwds):
        self.movable = Moveable(x, y)
        super().__init__(**kwds)
    def draw(self):
        self.movable.draw()
        super().draw()

class MovableColoredShape(ColoredShape, MoveableAdapter):
    pass

MovableColoredShape(color='red', shapename='triangle',
                    x=10, y=20).draw()

__________________________________________________________________________________________________________________

**Complete Example – Just for Fun**

In Python 2.7 and 3.2, the collections module has both a _Counter_class and an _OrderedDict_ class. Those classes are easily composed to make an _OrderedCounter_:

from collections import Counter, OrderedDict

class OrderedCounter(Counter, OrderedDict):
     'Counter that remembers the order elements are first seen'
     def __repr__(self):
         return '%s(%r)' % (self.__class__.__name__,
                            OrderedDict(self))
     def __reduce__(self):
         return self.__class__, (OrderedDict(self),)

oc = OrderedCounter('abracadabra')

__________________________________________________________________________________________________________________

**Notes and References**

***** When subclassing a builtin such as dict(), it is often necessary to override or extend multiple methods at a time. In the above examples, the __setitem__ extension isn’t used by other methods such as _dict.update_, so it may be necessary to extend those also. This requirement isn’t unique to super(); rather, it arises whenever builtins are subclassed.

***** If a class relies on one parent class preceding another (for example, _LoggingOD_ depends on _LoggingDict_ coming before _OrderedDict_which comes before _dict_), it is easy to add assertions to validate and document the intended method resolution order:

position = LoggingOD.__mro__.index
assert position(LoggingDict) < position(OrderedDict)
assert position(OrderedDict) < position(dict)

***** Good write-ups for linearization algorithms can be found at [Python MRO documentation](http://www.python.org/download/releases/2.3/mro/) and at [Wikipedia entry for C3 Linearization](http://en.wikipedia.org/wiki/C3_linearization).

***** The [Dylan programming language](http://en.wikipedia.org/wiki/Dylan_(programming_language)) has a _next-method_ construct that works like Python’s super(). See [Dylan’s class docs](http://www.opendylan.org/books/dpg/db_347.html) for a brief write-up of how it behaves.

***** The Python 3 version of super() is used in this post. The full working source code can be found at:  [Recipe 577720](http://code.activestate.com/recipes/577720-how-to-use-super-effectively/). The Python 2 syntax differs in that the _type_ and _object_ arguments to super() are explicit rather than implicit. Also, the Python 2 version of super() only works with new-style classes (those that explicitly inherit from _object_ or other builtin type). The full working source code using Python 2 syntax is at [Recipe 577721](http://code.activestate.com/recipes/577721-how-to-use-super-effectively-python-27-version/).
__________________________________________________________________________________________________________________

**Acknowledgements**

Serveral Pythonistas did a pre-publication review of this article.  Their comments helped improve it quite a bit.

They are:  Laura Creighton, Alex Gaynor, Philip Jenvey, Brian Curtin, David Beazley, Chris Angelico, Jim Baker, Ethan Furman, and Michael Foord.  Thanks one and all.
