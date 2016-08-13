---
layout:     post
title:      "《猴年猴赛雷,每天100分钟,学习Flask Web 开发》之四"
date:       2016-02-14
author:     "J3n5en"
tags:
    - Flask
    - Python
---

​		今天是计划的第四天。ok~继续！

## 第四章、数据库

### 1. 使用Flask-SQLAlchemy管理数据库

这本书里面使用的数据库框架是Flask-SQLAlchmy。

>   依旧使用 `pip` 进行安装，`pip install flask-sqlalchemy`

在Flask-SQLAlchmy中，数据库使用URL指定。

| 数据库引擎        | URL                                      |
| ------------ | ---------------------------------------- |
| MySQL        | mysql://usernam:password@hostname/database |
| Postgres     | posrpresql://usernam:password@hostname/database |
| SQLite(Unix) | sqlite:////数据库的绝对路径                      |
| SQLite(Win)  | sqlite:///c:/绝对路径                        |

使用数据库时，必须把数据库的URL保存到Flask配置对象的`SQLALCHEMY_DATABASE_URL`中。还要注意的一个配置选项是`SQLALCHEMY_COMMIT_ON_TEARDOWN` ，当设定为True时，每次请求结束后，都会自动提交数据库的变动。

``` python
from flask.ext.sqlalchemy import SQLAlchemy

basedir = os.path.abspath(os.path.dirname(__file__))

app = Flask(__name__)
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///' + os.path.join(basedir,'data.sqlite')
app.config['SQLALCHEMY_COMMIT_ON_TEARDOWN'] = True
db = SQLAlchemy(app)
```

db对象就是SQLAlchemy类的实例，表示程序使用的数据库，同时还获得了Flask-SQLAlchemy的所有功能。

### 2. 定义模型

``` python
#hello.py
#定义Role和User的例子
class Role(db.Model):
  __tablename__ = 'roles'
  id = db.Column(db.Integer,primary_key=True)
  name = db.Column(db.String(64),unique=True)
  
  def __repr__(self):
    return '<Role %r>' % self.name

class User(db.Model):
  __tablename__ = 'users'
  id = db.Column(db.Integer, primary_key=True)
  username = db.Column(db.String(64), unique=True, index=True)
  
  def __repr__(self):
    return '<Role %r>' % self.username
```

其中`__tablename__`定义在数据库中使用的表明，没有定义的话Flask将会使用一些默认的表名。

最常用的SQLAlchemy列类型

| 类型名          | Python类型           | 说明                  |
| ------------ | ------------------ | ------------------- |
| Integer      | int                | 普通整数，一般32位          |
| SmallInteger | int                | 取值范围小的整数，一般16位      |
| BigInteger   | int或long           | 不限制精度的整数            |
| Float        | float              | 浮点型                 |
| Numeric      | decimal.Decimal    | 定点数                 |
| String       | str                | 变长字符串               |
| Text         | str                | 变长字符串，对长字符串坐了优化     |
| Unicode      | unicode            | 变长unicode字符串        |
| Unicode      | unicode            | 变长unicode字符串，对长做了优化 |
| Boolean      | bool               | 布尔值                 |
| Date         | datetime.date      | 日期                  |
| Time         | datetime.time      | 时间                  |
| DateTime     | datetime.datetime  | 日期和时间               |
| Interval     | datetime.timedelta | 时间间隔                |
| Enum         | str                | 一组字符串               |
| PickleType   | 任何的Python对象        | 自动使用Pickle序列化       |
| LargeBinary  | str                | 二进制文件               |

最常用的SQLAchemy列选项

| 选项名         | 说明          |
| ----------- | ----------- |
| primary_key | 主键          |
| unique      | 唯一          |
| index       | 创建引索，提升查询效率 |
| nullable    | 能否为空        |
| default     | 定义默认值       |

>   注：Flask-SQLAchemy要求每个模型都要定义主键，这一列经常命名为id。

### 3. 关系

一对多。

``` python
class Role(db.Model):
    # ...
    users = db.relationship('User', backref='role')
class User(db.Model):
    # ...
    role_id = db.Column(db.Integer, db.ForeignKey('roles.id'))
```

### 4. 数据库操作

#### 4.1. 创建表

在Flask-SQLAlchemy中使用`db.create_all()` 创建表。

``` shell
python hello.py shell
>>from hello import db
>>db.create_all()
```

如果数据库已经存在的话则不会重新创建或更新，那么粗暴的解决方法就是先删除，再重建。

``` python
db.drop_all()
db.create_all()
```

#### 4.2. 插入项

如下例子尝试插入users和roles的一些数据行：

