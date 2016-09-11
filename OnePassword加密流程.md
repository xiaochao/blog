Title: OnePassword密码加密方法
Meta: OnePassword密码加密方法
Date: 2016-09-11
Tags: OnePassword,密码,加密流程,加密方法,加密
Category: 密码加密
Slug: onepassword-encrypt
Author: 笨熊


#### 简介
- OnePassword是目前比较流行的密码保管软件，能够保存密码、自动同步、自动填充。作为一个密码保管软件，如果没有比较好的加密方法，那么很容易造成密码的泄露，给用户带来巨大的损害。OnePassword之所以能这么流行，也跟它的安全性有很大的关系。
- 下面，是我对OnePassword的加密方法的理解，仅代表个人观点。并且以下的加密方法适用于onepassword4.

#### 加密方法
- OnePassword加密流程大致如下：
	- 获取所有数据
	- 把数据分片
	- 对分片的数据进行加密
	- 对分片里的不同数据，采用不同的秘钥加密
- OnePassword主要采用AES-256做数据加密，使用HMAC算法做数据签名验证。而不同的数据，采用不同的秘钥，同样的算法进行加密。
- OnePassword加密和签名使用的秘钥，是有软件在第一次使用时，随机生成的，而这个这些key，使用的是用户设置的主密码生成的秘钥进行加密存储和同步传输。这样做的好处是，加密秘钥很难被破解，并且用户更换主密码，不会影响已经加密的数据。

#### 密钥分类
- Master Password 
	- 用户设置的主密码- Derived encryption key.
	- 加密数据使用的key，需要同步到其他设备，同步过程用，这些key需要加密，而加密使用的key，就是这个Derived encryption key- Derived MAC key.
	- 校验Derived encryption key加密数据- Master encryption key.
	- 加密主要数据的key，如密码等- Master MAC key.
	- 校验Master encryption key加密的数据- Overview encryption key.
	- 加密次要的数据，如url，名称等- Overview MAC key.
	- 校验Overview encryption key加密的数据- Item encryption key (item specific).	- 加密备注类的信息- Item MAC key (item specific).
	- 校验Item encryption key加密的数据
	
所有数据的加密方法是AES-256,校验的方法是HMAC算法，并且Item encryption key和Item MAC key是指定的，不是生成的。


#### 秘钥生成
- 如图
![密钥生成流程图](http://7xpx6h.com1.z0.glb.clouddn.com/1Password%E5%AF%86%E9%92%A5%E7%94%9F%E6%88%90%E6%B5%81%E7%A8%8B.png)