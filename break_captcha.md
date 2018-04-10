Title: 15分钟破解网站验证码
Meta: 很多开发者都讨厌网站的验证码，特别是写网络爬虫的程序员.
Date: 2017-12-21
Tags: Python,验证码,破解,captcha,cnn,机器学习,tensorflow
Category: 机器学习
Slug: break_captcha
Author: 笨熊

## 概述
&emsp;&emsp;很多开发者都讨厌网站的验证码，特别是写网络爬虫的程序员，而网站之所以设置验证码，是为了防止机器人访问网站，造成不必要的损失。现在好了，随着机器学习技术的发展，机器识别验证码的问题比较好解决了。

## 样本采集工具
&emsp;&emsp;这里我们采用wordpress的Really Simple CAPTCHA生成验证码的插件，之所以选择这个插件，一个是它的安装量很大，二个是因为它是开源的，我们可以利用它批量的生成验证码图片。

## 目标估计
&emsp;&emsp;我们通过demo网站得知，Really Simple CAPTCHA生成的是包含4个数字或者字母的图片，通过阅读源码得知，这个插件还屏蔽了O和I这两个比较容易混淆的字母，也就是说，还剩下32个字符，看来可以完成。
&emsp;&emsp;目前花费了两分钟。

## 依赖
&emsp;&emsp;我们要用到以下的工具和库。

- python3
- opencv
- keras
- tensorflow

## 创建样本集
&emsp;&emsp;为了达到目的，我们首先要准备样本集，样本如下：
![样本](http://7xpx6h.com1.z0.glb.clouddn.com/d7b6ad66fc8497c2bbc9b9178721b8b9)

使用Really Simple CAPTCHA插件的源码，我们很方便的批量生成10000个验证码图片和对应的结果，待我们生成完成后，大概如下：
![样本集](http://7xpx6h.com1.z0.glb.clouddn.com/3472750b24fafd10795f1d76a00068ac
)

这地方大家可以根据自己的实际情况修改Really Simple CAPTCHA插件的源码，来生成自己想要的样本集。如果你觉着麻烦，也可以下载我生成好的。

&emsp;&emsp;目前为止，我们花了五分钟。

## 如何训练
&emsp;&emsp;我们现在有了样本集了，我们可以直接那图片和对应的结果直接进行神经网络的训练。
![训练](http://7xpx6h.com1.z0.glb.clouddn.com/91ebda315025fc8f34851d890bb9753b
)
只要我们的样本够多，最终也能达到我们想要的效果。

&emsp;&emsp;但我们也可以采用更好的训练方法，这个训练方法使用更少的样本数据，但是结果要比直接训练的方法好很多，我想你已经猜到了，这个方法就是把图片中的四个字符切割开，形成四个样本。这方法之所以可行，是因为所有的验证码图片都是4个字符的。
![split](http://7xpx6h.com1.z0.glb.clouddn.com/e6a80add1873b633114da416aeb75d5b
)

&emsp;&emsp;10000张图片，一张一张手动用PS去切割，肯定不现实，而且由于图片的横向排列并不是等间距的，字符间的距离大小不一致，手动切割肯定不可能了。
![split2](http://7xpx6h.com1.z0.glb.clouddn.com/1_yyfjNSCKt8IvY7JqANnOZg.gif)

&emsp;&emsp;其实我们只要画出一个矩形，保证矩形框里只有字符就可以，然后从图片中切出这样的一个矩形，就形成了一个单个字符的图片样本。幸运的是，这个操作opencv已经帮我们实现了，opencv有个函数叫做findContours()，可以按照同样色值的区域裁剪我们想要的矩形。
- 首先准备一个图片：
![](http://7xpx6h.com1.z0.glb.clouddn.com/6ac9c97fff05d66c603bc8e2b4a43622
)
- 转换图片为黑白色。这样有字符的地方为黑色，空白为白色，便于opencv裁剪。
![](http://7xpx6h.com1.z0.glb.clouddn.com/d81d5d535b57ca6128ef636df518839e
)
-接下来我们用opencv的findContours函数切割图片。
![](http://7xpx6h.com1.z0.glb.clouddn.com/5be4cd022968c4ac2074c6bcc6ed27d0
)

&emsp;&emsp;接下来，我们就把图片从左到右进行切割，并存储切割后的图片，以及图片对应的字符。但是实际操作的过程中，我发现一个问题，就是有时候两个字符靠的太近，导致opencv在切割的时候，把两个字符切割刀一个图片里了，比如：
![](http://7xpx6h.com1.z0.glb.clouddn.com/9c581bdf088a5cf8c00f9095e5100c25
)
切割完的效果是：
![](http://7xpx6h.com1.z0.glb.clouddn.com/87fc46e117afa15725cbe868bac1f1ab
)
如果不解决这个问题，我们的样本集就不准了，那训练出来的模型也就不可能正确了。我的解决方法是，首先设置一个字符宽最大的像素，如果超过这个像素，则认为一个图片中包含了两个字符，然后我们选择把这个图片对半切割，分成两个字符。例如：
![](http://7xpx6h.com1.z0.glb.clouddn.com/11485e04bce19c0324c66a64baa24a4b
)
好，我们现在得到了一个验证码图片对应的4个字符的图片，现在我们把所有的样本图片都切割好，然后，把相同的字符对应的图片放到一个文件夹，这么做的目的是尽量多的找出同一个字符的多种样式。结果如下：
![](http://7xpx6h.com1.z0.glb.clouddn.com/a77740cbaf3d2fd3d8a8f7ab7baf7c08
)
&emsp;&emsp;到目前为止，我花了10分钟。

## 训练模型
&emsp;&emsp;因为我们只是识别图片对应的数字或者字母，所以我们不需要特别复杂的神经网络算法。识别字符比识别小猫小狗的简单多了。
&emsp;&emsp;我这地方使用卷积神经网络，two convolutional layers and two fully-connected layers。
![](http://7xpx6h.com1.z0.glb.clouddn.com/7ff8e602cb0fe1fc73673a4c568275bd
)
这地方对卷积神经网络算法就不做详细介绍，感兴趣的同学，可以google学习一下。
&emsp;&emsp;训练完成后，我们需要测试一下。15分钟花完。

## 总结
整个过程看起来很简单：
- 从使用我们上述提到的插件的wordpress网站上下载验证码图片
- 把图片切割成包含单个字符的小图片
- 使用神经网络算法训练模型
- 预测新的验证码图片对应的字符

下面是我的测试：
![](http://7xpx6h.com1.z0.glb.clouddn.com/1_7RE-Ql6jaDu1jCfi0OgtTw.gif)

## 代码
你可以[从这](https://s3-us-west-2.amazonaws.com/mlif-example-code/solving_captchas_code_examples.zip)得到完整的代码和示例图片，你可以参照README来运行相关的程序。

## 转载自我的博客[捕蛇者说]()
- [英文原文](https://medium.com/@ageitgey/how-to-break-a-captcha-system-in-15-minutes-with-machine-learning-dbebb035a710)
- [下载代码](https://s3-us-west-2.amazonaws.com/mlif-example-code/solving_captchas_code_examples.zip)



