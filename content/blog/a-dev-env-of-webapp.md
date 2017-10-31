+++
date = "2017-10-16T20:26:19+08:00"
title = "Flask 开发 1"
showonlyimage = false
image = "/img/blog/a-dev-env-of-webapp/multitasking.png"
draft = false
weight = 101
+++

麻雀虽小但五脏俱全的开发环境
<!--more-->

本系列是 [Microservices with Docker, Flask, and React](https://testdriven.io/) 我的实践版——只要能达到相同效果，我会尝试替换更新原教程中的工具，升级版本，并简述填过或待填的坑。

项目架构

<img alt="Architeture" src="/img/blog/a-dev-env-of-webapp/architecture.png" class="img-responsive">

代码高层组织分为 5 个微服务: main, users, client, swapper, eval

## 脚手架
### 虚拟环境

为每一个项目创建独立虚拟环境并引入 requirements.txt 已成为 python 开发的标准操作。而当下可用于创建虚拟环境的模块很多，详见[What is the difference between venv, pyvenv, pyenv, virtualenv, virtualenvwrapper, pipenv, etc?](https://stackoverflow.com/a/41573588/4393386)

Kenneth Reitz 在 (2016-02-25) [A Better Pip Workflow™](https://www.kennethreitz.org/essays/a-better-pip-workflow) 提到了 requirements.txt 中常见的使用方式的问题，在提出了一个暂时解决方案后，作者在 (2017-01-22) [Announcing Pipenv!](https://www.kennethreitz.org/essays/announcing-pipenv) 又发布了一个旨在完美解决此问题的工具。新方案简洁有效、安全，得到了 python 官方的推荐。

{{< highlight console >}}
$ mkdir users && cd users
# pipenv install --three flask==0.12.2
$ pipenv install flask==0.12.2
$ mkdir project && touch project/__init__.py
{{< /highlight >}}

<br />

### hello world

> TODO: 后续应该按照 Robert Picard 的开源书 Explore Flask 中 [Organizing your project](https://exploreflask.com/en/latest/organizing.html) 微调下面的文件位置。


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

原始教程又额外安装了 flask-script，但 flask 从 0.11 开始内置已经了命令行工具，因此可以跳过此步。

{{< highlight console >}}
$ pipenv shell
# enter into virutalenv
$ export FLASK_APP=./project/__init__.py
$ flask run
 * Serving Flask app "project"
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
127.0.0.1 - - [DateTime] "GET / HTTP/1.1" 404 -
127.0.0.1 - - [DateTime] "GET /ping HTTP/1.1" 200 -
{{< /highlight >}}

确认通过浏览器访问 http://localhost:5000/ping 可以获得下面 json 后，继续为项目添加 config.py。

{{< highlight json >}}
{
  "message": "pong!",
  "status": "success"
}
{{< /highlight >}}
<br />


### 配置隔离

{{< highlight python >}}
diff --git a/project/__init__.py b/project/__init__.py
index 4cc950a..614cbbc 100644
--- a/project/__init__.py
+++ b/project/__init__.py
@@ -5,6 +5,9 @@ from flask import Flask, jsonify
 # instantiate the app
 app = Flask(__name__)
 
+# set conf
+app.config.from_object('project.config.DevelopmentConfig')
+
 
 @app.route('/ping', methods=['GET'])
 def ping_pong():
diff --git a/project/config.py b/project/config.py
new file mode 100644
index 0000000..0f9cd90
--- /dev/null
+++ b/project/config.py
@@ -0,0 +1,23 @@
+# project/config.py
+
+
+class BaseConfig:
+    """Base conf"""
+    DEBUG = False
+    TESTING = False
+
+
+class DevelopmentConfig(BaseConfig):
+    """Dev conf"""
+    DEBUG = True
+
+
+class TestingConfig(BaseConfig):
+    """Testing conf"""
+    DEBUG = True
+    TESTING = True
+
+
+class ProductionConfig(BaseConfig):
+    """Production conf"""
+    DEBUG = False
{{< /highlight >}}

{{< highlight console >}}
$ export FLASK_DEBUG=1
$ flask run
 * Serving Flask app "project"
 * Forcing debug mode on
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 110-739-931
{{< /highlight >}}

> TODO: 当环境 FLASK_DEBUG 为 1 时，console 才明确打印出 Debugger 状态和 PIN。但没有 FLASK_DEBUG 变量时，若打印 app.config 其中的 DEBUG 也确实为 True。不知有何区别，待查

## 容器封装

### 宿主服务器

TODO: 编排组织各个 container 和微服务当然可以用 docker-compose ，但也应该尝试一下 minikube 的配置方式。

{{< highlight console >}}
$ sudo pacman -S docker docker-compose docker-machine
$ sudo usermod -aG docker wushuzh

$ docker -v
Docker version 17.10.0-ce, build f4ffd2511c
$ docker-compose -v
docker-compose version 1.16.1, build unknown
$ docker-machine -v
docker-machine version 0.13.0, build HEAD

{{< /highlight >}}

docker-machine 首先下载最新宿主服务器iso，并以此创建虚拟机，比如名为 dev。这之后所有操作都是通过 docker 客户端联系远端虚拟机 docker 守护进程完成的。

好处是不会弄乱你本机配置，搞砸了直接删除虚拟机，不会留下过多垃圾在系统中。但引入的虚拟机使得 debug 变得繁琐。但 docker-machine 指令支持很多选项，是不是可以将相关服务的日志 rsyslog 转入本机使得后续查看更为方便。

虚拟机的相关配置在 ```$HOME\.docker\machine\machines\default\config.json```，更改后需要执行 ```docker-machine provision``` (待验证)

{{< highlight console >}}
$ docker-machine create -d "virtualbox" \
    --engine-env HTTP_PROXY=your_proxy  \
    --engine-env HTTPS_PROXY=your_proxy \
    dev

$ docker-machine ls

{{< /highlight >}}

通过执行 eval 指令更改当前的环境变量，将后面的 docker 指令传送到虚拟机 dev 上执行。

{{< highlight console >}}
$ docker-machine env --no-proxy dev
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="/home/wushuzh/.docker/machine/machines/dev"
export DOCKER_MACHINE_NAME="dev"
export NO_PROXY="192.168.99.100"
# Run this command to configure your shell: 
# eval $(docker-machine env --no-proxy dev)

$ docker run hello-world
{{< /highlight >}}

### 容器化

创建 Docker 镜像: 含当前代码及其运行环境，并启动 web 服务

原教程中直接上了 docker-compose ，而我是先用原生 Dockerfile 完成，之后再转化为 compose 模式。

{{< highlight dockerfile >}}
FROM python:3.6.1

# set working directory
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

# add requirements (to leverage Docker cache)
ADD ./requirements.txt /usr/src/app/requirements.txt

# install requirements
RUN pip install -r requirements.txt

# add app
ADD . /usr/src/app

EXPOSE 5000

ENV FLASK_APP project/__init__.py

# run server
CMD flask run -h 0.0.0.0
{{< /highlight >}}

{{< highlight console >}}
docker build \
  -t test-users \
  --build-arg HTTPS_PROXY=your_proxy \
  . 
docker images
docker run -d -p 5001:5000 test-users
docker ps
curl http://$(docker-machine ip dev):5001/ping
docker stop/kill <hash>
docker rm <hash>
docker rmi test-users
{{< /highlight >}}

### 服务化

{{< highlight yaml >}}
version: "3.2"

services:
    users-service:
        container_name: users-service
        build: .
        volumes:
            - type: bind
              source: .
              target: /usr/src/app
        ports:
            - 5001:5000 # expose ports - HOST:CONTAINER
        environment:
            - FLASK_APP=project/__init__.py
{{< /highlight >}}
> 我安装的 docker 版本很新，所以直接将原教程中 version 2 换为 3

注意上面写法将原 Dockerfile 中 EXPOSE 和 ENV 等指令转到了 compose yaml 文件中，并引入了 volumes , 而且是 bind-mount 模式，这里并不和之前 Dockerfile 中的 ADD [冲突](https://github.com/docker/compose/issues/1616#issuecomment-117716753): 这个设定将本地的代码修改就能直接映射到容器中，无需重新 build 镜像。关于 volume 、bind mount、tmpfs mount 的细节和各自用例，详见 [Manage data in Docker](https://docs.docker.com/engine/admin/volumes/)


{{< highlight console >}}
$ docker-compose build \
    --build-arg HTTPS_PROXY=10.144.1.10:8080 \
    users-service
$ docker-compose up -d
$ docker ps -a

$ docker logs <hash>
Usage: flask run [OPTIONS]

Error: The file/path provided (project/__init__.py) 
  does not appear to exist.  Please verify the path is correct. 
If app is not on PYTHONPATH, ensure the extension is .py

$ docker inspect <hash> |grep -A 9 Mounts
        "Mounts": [
            {
                "Type": "bind",
                "Source": "/myhome/pywork/try-flask/users",
                "Destination": "/usr/src/app",
                "Mode": "rw",
                "RW": true,
                "Propagation": "rprivate"
            }
        ],
{{< /highlight >}}

> TODO: 上面的报错应该时 bind-mount 没有正确的工作，这个问题我还没有查找到原因。

关于共享目录，首先是查到 nathanleclaire (2014-12-31) [Proposal: machine share](https://github.com/docker/machine/issues/179#issue-53144018) 而当下环境是通过 virtualbox 创建容器宿主机 dev ，登录上是可以看到当前用户的家目录已经通过 vboxsf 协议共享，所以问题大概仍在 dev 和容器之间？

{{< highlight console >}}
$ docker-machine ssh dev
docker@dev:~$ df -Th|grep vboxsf
hosthome   vboxsf   78.2G   22.5G  55.7G  29% /hosthome

{{< /highlight >}}

### 其他管理指令

如果上述过程中遇到了错误，下面指令可用于清理残余垃圾。

{{< highlight console >}}

$ docker-compose down

$ docker attach <hash>
# type <C-p> <C-q> detach from interactive mode 

# try to start a stop container again
$ docker start <hash>

# remove exited container
$ docker ps -a
$ docker ps -a -f status=exited
$ docker rm $(docker ps -a -f status=exited -q)

# remove dangling images
$ docker images -f dangling=true
$ docker rmi $(docker images -f dangling=true -q)

{{< /highlight >}}

### 环境变量

通过在 compose yaml 中制定环境变量，从而达到切换不同阶段(开发、测试、生产)配置的目的

{{< highlight diff >}}
diff --git a/docker-compose.yml b/docker-compose.yml
index ec660ec..a7d474b 100644
--- a/docker-compose.yml
+++ b/docker-compose.yml
@@ -9,3 +9,5 @@ services:
 #      - '.:/usr/src/app'
     ports:
       - 5001:5000 # expose ports - HOST:CONTAINER
+    environment:
+      - APP_SETTINGS=project.config.DevelopmentConfig
diff --git a/project/__init__.py b/project/__init__.py
index d3355e2..1566c3d 100644
--- a/project/__init__.py
+++ b/project/__init__.py
@@ -8,7 +8,8 @@ from flask import Flask, jsonify
 app = Flask(__name__)

 # set config
-app.config.from_object('project.config.DevelopmentConfig')
+app_settings = os.getenv('APP_SETTINGS')
+app.config.from_object(app_settings)

 @app.route('/ping', methods=['GET'])
 def ping_pong():
{{< /highlight >}}


## 数据持久化

首先加入数据库相关的包，在 User 类中定义 model : 表名，各列类型定义。

{{< highlight diff >}}
diff --git a/requirements.txt b/requirements.txt
index 9390a0b..73d1689 100644
--- a/requirements.txt
+++ b/requirements.txt
@@ -1,2 +1,4 @@
 Flask==0.12.1
 Flask-Script==2.0.5
+Flask-SQLAlchemy==2.2
+psycopg2==2.7.1

diff --git a/project/__init__.py b/project/__init__.py
index 1566c3d..bbdb3ba 100644
--- a/project/__init__.py
+++ b/project/__init__.py
@@ -1,7 +1,9 @@
 # project/__init__.py

-
+import os
+import datetime
 from flask import Flask, jsonify
+from flask_sqlalchemy import SQLAlchemy


 # instantiate the app
@@ -11,6 +13,25 @@ app = Flask(__name__)
 app_settings = os.getenv('APP_SETTINGS')
 app.config.from_object(app_settings)

+# instantiate the db
+db = SQLAlchemy(app)
+
+# model
+class User(db.Model):
+    __tablename__ = "users"
+    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
+    username = db.Column(db.String(128), nullable=False)
+    email = db.Column(db.String(128), nullable=False)
+    active = db.Column(db.Boolean(), default=False, nullable=False)
+    created_at = db.Column(db.DateTime, nullable=False)
+
+    def __init__(self, username, email):
+        self.username = username
+        self.email = email
+        self.created_at = datetime.datetime.utcnow()
+
+
+# routes
 @app.route('/ping', methods=['GET'])
 def ping_pong():
     return jsonify({

{{< /highlight >}}

### 容器数据库


引入 DB 容器，并分别为开发、测试、生产创建数据库。

{{< highlight diff >}}
diff --git a/project/db/Dockerfile b/project/db/Dockerfile
new file mode 100644
index 0000000..5107468
--- /dev/null
+++ b/project/db/Dockerfile
@@ -0,0 +1,4 @@
+FROM postgres
+
+# run create.sql on init
+ADD create.sql /docker-entrypoint-initdb.d
diff --git a/project/db/create.sql b/project/db/create.sql
new file mode 100644
index 0000000..dc1815a
--- /dev/null
+++ b/project/db/create.sql
@@ -0,0 +1,3 @@
+CREATE DATABASE users_prod;
+CREATE DATABASE users_dev;
+CREATE DATABASE users_test;
{{< /highlight >}}

在 compose yaml 中，定义服务: users-db 作为被依赖服务，和原有 users-service 相连，最后添加环境变量:定义数据库的连接串。

{{< highlight diff >}}
diff --git a/docker-compose.yml b/docker-compose.yml
index a7d474b..2f66cb2 100644
--- a/docker-compose.yml
+++ b/docker-compose.yml
@@ -2,6 +2,17 @@ version: '2.1'

 services:

+  users-db:
+    container_name: users-db
+    build: ./project/db
+    ports:
+      - 5435:5432  # expose ports - HOST:CONTAINER
+    environment:
+      - POSTGRES_USER=postgres
+      - POSTGRES_PASSWORD=postgres
+    healthcheck:
+      test: exit 0
+
   users-service:
     container_name: users-service
     build: .
@@ -11,3 +22,10 @@ services:
       - 5001:5000 # expose ports - HOST:CONTAINER
     environment:
       - APP_SETTINGS=project.config.DevelopmentConfig
+      - DATABASE_URL=postgres://postgres:postgres@users-db:5432/users_dev
+      - DATABASE_TEST_URL=postgres://postgres:postgres@users-db:5432/users_test
+    depends_on:
+      users-db:
+        condition: service_healthy
+    links:
+      - users-db
diff --git a/project/config.py b/project/config.py
index b92979b..9feb7ea 100644
--- a/project/config.py
+++ b/project/config.py
@@ -1,22 +1,28 @@
 # project/config.py
+import os


 class BaseConfig:
     """Base configuration"""
     DEBUG = False
     TESTING = False
+    SQLALCHEMY_TRACK_MODIFICATIONS = False


 class DevelopmentConfig(BaseConfig):
     """Development configuration"""
     DEBUG = True
+    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL')


 class TestingConfig(BaseConfig):
     """Testing configuration"""
     DEBUG = True
     TESTING = True
+    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_TEST_URL')


 class ProductionConfig(BaseConfig):
+    """Production configuration"""
     DEBUG = False
+    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL')
{{< /highlight >}}

测试当前配置 ```docker-compose up -d --build```

最后，为数据库注册新指令 recreate_db :

{{< highlight diff >}}
diff --git a/manage.py b/manage.py
index 6cd23e3..88c8283 100644
--- a/manage.py
+++ b/manage.py
@@ -1,9 +1,18 @@
 # manage.py

 from flask_script import Manager
-from project import app
+from project import app, db

 manager = Manager(app)

+
+@manager.command
+def recreate_db():
+    """Recreates a database."""
+    db.drop_all()
+    db.create_all()
+    db.session.commit()
+
+
 if __name__ == '__main__':
     manager.run()

{{< /highlight >}}

测试执行指令，创建 users 表，进入 psql 查看结果:

{{< highlight console >}}
$ docker-compose run users-service python manage.py recreate_db

$ docker exec -ti $(docker ps -aqf "name=users-db") psql -U postgres

psql (10.0)
Type "help" for help.

postgres=# \c users_dev
You are now connected to database "users_dev" as user "postgres".
users_dev=# \dt
         List of relations
 Schema | Name  | Type  |  Owner   
--------+-------+-------+----------
 public | users | table | postgres
(1 row)

users_dev=# \q


{{< /highlight >}}

参考文档

> - [Python's Official Repository Included 10 'Malicious' Typo-Squatting Modules](https://developers.slashdot.org/story/17/09/16/2030229/pythons-official-repository-included-10-malicious-typo-squatting-modules)
> - [Why does the logo of Flask (Python framework) not look like a flask?](https://www.quora.com/Why-does-the-logo-of-Bottle-web-framework-look-like-Flask-Python-framework-Why-does-the-logo-of-Flask-Python-framework-not-look-like-a-flask)
> - Full Stack Python [Flask](https://www.fullstackpython.com/flask.html)
> - [why local bind mount not work with container on docker machine](https://bbs.archlinux.org/viewtopic.php?id=231232)
> - Amy Hoy (2006-12-22) [Help Vampires: A Spotter’s Guide](http://slash7.com/2006/12/22/vampires/)

封面图片来自 [Multitasking](https://dribbble.com/shots/3491993-Multitasking) <a href="https://dribbble.com/kunchevsky"><i class="fa fa-dribbble" aria-hidden="true"></i> Alex Kunchevsky</a>  
