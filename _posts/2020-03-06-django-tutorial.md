---
layout: article
title: Django框架简单应用
tags: Python Django
article_header:
  type: overlay
  theme: dark
  background_color: '#123'
  background_image: false
---

Django 框架在服务器上的配置，以及简易web应用

<!--more-->

# 安装

建议在虚拟环境中来运行，以下的应用是基于`Centos7`系统，`Anaconda Python3.7`虚拟环境下，`Django2.2`版本

Anaconda的相关用法参考[这里]()

在conda虚拟环境下安装Django：
```
# pip install django
```

# 创建Django项目

使用`django-admin`命令来创建

```
# django-admin startproject mysite
```

会自动在当前目录创建一个`mysite`文件夹，里面包含了初始化的项目文件结构：
```
mysite/
    manage.py
    db.sqlite3
    mysite/
        __init__.py
        settings.py
        urls.py
        wsgi.py
```

其中：

- mysite: 项目的容器，包含主要的配置文件
- manage.py: Django的命令行工具，它包含了可对Django项目进行操控的命令集
- mysite/__init__.py: 一个空文件
- mysite/settings.py: Django 项目的配置文件
- mysite/urls.py: Django 项目的 URL 注册文件
- mysite/wsgi.py: 一个 WSGI 兼容的 Web 服务器的入口


这时可以测试一下是否创建成功：
```
# python manage.py runserver 0:8000    // 后面的地址和端口可以不加，默认是localhost:8000
```

然后在浏览器上输入`localhost:8000`会看到django的默认页面

- 首次运行时可能会在页面上可能到`DisallowHost`错误，这时要修改`mysite/settings.py`文件，将其中的`ALLOWED_HOSTS = []`括号中添加ip地址，
或者直接`ALLOWED_HOSTS = ['*']`来允许所有ip访问


# 运行一个Django项目

## 创建app

django的程序运行一般是通过app来实现的，可以在命令行创建app：

```
# python manage.py startapp polls     // 创建名为polls的app
```

这时整个mysite的结构变为：
```
mysite/
    manage.py
    db.sqlite3
    mysite/
        __init__.py
        settings.py
        urls.py
        wsgi.py
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

## 编辑网页内容

下一步是设计你想要在页面上显示的内容，它是由app目录下的views.py文件决定的，对于polls app来说，就是`polls/views.py`文件

```python
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
    
def another(request):
    return HttpResponse("Now here is another page.")
```

Django的网页内容访问通常都是访问的views.py中的`函数`，比如上面的index函数

## 注册url地址

然后为你刚刚涉及的内容注册一个url地址，以便能够在浏览器地址输入栏中找到它

在Django中，url的注册有点类似`层层递进`，需要配置两个文件：

- 1. mysite目录下的urls.py文件

修改`mysite/urls.py`成以下内容

```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path('polls/', include('polls.urls')),
    path('admin/', admin.site.urls),
]
```

其中path()后面的第一个参数`'polls/'`指的是url子地址；第二个参数`include('polls.urls')`意为该url子地址指向的是polls这个app

- 2. app目录下的urls.py文件

新建`polls/urls.py`文件，输入以下内容：

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.index, name='index'),
    path('another', views.another, name='another'),
]
```

与上面类似，path()后面的第一个参数指的是url子地址（在本例中将index函数设计成polls app的根地址，所以该参数是一个空字符串）；第二个参数意为
该url子地址指向的是`views.py中的某个函数`；第三个name是一个可选参数，为url地址命名，这样可以方便模板套用等统一的操作

## 运行

经过以上两个文件的配置，此刻再运行这个项目
```
# python manage.py runserver 0:8000
```

然后在浏览器中输入`localhost:8000/polls/`和`localhost:8000/polls/another`就能分别看到index和another两个函数返回的字符串


# 使用模板

在前面的例子中，页面上显示的内容是在views.py中编辑的，但是对于大型网页，会使views.py变得非常复杂

通常的做法是将页面内容写在一个`html`文件中，再在`views.py`中渲染它

## 新建模板文件

首先在mysite的根目录下创建一个`templates`文件夹，用于存放html模板文件
```
# mkdir templates
```

然后在templates中创建一个base.html文件
```html
<!DOCTYPE html>

<head>
	<meta charset="UTF-8">
	<title>Test Page</title>
	<meta charset="UTF-8"/>
</head>

<body>
  <h1>h1h1h1h1h1</h1>
  <h2>h2h2h2h2h2h2h2h2</h2>
</body>
</html>
```

## 添加路径到Django的默认设置

为了使用方便，可以将templates文件添加到Django项目的默认路径中

修改`mysite/settings.py`文件：
```python
TEMPLATE_DIR = os.path.join(BASE_DIR, 'templates')  # 新建一个路径变量

TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [TEMPLATE_DIR,],    # 添加到这里
        'APP_DIRS': True,
        'OPTIONS': {
            'context_processors': [
                'django.template.context_processors.debug',
                'django.template.context_processors.request',
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ],
        },
    },
]
```

