---
layout: article
tags: uWSGI Python Django
title: 通过uWSGI+nginx部署Django项目
article_header:
  type: overlay
  theme: dark
  background_color: '#123'
  background_image: false
---

在服务器上利用uwsgi和nginx部署Django项目

<!--more-->

WSGI(Web Server Gateway Interface) Web服务器网关接口，是Python中定义的一种`Web服务器`和`Web应用`之间的通用接口，是一种`协议`

uWSGI是一个web服务器应用，通过它可以实现WSGI协议，部署Django项目的时候性能更好

## 安装uwsgi

在anaconda虚拟环境中：

```
# conda install -c conda-forge libiconv
# conda install -c conda-forge uwsgi
```

## 搭建

### 测试

编写一个test.py文件：

```python
def application(env, start_response):
    start_response('200 OK', [('Content-Type','text/html')])
    return [b"Hello World"]
```

在命令行运行：

```
# uwsgi --http :8000 --wsgi-file test.py
```

在浏览器输入`your_ip:8000`可以看到页面


### 通过配置文件启动Django项目：

编写`myproject.ini`文件

```
[uwsgi]
http = :8000
chdir = /my/own/project/location/
module = myproject.wsgi:application
```

其中chdir是Django项目的路径，module是wsgi.py文件中的appliacation

通过`uwsgi myproject.ini`启动

注意配置端口那里使用的是`http`，这样才可以通过浏览器访问页面


### 配置nginx

- 1.编辑配置文件

首先在nginx目录下新建一个`sites-available`文件夹

```
# sudo mkdir /etc/nginx/sites-available
```

然后在sites-available中新建一个配置文件`mysite_nginx.conf`，输入以下内容：

```
# mysite_nginx.conf

# the upstream component nginx needs to connect to
upstream django {
    # server unix:///path/to/your/mysite/mysite.sock; # for a file socket
    server 127.0.0.1:8001; # for a web port socket (we'll use this first)
}

# configuration of the server
server {
    # the port your site will be served on
    listen      8000;
    # the domain name it will serve for
    server_name example.com; # substitute your machine's IP address or FQDN
    charset     utf-8;

    # max upload size
    client_max_body_size 75M;   # adjust to taste

    # Django media
    location /media  {
        alias /path/to/your/mysite/media;  # your Django project's media files - amend as required
    }

    location /static {
        alias /path/to/your/mysite/static; # your Django project's static files - amend as required
    }

    # Finally, send all non-media requests to the Django server.
    location / {
        uwsgi_pass  django;
        include     /path/to/your/mysite/uwsgi_params; # the uwsgi_params file you installed, usually at '/etc/nginx/uwsgi_params'
    }
}
```

**注意** 这里如果想配置成不带端口号即可访问页面的形式，只需将`listen   8000;`中的8000改为80

- 2.创建sites-enabled文件夹

可以先查看一下/etc/nginx/目录下有没有`sites-enabled`文件夹，如果有的话就直接跳到第3步

如果没有的话，首先编辑`/etc/nginx/nginx.conf`，在http block内添加一句：

```
include /etc/nginx/sites-enabled/*;
```

然后在/etc/nginx目录下新建一个sites-enabled文件夹

```
# sudo mkdir /etc/nginx/sites-enabled
```

重启nginx

```
# sudo systemctl restart nginx
```

- 3.指定nginx链接

```
# sudo ln -s /etc/nginx/sites-available/mysite_nginx.conf /etc/nginx/sites-enabled/
```

然后重启nginx

- 4.测试

此时已经将uwsgi添加到了nginx中，注意上面配置文件中指定的端口`8000`和`8001`

首先修改一下`myproject.ini`文件

```
[uwsgi]
socket = :8001
chdir = /my/own/project/location/
module = myproject.wsgi:application
```

注意到文件中端口部分改成了`socket`

然后运行：

```
# uwsgi myproject.ini
```

此时可以在浏览器中输入`your_ip:8000`来访问页面，注意两次使用的端口与配置文件`mysite_nginx.conf`中配置的端口的关系

### 负载均衡

为了提高性能，可以将应用部署在多个服务器上，通过nginx的反向代理使得每次请求指向不同的服务器，达到负载均衡的效果

只需要在`mysite_nginx.conf`中做出如下修改：

```
# 测试中使用了最简单的轮询方式
upstream django {
    server 127.0.0.1:8001;
    server 111.1.1.1:8002; # 在upstream中添加对应服务的IP+PORT
}

server {
    
    location / {
        proxy_pass  http://django;  将uwsgi_pass改成proxy_pass，并加上http前缀
        include     /path/to/your/mysite/uwsgi_params;
    }
}
```

并且启动本地的Django项目时，也要相应的改成http形式

```
[uwsgi]
http = :8001
chdir = /my/own/project/location/
module = myproject.wsgi:application
```

而另一个服务器上111.1.1.1:8002的Django按照上面配置即可

重启nginx，在浏览器输入`your_ip:8000`，并多刷新几次，就可以分别得到几个服务器上的响应
