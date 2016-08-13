---
layout:     post
title:      "《猴年猴赛雷,每天100分钟,学习Flask Web 开发》之一"
date:       2016-02-11
author:     "J3n5en"
tags:
    - Flask
    - Python
---


​		今天是大年初一,我将在这几天里,每天抽出至少100分钟来学习Flask Web开发，并记录过程。

所读教材:《Flask Web 开发》

使用编辑器：Pycharm。

编程环境：win10 + Python2.7

## 第一章

*   安装

      安装Flask和安装其他Python包一样可以使用pip进行快速的安装`pip install flask`


-   Flask 程序的初始化

    ​	所有的Flask程序都必须创建一个程序实例。Web服务器使用WSGI (Web Server Gateway Interface)协议，把所有的请求都转给Flask程序实例这个对象进行处理。一般使用下列代码创建：

``` python
from flask import Flask
app = Flask( __name__ )
```

-   路由和视图函数

    ​	在Flask中定义路由的最简便方式，是使用实例提供的 `@app.route` 装饰器。把装饰的函数注册为路由。

``` python
@app.route('/')
def index():
	return '<h1>hello World </h1>''
```

这里的 `index()` 用来返回响应，这样的函数被称为视图函数。

除了这种简单的URL，Flask当然也支持动态的URL。

``` python
@app.route('/user/<username>')
def user(username):
  return '<h1> Hello %s ! </h1>' % username
```

-   启动服务器

    ​	实例用 `run` 方法启动Flask集成的开发Web服务器

``` python
if __name__ == '__main__':
  app.run(debug=True)
```

-   一个完整的程序

``` python
#hello.py
# coding:utf-8
from flask import Flask
app = Flask(__name__)

@app.route('/')
def index():
    return '<h1> hello word ! </h1>'

@app.route('/user/<name>')
def user(name):
    return '<h1>hello %s </h1>' % name

if __name__ == '__main__':
    app.run(debug=True)
```

然后使用 `python hello.py ` 启动程序

-   请求上下文

``` python
from flask import request

@app.route('/')
def index():
  user_agent = request.headers.get('User-Agent')
  return '<h1>Your broswer is %s </h1>' % user_agent
```

Flask上下文全局变量

|     变量名     | 上下文   | 说    明                   |
| :---------: | ----- | ------------------------ |
| current_app | 程序上下文 | 当前激活的程序的程序实例             |
|      g      | 程序上下文 | 处理请求时用作临时储存的对象，每次请求都会重设  |
|   request   | 请求上下文 | 请求对象，封装了客户端发出的HTTP请求中的内容 |
|   session   | 请求上下文 | 用户回话，用于存储请求之间需要记住的值的词典   |

-   请求钩子

      1.  before_first_request: 注册一个函数，在处理第一个请求之前运行。
      2.  before_requesr:每次请求之前运行。
      3.  after_request：如果没有未处理的异常，在每次请求之后运行。
      4.  tear_request：在每次请求之后运行。

    ``` python
      @app.before_request
      def before_request():
          pass
    ```

-   响应
除了像之前一样返回字符串以外，我们还可以返回状态码，

    ``` python
      @app.route('/')
      def index():
      	return '<h1> Bad Request </h1>' , 400
    ```

      还可以设置响应的headers (一般不这样做)。

    ``` python
      from falsk import make_response

      @app.route('/')
      def index():
        response = make_response("<h1>set cookies</h1>")
        response.set_cookie('man' , 'j3n5en')
        return response
    ```

      重定向响应 -> redirect()

    ``` python
      from flask import redirect

      @app.route('/')
      def index():
        return redirect("http://j3n5en.com")
    ```

      处理错误的响应：

    ``` python
      from flask import abort
      @app.route('/user/<name>')
      def get_username(name):
        user = load_user(name)
        if not user:
          abort(404)
        return '<h1>hello %s !</h1>' % name
    ```

      这样的话当用户不存在就会抛出404异常。

-   Flask扩展之——Flask-Script

      Flask-Script是一个为Flask程序添加命令行解析器的插件。

      首先用`pip install flask-script` 进行安装。

    ``` python
      # coding:utf-8
      from flask import Flask
      from flask.ext.script import Manager
      app = Flask(__name__)
      manager = Manager(app)
      from flask import request

      @app.route('/')
      def index():
        user_agent = request.headers.get('User-Agent')
        return '<h1>Your broswer is %s </h1>' % user_agent

      if __name__ == '__main__':
          manager.run()
    ```
