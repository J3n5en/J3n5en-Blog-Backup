---
layout:     post
title:      "《猴年猴赛雷,每天100分钟,学习Flask Web 开发》之九"
date:       2016-02-29
author:     "J3n5en"
tags:
    - Flask
    - Python
---

## 第九章、用户资料

在这篇博文里面我们即将实现Flasky用户的个人页面。

### 9.1. 资料信息

为了让用户的个人页面更吸引人，我们可以在其中添加一些关于用户的其他信息。（完善user models）

``` python
#app/models.py
class User(UserMixin,db.Model):
    #...
    name = db.Column(db.String(64))
    locations = db.Column(db.String(64))
    about_me = db.Column(db.Text())
    member_since = db.Column(db.DateTime(),default=detetime.utcnow)
    last_seen = db.Column(db.DateTime(),default=detetime.utcnow)
```

`last_seen` 用来保存用户的最后一次登陆时间，为了实现这个功能，我们可以在User models中添加一个方法。

``` python
class User(UserMixin,db.Model):
    #...
	def ping(self): # 实现每次登陆都更新最后登陆时间
        self.last_seen = datetime.utcnow()
        db.session.add(self)
```

每次调用`ping()` 方法都会更新最后登陆时间，还记得`before_app_request` 么？他会在每次请求前运行，所以很适合当前的需求。

``` python
@auth.before_app_request():
  def before_request():
    if current_user.is_authenticated():
        current_user.ping()
        if not current_user.confirmed and request.endpoint[:5] != 'auth':
            return redirect(url_for('auth.unconfirmed'))
```

### 9.2. 用户资料页面

>   Happy Hacking !

\# EOF \#

