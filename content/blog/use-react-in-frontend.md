+++
date = "2017-12-07T22:01:09+08:00"
title = "Flask 开发 4"
showonlyimage = false
image = "/img/blog/use-react-in-frontend/react.png"
draft = false
weight = 602
tags = ["flask", "react"]
+++

从现在开始 React 复杂前端
<!--more-->

React 是针对 MVC 中 View 的解决方案。重要概念: 组件和它的属性及状态。组件主要的生命周期有 render、getInitialState、getDefaultProps、componentWillMount、componentDidMount

Jumbotron

### Setup React

{{< highlight console >}}
$ yarn global add create-react-app
success Installed "create-react-app@1.4.3" with binaries:
      - create-react-app

$ yarn global list
info "create-react-app@1.4.3" has binaries:
   - create-react-app

$ yarn global bin
$ mkdir learn-react && cd learn-react
$ ~/.yarn/bin/create-react-app fb-tutorial

{{< /highlight >}}

> 顺便安装 Facebook 的 eslint 配置 [eslint-config-fbjs](https://www.npmjs.com/package/eslint-config-fbjs)

### first Compontent JSX

### Class-based Component

### Ajax call to Flask App


参考文档

> - React [Hello world](https://reactjs.org/docs/hello-world.html) [Tutorial](https://reactjs.org/tutorial/tutorial.html)
> - MDN [重新介绍 JavaScript](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/A_re-introduction_to_JavaScript)
> - 阮一峰 (2014-04-30) [《ECMAScript 6入门》上线了]()
> - 阮一峰 (2016-09-23) [React 技术栈系列教程](http://www.ruanyifeng.com/blog/2016/09/react-technology-stack.html)


封面图片来自 [React Illustration](https://dribbble.com/shots/2484828-React-Illustration) <a href="https://dribbble.com/jucha"><i class="fa fa-dribbble" aria-hidden="true"></i> Justas Galaburda</a>
