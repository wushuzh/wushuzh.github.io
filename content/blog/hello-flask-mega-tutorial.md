+++
date = "2018-01-29T22:07:11+08:00"
title = "Flask 教程1"
showonlyimage = false
image = "/img/blog/hello-flask-mega-tutorial/list.png"
topImage = "/img/blog/hello-flask-mega-tutorial/list.gif"
draft = false
weight = 603
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
- 之后导入的 routes.py 中定义了 view 函数，放在第三行是因为它不能先于实例 app 存在;
- 装饰器 @app.route 将自定义函数`index`注册为该 URL 的回调;
- 在最外层创建 `microblog.py` 导入包，将其定义为 FLASK_APP 变量，使用 flask 命令行运行即可。

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

{{< /highlight >}}


参考文档

> - Miguel Grinberg (2017-12-06) [The Flask Mega-Tutorial Part I: Hello, World!](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world)

封面图片来自 [UI Animation Concept](https://dribbble.com/shots/2258225-UI-Animation-Concept) <a href="https://dribbble.com/Kualla"><i class="fa fa-dribbble" aria-hidden="true"></i> Alla Kudin</a>
