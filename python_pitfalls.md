Title: Python里常见的坑
Meta: python是一门简洁的语言，但也充满着各种坑，一不小心，就万劫不复。哈哈哈
Date: 2017-08-22
Tags: Python,list,dict,坑,Pitfalls
Category: python库
Slug: Python Pitfalls
Author: 笨熊

# Python里的那些坑
- Python是一门清晰简洁的语言，如果你对一些细节不了解的话，就会掉入到那些深不见底的“坑”里，下面，我就来总结一些Python里常见的坑。

## 列表创建和引用
### 嵌套列表的创建
- 使用*号来创建一个嵌套的list:
    
        li = [[]] * 3
        print(li)
        # Out: [[], [], []]
        
- 通过这个方法，可以得到一个包含3个list的嵌套list，我们来给第一个list增加一个元素：

        li[0].append(1)
        print(li)
        # Out: [[1], [1], [1]]
         
- 通过输出的结果可以看初，我们只给第一元素增加元素，结果三个list都增加了一个元素。这是因为[[]]*3并不是创建了三个不同list，而是创建了三个指向同一个list的对象，所以，当我们操作第一个元素时，其他两个元素内容也会发生变化的原因。效果等同于下面这段代码：

        li = []
        element = [[]]
        li = element + element + element
        print(li)
        # Out: [[], [], []]
        element.append(1)
        print(li)
        # Out: [[1], [1], [1]]

- 我们可以打印出元素的内存地址一探究竟：
    
        li = [[]] * 3
        print([id(inner_list) for inner_list in li])
        # Out: [6830760, 6830760, 6830760]

- 到这我们可以明白原因了。那如何解决了？可以这样：

        li = [[] for _ in range(3)]

- 这样我们就创建了三个不同的list对象
        
        print([id(inner_list) for inner_list in li])
        # Out: [6331048, 6331528, 6331488]
        
### 列表元素的引用
- 不要使用索引方法遍历list，例如：
    
        for i in range(len(tab)):
            print(tab[i])
            
比较好的方法是：
        
        for elem in tab:
        print(elem)
        
for语句会自动生成一个迭代器。如果你需要索引位置和元素，使用enumerate函数：
        
        for i, elem in enumerate(tab):
            print((i, elem))

## 注意 == 符号的使用

        if (var == True):
            # 当var是：True、1、 1.0、 1L时if条件成立
        
        if (var != True):
            # 当var不是 True 和 1 时if条件成立
        
        if (var == False):
            # 当var是 False 或者 0 (or 0.0, 0L, 0j) if条件成立
        
        if (var == None):
            # var是None if条件成立
        
        if var:
            # 当var非空(None或者大小为0)对象 string/list/dictionary/tuple, non-0等if条件成立
        
        if not var:
            # 当var空(None或者大小为0)对象 string/list/dictionary/tuple, non-0等if条件成立

        
        if var is True:
            # 只有当var时True时 if条件成立 1也不行
        
        if var is False:
            # 只有当var时False时 if条件成立 0也不行
        
        if var is None:
        # 和var == None 一致
        
## 捕获异常由于提前检查
- 不够优雅的代码：
    
        if os.path.isfile(file_path):
            file = open(file_path)
        else:
            # do something
            
比较好的做法：

        try:
            file = open(file_path)
        except OSError as e:
            # do something
            
在python2.6+的里面可以更简洁：
        
        with open(file_path) as file:
        
之所以这么用，是这么写更加通用，比如file_path给你传个None就瞎了，还得判断是不是None，如果不判断，就又得抓异常，判断的话，代码有多写了很多。

## 类变量初始化
- 不要在对象的__init__函数之外初始化类属性，主要有两个问题
    - 如果类属性更改，则初始值更改。
    - 如果将可变对象设置为默认值，您将获得跨实例共享的相同对象。
    
    错误示范(除非你想要静态变量)
    
        class Car(object):
            color = "red"
            wheels = [Wheel(), Wheel(), Wheel(), Wheel()]
            
    正确的做法：
        
        class Car(object):
            def __init__(self):
                self.color = "red"
                self.wheels = [Wheel(), Wheel(), Wheel(), Wheel()]
                
## 函数默认参数
 
    def foo(li=[]):
        li.append(1)
        print(li)

    foo([2])
    # Out: [2, 1]
    foo([3])
    # Out: [3, 1]
    
该代码的行为与预期的一样，但如果我们不传递参数呢？
    
    foo()
    # Out: [1] As expected...
    
    foo()
    # Out: [1, 1]  Not as expected...
    
