---
title: Django1.10静态文件不能成功读取解决方案
date: 2017-03-10 00:50:53
tags: [PythonWeb]
---

问题：
初学Django，按照官网配置settings.py中static相关部分后不能成功读取到配置文件。
使用Chrome调试工具查看后发现没有报404错误，而是返回200.
Console  提示信息
`
Resource interpreted as Stylesheet but transferred with MIME type text/html: "http://localhost:8000/bbs/index/static/css/common.css".
`

<!--more-->

**解决方案**
### 1.settings中静态文件配置部分如下
```python


SITE_ROOT = os.path.join(os.path.abspath(os.path.dirname(__file__)),'..')  
STATIC_URL = '/static/'  
STATIC_ROOT = os.path.join(SITE_ROOT,'static')  
STATICFILES_DIRS = (  
    # os.path.join(BASE_DIR,STATIC_URL),  
    ('css',os.path.join(STATIC_ROOT,'css').replace('\\','/') ),  
    ('js',os.path.join(STATIC_ROOT,'js').replace('\\','/') ),  
    ('images',os.path.join(STATIC_ROOT,'images').replace('\\','/') ),  
    ('upload',os.path.join(STATIC_ROOT,'upload').replace('\\','/') ),  
)
 ```
### 2.目录结构
### 3.HTML引用静态文件
```html

<!DOCTYPE html>  
{% load static %}  
{#{% load staticfiles %}#}  
<html lang="en" >  
<head>  
    <meta charset="utf-8">  
    <title>BBS社区</title>  
</head>  
<link rel="stylesheet" href="{% static 'css/common.css' %}" >  
<body>  
    <div class="pg-header">  
        <div class="header-menu">  
            <img src="/static/images/logo.png" href="/bbs/index/" class="digg-logo"></img>  
            <a href="/bbs/index/" class="tb active">全部</a>  
            <a href="/bbs/index/" class="tb">TabA</a>  
            <a href="/bbs/index/" class="tb">TabB</a>  
            <a href="/bbs/index/" class="tb">TabC</a>  
        </div>  
        <div class="header-search"></div>  
    </div>  
    <div class="pg-body"></div>  
    <div class="pg-bottom"></div>  
{#    <img src="{% static 'images/test.png' %}">#}  
</body>  
</html>  

```

### 4.重启server再试试吧，good luck.
