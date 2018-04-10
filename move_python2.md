Title: Python3的这些新特性很方便
Meta: 随着Python在机器学习和数据科学领域的应用越来越广泛，相关的Python库也增长的非常快。
Date: 2018-01-31
Tags: Python3,迁移,特性
Category: python库
Slug: move_python2
Author: 笨熊

###概述
&emsp;&emsp;随着Python在机器学习和数据科学领域的应用越来越广泛，相关的Python库也增长的非常快。但是Python本身存在一个非常要命的问题，就是Python2和Python3，两个版本互不兼容，而且Github上Python2的开源库有很多不兼容Python3，导致大量的Python2的用户不愿意迁移到Python3。
&emsp;&emsp;Python3在很多方面都做出了改变，优化了Python2的很多不足，标准库也扩充了很多内容，例如协程相关的库。现在列举一些Python3里提供的功能，跟你更好的从Python2迁移到Python3的理由。

### 系统文件路径处理库：pathlib
&emsp;&emsp;使用Python2的同学，应该都用过os.path这个库，来处理各种各样的路径问题，比如拼接文件路径的函数：<code>os.path.join()</code>，用Python3，你可以使用pathlib很方便的完成这个功能：

```python
from pathlib import Path

dataset = 'wiki_images'
datasets_root = Path('/path/to/datasets/') 

train_path = datasets_root / dataset / 'train'
test_path = datasets_root / dataset / 'test'

for image_path in train_path.iterdir():
    with image_path.open() as f: # note, open is a method of Path object
        # do something with an image
```

相比与<code>os.path.join()</code>函数，pathlib更加安全、方便、可读。pathlib还有很多其他的功能。

```python
p.exists()
p.is_dir()
p.parts()
p.with_name('sibling.png') # only change the name, but keep the folder
p.with_suffix('.jpg') # only change the extension, but keep the folder and the name
p.chmod(mode)
p.rmdir()
```

### 类型提醒: Type hinting
&emsp;&emsp;在Pycharm中，类型提醒是这个样子的：
![](http://7xpx6h.com1.z0.glb.clouddn.com/bd231966d3936432ed6ef90b416e4a19
)

&emsp;&emsp;类型提醒在复杂的项目中可以很好的帮助我们规避一些手误或者类型错误，Python2的时候是靠IDE来识别，格式IDE识别方法不一致，并且只是识别，并不具备严格限定。例如有下面的代码，参数可以是numpy.array , astropy.Table and astropy.Column, bcolz, cupy, mxnet.ndarray等等。

```python
def repeat_each_entry(data):
    """ Each entry in the data is doubled 
    <blah blah nobody reads the documentation till the end>
    """
    index = numpy.repeat(numpy.arange(len(data)), 2)
    return data[index]
```
同样上面的代码，传入pandas.Series类型的参数也是可以，但是运行时会出错。

```python
repeat_each_entry(pandas.Series(data=[0, 1, 2], index=[3, 4, 5])) # returns Series with Nones inside
```

&emsp;&emsp;这还只是一个函数，对于大型的项目，会有好多这样的函数，代码很容易就跑飞了。所以确定的参数类型对于大型项目来说非常重要，而Python2没有这样的能力，Python3可以。

```python
def repeat_each_entry(data: Union[numpy.ndarray, bcolz.carray]):
```

&emsp;&emsp;目前，比如JetBrains家的PyCharm已经支持Type Hint语法检查功能，如果你使用了这个IDE，可以通过IDE功能进行实现。如果你像我一样，使用了SublimeText编辑器，那么第三方工具mypy可以帮助到你。
&emsp;&emsp;PS:目前类型提醒对ndarrays/tensors支持不是很好。

### 运行时类型检查：
正常情况下，函数的注释处理理解代码用，其他没什么用。你可以是用<code>enforce</code>来强制运行时检查类型。

```python
@enforce.runtime_validation
def foo(text: str) -> None:
    print(text)

foo('Hi') # ok
foo(5)    # fails


@enforce.runtime_validation
def any2(x: List[bool]) -> bool:
    return any(x)

any ([False, False, True, False]) # True
any2([False, False, True, False]) # True

any (['False']) # True
any2(['False']) # fails

any ([False, None, "", 0]) # False
any2([False, None, "", 0]) # fails
```

### 使用@特殊字符表示矩阵乘法
如下代码：

