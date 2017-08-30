+++
date = "2017-08-29T22:07:31+08:00"
title = "Archlinux 中文"
showonlyimage = false
image = "/img/blog/dual-os-4-i18n-arch/journey-chinese-chars.png"
draft = false
weight = 65
+++

重点是在 Archlinux Gnome 中显示和输入中文
<!--more-->

## 扩展 Chrome

["上篇"]({{< relref "blog/dual-os-3-setup-arch.md" >}})
的最后安装了 Chrome 浏览器，为了能上网，首先要正确的设置代理。如果设置浏览器代理的时候发现这样的报错信息```The NetworkManager needs to be running```，则需要确认系统是否安装并启动了相应的网络管理服务。

{{< highlight console >}}
pacman -Q networkmanager
systemctl status NetworkManager
{{< /highlight >}}

为了让浏览掐用的更顺手，还需要安装一些扩展插件。
> 如果你的 Chrome 已经用的很顺手，那 google 自己的浏览器同步服务可以很好的将你的书签，浏览历史，安装插件都无差别的同步到新登录的浏览器上。

- [SwitchOmega](https://github.com/FelisCatus/SwitchyOmega) 我都没注意 [switchysharp](https://github.com/FelisCatus/switchysharp) 已经不再维护，这是它的升级版
- [OneTab](https://www.one-tab.com/) 将打开的一堆 tab 页的 URL 保存一下，瞬间减少 9 成以上的内存
- [Fireshot](https://getfireshot.com/) 一个做网页截图、标注的拓展工具
- [Vimium](http://vimium.github.io/) 能让你使用 Vim 的快捷键完成最常见的浏览器控制，上下翻页，Tab 切换
- [Momentum](https://momentumdash.com/) 每天一幅风景画
- [World Clocks](https://chrome.google.com/webstore/detail/world-clocks/innfmeekncjandlanpgdmmogkcimekgo) 如果需要查看多个时区时间
- [Google Dict](https://chrome.google.com/webstore/detail/google-dictionary-by-goog/mgijmajocgfcbeboacabfgobmjgjcoja) 有时候确实会遇到不认识的各国单词
- [Octotree](https://github.com/buunguyen/octotree) 通过树形结构查看 github 项目的文件，挺方便
- [Markdown Here](http://markdown-here.com/) 将 Markdown 文本转化为网页
- [Block U from GA](https://www.igorware.com/extensions/block-yourself-from-analytics) 让 Google Analyze 忽略来自自己的访问

<br />

## 中文显示

首先查看任何一个含有中日韩的网页，如果你看到乱码，则说明你的系统中还未安装中文字体。

> 关于字体知识的拓展阅读，推荐收听不定期更新的 podcast [内核恐慌](https://kernelpanic.fm/) 上 [字谈字串(一)](https://kernelpanic.fm/39) 等过往节目。

在 Archlinux Wiki 的 [Font](https://wiki.archlinux.org/index.php/Fonts) 页面中非拉丁字符中挑选自己喜欢的字体。比如我选择了一揽子解决方案 Adobe 的 [Source Han Sans](https://github.com/adobe-fonts/source-han-sans)

{{< highlight bash >}}
$ sudo pacman -S adobe-source-han-sans-otc-fonts
{{< /highlight >}}

<br />

## 中文输入

在之后就该考虑中文的输入了。首先要从[Fcitx (Flexible Input Method Framework)](https://wiki.archlinux.org/index.php/Fcitx)、[IBus (Intelligent Input Bus)](https://wiki.archlinux.org/index.php/IBus)、[SCIM (Smart Common Input Method platform)](https://wiki.archlinux.org/index.php/Smart_Common_Input_Method_platform) 三种常见的输入法框架中选出一个。

Fcitx 由国内的开发者发明维护。这次试用了一下，除了首次安装时需要折腾一下，后期日常使用起来，无论从调用时输入法切换速度，到键入拼音后备选词列表的显示，体验都很不错。

如果你系统中尚未安装、激活其他的输入法框架，比如 ibus，那 fcitx 对应的安装、配置应该很简单直接。执行完 pacman ，重新进入 GNOME 下应该能看到 fcitx-configtool 处于界面的左下角——fcitx 似乎没有系统托盘的图标。一开始只能看到英语作为唯一的输入法，点击加号通过关键字 Pinyin 搜索输入法。选中后就应该能生效了。

> - 我习惯 startx 式启动，还要在 ```/etc/enviroment``` 中添加环境变量，明确将 fcitx 指定为 GTK、QT 下的输入法模块  
> - fcitx wiki 官网上[对于 Gnome 3.6 版本以上的相关建议](https://fcitx-im.org/wiki/Note_for_GNOME_Later_than_3.6)也可以尝试一下(除了第一步)

{{< highlight bash >}}
$ sudo pacman -Syu fcitx-im fcitx-configtool
# when U in trouble, exec diagnosis tool and focus on red lines
$ fcitx-diagnose
{{< /highlight >}}

<br />

### 文本编辑器

作为一名不太坚定的 vim 用户。我优先安装了 atom ，并且我发现年纪越大，对这种将那些最基本的设置在出厂时就配置个八九不离十的编辑器有了由衷的好感。唉，是不是老了……下定了半点决心，还是决定安装 oni : 一个尚处于初级阶段，但集成了前端和常用插件的 neovim

{{< highlight bash >}}
$ sudo pacman -S atom

$ cd /tmp
$ git clone https://aur.archlinux.org/oni.git
$ cd oni
$ makepkg -si

$ sudo pacman -S hugo gimp ...

{{< /highlight >}}

<br />

### 参考文档

> - Zhang Fan (2015-10-10) [Installing fcitx (a Chinese IME) on Arch Linux](http://www.fanz.io/2015/10/10/fcitx-notes.html)

封面图片来自 [A Journey of Chinese Characters](https://dribbble.com/shots/3588846-A-Journey-of-Chinese-Characters) <a href="https://dribbble.com/siyangli"><i class="fa fa-dribbble" aria-hidden="true"></i> Siyang Li</a>  
