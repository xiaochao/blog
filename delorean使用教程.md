Title: delorean使用教程
Date: 2015-09-22
Tags: Python, 时间库, Time, python时间计算
Category: python库
Slug: delorean
Author: 笨熊

##首先，约定三个概念。

* navie datetime:没有指定时区的datetime对象
* localized datetime:指定时区的datetime对象
* localizing:指定市区的的navie datetime
* normalizing:datetime时区切换

##一些例子

首先，导入Delorean

	>>> from delorean import Delorean
	
使用UTC格式的当前时间创建一个datetime
	
	>>> d = Delorean()
	>>> d
	Delorean(datetime=datetime.datetime(2013, 1, 12, 6, 10, 33, 110674),  timezone='UTC')
	
简单的时区切换

	>>> d = d.shift('US/Eastern')
	>>> d
	Delorean(datetime=datetime.datetime(2013, 1, 12, 1, 10, 38, 102223), timezone='US/Eastern')
	
转换成datetime看看

	>>> d.datetime
	datetime.datetime(2013, 1, 12, 01, 10, 38, 102223, tzinfo=<DstTzInfo 'US/Eastern' EST-1 day, 19:00:00 STD>)
	>>> d.date
	datetime.date(2013, 1, 12)
	
单纯的输入时间看看
	
	>>> d.naive()
	datetime.datetime(2013, 1, 12, 1, 10, 38, 102223)
	>>> d.epoch()
	1357971038.102223
	
也是用unix时间戳初始化Delorean
	
	>>> from delorean import epoch
	>>> epoch(1357971038.102223).shift("US/Eastern")
	Delorean(datetime=datetime.datetime(2013, 1, 12, 1, 10, 38, 102223), timezone='US/Eastern')
	
初始化后，就可以方便的切换到自己所需的时区
Delorean也可以使用指定的datetime对象进行初始化，Delorean会自动处理时区和时间

	>>> tz = timezone("US/Pacific")
	>>> dt = tz.localize(datetime.utcnow())
	datetime.datetime(2013, 3, 16, 5, 28, 11, 536818, tzinfo=<DstTzInfo 'US/Pacific' PDT-1 day, 17:00:00 DST>)
	>>> d = Delorean(datetime=dt)
	>>> d
	Delorean(datetime=datetime.datetime(2013, 3, 16, 5, 28, 11, 536818), timezone='US/Pacific')
	>>> d = Delorean(datetime=dt, timezone="US/Eastern")
	>>> d
	Delorean(datetime=datetime.datetime(2013, 3, 16, 5, 28, 11, 536818), timezone='US/Pacific')
	
Delorean支持timedelta的时间加减法。Delorean可以使用timedelta进行加减，得到一个Delorean对象
	
	>>> d = Delorean()
	>>> d
	Delorean(datetime=datetime.datetime(2014, 6, 3, 19, 22, 59, 289779), timezone='UTC')
	>>> d += timedelta(hours=2)
	>>> d
	Delorean(datetime=datetime.datetime(2014, 6, 3, 21, 22, 59, 289779), timezone='UTC')
	>>> d - timedelta(hours=2)
	Delorean(datetime=datetime.datetime(2014, 6, 3, 19, 22, 59, 289779), timezone='UTC')
	>>> d2 = d + timedelta(hours=2)
	>>> d2 - d
	datetime.timedelta(0, 7200)
	
Delorean也支持两个时间比较

	>>> d1 = Delorean(datetime(2015, 1, 1), timezone='US/Pacific')
	>>> d2 = Delorean(datetime(2015, 1, 1, 8), timezone='UTC')
	>>> d1 == d2
	True
	
Delorean提供多种方法获取一个指定的时间，如明年或者下周三
Delorean提供了一些方便的方法进行如上操作。

	>>> d = Delorean()
	>>> d
	Delorean(datetime=datetime.datetime(2013, 1, 20, 19, 41, 6, 207481), timezone='UTC')
	>>> d.next_tuesday()
	Delorean(datetime=datetime.datetime(2013, 1, 22, 19, 41, 6, 207481), timezone='UTC')
	
上周二、过去第二个周二午夜

	>>> d.last_tuesday()
	Delorean(datetime=datetime.datetime(2013, 1, 15, 19, 41, 6, 207481), timezone='UTC')
	>>> d.last_tuesday(2).midnight()
	datetime.datetime(2013, 1, 8, 0, 0, tzinfo=<UTC>)
	
##过滤
通常情况下我们不关心有多少微妙或者多少秒。例如，我们很难区别同一分钟的两个datetime对象。我们补习吧不关心的字段设置为0。
Delorean提供了很方便的方法按照微妙、秒、分钟、小时进行过滤

	>>> d = Delorean()
	>>> d
	Delorean(datetime=datetime.datetime(2013, 1, 21, 3, 34, 30, 418069), timezone='UTC')
	>>> d.truncate('second')
	Delorean(datetime=datetime.datetime(2013, 1, 21, 3, 34, 30), timezone='UTC')
	>>> d.truncate('hour')
	Delorean(datetime=datetime.datetime(2013, 1, 21, 3, 0), timezone='UTC')
	
