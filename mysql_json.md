Title: MySQL JSON数据类型操作
Meta: mysql自5.7.8版本开始，就支持了json结构的数据存储和查询，这表明了mysql也在不断的学习和增加nosql数据库的有点
Date: 2017-10-16
Tags: mysql,json,索引
Category: mysql
Slug: mysqljson
Author: 笨熊

## 概述
mysql自5.7.8版本开始，就支持了json结构的数据存储和查询，这表明了mysql也在不断的学习和增加nosql数据库的有点。但mysql毕竟是关系型数据库，在处理json这种非结构化的数据时，还是比较别扭的。

## 创建一个JSON字段的表
首先先创建一个表，这个表包含一个json格式的字段：

    CREATE TABLE table_name (
        id INT NOT NULL AUTO_INCREMENT, 
        json_col JSON,
        PRIMARY KEY(id)
    );

上面的语句，主要注意json_col这个字段，指定的数据类型是JSON。

## 插入一条简单的JSON数据
    INSERT INTO
        table_name (json_col) 
    VALUES
        ('{"City": "Galle", "Description": "Best damn city in the world"}');
        
上面这个SQL语句，主要注意VALUES后面的部分，由于json格式的数据里，需要有双引号来标识字符串，所以，VALUES后面的内容需要用单引号包裹。

## 插入一条复杂的JSON数据
    INSERT INTO table(col) 
    VALUES('{"opening":"Sicilian","variations":["pelikan","dragon","najdorf"]}');
    
这地方，我们插入了一个json数组。主要还是注意单引号和双引号的问题。

## 修改JSON数据
之前的例子中，我们插入了几条JSON数据，但是如果我们想修改JSON数据里的某个内容，怎么实现了？比如我们向 variations 数组里增加一个元素，可以这样：

    UPDATE myjson SET dict=JSON_ARRAY_APPEND(dict,'$.variations','scheveningen') WHERE id = 2;

这个SQL语句中，$符合代表JSON字段，通过.号索引到variations字段，然后通过JSON_ARRAY_APPEND函数增加一个元素。现在我们执行查询语句：
    
    SELECT * FROM myjson
    
得到的结果是：

    +----+-----------------------------------------------------------------------------------------+
    | id | dict                                                                                    |
    +---+-----------------------------------------------------------------------------------------+
    | 2  | {"opening": "Sicilian", "variations": ["pelikan", "dragon", "najdorf", "scheveningen"]} |
    +----+-----------------------------------------------------------------------------------------+
    1 row in set (0.00 sec)
    
关于MySQL中，JSON数据的获取方法，参照官方链接[JSON Path Syntax](https://dev.mysql.com/doc/refman/5.7/en/json-path-syntax.html)

## 创建索引
MySQL的JSON格式数据不能直接创建索引，但是可以变通一下，把要搜索的数据单独拎出来，单独一个数据列，然后在这个字段上键一个索引。下面是官方的例子：

    mysql> CREATE TABLE jemp (
        ->     c JSON,
        ->     g INT GENERATED ALWAYS AS (c->"$.id"),
        ->     INDEX i (g)
        -> );
    Query OK, 0 rows affected (0.28 sec)
    
    mysql> INSERT INTO jemp (c) VALUES
         >   ('{"id": "1", "name": "Fred"}'), ('{"id": "2", "name": "Wilma"}'),
         >   ('{"id": "3", "name": "Barney"}'), ('{"id": "4", "name": "Betty"}');
    Query OK, 4 rows affected (0.04 sec)
    Records: 4  Duplicates: 0  Warnings: 0
    
    mysql> SELECT c->>"$.name" AS name
         >     FROM jemp WHERE g > 2;
    +--------+
    | name   |
    +--------+
    | Barney |
    | Betty  |
    +--------+
    2 rows in set (0.00 sec)
    
    mysql> EXPLAIN SELECT c->>"$.name" AS name
         >    FROM jemp WHERE g > 2\G
    *************************** 1. row ***************************
               id: 1
      select_type: SIMPLE
            table: jemp
       partitions: NULL
             type: range
    possible_keys: i
              key: i
          key_len: 5
              ref: NULL
             rows: 2
         filtered: 100.00
            Extra: Using where
    1 row in set, 1 warning (0.00 sec)
    
    mysql> SHOW WARNINGS\G
    *************************** 1. row ***************************
      Level: Note
       Code: 1003
    Message: /* select#1 */ select json_unquote(json_extract(`test`.`jemp`.`c`,'$.name'))
    AS `name` from `test`.`jemp` where (`test`.`jemp`.`g` > 2)
    1 row in set (0.00 sec)

这个例子很简单，就是把JSON字段里的id字段，单独拎出来成字段g，然后在字段g上做索引，查询条件也是在字段g上。

## 字符串转JSON格式
把json格式的字符串转换成MySQL的JSON类型:

    SELECT CAST('[1,2,3]' as JSON) ;
    SELECT CAST('{"opening":"Sicilian","variations":["pelikan","dragon","najdorf"]}' as JSON);
    
## 所有MYSQL JSON函数
Name      | Description
------ | -------------
JSON_APPEND()   | Append data to JSON document
JSON_ARRAY() | Create JSON array
JSON_ARRAY_APPEND() | Append data to JSON document
JSON_ARRAY_INSERT() | Insert into JSON array->  Return value from JSON column after evaluating path; equivalent to JSON_EXTRACT().
JSON_CONTAINS() | Whether JSON document contains specific object at path
JSON_CONTAINS_PATH() | Whether JSON document contains any data at path
JSON_DEPTH() | Maximum depth of JSON document
JSON_EXTRACT() |    Return data from JSON document->>   Return value from JSON column after evaluating path and unquoting the result; equivalent to JSON_UNQUOTE(JSON_EXTRACT()).
JSON_INSERT() | Insert data into JSON document
JSON_KEYS()  | Array of keys from JSON document
JSON_LENGTH() | Number of elements in JSON document
JSON_MERGE() | Merge JSON documents, preserving duplicate keys. Deprecated synonym for JSON_MERGE_PRESERVE()
JSON_MERGE_PRESERVE() | Merge JSON documents, preserving duplicate keys
JSON_OBJECT() | Create JSON object
JSON_QUOTE() | Quote JSON document
JSON_REMOVE() | Remove data from JSON document
JSON_REPLACE()   | Replace values in JSON document
JSON_SEARCH() | Path to value within JSON document
JSON_SET() | Insert data into JSON document
JSON_TYPE() |   Type of JSON value
JSON_UNQUOTE() |    Unquote JSON value
JSON_VALID() | Whether JSON value is valid
