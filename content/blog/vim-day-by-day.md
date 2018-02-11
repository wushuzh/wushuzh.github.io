+++
date = "2017-11-29T11:37:35+08:00"
title = "Vim 学习"
showonlyimage = false
image = "/img/blog/vim-day-by-day/vim.png"
draft = false
weight = 1000
+++

珍爱生命，每天学一点 vim 技巧
<!--more-->

### 版本控制

{{< highlight console >}}
$ pacman -Qs -q vim
gvim
neovim-git
nerd-fonts-complete
oni
python-neovim
python2-neovim
vim-runtime
{{< /highlight >}}

Nicola Paolucci 发表了博客 (2016-02-17) [The best way to store your dotfiles: A bare Git repository](https://developer.atlassian.com/blog/2016/02/best-way-to-store-dotfiles-git-bare-repo/) 介绍在从 Hacker News 上了解到 StreakyCobra 分享的[管理私有配置的思路](https://news.ycombinator.com/item?id=11071754) ，他的迁移过程。这种做法的优势有

- 无需其他工具和软连接，将文件置于版本控制之下
- 不同的机器使用不同的分支
- 在新安装的机器上配置简单

下面的指令只需在首次创建仓库时执行一次: 设定 git 别名 config 专用于操作此仓库，设定此仓库仅显示明确要求追踪文件的状态。

{{< highlight bash >}}
git init --bare $HOME/.cfg
alias config='/usr/bin/git --git-dir=$HOME/.cfg/ --work-tree=$HOME'
config config --local status.showUntrackedFiles no
echo "alias config='/usr/bin/git --git-dir=$HOME/.cfg/ --work-tree $HOME'" >> $HOME/.bashrc
{{< /highlight >}}

> 为 vimrc 1) 修改设置快捷键 2) 自动生效新设置 见[#24 Updating your vimrc file on the fly](http://vimcasts.org/episodes/updating-your-vimrc-file-on-the-fly/)

### 阅读文档

RTFM 是 Read The Fucking Manual 的缩写。无论是 vim 核心还是拓展插件都有使用文档，逐字逐句地阅读相关手册其实是更有效快捷的学习方法。没有网络的时候你就可以在 vim 中敲入 :h 获得帮助文档。

### 插件管理

{{< highlight console >}}
$ git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
$ vim ~/.vimrc
# ask vundle to install tpope/vim-fugitive

$ config add .vimrc
$ config st
$ config commit -m "add init vimrc"
$ config remote add origin https://github.com/wushuzh/.cfg.git
$ config push -u origin master

{{< /highlight >}}

按照 [VimAwesome](https://vimawesome.com) 排名，首先通过 Vundle 安装 Fugitive 插件。而基本使用可以观看

- [#31 a complement to command line git](http://vimcasts.org/episodes/fugitive-vim---a-complement-to-command-line-git/)
- [#32 working with the git index](http://vimcasts.org/episodes/fugitive-vim-working-with-the-git-index/)
- [#33 resolving merge conflicts with vimdiff](http://vimcasts.org/episodes/fugitive-vim-resolving-merge-conflicts-with-vimdiff/)
- [#34 browsing the git object database](http://vimcasts.org/episodes/fugitive-vim-browsing-the-git-object-database/)
- [#35 exploring the history of a git repo](http://vimcasts.org/episodes/fugitive-vim-exploring-the-history-of-a-git-repository/) Glog 指令配合 UNIMPAIRED 插件能很方便地启动一个新 tab 页，查看该文件的所有历史修改版本

:h vundle
:h fugitive

### 编辑

基于 X11 的系统有两个独立的剪贴板:

- PRIMARY: 通过框选复制，使用鼠标的中间粘贴，访问可用 “*
- CLIPBOARD: 和 MS Windows 中的系统剪贴版同义，访问可用 "+

插件 Goyo (韩语中宁静之意) 可用于 Markdown 写作

### 特殊字符

插件 vim-airline 在编辑器底部显示当前文件信息: 模式、版本、文件名、文件类新、编码、位置和警告信息。

插件 vim-devicons 可以为不同类型文件加上图表，但安装前要对字体修正，可直接安装 nerd-fonts-complete ，并在 .vimrc 中对双字节字符(中文)使用特定字体

{{< highlight console >}}
$ cower -d nerd-fonts-complete
$ fc-list :lang=zh
{{< /highlight >}}

{{< highlight vim >}}
if has('gui_running')
  set background=light
  set lines=999 columns=999
  set guifont=DroidSansMono\ Nerd\ Font\ 11
  set guifontwide=Source\ Han\ Sans\ 12
else
  set background=dark
endif
{{< /highlight >}}

> :h air-line

[#1 Show invisibles](http://vimcasts.org/episodes/show-invisibles/)

- 将不可见字符 tab: ^I eol: $ 改为 ▸ (U+25B8) ¬ (U+00AC)
- 用 ctrl-vu 数字 插入 Unicode 字符
- 将 solarized 配色方案中 solarized_visibility 设为 low，淡化这类字符的显示

[#2 Tabs and Spaces](http://vimcasts.org/episodes/tabs-and-spaces/):

### 打开文件

用于查找文件的插件 [ctrlp.vim](https://vimawesome.com/plugin/ctrlp-vim-red) 可以通过文件名模糊匹配定位所需文件。详见 :h ctrlp

依赖视觉的查找其实效率不高，因此 nerdtree 仅被用文件系统添改删操作。详见 :h nerdtree

EasyMotion 是一个依赖视觉的瞬移光标插件, 敲入 <Leader><Leader>w 就会触发用于跳转的提示字母。详见 :h easymotion.txt


### 折叠

[#37 How to fold](http://vimcasts.org/episodes/how-to-fold/)

### 前端

Emmet 支持所有主流编辑器，是网页开发的利器:

- 快速插入 html 模板，标签可附加 #id 及 .class
- 生成插入 n 个单词的随机假文 lorem[n]
- 同层: + 子层: > 父层: ^ 递增编号: $
- <c-y>comma 拓展, <c-y>/ 注释

:h emmet

### 编码

Syntastic 插件可调用各种不同的语法检查器提示代码中错误和潜在问题，和 vim-airline 自动集成，无需配置。详见 :h syntastic

{{< highlight console >}}
$ pacman -Qs lint
local/eslint 4.13.1-1
    An AST-based pattern checker for JavaScript
local/python-pyflakes 1.6.0-1
    A lint-like tool for Python to identify common errors quickly without executing code
{{< /highlight >}}

surround.vim 插件用于处理小括号、中括号、单双引号、XML 标签。假设已有 “Hi Vim!" 常见操作:

- 处于单词中，敲入 cs“‘ 即更改引用——双引号变单引号
- 继续敲入 cs’<q> 将单引号改为标签引用
- 继续敲入 cst" 将标签改为双引号，恢复最初状态
- 继续敲入 ds" 删除引用 ...
- ysiw" 将光标所在单词用双引号括起
- yss"  将光标所在整行用双引号括起

vim-javascript (TODO)

### 迁移到 neovim

安装 AUR neovim-git，是为了和最新版本同步，每次同步时借用 AUR 的 makepkg 脚本克隆最新代码、编译、打包、安装即可。

另一个 AUR neovim-gnome-terminal-wrapper 则会在一个无菜单的 gnome-terminal windows 中启动 neovim。

为了让 vim 和 neovim 使用相同的配置文件，可为 neovim 做如下配置:

{{< highlight vim >}}
" ~/.config/nvim/init.vim
set runtimepath+=~/.vim,~/.vim/after
set packpath+=~/.vim
source ~/.vimrc
{{< /highlight >}}

neovim 的一个很吸引人的特性是拥有了内置的终端窗口。

- [#71 Meet Neovim](http://vimcasts.org/episodes/meet-neovim/)
- [#74 Neovim's terminal emulator](http://vimcasts.org/episodes/neovim-terminal/)
- [#75 Creating mappings for :terminal](http://vimcasts.org/episodes/neovim-terminal-mappings/)

neovim 依赖外部工具才能访问系统剪贴板，所以首先安装 xclip , 这样 + 就会直接和系统剪贴板绑定。详见:h clipboard

- [How can I copy text to the system clipboard from Vim](https://vi.stackexchange.com/questions/84/how-can-i-copy-text-to-the-system-clipboard-from-vim)

参考文档

> - Drew Neil [Vim Casts](http://vimcasts.org/episodes)
> - Greg Hurrel [Vim screencast](https://www.youtube.com/playlist?list=PLwJS-G75vM7kFO-yUkyNphxSIdbi_1NKX)
> - Archlinux wiki [dotfiles](https://wiki.archlinux.org/index.php/Dotfiles)
> - Jeff Kreeftmeijer [Vim's absolute, relative and hybrid line numbers](https://jeffkreeftmeijer.com/vim-number/)
> - Archlinx wiki [Fonts](https://wiki.archlinux.org/index.php/Fonts)
> - 希锐亚 (2011-06-20) [在Gvim中给中文英文设置不同的字体](http://xxxcjr.blogspot.fi/2011/06/gvim.html)
> - Marco Hinz [vim-galore](https://github.com/mhinz/vim-galore)
> - [UPCASE](https://thoughtbot.com/upcase/vim)
> - [VimGolf](http://www.vimgolf.com/)

封面图片来自 [Up & Running with Vim](https://dribbble.com/shots/2668712-Up-Running-with-Vim) <a href="https://dribbble.com/federicafragapane"><i class="fa fa-dribbble" aria-hidden="true"></i> Federica Fragapane</a>
