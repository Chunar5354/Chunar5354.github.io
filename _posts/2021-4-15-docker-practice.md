---
layout: article
title: Docker应用中的一些问题记录
tags: Docker
article_header:
  type: overlay
  theme: dark
  background_color: '#123'
  background_image: false
---

Docker实践中遇到的一些问题和解决方法

<!--more-->

### 换源

在自建镜像时，通常需要依赖基础镜像，而基础镜像往往是很纯净的，需要自己安装一些辅助软件

而默认的安装源都是国外的地址，下载速度比较慢，所以可以在Dockerfile中写入换源指令，这样新的镜像就能够使用国内源加速下载

- apt-get换源

```Dockerfile
RUN sed -i "s@http://deb.debian.org@http://mirrors.aliyun.com@g" /etc/apt/sources.list && rm -Rf /var/lib/apt/lists/* && apt-get update
```

实际执行了三个指令：写入新的源，删除旧的源以及更新（update之后才能找到所需要安装的软件包）

上面使用了文本处理命令sed把文件中的`http://deb.debian.org`替换成`http://mirrors.aliyun.com`

- pip换源

除了使用编辑文本的方式，还可以直接在宿主机编写一份配置文件，然后复制到镜像中

在Dockerfile同路径下编写一个`pip.conf`文件：

```conf
[global]
index-url = https://mirrors.aliyun.com/pypi/simple/
[install]
trusted-host=mirrors.aliyun.com
```

并在Dockerfile中添加：

```Dockerfile
COPY pip.conf /root/.pip/pip.conf
```

### mysql自动创建库表

新键的mysql容器中mysql是空的，为了更方便的执行业务，最好是可以在创建时自动把需要的库表数据添加好

只需要在mysql镜像的Dockerfile中添加一句：

```Dockerfile
COPY ./*.sql /docker-entrypoint-initdb.d/
```

即把本地的`.sql`文件都添加到容器的`/docker-entrypoint-initdb.d`路径下，在容器创建时会自动执行这里的sql文件来初始化数据库

### mysql设置初始密码

如果在命令行通过docker run启动，通过`-e`标签来设置，如：

```
$ sudo docker run --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql
```

如果用docker-compose启动，需要在mysql服务的字段下添加一个environment：

```yml
services:
    mysql:
        image: mysql
        container_name: mysql
        ports:
            - "3306:3306"
        environment: 
            MYSQL_ROOT_PASSWORD: "123456"
```

### redis设置密码

本质上是在启动容器后自动运行一条redis-server设置密码的命令

如果在命令行通过docker run启动，需要加上`--requirepass`标签：

```
$ docker run -d --name my_redis -p 6379:6379 redis --requirepass "123456"
```

如果用docker-compose启动，添加一条command：

```yml
services:
    redis:
        image: redis
        container_name: redis
        ports:
            - "6379:6379"
        command: ["redis-server", "--appendonly", "yes", "--requirepass","123456"]
```

### 前台进程

Docker的容器必须要有前台进程在运行，否则就会自动退出，变成Exited状态

如果某个容器的目的是为了运行自己写的程序，并想让程序在后台运行，如果不给一个占用前台进程的命令，容器就会退出

此时可以编写一个脚本，在运行完自己的程序之后，添加一个前台命令，如：

```sh
#!/bin/bash

# 运行其他程序
tail -f /dev/null
```

`tail -f`表示显示某个文件的最后10行，当有新内容添加时会继续显示，这个命令会一直在前台循环

`/dev/null`表示空设备，它没有任何内容，所以这条命令几乎不消耗，但可以一直占用前台进程

### 数据卷挂载的问题

有时在创建镜像的时候已经在Dockerfile中复制文件并指定了工作目录

```Dockerfile
RUN mkdir -p /home/my_project

WORKDIR /home/my_project

ADD . /home/my_project
```

这种情况下如果在运行容器的时候通过`-v`标签指定了数据卷挂载，比如

```
$ docker run --name my_con -v /usr/local/my_project:/home/my_project my_image
```

由于Dockerfile中复制的目标路径和运行时挂载的目标路径相同，可能会产生冲突，导致通过Dockerfile复制的文件`丢失`