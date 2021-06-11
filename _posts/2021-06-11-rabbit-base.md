---
layout: article
tags: middleware RabbitMQ
title: RabbitMQ基本配置
article_header:
  type: overlay
  theme: dark
  background_color: '#123'
  background_image: false
---

RabbitMQ消息队列的基本配置以及简单应用

<!--more-->

## 安装

- 通过Docker安装

直接运行官方镜像即可

```
docker run -d --hostname -p 5672:5672 my-rabbit --name some-rabbit rabbitmq:latest
```

## 运行起来

RabbitMQ的默认端口是`5672`，要通过`amqp`协议来连接服务，链接的形式如下

```
amqp://用户名：密码@服务器地址：端口号/vhost
```

所以要想要进行连接，需要新建`user`和`vhost`，并赋予`权限`

- 新建用户

```
$ rabbitmqctl add_user <Username> <Password>
```

- 添加vhost

```
$ rabbitmqctl add_vhost <some_vhost>
```

- 赋予权限

```
rabbitmqctl set_permissions -p <some_vhost> <Username> ".*" ".*" ".*"
```

然后通过地址`amqp://Username:Password@IP:port/some_vhost`就可以连接到rabbitmq服务器

测试一下官方的[Hello World Demo](https://www.rabbitmq.com/tutorials/tutorial-one-python.html)


## 整体架构

如图所示：

[![2fADfO.png](https://z3.ax1x.com/2021/06/11/2fADfO.png)](https://imgtu.com/i/2fADfO)

有以下几个成员：

- Publiser，生产者，推送消息的一方，消息由消息体(`payload`)和标签(`label`)组成

- Broker，可以看作一个rabbitmq服务器节点

- vhost，虚拟主机，多用户使用同一个节点时，通过划分vhost来区分不同的用户，每个vhost有自己的交换机和连接

- Exchange，交换机，将接收到的消息按约定`路由`到一个或多个队列中

- Queue，队列，消息被送到queue中等待消费

- Connection，连接，生产者和消费者都需要与Broker建立`TCP连接`

- Channel，信道，在一个connection中建立的`逻辑连接`，防止频繁创建TCP连接带来的开销

- Consumer，消费者，接收消息的一方，消费者只会得到payload，label会在消息路由过程中被`丢弃`

## 用户管理

### 用户类型

rabbitmq中有以下几种用户类型：

- `none`，新建用户时的默认类型，没有管理功能

- `management`，可以访问management plugin

- `policmaker`，可以设置管理策略和设置相关的vhost

- `monitoring`，在management的基础上，可以管理连接和通道

- `administrator`，具有所有权限

### 用户配置

刚启动一个rabbitmq server时，系统中有一个默认的用户`guest`，密码也是`guest`，权限是administrator

可以通过`list_user`命令查看：

```
$ rabbitmqctl list_users
Listing users ...
user	tags
guest	[administrator]
```

rabbitmq提供了一个图形管理控制台，首先需要启动：

```
$ rabbitmq-plugins enable rabbitmq_management
```

会通过`15672`端口开放一个HTTP API，输入`http://localhost:15672`来访问，使用默认的guest用户可以登录

- 新建用户

通过add_user命令来新建用户

```
$ rabbitmqctl add_user chunar 123456
```

新建用户的tag是none：

```
$ rabbitmqctl list_users
Listing users ...
user	tags
chunar	[]
guest	[administrator]
```

none用户是不能登录管理控制台的

- 修改用户权限

通过`set_user_tags`可以修改用户的权限

```
$ rabbitmqctl set_user_tags chunar management
Setting tags for user "chunar" to [management] ...
```

此时chunar用户可以登录管理控制台，但由于不是administrator，不能够修改设置

- 修改密码

```
$ rabbitmqctl change_password chunar 456789
```

- 删除用户

```
$ rabbitmqctl delete_user 'username'
```

### Virtual Host管理

为了防止频繁的创建用户，rabbitmq在同一个用户下支持创建多个虚拟用户`vhost`

rabbitmq默认有一个属于guest用户的根vhost`/`，查看：

```
$ rabbitmqctl list_vhosts
Listing vhosts ...
name
/
```

- 创建vhost

```
$ rabbitmqctl add_vhost vChunar
```

- 设置vhost权限

```
$ rabbitmqctl set_permissions -p vChunar chunar ".*" ".*" ".*"
```

三个`".*"`分别表示对`所有实体`的`配置、写和读`权限

- 查看权限

查看指定vhost的所有用户权限

```
$ rabbitmqctl list_permissions -p vChunar
Listing permissions for vhost "vChunar" ...
user	configure	write	read
chunar	.	.	.*
```

查看某个用户的权限

```
$ rabbitmqctl list_user_permissions chunar
Listing permissions for user "chunar" ...
vhost	configure	write	read
vChunar	.*	.*	.*
```

- 清除权限

清除vChunar vhost在chunar用户的权限：

```
$ rabbitmqctl clear_permissions -p vChunar chunar
Clearing permissions for user "chunar" in vhost "vChunar" ...
```