这样项目中的其他文件在寻找html模板时就会自动到templates中查找

## 在views.py中渲染模板

在上面的polls app的基础上，修改`views.py`文件：
```
from django.http import HttpResponse
from django.shortcuts import render  # 导入Django的渲染方法

def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
    
def another(request):
	  return render(request, 'base.html')   # 在这里直接使用base.html作为参数，会自动向templates路径下查找
```

## 运行

此时再运行这个项目
```
# python manage.py runserver 0:8000
```

在浏览器中输入`localhost:8000/polls/another`就能看到页面显示的是base.html中的内容


# 使用数据库

Django非常优秀的一点是它可以很方便的与服务器上的数据库连接，下面以MYSQL数据库为例展示简单用法

## 修改默认数据库

Django的默认数据库是Sqlite3，如果想要为项目使用其他的数据库（比如MYSQL）需要修改配置文件`mysite/settings.py`：
```python
# 修改DATABASES字段
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'db_name',         # 数据库名称，它必须是已存在的数据库
        'USER': 'test',
        'PASSWORD': 'test123',
        'HOST':'localhost',
        'PORT':'3306',
    }
}
```

然后运行命令：
```
# python manage.py migrate
```
就会在`db_name`数据库中创建一系列表（此前确保db_name数据库已经存在，否则会报错）

所创建的表是依据`mysite/settings.py`文件中`INSTALLED_APPS`配置项里面的内容

### 可能出现的错误

有时执行migrate会提示找不到mysqlclient（比如在centos7系统中），如何在centos7中安装mysqlclient参考
[这里](https://github.com/Chunar5354/some_notes/blob/master/notes/Centos%E7%9B%B8%E5%85%B3%E9%85%8D%E7%BD%AE.md)

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

## 在数据库中添加自定义的数据

针对MYSQL数据库，在数据库中创建表是通过在app目录下models.py创建新类来实现的

- 1. 首先创建models

编辑`polls/models.py`文件：

```python
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

为了能够将polls这个app连接到数据库，还需要进行一项配置：在`mysite/settings.py`文件的`INSTALLED_APPS`配置项中，添加：
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
# python manage.py makemigrations polls
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
# python manage.py migrate
```

会看到类似如下信息：
```
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, polls, sessions
Running migrations:
  Rendering model states... DONE
  Applying polls.0001_initial... OK
```

说明表格创建成功，可以到mysql数据库中查看

## 在Django中添加已有的数据库

首先修改`mysite/settings.py`中的DATABASES：
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME':'name',            # 改成要添加的数据库名
        'HOST':'127.0.0.1',
        'PORT':3306,
        'USER':'root',
        'PASSWORD':'yourpasswd',
    }
}
```

然后在命令行中：
```
# python manage.py inspectdb > ./polls/models.py
```

就会在polls文件夹中生成新的models.py替换原来的文件

再执行：
```
# python manage.py migrate
```
就将新的数据库添加到了django的配置当中

- tips：注意要在`mysite/setings.py`中的 `INSTALLED_APPS` 配置项中添加对应的app

## 管理已有数据库

用上面的方法添加已有数据库可能会出现一些问题，比如在migrate的时候因为格式不匹配等，可能会强制改变原来数据库的数据格式，
下面介绍一种方式可以将原有的数据库比较好的匹配到django框架当中：

- 1.首先将数据库备份

```
# mysqldump -u user -ppassword database > database.sql
```

user、password、database分别对应用户名、密码和数据库名

- 2.为django app生成model

```
# python manage.py inspectdb > ./app/models.py
```

此时会在该app的models.py文件中生成相应的代码用来构造数据库

注意要将models.py中的Meta里面managed属性修改为True，使得Django有操作数据库的权限：
```python
class Meta:
    managed = True
```

- 3.创建新的数据库并添加到Django的settings中

首先在mysql中创建一个新的数据库，比如`djangodb`：
```
MariaDB [(none)]> CREATE DATABASE djangodb;
```

然后修改mysite/settings.py，配置数据库和app信息（注意要修改两处）：
```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'djangodb',         # 改成新的数据库
        'USER': 'test',
        'PASSWORD': 'test123',
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
# python manage.py makemigrations app_name
```

然后全部迁移：
```
# python manage.py migrate
```

- 5.从备份文件导入数据

进入到mysql中，选中`djangodb`数据库：
```
MariaDB [(none)]> use djangodb;
```

将备份文件中的数据导入数据库：
```
MariaDB [(none)]> source database.sql;
```

这种方法比较神奇的地方在于导入数据之后数据的格式和原本的数据库格式相同

此时原来的数据库的数据就完美的融入了Django框架啦


## 在Django中使用redis数据库作为缓存

设置了redis作为django的缓存数据库后就可以使用django内置的cache方法来直接缓存数据，不需要再通过额外的调用python的redis扩展库

需要安装`django-redis`扩展库，[官方文档](https://django-redis-chs.readthedocs.io/zh_CN/latest/)

```
# pip install django-redis
```

### 使用方法

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
