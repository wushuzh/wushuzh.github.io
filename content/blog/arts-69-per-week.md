+++
date = "2019-10-14T22:30:48+08:00"
title = "ARTS 2019w42"
image = "/img/blog/arts-69-per-week/"
draft = true
weight = 1942
tags = ["ARTS"]
+++


<!--more-->

## Algorithm

## Review

## Tip

github 的 [pages 服务](https://pages.github.com/)，可将按一定规定组织的普通文本自动转化为一个漂亮的网站。有时候需要在提交前在本地环境中预览网页效果，需要借助 [Jekyll](https://jekyllrb.com/) , 当前 Windows 10 WSL Debian 操作系统版本为 。相应的导致提供的配套软件的版本也不够新，很大可能会影响后续 rubygems 的安装。

    sudo apt update
    sudo apt install jekyll
    sudo apt install bundler

    cd rootdir-project

    cat Gemfile

    gem "github-pages", group: :jekyll_plugins

    bundle config path "$(ruby -e 'puts Gem.user_dir')"
    bundle install 

    bundler exec jekyll serve 

在 archlinux 下安装比较顺利，以下是安装过程。

- ruby 软件库不应安装在系统级别，避免对系统功能的影响
- bundle 用于管理项目的软集库依赖，但同样的，需要变更其安装位置限制在每个用户内部

    sudo pacman -S ruby
    edit .bash_profile for PATH to execute lib bin utility directly from shell
    gem env

    gem install bundle
    bundle config path "$(ruby -e 'puts Gem.user_dir')"

    enter a jekyll project and confirm Gemfile content

    bundle install
    bundle exec jekyll serve




## Share


封面图片来自 []() <a href="h"><i class="fa fa-dribbble" aria-hidden="true"></i> </a>