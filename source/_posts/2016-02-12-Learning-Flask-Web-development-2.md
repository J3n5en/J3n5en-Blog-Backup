---
layout:     post
title:      "《猴年猴赛雷,每天100分钟,学习Flask Web 开发》之二"
date:       2016-02-12
author:     "J3n5en"
tags:
    - Flask
    - Python
---
​		今天是计划的第二天。ok~继续！

## 第二章、模版

Flask中使用的是Jinja2，当初我在使用Django的时候也是喜欢用这款模版引擎。这里就当是巩固学习把。

### 1. 渲染模板(render_template)

默认情况下，Flask程序会在根目录下的templates子文件夹寻找模版。我们把之前的hello.py中的模板保存到templates文件夹中、分别命名为index.html 和 user.html

``` python
# hello.py
from flask import Flask , render_template
# ...

@app.route('/')
def index():
  return render_template('index.html')

@app.route('/user/<name>')
def user(name):
  renturn render_template('user.html' , name=name)
```

Flask中的render_template把Jinja2模板引擎集成到程序中。render_template函数的第一个参数是模板的文件名。随后的参数都是键值对。

### 2. 变量

在模板中使用 {{ name }}结构表示一个变量。Jinja2 能识别所有类型的变量（甚至列表，字典，对象）。

``` jinja2
<p> 一个来自字典的值 {{ onedict['key'] }} .</p>
<p> 一个来自列表的值 {{ onelist[2] }} .</p>
<p> 一个来自对象的值 {{ oneobj.somemethod() }} .</p>
```

Jinja2 中还内置了一些常用的过滤器。过滤器名称加在变量名后面，中间用竖线( "|" )分隔。例如

``` jinja2
Hello, {{ name | capitalize }}
```

除了这个首字母转换大写以外，Jinja2还内置了其他过滤器如下表。

| 过滤器名       | 说明                 |
| ---------- | ------------------ |
| safe       | 渲染值的时候不进行转义        |
| capitalize | 把值转换成首字母大写其他小写     |
| lower      | 把值转换成小写            |
| upper      | 把值转换成大写            |
| title      | 把值中的每个单词的首字母都转换成大写 |
| trim       | 值的首尾去空格            |
| striptags  | 渲染前把值中的所有HTML标签都去掉 |

### 3. 控制结构

-   条件控制语句

    ``` jinja2
    {% if user %}
    	Hello , {{ user }} ！
    {% else %}    
    	Hello , Stranger !
    {% endif %}
    ```

-   for循环

    ``` jinja2
    <ul>
    	{% for comment in comments %}
        	<li>{{ comment }}</li>
        {% endfor %}
    </ul>
    ```

-   宏

    ``` jinja2
    {% macro render_comment(comment) %}
    	<li>{{ comment }}</li>
    {% endmacro %}

    <ul>
    	{% for comment in comments %}
        	{{ render_template(comment) }}
        {% endfor %}
    </ul>
    ```

    为了重复使用宏，可以将宏保存到一个单独的文件中，需要时在进行导入

    ``` jinja2
    {% import 'macros.html' as macros %}
    <ul>
    	{% for comment in comments %}
        	{{ render_template(comment) }}
        {% endfor %}
    </ul>
    ```

    需要多处利用的代码片段可以使用

    ``` jinja2
    {% include 'comment.html' %}
    ```

    或着使用强大的模板继承。

-   模板继承

    jinja2中的模板继承有点像Python中的类继承。首先创建一个名为`base.html`的基模板

    ``` jinja2
    <html>
    <head>
    	{% block head %}
        <title>{% block title  %}{% endblock %} - J3n5en's blog </title>
        {% endblock %}
    </head>
    <body>
    	{% block body %}
        {% endblock %}
    </body>
    </html>
    ```

    再创建这个基模板的衍生模板：

    ``` jinja2
    {% extends 'base.html' %}
    {% block title %}Home{% endblock %}
    {% block head %}
    {{ super() }}
    	<style>
        	*{margin:0;padding:0}
        </style>
    {% endblock %}
    {% block body %}
    	<h1> Hello , World !</h1>
    {% endblock %}
    ```

### 4. 自定义错误页面

如果用户输入了不可用的路由，那么将会显示一个状态码为404的错误页面。但是默认的404页面奇丑无比。那我们就只能让其丑下去？当然不是。我们可以自定义错误页面。例如：

``` python
@app.errorhandler(404)
def page_not_found(e):
  retuan render_template('404.html'), 404
```

### 5. 链接

在jinja2中链接到其他路由。可以使用url_for()

在

``` python
@app.route('/')
def index():
  pass
```

 中,在模板中使用`url_for('index')`得到的是 `/` ，使用`url_for('index' , _external=True)`的话得到的则是绝对地址,`http://localhost:5000/`

生成动态的地址时，可以将动态部分作为关键字传入。例如 `url_for('user', name='J3n5en',_external=True)`

还能将任何额外参数添加到查询字符串中，例如 `url_for('index', page=2)` 得到的地址是`/?page=2`

### 6. 静态文件

为了处理静态文件，Flask中有一个特殊的路由即 `/static/<filename>` 例如调用 `url_for('static', filename='css/style.css' , _external=True)` 返回的地址是 `http://localhost:5000/static/css/style.css` 。
\# EOF \#