同样，也支持按照年、月份

	>>> d = Delorean(datetime=datetime(2012, 5, 15, 03, 50, 00, 555555), timezone="US/Eastern")
	>>> d
	Delorean(datetime=datetime.datetime(2012, 5, 15, 3, 50, 0, 555555), timezone='US/Eastern')
	>>> d.truncate('month')
	Delorean(datetime=datetime.datetime(2012, 5, 1), timezone='US/Eastern')
	>>> d.truncate('year')
Delorean(datetime=datetime.datetime(2012, 1, 1), timezone='US/Eastern')

##字符串处理
另一个麻烦事是处理datetime格式的字符串。Delorean可以很方便的处理

	>>> from delorean import parse
	>>> parse("2011/01/01 00:00:00 -0700")
	Delorean(datetime=datetime.datetime(2011, 1, 1, 7), timezone='UTC')	
##歧义字段的处理
Delorean提供了两个字段dayfirst=True and yearfirst=True用来处理相应格式的字符串，如果dayfirst和yearfirst是True
	
1. YY-MM-DD
2. DD-MM-YY
3. MM-DD-YY
	
默认情况下，对于May 6th, 2013格式，Delorean返回‘2013-05-06

	>>> parse("2013-05-06")
	Delorean(datetime=datetime.datetime(2013, 5, 6), timezone='UTC')
	
dayfirst和yearfirst的配置如下：

* 如果dayfirst是False，yearfirst是False
	1. MM-DD-YY
	2. DD-MM-YY
	3. YY-MM-DD
* 如果dayfirst是True，yearfirst是False
	1. DD-MM-YY
	2. MM-DD-YY
	3. YY-MM-DD
* 如果dayfirst是False，yearfirst是True
	1. YY-MM-DD
	2. MM-DD-YY
	3. DD-MM-YY
	
##时间步进
	>>> import delorean
	>>> from delorean import stops
	>>> for stop in stops(freq=delorean.HOURLY, count=10):    print stop
	...
	Delorean(datetime=datetime.datetime(2013, 1, 21, 6, 25, 33), timezone='UTC')
	Delorean(datetime=datetime.datetime(2013, 1, 21, 7, 25, 33), timezone='UTC')
	Delorean(datetime=datetime.datetime(2013, 1, 21, 8, 25, 33), timezone='UTC')
	Delorean(datetime=datetime.datetime(2013, 1, 21, 9, 25, 33), timezone='UTC')
	Delorean(datetime=datetime.datetime(2013, 1, 21, 10, 25, 33), timezone='UTC')
	Delorean(datetime=datetime.datetime(2013, 1, 21, 11, 25, 33), timezone='UTC')
	Delorean(datetime=datetime.datetime(2013, 1, 21, 12, 25, 33), timezone='UTC')
	Delorean(datetime=datetime.datetime(2013, 1, 21, 13, 25, 33), timezone='UTC')
	Delorean(datetime=datetime.datetime(2013, 1, 21, 14, 25, 33), timezone='UTC')
	Delorean(datetime=datetime.datetime(2013, 1, 21, 15, 25, 33), timezone='UTC')
注意：stops只接受naive datetime	
可以指定开始和结束的时间

	>>> for stop in stops(freq=delorean.DAILY, count=10, timezone="US/Eastern", start=d1, stop=d2):    print stop
	...
	Delorean(datetime=datetime.datetime(2012, 5, 6), timezone='US/Eastern')
	Delorean(datetime=datetime.datetime(2012, 5, 7), timezone='US/Eastern')
	Delorean(datetime=datetime.datetime(2012, 5, 8), timezone='US/Eastern')
	Delorean(datetime=datetime.datetime(2012, 5, 9), timezone='US/Eastern')
	Delorean(datetime=datetime.datetime(2012, 5, 10), timezone='US/Eastern')
	Delorean(datetime=datetime.datetime(2012, 5, 11), timezone='US/Eastern')
	Delorean(datetime=datetime.datetime(2012, 5, 12), timezone='US/Eastern')
	Delorean(datetime=datetime.datetime(2012, 5, 13), timezone='US/Eastern')
	Delorean(datetime=datetime.datetime(2012, 5, 14), timezone='US/Eastern')
	Delorean(datetime=datetime.datetime(2012, 5, 15), timezone='US/Eastern')
	
只指定结束时间是不行的
	
	>>> for stop in stops(freq=delorean.DAILY, timezone="US/Eastern", stop=d2):    print stop
	...
	Traceback (most recent call last):
		File "<stdin>", line 1, in <module
		File "delorean/interface.py", line 63, in stops
    	  bysecond=None, until=until, dtstart=start):
	TypeError: can't compare offset-naive and offset-aware datetimes
	
