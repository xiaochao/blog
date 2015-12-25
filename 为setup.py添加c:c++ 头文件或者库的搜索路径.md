Title: 为setup.py添加c/c++头文件或者库的搜索路径
Date: 2015-01-09
Tags: Python, setup.py, 头文件, include
Category: python库
Slug: 
Author: 笨熊

##背景
通过源码安装python第三方库时，经常会出现该库依赖的c/c++头文件、库找不到的情况，特别是自己编译安装的c/c++库时。比如leveldb，mysql等等。我今天安装leveldb的python库时，就遇到了这个情况。

##解决：
通过setup.py的错误，定位到出错的setup.py代码行，如下：

		ext_modules = [
    		Extension(
        		'plyvel._plyvel',
        		sources=['plyvel/_plyvel.cpp', 'plyvel/comparator.cpp'],
        		libraries=['leveldb'],
        		extra_compile_args=['-Wall', '-g']
    		)	
		]
通过上网查询，得知，setup.py的ext_modules参数的详细解释，所以，只用在Extension中加上两个参数，加完后：
		
		ext_modules = [
    		Extension(
        		'plyvel._plyvel',
        		sources=['plyvel/_plyvel.cpp', 'plyvel/comparator.cpp'],
        		libraries=['leveldb'],
        		extra_compile_args=['-Wall', '-g'],
				include_dirs = ['/Users/simon/Downloads/leveldb-1.15.0/include'],
        		library_dirs = ['/Users/simon/Downloads/leveldb-1.15.0']
    		)	
		]
include_dirs指定了搜索的头文件路径，library_dirs指定了搜索的动态库或者静态库的路径

##后记
以前经常遇到，现在解决了，留个学习笔记，方便自己以后查看

setup.py参数详解：<http://blog.csdn.net/yiliumu/article/details/30841377>