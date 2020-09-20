---
layout: article
tags: Python Django Web
title: 在Django中发送电子邮件
article_header:
  type: overlay
  theme: dark
  background_color: '#123'
  background_image: false
---

在Django网页应用中实现电子邮件服务

<!--more-->

我们在浏览网页的时候，有时会遇到一些需要邮箱验证的服务，比如发送验证码等等，Django中提供了一个`mail`模块来实现邮件服务

完整代码请见[这里](https://github.com/Chunar5354/Django-demos/tree/master/EmailDemo)

在Python中邮件的处理是基于SMTP协议的，[什么是SMTP？](https://zh.wikipedia.org/wiki/%E7%AE%80%E5%8D%95%E9%82%AE%E4%BB%B6%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)

要在Django项目中实现电子邮件服务，首先需要为邮箱开通SMTP服务，并获取授权码，本例中使用的是QQ邮箱，[如何获取QQ邮箱授权码？](https://service.mail.qq.com/cgi-bin/help?subtype=1&&no=1001256&&id=28)

## 配置

创建好Django项目后，首先需要在settings.py中配置`Email backends`，在其中填写以下内容：

```python
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'

# 指定邮箱地址，也可以换成163邮箱等
EMAIL_HOST = 'smtp.qq.com'
# 指定SMTP端口，默认是25，如果使用SSL加密的话设置成465或587
EMAIL_PORT = 465
# 指定发送邮件的邮箱账号
EMAIL_HOST_USER = '123456@qq.com'
# 邮箱的授权码
EMAIL_HOST_PASSWORD = 'qgcttpwoxmnbbaha'
# 为邮件开启SSL加密
EMAIL_USE_SSL = True
```

更多的配置参数请参考[Django官方文档](https://docs.djangoproject.com/en/3.1/topics/email/#email-backends)

## 代码实现

在views.py中：

```python
from django.shortcuts import render, HttpResponse
from django.core.mail import send_mail


def mail_send(request):
    send_mail(
         'Chunar sends you a message.',  # 邮件主题
         'hellow man',  # 邮件正文
         '123456@qq.com',  # 发送方账号
        ['654321@163.com',],  # 接收方账号，可以指定多个
        fail_silently = False,  # 发送失败是否返回错误信息
    )
```

### 页面消息提醒

在网页中点击发送邮件后，我们希望能够在页面上给出一个反馈，这一功能可以通过Django中的`messages`来实现

在views.py中：

```python
from django.contrib import messages

def mail_send(request):
    ...
    messages.success(request, 'Successfully sended')  # 发送一个处理成功提醒
```

并在相应的html页面中：

```html
<script>
    { if messages }
        { for msg in messages }
            alert('{{ msg.message }}');  //弹出提醒窗口
        { endfor }
    { endif }
</script>
```

（注意这里所有单独的的{}中还需要加上%，因为根Jekyll格式有冲突，所以在这里用{}表示）

此时views.py的完整代码就变成：

```python
from django.shortcuts import render
from django.http import HttpResponse, HttpResponseRedirect
from django.core.mail import send_mail
from django.contrib import messages


def index(request):
	return render(request, 'index.html')

def mail_send(request):
	send_mail(
		'Chunar sends you a message.',
		'hellow man',
		'123456@qq.com',
		['654321@163.com',],
		fail_silently = False,
	)
    # 成功发送后在页面上给出提醒
	messages.success(request, 'Successfully sended')
	return HttpResponseRedirect('/')  # 刷新页面
```
