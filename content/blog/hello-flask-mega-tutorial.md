+++
date = "2018-01-29T22:07:11+08:00"
title = "Flask 教程1"
showonlyimage = false
image = "/img/blog/hello-flask-mega-tutorial/list.png"
topImage = "/img/blog/hello-flask-mega-tutorial/list.gif"
draft = false
weight = 603
tags = ["flask"]
+++

flask 教程: 创建一个类微博原型站点
<!--more-->

## Hello Flask

python3 内置了 venv 模块，先建立虚拟环境，安装 flask 。

{{< highlight console >}}
$ cd microblog-flask/
[microblog-flask]$ python -m venv venv
[microblog-flask]$ source venv/bin/activate
(venv) [microblog-flask]$ pip install flask
(venv) [microblog-flask]$ tree -L 2
.
├── app
│   ├── __init__.py
│   └── routes.py
├── microblog.py
└── venv
{{< /highlight >}}

最终站点封装于一个名为 app 的包中。

- `__init__.py` 定义了包对外暴露的所有符号，在导入时执行，内部用`__name__` (预定义变量代表模块名)创建了 flask 实例 ;
- 之后导入的 routes.py 中含有 url 和关联的 view 函数，注意它不能先于实例 app 存在;
- 装饰器 @app.route 的作用是为 ROOT URL 注册其回调函数;
- flask 命令 run 通过环境变量 FLASK_APP 找到应用入口文件: 处于 app 包之外的 `microblog.py`

{{< highlight python >}}
# filename: __init__.py
from flask import Flask

app = Flask(__name__)

from app import routes

# filename: routes.py
from app import app


@app.route('/')
@app.route('/index')
def index():
    return "Hello, World!"

# filename: microblog.py
from app import app
# export FLASK_APP=microblog.py
# flask run
# curl http://localhost:5000

{{< /highlight >}}

<br/>

## 网页模板

一般 view 函数的返回值决定了用户请求的响应。但通过 `return` 返回大量 html 并不推荐。所以模板文件应运而生，利用它来分离请求的处理逻辑和响应网页的基本布局。

模板中可以使用`{{ variable }}` 或 `{% statement %}` ，完成页面内容的动态生成。

{{< highlight python >}}
from flask import render_template
from app import app


@app.route('/')
@app.route('/index')
def index():
    user = {'username': 'wushuzh'}
    posts = [
        {
            'author': {'username': 'John'},
            'body': 'Beautiful day in Wanderland!'
        },
        {
            'author': {'username': 'Susan'},
            'body': 'The movie was so cool!'
        }
    ]
    return render_template('index.html', title='Home', user=user, posts=posts)
{{< /highlight >}}

模板继承是指从多个网页中抽取公共部分作为基础模板，在其中留出占位块。然后针对每个具体页面继承基础模板的布局，仅仅将其特定内容块定义好即可。

{{< highlight htmldjango >}}
<!-- filename: app/templates/base.html -->
<html>
    <head>
        {% if title %}
        <title>{{ title }} - Microblog</title>
        {% else %}
        <title>Welcome - Microblog</title>
        {% endif %}
    </head>
    <body>
        <div>Microblog: <a href="/index">Home</a></div>
        <hr>
        {% block content %}{% endblock %}
    </body>
</html>


<!-- filename: app/templates/index.html -->
{% extends "base.html" %}

{% block content %}
    <h1>Hello, {{ user.username }} !</h1>
    {% for post in posts %}
    <div><p>{{ post.author.username }} says: <b>{{ post.body }}</b></p></div>
    {% endfor %}
{% endblock %}
{{< /highlight >}}

## 输入表单

flask 生态中一个重要的部分就是拓展，比如针对 WTForms 的封装 flask-wtf ，依然通过 pip 在 venv 中安装。

### 表单提交与防伪

CSRF/XSRF (跨站请求伪造) 是一种常见于表单的攻击。恶意站点会利用某些网站对已登录用户的信任，伪造请求执行非用户本意的操作。一般的防御方式就是使用预设密钥生成一个一次性的令牌嵌入本站的交互网页中，使其和表单中其他显式输入项一并提交，这样就能区分外站的伪造请求。

而只要这个密钥不暴露，站点就能对此种攻击免疫。

- 将密钥在独立模块 config 中 Config 类中定义，之后赋给 app.config 即可;
- 将所有表单定义都存放在模块 forms 中：登录表单类 LoginForm 继承自 FlaskForm ，然后将每个输入项定义为该表单的类变量即可;
- 各类输入项在 wtforms 中都有相应类负责对应，其初始化参数为网页标签和验证器列表(可选);

{{< highlight python >}}
# filename: config.py
import os


class Config(object):
    SECRET_KEY = os.environ.get("SECRET_KEY") or 'my-secret-key'

# filename: __init__.py
from flask import Flask
from config import Config

app = Flask(__name__)
app.config.from_object(Config)

from app import routes

