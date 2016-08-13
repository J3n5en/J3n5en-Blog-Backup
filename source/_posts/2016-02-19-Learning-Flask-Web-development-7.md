---
layout:     post
title:      "《猴年猴赛雷,每天100分钟,学习Flask Web 开发》之七"
date:       2016-02-19
author:     "J3n5en"
tags:
    - Flask
    - Python
---

​			这几天有点事做，差点就弃坑不更新了。今天看到一句话：

>   就我们目前的努力程度，远未到拼天赋的地步。

啥都不说了，继续努力，Fighting ！。从这篇博文起我们就进入Flask的实战——社交博客程序。

---

## 第七章、用户认证

在这章里面我们将会为我们的社交博客程序Flasky开发一个完整的认证系统。

### 7.1 Flask的认证扩展

在这里我们将使用多个Python包，并编写胶水代码，将他们良好地协作起来。

-   Flask-Login：管理已经登陆用户的用户会话
-   Werkzeug：计算密码的散列值并进行核对
-   itsdangerous：生成并核对加密安全令牌
-   Flask-Mail：发送与认证相关的Email
-   Flask-Bootstrap：HTML模板
-   Flask-WTF：Web表单

### 7.2 密码安全性

为了保证我们的账号安全，我们必须使用高强的加密，而不是使用明文储存（相信我我没在鄙视某些厂）。

这里我们使用Werkzeug实现密码散列的计算。在Werkzeug中的security模块能够很方便地实现密码散列的计算（只需要两个函数）。

-   ` generate_password_hash(password, method=pbkdf2:sha1, salt_length=8)` 

      这个函数的将原始密码作为输入，以字符串形式输出密码的Hash，输出的值可以保存到用户的数据库中。`method`和`salt_length`的默认值就能满足大多数的需求.

-   ` check_password_hash(hash, password)`:

      这个函数将用户输入的密码值与数据库中的hash值进行对比,若密码正确则返回`True`

``` python
from . import db, login_manager
from werkzeug.security import check_password_hash, generate_password_hash

class User(db.Model):
    password_hash = db.Column(db.String(128))

    @property
    def password(self):
        raise AttributeError('password is not a readable attribute')

    @password.setter
    def password(self, password):
        self.password_hash = generate_password_hash(password)
    def verify_password(self,password):
        return check_password_hash(self.password_hash,password)
```

计算密码的Hash函数通过名为password的只写属性实现。设定这个属性的值时调用`generate_password_hash` 将加密后的值赋值给`password_hash` 字段.当读取password的值时,则返回错误.

### 7.2 创建认证Blueprint

为了方便管理,我们将与用户认证相关的路由定义在auth的blueprint中.

``` python
# app/auth/__init__.py
from flask import Blueprint
auth = Blueprint('auth',__name__)
from . import views
```

在`app/auth/views.py` 模块中引入Blueprint.

``` python
from . import auth
from flask import render_template

@auth.route('/login'):
  def login():
    return render_template('auth/login.html')
```

在`create_app` 中注册blueprint

``` python
# app/__init__.py
def create_app(config_name):
  from .auth import auth as auth_blueprint
  app.register_blueprint(auth, url_prefix="/auth")
  return app

```

### 7.3 使用Flask-Login认证用户

安装 `pip install flask-login` 

想要使用flask-login扩展，程序的User模型必须实现几个方法。

| 方法                 | 说明             |
| ------------------ | -------------- |
| is_authenticated() | 用户已登陆？         |
| is_active()        | 是否允许登陆？        |
| is_anonymous()     | 对普通用户必须返回False |
| get_id()           | 返回用户唯一标识（）     |

这四个方法可以在模型类中作为方法直接实现。或者使用Flask-login内置的UserMixin类，足以满足大多需求。

``` python
from falsk.ext.login import UserMixin
class User(db.Model, UserMixin):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(128), unique=True, index=True)
    email = db.Column(db.String(128), unique=True, index=True)
    role_id = db.Column(db.Integer, db.ForeignKey('roles.id'))
    password_hash = db.Column(db.String(128))
```

在工厂函数中进行初始化。

``` python
# app/__init__.py
from flask.ext.login import LoginManager
login_manager = LoginManager(app)
login_manager.session_protection = 'strong'
login_manager.login_view = 'auth.login'

def create_app(config_name):
  login_manager.init_app(app)
```

session_protection可以设置为None，'basic','strong'，用来提供不同的安全等级防止用户会话遭到篡改。login_view属性设置登陆页面的端点。

最后，flask-login需要一个回调函数，使用标识符加载用户。

``` python
from . import login_manager
class User(db.Model, UserMixin):
  #...
  @login_manager.user_loader
  def load_user(user_id):
    return User.query.get(int(user_id))
```

为了保护路由只让已认证的用户访问，flask-login提供了一个`login_required` 修饰器。

``` python
from flask.ext.login import login_required
@app.route('/secret')
@login_required
def secret():
  return "只允许已登陆用户访问" 
```

接下来我们添加登录的表单，表单里面需要包含一个邮件输入框，密码输入框，“记住我”复选框和提交按钮。我们使用Flask-WTF类

