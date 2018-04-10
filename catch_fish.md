Title: 脚本薅区块鱼羊毛
Meta: 这是一个薅区块鱼羊毛的脚本，通过模拟用户注册、激活，来完成模拟用户邀请
Date: 2018-01-23
Tags: python,selenium,webdriver,教程,薅羊毛
Category: python库
Slug: catch_fish
Author: 笨熊

### 概述
&emsp;&emsp;[区块鱼](https://fishbank.io/)是一个基于区块链的游戏，这个游戏目前有个邀请活动，邀请好友注册，送相应种类的鱼，不同的鱼价格不一样

- 普通鱼 0.015 ETH起(邀请3个好友)
- 稀缺鱼 0.05 ETH起(邀请15个好友)
- 史诗鱼 0.35 ETH起(邀请100个好友)
- 传奇鱼 1.5 ETH起(邀请1000个好友)

最重要的是，这个网站是用邮箱注册的，并且没有屏蔽掉临时邮件服务，所以我们就有嘿嘿嘿了。

### 工具准备
- 一个临时邮箱服务：我这地方选用的是[YOPmail](http://www.yopmail.com/zh/)
- Python
- chrome webdriver
- selenium
- iterm2(安装imgcat 工具): 因为要输入验证码，所以选用iterm2，这样可以把验证码图片输出到终端。

### 步骤
#### 获取自己的邀请链接
&emsp;&emsp;去区块鱼的网站，注册一个账户，获取到自己的邀请链接。

#### 获取一个临时邮箱
- 打开YOPmail网站，获取一个邮箱地址，如下图
![](http://7xpx6h.com1.z0.glb.clouddn.com/797f2c581dcc045b1e9f265916e3f736)
![](http://7xpx6h.com1.z0.glb.clouddn.com/ff76d7817b143a84aee7666fbcf9acce)
注意第一张图片，有个查看邮箱按钮，输入临时邮箱，点击这个按钮，就可以查看这个临时邮箱收到的邮件，这在获取激活连接有用。整个流程代码实现如下：

```python
    driver.get('http://www.yopmail.com/zh/email-generator.php')
    time.sleep(1)
    email = driver.find_element_by_id("login")
    email = email.get_attribute('value')
```

#### 模拟邀请注册
- 上一步，我们获取到了临时邮箱
- 打开自己的邀请链接，然后依次点击登录->注册，然后填写注册信息，如下图
![](http://7xpx6h.com1.z0.glb.clouddn.com/a0bab39b362701b2230081663a3d3db8)
![](http://7xpx6h.com1.z0.glb.clouddn.com/139ded20400a8a8920d92094da7745b5)
![](http://7xpx6h.com1.z0.glb.clouddn.com/d7bf570ea5596073650c99eeaf92365f)

- 代码如下

```python
    driver.get('http://my.fishbank.io/go/122169')
    time.sleep(1)
    login_btn = driver.find_element_by_css_selector('.button.red.bigrounded.big')
    login_btn.click()
    driver.get('https://my.fishbank.io/register')
    time.sleep(1)
    email_input = driver.find_element_by_id('user_email')
    password_one = driver.find_element_by_id('user_plainPassword_first')
    password_two = driver.find_element_by_id('user_plainPassword_second')
    cap_input = driver.find_element_by_id('user_captcha')
    register_btn = driver.find_element_by_css_selector('.button.green.bigrounded.mid')
```
- 因为有验证码的问题，而且简单的验证码识别库还得识别不出来，所以，这地方不打算花太多时间，直接把验证码图打印到终端，手动输入

```python
    cap = driver.find_element_by_class_name('captcha_image')
    with open(image_path, 'wb') as fi:
        fi.write(base64.b64decode(cap.get_attribute('src').split(',')[1]))
    os.system(imgcat+' '+image_path)
    code = input('输入验证码')
```

- 填入数据，点击注册按钮

```python
    password = ''.join(random.sample(string.ascii_letters+string.digits, 10))
    email_input.send_keys(email)
    password_one.send_keys(password)
    password_two.send_keys(password)
    cap_input.send_keys(code)
    time.sleep(2)
    register_btn.click()
```

#### 邮箱激活
- 注册成功后，我们的临时邮箱就会收到一封注册激活的邮件，打开第一步的邮箱页面，输入邮箱，点击检查按钮，就可以打开邮箱了。

```python
    driver.get('http://www.yopmail.com/zh/')
    time.sleep(1)
    email_input = driver.find_element_by_id('login')
    check_btn = driver.find_element_by_class_name('sbut')
    email_input.send_keys(email)
    check_btn.click()
```
- 打开邮箱页面后，我发现，邮箱的内容是以iframe的形式展现的，所以，这地方要处理一下：

```python
    driver.switch_to_frame(driver.find_element_by_id('ifmail'))
    try:
        html = driver.find_element_by_id('mailmillieu')
    except Exception as e:
        input('遇到机器识别的问题，切换到浏览器点击一下，验证完敲一下回车')
        html = driver.find_element_by_id('mailmillieu')
    html = html.text
    active_url = html.split('account:')[1].strip()
    driver.get(active_url)
    time.sleep(1)
    driver.delete_all_cookies()
    time.sleep(1)
```

- 这地方有个需要注意的地方，就是打开邮箱次数多了，YOPmail会出一个机器识别的检测，所以代码中有个try catch语句，来判断是否遇到了这个机器检测，如果遇到了，则需要自己点击一下那个检测，然后继续运行代码。
- 获取到注册链接后，直接打开激活就可以了。

#### 成果展示
![](http://7xpx6h.com1.z0.glb.clouddn.com/5d4b9fd22f673b7e6f9c8fbdc2b3641e)

### 总结
- 这个脚本就是简单的利用python的selenium库，来模拟用户注册的流程，以达到邀请用户的目的。
- 这个脚本也有很多不完善的地方，比如验证码识别、机器人检测、一些错误判断都没有，待完善的地方还有很多。
- 这个脚本只是为了和大家交流学习。

### 相关资源
- 详细代码地址：[https://github.com/xiaochao/CatchFish](https://github.com/xiaochao/CatchFish)


