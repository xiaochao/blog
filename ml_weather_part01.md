Title: 使用机器学习预测天气(第一部分)
Meta: 本章是使用机器学习预测天气系列教程的第一部分，使用Python和机器学习来构建模型，根据从Weather Underground收集的数据来预测天气温度。
Date: 2017-12-28
Tags: Python,机器学习,天气预测
Category: 机器学习
Slug: mlweatherpart01
Author: 笨熊

# 使用机器学习预测天气(第一部分)
## 概述
&emsp;&emsp;本章是使用机器学习预测天气系列教程的第一部分，使用Python和机器学习来构建模型，根据从Weather Underground收集的数据来预测天气温度。该教程将由三个不同的部分组成，涵盖的主题是：

- 数据收集和处理（本文）
- 线性回归模型（第2章）
- 神经网络模型（第3章）

&emsp;&emsp;本教程中使用的数据将从Weather Underground的免费层API服务中收集。我将使用python的requests库来调用API，得到从2015年起Lincoln, Nebraska的天气数据。 一旦收集完成，数据将需要进行处理并汇总转成合适的格式，然后进行清理。
&emsp;&emsp;第二篇文章将重点分析数据中的趋势，目标是选择合适的特性并使用python的statsmodels和scikit-learn库来构建线性回归模型。 我将讨论构建线性回归模型，必须进行必要的假设，并演示如何评估数据特征以构建一个健壮的模型。 并在最后完成模型的测试与验证。
&emsp;&emsp;最后的文章将着重于使用神经网络。 我将比较构建神经网络模型和构建线性回归模型的过程，结果，准确性。

## Weather Underground介绍
&emsp;&emsp;Weather Underground是一家收集和分发全球各种天气测量数据的公司。 该公司提供了大量的API，可用于商业和非商业用途。 在本文中，我将介绍如何使用非商业API获取每日天气数据。所以，如果你跟随者本教程操作的话，您需要注册他们的免费开发者帐户。 此帐户提供了一个API密钥，这个密钥限制，每分钟10个，每天500个API请求。
&emsp;&emsp;获取历史数据的API如下：

```python
http://api.wunderground.com/api/API_KEY/history_YYYYMMDD/q/STATE/CITY.json  
```
- API_KEY: 注册账户获取
- YYYYMMDD: 你想要获取的天气数据的日期
- STATE: 州名缩写
- CITY: 你请求的城市名

## 调用API
&emsp;&emsp;本教程调用Weather Underground API获取历史数据时，用到如下的python库。

名称   | 描述    |来源
------ | ------ | -------
datetime | 处理日期 | 标准库
time | 处理时间  | 标准库
collections | 使用该库的namedtuples来结构化数据 | 标准库
pandas | 处理数据  | 第三方
requests | HTTP请求处理库 | 第三方
matplotlib | 制图库 |第三方

&emsp;&emsp;好，我们先导入这些库：

```python
from datetime import datetime, timedelta  
import time  
from collections import namedtuple  
import pandas as pd  
import requests  
import matplotlib.pyplot as plt  
```
接下里，定义常量来保存API_KEY和BASE_URL，注意，例子中的API_KEY不可用，你要自己注册获取。代码如下：

```python
API_KEY = '7052ad35e3c73564'  
# 第一个大括号是API_KEY，第二个是日期
BASE_URL = "http://api.wunderground.com/api/{}/history_{}/q/NE/Lincoln.json"  
```

然后我们初始化一个变量，存储日期，然后定义一个list，指明要从API返回的内容里获取的数据。然后定义一个namedtuple类型的变量DailySummary来存储返回的数据。代码如下：

```python
target_date = datetime(2016, 5, 16)  
features = ["date", "meantempm", "meandewptm", "meanpressurem", "maxhumidity", "minhumidity", "maxtempm",  
            "mintempm", "maxdewptm", "mindewptm", "maxpressurem", "minpressurem", "precipm"]
DailySummary = namedtuple("DailySummary", features)  
```