```python
# l2-regularized linear regression: || AX - b ||^2 + alpha * ||x||^2 -> min

# Python 2
X = np.linalg.inv(np.dot(A.T, A) + alpha * np.eye(A.shape[1])).dot(A.T.dot(b))
# Python 3
X = np.linalg.inv(A.T @ A + alpha * np.eye(A.shape[1])) @ (A.T @ b)
```

使用@符号，整个代码变得更可读和方便移植到其他如<code>numpy、tensorflow</code>等库。

### **特殊字符来递归文件路径

在Python2中，递归查找文件不是件容易的事情，即使使用glob库，但是python3中，可以通过通配符简单的实现。

```python
import glob

# Python 2
found_images = \
    glob.glob('/path/*.jpg') \
  + glob.glob('/path/*/*.jpg') \
  + glob.glob('/path/*/*/*.jpg') \
  + glob.glob('/path/*/*/*/*.jpg') \
  + glob.glob('/path/*/*/*/*/*.jpg') 

# Python 3
found_images = glob.glob('/path/**/*.jpg', recursive=True)
```

和之前提到的<code>pathlib</code>一起使用，效果更好：

```python
# Python 3
found_images = pathlib.Path('/path/').glob('**/*.jpg')
```

### Print函数
打印到指定文件

```python
print >>sys.stderr, "critical error"      # Python 2
print("critical error", file=sys.stderr)  # Python 3
```

不使用join函数拼接字符串

```python
# Python 3
print(*array, sep='\t')
print(batch, epoch, loss, accuracy, time, sep='\t')
```

重写print函数

```python
# Python 3
_print = print # store the original print function
def print(*args, **kargs):
    pass  # do something useful, e.g. store output to some file
```
再比如下面的代码

```python
@contextlib.contextmanager
def replace_print():
    import builtins
    _print = print # saving old print function
    # or use some other function here
    builtins.print = lambda *args, **kwargs: _print('new printing', *args, **kwargs)
    yield
    builtins.print = _print

with replace_print():
    <code here will invoke other print function>
```
虽然上面这段代码也能达到重写print函数的目的，但是不推荐使用。

### 字符串格式化
python2提供的字符串格式化系统还是不够好，太冗长麻烦，通常我们会写这样一段代码来输出日志信息：

```python
# Python 2
print('{batch:3} {epoch:3} / {total_epochs:3}  accuracy: {acc_mean:0.4f}±{acc_std:0.4f} time: {avg_time:3.2f}'.format(
    batch=batch, epoch=epoch, total_epochs=total_epochs, 
    acc_mean=numpy.mean(accuracies), acc_std=numpy.std(accuracies),
    avg_time=time / len(data_batch)
))

# Python 2 (too error-prone during fast modifications, please avoid):
print('{:3} {:3} / {:3}  accuracy: {:0.4f}±{:0.4f} time: {:3.2f}'.format(
    batch, epoch, total_epochs, numpy.mean(accuracies), numpy.std(accuracies),
    time / len(data_batch)
))

```
输出的结果是：

```python
120  12 / 300  accuracy: 0.8180±0.4649 time: 56.60
```

python3.6的f-strings功能实现起来就简单多了。

```python
# Python 3.6+
print(f'{batch:3} {epoch:3} / {total_epochs:3}  accuracy: {numpy.mean(accuracies):0.4f}±{numpy.std(accuracies):0.4f} time: {time / len(data_batch):3.2f}')
```

而且，在编写查询或生成代码片段时非常方便：

```python
query = f"INSERT INTO STATION VALUES (13, '{city}', '{state}', {latitude}, {longitude})"
```

### 严格排序
下面这些比较操作在python3里是非法的

```python
# All these comparisons are illegal in Python 3
3 < '3'
2 < None
(3, 4) < (3, None)
(4, 5) < [4, 5]

# False in both Python 2 and Python 3
(4, 5) == [4, 5]
```

不同类型的数据无法排序

```python
sorted([2, '1', 3])  # invalid for Python 3, in Python 2 returns [2, 3, '1']
```

### NLP Unicode问题
```python
s = '您好'
print(len(s))
print(s[:2])

Output:

Python 2: 6\n��
Python 3: 2\n您好.


x = u'со'
x += 'co' # ok
x += 'со' # fail
```
下面这段代码在Python2里运行失败但是Python3会成功运行，Python3的字符串都是Unicode编码，所以这样对NLP来说很方便，再比如：

