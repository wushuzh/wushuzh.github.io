+++
date = "2018-01-30T21:17:49+08:00"
title = "Flask 教程2"
showonlyimage = false
image = "/img/blog/dev-db-model-in-flask/db.png"
draft = false
weight = 603
tags = ["flask"]
+++

flask 教程: 创建一个类微博原型站点
<!--more-->

## 数据库工作流程

Flask 并不含对任何特定数据库的原生支持，开发者根据自身需求安装相应拓展进行自由选择。

这里选用的 flask 拓展是 Flask-SQLAlchemy —— 作为 python 对象和关系数据库记录的中继层。另一个 flask 拓展是 flask-migrate —— 它可以为开发过程中各种数据定义的变更保驾护航。

安装好拓展后，先在配置模块中添加本地开发使用的 sqlite 数据库的配置信息，然后再为 SQLAlchemy 和 Migrate 创建各自实例 —— 基本上所有 flask 的拓展都使用类似方式完成初始化。

{{< highlight python >}}
# filename: config.py
import os
basedir = os.path.abspath(os.path.dirname(__file__))


class Config(object):
    # get SECRET_KEY from env or default hardcoded value
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
        'sqlite:///' + os.path.join(basedir, 'app.db')
    SQLALCHEMY_TRACK_MODIFICATIONS = False


# filename: app/__init__.py

# import ...
from flask_sqlalchemy import SQLAlchemy
from flask_migrate import Migrate

app = Flask(__name__)
app.config.from_object(Config)
db = SQLAlchemy(app)
migrate = Migrate(app, db)

from app import routes, models

{{< /highlight >}}

新建一个模块名为 models ，首先定义 User :

- 表主键 id 由数据库自动分配;
- 保存用户名，类型字符串，需保证唯一同时做索引;
- 保存用户邮箱，和用户名类似需求，只是长度更长;
- 保存密码的散列，为了安全，要将密码散列为定长字符存储;

{{< highlight python >}}
# filename: app/models.py
from app import db

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(120), index=True, unique=True)
    password_hash = db.Column(db.String(128))

    def __repr__(self):
        return '<User {}>'.format(self.username)

{{< /highlight >}}

所有字段作为 User 类的类变量，本质都是由代表每列各自属性的不同参数的 db.Column 类的实例化对象。

数据库方面分三步走:

- 初始化指令(init) 为数据库的相关脚本准备基本目录结构;
- 迁移指令(migrate) 会结合当前 models 代码和数据库实际情况准备好升级、降级脚本;
- 升降指令(upgrade) 才真正地在库表上执行变更操作;

> 注意 MySQL 和 PostgreSQL 比 sqlite 复杂的地方在于，我们需要在执行 upgrade 之前创建数据库。

{{< highlight console >}}
(venv) [microblog-flask]$ flask db init
  Creating directory /p/microblog-flask/migrations ... done
  Creating directory /p/microblog-flask/migrations/versions ... done
  Generating /p/microblog-flask/migrations/script.py.mako ... done
  Generating /p/microblog-flask/migrations/alembic.ini ... done
  Generating /p/microblog-flask/migrations/README ... done
  Generating /p/microblog-flask/migrations/env.py ... done
  Please edit configuration/connection/logging settings in '/p/microblog-flask/migrations/alembic.ini' before proceeding.

(venv) [microblog-flask]$ flask db migrate -m "users table"
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.autogenerate.compare] Detected added table 'user'
INFO  [alembic.autogenerate.compare] Detected added index 'ix_user_email' on '['email']'
INFO  [alembic.autogenerate.compare] Detected added index 'ix_user_username' on '['username']'
  Generating /p/microblog-flask/migrations/versions/398e0b87f3af_users_table.py ... done

(venv) [microblog-flask]$ flask db upgrade
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> 398e0b87f3af, users table
{{< /highlight >}}

## 模型关系

下面引入的另一个对微博的模型 Post ，除了常规的内部唯一 ID 标识，内容，发布时间，还有一个用于标识发帖用户的外键。

这里的模型关系是一个典型 1 对 n 的关系: 一个用户可以发多个帖子。

