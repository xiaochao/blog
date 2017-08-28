Title: 慎用python os库
Meta: os库里那些隐藏的坑
Date: 2017-07-18
Tags: Python,os,system,失败,异常
Category: python库
Slug: os.system
Author: 笨熊

## 概述 
最近在一个项目用，大量的使用的os.system函数，来执行各式各样的shell命令，随之而来的，是各种坑。
## 环境
- python:2.7
- os: centos6

## 问题复现
有一行代码如下：
    
    os.system('cp path1 path2')
假设path1和path2两个路径都存在，并且path1的文件比较大，大家想想，这段代码执行会有问题吗？具体问题表现是什么样的。

再有一行代码如下：
    
    os.rename('cp path1 path2')
    
假设path1和path2都是存在的，这段代码会有问题吗？

## 你猜不到的答案
- 第一行代码，正常情况下不会有什么问题，可以一旦当这个文件太大，就会引发OOM(out of memory)错误，导致cp那个命令执行失败，下面这句话画重点，这个错误不会抛异常、不会抛异常、不会抛异常。（重要的事情说三遍）
- 第二行代码，正常情况下也不会有什么问题，可以当你a b两个目录不在同一个磁盘或者是nfs这种共享目录的话，就会导致a文件在move之后不会被删除。

## 这是为什么了
- os.system 这个函数在linux环境下，只是封装了c语言的system这个函数，并且没做任何修改，所以，system函数返回什么，os.system就返回什么，而正常c语言system这个函数是启动的子进程被杀是不会发生异常的。
- os.rename 这个函数在源文件和目的文件在一个硬盘，不会有问题，但是当不同磁盘间移动时，就会出问题，用官方的话说

      The operation may fail on some Unix flavors if src and dst are on different filesystems
但是一旦成功，就是个原子操作，就是说，可mv命令一致了。

## 解决方案
- os.system 可以是用subprocess模块代替
- os.rename 使用shutil模块代替，shutil提供了文件处理相关的函数

## 最后说一句
- os库里的好多函数都已经过期了，推荐大家在以后的编码中，尽量不要使用os库里的函数，万一掉进去，都是巨深无比的坑。