```python
'a' < type < u'a'  # Python 2: True
'a' < u'a'         # Python 2: False
```
```Python
from collections import Counter
Counter('Möbelstück')
Python 2: Counter({'\xc3': 2, 'b': 1, 'e': 1, 'c': 1, 'k': 1, 'M': 1, 'l': 1, 's': 1, 't': 1, '\xb6': 1, '\xbc': 1})
Python 3: Counter({'M': 1, 'ö': 1, 'b': 1, 'e': 1, 'l': 1, 's': 1, 't': 1, 'ü': 1, 'c': 1, 'k': 1})
```

### 字典
CPython3.6+里的dict默认的行为和orderdict很类似

```python
import json
x = {str(i):i for i in range(5)}
json.loads(json.dumps(x))
# Python 2
{u'1': 1, u'0': 0, u'3': 3, u'2': 2, u'4': 4}
# Python 3
{'0': 0, '1': 1, '2': 2, '3': 3, '4': 4}
```

同样的，**kwargs字典内容的数据和传入参数的顺序是一致的。

```python
from torch import nn

# Python 2
model = nn.Sequential(OrderedDict([
          ('conv1', nn.Conv2d(1,20,5)),
          ('relu1', nn.ReLU()),
          ('conv2', nn.Conv2d(20,64,5)),
          ('relu2', nn.ReLU())
        ]))

# Python 3.6+, how it *can* be done, not supported right now in pytorch
model = nn.Sequential(
    conv1=nn.Conv2d(1,20,5),
    relu1=nn.ReLU(),
    conv2=nn.Conv2d(20,64,5),
    relu2=nn.ReLU())
)        
```

### Iterable unpacking
```python
# handy when amount of additional stored info may vary between experiments, but the same code can be used in all cases
model_paramteres, optimizer_parameters, *other_params = load(checkpoint_name)

# picking two last values from a sequence
*prev, next_to_last, last = values_history

# This also works with any iterables, so if you have a function that yields e.g. qualities,
# below is a simple way to take only last two values from a list 
*prev, next_to_last, last = iter_train(args)
```

### 更高性能的默认pickle engine
```python
# Python 2
import cPickle as pickle
import numpy
print len(pickle.dumps(numpy.random.normal(size=[1000, 1000])))
# result: 23691675

# Python 3
import pickle
import numpy
len(pickle.dumps(numpy.random.normal(size=[1000, 1000])))
# result: 8000162
```
缩短到Python2时间的1/3

### 更安全的列表推导
```python
labels = <initial_value>
predictions = [model.predict(data) for data, labels in dataset]

# labels are overwritten in Python 2
# labels are not affected by comprehension in Python 3
```

### 更简易的super()
```python
# Python 2
class MySubClass(MySuperClass):
    def __init__(self, name, **options):
        super(MySubClass, self).__init__(name='subclass', **options)
        
# Python 3
class MySubClass(MySuperClass):
    def __init__(self, name, **options):
        super().__init__(name='subclass', **options)
```

### Multiple unpacking
合并两个Dict

```python
x = dict(a=1, b=2)
y = dict(b=3, d=4)
# Python 3.5+
z = {**x, **y} 
# z = {'a': 1, 'b': 3, 'd': 4}, note that value for `b` is taken from the latter dict.
```
Python3.5+不仅仅合并dict很方便，合并list等也很方便

```python
[*a, *b, *c] # list, concatenating 
(*a, *b, *c) # tuple, concatenating 
{*a, *b, *c} # set, union 
```
```python
Python 3.5+
do_something(**{**default_settings, **custom_settings})

# Also possible, this code also checks there is no intersection between keys of dictionaries
do_something(**first_args, **second_args)
```
 
### 整数类型
python2提供了两个整数类型：int和long，python3只提供有个整数类型：int，如下的代码：

```python
isinstance(x, numbers.Integral) # Python 2, the canonical way
isinstance(x, (long, int))      # Python 2
isinstance(x, int)              # Python 3, easier to remember
``` 

### 总结
python3提供了很多新的特性，方便我们编码的同时，也带来了更好的安全性和较高的性能。而且官方也一直推荐尽快迁移到python3。当然，迁移的代价因系统而异，希望这篇文章能对你迁移python2到python3有些帮助。

### 相关文章
- [英文原文](https://github.com/arogozhnikov/python3_with_pleasure)



