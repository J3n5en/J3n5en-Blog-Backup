---
layout:     post
title:      "《猴年猴赛雷,每天100分钟,学习Flask Web 开发》之五"
date:       2016-02-15
author:     "J3n5en"
tags:
    - Flask
    - Python
---

​		今天是计划的第五天。今晚顾着看我是歌手，竟然拖到这么晚。。。

### 第五章、电子邮件

很多时候我们都希望我们的程序能通过Email与用户进行交流。在这里我们利用Flask-Mail来实现

安装 `pip install flask-mail`

#### 1. Flask-Mail配置

Flask-Mail SMTP服务器的配置

| 配置            | 默认值       | 说明                 |
| ------------- | --------- | ------------------ |
| MAIL_SERVER   | localhost | Email服务器的ip地址或者主机名 |
| MAIL_PORT     | 25        | Email服务器端口         |
| MAIL_USE_TLS  | False     | 启用传输层安全协议          |
| MAIL_USE_SSL  | False     | 启用安全套接曾协议          |
| MAIL_USERNAME | None      | Email用户名           |
| MAIL_PASSWORD | None      | Email密码            |

``` python
from flask.ext.mail import Mail
# email setting
app.config['MAIL_SERVER'] = 'mail.xxx.com'
app.config['MAIL_USERNAME'] = 'username'
app.config['MAIL_PASSWORD'] = 'pwd'
mail = Mail(app) # 这个要在email setting后面，当初这个坑了我
```

>   注：在实际生产中我们应该把账号信息等重要信息保存到环境变量中，而不是程序里。

### 2. 在Python Shell中发送Email

``` python
python mail.py shell
>>> from flask.ext.mail import Message
>>> from mail import mail
>>> msg = Message('qweq',sender='xxxx@gmail.com',recipients=['xxx@qq.com'])
>>> msg.body = 'text body'
>>> msg.html = '<b>HTML</b> body'
>>> with app.app_context():
  		mail.send(msg)
```

### 3. 在程序中集成发送Email功能

为了更方便，更灵活地发送右键，我们可以把程序发送Email的通用部分抽出来定义成一个函数，那么以后我们就可以用Jinja2模板渲染邮件正文。

``` python
#mail.py
from flask.ext.mail import Message

app.config['FLASKY_MAIL_SUBJECT_PREFIX'] = '[Flasky]'
app.config['FLASKY_MAIL_SENDER'] = 'Flasky Admin <flasky@example.com>'

def send_email(to, subject, template, **kwargs):
    msg = Message(app.config['FLASKY_MAIL_SUBJECT_PREFIX'] + subject,
                  sender=app.config['FLASKY_MAIL_SENDER'], recipients=[to])
    msg.body = render_template(template + '.txt', **kwargs)
    msg.html = render_template(template + '.html', **kwargs)
    mail.send(msg)
```

这样我们就能够很简单地调用这个函数进行Email的发送，例如：

``` python
app.config['FLASKY_ADMIN'] = os.environ.get('FLASKY_ADMIN')
#...
@app.route('/', methods=['GET', 'POST'])
def index():
    form = NameForm()
    if form.validate_on_submit():
        user = User.query.filter_by(username=form.name.data).first()
        if user is None:
            user = User(username=form.name.data)
            db.session.add(user)
            session['known'] = False
            if app.config['FLASKY_ADMIN']:
                send_email(app.config['FLASKY_ADMIN'], 'New User',
                           'mail/new_user', user=user)
        else:
            session['known'] = True

        session['name'] = form.name.data
        form.name.data = ''

        return redirect(url_for('index'))
    return render_template('index.html', form=form, name=session.get('name'), known=session.get('known', False))
```

### 4. 异步发送Email

在运行上面的例子时，我们可以明显发现发送邮件时，程序被堵塞了，浏览器就像失去了响应一样。为了避免这种情况，我们可以把发送Email的函数移到后台的线程里。

``` python
from threading import Thread

def send_async_email(app, msg):
    with app.app_context():
        mail.send(msg)

def send_email(to, subject, template, **kwargs):
    msg = Message(app.config['FLASKY_MAIL_SUBJECT_PREFIX'] + subject,
                  sender=app.config['FLASKY_MAIL_SENDER'], recipients=[to])
    msg.body = render_template(template + '.txt', **kwargs)
    msg.html = render_template(template + '.html', **kwargs)
    thr = Thread(target=send_async_email, args=[app, msg])
    thr.start()
    return thr
```

>   Happy Hacking ！

\# EOF \#