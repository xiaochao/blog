Title: Python这些问题你会吗？
Meta: 这里列举了一些python里大家不太注意的问题，虽然简单，但实现起来，还是需要技巧的。
Date: 2017-12-18
Tags: Python,问题,技巧,面试
Category: python库
Slug: python_questions
Author: 笨熊

## final作用域的代码一定会被执行吗？
正常的情况下，finally作用域的代码一定会被执行的，不管是否发生异常。哪怕是调用了sys.exit函数，finally也是会被执行的，那怎么样才能让finally代码不执行了。

```python
import time
choice = True
try:
    if choice:
        while True:
            pass
    else:
        print "Please pull the plug on your computer sometime soon..."
        time.sleep(60 * 60 * 24 * 365)
finally:
    print "Finally ..."
```
上面的代码主要是通过让流程停滞在try作用域里，从而实现了需求。上面的代码不排除有点投机取巧的意思，但是我们实习了题目的需求不是吗。

## 可以对含有任意的元素的list进行排序吗？
正常情况下：

```python
>>> a = [1, '2', '3', '1']
>>> a.sort()
>>> a
[1, '1', '2', '3']
```

那是不是以为着，任何list都可以调用sort函数进行排序了？

```python
>>> x = [1, 1j]
>>> x.sort()
Traceback (most recent call last):
  File "<pyshell#13>", line 1, in ?
    x.sort()
TypeError: cannot compare complex numbers using <, <=, >, >=
```

python里1j是一个特殊符号代表-1的平方根，出现这个问题的原因是sort函数调用的对象的__lt__函数来比较两个对象的，而复杂的数字类型是不可比较的，也就说没有实现__lt__函数，所以比较不了。因此，对于list里包含的对象如果都是可以比较的，也就是说实现了__lt__函数，那么对list调用sort函数是没问题的。

## Python可是使用++x或者x++之类的操作吗？
- ++x操作是可以的，但是这个操作产生的结果和C语言里该操作产生的结果是不一样的，Python里++x操作里的加好只是一个一元操作符，所以，++x等价于+(+x)，所以++x == x。
- x++操作是不合法的，虽然有些情况下，x++看着是合法的，比如：x++-y，但其实这个表达式等价于x+(+(-y)) = x-y，所以正常情况下，x++是不合法的。

## Python里如何实现类似于C++里的cout<<x<<y操作？
实现的方法如下：

```python
import sys

class ostream:
    def __init__(self, file):
        self.file = file
        
    def __lshift__(self, obj):
        self.file.write(str(obj));
        return self

cout = ostream(sys.stdout)
cerr = ostream(sys.stderr)
nl = '\n'

cout << x << " " << y << nl
```

这地方并不是展示了一个新的python语法，这只是对python的str对象进行了封装。

## Python里如何实现C++里的printf函数？
在python2中，print是一个表达式，python3里是个函数。所以在python2里，我们可以这么做：

```python
def printf(format, *args): print format % args,
```

上面的代码虽然只有一行，但是，有些地方还是需要注意的。第一个地方，就是最后使用了都好结尾，这样的话会更像c++的printf函数，如果想换行，则需要传入换行符。第二个地方是这个代码会在最后多打印一个空格，如果不想要这个空格，可以使用sys.stdout.write函数。第三的方面，这行代码除了更像C++风格的printf，还有其他好处吗？当然是有的，参数是比较灵活的。

## Python里逗号等号(,=)是什么意思？
你可以能见过下面的代码：

```python
>>> x ,= range(1)
>>> x
0
```

实际上，没有逗号等号(,=)这种操作符，上面的代码等价于 (x,) =  range(1)。
这只是一个赋值语句，在左边有一个元组，意味着将元组的每个元素赋给右边的相应元素; 在这种情况下，x被赋值为0

## 下面的代码是否意味着python里有阶乘的操作符？
比如下面的代码：

```python
assert 0!=1
assert 3!=6
assert 4!=24
assert 5!=120
```

其实上面的代码并不是阶乘的结果，只是有意的构造代码的结果，实际上，上面的代码等价于：

```python
assert 0 != 1
assert 3 != 6
assert 4 != 24
assert 5 != 120
```

这样一看，其实assert判断是不等于的关系，所以都是True。

## 如何快速的给Python的对象增加属性
通常我们的做法是，在对象定义的时候，定义相关的属性，那如何自由的添加对象属性了。

```python
class Struct:
    "A structure that can have any fields defined."
    def __init__(self, **entries): self.__dict__.update(entries)

>>> options = Struct(answer=42, linelen=80, font='courier')
>>> options.answer
42
>>> options.answer = 'plastics'
>>> vars(options)
{'answer': 'plastics', 'font': 'courier', 'linelen': 80}
```

## 如何定义一个包含默认值的dict
在python2.7之前，必须定义一个类来处理这样的需求，现在，可以使用collections.defaultdict和collections.Counte来实现。

```python
from collections import Counter

words = 'this is a test this is only a test'.split()

>>> Counter(words)
Counter({'this': 2, 'test': 2, 'a': 2, 'is': 2, 'only': 1})
```

## 如何计算函数的执行时间

```python
def timer(fn, *args):
    "Time the application of fn to args. Return (result, seconds)."
    import time
    start = time.clock()
    return fn(*args), time.clock() - start

>>> timer(max, range(1e6))
(999999, 0.4921875)
```
当然，python还有很多现成的轮子，可以更好的计算程序每个步骤的详细信息。

## 如何实现单例模式
网上有很多方法，但是我知道的最简单的方式如下：

```python
def singleton(object):
    "Raise an exception if an object of this class has been instantiated before."
    cls = object.__class__
    if hasattr(cls, '__instantiated'):
        raise ValueError("%s is a Singleton class but is already instantiated" % cls)
    cls.__instantiated = True

class YourClass:
    "A singleton class to do something ..."
    def __init__(self, args):
        singleton(self)
        ...
```


