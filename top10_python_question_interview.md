Title: 10个Python面试常问的问题
Meta: Python是个非常受欢迎的编程语言，随着近些年机器学习、云计算等技术的发展，Python的职位需求越来越高。下面我收集了10个Python面试官经常问的问题，供大家参考学习。
Date: 2017-12-04
Tags: Python
Category: python库
Slug: PythonQuestions
Author: 笨熊

## 概述
Python是个非常受欢迎的编程语言，随着近些年机器学习、云计算等技术的发展，Python的职位需求越来越高。下面我收集了10个Python面试官经常问的问题，供大家参考学习。

## 类继承
有如下的一段代码：

    class A(object):
        def show(self):
            print 'base show'
 
    class B(A):
        def show(self):
            print 'derived show'
     
    obj = B()
    obj.show()
    
如何调用类A的show方法了。
方法如下：

    obj.__class__ = A
    obj.show() 
    
\__class__方法指向了类对象，只用给他赋值类型A，然后调用方法show，但是用完了记得修改回来。

## 方法对象
问题：为了让下面这段代码运行，需要增加哪些代码？

    class A(object):
        def __init__(self,a,b):
            self.__a = a
            self.__b = b
        def myprint(self):
            print 'a=', self.__a, 'b=', self.__b
        
     
    a1=A(10,20)
    a1.myprint()
     
    a1(80)

答案：为了能让对象实例能被直接调用，需要实现__call__方法

    class A(object):
        def __init__(self, a, b):
            self.__a = a
            self.__b = b
        def myprint(self):
            print 'a=', self.__a, 'b=', self.__b
        def __call__(self, num):
            print 'call:', num + self.__a

## new和init
下面这段代码输入什么？

    class B(object):
        def fn(self):
            print 'B fn'
        def __init__(self):
            print "B INIT"
     
     
    class A(object):
        def fn(self):
            print 'A fn'
     
        def __new__(cls,a):
                print "NEW", a
                if a>10:
                    return super(A, cls).__new__(cls)
                return B()
     
        def __init__(self,a):
            print "INIT", a
     
    a1 = A(5)
    a1.fn()
    a2=A(20)
    a2.fn()
    
答案：
    
    NEW 5
    B INIT
    B fn
    NEW 20
    INIT 20
    A fn

使用__new__方法，可以决定返回那个对象，也就是创建对象之前，这个可以用于设计模式的单例、工厂模式。__init__是创建对象是调用的。

## Python list和dict生成
下面这段代码输出什么?

    ls = [1,2,3,4]
    list1 = [i for i in ls if i>2]
    print list1
     
    list2 = [i*2 for i in ls if i>2]
    print list2
     
    dic1 = {x: x**2 for x in (2, 4, 6)}
    print dic1
     
    dic2 = {x: 'item' + str(x**2) for x in (2, 4, 6)}
    print dic2
     
    set1 = {x for x in 'hello world' if x not in 'low level'}
    print set1
    
答案:
    
    [3, 4]  
    [6, 8]
    {2: 4, 4: 16, 6: 36}
    {2: 'item4', 4: 'item16', 6: 'item36'}
    set(['h', 'r', 'd'])
    
## 全局和局部变量
下面这段代码输出什么?

    num = 9
 
    def f1():
        num = 20
     
    def f2():
        print num
     
     
    f2()
    f1()
    f2()
    
答案:
    
    9
    9

num不是个全局变量，所以每个函数都得到了自己的num拷贝，如果你想修改num，则必须用global关键字声明。比如下面这样

    num = 9
     
    def f1():
        global num
        num = 20
     
    def f2():
       print num
     
    f2()
    f1()
    f2()
     
    # prints:
    #      9
    #      20
    
## 交换两个变量的值
一行代码交换两个变量值

    a=8
    b=9
    
答案：
    
    (a,b) = (b,a)
    
## 默认方法
如下的代码
    
    class A(object):
        def __init__(self,a,b):
            self.a1 = a
            self.b1 = b
            print 'init'
        def mydefault(self):
            print 'default'
     
    a1 = A(10,20)
    a1.fn1()
    a1.fn2()
    a1.fn3()
    
方法 fn1/fn2/fn3 都没有定义，添加代码，是没有定义的方法都调用mydefault函数，上面的代码应该输出

    default
    default
    default
    
答案：

    class A(object):
        def __init__(self,a,b):
            self.a1 = a
            self.b1 = b
            print 'init'
        def mydefault(self):
            print 'default'
        def __getattr__(self,name):
            return self.mydefault
     
    a1 = A(10,20)
    a1.fn1()
    a1.fn2()
    a1.fn3()
    
方法__getattr__只有当没有定义的方法调用时，才是调用他。当fn1方法传入参数时，我们可以给mydefault方法增加一个*args不定参数来兼容。

    class A(object):
        def __init__(self,a,b):
            self.a1 = a
            self.b1 = b
            print 'init'
        def mydefault(self,*args):
            print 'default:' + str(args[0])
        def __getattr__(self,name):
            print "other fn:",name
            return self.mydefault
     
    a1 = A(10,20)
    a1.fn1(33)
    a1.fn2('hello')
    a1.fn3(10)
    
## 包管理
一个包里有三个模块，mod1.py, mod2.py, mod3.py，但使用from demopack import *导入模块时，如何保证只有mod1、mod3被导入了。

答案:增加__init__.py文件，并在文件中增加：

    __all__ = ['mod1','mod3']
    
## 闭包
写一个函数，接收整数参数n，返回一个函数，函数的功能是把函数的参数和n相乘并把结果返回。

答案:
    
    def mulby(num):
        def gn(val):
            return num * val
     
        return gn
     
     
    zw = mulby(7)
    print(zw(9));
    
## 性能
解析下面的代码慢在哪

    def strtest1(num):
        str='first'
        for i in range(num):
            str+="X"
        return str
        
答案：python的str是个不可变对象，每次迭代，都会生成新的str对象来存储新的字符串，num越大，创建的str对象越多，内存消耗越大。

