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

<img width="800" height="300" src="https://raw.githubusercontent.com/Chunar5354/Chunar5354.github.io/master/_posts/images/headers.png"/>


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

下面介绍几种解析html字符串的常用方法

### XPath

XPath（XML Path Language）是一种寻找XML文件中某个节点（标签）位置的查询语言

XML文件是由若干组封闭的标签组成的，比如下面的html文件

```html
<head>
  <title> XML TEST </title>
</head>
<body>
  <article>
    <div class="logo"><h1>LOGO</h1></div>
    <name>Spider</name>
  <article>
</body>
```

其中被`<>`包含的就是标签，通过XPath可以定位到指定标签的位置，从而获取到标签中的内容，比如上面文件中<name>标签包含的文本"Spider"
  
那么我们怎么才能知道想要的内容包含在那个标签中呢？

同样可以借助Chrome的控制台，按`F12`打开，并选择`Elements`一栏，点击控制台左上角的箭头，然后就可以用鼠标去点击页面上想要爬取的内容，点击后该部分的XML代码会显示在控制台中，如下图所示

<img width="950" height="400" src="https://raw.githubusercontent.com/Chunar5354/Chunar5354.github.io/master/_posts/images/xpath_select.png"/>


在Python中应用XPath，需要借助`lxml`库，安装：

```
# pip install lxml
```

关于XPath的语法，可以查阅w3schools的[XPath Tutoraial](https://www.w3schools.com/xml/xpath_syntax.asp)，写的比较详细

这里直接以一个实际的爬取[当当网小说](http://e.dangdang.com/list-XS2-comment-0-1.html)的实例来进行展示，用到的语法会在注释中说明

```python
import requests
import json
from lxml import etree


class Dang():
    def __init__(self):
        '''
        初始化全局url地址，设置headers
        '''
        self.start_url = "http://e.dangdang.com/list-XS2-comment-0-{}.html"  # 这里设置了一个'{}'是为了后面可以改变页码
        self.headers = {
                    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36"
                    }
        self.page = 0

    def get_url(self, url):
        '''
        获取指定url地址的html字符串
        '''
        response = requests.get(url, headers = self.headers)
        html_str = response.content.decode()
        element = etree.HTML(html_str)
        return element

    def parse(self, element):
        '''
        使用xpath从html字符串中解析出目标内容
        '''
        # // 的意思是选取当前节点下符合条件的全部节点并作为一个 list 对象返回，而且以 '/' 开头表示从根节点开始选取
        element_list = element.xpath("//div[@class='book book_list clearfix']")

        book_list = []
        for book in element_list:
            # '.' 表示从当前节点开始选取，下面这个语句的意思是选取当前节点下class属性为'title'的所有<div>节点中的文本
            title_list = book.xpath(".//div[@class='title']/text()") 
            author_list = book.xpath(".//div[@class='author']/text()")
            href_list = book.xpath(".//a/@href")
            price_list = book.xpath(".//span[@class='now']/text()")

        for i in range(len(title_list)):
            # 将上面每一本书的内容按照（页码，编号，书名，作者，链接，价格）的顺序保存
            if i == 0:
                self.page += 1
            book = [self.page, i+1, title_list[i], author_list[i], href_list[i], price_list[i]]

            book_list.append(book)

        return book_list

    def write_to_file(self, book_list, page):
        '''
        将爬取的内容写入到本地文件
        '''
        with open("dangdang.txt", "a", encoding = "utf-8") as f:
            for book in book_list:
                # 这里使用json.dumps() 将Python对象解析为可写入文件的json字符串
                f.write(json.dumps(book, ensure_ascii = False))
                f.write("\n")
        f.close()
        print("successfully wrote page {}".format(page))

    def run(self):
        num = 1
        while num < 11: # 爬取前10页的内容
            # 1.初始url地址
            url = self.start_url.format(num)
            # 2.获取响应
            element = self.get_url(url)
            # 3.xpath解析
            book_list = self.parse(element)
            # 4.保存数据
            self.write_to_file(book_list, num)
            num +=1

dangdang = Dang()
dangdang.run()
```

运行该程序后会生成一个`dangdang.txt`文件，内容类似下面的格式：

```
[1, 1, "坏小孩（《隐秘的角落》原著小说，秦昊王景春领衔主演）", "紫金陈", "http://e.dangdang.com/products/1900363134.html", "促销价:￥9.99"]
[1, 2, "长夜难明", "紫金陈", "http://e.dangdang.com/products/1900694467.html", "￥9.90"]
[1, 3, "三叉戟", "吕铮", "http://e.dangdang.com/products/1901217746.html", "￥19.99"]
...
```

这里有几点需要注意：

- 1.为了找到需要的内容，可能需要在控制台一层层的点开标签，最好是能够找到有共同特征的标签，比如上面的程序中所有的书名都包含在class属性为'title'的<div>标签中
  
- 2.有时候我们不清楚自己写的XPath语句好不好用，可以在Chrome控制台的`Console`一栏进行测试，方式是输入命令`$x("something")`，如下图所示：

<img width="1000" height="300" src="https://raw.githubusercontent.com/Chunar5354/Chunar5354.github.io/master/_posts/images/xpath_console.png"/>
