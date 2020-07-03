---
layout: article
tags: Python
title: Python爬虫记录
article_header:
  type: overlay
  theme: dark
  background_color: '#123'
  background_image: false
---

一些Python常用的爬虫方法

<!--more-->

# 基础知识

## 什么是爬虫

爬虫就是能够自动的获取网络中信息的程序或脚本

大部分爬虫程序是通过`模拟浏览器的操作`来实现的，所以使用爬虫就需要对浏览器的工作流程有一定的了解

## 浏览器的工作方式

我们在使用浏览器时，需要在地址框中输入一串网址，然后才能获取到网络页面

这样一系列操作背后的原理可以简单描述为：

- 1.浏览器通过`HTTP`协议向指定`URL`地址的服务器发送`请求`

- 2.服务器根据浏览器的请求信息，找到相应的资源，作为`响应`发送给浏览器

- 3.浏览器接收到响应，经过处理得到页面信息并显示出来，就是我们看到的网页

所以只要我们能够模拟浏览器的行为，能够实现发送请求和接收响应，并从响应信息中提取出所需的内容，就可以实现爬虫的功能了

## 请求头header

前面提到过，服务器需要接收到浏览器发出的请求来返回资源。那么在发送请求的时候就需要告诉服务器，我这一次发给你的是什么样的请求，需要以怎样的形式来返回响应，这样服务器才能准确的返回想要的内容

在HTTP协议中，关于请求的信息是存放在请求头`header`中的

我们可以在Chrome浏览器中按`F12`快捷键来查看headers的相关内容，如图中所示：

<img class="image image--md" src="https://raw.githubusercontent.com/Chunar5354/Chunar5354.github.io/master/_posts/images/headers.png"/>


其中`General`和`Request Headers`都是包含在请求头中的内容（General Headers是共同存在于请求头和响应头中的通用内容）

请求头中有几个比较重要的字段：

- Request URL：表示该请求指向的地址
- Request Method：表示请求方法
- Status Code：状态码，表示服务器响应的状态
- User-Agent：用户代理，表示用户使用的操作系统以及浏览器型号

在使用浏览器访问网址时，请求头是浏览器自动添加的。而在编写爬虫时，需要人为的添加请求头来尽可能地模拟浏览器的行为


# Python爬虫编程

## 爬虫初体验

浏览器是在HTTP协议下运作的，所以爬虫需要有发送HTTP请求的能力，可以通过Python的`requests`来实现

- 首先需要安装

```
# pip install requests
```

看一个简单应用：

```python
import requests

url = https://translate.google.cn/

headers = {'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36',}

# requests.get 使用GET方法
response = requests.get(url, headers=headers)
response.encoding = 'utf-8'
# response是一个Response对象，我们想要的页面信息在response.text里面
data = response.text
print(data)  # data是一个含有页面信息的字符串
```

这就是一个简易的模拟浏览器发送请求并得到响应的过程:

通过GET方法向Google翻译主页发起请求，并设定User-agent，将得到的响应编码后打印出来

此时响应内容`data`中包含大量混乱的字符，需要用适当的方法解析出我们想要的信息

## 解析响应
