+++
date = "2017-06-30T19:19:19+08:00"
title = "看不懂的报错，咋整？"
showonlyimage = false
image = "/img/blog/deal-with-wired-errmsgs/pc-gone-2.png"
draft = false
weight = 41
tags = [ "kvm", "Linux" ]
+++

追踪本质，别被报错信息带歪了方向……
<!--more-->

### 现象分析

Lucy 的服务器才从内核恐慌中恢复过来，接着创建虚拟机的时候，出现错误，对应报错信息邮件转发如下：

{{< highlight console >}}
```
Traceback (most recent call last):
  File "/usr/share/virt-manager/virtManager/asyncjob.py", line 44,
    in cb_wrapper callback(asyncjob, *args, **kwargs)
  File "/usr/share/virt-manager/virtManager/create.py", line 1932,
    in do_install guest.start_install(False, meter=meter)
  File "/usr/lib/python2.6/site-packages/virtinst/Guest.py", line 1229,
    in start_install noboot)
  File "/usr/lib/python2.6/site-packages/virtinst/Guest.py", line 1297,
    in _create_guest dom = self.conn.createLinux(start_xml or final_xml, 0)
  File "/usr/lib64/python2.6/site-packages/libvirt.py", line 2686,
    in createLinux if ret is None:
      raise libvirtError('virDomainCreateLinux() failed', conn=self)
libvirtError: Unable to read from monitor: Connection reset by peer
```
{{< /highlight >}}

服务器是一台 KVM 宿主服务器，virsh 查看正在运行的虚拟机一切正常。Lucy 并没说安装那台机器，**但查问题总需要一个问题重现步骤** ，好在通过 kickstart 创建安装一台虚拟机也不麻烦，直接试一下。下面的步骤将触发一个标准的 kickstart 安装，并将安装过程直接输出到终端，供用户监控安装过程。

### 尝试问题重现

{{< highlight bash >}}
qemu-img create -f qcow2 wushuzh.qcow2 10G

# prepare a kickstart file, a few notes:
#  1. set cdrom as the installation media
#  2. set text so no vnc/gui is needed
#  3. use dhcp for network (activate) , set local ntp
#  4. set hostname and crypted passwd of root
#  5. use autopart and minimal installation to speed up progress

virt-install --name wushuzh  --vcpus=2 --ram 4096 \
  --os-type=linux --os-variant rhel7 \
  --disk /kvm/vmimg/wushuzh.qcow2,device=disk,bus=virtio \
  --location /repo/isos/rhel-server-7.3-x86_64-dvd.iso \
  --extra-args="ks=file:/wushuzh.ks console=tty0 console=ttyS0,115200n8" \
  --network network:default --initrd-inject=wushuzh.ks  --nographics
{{< /highlight >}}

结果安装一切顺利，虚拟机成功启动。

{{< highlight bash >}}
# consider add this into /etc/profile
export LIBVIRT_DEFAULT_URI=qemu:///system

virsh console wushuzh

{{< /highlight >}}

### 深入查找日志

可以断定，这台服务器上的基本 libvirtd 服务没问题。产生问题可能来自不当的配置执行，或是特定的环境设置。

