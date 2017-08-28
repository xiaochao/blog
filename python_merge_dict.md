Title: Python优雅的合并两个Dict
Meta: Python优雅的合并两个Dict
Date: 2017-08-09
Tags: Python,dict,合并
Category: python库
Slug: python_merge_dict
Author: 笨熊

## 一行代码合并两个dict
假设有两个dict x和y，合并成一个新的dict，不改变 x和y的值，例如
     
     x = {'a': 1, 'b': 2}
     y = {'b': 3, 'c': 4}
期望得到一个新的结果Z，如果key相同，则y覆盖x。期望的结果是

    >>> z
    {'a': 1, 'b': 3, 'c': 4}
在PEP448中，有个新的语法可以实现，并且在python3.5中支持了该语法，合并代码如下

    z = {**x, **y}
妥妥的一行代码。
由于现在很多人还在用python2，对于python2和python3.0-python3.4的人来说，有一个比较优雅的方法，但是需要两行代码。

    z = x.copy()
    z.update(y)
上面的方法，y都会覆盖x里的内容，所以最终结果b=3.

## 不使用python3.5如何一行完成了
如果您还没有使用Python 3.5，或者需要编写向后兼容的代码，并且您希望在单个表达式中运行，则最有效的方法是将其放在一个函数中：

    def merge_two_dicts(x, y):
        """Given two dicts, merge them into a new dict as a shallow copy."""
        z = x.copy()
        z.update(y)
        return z
然后一行代码完成调用:

     z = merge_two_dicts(x, y)
你也可以定义一个函数，合并多个dict，例如

    def merge_dicts(*dict_args):
        """
        Given any number of dicts, shallow copy and merge into a new dict,
        precedence goes to key value pairs in latter dicts.
        """
        result = {}
        for dictionary in dict_args:
            result.update(dictionary)
        return result
然后可以这样使用

    z = merge_dicts(a, b, c, d, e, f, g) 
所有这些里面，相同的key，都是后面的覆盖前面的。

##  一些不够优雅的示范
### items
有些人会使用这种方法：

     z = dict(x.items() + y.items())
这其实就是在内存中创建两个列表，再创建第三个列表，拷贝完成后，创建新的dict，删除掉前三个列表。这个方法耗费性能，而且对于python3，这个无法成功执行，因为items()返回是个对象。

    >>> c = dict(a.items() + b.items())
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    TypeError: unsupported operand type(s) for +: 'dict_items' and 
    'dict_items'
你必须明确的把它强制转换成list，z = dict(list(x.items()) + list(y.items()))，这太浪费性能了。
另外，想以来于items()返回的list做并集的方法对于python3来说也会失败，而且，并集的方法，导致了重复的key在取值时的不确定，所以，如果你对两个dict合并有优先级的要求，这个方法就彻底不合适了。

    >>> x = {'a': []}
    >>> y = {'b': []}
    >>> dict(x.items() | y.items())
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    TypeError: unhashable type: 'list'
这里有一个例子，其中y应该具有优先权，但是由于任意的集合顺序，x的值被保留：

    >>> x = {'a': 2}
    >>> y = {'a': 1}
    >>> dict(x.items() | y.items())
    {'a': 2}

### 构造函数
也有人会这么用

    z = dict(x, **y)
这样用很好，比前面的两步的方法高效多了，但是可阅读性差，不够pythonic，如果当key不是字符串的时候，python3中还是运行失败

    >>> c = dict(a, **b)
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    TypeError: keyword arguments must be strings
Guido van Rossum 大神说了：宣告dict（{}，** {1：3}）是非法的，因为毕竟是滥用**机制。虽然这个方法比较hacker，但是太投机取巧了。

## 一些性能较差但是比较优雅的方法
下面这些方法，虽然性能差，但也比items方法好多了。并且支持优先级。
    
    {k: v for d in dicts for k, v in d.items()}
python2.6中可以这样

     dict((k, v) for d in dicts for k, v in d.items())
itertools.chain 将以正确的顺序将键值对上的迭代器链接：

    import itertools
    z = dict(itertools.chain(x.iteritems(), y.iteritems()))

## 性能测试
以下是在Ubuntu 14.04上完成的，在Python 2.7（系统Python）中：

    >>> min(timeit.repeat(lambda: merge_two_dicts(x, y)))
    0.5726828575134277
    >>> min(timeit.repeat(lambda: {k: v for d in (x, y) for k, v in d.items()} ))
    1.163769006729126
    >>> min(timeit.repeat(lambda: dict(itertools.chain(x.iteritems(),y.iteritems()))))
    1.1614501476287842
    >>> min(timeit.repeat(lambda: dict((k, v) for d in (x, y) for k, v in d.items())))
    2.2345519065856934

在python3.5中

    >>> min(timeit.repeat(lambda: {**x, **y}))
    0.4094954460160807
    >>> min(timeit.repeat(lambda: merge_two_dicts(x, y)))
    0.7881555100320838
    >>> min(timeit.repeat(lambda: {k: v for d in (x, y) for k, v in d.items()} ))
    1.4525277839857154
    >>> min(timeit.repeat(lambda: dict(itertools.chain(x.items(), y.items()))))
    2.3143140770262107
    >>> min(timeit.repeat(lambda: dict((k, v) for d in (x, y) for k, v in d.items())))
    3.2069112799945287
