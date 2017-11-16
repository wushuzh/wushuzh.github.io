+++
date = "2017-09-11T23:00:07+08:00"
title = "搭建 py 开发环境"
showonlyimage = false
image = "/img/blog/get-start-with-py/python.png"
draft = true
weight = 102
+++

如何在不同的环境下自由穿行
<!--more-->

基本上全面参考了 Henrique Bastos (2017-01-07) [The definitive guide to setup my Python workspace](https://medium.com/@henriquebastos/the-definitive-guide-to-setup-my-python-workspace-628d68552e14) 。下面简略记录一下过程。

## 需求

- 同时能用 PY2 和 PY3，但要也能装其他版本(如 PyPy 和 Anaconda)
- 拥抱 PY3，设为默认解释器，但必要时要能回退到 PY2
- 配置一个可以同时和 PY2 和 PY3 的 Jupyter Notebook，并能检测其运行时的虚拟环境版本
- 分别配置 PY2 和 PY3 的两个 iPython，这样就不用在项目的虚拟环境中安装 iPython 了
- 将 Python 实现的常用程序全局有效，同时不污染系统的 Py 解释器和库
- 仅需一条命令就可以借助 virtualenvwrapper 切换各个开发项目的环境
- 保持上述配置的可维护性，别在 PATH 中更改太多东西

## pyenv

pyenv 的原理是借助在 PATH 前排插入的 [shim 层](https://en.wikipedia.org/wiki/Shim_(computing))，判定当前应该使用的 Python 版本，再把指令转给正确的 Python 程序。

{{< highlight console >}}
$ bower -d pyenv
$ bower -d pyenv-virtualenv
$ bower -d pyenv-virtualenvwrapper
# install one by one
# ...

$ mkdir ~/.ve
$ mkdir ~/pywork

{{< /highlight >}}

{{< highlight bash >}}
# add the following to end of .bashrc
export WORKON_HOME=~/.ve
export PROJECT_HOME=~/workspace
eval "$(pyenv init -)"
{{< /highlight >}}


{{< highlight console >}}
$ pyenv install 3.6.2
$ pyenv install 2.7.13
$ pyenv versions
* system (set by ~/.pyenv/version)
  2.7.13
  3.6.2
{{< /highlight >}}

## virtualenvwrapper

在 .bashrc 中激活插件，并重新进入 shell

{{< highlight bash >}}
# add the following to end of .bashrc
pyenv virtualenvwrapper_lazy
{{< /highlight >}}


参考文档

> - Full Stack Python [Development Environments](https://www.fullstackpython.com/development-environments.html)
> -
> - David Robinson (2017-09-06) [The Incredible Growth of Python](https://stackoverflow.blog/2017/09/06/incredible-growth-python/)
> - [Python 2 Death Clock](https://pythonclock.org/)

封面图片来自 [Wrapping Your Head Around Python](https://dribbble.com/shots/2758651-Wrapping-Your-Head-Around-Python) <a href="https://dribbble.com/whoiskatja"><i class="fa fa-dribbble" aria-hidden="true"></i> Kat Bak</a>