定义一个函数，调用API，获取指定target_date开始的days天的数据，代码如下：

```python
def extract_weather_data(url, api_key, target_date, days):  
    records = []
    for _ in range(days):
        request = BASE_URL.format(API_KEY, target_date.strftime('%Y%m%d'))
        response = requests.get(request)
        if response.status_code == 200:
            data = response.json()['history']['dailysummary'][0]
            records.append(DailySummary(
                date=target_date,
                meantempm=data['meantempm'],
                meandewptm=data['meandewptm'],
                meanpressurem=data['meanpressurem'],
                maxhumidity=data['maxhumidity'],
                minhumidity=data['minhumidity'],
                maxtempm=data['maxtempm'],
                mintempm=data['mintempm'],
                maxdewptm=data['maxdewptm'],
                mindewptm=data['mindewptm'],
                maxpressurem=data['maxpressurem'],
                minpressurem=data['minpressurem'],
                precipm=data['precipm']))
        time.sleep(6)
        target_date += timedelta(days=1)
    return records
```

首先，定义个list records，用来存放上述的DailySummary，使用for循环来遍历指定的所有日期。然后生成url，发起HTTP请求，获取返回的数据，使用返回的数据，初始化DailySummary，最后存放到records里。通过这个函数的出，就可以获取到指定日期开始的N天的历史天气数据，并返回。

## 获取500天的天气数据
&emsp;&emsp;由于API接口的限制，我们需要两天的时间才能获取到500天的数据。你也可以下载我的测试数据，来节约你的时间。

```python
records = extract_weather_data(BASE_URL, API_KEY, target_date, 500)  
```

## 格式化数据为Pandas DataFrame格式
&emsp;&emsp;我们使用DailySummary列表来初始化Pandas DataFrame。DataFrame数据类型是机器学习领域经常会用到的数据结构。

```python
df = pd.DataFrame(records, columns=features).set_index('date')
```

## 特征提取
&emsp;&emsp;机器学习是带有实验性质的，所以，你可能遇到一些矛盾的数据或者行为。因此，你需要在你用机器学习处理问题是，你需要对处理的问题领域有一定的了解，这样可以更好的提取数据特征。
&emsp;&emsp;我将采用如下的数据字段，并且，使用过去三天的数据作为预测。

- mean temperature
- mean dewpoint
- mean pressure
- max humidity
- min humidity
- max dewpoint
- min dewpoint
- max pressure
- min pressure
- precipitation

首先我需要在DataFrame里增加一些字段来保存新的数据字段，为了方便测试，我创建了一个tmp变量，存储10个数据，这些数据都有meantempm和meandewptm属性。代码如下：

```python
tmp = df[['meantempm', 'meandewptm']].head(10)  
tmp  
```
![](http://7xpx6h.com1.z0.glb.clouddn.com/ae9fba86ccfb28be99c493c36a2b5f61
)

对于每一行的数据，我们分别获取他前一天、前两天、前三天对应的数据，存在本行，分别以属性_index来命名，代码如下：

```python
# 1 day prior
N = 1

# target measurement of mean temperature
feature = 'meantempm'

# total number of rows
rows = tmp.shape[0]

# a list representing Nth prior measurements of feature
# notice that the front of the list needs to be padded with N
# None values to maintain the constistent rows length for each N
nth_prior_measurements = [None]*N + [tmp[feature][i-N] for i in range(N, rows)]

# make a new column name of feature_N and add to DataFrame
col_name = "{}_{}".format(feature, N)  
tmp[col_name] = nth_prior_measurements  
tmp  
```
![](http://7xpx6h.com1.z0.glb.clouddn.com/ea7572c6529c9b066e608f026b285adf
)

我们现在把上面的处理过程封装成一个函数，方便调用。

```python
def derive_nth_day_feature(df, feature, N):  
    rows = df.shape[0]
    nth_prior_measurements = [None]*N + [df[feature][i-N] for i in range(N, rows)]
    col_name = "{}_{}".format(feature, N)
    df[col_name] = nth_prior_measurements
```
好，我们现在对所有的特征，都取过去三天的数据，放在本行。

