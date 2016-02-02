Title: coverage教程(译)
Date: 2016-02-02
Tags: python,测试,自动化测试
Category: python库
Slug: coverage
Author: 笨熊

### 简介
coverage是一个检测单元测试覆盖率的工具，即检查你的测试用例是否覆盖到了所有的代码。
#### coverage命令行工具
&emsp;&emsp;当你通过pip install coverage成功安装完coverage后，就会在python命令的同级目录，生成一个coverage可执行程序。coverage对应不同版本的python程序，生成了不同版本的可执行程序，如python2平台的coverage2，python3平台的coverage3，以及coverage-X.Y,X、Y为python的版本号。

&emsp;&emsp;coverage命令共有5个指明coverage动作的参数，分别是：

* run - 运行一个python程序并收集运行数据
* report - 生成报告
* html - 把结果输出html格式
* xml - 把结果输出xml格式
* annotate - 运行一个python程序并收集运行数据
* erase - 清楚之前coverage收集的数据
* combine - 合并coverage收集的数据
* debug - 获取调试信息
* help - 查看coverage帮助信息，coverage help 动作/coverage 动作 --help，查看指定动作的帮助信息。
* 可以通过--rcfile=FILE的方式指定命令运行时的配置文件。所有命令行的参数都可以写到配置文件里面。

#### 运行命令
* 通过coverage run命令python程序，并收集信息。

		coverage run test.py #效果和执行python test.py效果差不多

* 你也可以使用-m参数指定运行一个python文件里面的可导出的模块，例如

		coverage run -m test.test	#执行test文件里的test模块
		
* 可以通过--source，--include，--omit指定运行的python文件所在的目录。但是一定要把这三个参数放在run后面，所运行的python文件前面。

		coverage run --source=project test.py
		
&emsp;&emsp;coverage可以处理多线程的程序，但是如果你使用 multiprocessing, greenlet, eventlet, gevent，那么coverage默认情况下就处理不了了，不过可以通过--concurrency参数，指明程序具体使用的库，则可以处理。默认情况下，coverage也不会处理python解析器的代码，如python自带的标准库os、sys等，如果你也想看这些系统库的数据，使用-L参数。如果有一些代码本应该被统计到，但却没有，那么加上--timid参数再运行一遍，这是一个比较慢的跟中算法，所以一般情况下，少用。如果你有多个进程或者机器需要运行coverage程序，可以是使用--parallel-mod将所有进程的统计数据分开。

&emsp;&emsp;在运行coverage过程中，coverage会产生一些警告，这些警告会影响到统计的进程。这些警告主要包括：

* “Trace function changed, measurement is likely wrong: XXX”

如果在运行的过程中，代码发生改变，则会报这个错误，xxx表示是修改后的名称。

* “Module XXX has no Python source”

使用了一个不存在的python文件

* “Module XXX was never imported”

运行的python文件中XXX模块不存在

* “No data was collected”

主要可能是你要运行的python文件中，一行代码都没有执行到

* “Module XXX was previously imported, but not measured.”38762
模块XXX在coverage运行时已经导入了，他的运行情况不会被coverage监控到。

#### 结果文件

&emsp;&emsp;默认情况下，coverage生成的结果文件为.coverage，你可以通过修改环境变量COVERAGE_FILE来修改这个文件的后缀名。你也可以是用-a把多次运行的结果合并到一个文件里，否则，每次生成的结果文件都是上一次运行的结果。你可以是用coverage erase清空之前运行的结果文件。

#### 合并结果文件

&emsp;&emsp;coverage可以把多个结果文件合并起来，首先把多个结果文件拷贝到同一个目录，然后运行combine选项，就可以把多个文件合并到一个.coverage文件了

		coverage combine
		
你也可以指定文件名或者目录
	
		coverage combine data1.dat windows_data_files/
		
这种情况下，coverage不会收集当前目录下的文件，如果你需要收集当前目录下的结果，你需要在命令行指定。
&emsp;&emsp;coverage只会收集.coverage的文件，如下格式的文件会被收集。
		
		.coverage.machine1
		.coverage.20120807T212300
		.coverage.last_good_run.ok
可以通过run --parallel-mode参数来控制每次运行是否参数独立结果文件，如果指定，产生的结果文件名机器名、进程id、随机数。例如

		.coverage.Neds-MacBook-Pro.local.88335.316857
		.coverage.Geometer.8044.799674
		
如果你在不同的机器上不同的目录运行coverage产生的结果文件无法合并，你可以通过paths参数来指明他们间的区别。具体可以通过[paths](https://coverage.readthedocs.org/en/coverage-4.0.3/config.html#config-paths)来配置。如果合并时，结果文件不可读，coverage会输出一个警告。

#### 结果报告
&emsp;&emsp;提供四种风格的输出文件格式。分别对应html，xml命令。他们的命令行参数是一致的。如果你想收集一系列文件中的某些文件的结果，你可以指定具体的文件名和模块名。--include --omit参数可以使用正则来指定要收集的文件。指定-i --ignore-error参赛忽略那些找不到文件的错误。--fail-under可以指定一个数字，当coverage的结果小于这个数字，coverage命令返回一个错误码2，但这个参数对annotate命令无效。

#### 覆盖报告简介
&emsp;&emsp;最简单的报告是report命令输出的概要信息，report包括执行的行数，没有执行的行数，覆盖百分比。

	$ coverage report
	Name                      Stmts   Miss  Cover
	---------------------------------------------
	my_program.py                20      4    80%
	my_module.py                 15      2    86%
	my_other_module.py           56      6    89%
	---------------------------------------------
	TOTAL                        91     12    87%
-m参数可以显示具体没有被执行的文件行。

	$ coverage report -m
	Name                      Stmts   Miss  Cover   	Missing
	-------------------------------------------------------
	my_program.py                20      4    80%   33-35, 39
	my_module.py                 15      2    86%   8, 12
	my_other_module.py           56      6    89%   17-23
	-------------------------------------------------------
	TOTAL                        91     12    87%
如果你使用branch coverage,branch的结果将显示在Branch和BrPart两列。例如
	
	$ coverage report -m
	Name                      Stmts   Miss Branch BrPart  Cover   Missing
	---------------------------------------------------------------------
	my_program.py                20      4     10      2    80%   33-35, 36->38, 39
	my_module.py                 15      2      3      0    86%   8, 12
	my_other_module.py           56      6      5      1    89%   17-23, 40->45
	---------------------------------------------------------------------
	TOTAL                        91     12     18      3    87%
你指定文件来查看特定文件的结果。

	$ coverage report -m my_program.py my_other_module.py
	Name                      Stmts   Miss  Cover   Missing
	-------------------------------------------------------
	my_program.py                20      4    80%   33-35, 39
	my_other_module.py           56      6    89%   17-23
	-------------------------------------------------------
	TOTAL                        76     10    87%
--skip-covered参数可以不输出覆盖率100%的文件。