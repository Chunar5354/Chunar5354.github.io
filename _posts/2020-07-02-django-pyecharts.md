---
layout: article
tags: Python Django Web
title: 在Django中嵌入图表
article_header:
  type: overlay
  theme: dark
  background_color: '#123'
  background_image: false
---

在Django的网页中嵌入图表，主要通过Python的pyecharts库来实现

<!--more-->

## pyecharts

- [官方文档](https://pyecharts.org/)

- 安装：

```
# pip install pyecharts
```

pyecharts提供了很好的图表嵌入功能，具体的使用方法根据官方文档的介绍很容易就能实现

## Django前后端分离

### 什么是前后端分离

网上的说法一大堆，在我的理解前后端分离就是为了使开发专一化，将前端和后端的代码分离开来，中间用统一的接口（通常是JSON数据）连接，
这样可以使前端和后端的开发者专注自己分块的工作。而在保留接口一致的情况下单独的修改一端不会对另一端产生影响。

### Django中的前后端分离

需要借助一个扩展库`rest_framework`

- 安装
```
# pip install djangorestframework
```

对于使用了 rest_framework 功能的项目，需要在`settings.py`中注册app的位置添加 rest_framework，比如：
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'demo',     #app 名称
    'rest_framework',     # 添加rest_framework
]
```

Django中前后端分离的用法就相当于用前端的html文件去调用后端的python文件（通常是views.py），比如pyecharts的
一个[官方示例](https://pyecharts.org/#/zh-cn/web_django?id=django-%e5%89%8d%e5%90%8e%e7%ab%af%e5%88%86%e7%a6%bb)

前端的html文件：

```html
<--! templates/index.html -->
<script>
    var chart = echarts.init(document.getElementById('bar'), 'white', {renderer: 'canvas'});

    $(
        function () {
            fetchData(chart);
        }
    );

    function fetchData() {
        $.ajax({
            type: "GET",
            url: "http://127.0.0.1:8000/demo/bar",  // 此处的url对应views.py中的一个函数
            // 实际使用的时候http链接的方式通常不能正确的访问到views.py中的函数，最好是写成相对地址的形式，比如 url: "../bar"
            dataType: 'json',
            success: function (result) {
                chart.setOption(result.data);
            }
        });
    }
</script>
```

后端的views.py文件：

```python
# demo/views.py

# ChartView在urls.py中注册，前端的html调用的就是这个类(/demo/bar)，而这个类又会调用bar_base来渲染一个图表
class ChartView(APIView):
    def get(self, request, *args, **kwargs):
        return JsonResponse(json.loads(bar_base()))

# IndexView也在urls.py中注册，通过在浏览器输入(http://localhost:8000/demo/index)来访问，他又会读取前端的html页面，最后跳到上面的ChartView类
class IndexView(APIView):
    def get(self, request, *args, **kwargs):
        return HttpResponse(content=open("./templates/index.html").read())

# 创建图表的函数
def bar_base() -> Bar:
    c = (
        Bar()
        .add_xaxis(["衬衫", "羊毛衫", "雪纺衫", "裤子", "高跟鞋", "袜子"])
        .add_yaxis("商家A", [randrange(0, 100) for _ in range(6)])
        .add_yaxis("商家B", [randrange(0, 100) for _ in range(6)])
        .set_global_opts(title_opts=opts.TitleOpts(title="Bar-基本示例", subtitle="我是副标题"))
        .dump_options_with_quotes()
    )
    return c
```

urls.py中注册views.py中的两个类：
```python
# demo/urls.py

from django.conf.urls import url
from . import views

urlpatterns = [
    url(r'^bar/$', views.ChartView.as_view(), name='demo'),
    url(r'^index/$', views.IndexView.as_view(), name='demo'),
]
```

这样整个流程就是：
- 1.在浏览器输入url访问views.py中的IndexView
- 2.IndexView读取index.html文件
- 3.在index.html中通过Ajax调用views.py中的ChartView

以上在views.py和index.html两个文件之间的接口是JSON数据（index.html读取ChartView的时候）以及html文件（IndexView读取index.html的时候），
对于前后端修改的时候只要保持这两个接口不变，单独的修改就不会影响到另一端


### 前后端互相传递数据

像前面所用到的一样，通过Ajax(Asynchronous JavaScript and XML)，可以实现前后端的异步数据交互

以上面的代码为例：

```html
<--! templates/index.html -->
<script>
    var chart = echarts.init(document.getElementById('bar'), 'white', {renderer: 'canvas'});

    $(
        function () {
            fetchData(chart);
        }
    );

    function fetchData() {
        $.ajax({
            type: "GET",
            url: "../bar",
            dataType: 'json',
            success: function (result) {
                chart.setOption(result.data);
            }
        });
    }
</script>
```

在`fetchData()`中，使用了Ajax向`../bar`指向的地址通过GET方法来获取数据

而该地址最终指向的是views.py中的`ChartView`函数，该函数将图表数据以json的格式通过GET方法发送给前端

```python
class ChartView(APIView):
    def get(self, request, *args, **kwargs):
        return JsonResponse(json.loads(bar_base()))
```

同样的，前端也可以通过Ajax向后端发送数据：

```javascript
function fetchData() {
    var data_send = "test";
    $.ajax({
        type: "GET",
        url: "../bar",
        dataType: 'json',
        data: {test: data_send},  // 指定发送的数据
        success: function (result) {
            chart.setOption(result.data);
        }
    });
 }
```

然后在views.py中把前端传过来的数据打印出来

```python
class ChartView(APIView):
    def get(self, request, *args, **kwargs):
        print(request.GET)
        return JsonResponse(json.loads(bar_base()))
```

这样页面运行时就能在控制台打印出`{"test": "test"}`字样

这里除了使用GET，也可以使用POST方法

- 题外话：

在html中嵌入的js代码，定义了一个函数之后，要通过：

```javascript
$(
    function () {
        fetchData(chart);
    }
);
```

这样的方式来使用