```python
for feature in features:  
    if feature != 'date':
        for N in range(1, 4):
            derive_nth_day_feature(df, feature, N)
```

处理完后，我们现在的所有数据特征为：

```python
df.columns  

Index(['meantempm', 'meandewptm', 'meanpressurem', 'maxhumidity',  
       'minhumidity', 'maxtempm', 'mintempm', 'maxdewptm', 'mindewptm',
       'maxpressurem', 'minpressurem', 'precipm', 'meantempm_1', 'meantempm_2',
       'meantempm_3', 'meandewptm_1', 'meandewptm_2', 'meandewptm_3',
       'meanpressurem_1', 'meanpressurem_2', 'meanpressurem_3',
       'maxhumidity_1', 'maxhumidity_2', 'maxhumidity_3', 'minhumidity_1',
       'minhumidity_2', 'minhumidity_3', 'maxtempm_1', 'maxtempm_2',
       'maxtempm_3', 'mintempm_1', 'mintempm_2', 'mintempm_3', 'maxdewptm_1',
       'maxdewptm_2', 'maxdewptm_3', 'mindewptm_1', 'mindewptm_2',
       'mindewptm_3', 'maxpressurem_1', 'maxpressurem_2', 'maxpressurem_3',
       'minpressurem_1', 'minpressurem_2', 'minpressurem_3', 'precipm_1',
       'precipm_2', 'precipm_3'],
      dtype='object')
```

## 数据清洗
&emsp;&emsp;数据清洗时机器学习过程中最重要的一步，而且非常的耗时、费力。本教程中，我们会去掉不需要的样本、数据不完整的样本，查看数据的一致性等。
&emsp;&emsp;首先去掉我不感兴趣的数据，来减少样本集。我们的目标是根据过去三天的天气数据预测天气温度，因此我们只保留min, max, mean三个字段的数据。

```python
# make list of original features without meantempm, mintempm, and maxtempm
to_remove = [feature  
             for feature in features 
             if feature not in ['meantempm', 'mintempm', 'maxtempm']]

# make a list of columns to keep
to_keep = [col for col in df.columns if col not in to_remove]

# select only the columns in to_keep and assign to df
df = df[to_keep]  
df.columns
Index(['meantempm', 'maxtempm', 'mintempm', 'meantempm_1', 'meantempm_2',  
       'meantempm_3', 'meandewptm_1', 'meandewptm_2', 'meandewptm_3',
       'meanpressurem_1', 'meanpressurem_2', 'meanpressurem_3',
       'maxhumidity_1', 'maxhumidity_2', 'maxhumidity_3', 'minhumidity_1',
       'minhumidity_2', 'minhumidity_3', 'maxtempm_1', 'maxtempm_2',
       'maxtempm_3', 'mintempm_1', 'mintempm_2', 'mintempm_3', 'maxdewptm_1',
       'maxdewptm_2', 'maxdewptm_3', 'mindewptm_1', 'mindewptm_2',
       'mindewptm_3', 'maxpressurem_1', 'maxpressurem_2', 'maxpressurem_3',
       'minpressurem_1', 'minpressurem_2', 'minpressurem_3', 'precipm_1',
       'precipm_2', 'precipm_3'],
      dtype='object')
```

为了更好的观察数据，我们使用Pandas的一些内置函数来查看数据信息，首先我们使用info()函数，这个函数会输出DataFrame里存放的数据信息。

