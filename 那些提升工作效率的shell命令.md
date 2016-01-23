Title: 那些提升工作效率的shell命令
Date: 2016-01-23
Tags: 运维,常用shell,提升,效率
Category: 运维工具
Slug: effectiveness-shell
Author: 笨熊

####切换目录
* 注意：当前用户是xiaochao，系统为centos6，并且，shell命令是严格区分大小写的。
* 显示当前目录路径：pwd
		
		pwd
		output:/home/xiaochao/Download

* 切换目录：cd
	
	1、使用相对路径
	
		cd Download #进入当前路径下的Download目录
	2、使用绝对路径
	
		cd /var/log
		
* 点号(.)的使用
		
		cd . #表示进入当前目录
		cd .. #表示进入当前目录的上一级目录
		cd ... #表示进入当前目录的上两级目录，这个bash不支持，zsh支持。依次类推。
		
* 波浪号(~)的使用

	* 波浪号表示用户目录，即环境变量$HOME的别名，对于教程环境，就是/home/xiaochao。
	* cd命令如果不加任何参数，则相当于执行cd ~ 命令。

* 短横号(-)的使用
	
	cd - 表示上一次cd命令进入的目录，功能类似于windowns文件管理器的后腿功能。只不过，当你使用cd -进入上一次的目录，那么当前所在的目录就变成了上一次目录，举个栗子。
		
		假设当前目录是/home/xiaochao
		cd Download	#当前目录为/home/xiaochao/Download
		cd /home/xiaochao	#当前目录为/home/xiaochao
		cd -	#当前目录为/home/xiaochao/Download
		cd -	#当前目录为/home/xiaochao
		cd -	#当前目录为/home/xiaochao/Download
		cd -	#当前目录为/home/xiaochao

* 转移
	
	当我们有两个目录，并且这两个目录里内容一致，目录名不一致，常见的场景是备份目录和源目录。在两个目录之间切换，可以使用cd转移功能，举个栗子。
		
		假设我们有连个目录，/home/xiaochao/aa/bb/cc/dd,/home/xiaochao/aa.back/bb/cc/dd
		cd /home/xiaochao/aa/bb/cc/dd	#进入目录
		cd aa aa.back	#进入/home/xiaochao/aa.back/bb/cc/dd
		
####执行多个命令
* 后一个命令依赖于前一个命令的输出，可以是用管道(|)
	
		ls | wc -l  #当前目录文件个数

* 后一个命令必须等前一个命令运行成功后在运行，可以使用双与号(&&)
		
		aa && ls	#只运行aa，ls不运行

* 后一个命令必须等前一个命令运行完，不关心是否成功，使用单与号(&)
		
		aa & ls		#aa和ls都运行，但是ls必须等aa运行完。
		
* 并行执行多个命令，使用两个竖号(||)
		
		aa || ls	#aa和ls并行执行，互不影响。
		
#### ctrl键的妙用
* ctrl+a：回到当前输入/便在行首插入字符，不用按住方向键了。
* ctrl+e:与上个组合相反，回到行尾。
* ctrl+l:清空当前的终端界面，效果等同于clear命令。
* ctrl+u:清空当前输入行的所有输入。假设你输入了aa bb，按下这个组合键，aa bb就被删掉了。
* ctrl+y:就是把ctrl+u删除的字符串粘贴回来。
* ctrl+r:历史命令搜索。按下ctrl+r后，会搜索包含你输入的字符串的命令。
* ctrl+c:终止当前终端正在运行的程序。
* ctrl+d:推送当前终端。
* ctrl+z:把终端当前正在运行的程序放到后台运行。

#### 其他常用的shell命令
* $?:上一条命令的返回的结果。
* !$:上一个命令的最后一个字符串
* !!:上一个命令
* man ascii:查看ascii码表，按q退出。
* \>file.txt:创建一个文件，比touch短。
* du -s * | sort -n | tail: 列出当前目录下最大的10个文件。
* ssh user@server bash < script.sh: 远程执行一个shell脚本。不用拷贝。
* convert input.png -gravity NorthWest -background transparent -extent 720×200  output.png:改变图片的大小，不用装ps那么大的东西了。
* fgrep -r "Hello World" ./* :查询当前目标下，包含hello world的文件，-r表示查询包括子目录。
* locate:查询特定文件名的文件，但是需要安装mlocate，并且使用updatedb命令定期更新索引。
	
	
