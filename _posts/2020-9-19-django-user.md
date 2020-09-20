---
layout: article
tags: Python Django Web
title: 为Django网页添加用户登录功能
article_header:
  type: overlay
  theme: dark
  background_color: '#123'
  background_image: false
---

通过Django内置的User实现网页的用户登录功能，包含用户注册、登录以及登出

<!--more-->

主要参考了[这篇文章](https://blog.csdn.net/weixin_43249914/article/details/86772432)

以及Django[官方文档](https://docs.djangoproject.com/en/3.1/topics/auth/)

完整代码在[这里](https://github.com/Chunar5354/Django-demos/tree/master/UserDemo)

## User模块用法简介

在WEb应用中常常会有需要用户验证的场景，Django提供了User模块来很方便的实现这个功能

- 创建用户：

```python
from django.contrib.auth.models import User

user = User.objects.create_user('john', 'lennon@thebeatles.com', 'johnpassword')
```

create_user()的参数分别对应用户名、邮箱和密码，这三个是必填项，此外还可以设置其他的信息，比如用户姓名以及用户组

创建的用户信息保存在数据库的`auth_user`表中

- 用户信息验证：

```python
from django.contrib.auth import authenticate

user = authenticate(username=data['username'], password=data['password'])
```

当用户存在时，authenticate()方法返回一个User对象，包含了这个用户的信息，如果不存在则返回一个None对象

- 用户登录

```python
from django.contrib.auth import login

def user_login(request):
    user = authenticate(username=data['username'], password=data['password'])
    login(request, user)
```

login()方法接收两个参数：一个HttpRequest对象和一个User对象

- 用户登出

```python
from django.contrib.auth import logout

def user_logout(request):
    logout(request)
```

logout()方法只接受一个HttpResponse对象

## 使用forms模块格式化输入

因为需要用户在页面上输入信息，可是每个用户输入内容的格式可能五花八门，为了统一这些信息的格式，可以使用Django的forms模块来将信息格式化

```python
from django import forms

class UserRegisterForm(forms.Form):
	username = forms.CharField()
	password = forms.CharField()
	email = forms.EmailField()
```

这里指定了用户注册时，username和password是字符串类型，email是Email类型，如果在Email一项中填入了不包含`@`的字符串，将视为非法

其实email的格式控制也可以通过前端文件来实现，在register.html中可以指定input的类型为email，这样在输入不正确的邮箱地址时会给出提示

```xml
<div class="input-field col s12">
    <label for="email">Email</label>
    <input type="email" class="form-control" id="email" name="email">
</div>
```

## 实例分析

完整的示例可以查看给出的源码，下面列出一些个人认为的重点

### 1.使用forms格式化输入信息

如上一节所描述的，通过forms模块将输入信息的格式进行统一，这里单独创建了一个文件`form_s.py`来生成Form对象，并在`views.py`中

```python
def user_login(request):
	if request.method == 'POST':
        # 将前端form表单中的数据传给UserLoginForm对象
		user_login_form = UserLoginForm(data=request.POST)
		if user_login_form.is_valid():
            # 通过cleaned_data将信息格式化
			data = user_login_form.cleaned_data
```

这样得到的data就是格式化之后的`UserLoginForm`对象

### 2.用户登录与否的两种响应方式

既然要使用用户功能，就表示应该处理`用户未登录`和`用户已登录`两种响应，在views.py中：

```python
if request.method == 'POST':
    ...
    # 请求信息中的请求方法为POST，意味着这是用户通过提交表单来发起的请求，说明这是一个用户登录的过程
    if request.method == 'POST':    
		if user:
			login(request, user)
            # 如果用户登陆成功，就通过redirect来刷新页面
			return HttpResponseRedirect('/')
		else:
			return HttpResponse('current user not exists')

    # 如果请求方法为GET，说明是发起的一次页面请求，这时应该显示供用户输入信息的登录界面
	elif request.method == 'GET':
		user_login_form = UserLoginForm()
		context = {'form': user_login_form}
		return render(request, 'authenticate/login.html', context)
```

也可以将用户登陆操作和提供用户登录界面这两种响应方式分别放在两个函数中


### 3.用户登录与否的两种界面显示

同样的，在前端的界面显示中，也需要有已登录和未登录两种模式

在Django中，前端html文件也含有user对象，可以直接进行判断

```html
{% if user.is_authenticated %}
	<div class="container">
		{% block content %}

		{% endblock content %}
	</div>
{% else %}
	<h5 style="text-align: center">Please <a href="/login">sign in</a></h5>
{% endif %}
```

上面的代码表示如果用户已登录，将显示content中的内容，否则给出登录提示

这里要注意不能把登录界面也放在content中，因为这样在未登录的时候无论如何也不会显示登陆界面

这里我是额外复制了一份base.html的内容，这样就多了一份文件，不过暂时没有想到更好的解决办法

