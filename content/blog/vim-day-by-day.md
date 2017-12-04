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
oni
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
- [#35 exploring the history of a git repo](http://vimcasts.org/episodes/fugitive-vim-exploring-the-history-of-a-git-repository/)


### 特殊字符

[#1 Show invisibles](http://vimcasts.org/episodes/show-invisibles/)

- 显示不可见字符 tab: ^I eol: $
- 更改默认不可见字符的显示符号 tab ▸ (U+25B8) eol ¬ (U+00AC)
- 如何输入 Unicode 字符

[#2 Tabs and Spaces](http://vimcasts.org/episodes/tabs-and-spaces/):

### 折叠

[#37 How to fold](http://vimcasts.org/episodes/how-to-fold/)


参考文档

> - Drew Neil [Vim Casts](http://vimcasts.org/episodes)
> - Greg Hurrel [Vim screencast](https://www.youtube.com/playlist?list=PLwJS-G75vM7kFO-yUkyNphxSIdbi_1NKX)
> - Archlinux wiki [dotfiles](https://wiki.archlinux.org/index.php/Dotfiles)
> - Jeff Kreeftmeijer [Vim's absolute, relative and hybrid line numbers](https://jeffkreeftmeijer.com/vim-number/)

封面图片来自 [Up & Running with Vim](https://dribbble.com/shots/2668712-Up-Running-with-Vim) <a href="https://dribbble.com/federicafragapane"><i class="fa fa-dribbble" aria-hidden="true"></i> Federica Fragapane</a>
