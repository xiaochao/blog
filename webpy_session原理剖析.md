Title: webpy session原理剖析
Meta: webpy session原理剖析
Date: 2016-02-02
Tags: Python,webpy,session,加密
Category: python库
Slug: webpysession
Author: 笨熊

#### 简介
webpy的session位于源码目录的session.py文件中。所有session相关的操作都在这个文件里。

#### session id和value是如何加密的

* 位于session.py的154~158行，是session key的加密方式，具体代码如下：
		
		rand = os.urandom(16)
		now = time.time()
		secret_key = self._config.secret_key
		session_id = sha1("%s%s%s%s" %(rand, now, utils.safestr(web.ctx.ip), secret_key))
		session_id = session_id.hexdigest()
	首先生成16个随机字节，然后获取当前时间，然后获取session加密的key，然后把这三个数据连同客户端ip，按照一定的规则(随机数+时间+ip+secret_key),使用sha1加密。
	这地方os.urandom和random的区别是，os.urandom系统的随机数，random伪随机数。
		

* 位于session.py的204、205行，是session的加密方式，具体加密方式如下：
		
		pickled = pickle.dumps(session_dict)
		return base64.encodestring(pickled)
	我们可以看出，其实就是把session的数据dict对象直接序列化，然后进行了base64编码，整个过程很简单，说加密，其实根本算不上加密。
