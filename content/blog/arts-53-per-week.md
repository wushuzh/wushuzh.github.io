+++
date = "2019-06-24T11:29:41+08:00"
title = "ARTS 2019w26"
image = "/img/blog/arts-53-per-week/"
draft = true
weight = 1926
tags = ["ARTS"]
+++


<!--more-->

## Algorithm

## Review


## Tip

### 在Linux下为 Chrome 浏览器设置代理

最近遇到一个问题，GNOME3 下首次使用 Chrome , 原生 Chrome 浏览器本身不提供代理的设置，而是依赖于桌面系统的相关设置， chrome 的插件(比如 proxy SwitchySharp)提供了设置，但无论是登录商店安装插件，还是登录谷歌账户来直接同步插件，都需要先设置代理。成了一个鸡生蛋、蛋生鸡的问题。

之所以对我来说曾为一个问题，主要是由于下面的限制：

- 我使用 systemd-networkd 而不是 NetworkManager 来管理网络，但 GNOME 的网络和代理设置都是通过 NetworkManager 服务完成;
- 我只知道可以设置 http(s)_proxy 的环境变量，但对于socks 代理应该设置哪个环境变量不清楚；

### 试了但不成功的方法

按照 [Proxy server wiki](https://wiki.archlinux.org/index.php/Proxy_server#Using_a_SOCKS_proxy)

- 安装配置 proxychains-ng, 出现 core dumped, 无法启动浏览器
- 安装配置 tsocks, 可以启动浏览器，但出现错误，无法通过代理上网

{{< highlight console >}}
[wushuzh@veronica ~]$ proxychains google-chrome-stable
[proxychains] config file found: /etc/proxychains.conf
[proxychains] preloading /usr/lib/libproxychains4.so
[proxychains] DLL init: proxychains-ng 4.14
......
Trace/breakpoint trap (core dumped)

wushuzh@veronica ~]$ tsocks google-chrome-stable
libtsocks: Unresolved symbol: close
......
[...] InitializeSandbox() called with multiple threads in process gpu-process.
[...] ERROR:ssl_client_socket_impl.cc(969)] handshake failed; returned -1, SSL error code 1, net_error -100
......
{{< /highlight >}}

### 可行的方法：

通过 gsettings 无需 NetworkManager 即可[设置代理](https://developer.gnome.org/ProxyConfiguration/)

{{< highlight console >}}
# set socks server
gsettings set org.gnome.system.proxy mode 'manual'
gsettings set org.gnome.system.proxy.socks host '127.0.0.1' 
gsettings set org.gnome.system.proxy.socks port 10808
# restore the default value
gsettings set org.gnome.system.proxy mode 'none'
gsettings set org.gnome.system.proxy.socks host ''
gsettings set org.gnome.system.proxy.socks port 0
{{< /highlight >}}

## Share


封面图片来自 []() <a href="h"><i class="fa fa-dribbble" aria-hidden="true"></i> </a>