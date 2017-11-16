+++
date = "2017-11-14T22:47:09+08:00"
title = "Flask 开发 2"
showonlyimage = false
image = "/img/blog/a-dev-env-of-webapp-2/user-testing.png"
topImage = "/img/blog/a-dev-env-of-webapp-2/user-testing.gif"
draft = false
weight = 102
+++

集成容器数据库，为应用创建测试集
<!--more-->

本系列是 [Microservices with Docker, Flask, and React](https://testdriven.io/) 我的实践版——只要能达到相同效果，我会尝试替换更新原教程中的工具，升级版本，并简述填过或待填的坑。

### 容器数据库

引入 Postgresql DB 容器，通过在特定目录放入建库 SQL 脚本，为开发、测试、生产创建各自数据库。

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

{{< highlight console >}}
$ cd db 
$ docker build -t users-db .
{{< /highlight >}}

在 compose yaml 中，原有的 users-service 需要与 DB 相连，故将其定义为“依赖”服务 users-db 。DB 初始化通常会花费一定时间，原教程中 depends_on 和 condition 连用的形式[在 compose v3 中已不再支持](https://docs.docker.com/compose/compose-file/#depends_on)，按照 [Ctrl startup order in Compose](https://docs.docker.com/compose/startup-order/) 的建议，我引入 wait-for-it 脚本判定后端启动时机，并使用 pg_isready 进行 db 服务的健康测试。其他更改和原教程相同: 将数据库连接串设定为环境变量和 project/config.py 配合使用。

{{< highlight diff >}}
diff --git a/docker-compose.yml b/docker-compose.yml
index 2369320..51972f9 100644
--- a/docker-compose.yml
+++ b/docker-compose.yml
@@ -1,6 +1,19 @@
 version: "3.2"
 
 services:
+
+    users-db:
+        container_name: users-db
+        # build: ./project/db
+        image: users-db
+        ports:
+            - 5435:5432 # expose ports - HOST:CONTAINER
+        environment:
+            - POSTGRES_USER=postgres
+            - POSTGRES_PASSWORD=postgres
+        healthcheck:
+            test: "pg_isready -U postgres"
+
     users-service:
         container_name: users-service
         #build: .
@@ -14,3 +27,11 @@ services:
         environment:
             - FLASK_APP=project/__init__.py
             - APP_SETTINGS=project.config.DevelopmentConfig
+            - DATABASE_URL=postgres://postgres:postgres@users-db:5432/users_dev
+            - DATABASE_TEST_URL=postgres://postgres:postgres@users-db:5432/users_test
+        depends_on:
+            - users-db
+        command: ["./wait-for-it.sh", "users-db:5432", "-t", "90", "--", "flask", "run", "-h", "0.0.0.0"]
+        links:
+            - users-db
+
{{< /highlight >}}
<br />

### Model 层

为当下虚拟环境添加负责 ORM 及 DBAPI 包:

- Flask-SQLAlchemy 
- psycopg2 [SQLAlchemy PostgreSQL default API](http://docs.sqlalchemy.org/en/latest/core/engines.html#postgresql)

User 类中对表名和各 col 的详细定义

{{< highlight diff >}}
diff --git a/project/__init__.py b/project/__init__.py
index 45cd669..1f4d324 100644
--- a/project/__init__.py
+++ b/project/__init__.py
@@ -2,6 +2,8 @@
 
 import os
 from flask import Flask, jsonify
+import datetime
+from flask_sqlalchemy import SQLAlchemy
 
 # instantiate the app
 app = Flask(__name__)
@@ -10,10 +12,39 @@ app = Flask(__name__)
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
+# customer cmds
+@app.cli.command()
+def recreate_db():
+    """Recreates a database."""
+    db.drop_all()
+    db.create_all()
+    db.session.commit()
+
+
+# routes
 @app.route('/ping', methods=['GET'])
 def ping_pong():
     return jsonify({
         'status': 'success',
         'message': 'pong!'
     })
+
{{< /highlight >}}

重新打包最新 users-service 的镜像，启动各个服务，查看相关日志，并在 users-service 容器中执行重建库表的指令。

{{< highlight console >}}
$ docker build -t users-service \
    --build-arg https_proxy=ip:port \
    .
$ docker-compose up -d
Creating network "users_default" with the default driver
Creating users-db ... 
Creating users-db ... done
Creating users-service ... 
Creating users-service ... done

$ docker-compose logs -f
...
users-db      | PostgreSQL init process complete; ready for start up.
users-db      | ts: listening on IPv4 address "0.0.0.0", port 5432
users-service | wait-for-it.sh: users-db:5432 is available after 83 seconds
...

$ curl http://$(docker-machine ip vdev):5001/ping 
{
  "message": "pong!", 
  "status": "success"
}

$ docker-compose run users-service flask recreate_db
Starting users-db ... done
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

### 添加测试集

{{< highlight python >}}
# project/tests/base.py

from flask_testing import TestCase
from project import app, db


class BaseTestCase(TestCase):
    def create_app(self):
        app.config.from_object('project.config.TestingConfig')
        return app

    def setUp(self):
        db.create_all()
        db.session.commit()

    def tearDown(self):
        db.session.remove()
        db.drop_all()

# /project/tests/test_users.py

import json

from project.tests.base import BaseTestCase


class TestUserService(BaseTestCase):
    """Tests for the Users Service."""
    def test_users(self):
        """Ensure the /ping route behaves correctly."""
        response = self.client.get('/ping')
        data = json.loads(response.data.decode())
        self.assertEqual(response.status_code, 200)
        self.assertIn('pong!', data['message'])
        self.assertIn('success', data['status'])


# project/tests/test_config.py

import unittest

from flask import current_app
from flask_testing import TestCase

from project import app


class TestDevelopmentConfig(TestCase):
    def create_app(self):
        app.config.from_object('project.config.DevelopmentConfig')
        return app

    def test_app_is_developement(self):
        self.assertTrue(app.config['SECRET_KEY'] == 'my_precious')
        self.assertTrue(app.config['DEBUG'] is True)
        self.assertFalse(current_app is None)
        self.assertTrue(
            app.config['SQLALCHEMY_DATABASE_URI'] ==
            'postgres://postgres:postgres@users-db:5432/users_dev'
        )

{{< /highlight >}}


compose 中指定应用的配置为 development，而我们可以通过测试重建各个环境下的 flask 应用并查看不同的配置是否生效。另外每次通过 curl 查看 ping 可以通过自动测试完成。

> 我不得不在每次代码改动后都通过 docker cli 单独编译 image ，其实如果 bind mount 可以工作，是可以直接先行测试结果的。而且 docker-compose 也可以帮助编译，但我的 compose 中 build 选项却一直注释掉，主要是因为 image 安装新的软件包在出入编译时环境变量 (https_proxy) ，但 build 和 --build-arg 一切使用必须明确制定 service 名字，结果是 image 的名字被强行加入 project 前缀

{{< highlight console >}}
$ docker build -t users-service --build-arg https_proxy=ip:port .
...
$ docker-compose restart 
Restarting users-service ... done
Restarting users-db      ... done

$ docker-compose run users-service flask test
Starting users-db ... done
test_app_is_developement (test_config.TestDevelopmentConfig) ... ok
test_app_is_production (test_config.TestProductionConfig) ... ok
test_app_is_testing (test_config.TestTestingConfig) ... ok
test_users (test_users.TestUserService)
Ensure the /ping route behaves correctly. ... ok

----------------------------------------------------------------------
Ran 4 tests in 0.349s

OK
{{< /highlight >}}


参考文档

> - Horst Gutmann (2017-09-02) [Docker Healthchecks](https://zerokspot.com/weblog/2017/09/02/docker-healthchecks/)
> - [docker-compose run](https://docs.docker.com/compose/reference/run/)
> - [docker-compose build](https://docs.docker.com/compose/reference/build/)

封面图片来自 [Multitasking](https://dribbble.com/shots/3491993-Multitasking) <a href="https://dribbble.com/kunchevsky"><i class="fa fa-dribbble" aria-hidden="true"></i> Alex Kunchevsky</a>  