```python
df.info()
<class 'pandas.core.frame.DataFrame'>  
DatetimeIndex: 1000 entries, 2015-01-01 to 2017-09-27  
Data columns (total 39 columns):  
meantempm          1000 non-null object  
maxtempm           1000 non-null object  
mintempm           1000 non-null object  
meantempm_1        999 non-null object  
meantempm_2        998 non-null object  
meantempm_3        997 non-null object  
meandewptm_1       999 non-null object  
meandewptm_2       998 non-null object  
meandewptm_3       997 non-null object  
meanpressurem_1    999 non-null object  
meanpressurem_2    998 non-null object  
meanpressurem_3    997 non-null object  
maxhumidity_1      999 non-null object  
maxhumidity_2      998 non-null object  
maxhumidity_3      997 non-null object  
minhumidity_1      999 non-null object  
minhumidity_2      998 non-null object  
minhumidity_3      997 non-null object  
maxtempm_1         999 non-null object  
maxtempm_2         998 non-null object  
maxtempm_3         997 non-null object  
mintempm_1         999 non-null object  
mintempm_2         998 non-null object  
mintempm_3         997 non-null object  
maxdewptm_1        999 non-null object  
maxdewptm_2        998 non-null object  
maxdewptm_3        997 non-null object  
mindewptm_1        999 non-null object  
mindewptm_2        998 non-null object  
mindewptm_3        997 non-null object  
maxpressurem_1     999 non-null object  
maxpressurem_2     998 non-null object  
maxpressurem_3     997 non-null object  
minpressurem_1     999 non-null object  
minpressurem_2     998 non-null object  
minpressurem_3     997 non-null object  
precipm_1          999 non-null object  
precipm_2          998 non-null object  
precipm_3          997 non-null object  
dtypes: object(39)  
memory usage: 312.5+ KB
```
注意：每一行的数据类型都是object，我们需要把数据转成float。

```python
df = df.apply(pd.to_numeric, errors='coerce')  
df.info()
<class 'pandas.core.frame.DataFrame'>  
DatetimeIndex: 1000 entries, 2015-01-01 to 2017-09-27  
Data columns (total 39 columns):  
meantempm          1000 non-null int64  
maxtempm           1000 non-null int64  
mintempm           1000 non-null int64  
meantempm_1        999 non-null float64  
meantempm_2        998 non-null float64  
meantempm_3        997 non-null float64  
meandewptm_1       999 non-null float64  
meandewptm_2       998 non-null float64  
meandewptm_3       997 non-null float64  
meanpressurem_1    999 non-null float64  
meanpressurem_2    998 non-null float64  
meanpressurem_3    997 non-null float64  
maxhumidity_1      999 non-null float64  
maxhumidity_2      998 non-null float64  
maxhumidity_3      997 non-null float64  
minhumidity_1      999 non-null float64  
minhumidity_2      998 non-null float64  
minhumidity_3      997 non-null float64  
maxtempm_1         999 non-null float64  
maxtempm_2         998 non-null float64  
maxtempm_3         997 non-null float64  
mintempm_1         999 non-null float64  
mintempm_2         998 non-null float64  
mintempm_3         997 non-null float64  
maxdewptm_1        999 non-null float64  
maxdewptm_2        998 non-null float64  
maxdewptm_3        997 non-null float64  
mindewptm_1        999 non-null float64  
mindewptm_2        998 non-null float64  
mindewptm_3        997 non-null float64  
maxpressurem_1     999 non-null float64  
maxpressurem_2     998 non-null float64  
maxpressurem_3     997 non-null float64  
minpressurem_1     999 non-null float64  
minpressurem_2     998 non-null float64  
minpressurem_3     997 non-null float64  
precipm_1          889 non-null float64  
precipm_2          889 non-null float64  
precipm_3          888 non-null float64  
dtypes: float64(36), int64(3)  
memory usage: 312.5 KB  
```
现在得到我想要的数据了。接下来我们调用describe()函数，这个函数会返回一个DataFrame，这个返回值包含了总数、平均数、标准差、最小、25%、50%、75%、最大的数据信息。

&emsp;&emsp;接下来，使用四分位的方法，去掉25%数据里特别小的和75%数据里特别大的数据。

