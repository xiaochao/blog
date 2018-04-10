Title: Python两个对象相等的原理
Meta: 大部分的python程序员平时编程的时候，很少关心两个对象为什么相等，因为教程和经验来说，他们就应该相等，比如1==1就应该返回True，可是当我们想要定义自己的对象或者修改默认的对象行为时，通常会因为不了解原理而导致各种奇奇怪怪的错误。
Date: 2017-12-12
Tags: Python,equal,hash
Category: python库
Slug: python_equal
Author: 笨熊

## 概述
&emsp;&emsp;大部分的python程序员平时编程的时候，很少关心两个对象为什么相等，因为教程和经验来说，他们就应该相等，比如1==1就应该返回True，可是当我们想要定义自己的对象或者修改默认的对象行为时，通常会因为不了解原理而导致各种奇奇怪怪的错误。

## 两个对象如何相等
&emsp;&emsp;两个对象如何才能相等要比我们想象的复杂很多，但核心的方法是重写__eq__方法，这个方法返回True，则表示两个对象相等，否则，就不相等。相反的，如果两个对象不相等，则重写__ne__方法。
&emsp;&emsp;默认情况下，如果你没有实现这个方法，则使用父类(object)的方法。父类的方法比较是的两个对象的ID(可以通过id方法获取对象ID)，也就是说，如果对象的ID相等，则两个对象也就相等。因此，我们可以得知，默认情况下，对象只和自己相等。例如：
    
    >>> class A(object):
    ...     pass
    ...
    >>>
    >>> a = A()
    >>> b = A()
    >>> a == a
    True
    >>> a == b
    False
    >>> id(a)
    4343310992
    >>> id(b)
    4343310928

&emsp;&emsp;Python2程序员经常犯的一个错误是，只重写了__eq__方法，而没有重写__ne__方法，导致不可预计的错误。而Python3会自动重写__ne__方法，如果你没有重写的话。

## 对象的Hash方法
&emsp;&emsp;Python里可Hash的对象，都有一个数字ID代表了它在python里的值，这个ID是由对象的__hash__方法返回的。因此，如果想让一个对象可Hash，那必须实现__hash__方法和之前提到的__eq__方法。和对象相等一样，默认情况下，对象的__hash__方法继承自Object对象，而Object对象的__hash__方法只计算对象ID，因此两个对象始终拥有两个不一样的hash id，不管他们是多么相似。
&emsp;&emsp;当我们把一个不可Hash的对象加入到set或者dict时，会发生什么了？

    >>> set().add({})
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    TypeError: unhashable type: 'dict'
    unhashable type: 'dict'

原因是set()和dict()使用对象的hash值作为内部索引，以便能快速索引到指定对象。因此，同一个对象返回相同的hash id就很重要了。

## 对象的Hash值在它的生命周期内不能改变
&emsp;&emsp;如果你想定义一个比较完美的对象，并且实现了__eq__和__hash__方法来定义对象的比较行为和hash值，那么你就需要保证对象的相关属性不能发生更改。不然会导致很诡异的错误，比如下面的例子。

    >>> class C:
    ...     def __init__(self, x):
    ...         self.x = x
    ...     def __repr__(self):
    ...         return "C({"+str(self.x)+"})"
    ...     def __hash__(self):
    ...         return hash(self.x)
    ...     def __eq__(self, other):
    ...         return (
    ...             self.__class__ == other.__class__ and
    ...             self.x == other.x
    ...         )
    >>> d = dict()
    >>> s = set()
    >>> c = C(1)
    >>> d[c] = 42
    >>> s.add(c)
    >>> d, s
    ({C(1): 42}, {C(1)})
    >>> c in s and c in d  # c is in both!
    True
    >>> c.x = 2
    >>> c in s or c in d   # c is in neither!?
    False
    >>> d, s
    ({C(2): 42}, {C(2)})   # but...it's right there!
    
在我们没有修改对象的属性时(c.x=2)之前，所有行为都符合预期。当我们通过c.x=2时修改属性后，执行c in s or c in d返回False，但是内容却是修改后的，是不是很奇怪。这也就解释了为什么str、tuple是可Hash的，而list和dict是不可hash的。

因此我们可以得出结论，如果两个对象相等的话，那它们的hash值必然也是相等的。


## 总结
讲了这么多有什么用了。
1. 当我们遇到unhashable type这个异常时，我们能够知道为什么报这个错误。
2. 如果定义了一个可比较的对象，那么最好保证对象hash值相关的属性在生命周期内不能发生改变，不然会发生意想不到的错误。
