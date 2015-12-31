Title: Django session源码阅读
Date: 2014-12-20
Tags: Python, Django, Session
Category: python库
Slug: 
Author: 笨熊
## 背景
* 最近在做djnago开发时，遇到一个session问题，过程如下，第一个POST请求时，把数据存放在session，在第二次POST时，从session中读取数据，完成用户注册。在实际的环境中，发现有时第二次获取到的数据为空。初步的猜想是第一次和第二次请求间隔太短，数据还没有存进mysql，到时读取失败，带着这样的疑问，阅读了django session的源码。
* django session源码非常简单，没有复杂的数据结构和算法，读起来没费什么劲。

## session总体结构
session目录结构如下图

![image](http://sfault-image.b0.upaiyun.com/635/172/635172686-54918df1d0db6_articlex)

* backends这个目录中定义了session的数据结构和几种存储模式
	1. base.py这个定义了session的类dict的数据结构
	2. cache.py定义了session的缓存存储，缓存从django.core.cache中获取
	3. cached_db.py定义了session的缓存+数据库存储方式
	4. db.py定义了session的数据库存储方式
	5. file.py定义了session的文件存储
	6. signed_cookies.py定义了用于签名的session存储方式
* middleware.py实现了session中间件的处理过程
* models.py定义了session数据库结构
* management中是一个工具脚本，作用是清理session

## session处理流程

### 中间件流程

* 处理请求前
	1. 从配置文件SESSION_ENGINE中导入session存储方式
	2. 从用户cookie中读取session_key
	3. 把session存储方式赋值给request.session
* 返回请求前
	1. 读取是否修改session的标志
	2. 读取session的过期时间
	3. 判断如果session被标志位修改或者配置文件中指定了SESSION_SAVE_EVERY_REQUEST，在状态码不为500时，存储session，设置用户cookie
	
### session数据库结构
models.py定义了基于django.db.models session的数据库表结构，以及存储的方法，如果session的value为空的话，则删除该session，否则把session插入到数据库。
session的表名为django_session,三个字段分别是sessin的key,value以及过期时间

### session处理流程
1. session初始化，根据session key获取session内容
	* 从cookie中读取session_key，初始化session,并赋值给request.session
	* 根据cookie中的内容初始化session_key，session的两个标志位access和modify为false
	* 初始化session为python dcit，如果已经初始化了，则直接返回该session，如果session_key为None，或者设置no_load标志为True，则直接返回个空dict，如果session_key不为None并且设置no_load标志为False，则load数据
	* 如果本地数据库已经存在该session，则导入到内存，如果不存在，则创建session
		- 第一步，生成sessin_key,生成算法如下：
		
				def _get_new_session_key(self):	
        		    while True:
            		    session_key = get_random_string(32, VALID_KEY_CHARS)
            			if not self.exists(session_key):
                			break
        			return session_key```
        	这边有个问题，但用户量上百万时，这个生成算法随机重复的概率还是挺高的，所以这地方可以根据自己的需求做适当的优化
        - 第二步：保存session到数据库
        - 第三步：初始化session为空
2. session的操作
	* session为类dict结构，可以像操作dict一样操作session
3. session的存储，修改session内容
	* 在返回用户请求时，根据session.access和session.modify标志位设置用户cookie和存储session，如果sesssion.access则设置用户cookie
	* 如果session.modify为True或者设置setting.SESSION_SAVE_EVERY_REQUEST为True，则先获取过期时间，如果http状态不为500，则保存session到数据库，并设置用户cookie
4. 上述可见，session的过期是靠设置cookie的过期时间来实现的。当cookie过期后，用户获取的session_key就为空，就会为用户重新初始化生成session。那这样就会出现一个问题，旧的session_key就会越来越多，随机生成新session_key就越难，因此django提供了一个清理的工具在management目录下，官方的说法如下

		Can be run as a cronjob or directly to clean out expired sessions (only with the database backend at the moment).
		
### sesssion加密过程
session加密只是对session value值进行加密，加密的步骤为

1. 序列化session value
2. 设置加密salt为django.contrib.session+当前类名
3. 使用django.utils.crypto.salted_hmac进行加密
4. 把加密字符串进行base64编码，并转换为ascii编码，形成最终结果