```python
# Call describe on df and transpose it due to the large number of columns
spread = df.describe().T

# precalculate interquartile range for ease of use in next calculation
IQR = spread['75%'] - spread['25%']

# create an outliers column which is either 3 IQRs below the first quartile or
# 3 IQRs above the third quartile
spread['outliers'] = (spread['min']<(spread['25%']-(3*IQR)))|(spread['max'] > (spread['75%']+3*IQR))

# just display the features containing extreme outliers
spread.ix[spread.outliers,]  
```
![](http://7xpx6h.com1.z0.glb.clouddn.com/8ce8a308b12f48ca6ff13c861a94dcbd
)
&emsp;&emsp;评估异常值的潜在影响是任何分析项目的难点。 一方面，您需要关注引入虚假数据样本的可能性，这些样本将严重影响您的模型。 另一方面，异常值对于预测在特殊情况下出现的结果是非常有意义的。 我们将讨论每一个包含特征的异常值，看看我们是否能够得出合理的结论来处理它们。

&emsp;&emsp;第一组特征看起来与最大湿度有关。 观察这些数据，我可以看出，这个特征类别的异常值是非常低的最小值。这数据看起来没价值，我想我想仔细看看它，最好是以图形方式。 要做到这一点，我会使用直方图。

```python
%matplotlib inline
plt.rcParams['figure.figsize'] = [14, 8]  
df.maxhumidity_1.hist()  
plt.title('Distribution of maxhumidity_1')  
plt.xlabel('maxhumidity_1')  
plt.show()
```
![](http://7xpx6h.com1.z0.glb.clouddn.com/dfe783975866aead6b8642856d3f4699
)
查看maxhumidity字段的直方图，数据表现出相当多的负偏移。 在选择预测模型和评估最大湿度影响的强度时，我会牢记这一点。 许多基本的统计方法都假定数据是正态分布的。 现在我们暂时不管它，但是记住这个异常特性。

&emsp;&emsp;接下来我们看另外一个字段的直方图

```python
df.minpressurem_1.hist()  
plt.title('Distribution of minpressurem_1')  
plt.xlabel('minpressurem_1')  
plt.show() 
```
![](http://7xpx6h.com1.z0.glb.clouddn.com/90ffb866175c0b4f9adaf6660bbdabc3
)

&emsp;&emsp;要解决的最后一个数据质量问题是缺失值。 由于我构建DataFrame的时候，缺少的值由NaN表示。 您可能会记得，我通过推导代表前三天测量结果的特征，有意引入了收集数据前三天的缺失值。 直到第三天我们才能开始推导出这些特征，所以很明显我会想把这些头三天从数据集中排除出去。
再回头再看一下上面info()函数输出的信息，可以看到包含NaN值的数据特征非常的少，除了我提到的几个字段，基本就没有了。因为机器学习需要样本字段数据的完整性，因为如果我们因为降水量那个字段为空，就去掉样本，那么会造成大量的样本不可用，对于这种情况，我们可以给为空的降水量字段的样本填入一个值。根据经验和尽量减少由于填入的值对模型的影响，我决定给为空的降水量字段填入值0。

```python
# iterate over the precip columns
for precip_col in ['precipm_1', 'precipm_2', 'precipm_3']:  
    # create a boolean array of values representing nans
    missing_vals = pd.isnull(df[precip_col])
    df[precip_col][missing_vals] = 0
```

填入值后，我们就可以删掉字段值为空的样本了，只用调用dropna()函数。

```python
df = df.dropna()  
```

## 总结
&emsp;&emsp;这篇文章主要介绍了数据的收集、处理、清洗的流程，本篇文章处理完的处理，将用于下篇文章的模型训练。
&emsp;&emsp;对你来说，这篇文章可能很枯燥，没啥干货，但好的样本数据，才能训练处好的模型，因此，样本数据的收集和处理能力，直接影响你后面的机器学习的效果。

- [英文原文](http://stackabuse.com/using-machine-learning-to-predict-the-weather-part-1/)
- [使用机器学习预测天气(第二部分)](http://www.bugcode.cn/mlweatherpart02.html)
- [使用机器学习预测天气(第三部分)](http://www.bugcode.cn/mlweatherpart03.html)