``` python
>>> from hello import Role, User
>>> admin_role = Role(name='Admin')
>>> mod_role = Role(name='Moderator')
>>> user_role = Role(name='User')
>>> user_john = User(username='john', role=admin_role)
>>> user_susan = User(username='susan', role=user_role)
>>> user_david = User(username='david', role=user_role)
```

models的构造函数接收属性值作为参数，注意虽然role属性被使用了，但它不是真正的数据库列，它只是一个高层次的one-to-many的relationship的展示。这些role的id都还没有被设置：因为它们是由Flask-SQLAlchemy来维护的，到目前为止它们只是一些Python对象：

``` python
>>> print(admin_role.id)
None
>>> print(mod_role.id)
None
>>> print(user_role.id)
None
```

所有数据库改动都被记录到了数据库提供的session中，这里你可以通过Flask-SQLAlchemy的db.session获取到它，为了把对象写到数据库它们必须先保存到session中：

``` python
>>> db.session.add(admin_role)
>>> db.session.add(mod_role)
>>> db.session.add(user_role)
>>> db.session.add(user_john)
>>> db.session.add(user_susan)
>>> db.session.add(user_david)
```

或者更简单地：

``` python
>>> db.session.add_all([admin_role, mod_role, user_role, user_john, user_susan, user_david])
```

然后你要把所有的数据库改动提交：

``` python
>>> db.session.commit()
```

这时可以检查id属性值：

``` python
>>> print(admin_role.id)
1
>>> print(mod_role.id)
2
>>> print(user_role.id)
3
```

>   数据库也能回滚操作，如果`db.session.rollback()`被调用，所有数据库session中的对象都会恢复到数据库中的状态。

#### 4.3. 修改行

数据库session中的add()方法同样也能被用于更新模型，如下的例子把role从“Admin”重命名为“Administrator”：

``` python
>>> admin_role.name = 'Administrator'
>>> db.session.add(admin_role)
>>> db.session.commit()
```

#### 4.4. 删除行

可以使用session中的delete()方法来删除数据，更其他操作一样，删除也要通过session.commit()才能生效：

``` python
>>> db.session.delete(mod_role)
>>> db.session.commit()
```

#### 4.5 查询行

最基本的model的查询操作是返回整个表格的数据：

``` python
>>> Role.query.all()
[<Role u'Administrator'>, <Role u'User'>]
>>> User.query.all()
[<User u'john'>, <User u'susan'>, <User u'david'>]
```

你还可以通过配置过滤器来限制查询条件：

``` python
 >>> User.query.filter_by(role=user_role).all()
[<User u'susan'>, <User u'david'>]
```

还可以获取到SQLAlchemy生成的原生的查询语句：

``` python
>>> str(User.query.filter_by(role=user_role))
'SELECT users.id AS users_id, users.username AS users_username,
users.role_id AS users_role_id FROM users WHERE :param_1 = users.role_id'
```

如果关闭了shell窗口以后，之前创建的Python对象就都消失了，但是会存在于数据库表中。你可以开一个新的窗口，然后导入model并重建这些对象。如下一个未曾导入就尝试查询名字为“User”的role的例子：

``` python
>>> user_role = Role.query.filter_by(name='User').first()
```

常用的过滤器

| 过滤器         | 说明                 |
| ----------- | ------------------ |
| filter()    | 把过滤器添加到查询上，返回一个新查询 |
| filter_by() | 把等值过滤器添加到原查询上，     |
| limit()     | 使用指定的值限制原查询返回的结果数量 |
| offset()    | 偏移原查询返回的结果         |
| order_by()  | 排序                 |
| group_by()  | 分组                 |

常用查询函数

| 函数             | 说明                      |
| -------------- | ----------------------- |
| all()          | 所有                      |
| first()        | 返回第一个没有则None            |
| first_or_404() | 返回第一个没有则终止请求返回404错误响应   |
| get()          | 返回指定对应主键的值，None         |
| get_or_404()   | 返回指定对应主键的值，404          |
| count()        | 返回总个数                   |
| paginate()     | 返回paginate对象，包含指定范围内的结果 |

#### 4.6. 在视图中操作数据库

其实前面部分的数据库操作可以直接在视图方法中使用

``` python
@app.route('/', methods=['GET', 'POST'])
def index():
    form = NameForm()
    if form.validate_on_submit():
        user = User.query.filter_by(username=form.name.data).first()
        if user is None:
            user = User(username = form.name.data)
            db.session.add(user)
            session['known'] = False
        else:
            session['known'] = True

        session['name'] = form.name.data
        form.name.data = ''

        return redirect(url_for('index'))
    return render_template('index.html',
        form = form, name = session.get('name'), known = session.get('known', False))
```

