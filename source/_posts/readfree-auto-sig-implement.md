---
title: readfree.me自动签到脚本
date: 2017-07-17 08:36:28
tags:
---

# readfree.me自动签到脚本

## 0. 前言

我们要知道readfree.me网站的签到规则，查询readfree的论坛找到自动签到获取积分的方法，就是在每天第一次访问网站时，会跳转到亚马逊，此时就完成了签到。

该签到规则是来自于[【重要】readfree额度规则变更通知](http://forum.readfree.me/t/readfree/302) ，其具体的规则如下：

>- Free用户通过每日签到，随机获取1-3个额度，且可累计。
>- 对Free用户的额度规则也适用于Pro用户，不过Pro用户的签到是在后台自动完成，无需跳转到亚马逊。
>
>
>- 签到是自动的，在你每天第一次访问网站时，会跳转到亚马逊，此时就完成了签到，你自己不需要做任何操作。

因此只要能够实现自动登录功能，然后每天自动登录一次，就可以实现自动签到功能。



##  1. 自动登录实现分析

首先想到的是使用post模拟登录网站的过程，以实现自动签到的功能。

直接查看登录页面，获取登录表单的代码，其代码如下：

```html
<form action="" id="id_signin_form" class="form" method="post">
    <input type="hidden" name="csrfmiddlewaretoken"  value="Mjh1RrCC6VJzOdIbOfD4f99H5YEas287oltY0ve4H7BtqiqzRtMcMUpRkai4g9O3">
    <label for="id_email">邮箱:</label>
  	<input type="email" name="email" maxlength="50" required="" id="id_email">
	<label for="id_password">密码:</label>
  	<input type="password" name="password" required="" id="id_password">
	<label for="id_captcha_1">认证码:</label>
	<img src="/captcha/image/1c84bab76745e92b7b856d05698f781bc5c8aa54/" alt="captcha" class="captcha">
  <input id="id_captcha_0" name="captcha_0" type="hidden" value="1c84bab76745e92b7b856d05698f781bc5c8aa54">
	<input autocapitalize="off" autocomplete="off" autocorrect="off" spellcheck="false" id="id_captcha_1" name="captcha_1" type="text">

    <div class="form-actions">
        <button type="submit" class="btn btn-primary btn-large"><i class="icon-ok icon-white"></i>登录</button>
    </div>
</form>
```

从上述表单中可以看到登录readfree会提交如下数据：

* 邮箱
* 密码
* csrf验证
* 验证码

其中邮箱、密码和csrf验证都比较好获取，因为在网页中可以直接获取到值，主要困难的是如何获取到验证码的值。



通过查询资料，发现还有一种更加简单的方法，直接使用登录后的cookie访问网站，可以绕过验证码。

使用chrome抓取登录时的http请求，在登录成功的页面上找到Request Headers，其中Cookie字段的值就是登录时使用的Cookie值:

![readfree_request_headers_cookie](readfree_request_headers_cookie.png)

分析一下Cookie的值其结果如下：

![cookie_table](cookie_table.png)

从上表可以看到，其总共包含6个字段，其中sessionid应该是表示用户登录的唯一标识。获取到了cookie的值后，就可以使用该cookie进行登录了。



Cookie的过期时间，通过抓取到的报文，可以获得cookie是什么时候能会过期。expires 代表 cookie 过期时间，这个属性有些过时。Max-Age 作为替代，代表cookie的生存期，以秒为单位。

![request_header](request_header.png)

可以看到Cookie的超期时间是1年，那么我们就可以安心的使用1年自动登录的脚本，而不用频繁的更换cookie。



## 2. 代码实现

通过python的requests发送登录请求，其代码实现如下：

```python
import requests
import logging
    
url='http://readfree.me/accounts/checkin'
cookie_str=''
    
logging.basicConfig(level=logging.INFO,
                    filename='/var/log/readfree_auto_sig.log',
                    filemode='a',
                    format='%(asctime)s - [line:%(lineno)d] - %(levelname)s: %(message)s')
    
if __name__ == "__main__":
    
    cookies={}
    for ck in cookie_str.split(';'):
        name,value=ck.strip().split('=',1)
        cookies[name]=value
    
    logging.info('start to sign ...')
    ret=requests.get(url, cookies=cookies)
    logging.info('sign %s'%ret)
```

## 3. 测试结果

使用Crontab设置定时任务，每天早上2点自动登录readfree.me网站，实现自动签到功能。

下面是3天的测试结果：

![测试结果](readfree_auto_sig_test_result.png)

执行readfree.py打印的日志如下：

```
[root@iZuf66fsxeqvr0ak92q63iZ ~]# cat /var/log/readfree_auto_sig.log 
2017-07-13 01:20:25,415 - [line:19] - INFO: start to sign ...
2017-07-13 01:20:26,286 - [line:21] - INFO: sign <Response [200]>
2017-07-13 01:23:01,341 - [line:19] - INFO: start to sign ...
2017-07-13 01:23:01,964 - [line:21] - INFO: sign <Response [200]>
2017-07-13 01:24:02,135 - [line:19] - INFO: start to sign ...
2017-07-13 01:24:02,798 - [line:21] - INFO: sign <Response [200]>
2017-07-13 02:00:02,035 - [line:19] - INFO: start to sign ...
2017-07-13 02:00:02,731 - [line:21] - INFO: sign <Response [200]>
2017-07-14 02:00:01,824 - [line:19] - INFO: start to sign ...
2017-07-14 02:00:02,806 - [line:21] - INFO: sign <Response [200]>
2017-07-15 02:00:02,140 - [line:19] - INFO: start to sign ...
2017-07-15 02:00:03,277 - [line:21] - INFO: sign <Response [200]>
2017-07-16 02:00:02,751 - [line:19] - INFO: start to sign ...
2017-07-16 02:00:03,695 - [line:21] - INFO: sign <Response [200]>
2017-07-17 02:00:02,942 - [line:19] - INFO: start to sign ...
2017-07-17 02:00:03,738 - [line:21] - INFO: sign <Response [200]>
```

后续会跟踪一段时间，看看是否还存在其他的问题。

## 4. 参考资料
1. [python接口自动化4-绕过验证码登录（cookie）](http://www.cnblogs.com/yoyoketang/p/6838596.html)
2. [python requests的安装与简单运用](http://blog.csdn.net/alpha5/article/details/24964009)
3. [python的requests在网络请求中添加cookies参数](http://blog.csdn.net/winterto1990/article/details/51213029)
4. [chrome浏览器怎么查看传入后台的POST的值？](https://segmentfault.com/q/1010000002902690)