+++
date = "2018-04-16T22:45:31+08:00"
title = "视频下载"
showonlyimage = false
image = "/img/blog/download-vids-using-youtube-dl/ladder.png"
draft = false
weight = 68
tags = ["tools"]
+++

油管视频下载和播放的利器
<!--more-->

## 工具安装

{{< highlight console >}}
pacman -S youtube-dl

youtube-dl -o '%(playlist)s/%(playlist_index)s - %(title)s.%(ext)s' --all-subs URL_youtube_playlist
{{< /highlight >}}

youtube-dl 默认先分别下载最高品质的视频和音频文件，然后将二者合并为一个文件。

youtube-dl 有很多可选参数，对我比较有用的是下载字幕，甚至可以将字幕合并到最终文件中。

常用的下载配置可以保存在用户配置目录中。

## 播放器

{{< highlight console >}}
$ sudo pacman -S vlc

PS> choco install vlc
{{< /highlight >}}

[VLC](https://en.wikipedia.org/wiki/VLC_media_player) 是一个跨平台免费开源的媒体播放器，内置了大量开源的编解码库，其基于包的播放特性使得它甚至可以播放正在下载或部分损坏的文件。借助 VLC 你甚至做下面的事情:

- 可以做各种格式转化;
- 视频音频录制、录屏、录制摄像头当前采集内容;
- 获取流媒体的详细信息，从而获得可通过浏览器下载的链接;

快捷键:

- 调整播放速率 `[` 减速 `]`加速 `=` 恢复正常速度
- `space` 暂停、`m` 静音、`v` 字幕切换
- `f` 全屏、`t` 已播放时间和总时间

https://github.com/siomiz/SoftEtherVPN
https://github.com/hwdsl2/docker-ipsec-vpn-server
https://github.com/hwdsl2/setup-ipsec-vpn/issues/158
https://github.com/haoel/haoel.github.io
https://www2.linode.com/stackscripts/view/237943


### 参考文档

> - alimiracle (2015-10-01) [Download YouTube Vides in Linux Command Line](https://itsfoss.com/download-youtube-linux/)

封面图片来自 [Cut-paper](https://dribbble.com/shots/3790618-Cut-paper) <a href="https://dribbble.com/ccaldwell"><i class="fa fa-dribbble" aria-hidden="true"></i> Christopher Caldwell</a>
