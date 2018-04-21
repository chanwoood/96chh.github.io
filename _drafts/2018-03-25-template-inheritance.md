---
layout: post
title:  '模板继承'
categories: Django
author: CHH
---

* content
{:toc}

```
templates
  │ base.html
  │  
  └─app
          index.html
          post.html
```




{% raw %}
## 结构树

index.html 和 post.html 都是继承自 base.html 的。

## block 模板标签

举个栗子能说明一切。

base.html 的内容：

```
内容一
{% block A %}
{% endblock %}
内容二
```

post.html 的内容：

```
{% extends 'base.html' %}
{% block A %}
内容三
[% endblock %}
```

最终得出 post.html 的内容：

```
内容一
内容三
内容二
```
{% endraw %}