- 在原有 User 模型(关系中的1)中，用 `db.relationship` 定义一个虚拟字段 posts: 这样实际访问 (关系中的n) 时会比较方便;
- 参数 backref 则是标识了通过 Post 对象(关系中的n)访问 User 对象时使用的虚拟字段;
- 新建 Post 模型中定义时间戳字段，传入 default 的参数为函数引用，SQLAlchemy 就可以在实例化 Post 时代为调用该函数;
- 最后 Post 模型中的 user_id 字段是一个数据库实际字段，将借用 User 表中 id 字段，同时作为 Post 模型的外键。


{{< highlight python >}}
from app import db
from datetime import datetime


class User(db.Model):
    # previous defined fields
    posts = db.relationship('Post', backref='author', lazy='dynamic')

    # __repr__

class Post(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    body = db.Column(db.String(140))
    timestamp = db.Column(db.DateTime, index=True, default=datetime.utcnow)
    user_id = db.Column(db.Integer, db.ForeignKey('user.id'))

    def __repr__(self):
        return '<Post {}>'.format(self.body)
{{< /highlight >}}

{{< highlight console >}}

(venv) [microblog-flask]$ flask db migrate -m "posts table"
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.autogenerate.compare] Detected added table 'post'
INFO  [alembic.autogenerate.compare] Detected added index 'ix_post_timestamp' on '['timestamp']'
  Generating /p/microblog-flask/migrations/versions/fceea7a95e54_posts_table.py ... done

(venv) [microblog-flask]$ flask db upgrade
INFO  [alembic.runtime.migration] Context impl SQLiteImpl.
INFO  [alembic.runtime.migration] Will assume non-transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade 398e0b87f3af -> fceea7a95e54, posts table

{{< /highlight >}}

## 后端调试

在 python 终端导入数据库和 User、Post 模型。创建 2 个用户和 1 个帖子。
> 注意新建帖子的代码，只需给出反向引用的 author 对象，SQLAlchemy 会自动从中提取 user.id

{{< highlight pycon>}}
>>> from app import db
>>> from app.models import User, Post
>>>
>>> u = User(username='john', email='j@e.c')
>>> db.session.add(u)
>>> db.session.add(User(username='susan', email='s@e.com'))
>>> db.session.commit()
>>>
>>> User.query.all()
[<User john>, <User susan>]
>>> u = User.query.get(1)
>>> u
<User john>
>>> u.email
'j@m.c'
>>>
>>> p = Post(body='first post', author=u)
>>> db.session.add(p)
>>> db.session.commit()
>>> u.posts.all()
[<Post first post>]
>>> p.author.email
'j@m.c'
>>>
>>> users = User.query.all()
>>> for u in users:
...   db.session.delete(u)
...
>>> posts = Post.query.all()
>>> for p in posts:
...   db.session.delete(p)
...
>>> db.session.commit()
>>> User.query.all()
[]
>>> Post.query.all()
[]
>>>
{{< /highlight >}}

flask 提供了另一个核心命令 `shell`，使开发者能进入直接带有预定义导入的类变量方便调试。

{{< highlight python >}}
# filename: microblog.py
from app import app, db
from app.models import User, Post


@app.shell_context_processor
def make_shell_context():
    return {'db': db, 'User': User, 'Post': Post}

{{< /highlight >}}

{{< highlight pycon >}}
(venv) [microblog-flask]$ export FLASK_APP=microblog.py
(venv) [microblog-flask]$ flask shell
Python 3.6.3 (default, Oct 24 2017, 14:48:20)
[GCC 7.2.0] on linux
App: app
Instance: /p/microblog-flask/instance
>>> db
<SQLAlchemy engine=sqlite:////p/microblog-flask/app.db>
>>> User
<class 'app.models.User'>
>>> Post
<class 'app.models.Post'>
>>>
{{< /highlight >}}

参考文档

Miguel Grinberg

> - (2017-12-27) [The Flask Mega-Tutorial Part Ⅳ: Database](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world)

封面图片来自 [Conquering SQL](https://dribbble.com/shots/3045907-Conquering-SQL) <a href="https://dribbble.com/DaytimeAV"><i class="fa fa-dribbble" aria-hidden="true"></i> AV</a>
