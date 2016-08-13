---
layout:     post
title:      "《猴年猴赛雷,每天100分钟,学习Flask Web 开发》之六"
date:       2016-02-16
author:     "J3n5en"
tags:
    - Flask
    - Python
---

​		今天是计划的第六天。估计今晚没空，早点写完他吧。

## 第六章、大型程序的结构

之前我们的Demo都是单文件的小脚本，的确很方便，但是当程序变得复杂，单文件就不可取了。通常我们都会使用包和模块组织大型程序。

### 1. 项目结构

``` shell
├─Flsky
│  config.py # 储存配置
│  manage.py # 启动程序&其他程序任务
│  requirements.txt # 依赖包
│
├─app # flask 程序
│  │  email.py
│  │  models.py
│  │  __init__.py
│  │
│  ├─main
│  │      errors.py
│  │      forms.py
│  │      views.py
│  │      __init__.py
│  │
│  ├─static
│  └─templates
├─migrations #数据库迁移脚本
├─tests # 单元测试
│      test*.py
│      __init__.py
│
└─venv # python虚拟环境
```

### 2. 配置选项

在实际的开发中我们一般需要几个配置，因为一般开发，测试和生产环境需要使用不同的数据库才不会互相影响。从现在开始我们我们的配置不会再像之前那样用简单的字典结构，而是使用配置类。

``` python
#config.py
import os
basedir = os.path.abspath(os.path.dirname(__file__))

class Config:
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'hard to guess string'
    SQLALCHEMY_COMMIT_ON_TEARDOWN = True
    FLASKY_MAIL_SUBJECT_PREFIX = '[Flasky]'
    FLASKY_MAIL_SENDER = 'Flasky Admin <flasky@example.com>'
    FLASKY_ADMIN = os.environ.get('FLASKY_ADMIN')

    @staticmethod
    def init_app(app):
        pass

class DevelopmentConfig(Config): DEBUG = True
    MAIL_SERVER = 'smtp.googlemail.com'
    MAIL_PORT = 587
    MAIL_USE_TLS = True
    MAIL_USERNAME = os.environ.get('MAIL_USERNAME')
    MAIL_PASSWORD = os.environ.get('MAIL_PASSWORD')
    SQLALCHEMY_DATABASE_URI = os.environ.get('DEV_DATABASE_URL') or \
        'sqlite:///' + os.path.join(basedir, 'data-dev.sqlite')

class TestingConfig(Config):
    TESTING = True
    SQLALCHEMY_DATABASE_URI = os.environ.get('TEST_DATABASE_URL') or \ 'sqlite:///' + os.path.join(basedir, 'data-test.sqlite')

class ProductionConfig(Config):
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
        'sqlite:///' + os.path.join(basedir, 'data.sqlite')

config = {
    'development': DevelopmentConfig,
    'testing': TestingConfig,
    'production': ProductionConfig,
    'default': DevelopmentConfig
}
```

其中基类包含通用配置，子类分别定义专用配置。配置类还可以定义`init_app()` 类方法。其参数是程序实例。在这个方法里面我们可以执行当前环境的配置初始化。在配置脚本末尾的config字典中注册了不同的配置环境。而且还注册了默认的配置。

### 3. 程序包

程序包用来保存程序的所有代码、模板和静态文件。一般我们把这个程序包称为app。templates和static文件夹是app的一部分。

### 4. 使用程序工厂函数

在单文件版本中创建应用程序实例很方便，但是通常会有缺陷。因为应用程序实例在全局作用于下被创建，而实例被创建后是没办法动态修改配置的。 尤其在做单元测试时，因为要跑不同的数据库，所以我们要应用不同的配置。解决办法就是通过使用工厂方法延迟应用程序实例的创建，这样不仅仅是延迟了创建时间还让脚本有创建多个应用程序实例的能力，这对于测试尤其有用。在先买年的例子中在app包中定义了了这样一个工厂方法。app包导入了Flask目前会用到的扩展，但因为应用程序实例还没有被构建出来，它们都还没有被正确初始化。create_app()这个工厂方法接受一个配置名称作为参数，通过使用Flask提供的app.config的from_object()方法，我们就能从config.py中导入所需要的配置。一旦应用程序实例被创建出来，扩展就能够通过调用init_app()来完成初始化。

