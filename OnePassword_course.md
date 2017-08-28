Title: onepassword使用教程
Meta: Onepass使用教程，Onepassword收费，Onepassword安全吗
Date: 2017-07-20
Tags: onepassword,使用,教程,收费,安全
Category: onepassword
Slug: onepassword_course
Author: 笨熊

## 先回答几个问题
- onepassword收费吗？
    - 收费，分为个人版和企业版
- onepassword安全吗？
    - 相对来说，我认为是最安全的，原因参照下面的分析
- 有免费的吗？
    - 有，比如dashline,keepass,lastpass等等，但这些工具有好多是只存在云端的，而且，ugly,丑，磕碜。
    
## 简述
- onepassword 是个保存账户 密码的工具软件，由于现在互联网上经常出现的密码泄露事件，导致很多人的常用密码基本都泄露了。所以现在出现出现了一个尴尬的事情，用常用密码的话，账户容易被盗窃，不用常用密码的话，记不住密码。于是，onepassword的使用场景出现了，记密码，自动填充，安全的密码管理，多端云同步。

## 各种奇葩的密码保存方式
- 云盘: 明文，容易被盗，云端。云盘相信大家都了解，近些年云盘账户泄露的事件也不少。而且使用起来很不方便
- excel: 有很多人用excel或者记事本来保存密码，密码少还行，多了，基本就是一团浆糊。而且需要跨平台的时候，就瞎了。
- 写纸上，这个我就不吐槽了

## onepassword优点
- 密码本地密文保存
- 支持通过dropbox和icloud同步，也支持导入导出。
- 结合浏览器插件，可以自动填充密码，发现登录密码并提示保存。
- 功能丰富，除了支持web密码保存填充，还支持信用卡，wifi，服务器等等，一共十几个分类
- 简洁美观，便捷方便
- 可以使用邮件、sms等多种方式加密分享

## onepassword收费模式
- onepassword收费分为个人版、家庭版、企业版
    - 个人版：2.99美元一个月
    - 家庭版：4.99美元一个月，可以5个人一起用，但是密码是存在云端的
    - 团队/企业版：3.99美元个成员，具备密码权限管理
- 有没有便宜的了
    - 如果是个人版并且是mac电脑，那么你可以去淘宝买个appstore家庭账号，也就几块钱，可以一直用。
    - 其他的就只能下载破解版了。

## onepassword安全性
- onepassword采用了多重加密，保证了数据的安全性，即使是云端同步，同步的文件也是加密过得，具体加密流程，大家可以参考我[这篇文章](http://www.bugcode.cn/onepassword-encrypt.html)
- 本地存储，所以担心云端安全的同学可以放心了。
- 手机端支持指纹识别，方便

## 下载onepassword
- 官网:https://1password.com/
- 各大商店

## 基本使用教程
### 第一次打开
- 第一次打开 1Password，软件会问你是不是老用户，不是老用户的话就需要创建一个新的数据库来存储你的密码。

- 主密码就像是那一间放着所有其它钥匙的屋子的钥匙，所以理论上它应该只存在于你的脑子里，所以我不太建议用 1Password 的提示去创建主密码，我们应该以自己的思维方式创建出属于自己的密码。
### 第一次进入主界面
- 第一次进入主界面，1Password 就会给你 3 个弹窗来介绍它的三个很友好的特色功能，这三个功能提供的默认选项非常贴心，你可以直接像你平常安装软件时的操作方式一样，一路继续地点过去
#### 弹窗一：让你设定 1Password 在何时选择自动锁定
- 因为 1Password 里面放着你所有的密码，所以它不能是实时开放着的，而应该是实时关闭，只有我们在用的时候再打开它。
![一](https://cdn.sspai.com/attachment/origin/2016/08/18/343224.png?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)
- 1Password 默认选择了电脑闲置 5 分钟后锁定，这是个相对来说比较聪明的功能，对于普通的生活和办公环境都较为合适。对于一些安全洁癖使用者，1Password 也提供了每次关闭 1Password 窗口时都会锁定的选项。
![23](https://cdn.sspai.com/attachment/origin/2016/08/18/343226.png?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)
#### 弹窗二：告诉你 1Password 提供了网站以及软件图标
- 人眼对图像的识别远高于对文字以及数字的识别，在列表里，文字标题应该作为图片的辅助。所以你应该选择使用丰富的图标。
![图标](https://cdn.sspai.com/attachment/origin/2016/08/18/343229.png?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

#### 弹窗三：介绍了 1Password mini
- 1Password mini 是 1Password 的一个核心功能，在 1Password for Mac 的使用过程中，你可能用的更多的是 1Password mini 而不是 1Password 软件本身。1Password mini 几乎可以做到主软件的所有功能，而且继承了所有的快捷键。如果你勾选了图中的始终保持 1Password mini 的运行然后点击了开始使用 1Password，那么你的菜单栏里就会出现一把小钥匙
![1pas](https://cdn.sspai.com/attachment/origin/2016/08/18/343230.png?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)
![mini](https://cdn.sspai.com/attachment/origin/2016/08/18/343231.png?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

### 创建第一个登录
- 主密码设置完成后就可以进入主界面了。我们第一个要点的是这个「+」号：
![创建](https://cdn.sspai.com/attachment/origin/2016/08/18/343232.png?imageView2/2/w/1120/q/90/interlace/1/ignore-error/1)

#### 教程部分摘自网络博客:https://sspai.com/post/35195