# filename: app/forms.py
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, BooleanField, SubmitField
from wtforms.validators import DataRequired


class LoginForm(FlaskForm):
    username = StringField('Username', validators=[DataRequired()])
    password = PasswordField('Password', validators=[DataRequired()])
    remember_me = BooleanField('Remember Me')
    submit = SubmitField('Sign In')

# filename: app/route.py
from app.forms import LoginForm

# ...

@app.route('/login')
def login():
    form = LoginForm()
    return render_template('login.html', title='Sign In', form=form)

{{< /highlight >}}

上面最后是对路由 `/login` 建立的 view 函数。引入 LoginForm 并用指代变量 form 传入模板渲染函数。

下面就是 login 页面模板，表单的 action 属性值为表单提交的 URL，空表示浏览器当下地址栏的 URL，而 `method` 注明了提交的类型: 默认为 GET —— 这会导致 URL 上跟随一大堆参数，推荐使用 POST —— 将表单参数放入请求体内而不污染 URL:

- `form.hidden_tag()` 会生成防止 CSRF 攻击的隐藏令牌;
- `form.any_field.label` 指代字段描述，`form.any_field()` 生成真正的输入组件，可接受 html 属性，css 类，id 名等值;

{{< highlight htmldjango >}}
# filename: app/templates/login.html
{% extends "base.html" %}


{% block content %}
    <h1>Sign In</h1>
    <form action="" method="post">
        {{ form.hidden_tag() }}
        <p>
            {{ form.username.label }}<br>
            {{ form.username(size=32) }}
        </p>
        <p>
            {{ form.password.label }}<br>
            {{ form.password(size=32) }}
        </p>
        <p>{{ form.remember_me() }} {{ form.remember_me.label }}</p>
        <p>{{ form.submit() }}</p>
    </form>
{% endblock %}
{{< /highlight >}}

### 表单处理和数据验证

客户端浏览器以 POST 方式提交了表单数据，因此要对此路由开放 POST 请求和相应处理，即:

- 浏览器若发来 GET 请求，`form.validate_on_sumbit()`返回 False ，直接返回空表单;
- 若用户发来含有表单数据的 POST 请求，若数据验证无误，用户提交的数据就可以通过表单的类变量获得;
- `flash(msg)` 调用常用于交互提示，需要在前端模板中要作出相应判断和调用才能正确显示;
- `redirect(url)` 调用将页面导航到指定的页面;

{{< highlight python >}}
# filename: app/route.py
from flask import render_template, flash, redirect

# ...

@app.route('/login', methods=['GET', 'POST'])
def login():
    form = LoginForm()
    if form.validate_on_submit():
        flash('Login requested for user {}, remember_me={}'.format(
            form.username.data, form.remember_me.data))
        return redirect('/index')
    return render_template('login.html', title='Sign In', form=form)
{{< /highlight >}}

`get_flashed_messages()` 用于获取 flash 消息列表，检测存在后将消息逐一显示。

针对表单中一些字段已经使用了检验器，以此对于不合法的数据，服务端 `form.validate_on_form()` 返回 False 从而拒绝接受。但为了提升用户体验，我们应该将字段检验结果在前端模板显示给用户。

最后 flask 中有一个用于生成超链接的函数 `url_for()`，可以应对各类需要 URL 的情形，这里就将模板和路由模块中成功登录后的重定向都使用这个函数来生成相应链接。

{{< highlight htmldjango>}}
<!-- filename: app/templates/base.html -->
<html>
    <!-- header -->
    <body>
        <div>
            Microblog:
            <a href="{{ url_for('index') }}">Home</a>
            <a href="{{ url_for('login') }}">Login</a>
        </div>
        <hr>
        {% with messages = get_flashed_messages() %}
        {% if messages %}
        <ul>
            {% for message in messages %}
            <li>{{ message }}</li>
            {% endfor %}
        </ul>
        {% endif %}
        {% endwith %}
        {% block content %}{% endblock %}
    </body>
</html>


<!-- filename: app/templates/login.html -->
{% extends "base.html" %}
{% block content %}
    <form action="" method="post">
      <!-- -->
      {% for error in form.field.errors %}
      <span style="color: red;">[{{ error }}]</span>
      {% endfor %}
    </form>
{% endblock %}

{{< /highlight >}}



参考文档

Miguel Grinberg

> - (2017-12-06) [The Flask Mega-Tutorial Part Ⅰ: Hello, World!](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world)
> - (2017-12-13) [The Flask Mega-Tutorial Part Ⅱ: Templates](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-ii-templates)
> - (2017-12-20) [The Flask Mega-Tutorial Part Ⅲ: Web Forms](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-iii-web-forms)

封面图片来自 [UI Animation Concept](https://dribbble.com/shots/2258225-UI-Animation-Concept) <a href="https://dribbble.com/Kualla"><i class="fa fa-dribbble" aria-hidden="true"></i> Alla Kudin</a>