``` python
# app/__init__.py
from flask import Flask, render_template
from flask.ext.bootstrap import Bootstrap
from flask.ext.mail import Mail
from flask.ext.moment import Moment
from flask.ext.sqlalchemy import SQLAlchemy
from config import config

bootstrap = Bootstrap()
mail = Mail()
moment = Moment()
db = SQLAlchemy()

def create_app(config_name):
    app = Flask(__name__)
    app.config.from_object(config[config_name])
    config[config_name].init_app(app)
    bootstrap.init_app(app)
    mail.init_app(app)
    moment.init_app(app)
    db.init_app(app)
    # 附加路由和自定义错误页面

    return app
```

工厂方法返回的应用程序实例还不完整，因它们没有包含路由和错误处理功能，下一节会介绍如何解决这个问题。

### 4. 在blueprints中实现程序功能

用工厂方法构建应用程序实例会给路由设置带来一些麻烦。单脚本应用中，应用程序实例是全局的，路由能简单地用`@app.route` 来定义。但是现在应用程序实例是运行时创建的，`@app.route` 只在在 `create_app()` 以后才存在，除此之外`@app.errorhandler` 也有同样的问题。Flask提供的解决方案是使用blueprints来解决这个问题。blueprints跟application类似，也能定义路由。不同之处是它的路由都处于休眠状态，直到它被注册到应用程序实例后路由才是它的一部分。blueprint在全局作用域下使用，因此我们完全可以像在单文件中那样使用路由。当然你既能通过单文件也能通过更加组织良好的方式。为了达到最大程度的便利性，一个子包结构被创建用于管理blueprint。下面展示了在这个main包中如何创建blueprint：

``` python
from flask import Blueprint
main = Blueprint('main', __name__)
from . import views, errors
```

blueprints被创建为Blueprint的实例对象，构造函数有两个参数：blueprint的名字和它所在的模块或者包，在这个应用程序中，Python的 `__name__`  变量就是第二个参数所需要的值。

应用程序的路由被存储在app/main/views.py模块中， 错误处理则在app/main/errors.py。导入这些模块以后，路由和错误处理就和blueprint关联起来了。

有一点要注意路由和错误处理模块是在`app/__init__.py` 的底部被导入的，因为views.py 和 errors.py要导入main blueprint，所以为了避免循环依赖我们要等到main被创建出来才能够导入路由和错误处理。

``` python
# app/__init__.py 注册blueprint
def create_app(config_name):
    # ...
    from main import main as main_blueprint
    app.register_blueprint(main_blueprint)
    return app
```

错误处理程序

``` python
#app/main/error.py
from flask import render_template
from . import main

@main.app_errorhandler(404)
def page_not_found(e):
    return render_template('404.html'), 404

@main.app_errorhandler(500)
def internal_server_error(e):
    return render_template('500.html'), 500
```

在blueprint使用错误处理，如果使用@app.errorhandler，只有由blueprint定义的路由中导致的错误才会触发对应的handler，如果想要错误处理对整个应用程序可用，我们需要使用@main.app_errorhandler。

blueprint路由

``` python
# app/main/views.py
from datetime import datetime
from flask import render_template, session, redirect, url_for
from . import main
from .forms import NameForm
from .. import db
from ..models import User

@main.route('/', methods=['GET', 'POST'])
def index():
    form = NameForm()
    if form.validate_on_submit():
        # ...
        return redirect(url_for('.index'))
    return render_template('index.html',
                           form=form, name=session.get('name'),
                           known=session.get('known', False),
                           current_time=datetime.utcnow())
```

在blueprint中使用视图方法跟之前有两个不同的地方。第一个是route是来自blueprint，即-使用@main.route，第二个是url_for()方法的使用。在前面介绍过url_for()的参数默认是视图方法的名称，比如在单脚本应用中index()这个视图方法的URL能够通过url_for('index')获取到。

在blueprints中区别在于所有的作用域都来自于blueprint（作用域就是blueprint的名称，即Blueprint构造函数的第一个参数），因此index()视图方法需要通过main.index来获取到URL，即url_for('main.index')。url_for()方法同样支持参数的更短形式，通过将blueprint名字省略，我们可以简写为url_for('.index')。当然如果跨越不同的blueprints，blueprint的名字还是要加上的。

