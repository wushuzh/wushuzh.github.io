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

### 虚拟容器主机

{{< highlight console >}}
$ sudo pacman -S docker docker-compose docker-machine
$ sudo usermod -aG docker wushuzh

$ docker -v
Docker version 17.09.0-ce, build afdb6d44a8
$ docker-compose -v
docker-compose version 1.16.1, build unknown
$ docker-machine -v
docker-machine version 0.12.2, build HEAD
{{< /highlight >}}

下面指令将创建一个名为 dev 的虚拟机，作为所有容器的宿主服务器。之后进行的各种操作都是通过本地 docker 客户端 cli 连接处于虚拟机 dev 上的远端 docker 服务端捣鼓完成的。

这样的好处是不会弄乱你本机的配置，搞砸了直接删除虚拟机，不会留下过多垃圾在系统中。但可能给后续 debug 造成一些问题。但 docker-machine 指令貌似支持很多选型，是不是可以将相关服务的日志 rsyslog 转入本机使得后续查看更为方便。

虚拟机的相关配置在 ```$HOME\.docker\machine\machines\default\config.json```，更改后需要执行 ```docker-machine provision```

{{< highlight console >}}
$ docker-machine create \
    --engine-env HTTP_PROXY=your_proxy  \
    --engine-env HTTPS_PROXY=your_proxy \
    dev
...
(dev) Creating VirtualBox VM...
(dev) Creating SSH key...
(dev) Starting the VM...
(dev) Check network ...
...
Copying certs ...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
...

$ docker-machine ls

{{< /highlight >}}

通过执行 eval 指令更改当前的环境变量，将后面的 docker 指令传送到虚拟机 dev 上执行。

{{< highlight console >}}
$ docker-machine env --no-proxy dev
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.102:2376"
export DOCKER_CERT_PATH="~/.docker/machine/machines/dev"
export DOCKER_MACHINE_NAME="dev"
export no_proxy="localhost,192.168.99.100,127.0.0.1,192.168.99.102"
# Run this command to configure your shell:
# eval $(docker-machine env --no-proxy dev)

$ docker run hello-world
{{< /highlight >}}

{{< highlight dockerfile >}}
FROM python:3.6.1

ENV http_proxy your_proxy
ENV https_proxy your_proxy

# set working directory
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

# add requirements (to leverage Docker cache)
ADD ./requirements.txt /usr/src/app/requirements.txt

# install requirements
RUN pip install -r requirements.txt

# add app
ADD . /usr/src/app

# run server
CMD python manage.py runserver -h 0.0.0.0
{{< /highlight >}}

{{< highlight yaml >}}
version: '2.1'

services:

  users-service:
    container_name: users-service
    build: .
#    volumes:
#      - '.:/usr/src/app'
    ports:
      - 5001:5000 # expose ports - HOST:CONTAINER

{{< /highlight >}}


{{< highlight console >}}
$ docker-compose build
$ docker-compose up -d
$ docker-machine ip dev

$ curl http://ip-addr:5001/ping
{
  "message": "pong!",
  "status": "success"
}

{{< /highlight >}}


{{< highlight console >}}

$ docker-compose down

# remove exited container
docker ps -a
docker ps -a -f status=exited
docker rm $(docker ps -a -f status=exited -q)

# remove dangling images
$ docker images -f dangling=true
$ docker rmi $(docker images -f dangling=true -q)

{{< /highlight >}}

https://www.quora.com/Why-does-the-logo-of-Bottle-web-framework-look-like-Flask-Python-framework-Why-does-the-logo-of-Flask-Python-framework-not-look-like-a-flask

https://www.fullstackpython.com/flask.html

https://testdriven.io/part-one-intro/


封面图片来自 [Multitasking](https://dribbble.com/shots/3491993-Multitasking) <a href="https://dribbble.com/kunchevsky"><i class="fa fa-dribbble" aria-hidden="true"></i> Alex Kunchevsky</a>  
