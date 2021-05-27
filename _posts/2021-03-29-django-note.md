---
layout: article
tags: Python Django
title: Django学习笔记
article_header:
  type: overlay
  theme: dark
  background_color: '#123'
  background_image: false
---

学习Django框架时的一些知识点

<!--more-->


# 安装

## 树莓派上(Debian)

```
pip3 install Django
```
安装命令执行之后，会给出一条提醒，说没有将django添加到环境变量中，并会给出当前的django安装位置，可能是`home/pi/.local/bin`

将django添加到环境变量：
```
export PATH=$PATH:/home/pi/.local/bin
```

## centos7

```
pip3 install django
```

仍然需要添加环境变量，不过这里要保存在`~/.bash_profile`中，在~/.bash_profile中添加一行：
```
export PATH=$PATH:/usr/local/python3/bin
```

在运行时，可能会因为sqlite的版本过低而出错，即使更新了最新版本也不行，这时，需要在环境变量中添加共享库位置，还是在`~/.bash_profile`中添加一行：
```
 export LD_LIBRARY_PATH="/usr/local/lib"
```

每当修改过该文件之后，执行`source ~/.bash_profile`使改动生效

# 创建项目

可以使用`django-admin`工具来自动创建
```
django-admin startproject mysite
```

会自动在当前目录创建一个`mysite`文件夹，里面包含了初始化的项目文件结构：
```
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        wsgi.py
```

其中：

- mysite: 项目的容器
- manage.py: 一个实用的命令行工具，可让你以各种方式与该 Django 项目进行交互
- mysite/__init__.py: 一个空文件，告诉 Python 该目录是一个 Python 包
- mysite/settings.py: 该 Django 项目的设置/配置
- mysite/urls.py: 该 Django 项目的 URL 声明; 一份由 Django 驱动的网站"目录"
- mysite/wsgi.py: 一个 WSGI 兼容的 Web 服务器的入口，以便运行你的项目

这时可以测试一下是否创建成功：
```
python3 manage.py runserver 0.0.0.0:8000    // 后面的地址和端口可以不加，默认是localhost:8000
```

然后在浏览器上输入`localhost:8000`会看到django的默认页面

- 首次运行时可能会在页面上可能到`DisallowHost`错误，这时要修改`mysite/settings.py`文件，将其中的`ALLOWED_HOSTS = []`括号中添加ip地址，
或者直接`ALLOWED_HOSTS = ['*']`来允许所有ip访问

# 创建app

django的程序运行一般是通过app来实现的，可以在命令行创建app：

```
python manage.py startapp polls     // 创建名为polls的app
```

会新建一个名为polls的文件夹，内部初始结构：
```
polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    views.py
```

## 在浏览器上显示自定义内容

需要编辑`3个`文件
- 1.首先是`polls/views.py`：
```python
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```

- 2.然后在polls目录下新建一个`urls.py`文件：
```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
]
```

- 3.最后修改`mysite/urls.py`：
```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),   # 文件路径
    path('admin/', admin.site.urls),
]
```

启动django：
```
python manage.py runserver
```

在浏览器中输入`localhost:8000/polls`就能看到views中的内容

- 如果页面显示`DisallowedHost`的错误，需要到`mysite/mysite/settings.py`中改写这一段：
```python
ALLOWED_HOSTS = ['*']   # 在括号中添加'*'
```

# django操作mysql数据库

## 1.配置mysql支持并创建数据库

django默认的数据库是SQLite，要修改成mysql的支持，修改`mysite/settings.py`，将DATABASES配置项的内容替换成下面的内容：
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'db_name',                           # database name, must be created before
        'USER': 'test',                           # your own mysql user name
        'PASSWORD': 'test123',                    # your password to the user above
        'HOST':'localhost',
        'PORT':'3306',
    }
}
```

然后运行：
```
python manage.py migrate
```
就会在`db_name`数据库中创建一系列表（此前确保db_name数据库已经存在，否则会报错）

所创建的表是依据`mysite/settings.py`文件中`INSTALLED_APPS`配置项里面的内容

### 常见错误

有时执行migrate会提示找不到mysqlclient（比如在centos7系统中），如何在centos7中安装mysqlclient参考[这里](https://github.com/Chunar5354/some_notes/blob/master/notes/Centos%E7%9B%B8%E5%85%B3%E9%85%8D%E7%BD%AE.md)

或者使用`pymysql`来实现migrate，首先安装pymysql：
```
pip3 install pymysql
```

然后修改`mysite/__init__.py`，在其中添加：
```python
import pymysql

