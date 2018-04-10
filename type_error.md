Title: TypeError: object() takes no parameters
Meta: 你也经常遇到的Python错误：TypeError: object() takes no parameters，为什么会发生这样的错误了?
Date: 2017-10-10
Tags: Python,TypeError
Category: python库
Slug: typeerror
Author: 笨熊

日常编写Python代码的过程中，特别是Python新手，经常会遇到这样的错误：

    TypeError: object() takes no parameters

对于上面这个错误，很容易迷惑我们，因为这个错误信息没有很明确的指出，到底是哪段代码除了问题。那这个错误是怎么产生的了，请听我细细道来。

在python中，方法是一个属性，也就是说，当我们调用一个方法时，python需要所属方法名对应的属性，比如说：

    o.m()

python会现在对象o中搜索m属性，如果对象o有m属性(判断对象o有没有m属性，可以用hasattr函数)则调用它。

然而，python的方法是定义在一个class里的，而不是object里。也就是说如果m是o的方法，那就不可能是它的属性。正常情况下，python会先搜索对象的属性，如果没有，再去搜索类的属性，如果属性存在，则可以调用。(这地方可能大家会被类和对象两个概念搞混，不太准确的来说，类就是class，对象就是实例，具体大家可以查看文章[笨办法学Python](https://flyouting.gitbooks.io/learn-python-the-hard-way-cn/content/learn-python-the-hard-way-exercise42.html))

在python中，大多数的类都继承自object，在Python3中，如果你没有指定继承object，解释器会自动给你加上，而Python如果你没有指定，则为old-style class。大家在平时编写类时，建议大家都最好加上继承object，这样一个是代码兼容性号，一个是比较优雅。

如果属性在对象里不存在，我们会得到一个错误信息，指明了哪个地方的代码有问题和出问题的原因，但是和我们上面说的错误

    TypeError: object() takes no parameters
    
这个错误是我在创建对象实例时报的错误，例如：
    
    class Foo(object):
        pass

如果我这样：
    
    f = Foo()

就不会有任何问题，但是如果我这样：

    f = Foo(10)

然后我就会得到上面的错误，这究竟是为什么了？

这是因为Python在创建对象是，分为两个阶段：第一个阶段，对象是通过调用__new__方法来创建的，这个方法的细节我们基本上不用关心。__new__方法并不会立即返回一个对象实例，__new__方法之后，会调用__init__方法来给对象增加新的属性。对于上面的对象o，调用的就是

    o.__init__()
    
Python首先查找o的__init__方法，但是没找到，然后查找父类的__init__方法，假设父类是上面的Foo，可以方式__init__方法依然不存在，所以最后会找到object的__init__属性。object的__init__是存在的，并且是个方法，然后调用这个方法，传入相应的参数，但是object.__init__方法没有参数，然后我们就得到的上面的错误。

    TypeError: object() takes no parameters
    
整个流程下来，最让人迷惑的地方是，Python没有这样报错：

    “object.__init__()” takes no parameters
    
于是我们没法定为这个问题出在哪。

总结下来，在实现一个python的类时，最后写上__init__方法，这样就可以避免这样的迷惑性的错误。
