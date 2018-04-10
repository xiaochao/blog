Title: Mysql常见错误码讲解
Meta: Mysql常见错误码讲解
Date: 2017-09-04
Tags: mysql,错误码
Category: mysql
Slug: mysqlerrorcodes
Author: 笨熊

## Error code 1064: Syntax error
假设有一个sql语句

    select LastName, FirstName,from Person

执行的时候会包错误

    Error Code: 1064. You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'from Person' at line 2.
    
- 1064错误说明你的sql语句有语法错误，单看这个错误码，我们无法判断出具体是哪的错误。
- 仔细看报错信息的最后，有一段用单引号标识的对源sql语句的引用'from Person'，这表示的是这段sql语句无法被解析，但是对于我们这个例子，这个报错引用并没什么卵用。我们再注意观察，这个引用的信息前面多了一个逗号，这个逗号后面应该接的是个表中的列名，而不是from关键字。
- 1064的错误信息一般最后会有个... near '...'格式的信息，near后面的引用就是sql语句开始无法被解析的地方，当遇到这个错误，多观察这段无法解析的sql语句前后的字符。
- 有时候，你得到的错误信息是... near ''，near后面的引用是空的，这表示出错的地方位于sql语句的开头或者第一个字符，通常情况是单引号、引号、括号没有成对出现或者是结尾处没有正确的字符，如中文分号。
- 如果发现了1064错误，注意查看报错信息里引用的sql语句，多查看这个错误的sql语句前后部分。
- 如果有人向你询问1064的sql错误，你最好让他给你提供完整的sql语句和报错信息。

## Error code 1175: Safe Update
这个错误是由于你执行update或者delete语句时，没有指定where条件，如果想忽略这个错误，则修改配置

    SET SQL_SAFE_UPDATES = 0;

重新打开错误提醒

    SET SQL_SAFE_UPDATES = 1;

## 1067, 1292, 1366, 1411 - Bad Value for number, date, default, etc.
- 1067这个错误和TINESTAMP默认值有关，需查看官方文档
- 1292/1366 double和integer类型错误，检查语法和数值类型
- 1292 detatime错误，检查插入的时间数据格式，是否超出范围，带时区格式的时间字符串格式是否有问题
- 1292 VARIABLE 检查你设置的VARIABLE属性
- 1292 LOAD DATA 检查转义字符，检查数据类型
- 1411 STR_TO_DATE 检查时间字符串格式

## 1045 Access denied
权限错误，检查用户名密码是否正确，检查当前用户是否有权限访问数据。

## 1236 "impossible position" in Replication
- 通常情况下，这是由于mysql主节点挂掉了并且sync_binlog=OFF，解决方法是在从节点设置 POS=0。
- 当sync_binlog=OFF时，主节点会在把数据先发给从节点，然后写binlog。当主节点在写binlog之前挂掉了，这时候由于已经把数据发给从节点了，所以从节点在写完数据后，binlog被更新，导致主节点和从节点binlog指针位置不一致。所以，当主节点重新启动后，会开启一个新的binlog，所以这时候把从节点的binlog指针位置设置为0，从头重新开始。
- 最好的解决方法设置sync_binlog=ON,这样基于binlog同步，但会带来较多的i/o开销。

## 24 Can't open file (Too many open files) 
open_files_limit是个系统的设置，table_open_cache必须比系统的这个配置小

## 1062 - Duplicate Entry
这个错误通常有以下几个原因
1. 主键约束，Error Code: 1062. Duplicate entry ‘12’ for key ‘PRIMARY’，主键约束的数据必须是唯一的，解决的方法之一是设置主键是自增的，这样，插入数据时，设置主键的数据为NULL。
2. 唯一属性约束，Error Code: 1062. Duplicate entry ‘A’ for key ‘code’，这是你设置了数据是唯一的，但插入的数据和表中数据重复了，解决的方法是使用INSERT IGNORE代替INSERT，INSERT IGNORE插入数据的时候，如果重复了，就不做任何操作，也不报错，如果不重复，就和INSERT行为一致，插入数据。

## 126, 127, 134, 144, 145
当你访问数据时，可能会遇到这些错误。这是错误是由于mysql数据库内部错误引起的。比如：

    MySQL error code 126 = Index file is crashed
    MySQL error code 127 = Record-file is crashed
    MySQL error code 134 = Record was already deleted (or record file crashed)
    MySQL error code 144 = Table is crashed and last repair failed
    MySQL error code 145 = Table was marked as crashed and should be repaired
    
mysql的bug，被攻击了，服务挂了，不正确的关闭mysql，损坏的数据都有可能造成这些问题。当这些错误发生时，数据就无法访问了，并且一直永久的无法访问。所以，最好把数据做好备份，如果你没有备份，可以尝试去修复mysql。如果存储引擎是MyISAM，使用CHECK TABLE和REPAIR TABLE命令(mysql>=5.7)。

    CHECK TABLE <table name> ////To check the extent of database corruption
    REPAIR TABLE <table name> ////To repair table
    
## 1366
这通常意味着客户端和服务器之间的字符集处理不一致。

## 139
错误139可能意味着表定义中字段的数量和大小超过了一些限制。检查sql语句中异常长的字符串，异常大的整数等等

## 2002, 2003 Cannot connect
无法连接，如果服务正常启动，检查以下可能的项目
1、是不是防火墙的问题，关闭防火墙试试
2、检查mysql服务监听的IP
3、检查skip-name-resolve
4、检查socket文件路径

## 2014 Commands out of sync; you can't run this command now
这个是由于你运行sql查询语句的序列不正确造成的，官方的解释

    This can happen, for example, if you are using mysql_use_result() and try to execute a new query before you have called     mysql_free_result(). It can also happen if you try to execute two queries that return data without calling mysql_use_result() or mysql_store_result() in between.

总结起来意思就是你查询了结果，但是却没有把结果获取下来。造成mysql server一直在等你把结果取走。

## 1215: Cannot add foreign key constraint
添加外键错误，检查外键关联的两个字段数据类型是否一致。


