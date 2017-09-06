---
title: PythonWeb - 学习笔记（一）- Flask 
date: 2017-08-27 15:33:13
tags: [PythonWeb,Flask]
---


本部分基于图书《Flask Web开发》学习总结，后续将继续深入学习扩展知识。

**学习计划**

- **Flask框架学习与实战**
- Celery任务调度框架的深入学习与研究
- 基于WebSocket的网络通讯研究
- 高并发PyWeb框架Tornado

<!--more-->

## Chapter 1 安装

### 1.安装环境

为了防止学习过程中安装的libs干扰了工作环境，建议使用virtualenv或Anaconda来创建虚拟Python环境，与工作环境隔离。

**virtualenv与Anaconda对比**

- Anaconda支持多个python版本，可以创建py2、py3的环境，但是virtualenv只能创建目前系统正在使用的python版本来创建虚拟环境。
- Anaconda非常强大，有GUI操作界面，在学习机器学习时安装依赖非常方便。virtualenv就是单纯的创建了一个虚拟环境，很简单。

**安装virtualenv**

- sudo easy_install virtualenv
- mkdir  ~/py_envs && cd ~/py_envs && virtualenv flask_env
- source flask_env/bin/activate
- deactivate (退出虚拟环境)

**安装Flask**

- pip install flask

## Chapter 2 程序的基本结构

### 2.1 一个完整的程序


```python
from flask import Flask

app = Flask(__name__)


@app.route("/")
def index():
    return "<h1>Home Page222sadasdasd</h1>"


@app.route("/user")
def user():
    return "<h1>User Controller</h1>"


@app.route("/user/<path:name>")
def hello(name):
    return "<h1>Hello %s</h1>" % name


if __name__ == '__main__':
    app.run(debug=True)

```

### 2.2 请求-响应 

#### 2.2.1 上下文

| var_name | context | remark |
| --- | --- | --- |
| current_app | 程序上下文 | 当前激活程序的程序实例 |
| g | 程序上下文 | 处理请求时的临时存储变量 |
| request | 请求上下文 | 请求对象，封装了客户端的HTTP请求中的内容 |
| session | 请求上下文 | 用户会话，用于存储请求之间需要“记住”的内容 |



```python
from flask import current_app

from hello import app

if __name__ == '__main__':
    print current_app.name  # RuntimeError: 没激活程序上下文就调用会出现错误
    app_ctx = app.app_context()
    app_ctx.push()
    print 'current_app.name:',current_app.name
    app_ctx.pop()

```

*OUTPUT*
`current_app.name: hello`


#### 2.2.2 请求调度

    
```python
print 'app.url_map:\n', app.url_map
```

*OUTPUT*


```
Map([<Rule '/user' (HEAD, OPTIONS, GET) -> user>,
 <Rule '/' (HEAD, OPTIONS, GET) -> index>,
 <Rule '/static/<filename>' (HEAD, OPTIONS, GET) -> static>,
 <Rule '/user/<name>' (HEAD, OPTIONS, GET) -> hello>])
```

#### 2.2.3 请求钩子

- before_first_request: 第一个请求之前运行
- before_request: 每次请求之前运行
- after_request: 如果没有未处理的异常抛出，在每次请求后运行
- teardown_request: 即使有未处理的异常抛出，也在每次请求之后运行

#### 2.3.4 响应

`make_response("<h1>Hello %s</h1>" % name, 200)` 等价于  `return "<h1>Hello %s</h1>" % name, 200`
make_response的三个参数：返回字符串、状态码、HEAD

```python
@app.route("/user/<path:name>")
def hello(name):
    print 'cookie:', request.cookies.get("my_cookie")
    response = make_response("<h1>Hello %s</h1>" % name, 200)
    response.set_cookie('my_cookie', '100')
    return response

```

*OUTPUT1*
`cookie: None`

*OUTPUT2*
`cookie: 100`




### 2.3 Flask扩展

Flask被设计为可扩展形式，开发者可以自由选择合适的程序包。下面以Flask-Sript给Flask提供命令行扩展为例。

- `pip install flask-script` 安装
- 添加命令行解析功能


```python
from flask import Flask, make_response, request
from flask_script import Manager

app = Flask(__name__)
manager = Manager(app)

......

if __name__ == '__main__':
    # app.run(debug=True)
    manager.run()
```

- 执行main函数看到命令行提示


```
/Users/lixiwei-mac/app/anaconda/envs/py27/bin/python2.7 /Users/lixiwei-mac/Documents/IdeaProjects/flask-learning/hello.py
usage: hello.py [-?] {shell,runserver} ...

positional arguments:
  {shell,runserver}
    shell            Runs a Python shell inside Flask application context.
    runserver        Runs the Flask development server i.e. app.run()

optional arguments:
  -?, --help         show this help message and exit

```

- 在命令行执行 `python hello.py runserver` 启动flask应用(app.run())


```
(py27) NikoBelic@bogon:~/Documents/IdeaProjects/flask-learning$ python hello.py runserver
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
before_request
cookie: 100
127.0.0.1 - - [27/Aug/2017 19:01:45] "GET /user/Niko HTTP/1.1" 404 -

```

