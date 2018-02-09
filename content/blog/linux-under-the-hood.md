+++
date = "2018-02-08T10:39:04+08:00"
title = "Linux 内幕"
showonlyimage = false
image = "/img/blog/linux-under-the-hood/supercomputer.png"
draft = false
weight = 603
+++

深入探索 Linux 系统的各种细节
<!--more-->

lsmod
modinfo modename

grep -- sysconfig # enviroment variable
cat /proc/pid/cmdline
ls /proc/pid/fd/ # similiar to lsof

# system adm guide
less /usr/share/doc/pam-1.1.8/Linux-PAM_SAG.txt

static lib
dynamic lib
    ldd program
    /etc/ld.so.conf
    ldconfig -v # update lib cache

# system call

man man
man 2 intro
map 2 syscall
man 2 mmap
cat /proc/PID/maps

tracking the system call
strace ls
strace -c ls
strace ls 2>&1 | grep open  # strace output is consider as stderr

ltrace ls 2>&1 | less

# signal
man 7 signal

dd if=/dev/zero of=/dev/null &
kill -s USR1 $(pidof dd) # check progress of dd


参考文档

> -

封面图片来自 [Super Computer](https://dribbble.com/shots/1250466-Supercomputer) <a href="https://dribbble.com/illotv"><i class="fa fa-dribbble" aria-hidden="true"></i> ILLO</a>