pymysql.install_as_MySQLdb() 
```

此时运行`python manage.py migrate`仍然会报错，错误发生在一个`base.py`文件中，编辑它（换成自己的路径）：
```
vim /usr/local/python3/lib/python3.8/site-packages/django/db/backends/mysql/base.py
```

注释掉其中的35 36行

再次运行`python manage.py migrate`仍然会报错，错误发生在一个`operations.py`文件中，编辑它（换成自己的路径）：
```
vim /usr/local/python3/lib/python3.8/site-packages/django/db/backends/mysql/operations.py
```

将其146行的`decode`替换为`encode`

此时可以正常运行`python manage.py migrate`


## 2.为数据库添加自定义数据（models）

- 1.创建新表，编辑`polls/models.py`：
```
from django.db import models

class Question(models.Model):   # 类名对应表名
    # 变量名(question_text)对应列名，后面代表数据类型，CharField相当于mysql中的varchar
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

为了django能够找到polls这个app，还需要进行一项配置：在`mysite/settings.py`文件的`INSTALLED_APPS`配置项中，添加：
```python
INSTALLED_APPS = [
    'polls.apps.PollsConfig',  # 这一行是新添加的，代表polls app（polls路径下有一个apps.py文件）
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

- 2.然后运行命令：
```
python manage.py makemigrations polls
```

会看到打印出类似下面的信息，表示app包含成功：
```
Migrations for 'polls':
  polls/migrations/0001_initial.py:
    - Create model Choice
    - Create model Question
```

- 3.最后，在数据库中创建表格：
```
python manage.py migrate
```

会看到如下信息：
```
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, polls, sessions
Running migrations:
  Rendering model states... DONE
  Applying polls.0001_initial... OK
```
说明表格创建成功，可以到mysql数据库中查看

- tips

在执行完第2步makemigrations之后，会发现在`polls/migrations`目录中多出了一个文件`0001_initial.py`，里面包含了创建表的一些命令，此时可以通过下面的命令查看相应的SQL语句：
```
python manage.py sqlmigrate polls 0001
```

# 创建用户

创建用户之后可以在`localhost:8000/admin`界面登录来查看数据库，输入命令：
```
python manage.py createsuperuser
```

接下来根据提示依次设置用户名、邮箱和密码，新用户创建成功

不过这时在浏览器上登陆看不到我们自己的数据库，还需要将我们的app中数据库的类（在models中）添加到admin注册中，编辑`polls/admin.py`：
```python
from django.contrib import admin
from .models import Question      # 从models文件中引入表

admin.site.register(Question)     # 将表添加到admin中
```

保存该文件，再从浏览器上登录，就能够看见`Question`表，并对其进行操作

# 添加已有的数据库

首先修改`mysite/settings.py`中的DATABASES：
```python
DATABASES = {
    'default': {
        'NAME':'jd',            # 改成要添加的数据库名
        'HOST':'127.0.0.1',
        'PORT':3306,
        'USER':'root',
        'PASSWORD':'yourpasswd',
    }
}
```

然后在命令行中：
```
python manage.py inspectdb > ./polls/models.py
```

就会在polls文件夹中生成新的models.py替换原来的文件

再执行：
```
python manage.py migrate
```
就将新的数据库添加到了django的配置当中

- tips：注意要再`mysite/setings.py`中的 `INSTALLED_APPS` 配置项中包含models的app（因为有可能不是polls）

## 实现管理已有数据库

用上面的方法添加已有数据库可能会出现一些问题，比如在migrate的时候因为格式不匹配等，可能会强制改变原来数据库的数据格式，下面介绍一种方式可以将原有的数据库比较好的匹配到django框架当中：

- 1.首先将数据库备份

```
mysqldump -u user -ppassword database > database.sql
```

user、password、database分别对应用户名、密码和数据库名

- 2.为django app生成model

```
python manage.py inspectdb > ./app/models.py
```

此时会在该app的models.py文件中生成相应的代码用来构造数据库

注意要将models.py中的Meta里面managed属性修改为True，使得django有操作数据库的权限：
```python
class Meta:
    managed = True