本文开头的报错信息确实没什么用，看不出是哪台虚拟机报错，如何创建的。网上查了一下类似的报错，比如[这个](https://bugs.launchpad.net/nova/+bug/1255624)，建议查看 ```/var/log/libvirt/qemu/$GUESTNAME.log``` 中具体的日志。

带着这个信息，去问了下 Lucy ，观看了一下她的操作过程：通过 virt-manager 图形界面安装服务器。全部配置完成，在最后点击安装的时候，界面出现了错误。赶快去看一下相应的 libvirt 日志。

{{< highlight console >}}
[root@kvmhost qemu]# tail -4 some-vmname.log
Domain id=28 is tainted: host-cpu
char device redirected to /dev/pts/11
2017-06-28T05:57:05.872745Z qemu-kvm:
  -drive file=/some-base-folder/some-iso,
    if=none,media=cdrom,id=drive-ide0-1-0,readonly=on,format=raw:
  could not open disk image /some-base-folder/some-iso: Permission denied
2017-06-28 05:57:06.188+0000: shutting down

{{< /highlight >}}

权限有问题，继续查看 /var/log/message 或 security，果然也有相应错误。

{{< highlight console >}}
[root@kvmhost log]# tail -3 /var/log/messages
Jun 28 13:57:13 kvmhost setroubleshoot:
  SELinux is preventing /usr/libexec/qemu-kvm from read access
    on the lnk_file releases. For complete SELinux messages.
      run sealert -l ccc9f19d-f2b2-4a1e-b991-af6de737f285
{{< /highlight >}}

这个错误由 selinux 导致，但这个设置在最初搭建此服务器的时候，创建第一台虚拟机的时候遇到过——selinux认为安装介质通过含有 link 的目录进行应用会产生潜在的安全问题——但当时我就添加了相应规则。但这次又一次出现，猜测可能是之前替换系统文件或 rpm 安装包，由此产生的非常规更新导致？

保险起见，又去查看了 ```/some-base-folder/some-iso``` 的权限，以及用 lsof 和 fuser 查看是否有其他用户和进程也在使用该文件，都没查到明显的问题。

这个错误的修复也很简单。root 按照日志中建议为本地建立放行规则即可。

{{< highlight console >}}

[root@kvmhost log]# sealert -l ccc9f19d-f2b2-4a1e-b991-af6de737f285
PuTTY X11 proxy: Unsupported authorisation protocol
SELinux is preventing /usr/libexec/qemu-kvm from read access
  on the lnk_file some-folder.

*****  Plugin catchall_labels (83.8 confidence) suggests  **********

If you want to allow qemu-kvm to have read access on the releases lnk_file
Then you need to change the label on releases
Do
# semanage fcontext -a -t FILE_TYPE 'releases'
where FILE_TYPE is one of the following: proc_t, mnt_t, usr_t,
user_home_type, virt_etc_rw_t, textrel_shlib_t, rpm_script_tmp_t,
home_root_t, configfile, user_home_dir_t, cert_type, device_t,
configfile, sysfs_t, sssd_var_lib_t, virt_var_lib_t, virt_var_run_t,
svirt_t, qemu_var_run_t, svirt_tmp_t, virt_content_t, public_content_t,
public_content_rw_t, device_t, abrt_t, etc_t, ld_so_t, proc_t,
lib_t, sysfs_t, root_t, svirt_image_t, svirt_tmpfs_t, virt_image_t,
user_home_t, usr_t, base_ro_file_type, userdomain, device_t, devlog_t,
locale_t, etc_t, bin_t, sysfs_t, sysfs_t, usbfs_t.
Then execute:
restorecon -v 'some-folder'

*****  Plugin catchall (17.1 confidence) suggests  *****************

If you believe that qemu-kvm should be allowed read access
  on the some-folder lnk_file by default.
Then you should report this as a bug.
You can generate a local policy module to allow this access.
Do
allow this access for now by executing:
# grep qemu-kvm /var/log/audit/audit.log | audit2allow -M mypol
# semodule -i mypol.pp

Additional Information:
Source Context                system_u:system_r:svirt_t:s0:c282,c505
Target Context                system_u:object_r:default_t:s0
Target Objects                releases [ lnk_file ]
Source                        qemu-kvm
Source Path                   /usr/libexec/qemu-kvm
Port                          <Unknown>
Host                          nio136
Source RPM Packages           qemu-kvm-0.12.1.2-2.503.el6_9.3.x86_64
Target RPM Packages
Policy RPM                    selinux-policy-3.7.19-307.el6.noarch
Selinux Enabled               True
Policy Type                   targeted
Enforcing Mode                Enforcing
Host Name                     kvmhost
Platform                      Linux kvmhost kernel version etc
Alert Count                   2
First Seen                    Wed 28 Jun 2017 01:57:05 PM CST
Last Seen                     Wed 28 Jun 2017 01:57:05 PM CST
Local ID                      ccc9f19d-f2b2-4a1e-b991-af6de737f285

Raw Audit Messages
type=AVC msg=audit(1498629425.872:3540): avc:  
denied  { read } for  pid=11332 comm="qemu-kvm" name="some-folder"
dev=sdd3 ino=1572866
  scontext=system_u:system_r:svirt_t:s0:c282,c505
  tcontext=system_u:object_r:default_t:s0 tclass=lnk_file

type=SYSCALL msg=audit(1498629425.872:3540): arch=x86_64 syscall=open
 success=no exit=EACCES a0=7f255606c6c0 a1=81000 a2=0 a3=40 items=0
 ppid=1 pid=11332 auid=4294967295 uid=107 gid=107 euid=107 suid=107
 fsuid=107 egid=107 sgid=107 fsgid=107 tty=(none) ses=4294967295
 comm=qemu-kvm exe=/usr/libexec/qemu-kvm
   subj=system_u:system_r:svirt_t:s0:c282,c505 key=(null)

Hash: qemu-kvm,svirt_t,default_t,lnk_file,read

audit2allow

#============= svirt_t ==============
allow svirt_t default_t:lnk_file read;

audit2allow -R

#============= svirt_t ==============
allow svirt_t default_t:lnk_file read;


{{< /highlight >}}

最后查看一下安装中 ks 文件 nfs 共享是否依然生效。```showmount -e localhost``` 更多参见 [鸟哥的 linux 私房菜 NFS 章节](http://cn.linux.vbird.org/linux_server/0330nfs_2.php)

## 经验总结

1. 上报问题确实应该简洁，但需要包括最基本的问题重现的命令或操作
2. 明面上看到的报错信息，可能和最终产生错误原因相距甚远，了解去何处查找相关日志非常重要。
3. 虚拟机系统需要对各个用到的部件都有所了解才能定位问题，不然盲人摸象。比如 NFS， libvirt，virsh，tftp，nfs，甚至 selinux 需要好好了解学习一下。

参考文档

> - Superuser [How can I determine what process has a file open in Linux?]( https://superuser.com/questions/97844/how-can-i-determine-what-process-has-a-file-open-in-linux)
> -  LAKSHMANAN G. (2012-08-29)  [Linux lsof Command Examples (Identify Open Files)](http://www.thegeekstuff.com/2012/08/lsof-command-examples/)
> - https://www.cyberciti.biz/faq/howto-linux-get-list-of-open-files/
> - nixCraft (2008-03-05) [List Open Files for Process]( https://www.cyberciti.biz/faq/how-to-linux-re-apply-or-restore-selinux-security-labels-context/)
> - [鸟哥的 linux 私房菜 SELinux 管理原则] (http://cn.linux.vbird.org/linux_server/0210network-secure_4.php)


封面图片来自 [I'm a computa](https://dribbble.com/shots/1606433-I-m-a-computa) <a href="https://dribbble.com/Tymn"><i class="fa fa-dribbble" aria-hidden="true"></i> Tymn Armstrong</a>  

其他图片来源网络
