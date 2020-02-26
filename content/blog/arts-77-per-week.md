+++
date = "2019-12-02T17:49:58+08:00"
title = "ARTS 2019w50"
image = "/img/blog/arts-77-per-week/"
draft = true
weight = 1950
tags = ["ARTS"]
+++


<!--more-->

## Algorithm

## Review

## Tip

```
The provider 'virtualbox' that was requested to back the machine
'default' is reporting that it isn't usable on this system. The
reason is shown below:

Vagrant has detected that you have a version of VirtualBox installed
that is not supported by this version of Vagrant. Please install one of
the supported versions listed below to use Vagrant:

4.0, 4.1, 4.2, 4.3, 5.0, 5.1, 5.2, 6.0

A Vagrant update may also be available that adds support for the version
you specified. Please check www.vagrantup.com/downloads.html to download
the latest version.
```

choco uninstall virtualbox
choco install virtualbox --version 6.0.14



mkdir pxe
vagrant init centos/7
vagrant up --provider=virtualbox

bcdedit /set hypervisorlaunchtype off

sudo -i
 yum install tftp-server dhcp xinetd

 vi dhcpd.conf

cd ~
mkdir rpmfile
yum install --downloadonly --downloaddir=. shim-x64
yum install --downloadonly --downloaddir=. grub2-efi-x64

# legacy
 cp ./usr/share/syslinux/pxelinux.0 /var/lib/tftpboot/pxelinux
  mkdir /var/lib/tftpboot/pxelinux/pxelinux.cfg

  touch /var/lib/tftpboot/pxelinux/pxelinux.cfg/default

   mkdir -p /var/lib/tftpboot/images/centos-bios


# uefi

[root@localhost rpmfiles]# rpm2cpio shim-x64-15-2.el7.centos.x86_64.rpm | cpio -dimv
./boot/efi/EFI/BOOT/BOOTX64.EFI
./boot/efi/EFI/BOOT/fallback.efi
./boot/efi/EFI/BOOT/fbx64.efi
./boot/efi/EFI/centos/BOOT.CSV
./boot/efi/EFI/centos/BOOTX64.CSV
./boot/efi/EFI/centos/MokManager.efi
./boot/efi/EFI/centos/mmx64.efi
./boot/efi/EFI/centos/shim.efi
./boot/efi/EFI/centos/shimx64-centos.efi
./boot/efi/EFI/centos/shimx64.efi
15305 blocks


[root@localhost rpmfiles]# rpm2cpio grub2-efi-x64-2.02-0.80.el7.centos.x86_64.rpm | cpio -dimv
./boot/efi/EFI/centos
./boot/efi/EFI/centos/fonts
./boot/efi/EFI/centos/fonts/unicode.pf2
./boot/efi/EFI/centos/grubx64.efi
./boot/grub2/grubenv
./etc/grub2-efi.cfg
7133 blocks

[root@localhost rpmfiles]# cp boot/efi/EFI/centos/shim.efi /var/lib/tftpboot/
[root@localhost rpmfiles]# cp boot/efi/EFI/centos/grubx64.efi /var/lib/tftpboot/

[root@localhost tftpboot]# mkdir images
[root@localhost tftpboot]# cd images/
[root@localhost images]# mkdir centos7
[root@localhost images]# cd centos7/
[root@localhost centos7]# pwd
/var/lib/tftpboot/images/centos7
[root@localhost centos7]# curl -O -k http://mirrors.huaweicloud.com/centos/7/os/x86_64/images/pxeboot/initrd.img
[root@localhost centos7]# curl -O -k http://mirrors.huaweicloud.com/centos/7/os/x86_64/images/pxeboot/vmlinuz

## Share



封面图片来自 []() <a href="h"><i class="fa fa-dribbble" aria-hidden="true"></i> </a>