``` python
# app/auth/forms.py
# coding: utf-8

from flask_wtf import Form
from wtforms import StringField, PasswordField, SubmitField, BooleanField
from wtforms.validators import Required,Length,Email

class LoginForm(Form):
    email = StringField('username', validators=[Email(),Length(1,64),Required()])
    password = PasswordField('password', validators=[Required()])
    remember_me = BooleanField('Keep me logged in') 
    submit = SubmitField('login!')
```

登陆模板保存在auth/login.html文件中。

登入用户我们使用视图函数`login()` 实现。

``` python
# app/auth/views.py
from . import auth
from flask import render_template, url_for, redirect, flash
from flask_login import login_user
from app.models import User
from .forms import LoginForm
@auth.route('/login',method=['GET','POST'])
def login():
  form = LoginForm()
  if form.validate_on_submit():
    user = User.query.filter_by(email=form.email.data).first()
    if user is not None and user.verify_password(form.password.data):
      login_user(user,form.remember_me.date) # 登陆用户
      return redirect(request.args.get('next') or url_for('main.index'))
    flash(u'账号或密码错误')
  return render_template('auth/login.html',form=form)
```

登出用户的实现：

``` python
from flask_login import logout_user
@auth.route('/logout')
@login_required
def logout():
  logout_user()
  flash(u"你已安全退出")
  return redirect(url_for('mian.index'))
```

### 7.4 注册新用户

首先我们添加用户注册的表单。

``` python
# coding: utf-8

from flask_wtf import Form
from wtforms import StringField, PasswordField, SubmitField
from wtforms.validators import Required,Regexp,EqualTo,Length,Email
from ..models import User

class RegistrationForm(Form):
    email = StringField('Email',validators=[Required(),Length(1,64),Email()])
    username = StringField('Username',validators=[Required(),Length(1,64),
        Regexp('^[A-Za-z][A-Za-z_.]*$',0,u"用户名只能由数字、字母和下划线组成。")])
    password = PasswordField('password',validators=[Required(),EqualTo('password2',message="两次密码不相同")])
    password2 = PasswordField('comfirm password',validators=[Required()])
    submit = SubmitField('register')
    def validate_email(self,field):
        if User.query.filter_by(email=field.data).first():
            raise ValidationError('Email already registered')
    def validate_username(self,field):
        if User.query.filter_by(username=field.data).first():
            raise ValidationError('Username already in use.')
```

接着我们创建注册用户的视图

``` python
# app/auth/views.py
@auth.route('/register',methods=['GET','POST'])
def register():
    form = RegistrationForm()
    if form.validate_on_submit():
        user = User(
            email = form.email.data,
            username = form.username.data,
            password = form.password.data
            )
        db.session.add(user)
        flash(u"注册成功")
        return redirect(url_for('auth.login'))
    else:
        flash(u"error")
        return render_template('auth/register.html',form=form)
```

### 7.5 确认账户

为了防止恶意注册、及时联系用户等问题，我们一般会需要用户验证邮箱地址。这里我们使用itsdangerous生成确认令牌。我们将生成和校验令牌的功能添加到User模型中。

``` python
# app/models.py
class User(db.Model, UserMixin):
	#...
    confirmed = db.Column(db.Boolean,default=False)

    def generate_confirmation_token(self,expiration=3600):
        s = Serializer(current_app.config['SECRET_KEY'],expiration)
        return s.dumps({'confirm':self.id})
    def confirm(self,token):
        s = Serializer(current_app.config['SECRET_KEY'],)
        try:
            data = s.loads(token)
        except:
            return False
        if data.get('confirm') != self.id:
            return False
        self.confirmed = True
        db.session.add(self)
        return True
```

生成密令以后我们就要发送确认邮件给用户。

``` python
def register():
    	# ...
        db.session.add(user)
        db.session.commit()
        token = user.generate_confirmation_token()
        send_mail(user.email,'confirm your email','auth/mail',user=user,token=token)
```

因为当用户创建好才能获取token所以这里需要`db.session.commit()` 

接下来，我们创建确认用户的视图

``` python
@auth.route('/confirm/<token>')
@login_required
def confirm(token):
    if current_user.confirm:
        return redirect(url_for('main.index'))
    if current_user.confirm(token):
        flash(u"你已经确认了你的邮箱")
    else:
        flash(u"链接错误")
    return redirect(url_for('main.index'))
```

很多时候我们仅仅允许未激活账号的用户去访问激活的页面。我们可以使用Flask的before_requests钩子完成，对于blueprint我们必须使用before_app_requests修饰器，

``` python
# app/auth/views.py
@auth.before_app_request
def before_requests():
    if current_user.is_authenticated() and not current_user.confirmed and request.endpoint[:5] != 'auth.':
        return redirect(url_for('auth.unconfirmed'))

@auth.route('/unconfirmed')
def unconfirmed():
    if current_user.is_anonymous() or current_user.confirmed:
        return redirect(url_for('main.index'))
    return render_template('auth/unconfirmed.html')
```

` request.endpoint[:5] != 'auth.'` 即请求的端点不在auth的buleprint中。

为了防止邮件的丢失，我们还需要重新发送邮件的功能

``` python
@auth.route('/confirm')
@login_required
def resend_confirmation():
    token = current_user.generate_confirmation_token()
    send_mail(current_user.email,'confirm your email','auth/mail',user=current_user,token=token)
    flash(u"邮件已经发送，请查收")
    return redirect(url_for('main.index'))
```

至此我们的社交博客程序的用户认证基本已经实现！


>   Happy Hacking !
