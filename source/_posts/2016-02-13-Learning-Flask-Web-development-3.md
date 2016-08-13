---
layout:     post
title:      "《猴年猴赛雷,每天100分钟,学习Flask Web 开发》之三"
date:       2016-02-13
author:     "J3n5en"
tags:
    - Flask
    - Python
---

​		今天是计划的第三天。ok~继续！

## 第三章、Web表单

这里主要讲得是Flask-WTF扩展的使用。

安装: `pip install flask-wtf` 

### 1. 跨站请求伪造(CSRF)保护

为了CSRF保护，Flask-WTF需要程序设置一个密钥。

``` python
#hello.py
app = Flask(__name__)
app.config['SECRET_KEY'] = 'Some string,By J3n5n'
```

>   app.config字典可以用来存储框架、扩展、程序的配置变量。

### 2. 表单类

在Flask-WTF里，每个表单都由一个继承自Form的类表示，这个类定义表单中的一组字段。每个字段都用对象表示。字段对象可以附属一个或多个验证函数。验证函数用来验证用户提交的输入是否符合要求。

``` python
from flask.ext.wtf import Form
from wtforms import StringField, SubmitField
from wtfforms.validators import Required

class NameForm(Form):
  name = StringField(u'请输入您的用户名', validators=[Required()]) # 文本字段 相当于<input>中的 type='text' 。 
  submit = SubmitField(u'提交') # 提交按钮 相当于<input>中的type='submit'
```

`StringField` 构造函数中的可选参数validators指定一个有验证函数组成的列表，在接受用户提交的数据之前验证数据。验证函数Required()确保提交的字段不为空。

WTForms支持的HTML标准字段

| 字段类型                | 说明                     |
| ------------------- | ---------------------- |
| StringField         | 文本字段                   |
| TextAreaField       | 多行文本字段                 |
| PasswordField       | 密码文本字段                 |
| HiddenField         | 隐藏文本字段                 |
| DateField           | 文本字段，datetime.date     |
| DateTimeField       | 文本字段，datetime.datetime |
| IntegerField        | 文本字段，整数                |
| DecimalField        | 文本字段，decimal.Decimal   |
| FloatField          | 文本字段，浮点                |
| BooleanField        | 复选框，True/False         |
| RadioField          | 一组单选框                  |
| SelectMultipleField | 下拉列表，可多选               |
| SelectField         | 下拉列表                   |
| FileField           | 文件上传字段                 |
| submitField         | 提交按钮                   |
| FormField           | 把表单作为一个表单，嵌入另一个表单      |
| FieldList           | 一组指定类型的字段              |

WTForms内建的验证函数

| 验证函数        | 说明            |
| ----------- | ------------- |
| Email       | 验证Email       |
| EqualTo     | 比较两个字段的值      |
| IPAddress   | 验证IPv4网络地址    |
| Length      | 验证长度          |
| NumberRange | 数字范围          |
| Optional    | 无输入值时跳过其他函数   |
| Required    | 确保不为空         |
| Regexp      | 使用正则验证输入值     |
| URL         | 验证URL         |
| AnyOf       | 确保输入值在可选的列表中  |
| NoneOf      | 确保输入值不在可选的列表中 |

### 3. 把表单渲染成HTML

假设视图函数把一个NameForm实例通过参数form传入模板，在模板中可以生成一个简单的表单

``` jinja2
<form method='POST'>
  {{ form.hidden_tag() }}
  {{ form.name.label }}{{ form.name() }}
  {{ form.submit() }}
</form>
```

那我们怎么为字段指定id和class呢?

``` jinja2
<form method='POST'>
  {{ form.name.label }}{{ form.name(id='username') }}
  {{ form.submit() }}
</form>
```

### 4. 在视图函数中处理表单

``` python
#hello.py
@app.route('/',method=['GET', 'POSt'])
def index():
  name = None
  form = NameForm()
  if form.validate_on_submit():
    name = form.name.data
    form.name.data = ''
  return render_template("index.html", form=form, name=name)
```

  `validate_on_submit()` 会验证所有用户提交的数据。符合这返回`True`.

### 5. 重定向和用户会话

``` python
@app.route('/',method=['GET','POST'])
def index():
  form = NameForm()
  if form.validate_on_submit():
    session['name'] = form.name.data
    return redirect(url_for('index'))
  return render_template('index.html',form=form,name=session.get('name'))
```

`session.get('name')` 会从会话中读取name参数的值。当值不存在时返回None而不是返回异常。

### 6. Flash消息

一般请求完成后，需要用户知道状态发生了变化，就像是确认信息，错误信息，警告，提示，之类的。这种功能就是Flah消息的核心特性。

``` python
from flask import Flask,render_template, session, redirect, url_for, flash

@app.route('/')
def index():
  form = NameForm()
  if form.validate_on_submit():
    old_name = session.get('name')
    if old_name is not None and old_name != form.name.date:
      flash(u'修改了用户名')
    session['name'] = form.name.data
    return redirect(url_for('index'))
return render_template('index.html',form=form, name=session.get('name'))
```

仅仅调用Flash并不能把消息显示出来。还需要把`get_flashed_messages()` 函数开放给模板，用来获取渲染消息。

``` jinja2
{% block content %}
<div class='container'>
	{% for msg in get_flashed_messages() %}
    	{{msg}}
        ....
</div>
{% endblock %}
```

>   Happy Hacking ! 

\# EOF \#
