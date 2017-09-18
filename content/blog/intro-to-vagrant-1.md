+++
date = "2017-09-15T20:55:02+08:00"
title = "虚拟环境实验厂"
showonlyimage = false
image = "/img/blog/dual-os-7-ve/400.png"
topImage = "/img/blog/dual-os-7-ve/400.gif"
draft = false
weight = 68
+++

利用 Vagrant 管理多种虚拟环境
<!--more-->

## 虚拟层

KVM 无疑是最流行的开源虚拟层。为 archlinux 配置好虚拟实验环境，能让我们以极低的成本模拟各类系统和应用。相关概念有 QEMU 和 libvirt，前者是比 KVM 更为通用的虚拟仿真层，后者则为它们提供了一个统一的包含命令行、API 的管理工具库。更多详情见:

- Peeyush Gupta (2014-02-14) [KVM vs QEMU vs Libvirt](http://thegeekyway.com/kvm-vs-qemu-vs-libvirt/)
- Sriram S (2014-03-10) [KVM and QEMU – do you know the connection?](http://www.innervoice.in/blogs/2014/03/10/kvm-and-qemu/)
- Serverfault ans of [Difference between KVM and QEMU](https://serverfault.com/a/208788)

只要宿主服务器支持虚拟化，VT-x 或 AMD-V，就意味着可以通过 KVM 为虚拟化加速。（ BIOS 中要启用该选项 ）
{{< highlight console >}}
$ lscpu
Architecture:        x86_64
CPU op-mode(s):      32-bit, 64-bit
Byte Order:          Little Endian
CPU(s):              4
On-line CPU(s) list: 0-3
Thread(s) per core:  2
Core(s) per socket:  2
Socket(s):           1
NUMA node(s):        1
Vendor ID:           GenuineIntel
CPU family:          6
Model:               37
Model name:          Intel(R) Core(TM) i5 CPU    650  @ 3.20GHz
Stepping:            5
CPU MHz:             1866.000
CPU max MHz:         3333.0000
CPU min MHz:         1199.0000
BogoMIPS:            6386.65
Virtualization:      VT-x
L1d cache:           32K
L1i cache:           32K
L2 cache:            256K
L3 cache:            4096K
NUMA node0 CPU(s):   0-3
Flags:               fpu vme de pse tsc msr pae mce cx8 apic
sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr
sse sse2 ss ht tm pbe syscall nx rdtscp lm constant_tsc
arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc
cpuid aperfmperf pni pclmulqdq dtes64 monitor ds_cpl
vmx smx est tm2 ssse3 cx16 xtpr pdcm pcid sse4_1 sse4_2
popcnt aes lahf_lm tpr_shadow vnmi flexpriority ept vpid
dtherm ida arat

{{< /highlight >}}
{{< highlight console >}}

$ zgrep CONFIG_KVM /proc/config.gz
CONFIG_KVM_GUEST=y
# CONFIG_KVM_DEBUG_FS is not set
CONFIG_KVM_MMIO=y
CONFIG_KVM_ASYNC_PF=y
CONFIG_KVM_VFIO=y
CONFIG_KVM_GENERIC_DIRTYLOG_READ_PROTECT=y
CONFIG_KVM_COMPAT=y
CONFIG_KVM=m
CONFIG_KVM_INTEL=m
CONFIG_KVM_AMD=m
CONFIG_KVM_MMU_AUDIT=y

$ lsmod |grep kvm
kvm_intel             192512  0
kvm                   520192  1 kvm_intel
irqbypass              16384  1 kvm

{{< /highlight >}}
{{< highlight console >}}

$ zgrep VIRTIO /proc/config.gz
CONFIG_BLK_MQ_VIRTIO=y
CONFIG_VIRTIO_VSOCKETS=m
CONFIG_VIRTIO_VSOCKETS_COMMON=m
CONFIG_NET_9P_VIRTIO=m
CONFIG_VIRTIO_BLK=m
# CONFIG_VIRTIO_BLK_SCSI is not set
CONFIG_SCSI_VIRTIO=m
CONFIG_VIRTIO_NET=m
CONFIG_CAIF_VIRTIO=m
CONFIG_VIRTIO_CONSOLE=m
CONFIG_HW_RANDOM_VIRTIO=m
CONFIG_DRM_VIRTIO_GPU=m
CONFIG_VIRTIO=m
CONFIG_VIRTIO_PCI=m
CONFIG_VIRTIO_PCI_LEGACY=y
CONFIG_VIRTIO_BALLOON=m
CONFIG_VIRTIO_INPUT=m
CONFIG_VIRTIO_MMIO=m
CONFIG_VIRTIO_MMIO_CMDLINE_DEVICES=y
CONFIG_CRYPTO_DEV_VIRTIO=m

$ lsmod |grep virtio
# === nothing yet ====
# see more https://wiki.archlinux.org/index.php/Kernel_modules

{{< /highlight >}}
{{< highlight console >}}

$ sudo pacman -S qemu libvirt dnsmasq ebtables

{{< /highlight >}}

## 鉴权和服务

所以作为虚拟机管理员，我放弃使用过于底层的 qemu 指令，转而作为一个“客户端”，首先连接守护服务进程 libvirt ，再进而管理。

有连接就有鉴权，libvirt 安装时提供了相应的 policy 文件 ```/usr/share/polkit-1/actions/org.libvirt.unix.policy```。只要属于 wheel 组，用户就可以做 VM 管理。

还记得在之前的一篇 ["Archlinux 配置"]({{< relref "blog/dual-os-3-setup-arch.md" >}}) 中专門建立了 kvm 用户组，通过创建```/etc/polkit-1/rules.d/50-libvirt.rules```的 polkit rule 文件为 kvm 的组成员做免密配置。

{{< highlight javascript >}}
// Allow users in kvm group to manage
//   the libvirt daemon without authentication
polkit.addRule(function(action, subject) {
    if (action.id == "org.libvirt.unix.manage" &&
        subject.isInGroup("kvm")) {
            return polkit.Result.YES;
    }
});

{{< /highlight >}}

配置完毕，启动服务即可:

{{< highlight console >}}

$ sudo systemctl start libvirtd.service
$ sudo systemctl start virtlogd.service
$ sudo systemctl enable libvirtd.service

{{< /highlight >}}

## Vagrant 安装配置

{{< highlight console >}}

$ sudo pacman -S vagrant
$ CONFIGURE_ARGS='with-ldflags=-L/opt/vagrant/embedded/lib with-libvirt-include=/usr/include/libvirt with-libvirt-lib=/usr/lib' \
  GEM_HOME=~/.vagrant.d/gems GEM_PATH=$GEM_HOME:/opt/vagrant/embedded/gems PATH=/opt/vagrant/embedded/bin:$PATH \
  vagrant plugin install vagrant-libvirt

$ echo "export VAGRANT_DEFAULT_PROVIDER=libvirt" >> ~/.bashrc

$ mkdir testbed
$ cd testbed
$ vagrant init centos/7
$ vagrant up
{{< /highlight >}}


篇幅有限，未完待续

参考文档

> - https://wiki.archlinux.org/index.php/QEMU
> -

封面图片来自 [Space Frolicking](https://dribbble.com/shots/2618501-Space-Frolicking) <a href="https://dribbble.com/TomasBrunsdon"><i class="fa fa-dribbble" aria-hidden="true"></i> Tomas Brunsdon</a>