这是因为函数参数类型是定义是确认的而不是运行时，所以在两次函数调用时，li指向的是同一个list对象，如果要解决这个问题，可以这样：
    
    def foo(li=None):
        if not li:
            li = []
        li.append(1)
        print(li)
    
    foo()
    # Out: [1]
    
    foo()
    # Out: [1]
    
这虽然解决了上述的问题，但，其他的一些对象，比如零长度的字符串，输出的结果就不是我们想要的。

    x = []
    foo(li=x)
    # Out: [1]
    
    foo(li="")
    # Out: [1]
    
    foo(li=0) 
    # Out: [1]
    
最常用的办法是检查参数是不是None

    def foo(li=None):
        if li is None:
            li = []
        li.append(1)
        print(li)
    
    foo()
    # Out: [1]
    
## 在遍历时修改
- for语句在遍历对象是会生成一个迭代器，如果你在遍历的过程中修改对象，会产生意想不到的结果：

        alist = [0, 1, 2]
        for index, value in enumerate(alist):
            alist.pop(index)
        print(alist)
        # Out: [1]

    第二个元素没有被删除，因为迭代按顺序遍历索引。上述循环遍历两次，结果如下：
        
        # Iteration #1
        index = 0
        alist = [0, 1, 2]
        alist.pop(0) # removes '0'
        
        # Iteration #2
        index = 1
        alist = [1, 2]
        alist.pop(1) # removes '2'
        
        # loop terminates, but alist is not empty:
        alist = [1]

    如果避免这个问题了，可以创建另外一个list
    
        alist = [1,2,3,4,5,6,7]
        for index, item in reversed(list(enumerate(alist))):
            # delete all even items
            if item % 2 == 0:
                alist.pop(index)
        print(alist)
        # Out: [1, 3, 5, 7]

## 整数和字符串定义

- python预先缓存了一个区间的整数用来减少内存的操作，但也正是如此，有时候会出很奇特的错误，例如：

        >>> -8 is (-7 - 1)
        False
        >>> -3 is (-2 - 1)
        True
        
    另外一个例子
        
        >>> (255 + 1) is (255 + 1)
        True
        >>> (256 + 1) is (256 + 1)
        False
        
    通过不断的测试，会发现(-3,256)这区间的整数都返回True，有的甚至是(-8,257)。默认情况下，[-5,256]会在解释器第一次启动时创建并缓存，所以才会有上面的奇怪的行为。这是个很常见但很容易被忽略的一个坑。解决方案是始终使用equality（==）运算符而不是 identity（is）运算符比较值。
    
- Python还保留对常用字符串的引用，并且可以在比较is字符串的身份（即使用）时产生类似的混淆行为。

        >>> 'python' is 'py' + 'thon'
        True
        
    python字符串被缓存了，所有python字符串都是该对象的引用，对于不常见的字符串，即使字符串相等，比较身份也会失败。
    
        >>> 'this is not a common string' is 'this is not' + ' a common string'
        False
        >>> 'this is not a common string' == 'this is not' + ' a common string'
        True
    
    所以，就像整数规则一样，总是使用equal（==）运算符而不是 identity（is）运算符比较字符串值。
    
## 列表推导和循环中的变量泄漏
- 有个例子：

        i = 0
        a = [i for i in range(3)]
        print(i) # Outputs 2
    
    python2中列表推导改变了i变量的值，而python3修复了这个问题：
    
        i = 0
        a = [i for i in range(3)]
        print(i) # Outputs 0
    
    类似地，for循环对于它们的迭代变量没有私有的作用域
    
        i = 0
        for i in range(3):
            pass
        print(i) # Outputs 2
        
    这种行为发生在Python 2和Python 3中。

    为了避免泄漏变量的问题，请在列表推导和for循环中使用新的变量。
    
## or操作符
- 例如
    
        if a == 3 or b == 3 or c == 3:

    这个很简单，但是，再看一个：
        
        if a or b or c == 3: # Wrong
        
    这是由于or的优先级低于==，所以表达式将被评估为if (a) or (b) or (c == 3):。正确的方法是明确检查所有条件：
    
        if a == 3 or b == 3 or c == 3:  # Right Way
        
    或者，可以使用内置函数any()代替链接or运算符：
        
        if any([a == 3, b == 3, c == 3]): # Right
        
    或者，为了使其更有效率：
        
        if any(x == 3 for x in (a, b, c)): # Right
        
    更加简短的写法：
        
        if 3 in (a, b, c): # Right

