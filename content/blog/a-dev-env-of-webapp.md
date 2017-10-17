+++
date = "2017-10-16T20:26:19+08:00"
title = "Python Web 开发"
showonlyimage = false
image = "/img/blog/a-dev-env-of-webapp/multitasking.png"
draft = false
weight = 101
+++

麻雀虽小但五脏俱全的开发环境
<!--more-->

## 容器化项目

### 虚拟环境

通过 virtualenvwrapper 创建一个示例项目(flask-microservices-users)关联的同名虚拟环境。

{{< highlight console >}}
~$ mkproject flask-microservices-users
creating ~/.ve/...
Using base prefix '/usr'
New python executable in ...
Also creating executable in ...
Installing setuptools, pip, wheel...done.
(may hung up for a while)
creating ~/.ve/flask-microservices-users/...
...
Creating ~/pywork/flask-microservices-users
Setting project for flask-microservices-users to ~/pywork/flask-microservices-users

{{< /highlight >}}

{{< highlight console >}}
(flask-microservices-users) $ pip install flask==0.12.2

Installing collected packages:
  click, itsdangerous, Werkzeug, MarkupSafe, Jinja2, flask
Successfully installed
  Jinja2-2.9.6 MarkupSafe-1.0 Werkzeug-0.12.2
  click-6.7 flask-0.12.2 itsdangerous-0.24

{{< /highlight >}}
<br />

### 项目脚手架

> 后续应该按照 Robert Picard 的开源书 Explore Flask 中 [Organizing your project](https://exploreflask.com/en/latest/organizing.html) 微调下面的文件位置。

{{< highlight python >}}
# project/__init__.py

from flask import Flask, jsonify

# instantiate the app
app = Flask(__name__)

@app.route('/ping', method=['GET'])
def ping_pong():
    return jsonify({
        'status': 'success',
        'message': 'pong!'
    })
{{< /highlight >}}

{{< highlight console >}}
(flask-microservices-users) $ pip install flask-script==2.0.5

Installing collected packages:
  click, itsdangerous, Werkzeug, MarkupSafe, Jinja2, flask
Successfully installed
  Jinja2-2.9.6 MarkupSafe-1.0 Werkzeug-0.12.2
  click-6.7 flask-0.12.2 itsdangerous-0.24

$ pip freeze > requirements.txt
{{< /highlight >}}

{{< highlight python >}}
# manage.py

from flask_script import Manager
from project import app

manager = Manager(app)

if __name__ == '__main__':
    manager.run()
{{< /highlight >}}

{{< highlight console >}}
tty1 $ python manage.py runserver
tty2 $ curl localhost:5000/ping
{
  "message": "pong!",
  "status": "success"
}
{{< /highlight >}}
<br />

### 灰度发布

{{< highlight python >}}
# project/config.py


class BaseConfig:
    """Base configuration"""
    DEBUG = False
    TESTING = False


class DevelopmentConfig(BaseConfig):
    """Development configuration"""
    DEBUG = True


class TestingConfig(BaseConfig):
    """Testing configuration"""
    DEBUG = True
    TESTING = True


class ProductionConfig(BaseConfig):
    DEBUG = False

{{< /highlight >}}

{{< highlight python >}}
# project/__init__.py
#...

# line 10-11: set config
app.config.from_object('project.config.DevelopmentConfig')

#...
{{< /highlight >}}

{{< highlight console >}}
tty1 $ python manage.py runserver
* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
* Restarting with stat
* Debugger is active!
* Debugger PIN: 145-598-075

{{< /highlight >}}

## 容器封装

###

{{< highlight console >}}

$ docker -v
Docker version 17.09.0-ce, build afdb6d44a8
$ docker-compose -v
docker-compose version 1.16.1, build unknown
$ docker-machine -v
docker-machine version 0.12.2, build HEAD

{{< /highlight >}}

https://www.quora.com/Why-does-the-logo-of-Bottle-web-framework-look-like-Flask-Python-framework-Why-does-the-logo-of-Flask-Python-framework-not-look-like-a-flask

https://www.fullstackpython.com/flask.html

https://testdriven.io/part-one-intro/


封面图片来自 [Multitasking](https://dribbble.com/shots/3491993-Multitasking) <a href="https://dribbble.com/kunchevsky"><i class="fa fa-dribbble" aria-hidden="true"></i> Alex Kunchevsky</a>  
