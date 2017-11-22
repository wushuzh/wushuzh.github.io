+++
date = "2017-11-15T19:01:33+08:00"
title = "Flask 开发 3"
showonlyimage = false
image = "/img/blog/a-dev-env-of-webapp-3/blue-print.png"
draft = false
weight = 102
+++

运用蓝图和测试集将原应用拆解为多个组件
<!--more-->

本系列是 [Microservices with Docker, Flask, and React](https://testdriven.io/) 我的实践版——只要能达到相同效果，我会尝试替换更新原教程中的工具，升级版本，并简述填过或待填的坑。

### 蓝图

蓝图概念被引入到 flask 主要用于拆解一个大型应用为多个组件，这些蓝图可被各种复用，顺带的好处是解藕、降低开发查错的复杂度。

蓝图的组织既可以按职能划分，也可按照事业部制划分。选择哪个取决于各组件关联程度和整个应用的使用流程。

首先将 Users 的 Model 和 View 从原 init 文件中移出:

- 尤其是创建 users_blueprint 并与函数 ping_pong 关联
- 原 init 就做拓展 (db 建表)，再注册 users_blueprint 蓝图即可。

{{< highlight python >}}
# project/api/view.py
from flask import Blueprint, jsonify


users_blueprint = Blueprint('users', __name__)


@users_blueprint.route('/ping', methods=['GET'])
def ping_pong():
    return jsonify({
        'status': 'success',
        'message': 'pong!'
    })

# project/__init__.py
# import ...

# instantiate the db
db = SQLAlchemy()


def create_app():
    # instantiate the app
    app = Flask(__name__)

    # set conf
    app_settings = os.getenv('APP_SETTINGS')
    app.config.from_object(app_settings)

    # set up extensions
    db.init_app(app)

    # register bulueprints
    from project.api.views import users_blueprint
    app.register_blueprint(users_blueprint)

    return app

# remove route and view
{{< /highlight >}}

> 因为我使用 flask 原生 cli ，而 init 中 app 已被 create_app 封装，按照[Factory Functions](http://flask.pocoo.org/docs/0.12/cli/#factory-functions)的描述，flask 不知道如何创建应用。因而加入 autoapp.py ，并把自定义的指令都进入此文件。

### Restful POST

到现在为止，ping_pong 是仅有类似 hello_world 的 end_point 。我们下一步要围绕 Users 模型用 TDD 的方式实现“创建”、“获取单个/所有”这样的 Restful API POST/GET 操作

首先是为 POST 创建一个用户，在 TestUserService 类中添加一个测试用例:

{{< highlight python >}}
def test_add_user(self):
    """Ensure a new user can be added to the database."""
    with self.client:
        response = self.client.post(
            '/users',
            data=json.dumps(dict(
                username="me",
                email="me@wushuzh.com"
            )),
            content_type='application/json'
        )
        data = json.loads(response.data.decode())
        self.assertIn('me@wushuzh.com was added!', data['message'])
        self.assertIn('success', data['status'])

{{< /highlight >}}

在执行测试指令报告出错误后，再在 view 中添加 add_user 函数和此 URL 的 POST 相对应。

{{< highlight python >}}
@users_blueprint.route('/users', methods=['POST'])
def add_user():
    post_data = request.get_json()
    username = post_data.get('username')
    email = post_data.get('email')
    db.session.add(User(username=username, email=email))
    db.session.commit()
    response_object = {
        'status': 'success',
        'message': f'{email} was added!'
    }
    return jsonify(response_object), 201

{{< /highlight >}}

如果遇到下面的错误，由于 postgresql 镜像已经含有 expose 5432 的设定，这里可能是因为和上一次从外部测试 psql 连接的 5435 的设定冲突造成的 。

{{< highlight console >}}
File
  "/usr/local/lib/python3.6/site-packages/psycopg2/__init__.py",
  line 130, in connect
  conn = _connect(dsn, connection_factory=connection_factory, **kwasync)
psycopg2.OperationalError:
  could not connect to server: Connection refused
  Is the server running on host "users-db" (172.18.0.2)
  and accepting TCP/IP connections on port 5432?

# comment ports in docker-compose.yml 

$ docker-compose down
$ docker-compose up -d
$ docker-compose run users-service flask test
...
test_add_user (test_users.TestUserService)
Ensure a new user can be added to the database. ... ok
...

-------------------------------------------------------
Ran 5 tests in 0.982s

OK
{{< /highlight >}}

继续添加错误和异常的测试用例，并思考敲定对应操作的返回代码及结果的数据结构:

- 提交数据为空
- 数据内容错误
- 重复提交同一数据

实现思路就是收到 POST 数据后先确认是否空，然后到库中查重，而数据完整性可以借助数据表的约束异 IntegrityError 常完成。

{{< highlight console >}}

$ docker-compose run users-service flask test
...
test_add_user_duplicate_user (test_users.TestUserService)
Ensure err is thrown if the email already exists ... ok
test_add_user_invalid_json (test_users.TestUserService)
Ensure error is thrown if the JSON object is empty ... ok
test_add_user_invalid_json_keys (test_users.TestUserService)
Ensure err is thrown if json obj without username key ... ok

-----------------------------------------------------
Ran 8 tests in 1.836s

OK
{{< /highlight >}}

### Restful GET

然后是通过 user_id 获得详细信息，先测试后实现，先正常，再异常。

- 若 user_id 非数字则 int 函数转换时会有 ValueError 异常
- 若 user_id 不存在，则数据库查询无结果，检测后返回 404 即可

{{< highlight console>}}
test_single_user (test_users.TestUserService)
Ensure get single user behaves correctly. ... ok
test_single_user_incorrect_id (test_users.TestUserService)
Ensure err is thrown if given id does not exist ... ok
test_single_user_not_num_id (test_users.TestUserService)
Ensure err is thrown if given id is not number ... ok

----------------------------------------------------------
Ran 11 tests in 2.519s

OK

{{< /highlight >}}

最后是获取所有用户，同样是 GET 操作，访问 endpoint /users 时，借助 Model 的 API 查询到所有记录，转换格式后返回即可。开发流程上仍然是先写测试，再实现代码。另外，创建一个新指令 seed_db，用于插入 2 条种子记录到数据库中。

{{< highlight console>}}
$ docker-compose run users-service flask recreate_db
$ docker-compose run users-service flask seed_db
$ curl http://$(docker-machine ip vdev):5001/users
{{< /highlight >}}
{{< highlight json>}}
{
  "data": {
    "users": [
      {
        "created_at": "Sun, 12 Nov 2017 02:02:41 GMT",
        "email": "me@wushuzh.com",
        "id": 1,
        "username": "me"
      },
      {
        "created_at": "Sun, 12 Nov 2017 02:02:41 GMT",
        "email": "me2@wushuzh.io",
        "id": 2,
        "username": "me2"
      }
    ]
  },
  "status": "success"
}
{{< /highlight >}}




参考文档

> - [Modular Applications with Blueprints](http://flask.pocoo.org/docs/0.12/blueprints/)

封面图片来自 [Blueprint-Text](https://dribbble.com/shots/2783995-Blueprint-Text) <a href="https://dribbble.com/andreimarius"><i class="fa fa-dribbble" aria-hidden="true"></i> Andrei Marius</a>  