每次用户提交name到后台应用程序会首先使用 filter_by() 去数据库查询，并且会有一个known变量被传递到前台用于format问候语。对应的模板改动如下，新的模板会使用known变量来新增一条问候语，对于第一次访问和多次访问的用户问候语内容会有不同：

``` jinja2
{% extends "commonBase.html" %}
{% import "bootstrap/wtf.html" as wtf %}

{% block title %}Flasky{% endblock %}

{% block page_content %}
    <div class="page-header">
        <h1>Hello, {% if name %}{{ name }}{% else %}Stranger{% endif %}!</h1>
        {% if not known %}
            <p>Pleased to meet you!</p>
        {% else %}
            <p>Happy to see you again!</p>
        {% endif %}
    </div>
    {{ wtf.quick_form(form) }}
{% endblock %}
```

#### 4.7. 集成Python Shell

在shell中测试数据库操作，我们需要导入数据库实例db和对应的models，每次开一个新的shell都这样做未免显得繁琐了。Flask-Script的shell命令行能够配置成每次自动导入特定对象。为了把一些对象加入shell命令的导入列表，我们要给shell命令注册一个make_context的回调函数，具体如下：

``` python
from flask.ext.script import Shell
def make_shell_context():
    return dict(app=app, db=db, User=User, Role=Role)

manager.add_command("shell", Shell(make_context=make_shell_context))
```

make_shell_context()方法注册了程序，数据库实例和models，所以它们都能自动被导入到shell中了：

``` python
>>> app
<Flask 'hello'>
>>> db
<SQLAlchemy engine='sqlite:///F:\\data.sqlite'>
>>> User
<class '__main__.User'>
```

### 5. 使用Flask-Migrate来做数据库的Migrations

开发进行到一定阶段，你会发现model的结构需要发生改变，相应的数据库表结构也应该要更新。Flask-SQLAlchemy调用create_all()来新建表当且只发生在这些表不存在的时候，因此更新表结构的唯一办法就是先删除旧的表，当让这样不可避免地会把所有存储的数据也一并销毁了。更好的做法是使用数据库迁移框架，就像代码版本控制工具会监控代码改动一样，一个数据库迁移框架能够跟踪数据库表的变化，并且能把新的改动应用到到旧的表中。在这本书中使用的是Flask-Migrate。

安装 `pip install Flask-Migrate`

初始化配置：

``` python
from flask.ext.migrate import Migrate, MigrateCommand
# ...
migrate = Migrate(app, db)
manager.add_command('db', MigrateCommand)
```

#### 5.1. 创建迁移仓库

为了将数据库迁移的命令暴露出来，我们把MigrateCommand类添加到了Flask-Script的manager对象中，在这个例子中，暴露出来的MigrateCommand命令为db。在使用数据库迁移之前，需要首先通过init命令来创建一个迁移的资源库：

``` shell
      python hello.py db init
      Creating directory /home/flask/flasky/migrations...done
      Creating directory /home/flask/flasky/migrations/versions...done
      Generating /home/flask/flasky/migrations/alembic.ini...done
      Generating /home/flask/flasky/migrations/env.py...done
      Generating /home/flask/flasky/migrations/env.pyc...done
      Generating /home/flask/flasky/migrations/README...done
      Generating /home/flask/flasky/migrations/script.py.mako...done
      Please edit configuration/connection/logging settings in
      '/home/flask/flasky/migrations/alembic.ini' before proceeding.
```

init命令创建了迁移的文件夹，所有迁移脚本都会被存储在这个文件件中。

#### 5.2.创建迁移脚本

在Alembic中，数据库迁移是通过migration脚本来完成的，这个脚本有两个方法分别叫做upgrade() 和downgrade()。upgrade()方法会把新的数据库改动作为迁移的一部分，而downgrade则移除最新的改动。通过添加和移除改动，Alembic能够配置数据库到任何历史节点上。

``` shell
(venv) $ python hello.py db migrate -m "initial migration"
   INFO  [alembic.migration] Context impl SQLiteImpl.
   INFO  [alembic.migration] Will assume non-transactional DDL.
   INFO  [alembic.autogenerate] Detected added table 'roles'
   INFO  [alembic.autogenerate] Detected added table 'users'
   INFO  [alembic.autogenerate.compare] Detected added index
   'ix_users_username' on '['username']'
     Generating /home/flask/flasky/migrations/versions/1bc
     594146bb5_initial_migration.py...done
```

#### 5.3. Upgrading数据库

检查并修正迁移脚本之后，我们可以使用db upgrade 来更新数据库了，你可以把data.sqlite删除以后再执行命令，你会发现删除的数据库通过migration命令重又建了。

\# EOF \#
