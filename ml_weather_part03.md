Title: 使用机器学习预测天气(第三部分神经网络)
Meta: 这篇文章我们接着第一篇文章，使用Weather Underground网站获取到的数据，使用神经网络算法，预测未来的天气情况，并和上一篇文章使用的线性回归算法做比较。
Date: 2018-01-10
Tags: Python,机器学习,天气预测
Category: 机器学习
Slug: mlweatherpart03
Author: 笨熊

### 概述
&emsp;&emsp;这是使用机器学习预测平均气温系列文章的最后一篇文章了，作为最后一篇文章，我将使用google的开源机器学习框架tensorflow来构建一个神经网络回归器。关于tensorflow的介绍、安装、入门，请自己google，这里就不做讲述。

&emsp;&emsp;这篇文章我主要讲解一下几点：

- 了解人工神经网络理论
- tensorflow高级API:Estimators
- 构建DNN模型预测天气

### 人工神经网络基础理论
&emsp;&emsp;上一篇文章主要讲解了如何构建线性回归模型(这是最基础的机器学习算法)来预测内布拉斯加州林肯市每天的平均气温。线性回归模型非常有效，并且可以被用于数值化(比如分类)、预测(比如预测天气)。线性回归算法也比较与局限性，它要求数据之间具有线性关系。
&emsp;&emsp;针对于非线性关系的场景，数据挖掘和机器学习有数不清的算法来处理。近些年最火的要数神经网络算法了，它可以处理机器学习领域的好多问题。神经网络算法具备线性和非线性学习算法的能力。
&emsp;&emsp;神经网络受到大脑中的生物神经元的启发，它们在复杂的交互网络中工作，根据已经收集的信息的历史来传输，收集和学习信息。我们感兴趣的计算神经网络类似于大脑的神经元，因为它们是接收输入信号（数字量）的神经元（节点）的集合，处理输入并将处理后的信号发送给其他下游代理 网络。 信号作为通过神经网络的数字量的处理是一个非常强大的特征，不限于线性关系。
&emsp;&emsp;在这个系列中，我一直关注一种称为监督学习的特定类型的机器学习，也就说说训练的数据结果是已知的，根据历史已知的输入和输出，预测未来的输入对应的输出。 此外，预测的类型是数值的真实值，这意味着我们使用的是回归预测算法。
&emsp;&emsp;从图形上看，类似于本文中描述的神经网络如图:
![](http://7xpx6h.com1.z0.glb.clouddn.com/bee7b10a09ce9b895954dc57b1d5833e)
上面描述的神经网络在最左边包含一个输入层，即图中的x1和x2，这两个特征是神经网络输入值。这两个特征被输入到神经网络中，通过被称为隐藏层的两层神经元进行处理和传输。这个描述显示了两个隐藏层，每层包含三个神经元（节点）。 该信号然后离开神经网络，并作为单个数值预测值汇总在输出层。
&emsp;&emsp;让我花一点时间来解释箭头背后的含义，箭头表示数据在层间从一个节点到另一个节点的传输处理。 每个箭头代表一个数值的数学变换，从箭头的底部开始，然后乘以特定于该路径特定的权重。 一个图层中的每个节点将以这种方式得到一个值。 然后汇总所有在节点收敛的值。 这个就是我之前提到的神经网络的线性操作。
![](http://7xpx6h.com1.z0.glb.clouddn.com/c7530f87b79f58821d1149fa6c58cc42)
&emsp;&emsp;在每个节点上进行求和之后，将一个特殊的非线性函数应用到总和上，这在上面的图像中被描述为Fn（...）。 这种将非线性特征引入神经网络的特殊功能称为激活功能。 激活函数所带来的这种非线性特性赋予了多层神经网络以其功能。 如果不是将非线性加入到过程中，则所有层都将有效地代数地组合成一个常数运算，其中包括将输入乘以某个平坦系数值（即线性模型）。
好吧，这一切都很好，但是这怎么转化为学习算法呢？ 那么最直接的答案就是评估正在进行的预测，即模型“y”的输出，到实际预期值（目标），并对权重进行一系列调整，以改善整体 预测准确性。
&emsp;&emsp;在回归机器学习算法的世界中，通过使用成本（又名“损失”或“客观”）函数（即平方误差之和（SSE））评估准确性。 请注意，我把这个声明推广到整个机器学习的连续体，而不仅仅是神经网络。 在前面的文章中，普通最小二乘算法完成了这一工作，它发现了使误差平方和（即最小二乘）最小化的系数组合。
&emsp;&emsp;我们的神经网络回归器会做同样的事情。 它将迭代训练数据提取特征值，计算成本函数（使用SSE），并以最小化成本函数的方式调整权重。 通过算法迭代推送特征的过程和评估如何根据成本函数来调整权重。
&emsp;&emsp;模型优化算法在构建鲁棒神经网络中非常重要。 例如通过网络体系结构（即宽度和深度）馈送，然后根据成本函数进行评估，调整权重。 当优化器函数确定权重调整不会导致成本函数计算代价的变化，该模型被认为是“学习”。

### TensorFlow Estimator API
&emsp;&emsp;tensorflow由好几个部分组成，其中最常用的是Core API,它为用户提供了一套低级别的API来定义和训练使用符号操作的任何机器学习算法。这也是TensorFlow的核心功能，虽然Core API能应对大多数的应用场景，但我更关注Estimator API。
&emsp;&emsp;TensorFlow团队开发了Estimator API，使日常开发人员可以更方便地使用该库。这个API提供了训练模型、评估模型、以及和Sci-Kit库相似的对未知数据的预测接口，这是通过实现各种算法的通用接口来实现的。另外，构建在高级API中的是机器学习最佳实践，抽象和可伸缩性的负载。
&emsp;&emsp;所有这些机器学习的优点使得基础Estimator类中实现的一套工具以及多个预先封装的模型类型，降低了使用TensorFlow的入门门槛，因此可以应用于日常问题。通过抽象出诸如编写训练循环或处理会话之类的问题，开发人员能够专注于更重要的事情，如快速尝试多个模型和模型架构，以找到最适合他们需要的模型。
&emsp;&emsp;在这篇文章中，我将介绍如何使用非常强大的深度神经网络估计器之一DNN Regressor。

### 建立一个DNNRegressor来预测天气
&emsp;&emsp;我们先导入一些我们需要用到的库。

```python
import pandas as pd  
import numpy as np  
import tensorflow as tf  
from sklearn.metrics import 
explained_variance_score, \  
    mean_absolute_error, \
    median_absolute_error
from sklearn.model_selection import train_test_split  
```
我们来处理一下数据，所有的数据我都放在了Github上，大家可以去查阅clone。

```python
# read in the csv data into a pandas data frame and set the date as the index
df = pd.read_csv('end-part2_df.csv').set_index('date')

# execute the describe() function and transpose the output so that it doesn't overflow the width of the screen
df.describe().T  
```
![](http://7xpx6h.com1.z0.glb.clouddn.com/9e4cee2484350996f458fd1a1111aad9)
![](http://7xpx6h.com1.z0.glb.clouddn.com/d1e3ba193f964f2abbe7e89d7e9be018)

```python
# execute the info() function
df.info()
<class 'pandas.core.frame.DataFrame'>  
Index: 997 entries, 2015-01-04 to 2017-09-27  
Data columns (total 39 columns):  
meantempm          997 non-null int64  
maxtempm           997 non-null int64  
mintempm           997 non-null int64  
meantempm_1        997 non-null float64  
meantempm_2        997 non-null float64  
meantempm_3        997 non-null float64  
meandewptm_1       997 non-null float64  
meandewptm_2       997 non-null float64  
meandewptm_3       997 non-null float64  
meanpressurem_1    997 non-null float64  
meanpressurem_2    997 non-null float64  
meanpressurem_3    997 non-null float64  
maxhumidity_1      997 non-null float64  
maxhumidity_2      997 non-null float64  
maxhumidity_3      997 non-null float64  
minhumidity_1      997 non-null float64  
minhumidity_2      997 non-null float64  
minhumidity_3      997 non-null float64  
maxtempm_1         997 non-null float64  
maxtempm_2         997 non-null float64  
maxtempm_3         997 non-null float64  
mintempm_1         997 non-null float64  
mintempm_2         997 non-null float64  
mintempm_3         997 non-null float64  
maxdewptm_1        997 non-null float64  
maxdewptm_2        997 non-null float64  
maxdewptm_3        997 non-null float64  
mindewptm_1        997 non-null float64  
mindewptm_2        997 non-null float64  
mindewptm_3        997 non-null float64  
maxpressurem_1     997 non-null float64  
maxpressurem_2     997 non-null float64  
maxpressurem_3     997 non-null float64  
minpressurem_1     997 non-null float64  
minpressurem_2     997 non-null float64  
minpressurem_3     997 non-null float64  
precipm_1          997 non-null float64  
precipm_2          997 non-null float64  
precipm_3          997 non-null float64  
dtypes: float64(36), int64(3)  
memory usage: 311.6+ KB  
```
&emsp;&emsp;请注意，我们刚刚记录下了1000个气象数据记录，并且所有的特征都是数字性质的。 另外，由于我们在第一篇文章中的努力工作，所有记录都是完整的，因为它们不缺少任何值（没有非空值）。
&emsp;&emsp;现在我将删除“mintempm”和“maxtempm”这两列，因为它们对帮助我们预测平均温度毫无意义。 我们正在试图预测未来，所以我们显然不能掌握有关未来的数据。 我还将从目标（y）中分离出特征（X）。

```python
# First drop the maxtempm and mintempm from the dataframe
df = df.drop(['mintempm', 'maxtempm'], axis=1)

# X will be a pandas dataframe of all columns except meantempm
X = df[[col for col in df.columns if col != 'meantempm']]

# y will be a pandas series of the meantempm
y = df['meantempm']  
```
&emsp;&emsp;和所有监督机器学习应用程序一样，我将把我的数据集分成训练集和测试集。 但是，为了更好地解释训练这个神经网络的迭代过程，我将使用一个额外的数据集，我将其称为“验证集合”。 对于训练集，我将利用80％的数据，对于测试和验证集，它们将分别为剩余数据的10％。为了分解这些数据，我将再次使用Scikit Learn 库的train_test_split（）函数。

```python
# split data into training set and a temporary set using sklearn.model_selection.traing_test_split
X_train, X_tmp, y_train, y_tmp = train_test_split(X, y, test_size=0.2, random_state=23)  
# take the remaining 20% of data in X_tmp, y_tmp and split them evenly
X_test, X_val, y_test, y_val = train_test_split(X_tmp, y_tmp, test_size=0.5, random_state=23)

X_train.shape, X_test.shape, X_val.shape  
print("Training instances   {}, Training features   {}".format(X_train.shape[0], X_train.shape[1]))  
print("Validation instances {}, Validation features {}".format(X_val.shape[0], X_val.shape[1]))  
print("Testing instances    {}, Testing features    {}".format(X_test.shape[0], X_test.shape[1]))  
Training instances   797, Training features   36  
Validation instances 100, Validation features 36  
Testing instances    100, Testing features    36  
```
构建神经网络模型时要采取的第一步是实例化tf.estimator.DNNRegressor（）类。类的构造函数有多个参数，但我将重点关注以下参数：

- feature_columns：一种类似列表的结构，包含要输入到模型中的要素的名称和数据类型的定义
- hidden_​​units：一个类似列表的结构，包含神经网络数量宽度和深度的定义
- optimizer：tf.Optimizer子类的一个实例，在训练期间优化模型的权重;它的默认值是AdaGrad优化器。
- activation_fn：激活功能，用于在每一层向网络引入非线性;默认是ReLU
- model_dir：要创建的目录，其中包含模型的元数据和其他检查点保存
我将首先定义一个数字特征列的列表。要做到这一点，我使用tf.feature_column.numeric_column（）函数返回一个FeatureColumn实例。

```python
feature_cols = [tf.feature_column.numeric_column(col) for col in X.columns] 
```
&emsp;&emsp;使用定义的特性列，我现在可以实例化DNNRegressor类并将其存储在回归变量中。 我指定我想要一个有两层深度的神经网络，其中两层的宽度都是50个节点。 我还指出，我希望我的模型数据存储在一个名为tf_wx_model的目录中。

```python
regressor = tf.estimator.DNNRegressor(feature_columns=feature_cols,  
                                      hidden_units=[50, 50],
                                      model_dir='tf_wx_model')
INFO:tensorflow:Using default config.  
INFO:tensorflow:Using config: {'_tf_random_seed': 1, '_save_checkpoints_steps': None, '_save_checkpoints_secs': 600, '_model_dir': 'tf_wx_model', '_log_step_count_steps': 100, '_keep_checkpoint_every_n_hours': 10000, '_save_summary_steps': 100, '_keep_checkpoint_max': 5, '_session_config': None}  
```
接下来我想要做的是定义一个可重用的函数，这个函数通常被称为“输入函数”，我将调用wx_input_fn（）。 这个函数将被用来在训练和测试阶段将数据输入到我的神经网络中。有许多不同的方法来建立输入函数，但我将描述如何定义和使用一个基于tf.estimator.inputs.pandas_input_fn（），因为我的数据是在一个pandas数据结构。

```python
def wx_input_fn(X, y=None, num_epochs=None, shuffle=True, batch_size=400):  
    return tf.estimator.inputs.pandas_input_fn(x=X,
                                               y=y,
                                               num_epochs=num_epochs,
                                               shuffle=shuffle,
                                               batch_size=batch_size)
```
&emsp;&emsp;请注意，这个wx_input_fn（）函数接受一个必选参数和四个可选参数，然后将这些参数交给TensorFlow输入函数，专门用于返回的pandas数据。 这是TensorFlow API一个非常强大的功能。函数的参数定义如下：

- X：输入要输入到三种DNNRegressor接口方法中的一种（训练，评估和预测）
- y：X的目标值，这是可选的，不会被提供给预测调用
- num_epochs：可选参数。 当算法在整个数据集上执行一次时，就会出现一个新纪元。
- shuffle：可选参数，指定每次执行算法时是否随机选择数据集的批处理（子集）
- batch_size：每次执行算法时要包含的样本数

&emsp;&emsp;通过定义我们的输入函数，我们现在可以训练我们基于训练数据集上的神经网络。 对于熟悉TensorFlow高级API的读者，您可能会注意到我对自己的模型的培训方式有点不合常规。至少从TensorFlow网站上的当前教程和网络上的其他教程的角度来看。通常情况下，您将看到如下所示的内容。

```python
regressor.train(input_fn=input_fn(training_data, num_epochs=None, shuffle=True), steps=some_large_number)

.....
lots of log info  
....
```
然后，作者将直接展示evaluate（）函数，并且几乎没有提示它在做什么或为什么存在这一行代码。

```python
regressor.evaluate(input_fn=input_fn(eval_data, num_epochs=1, shuffle=False), steps=1)

.....
less log info  
....
```
在此之后，假设所有的训练模型都是完美的，他们会直接跳到执行predict()函数。

```python
predictions = regressor.predict(input_fn=input_fn(pred_data, num_epochs=1, shuffle=False), steps=1)  
```

&emsp;&emsp;我希望能够提供一个合理的解释，说明如何训练和评估这个神经网络，以便将这个模型拟合或过度拟合到训练数据上的风险降到最低。因此，我们不再拖延，让我定义一个简单的训练循环，对训练数据进行训练，定期对评估数据进行评估。

```python
evaluations = []  
STEPS = 400  
for i in range(100):  
    regressor.train(input_fn=wx_input_fn(X_train, y=y_train), steps=STEPS)
    evaluations.append(regressor.evaluate(input_fn=wx_input_fn(X_val,
                                                               y_val,
                                                               num_epochs=1,
                                                               shuffle=False)))
INFO:tensorflow:Create CheckpointSaverHook.  
INFO:tensorflow:Saving checkpoints for 1 into tf_wx_model/model.ckpt.  
INFO:tensorflow:step = 1, loss = 1.11335e+07  
INFO:tensorflow:global_step/sec: 75.7886  
INFO:tensorflow:step = 101, loss = 36981.3 (1.321 sec)  
INFO:tensorflow:global_step/sec: 85.0322  
... A WHOLE LOT OF LOG OUTPUT ...
INFO:tensorflow:step = 39901, loss = 5205.02 (1.233 sec)  
INFO:tensorflow:Saving checkpoints for 40000 into tf_wx_model/model.ckpt.  
INFO:tensorflow:Loss for final step: 4557.79.  
INFO:tensorflow:Starting evaluation at 2017-12-05-13:48:43  
INFO:tensorflow:Restoring parameters from tf_wx_model/model.ckpt-40000  
INFO:tensorflow:Evaluation [1/1]  
INFO:tensorflow:Finished evaluation at 2017-12-05-13:48:43  
INFO:tensorflow:Saving dict for global step 40000: average_loss = 10.2416, global_step = 40000, loss = 1024.16  
INFO:tensorflow:Starting evaluation at 2017-12-05-13:48:43  
INFO:tensorflow:Restoring parameters from tf_wx_model/model.ckpt-40000  
INFO:tensorflow:Finished evaluation at 2017-12-05-13:48:43  
INFO:tensorflow:Saving dict for global step 40000: average_loss = 10.2416, global_step = 40000, loss = 1024.16  
```
&emsp;&emsp;上面的循环迭代了100次。 在循环体中，我调用了回归器对象的train（）方法，并将其传递给了我的可重用的wx_input_fn（），后者又通过了我的训练功能集和目标。 我有意地将默认参数num_epochs等于None，基本上这样说：“我不在乎你通过训练集多少次，只是继续训练算法对每个默认batch_size 400”（大约一半的训练 组）。 我还将shuffle参数设置为默认值True，以便在训练时随机选择数据以避免数据中的任何顺序关系。 train（）方法的最后一个参数是我设置为400的步骤，这意味着训练集每个循环将被批处理400次。
&emsp;&emsp;这给了我一个很好的时间以更具体的数字来解释一个epoch的意义。 回想一下，当一个训练集的所有记录都通过神经网络训练一次时，就会出现一个epoch。 所以，如果我们的训练集中有大约800（准确的说是797）个记录，并且每个批次选择400个，那么每两个批次我们就完成了一个时间。 因此，如果我们遍历整个训练集100个迭代400个步骤，每个批次大小为400（每个批次的一个半个时间），我们得到：

```python
(100 x 400 / 2) = 20,000 epochs
```
&emsp;&emsp;现在你可能想知道为什么我为循环的每次迭代执行和evaluate()方法，并在列表中捕获它的输出。 首先让我解释一下每次train()方法被触发时会发生什么。它随机选择一批训练记录，并通过网络推送，直到做出预测，并为每条记录计算损失函数。 然后根据计算出的损失根据优化器的逻辑调整权重，这对于减少下一次迭代的整体损失的方向做了很好的调整。 一般而言，只要学习速率足够小，这些损失值随着时间的推移而逐渐下降。
&emsp;&emsp;然而，经过一定数量的这些学习迭代之后，权重开始不仅受到数据整体趋势的影响，而且还受到非实际的噪声在所有实际数据中的继承。 在这一点上，网络受到训练数据特性的过度影响，并且变得无法推广关于总体数据的预测。这与我之前提到的高级TensorFlow API许多其他教程不足之处有关。 在训练期间周期性地打破这一点非常重要，并评估模型如何推广到评估或验证数据集。 通过查看第一个循环迭代的评估输出，让我们花些时间看看evaluate()函数返回的结果。

```python
evaluations[0]  
{'average_loss': 31.116383, 'global_step': 400, 'loss': 3111.6382}
```
正如你所看到的，它输出的是平均损失（均方误差）和训练中的步骤的总损失（平方误差和），这一步是第400步。 在训练有素的网络中，你通常会看到一种趋势，即训练和评估损失或多或少地平行下降。 然而，在某个时间点的过度配置模型中，实际上在过拟合开始出现的地方，验证训练集将不再看到其evaluate()方法的输出降低。 这是你想停止进一步训练模型的地方，最好是在变化发生之前。
&emsp;&emsp;现在我们对每个迭代都有一个评估集合，让我们将它们作为训练步骤的函数来绘制，以确保我们没有过度训练我们的模型。 为此，我将使用matplotlib的pyplot模块中的一个简单的散点图。

```python
import matplotlib.pyplot as plt  
%matplotlib inline

# manually set the parameters of the figure to and appropriate size
plt.rcParams['figure.figsize'] = [14, 10]

loss_values = [ev['loss'] for ev in evaluations]  
training_steps = [ev['global_step'] for ev in evaluations]

plt.scatter(x=training_steps, y=loss_values)  
plt.xlabel('Training steps (Epochs = steps / 2)')  
plt.ylabel('Loss (SSE)')  
plt.show()  
```
![](http://7xpx6h.com1.z0.glb.clouddn.com/e99e012a51a252b5375666d776bc7119)

&emsp;&emsp;从上面的图表看来，在所有这些迭代之后，我并没有过度配置模型，因为评估损失从来没有呈现出朝着增加价值的方向的显着变化。现在，我可以安全地继续根据我的剩余测试数据集进行预测，并评估模型如何预测平均天气温度。
&emsp;&emsp;与我已经演示的其他两种回归方法类似，predict()方法需要input_fn，我将使用可重用的wx_input_fn()传递input_fn，将测试数据集交给它，将num_epochs指定为None，shuffle为False，因此它将依次送入所有的数据进行测试。
&emsp;&emsp;接下来，我做一些从predict()方法返回的dicts迭代的格式，以便我有一个numpy的预测数组。然后，我使用sklearn方法explain_variance_score()，mean_absolute_error()和median_absolute_error()来预测数组，以测量预测与已知目标y_test的关系。

```python
pred = regressor.predict(input_fn=wx_input_fn(X_test,  
                                              num_epochs=1,
                                              shuffle=False))
predictions = np.array([p['predictions'][0] for p in pred])

print("The Explained Variance: %.2f" % explained_variance_score(  
                                            y_test, predictions))  
print("The Mean Absolute Error: %.2f degrees Celcius" % mean_absolute_error(  
                                            y_test, predictions))  
print("The Median Absolute Error: %.2f degrees Celcius" % median_absolute_error(  
                                            y_test, predictions))
INFO:tensorflow:Restoring parameters from tf_wx_model/model.ckpt-40000  
The Explained Variance: 0.88  
The Mean Absolute Error: 3.11 degrees Celcius  
The Median Absolute Error: 2.51 degrees Celcius  
```

我已经使用了与上一篇文章有关的线性回归技术相同的指标，以便我们不仅可以评估这个模型，还可以对它们进行比较。 正如你所看到的，这两个模型的表现相当类似，更简单的线性回归模型略好一些。然而，你可以通过修改学习速率，宽度和深度等参数来优化机器学习模型。

### 总结
&emsp;&emsp;本文演示了如何使用TensorFlow高级API Estimator子类DNNRegressor。并且，我也描述了神经网络理论，他们是如何被训练的，以及在过程中认识到过度拟合模型的危险性的重要性。
&emsp;&emsp;为了演示这个建立神经网络的过程，我建立了一个模型，能够根据本系列第一篇文章收集的数字特征预测第二天的平均温度。写这些文章的目的不是为了建立一个非常好的模型预测天气，我的目标是：

- 演示从数据收集，数据处理，探索性数据分析，模型选择，模型构建和模型评估中进行分析（机器学习，数据科学，无论...）项目的一般过程。
- 演示如何使用两个流行的Python库StatsModels和Scikit Learn来选择不违反线性回归技术关键假设的有意义的功能。
- 演示如何使用高级别的TensorFlow API，并直观地了解所有这些抽象层下正在发生的事情。
- 讨论与过度拟合模型相关的问题。
- 解释试验多个模型类型以最好地解决问题。

### 相关文章

[使用机器学习预测天气第二部分](http://www.bugcode.cn/mlweatherpart02.html)
[使用机器学习预测天气第一部分](http://www.bugcode.cn/mlweatherpart01.html)
[英文原文](http://stackabuse.com/using-machine-learning-to-predict-the-weather-part-3/)



