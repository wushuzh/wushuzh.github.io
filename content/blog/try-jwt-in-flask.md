+++
date = "2017-11-22T23:13:51+08:00"
title = "JWT 身份认证"
showonlyimage = false
image = "/img/blog/try-jwt-in-flask/auth.png"
topImage = "/img/blog/try-jwt-in-flask/auth.gif"
draft = true
weight = 110
tags = ["flask", "JWT", "security"]
+++

JSON Web Token 身份验证
<!--more-->

### 项目框架

{{< highlight console >}}
$ pipenv install flask \
    flask-restful \
    flask-jwt-extended passlib \
    Flask-SQLAlchemy

$ tree -th
.
├── [ 209]  Pipfile
├── [5.6K]  Pipfile.lock
├── [   0]  models.py
├── [   0]  resources.py
├── [  81]  run.py
└── [ 127]  views.py

0 directories, 6 files
{{< /highlight >}}

{{< highlight python>}}
# run.py 
from flask import Flask


app = Flask(__name__)

import views, models, resources


# views.py 
from run import app
from flask import jsonify


@app.route('/')
def index():
    return jsonify({'message': 'Hello, World!'})

{{< /highlight >}}

{{< highlight console>}}
$ curl http://localhost:5000/
{
  "message": "Hello, World!"
}
{{< /highlight >}}

### RESTful 扩展

[Flask-RESTful](https://flask-restful.readthedocs.io/en/latest/) 能为 Flask 提供快速添加 REST APIs 。

首先资源(Resource)和 Restful APIs 的每个 endpoints 对应，其本质是 view 。这些继承了 Resource 的子类内部也都是 get、post 等方法。

然后在主程序中通过 flask_restful.Api 将上述 resources 和 url 对应起来。

{{< highlight python >}}
# resources.py
from flask_restful import Resource


class UserRegistration(Resource):
    def post(self):
        return {'message': 'User registration'}

# 其他 UserXXX 类定义

# run.py
...
from flask_restful import Api

app = Flask(__name__)
api = Api(app)

import views, models, resources

api.add_resource(resources.UserRegistration, '/registration')
...

{{< /highlight >}}

{{< highlight console>}}
$ curl http://localhost:5000/registration
{
  "message": "The method is not allowed for the requested URL."
}
$ curl http://localhost:5000/registration -X POST
{
  "message": "User registration"
}
{{< /highlight >}}


> 和原生 flask 实现先比，有了 flask_restful 就不再需要对 get post 的返回值进行 jsonify 封装。

对于 POST 操作，输入数据的解析可以借助 flask_restful.reqparse 完成。

{{< highlight python >}}
from flask_restful import Resource, reqparse


parser = reqparse.RequestParse()
parser.add_argument('username',
                    help='This field cannot be blank', required=True)
parser.add_argument('password',
                    help='This field cannot be blank', required=True)


class UserRegistration(Resource):
    def post(self):
        data = parser.parse_args()
        return data
{{< /highlight >}}

参考文档

> - Oleg Agapov (2017-11-21) [JWT authorization in Flask](https://codeburst.io/jwt-authorization-in-flask-c63c1acf4eeb)
> - [curl POST example](https://gist.github.com/subfuzion/08c5d85437d5d4f00e58)

封面图片来自 [Two-factor Authentication](https://dribbble.com/shots/3643790-Two-factor-Authentication) <a href="https://dribbble.com/garretbeard"><i class="fa fa-dribbble" aria-hidden="true"></i> Garret J Beard</a>
