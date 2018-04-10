Title: 使用机器学习预测天气(第二部分)
Meta: 这篇文章我们接着前一篇文章，使用Weather Underground网站获取到的数据，来继续探讨用机器学习的方法预测内布拉斯加州林肯市的天气。
Date: 2018-01-03
Tags: Python,机器学习,天气预测
Category: 机器学习
Slug: mlweatherpart02
Author: 笨熊

## 概述 
&emsp;&emsp;这篇文章我们接着前一篇文章，使用Weather Underground网站获取到的数据，来继续探讨用机器学习的方法预测内布拉斯加州林肯市的天气
&emsp;&emsp;上一篇文章我们已经探讨了如何收集、整理、清洗数据。这篇文章我们将使用上一篇文章处理好的数据，建立线性回归模型来预测天气。为了建立线性回归模型，我要用到python里非常重要的两个机器学习相关的库：Scikit-Learn和StatsModels 。
&emsp;&emsp;第三篇文章我们将使用google TensorFlow来建立神经网络模型，并把预测的结果和线性回归模型的结果做比较。
&emsp;&emsp;这篇文章中会有很多数学概念和名词，如果你理解起来比较费劲，建议你先google相关数据概念，有个基础的了解。

## 获取数据
&emsp;&emsp;在这个[Github仓库](https://github.com/amcquistan/WeatherPredictPythonML)，有个名为Underground API.ipynb的 Jupyter Notebook文件，这个文件记录了我们这篇文章和下一篇文章要用到的数据的获取、整理步骤。你还会发现一个名为end-part1_df.pkl的文件，如果你自己没有处理好数据，可以直接用这个文件，然后使用下面的代码来把数据转成Pandas DataFrame类型。

```python
import pickle  
with open('end-part1_df.pkl', 'rb') as fp:  
    df = pickle.load(fp)
```
如果运行上面的代码遇到错误：<code>No module named 'pandas.indexes'</code>，那么说明你用的pandas库的版本和我的(v0.18.1)不一致，为了避免这样的错误发生，我还准备了个csv文件，你可以从上述的Github仓库获取，然后使用下面的代码读入数据。

```python
import pandas as pd  
df = pd.read_csv('end-part2_df.csv').set_index('date')
```

## 线性回归算法
&emsp;&emsp;线性回归模型的目标是使用一系列线性相关的数据和数字技术来根据预测因素X(自变量)来预测可能的结果Y(因变量)，最终建立一个模型(数学公式)来预测给定任意的预测因素X来计算对应的结果Y。
&emsp;&emsp;线性回归的一般公式为：

```python
ŷ = β0 + β1 * x1 + β2 * x2 + ... + β(p-n) x(p-n) + Ε
```
关于公式的详细解释，查看[百度百科-线性回归模型](https://baike.baidu.com/item/%E7%BA%BF%E6%80%A7%E5%9B%9E%E5%BD%92%E6%96%B9%E7%A8%8B)

## 为模型选取特征数据
&emsp;&emsp;线性回归技术要求的关键假设是，因变量和每个自变量之间有一个线性关系。针对我们的数据，就是温度和其他变量，然后计算Pearson相关系数。Pearson相关系数（r）是输出范围为-1到1的值的等长阵列之间的线性相关量的量度。范围从0到1的相关值表示越来越强的正相关性。 这意味着当一个数据序列中的值与另一个序列中的值同时增加时，两个数据序列呈正相关，并且由于它们两者的上升幅度越来越相等，Pearson相关值将接近1。从0到-1的相关值被认为是相反或负相关的，因为当一个系列的值增加相反系列减小的相应值时，但是当系列之间的幅度变化相等（相反的方向）时， 相关值将接近-1。 紧密地跨越零的Pearson相关值暗示着具有弱的线性关系，随着值趋近于零而变弱。
&emsp;&emsp;关于相关系数的强度界定，统计学家和统计书籍中的观点各不相同。 但是，我发现一个普遍接受的关联强度分类集合如下：
![](http://7xpx6h.com1.z0.glb.clouddn.com/62278a2857afb2377b537d9865b7b023)
为了评估这个数据中的相关性，我将调用Pandas DataFrame对象的corr（）方法。通过corr()函数的调用，我可以选择我感兴趣的数据(meantempm)，然后再对返回的结果（Pandas Series object）调用sort_values()函数，这将输出从最负相关到最正相关的相关值。

```python
df.corr()[['meantempm']].sort_values('meantempm')  
```
![](http://7xpx6h.com1.z0.glb.clouddn.com/94ce38329daf67fd6122384ebc0c58eb)
![](http://7xpx6h.com1.z0.glb.clouddn.com/e2c51da1327b99a4afefe556d49c9085)
&emsp;&emsp;在选择包括在这个线性回归模型中的特征时，我想在包含具有中等或较低相关系数的变量时略微宽容一些。 所以我将删除相关值的绝对值小于0.6的特征。 此外，由于“mintempm”和“maxtempm”变量与预测变量“meantempm”同一天，我也将删除这些（即，如果我已经知道最高和最低温度，那么我已经有了我的答案）。有了这些信息，我现在可以创建一个新的DataFrame，它只包含我感兴趣的变量。

```python
predictors = ['meantempm_1',  'meantempm_2',  'meantempm_3',  
              'mintempm_1',   'mintempm_2',   'mintempm_3',
              'meandewptm_1', 'meandewptm_2', 'meandewptm_3',
              'maxdewptm_1',  'maxdewptm_2',  'maxdewptm_3',
              'mindewptm_1',  'mindewptm_2',  'mindewptm_3',
              'maxtempm_1',   'maxtempm_2',   'maxtempm_3']
df2 = df[['meantempm'] + predictors]  
```

## 可视化展示数据关系
因为大多数人，包括我自己在内，都更习惯于用视觉来评估和验证模式，所以我将绘制每个选定的预测因子来证明数据的线性关系。 要做到这一点，我将利用matplotlib的pyplot模块。
对于这个图，我希望将因变量“meantempm”作为沿所有18个预测变量图的一致y轴。 一种方法是创建一个的网格。 Pandas的确有一个有用的绘图函数叫做scatter_plot（），但是通常只有当大约只有5个变量时才使用它，因为它将绘图变成一个N×N的矩阵（在我们的例子中是18×18） 变得难以看到数据中的细节。 相反，我将创建一个六行三列的网格结构，以避免牺牲图表的清晰度。

```python
import matplotlib  
import matplotlib.pyplot as plt  
import numpy as np
# manually set the parameters of the figure to and appropriate size
plt.rcParams['figure.figsize'] = [16, 22]
# call subplots specifying the grid structure we desire and that 
# the y axes should be shared
fig, axes = plt.subplots(nrows=6, ncols=3, sharey=True)

# Since it would be nice to loop through the features in to build this plot
# let us rearrange our data into a 2D array of 6 rows and 3 columns
arr = np.array(predictors).reshape(6, 3)

# use enumerate to loop over the arr 2D array of rows and columns
# and create scatter plots of each meantempm vs each feature
for row, col_arr in enumerate(arr):  
    for col, feature in enumerate(col_arr):
        axes[row, col].scatter(df2[feature], df2['meantempm'])
        if col == 0:
            axes[row, col].set(xlabel=feature, ylabel='meantempm')
        else:
            axes[row, col].set(xlabel=feature)
plt.show()
```
![](http://7xpx6h.com1.z0.glb.clouddn.com/9af38676833ed91082b77789c820abd1)
![](http://7xpx6h.com1.z0.glb.clouddn.com/2df33c0dad13e5fac8faaecdf1488e86)
从上面的图中可以看出，所有其余的预测变量与响应变量（“meantempm”）显示出良好的线性关系。 此外，值得注意的是，这些关系都是均匀随机分布的。 我的意思是，在没有任何扇形或圆锥形状的情况下，数值的扩散似乎有相对相等的变化。 使用普通最小二乘算法的线性回归的另一个重要假设是沿点的均匀随机分布。

### 使用逐步回归建立一个健壮的模型
&emsp;&emsp;一个强大的线性回归模型必须选取有意义的、重要的统计指标的指标作为预测指标。 为了选择统计上显着的特征，我将使用Python statsmodels库。然而，在使用statsmodels库之前，我想先说明采取这种方法的一些理论意义和目的。
&emsp;&emsp;在分析项目中使用统计方法（如线性回归）的一个关键方面是建立和测试假设检验，以验证所研究数据假设的重要性。 有很多假设检验已经被开发来测试线性回归模型对各种假设的稳健性。 一个这样的假设检验是评估每个包含的预测变量的显着性。
&emsp;&emsp;βj参数意义的假设检验的正式定义如下：

- H0：βj= 0，零假设表明预测变量对结果变量的值没有影响
- Ha：βj≠0，可选假设是预测变量对结果变量的值有显着影响

通过使用概率测试来评估每个βj在选定阈值Α处超过简单随机机会的显着性的可能性，我们可以在选择更严格数据，以保证模型的鲁棒性。
&emsp;&emsp;然而在许多数据集中，数据间的相互影响会导致一些简单的假设检验不符合预期。 为了在线性回归模型中测试相互作用对任何一个变量的影响，通常应用被称为逐步回归的技术。 通过增加或者删除变量来评估每个变量的变化，对产生的模型的影响。在本文中，我将使用一种称为“后向消除”的技术，从一个包含我感兴趣数据的模型开始。

&emsp;&emsp;后向消除工作流程如下：

- 选择一个重要的阶段A，能够确定一个数据是否能通过建设检验。
- 把预测数据填入模型
- 评估βj系数的p值和p值最大的p值，如果p值>Α进行到第4步，如果不是，则得到最终模型
- 删除步骤3中确定的预测变量
- 再次安装模型，但这次没有删除变量，然后循环回到第3步

&emsp;&emsp;下面我们使用statsmodels按照上面的步骤，来构建我们的模型。

```python
# import the relevant module
import statsmodels.api as sm

# separate our my predictor variables (X) from my outcome variable y
X = df2[predictors]  
y = df2['meantempm']

# Add a constant to the predictor variable set to represent the Bo intercept
X = sm.add_constant(X)  
X.ix[:5, :5]  
```

![](http://7xpx6h.com1.z0.glb.clouddn.com/b3c793866d53199ea7113cc95361e998)

```python
# (1) select a significance value
alpha = 0.05

# (2) Fit the model
model = sm.OLS(y, X).fit()

# (3) evaluate the coefficients' p-values
model.summary()  
```

调用summary()函数输出的数据如下：

![](http://7xpx6h.com1.z0.glb.clouddn.com/8dbdbe291b4eae49f8a2dd5d0cc13148)
![](http://7xpx6h.com1.z0.glb.clouddn.com/b098830b6c82bc26ef38ece97605aa1e)
![](http://7xpx6h.com1.z0.glb.clouddn.com/78ca605307ffd57a3fd48a794c3fa978)

- 好的，我认识到，对summary（）的调用只是把大量的信息打印在屏幕上。在这篇文章中，我们只关注2-3个值：

- P>| T | - 这是我上面提到的p值，我将用它来评估假设检验。 这是我们要用来确定是否消除这个逐步反向消除技术的变量的价值。
R平方 - 一个衡量标准，我们的模型可以解释结果的整体变化的多少
- ADJ。 R平方 - 与R平方相同，但是，对于多元线性回归，根据包含的变量数来解释过度拟合水平，该值会受到惩罚。

这并不是说在这个输出中的其他价值是没有价值的，恰恰相反，它们涉及到线性回归的更深奥的特质，我们现在根本没有时间考虑到。对于他们的完整解释，我会推迟到高级回归教科书，[如Kutner的应用线性回归模型，第五版](http://stackabu.se/applied-linear-regression-models)。 以及statsmodels文件。

```python
# (3) cont. - Identify the predictor with the greatest p-value and assess if its > our selected alpha.
#             based off the table it is clear that meandewptm_3 has the greatest p-value and that it is
#             greater than our alpha of 0.05

# (4) - Use pandas drop function to remove this column from X
X = X.drop('meandewptm_3', axis=1)

# (5) Fit the model 
model = sm.OLS(y, X).fit()

model.summary()  
```
![](http://7xpx6h.com1.z0.glb.clouddn.com/a2af06781e03534de22e984b359e6068)
![](http://7xpx6h.com1.z0.glb.clouddn.com/3a9dd79d1ffd5d0aba0dc68b75a3d226)
![](http://7xpx6h.com1.z0.glb.clouddn.com/aa58f1f011716bf8e5c3bdedde6f187e)

关于您的阅读时间，为了保持文章的合理长度，我将省略构建每个新模型所需的剩余消除周期，评估p值并删除最不重要的值。 相反，我会跳到最后一个周期，并为您提供最终的模型。 毕竟，这里的主要目标是描述过程和背后的推理。下面你会发现我应用反向消除技术后收敛的最终模型的输出。 您可以从输出中看到，所有其余的预测变量的p值显着低于我们的0.05。 另外值得注意的是最终输出中的R平方值。 这里需要注意两点：（1）R平方和Adj。 R平方值是相等的，这表明我们的模型被过度拟合的风险最小，（2）0.894的值被解释为使得我们的最终模型解释了结果变量中观察到的变化的大约90％ ，“meantempm”。

```python
model = sm.OLS(y, X).fit()  
model.summary() 
```
![](http://7xpx6h.com1.z0.glb.clouddn.com/f9fc8768aa5945b27ee8563952d31326)
![](http://7xpx6h.com1.z0.glb.clouddn.com/7d5466f4140718be12c7fd667cac13a5)

## 使用SciKit-Learn的线性回归模块预测天气
&emsp;&emsp;现在我们已经完成了选择具有统计意义的预测指标（特征）的步骤，我们可以使用SciKit-Learn创建预测模型并测试其预测平均温度的能力。 SciKit-Learn是一个非常完善的机器学习库，在工业界和学术界广泛使用。关于SciKit-Learn的一件事非常令人印象深刻的是，它在许多数值技术和算法中保持了一个非常一致的“适应”，“预测”和“测试”API，使得使用它非常简单。除了这个一致的API设计，SciKit-Learn还提供了一些有用的工具来处理许多机器学习项目中常见的数据。

&emsp;&emsp;我们将通过使用SciKit-Learn从sklearn.model_selection模块中导入train_test_split（）函数来开始将我们的数据集分割成测试和训练集。我将把训练和测试数据集分成80％的训练和20％的测试，并且分配一个12的random_state，以确保你将得到和我一样的随机选择数据。这个random_state参数对结果的可重复性非常有用。

```python
from sklearn.model_selection import train_test_split  
# first remove the const column because unlike statsmodels, SciKit-Learn will add that in for us
X = X.drop('const', axis=1)

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=12) 
```
接下来的操作是使用训练数据集建立回归模型。 为此，我将从sklearn.linear_model模块导入并使用LinearRegression类。 正如前面提到的，scikit-learn分数通过通用的fit()和predict()这两个函数计算得到。

```python
from sklearn.linear_model import LinearRegression  
# instantiate the regressor class
regressor = LinearRegression()

# fit the build the model by fitting the regressor to the training data
regressor.fit(X_train, y_train)

# make a prediction set using the test set
prediction = regressor.predict(X_test)

# Evaluate the prediction accuracy of the model
from sklearn.metrics import mean_absolute_error, median_absolute_error  
print("The Explained Variance: %.2f" % regressor.score(X_test, y_test))  
print("The Mean Absolute Error: %.2f degrees celsius" % mean_absolute_error(y_test, prediction))  
print("The Median Absolute Error: %.2f degrees celsius" % median_absolute_error(y_test, prediction))

The Explained Variance: 0.90  
The Mean Absolute Error: 2.69 degrees celsius  
The Median Absolute Error: 2.17 degrees celsius  
```
正如你可以在上面几行代码中看到的那样，使用scikit-learn构建线性回归预测模型非常简单。 

为了获得关于模型有效性的解释性理解，我使用了回归模型的score（）函数来确定该模型能够解释在结果变量（平均温度）中观察到的约90％的方差。 此外，我使用sklearn.metrics模块的mean_absolute_error（）和median_absolute_error（）来确定平均预测值约为3摄氏度关闭，一半时间关闭约2摄氏度。

## 总结
在本文中，我演示了基于上一篇文章收集的数据如何使用线性回归机器学习算法来预测未来的平均天气温度。在本文中，我演示了如何使用线性回归机器学习算法来预测未来的平均天气温度，基于上一篇文章收集的数据。 我演示了如何使用statsmodels库来根据合理的统计方法选择具有统计显着性的预测指标。 然后，我利用这些信息来拟合基于Scikit-Learn的LinearRegression类的训练子集的预测模型。 然后使用这个拟合的模型，我可以根据测试子集的输入预测预期值，并评估预测的准确性。

## 相关文章
- [使用机器学习预测天气(第一部分)](http://www.bugcode.cn/mlweatherpart01.html)
- [使用机器学习预测天气(第三部分)](http://www.bugcode.cn/mlweatherpart03.html)
- [点击查看英文原文](https://translate.google.com.hk/#en/zh-CN/In%20this%20article%2C%20I%20demonstrated%20how%20to%20use%20the%20Linear%20Regression%20Machine%20Learning%20algorithm%20to%20predict%20future%20mean%20weather%20temperatures%20based%20off%20the%20data%20collected%20in%20the%20prior%20article.%20I%20demonstrated%20how%20to%20use%20the%20statsmodels%20library%20to%20select%20statistically%20significant%20predictors%20based%20off%20of%20sound%20statistical%20methods.%20I%20then%20utilized%20this%20information%20to%20fit%20a%20prediction%20model%20based%20off%20a%20training%20subset%20using%20Scikit-Learn's%20LinearRegression%20class.%20Using%20this%20fitted%20model%20I%20could%20then%20predict%20the%20expected%20values%20based%20off%20of%20the%20inputs%20from%20a%20testing%20subset%20and%20evaluate%20the%20accuracy%20of%20the%20prediction%2C%20which%20indicates%20a%20reasonable%20amount%20of%20accuracy.)

