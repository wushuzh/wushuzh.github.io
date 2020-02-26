+++
date = "2019-12-02T17:49:58+08:00"
title = "ARTS 2019w49"
image = "/img/blog/arts-76-per-week/"
draft = true
weight = 1949
tags = ["ARTS"]
+++


<!--more-->

## Algorithm

## Review

## Tip

sudo pacman -S vagrant
vagrant plugin install vagrant-libvirt

mkdir pxe
vagrant init centos/7
vagrant up --provider=libvirt

禁止检查更新
  config.vm.box_check_update = false

使用宿主机网络
  config.vm.network :public_network,
    :dev => "virbr0",
    :mode => "bridge",
    :type => "bridge"



## Share



封面图片来自 []() <a href="h"><i class="fa fa-dribbble" aria-hidden="true"></i> </a>