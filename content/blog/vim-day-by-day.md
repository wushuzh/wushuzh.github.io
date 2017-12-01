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

{{< highlight console >}}
$ pacman -Qs -q vim
gvim
neovim-git
oni
vim-runtime
{{< /highlight >}}

Nicola Paolucci 发表了博客 (2016-02-17) [The best way to store your dotfiles: A bare Git repository](https://developer.atlassian.com/blog/2016/02/best-way-to-store-dotfiles-git-bare-repo/) 介绍在从 Hacker News 上了解到 StreakyCobra 分享的[管理私有配置的思路](https://news.ycombinator.com/item?id=11071754) ，他的迁移过程。

{{< highlight console >}}
$ git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim
$ vim ~/.vimrc
{{< /highlight >}}

[Fugitive.vim - a complement to command line git](http://vimcasts.org/episodes/fugitive-vim---a-complement-to-command-line-git/)

### 特殊字符

[Show invisibles](http://vimcasts.org/episodes/show-invisibles/)

- 显示不可见字符 tab: ^I eol: $
- 更改默认不可见字符的显示符号 tab ▸ (U+25B8) eol ¬ (U+00AC)
- 如何输入 Unicode 字符

[Tabs and Spaces](http://vimcasts.org/episodes/tabs-and-spaces/):

### 折叠



参考文档

> - Drew Neil [Vim Casts](http://vimcasts.org/episodes)
> - Greg Hurrel [Vim screencast](https://www.youtube.com/playlist?list=PLwJS-G75vM7kFO-yUkyNphxSIdbi_1NKX)
> - Archlinux wiki [dotfiles](https://wiki.archlinux.org/index.php/Dotfiles)

封面图片来自 [Up & Running with Vim](https://dribbble.com/shots/2668712-Up-Running-with-Vim) <a href="https://dribbble.com/federicafragapane"><i class="fa fa-dribbble" aria-hidden="true"></i> Federica Fragapane</a>
