+++
date = "2018-02-01T20:19:27+08:00"
title = "Flask 教程3"
showonlyimage = false
image = "/img/blog/usr-login-and-register/pingu-login.png"
draft = false
weight = 603
+++

flask 教程: 创建一个类微博原型站点
<!--more-->

## 密码散列

{{< highlight pycon >}}
>>> u = User(username='susan', email='susan@example.com')
>>> u.set_password('supasswd')
>>> u.check_password('notsupasswd')
False
>>> u.check_password('supasswd')
True

{{< /highlight >}}

## 密码验证

首先安装 Flask-Login 拓展并像其他拓展一样进行初始化，只有用它来管理用户登录状态。

登录必然和 User 模型联合使用，只要实现了下面三个属性和一个方法, 我们就可以利用 Flask-Login 设计登录了:

- is_authenticated：布尔值标识用户是否有合法鉴权;
- is_active 布尔值标识账户是否激活;
- is_anonymous 布尔值标识否为特有匿名用户;
- get_id() 函数获取用户唯一标识;

我们可以用 flask-login 自带的 minix 类中的通用实现来完成相关设定。

登录的流程:

1. 用户通过浏览器，提交含有用户名、密码等字段的表单给后台;
2. 在后端通过用户名获取其预设密码的散列，然后使用此次提交的密码验证;
3. 成功，将用户传入 login_user 函数并回主页，否则警告并重回登录页;

## 强制登录

当用户访问被保护页面时，Flask-Login 自动将用户重定向到登录页，并仅在鉴权成功后自动返回之前保护页面:

首先告知 Flask-Login 拓展哪个 view 函数负责处理登录;
之后那些需要登录查看的 URL 装饰器后面加入 `@login_required` 装饰器;
最后是在鉴权成功后将用户返回之前的页面;

## 模板中动态显示登录用户

{{< highlight pycon >}}
>>> u = User(username='susan', email='susan@example.com')
>>> u.set_password('supasswd')
>>>
>>> db.session.add(u)
>>> db.session.commit()
{{< /highlight >}}


## 用户注册


## 查看用户详情

和 weibo 或 twitter 类似，我们系统上的用户是可以互相查看用户详情页。

为此我们引入了带有动态组件的路由 `/user/<username>`。这样 Flask 就会把符合规则的路由,解析出动态部分，作为参数交给对应的 view 函数。

## 用户头像

借助 Gravatar 服务，我们通过用户邮箱的 md5 散列获取其头像。

参考文档

Miguel Grinberg

> - (2018-01-03) [The Flask Mega-Tutorial Part Ⅴ: User Logins](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-v-user-logins)
> - (2018-01-10) [The Flask Mega-Tutorial Part Ⅵ: Profile Page](https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-vi-profile-page-and-avatars)

封面图片来自 [Pingu - Login Animation GIF](https://dribbble.com/shots/3064060-Pingu-Login-Animation-GIF) <a href="https://dribbble.com/vineetarora94"><i class="fa fa-dribbble" aria-hidden="true"></i> Vineet Arora</a>