- runserver 的参数


```
(py27) NikoBelic@bogon:~/Documents/IdeaProjects/flask-learning$ python hello.py runserver --help
usage: hello.py runserver [-?] [-h HOST] [-p PORT] [--threaded]
                          [--processes PROCESSES] [--passthrough-errors] [-d]
                          [-D] [-r] [-R]

Runs the Flask development server i.e. app.run()

optional arguments:
  -?, --help            show this help message and exit
  -h HOST, --host HOST
  -p PORT, --port PORT
  --threaded
  --processes PROCESSES
  --passthrough-errors
  -d, --debug           enable the Werkzeug debugger (DO NOT use in production
                        code)
  -D, --no-debug        disable the Werkzeug debugger
  -r, --reload          monitor Python files for changes (not 100{'const':
                        True, 'help': 'monitor Python files for changes (not
                        100% safe for production use)', 'option_strings':
                        ['-r', '--reload'], 'dest': 'use_reloader',
                        'required': False, 'nargs': 0, 'choices': None,
                        'default': None, 'prog': 'hello.py runserver',
                        'container': <argparse._ArgumentGroup object at
                        0x1027bb510>, 'type': None, 'metavar': None}afe for
                        production use)
  -R, --no-reload       do not monitor Python files for changes

```

- 其中 --host 非常有用，可以指定当前web服务只可以被指定的IP来源进行访问。
    
    - --host 127.0.0.1 web服务只接受来自本机的请求
    - --host 0.0.0.0 web服务可以接收同局域网中的其他PC的请求

    

## Chapter 3 模板