```

- 3.创建新的数据库并添加到django的settings中

首先在mysql中创建一个新的数据库，比如`djangodb`：
```
CREATE DATABASE djangodb;
```

然后修改mysite/settings.py，配置数据库和app信息（注意要修改两处）：
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'djangodb',                       # database name, must be created before
        'USER': 'test',                           # your own mysql user name
        'PASSWORD': 'test123',                    # your password to the user above
        'HOST':'localhost',
        'PORT':'3306',
    }
}

INSTALLED_APPS = [
    'app_name.apps.App_nameConfig',  # 将app_name和App_name换成自己app的名称
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
]
```

- 4.执行migrate

首先添加app的内容：
```
python3 manage.py makemigrations app_name
```

然后全部迁移：
```
python3 manage.py migrate
```

- 5.从备份文件导入数据

进入到mysql中，选中`djangodb`数据库：
```
use djangodb;
```

将备份文件中的数据导入数据库：
```
source database.sql;
```

这个命令最神奇的是导入数据之后数据的格式和原本的数据库格式相同

此时原来的数据库的数据就完美的融入了django框架啦

# 使用channels模块实现websocket

安装channels：
```
pip install channels
```

在项目文件的settings.py文件中添加channels到INSTALL_APPS，并指定ASGI的路由地址：
```python
# 添加channels
INSTALLED_APPS = [
    'django.contrib.staticfiles',
    'channels',
]

# 指定ASGI的路由地址
ASGI_APPLICATION = 'webapp.routing.application'
```

并在settings.py的同一路径下创建routing.py路由文件
```python
from channels.routing import ProtocolTypeRouter

application = ProtocolTypeRouter({
    # 暂时为空
})
```

此时已经基本搭建好了channels框架

# 设置时区以及中文显示

修改`mysite/settings.py`，文件，将其中的
```python
LANGUAGE_CODE = 'zh-Hans'     # 显示中文
TIME_ZONE = 'Asia/Shanghai'   # 设置时区
USE_TZ = False                # 如果原来数据库中已经有关于时间的数据，就要设置这个
```
三行改成这个样子


# Django中使用redis作为缓存

设置了redis作为django的缓存数据库后就可以使用django内置的cache方法来直接缓存数据，不需要再通过额外的调用python的redis扩展库

