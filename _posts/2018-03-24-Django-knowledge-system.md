---
layout: post
title:  'Django 知识体系'
categories: Django
author: CHH
---

* content
{:toc}

Django 学习入门，介绍 Django 的 MVT 模式背后的具体机理以及一些内置的应用。

![Django 一般流程](https://upload-images.jianshu.io/upload_images/5690299-a49c8701fd177603.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




{% raw %}
## 一般流程

1. 用户输入 **URL**。
2. 基于正则的 URL 分发器会匹配相应的**视图**。
3. 视图去**模型**那取数据。
4. 视图对数据进行**处理**。
5. 视图把数据传给**模板**。
6. 模板渲染出网页。

## 二、基本配置

**1. 创建**

- 创建项目：django-admin startproject projectname
- 创建应用：python manage.py startapp appname

**2. 项目目录**

```
projectname
 │  manage.py    // 管理项目
 │  
 ├─appname      // 一个 APP，也是一个包
 │  │  admin.py    // 在后台管理模型
 │  │  apps.py      // 配置 APP
 │  │  models.py  // 模型
 │  │  tests.py
 │  │  views.py    //视图
 │  │  __init__.py
 │  │  
 │  └─migrations    // 与数据库迁移有关
 │          __init__.py
 │          
 └─projectname  // 与项目名相同的 APP
       settings.py    // 整个项目的配置
       urls.py          // 整个项目的路由配置
       wsgi.py
       __init__.py
```

**3. 静态文件**

- 在项目根目录下创建 static 目录。
- 在 settings.py 下添加 STATICFILES_DIRS。

```py
# settings.py

STATIC_URL = '/static/'   # 默认已添加，使用静态文件时的前缀
STATICFILES_DIRS = [      # 静态文件搜索路径
        os.path.join(BASE_DIR,'static'),  
    ]
```

```html
<!-- static 模板标签位于 django.contrib.staticfiles 应用下的 staticfiles 模块 -->
<!-- 且 staticfiles 应用已默认注册了 -->
{% load staticfiles %}
<script src="{% static 'app/jquery-1.12.4.js' %}"></script>
```

**4. 模板文件**

- 在项目根目录下创建 templates 目录。
- 在 settings.py 中添加到   TEMPLATES 中的 DIRS 。

## 三、路由

- 每个 URL 都应以 / 结尾。

- **直接**捕获 URL 中的值，用括号括起来，当做**位置参数**传入视图。
`url(r'^(\d{4}/$)', views.index)` 对应的视图函数 `index(request, year)`

- 用**正则分组**来匹配 URL，类似**默认参数**传递给视图（参数名必须和组名相同）。
`url(r'^(?P<year>\d{4}/$)', views.index)` 对应的视图函数是 `index(request, year)`。

- 对某模式 URL 把**字典参数**传给视图，类似**默认参数**（参数名必须与键名相同）。
`url(r'^(?P<year>\d{4})/$', views.index, {'foo': 'bar'})`对应 `index(request, year, foo)`。

- **二级路由**：在项目路由中添加 `path('app/', include('app.urls'))`。

- 模板标签`url`可以解析视图得到 url 模式，并传入值构造出 url。

```py
url(r'^archives/(?P<month>\d{4})/(?P<day>\d+)/$', views.archives, name='archives')
```

```html
{% for date in date_list %}
<form action="{% url 'app:archives' date.month date.day %}" method="post">
```

- 通过调用自身方法构造 URL。

```html
<!-- 模板中代码 -->
href="{{ post.make_post_url }}"
```

```py
# Post 模型中代码
def make_post_url(self):
      return reverse('app:post', kwargs={'pk': self.pk}

# app 中的 urls.py
app_name = app
...
url(r'^post/(?P<pk>\d+)/$', views.post, name='post')
```

## 四、视图

HTTP 请求中有两个核心对象：HttpRequest、HttpResponse

### 1. HttpRequest 对象

就是视图函数的第一个参数 request，有许多属性。

```py
# views.py
def index(request):
  fmt = 'path: {}---method: {}---COOKIES: {}'
  return HttpResponse(fmt.format(request.path, request.method, request.COOKIES))
```

此时页面显示：path: /app/---method: GET---COOKIES: {}


### 2. HttpResponse 对象

视图函数的返回值必须是一个 HttpResponse 对象。

### 3. render 函数

该函数返回的就是一个 HttpResponse 对象。
两个位置参数：request，template_name。
一个常用的默认参数：context=None。

## 五、模型

内容有点多，故另起一文。

详见：[https://www.jianshu.com/p/7c514a9c348d](https://www.jianshu.com/p/7c514a9c348d)

## 六、模板

### 1. 模板的执行

从视图中的 context 获取数据，传给模板。

```py
# views.py
from django.shortcuts import render

def index(request):
	context = {'title': 'Hello', 'welcome': 'Django'}
	return render(request, 'index.html', context=context)
```

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{{ title }}</title>
</head>
<body>
<h1>{{ welcome }}</h1>
</body>
</html>
```

![效果图](https://upload-images.jianshu.io/upload_images/5690299-2370057c9ac4505b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 2. 模板语言

- 除了 item 用 {{  }}，其他含 Python 关键字的用 {%  %} 括起来。
- 字典数据类型的取值是通过 dict.xxx，而不是 dict[xxx] 。
- 每个实例都会自动添加 pk 属性，为主键值。

### 3. 自定义标签

为了在模板语言中做一些复杂操作。有两种：simple_tag、filter 。

simple_tag：任意传递参数，但是不能用作布尔判断。

filter：最多只能传递二个参数，可以用作布尔判断。

具体步骤如下：

a. 在已注册的 app 下创建一个 templatetags 包。具体目录如下：

```
app/      # app 应用
    __init__.py
    models.py
    templatetags/    # templatetags 包
        __init__.py
        mytag.py    # 自定义标签模块，相应的用法：{% load mytag %}
    views.py
```

b. 需要有一个叫 register 的全局变量使标签生效。

```py
from django import template

register = template.Library()
```

c. 注册自定义标签

```py
@register.simple_tag
def myadd(a, b, c):
    return  a + b + c
```

d. 使用标签，先 load 再使用。

```html
{% load mytag %}
...
{% myadd 3 4 5 %}   <!-- 参数之间用空格隔开，而不是逗号。 -->
```

e. 如果函数返回的是一个序列，用 as 把值存起来，再 for 循环。

```html
{% recent_post as posts %}
<ul>{% for post in posts %}
    <li>
        <a href="#">{{post.title}}</a>
    </li>{% endfor %}
</ul>
```

{% endraw %}

---

参考文档：

[https://www.cnblogs.com/LiCheng-/p/6920900.html](https://www.cnblogs.com/LiCheng-/p/6920900.html)

[https://docs.djangoproject.com/en/2.0/howto/custom-template-tags/#custom-template-tags-and-filters
](https://docs.djangoproject.com/en/2.0/howto/custom-template-tags/#custom-template-tags-and-filters
)