Flask内置Jinjia模板引擎，用于渲染视图，将数据与展示分离。
[jinjia2官方文档](http://jinja.pocoo.org/docs/2.9/)
[jinjia2中文翻译文档](http://docs.jinkan.org/docs/jinja2/)

### 3.1 Jinjia2模板引擎

**渲染**

- 默认情况下，flask会在templates文件夹中寻找模板
- 创建templates文件夹
- 创建user.html


```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Flask-Learning</title>
</head>
<body>
    <h1>Hello {{name}}</h1>
</body>
</html>
```

- 后台接口代码


```python
@app.route("/user/<path:name>")
def hello(name):
    print 'cookie:', request.cookies.get("my_cookie")
    return render_template('user.html', name=name)
```

- 请求该接口，返回渲染后的HTML内容


**变量**

- jinjia能识别所有类型的变量，例如 字典、列表、对象等。


```html
<p>a value from dictionary {{mydict['key']}}</p>
<p>a value from list {{mylist}}</p>
<p>a value from list with a variable index {{mylist[myindex]}}</p>
<p>a value from an object's method {{myobj.dosth() | upper}}</p>
```

    
```python
@app.route("/")
def index():
    mylist = [12, 3, 4, 5, 65]
    myindex = 2
    mydict = {'key': 'Fuck'}

    class myObj():
        def __init__(self):
            pass

        def dosth(self):
            return "Doing Something."

    myobj = myObj()
    return render_template('index.html', mylist=mylist, myindex=myindex, mydict=mydict, myobj=myobj)
```

![](15038425125552.jpg)

- 可以使用过滤器修改变量
·
    - safe 渲染时值不转义（不要在不可信的值上使用safe过滤器，例如用户表单锁输入的内容）
    - capitalize 手写字母转换成大写，其他小写
    - lower 转小写
    - upper 转大写
    - title 每个单词首字母大写
    - trim 去除首尾空格
    - striptags 渲染之前把值中所有的HTML标签都删掉


**控制结构**



```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Flask-Learning</title>
</head>
<body>
<!--If-Else条件判断-->
<h1>
    {% if name%}
    Hello {{name}}
    {% else %}
    Hello Stranger
    {%endif%}
</h1>

<!--宏定义，类似于python中的函数-->
{% macro render_comment(comment)%}
<li>{{comment}}</li>
{%endmacro%}

<!--For循环遍历-->
<ul>
    {% for comment in comments%}
    {{render_comment(comment)}}
    {% endfor%}
</ul>
</body>
</html>
```


```python
@app.route("/user/<path:name>")
def hello(name):
    comments = ['Hello', 'That is ridiculous']
    return render_template('user.html', name=name,comments=comments)
```
    
**导入模板**

*macros.html*


```html
<!--宏定义，类似于python中的函数-->
{% macro render_comment(comment)%}
<li>{{comment}}</li>
{%endmacro%}
```

*user.html*

```html
<!--导入模板-->
{%import 'macros.html' as macros%}
<ul>
    {% for comment in comments%}
    {{render_comment(comment)}}
    {% endfor%}
</ul>

```


**继承**

*base.html*


```html
<html lang="en">
<head>
    <meta charset="UTF-8">
    {% block head%}
    <title>{%block title%}{%endblock%} - My Application</title>
    {% endblock%}
</head>
<body>
{%block body%}
{%endblock%}

<p>a value from dictionary {{mydict['key']}}</p>
<p>a value from list {{mylist}}</p>
<p>a value from list with a variable index {{mylist[myindex]}}</p>
<p>a value from an object's method {{myobj.dosth() | upper}}</p>
</body>
</html>
```

*index.html*


```html
{% extends 'base.html'%}
{%block title %} Index {%endblock%}
{%block head%}
{{super()}}
<style></style>
{%endblock%}

{%block body%}
<h1>This is home page</h1>
{%endblock%}
```

![](15038933360689.jpg)



### 3.2 集成Flask-Bootstrap

Bootstrap的基模板(bootstrap/base.html)提供了一个网页框架，引入了Bootstrap中所有的CSS和JavaScript文件。

```python
from flask_bootstrap import Bootstrap
bootstrap = Bootstrap(app)
```
 

*base.html*

```html
{% extends "bootstrap/base.html" %}

{% block title %}Flasky{% endblock %}

{% block navbar %}
<div class="navbar navbar-inverse" role="navigation">
    <div class="container">
        <div class="navbar-header">
            <button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse">
                <span class="sr-only">Toggle navigation</span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
            </button>
            <a class="navbar-brand" href="/">Flasky</a>
        </div>
        <div class="navbar-collapse collapse">
            <ul class="nav navbar-nav">
                <li><a href="/">Home</a></li>
            </ul>
        </div>
    </div>
</div>
{% endblock %}

{% block content %}
<div class="container">
    {% block page_content %}{% endblock %}
</div>
{% endblock %}

```

*user.html*


```html
{% extends "base.html" %}

{% block title %}Flasky{% endblock %}

{% block page_content %}
<div class="page-header">
    <h1>Hello, {{ name }}!</h1>
</div>
{% endblock %}

```

![](15038947591463.jpg)


### 3.3 自定义错误页面



```python
@app.errorhandler(404)
def page_not_found(e):
    return render_template('404.html', e=e), 404


@app.errorhandler(500)
def internal_server_error(e):
    return render_template('500.html', e=e), 500
```

*404.html*

```html
{% extends 'base.html'%}

{% block page_content%}
<h3>{{e}}</h3>
{%endblock%}
```

![](15038955457613.jpg)


### 3.4 链接

在模板中使用url_for来创建链接地址，而不是写死

*base.html*


```html

{% block content %}
<div class="container">
    {% block page_content %}{% endblock %}
</div>


<div class="container">
    <h2>
        {% block url%}
        {{url_for('index')}}
        {% endblock %}
    </h2>
</div>


{% endblock %}
```

注意上面有坑: {% block content%} 是页面自定义内容，它对应的{% endblock %} 一定要放到所有需要展示内容的最后，否则不会被页面读取到。

*user.html*


```html
{% block url %}
    {{url_for('user',_external=True,name=name)}}
{% endblock %}
```

_external表示使用相对/绝对路径，一般使用相对路径。

### 3.5 静态文件

Jinjia默认情况下会到static文件夹中搜索静态文件
`<img src="{{url_for('static',filename='head.jpeg')}}"/>`


## Chapter 4 Web表单


**知识点：**

- 跨站请求伪造保护（CSRF）
- 继承Form表单的用户自定义表单类
    - 标准字段（XXXField）
    - 验证函数
- 把表单渲染成HTML 
    - `{{form.name()}}`
    - bootstrap的辅助函数  wtf.quick_form()
- 在视图函数中处理表单 form.name.data
- POST重定向GET模式（防止刷新浏览器出现重复踢脚表单提示）
    - redirect的使用
    - url_for('fun_name') 的参数fun_name不是路由地址而是试图函数的名称
- Flash消息
    - 试图函数中 使用flash(msg)创建消息
    - 模板中使用get_flashed_messages()获取消息列表


```python
app.config['SECRET_KEY'] = 'hard to guess string'


class NameForm(Form):
    name = StringField('what is your name?', validators=[Required()])
    submit = SubmitField('Submit')
    
    

@app.route("/user/<path:name>", methods=['GET', 'POST'])
def hello(name):
    print 'cookie:', request.cookies.get("my_cookie")
    response = make_response("<h1>Hello %s</h1>" % name, 404)
    response.set_cookie('my_cookie', '100')
    comments = ['Hello', 'That is ridiculous']

    name = None
    form = NameForm()
    if form.validate_on_submit():
        name = form.name.data
        if session.get('name') != name:
            flash('You have changed your name!')
        session['name'] = name

        form.name.data = ''
        return redirect(url_for('hello', name=name))
    print 'name', name
    return render_template('user.html', name=session.get('name'), comments=comments, form=form)
```

*user.html*


```html
<div>
    {% import 'bootstrap/wtf.html' as wtf%}
    {% for msg in get_flashed_messages()%}
    <div class="alert alert-warning">
        <button type="button" class="close" data-dismiss="alter"> &times;</button>
        {{msg}}
    </div>
    {% endfor %}
    {{wtf.quick_form(form)}}
    <!--    <form method="POST">
            {{form.hidden_tag()}}
            {{form.name.label}} {{form.name()}}
            {{form.submit()}}
        </form>-->
</div>
```


## Chapter 5 数据库