需要安装`django-redis`扩展库，[官方文档](https://django-redis-chs.readthedocs.io/zh_CN/latest/)

```
# pip install django-redis
```

## 使用方法

首先需要再settings中添加这样一段：
```python
CACHES = {
    "default": {
        "BACKEND": "django_redis.cache.RedisCache",
        "LOCATION": "redis://127.0.0.1:6379/1",      # IP 端口 和数据库编号
        "OPTIONS": {
            "CLIENT_CLASS": "django_redis.client.DefaultClient",
            "PASSWORD": "mysecret"    # 密码
        }
    }
}
```

然后就可以在django项目中直接访问redis
```python
from django.conf  import settings
from django.core.cache import cache   # 通过cache来访问缓存，此时的默认缓存已经设置成了redis

#read cahce user_id
def read_from_cache(self,username)
   key='user_id_of'+username
   value=cache.get(key)
   if value==none:
       data=none
   else:
       date=json.loads(value)
   return data

#write cahche
def write_to_cache(self,username)
    key='user_id_of'+username
    cache.set(key,json.dumps(username),settint.NEVER_REDIS_TIMEOUT)
```

- tips

django-redis的功能不是很全，查询和写入时只能够使用简单的get和set方法，不能使用诸如hash等功能

# Django支持markdown

首先安装Python的markdown模块

```
$ pip install markdown
```

在views.py中添加

```python
import markdown

def some_page(request):
    ...

    # markdown渲染
    article.body = markdown.markdown(article.body,
        extensions=[
        # 包含 缩写、表格等常用扩展
        'markdown.extensions.extra',
        # 语法高亮扩展
        'markdown.extensions.codehilite',
        ])
    
    ...
    
    return render(request, 'some_page.html')
```

# 前端小知识

在页面中为代码块插入语法高亮

可以到[highlight](https://highlightjs.org/)中去选择样式，然后通过cdn远程部署

将

```xml
<link rel="stylesheet"
      href="//cdnjs.cloudflare.com/ajax/libs/highlight.js/10.6.0/styles/default.min.css">
```

放到前面head里面

将

```xml
<script src="//cdnjs.cloudflare.com/ajax/libs/highlight.js/10.6.0/highlight.min.js"></script>
<!-- 自动识别代码块-->
<script>
  hljs.initHighlightingOnLoad();
</script>
```

放到页面最后

# 导出依赖文件

- 通过pip

```
$ pip freeze > requirements.txt
```

该命令会导出当前环境中的所有Python包，会包含大量的冗余

另一种方法是通过pipreqs导出只在当先路径下Python文件中import的包

- 通过pipreqs

首先安装pipreqs

```
$ pip install pipreqs
```

导出依赖：

```
$ pipreqs ./ --encoding=utf-8 --force
```

会自动在当前路径下生成`requirements.txt`文件


# 修改默认的用户表

在实际应用中常常需要保存用户的电话号码信息，而Django默认的auth_user表中并没有这个字段，就需要修改默认的用户模型

分为几个步骤

- 1.修改models.py

在某个APP下的models.py中输入下面的内容：

```python
from django.db import models
from django.contrib.auth.models import AbstractUser
 
 
class UserProfile(AbstractUser):
	mobile = models.CharField(max_length=11, primary_key=True, verbose_name="手机号", default="", error_messages={'unique': '手机号已存在'})
	username = models.CharField(max_length=30, verbose_name="用户名", default="")
 
	class Meta:
		verbose_name = "用户信息"
		verbose_name_plural = verbose_name
 
	def __str__(self):
		return self.username
```

自定义的用户类要继承`AbstractUser`类

这里做了两个比较重大的修改：

一是将mobile设置成了`主键`，它会取代原来默认的自增id主键，新建的表中将没有自增id字段

二是username原本是默认唯一（unique），在UserProfile中将它重写后将不再具有unique属性

- 2.修改settings.py

在settings.py中添加：

```python
AUTH_USER_MODEL = 'userprofile.UserProfile'
```

userprofile是我的app名称mUserProfile是类名

- 3.migrate

```
$ python manage.py makemigrations
$ python manage.py migrate
```

注意因为不仅是向原auth_user表添加新字段，还修改了原本的字段（主要是因为取消了原来的自增id主键），所以migrate操作一定要在建库的开始执行，否则可能会报错

如果不是在建库的开始执行，报错的话可以尝试删掉`app/migrations/`下的所有`000x_xxx.py`文件，重新执行makemigrations和migrate

或者可以备份数据后删库重建

执行成功后会在指定的库中创建`userprofile_userprofile`表(app_小写的class)

```
+--------------+--------------+------+-----+---------+-------+
| Field        | Type         | Null | Key | Default | Extra |
+--------------+--------------+------+-----+---------+-------+
| password     | varchar(128) | NO   |     | NULL    |       |
| last_login   | datetime(6)  | YES  |     | NULL    |       |
| is_superuser | tinyint(1)   | NO   |     | NULL    |       |
| username     | varchar(30)  | NO   |     | NULL    |       |
| first_name   | varchar(30)  | NO   |     | NULL    |       |
| last_name    | varchar(150) | NO   |     | NULL    |       |
| email        | varchar(254) | NO   |     | NULL    |       |
| is_staff     | tinyint(1)   | NO   |     | NULL    |       |
| is_active    | tinyint(1)   | NO   |     | NULL    |       |
| date_joined  | datetime(6)  | NO   |     | NULL    |       |
| mobile       | varchar(11)  | NO   | PRI | NULL    |       |
+--------------+--------------+------+-----+---------+-------+
```

注意原来的自增id没有了，而且username也不是unique了

# 修改用户验证方式

大多数的web应用都支持用户名、手机号和邮箱等多种登陆方式，但Django默认只支持用户名，需要对其进行扩展

在某APP的views.py中（可以在任何地方，views.py比较方便）：

```python
from django.contrib.auth.backends import ModelBackend
from django.db.models import Q

class CustomBackend(ModelBackend):
	def authenticate(self, request, username=None, password=None, **kwargs):
		try:
            # 传进的username参数等于表中的username，email，mobile中任何一个字段都可以
			user = UserProfile.objects.get(Q(username=username) | Q(email=username) | Q(mobile=username))
			if user.check_password(password):
				return user
		except Exception as e:
			return None
```

然后修改settings.py，添加：

```python
AUTHENTICATION_BACKENDS = [
    'userprofile.views.CustomBackend',
]
```

userprofile是我的app名，实际改成自己的

然后就可以在登录的时候使用：

```python
from django.contrib.auth import login
from django.http import HttpResponse

def user_login(request):
	...
	customer_auth = CustomBackend()
	user = customer_auth.authenticate(request, username=your_usernmae, password=your_password)
	if user:
		login(request, user)
		return HttpResponse('登录成功')
	else:
		return HttpResponse('账号或密码有误，请重新输入')
```
