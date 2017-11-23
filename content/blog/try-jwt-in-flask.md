+++
date = "2017-11-22T23:13:51+08:00"
title = "JWT 身份认证"
showonlyimage = false
image = "/img/blog/try-jwt-in-flask/auth.png"
topImage = "/img/blog/try-jwt-in-flask/auth.gif"
draft = false
weight = 110
+++

JSON Web Token 身份验证
<!--more-->

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


参考文档

> - Oleg Agapov (2017-11-21) [JWT authorization in Flask](https://codeburst.io/jwt-authorization-in-flask-c63c1acf4eeb)

封面图片来自 [Two-factor Authentication](https://dribbble.com/shots/3643790-Two-factor-Authentication) <a href="https://dribbble.com/garretbeard"><i class="fa fa-dribbble" aria-hidden="true"></i> Garret J Beard</a>
