+++
date = "2017-11-14T22:47:09+08:00"
title = "Flask 开发 2"
showonlyimage = false
image = "/img/blog/a-dev-env-of-webapp-2/user-testing.png"
topImage = "/img/blog/a-dev-env-of-webapp-2/user-testing.gif"
draft = false
weight = 102
+++

集成数据库和测试
<!--more-->

本系列是 [Microservices with Docker, Flask, and React](https://testdriven.io/) 我的实践版——只要能达到相同效果，我会尝试替换更新原教程中的工具，升级版本，并简述填过或待填的坑。

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

{{< highlight console >}}
$ cd db 
$ docker build -t users-db .
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
