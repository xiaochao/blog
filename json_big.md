Title: Python更快的解析JSON大文件
Meta: Python更快的解析JSON大文件
Date: 2017-07-11
Tags: Python,json,cjson,ujson,json大文件
Category: python库
Slug: json
Author: 笨熊

# Python更快的解析JSON大文件
## 提出问题
今天用python的simplejson库解析一个 >200MB 的JSON文件，发现一次decode/encode都得要 >10s，这个在我开来，实在太慢了，有没有更快的库了？

## 先给出我的简单测试结果
- json大小：245MB
- 测试方法：read文件内容，然后一次decode, 一次encode

|         | simplejson | json | ujson |
| ------  | ---------- | ---- | ------| 
| pypy    | 40s多       | 10s  | 无    |
| cpython | 12s多       |17s多  | 10s多 |

- 不成熟的结论： pypy+json最快

## 方法一：pypy+json
python自带的JSON库是用纯python代码实现的，而pypy对纯python代码的加速效果比较好。至于为什么，大家可以去google吧，很多文章解释的很好。

## 方法二：UltraJson
- 我首先想到的用C库来做JSON的解析，原因你懂的，而C语言有个JSON库叫CJSON，于是用python+cjson在google里找到了[UltraJson](!https://github.com/esnme/ultrajson)
- UltraJson是作者用C语言实现的JSON库，实际测试的效果是，整个encode的效率提升了2倍多。

## 使用方法：
- 安装：pip instal ujson

        >>> import ujson
        >>> ujson.dumps([{"key": "value"}, 81, True])
        '[{"key":"value"},81,true]'
        >>> ujson.loads("""[{"key": "value"}, 81, true]""")
        [{u'key': u'value'}, 81, True]
        
## 并不是所有情况下都适合
根据下面的BenchMark，在double数组的情况下，yajl的encode速度是比UltraJson的，所以，如果你的JSON文件较小的话，其实无所谓哪个库，如果是像我这样的大JSON文件，可以根据下面的表选择合适的JSON库。
        
## BenchMark
下面是作者给出的benchmark：表格中的数字是每秒的调用次数，也就是说，数字越大，表示效率越高。
~~~~~~~~
Versions:
~~~~~~~~

- CPython 2.7.6 (default, Jun 22 2015, 17:58:13) [GCC 4.8.2]
- blist     : 1.3.6
- simplejson: 3.8.1
- ujson     : 1.34 (0c52200eb4e2d97e548a765d5f089858c41967b0)
- yajl      : 0.3.5

| |  ujson      | yajl       | simplejson | json       |
| ------- | ----------- | ---------- | ---------- | ---------- |
| Array with 256 doubles                                                        |            |            |            |            |
| encode                                                                        |    3508.19 |    5742.00 |    3232.38 |    3309.09 |
| decode                                                                        |   25103.37 |   11257.83 |   11696.26 |   11871.04 |
| Array with 256 UTF-8 strings                                                  |            |            |            |            |
| encode                                                                        |    3189.71 |    2717.14 |    2006.38 |    2961.72 |
| decode                                                                        |    1354.94 |     630.54 |     356.35 |     344.05 |
| Array with 256 strings                                                        |            |            |            |            |
| encode                                                                        |   18127.47 |   12537.39 |   12541.23 |   20001.00 |
| decode                                                                        |   23264.70 |   12788.85 |   25427.88 |    9352.36 |
| Medium complex object                                                         |            |            |            |            |
| encode                                                                        |   10519.38 |    5021.29 |    3686.86 |    4643.47 |
| decode                                                                        |    9676.53 |    5326.79 |    8515.77 |    3017.30 |
| Array with 256 True values                                                    |            |            |            |            |
| encode                                                                        |  105998.03 |  102067.28 |   44758.51 |   60424.80 |
| decode                                                                        |  163869.96 |   78341.57 |  110859.36 |  115013.90 |
| Array with 256 dict{string, int} pairs                                        |            |            |            |            |
| encode                                                                        |   13471.32 |   12109.09 |    3876.40 |    8833.92 |
| decode                                                                        |   16890.63 |    8946.07 |   12218.55 |    3350.72 |
| Dict with 256 arrays with 256 dict{string, int} pairs                         |            |            |            |            |
| encode                                                                        |      50.25 |      46.45 |      13.82 |      29.28 |
| decode                                                                        |      33.27 |      22.10 |      27.91 |      10.43 |
| Dict with 256 arrays with 256 dict{string, int} pairs, outputting sorted keys |            |            |            |            |
| encode                                                                        |      27.19 |            |       7.75 |       2.39 |
| Complex object                                                                |            |            |            |            |
| encode                                                                        |     577.98 |            |     387.81 |     470.02 |
| decode                                                                        |     496.73 |     234.44 |     151.00 |     145.16 |


- CPython 3.4.3 (default, Oct 14 2015, 20:28:29) [GCC 4.8.4]
- blist     : 1.3.6
- simplejson: 3.8.1
- ujson     : 1.34 (0c52200eb4e2d97e548a765d5f089858c41967b0)
- yajl      : 0.3.5


|  | ujson       | yajl       | simplejson | json       |
| ------ | ----------- | ---------- | ---------- | ---------- |
| Array with 256 doubles                                                        |            |            |            |            |
| encode                                                                        |    3477.15 |    5732.24 |    3016.76 |    3071.99 |
| decode                                                                        |   23625.20 |    9731.45 |    9501.57 |    9901.92 |
| Array with 256 UTF-8 strings                                                  |            |            |            |            |
| encode                                                                        |    1995.89 |    2151.61 |    1771.98 |    1817.20 |
| decode                                                                        |    1425.04 |     625.38 |     327.14 |     305.95 |
| Array with 256 strings                                                        |            |            |            |            |
| encode                                                                        |   25461.75 |   12188.64 |   13054.76 |   14429.81 |
| decode                                                                        |   21981.31 |   17014.22 |   23869.48 |   22483.58 |
| Medium complex object                                                         |            |            |            |            |
| encode                                                                        |   10821.46 |    4837.04 |    3114.04 |    4254.46 |
| decode                                                                        |    7887.77 |    5126.67 |    4934.60 |    6204.97 |
| Array with 256 True values                                                    |            |            |            |            |
| encode                                                                        |  100452.86 |   94639.42 |   46657.63 |   60358.63 |
| decode                                                                        |  148312.69 |   75485.90 |   88434.91 |  116395.51 |
| Array with 256 dict{string, int} pairs                                        |            |            |            |            |
| encode                                                                        |   11698.13 |    8886.96 |    3043.69 |    6302.35 |
| decode                                                                        |   10686.40 |    7061.77 |    5646.80 |    7702.29 |
| Dict with 256 arrays with 256 dict{string, int} pairs                         |            |            |            |            | 
| encode                                                                        |      44.26 |      34.43 |      10.40 |      21.97 |
| decode                                                                        |      28.46 |      23.95 |      18.70 |      22.83 |
| Dict with 256 arrays with 256 dict{string, int} pairs, outputting sorted keys |            |            |            |            |
| encode                                                                        |      33.60 |            |       6.94 |      22.34 |
| Complex object                                                                |            |            |            |            |
| encode                                                                        |     432.30 |            |     351.47 |     379.34 |
| decode                                                                        |     434.40 |     221.97 |     149.57 |     147.79 |

### [请移步我的博客了解更多](http://bugcode.cn)