为了完成应用程序，我们还需要在app/main/forms.py模块导入form相关的一些对象。

### 5. 启动脚本

在根目录中的manage.py文件用于启动程序。

``` python
#!/usr/bin/env python
import os
from app import create_app, db
from app.models import User, Role
from flask.ext.script import Manager, Shell
from flask.ext.migrate import Migrate, MigrateCommand

app = create_app(os.getenv('FLASK_CONFIG') or 'default')
manager = Manager(app)
migrate = Migrate(app, db)

def make_shell_context():
    return dict(app=app, db=db, User=User, Role=Role)

manager.add_command("shell", Shell(make_context=make_shell_context))
manager.add_command('db', MigrateCommand)

if __name__ == '__main__':
    manager.run()
```

该脚本首先创建应用程序实例，然后从系统环境中读取FLASK_CONFIG变量，如果该变量没有定义则使用默认值。然后Flask-Script, Flask-Migrate等扩展的实例都被初始化。

### 6. *Requirements文件*

Applications应该包含一个requirements.txt，它记录了有着准确版本号的所有包依赖，这对以在其他电脑上初始化项目环境很重要。通过如下命令能够自动生成一个项目用到的包的requirement.txt文件：

``` shell
(venv) $ pip freeze >requirements.txt
```

在一个新的环境中，你如果要复制虚拟环境中的安装包，只需要执行如下命令即可：

``` shell
(venv) $ pip install -r requirements.txt
```

### 7. *单元测试*

到目前应用程序还很小，几乎还没有什么要测试的，但我们先来写一个小的测试例子：

``` python
import unittest
from flask import current_app
from app import create_app, db

class BasicsTestCase(unittest.TestCase):

    def setUp(self):
        self.app = create_app('testing')
        self.app_context = self.app.app_context()
        self.app_context.push()
        db.create_all()

    def tearDown(self):
        db.session.remove()
        db.drop_all()
        self.app_context.pop()

    def test_app_exists(self):
        self.assertFalse(current_app is None)

    def test_app_is_testing(self):
        self.assertTrue(current_app.config['TESTING'])
```

测试是按照Python包中的典型的单元测试的写法来构建的，setUp() 和 tearDown() 方法在每个测试方法执行前后都会运行，任何以test_ 开头的方法都会被当做测试方法来执行。setUp()方法创建了测试所需的环境， 他首先创建了应用程序实例用作测试的山下文环境，这样就能确保测试拿到current_app, 然后新建了一个全新的数据库。数据库和应用程序实例最后都会在tearDown() 方法被销毁。

第一个测试确保了应用程序实例是存在的，第二个测试应用程序实例在测试配置下运行。为了确保测试文件夹有正确的包结构，我们需要添加一个`tests/__init__.py` 文件，这样单元测试包就能扫描所有在测试文件夹中的模块了。

为了运行单元测试，我们可以在manage.py中添加一个自定义命令,

``` python
@manager.command
def test():
    """Run the unit tests."""
    import unittest
    tests = unittest.TestLoader().discover('tests')
    unittest.TextTestRunner(verbosity=2).run(tests)
```

下面是运行过程

``` shell
(venv) $ python manage.py test
test_app_exists (test_basics.BasicsTestCase) ... ok
test_app_is_testing (test_basics.BasicsTestCase) ... ok
.----------------------------------------------------------------------
Ran 2 tests in 0.001s
OK
```

### 8. *创建数据库*

重构后的应用程序使用了跟单文件本版本中完全不同的数据库。数据库URL会首先从环境变量中获取，然后把默认的SQLite数据库作为备选，在三个配置环境下数据库的名字是不同的。

不论数据库的URL是什么，只要是转换到一个新的数据库数，据库表一定要被重新创建使用Flask-Migrate进行迁移管理的过程中，数据库表能够通过如下命令被新建或者upgrade：

``` shell
(venv) $ python manage.py db upgrade
```

到这里为止，第一部分Flask简介就结束了，从下一章开始就会开始实例部分‘社交博客程序’。



>   Happy Hacking ！

\# EOF \#