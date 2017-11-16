+++
date = "2017-09-15T20:55:02+08:00"
title = "打造虚拟实验厂"
showonlyimage = false
image = "/img/blog/mgnt-virenv-7-vagrant/400.png"
topImage = "/img/blog/mgnt-virenv-7-vagrant/400.gif"
draft = false
weight = 68
+++

QEMU/KVM、Libvirt、Vagrant 的安装配置
<!--more-->

## 虚拟层

### KVM

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

### Virtualbox

vagrant 的默认虚拟层使用的是 virtualbox，这里也记录一下相关的安装配置。

{{< highlight console >}}

$ sudo pacman -S virtualbox
# choose vitualbox-host-modules-arch for linux kernel
# reboot so virtualbox kernel modules auto loaded, see file
# /usr/lib/modules-load.d/virtualbox-host-modules-arch.conf
# lsmod to list already loaded
# modinfo to get any module details even not loaded

$ sudo pacman -S net-tools virtualbox-guest-iso
# net-tools contains cli ifconfig and route
# iso contains additional guest tools

$ sudo pacman -S virtualbox-guest-utils
# manually load the modules in
# /usr/lib/modules-load.d/virtualbox-guest-modules-arch.conf
# modprobe -a vboxguest vboxsf vboxvideo
# improve video, input, sharing etc, launching
$ VBoxClient --clipboard --draganddrop \
        --seamless --display --checkhostversion

$ sudo usermod -a -G vboxsf user
{{< /highlight >}}

## 鉴权和服务

虚拟机管理的最佳实践是放弃过于底层的 qemu 指令，而将自己作为一个“客户端”，首先连接守护服务进程 libvirt ，再进而使用 virsh 和相应 API 完成管理维护。

有连接就有鉴权，libvirt 软件包提供了 policy 文件 ```/usr/share/polkit-1/actions/org.libvirt.unix.policy```。而在 arch 中只要是属于 wheel 组的用户就可以做 VM 管理。

在之前的一篇 ["Archlinux 配置"]({{< relref "blog/dual-os-3-setup-arch.md" >}}) 中专門建立了 kvm 用户组，这里则通过创建```/etc/polkit-1/rules.d/50-libvirt.rules```的 polkit rule 文件为 kvm 的组成员做免密配置。

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

如果你执行```getent group kvm```发现 kvm 的 gid 不是固定值 78 而是一个较大值，比如动态分配从 999 开始，详见 [FS#54943 - [systemd] [qemu] [libvirt] Two or more conflicting lines for kvm configured, ignoring.](https://bugs.archlinux.org/task/54943) 你需要编辑 ```/etc/libvirt/qemu.conf``` 将 group 的值从 78 调整为 kvm，这样你就不会在以后面启动虚拟机时遇到```Could not access KVM kernel module: Permission denied```错误。

配置完毕，启动服务，并测试一下 virsh 是否工作正常:

{{< highlight console >}}

$ sudo systemctl start libvirtd.service
$ sudo systemctl start virtlogd.service
$ sudo systemctl enable libvirtd.service
$ virsh -c qemu:///system

# add env var to avoid typing “-c uri” each time
$ echo "export LIBVIRT_DEFAULT_URI=qemu:///system" >> ~/.bashrc
{{< /highlight >}}

建议配置到这里重启一次系统，以便缺省的 Storage pools 名为 default 指向目录 ```/var/lib/libvirt/images/``` 已真正被创建并生效。不然后续调用 vagrant up 时会遇找不到 pool 的错误。

{{< highlight console >}}
$ virsh pool-list
$ virsh pool-info default
$ virsh pool-dumpxml default

{{< /highlight >}}

<br />

## Vagrant 安装配置

首先安装 vagrant 主程序。

{{< highlight console >}}
$ sudo pacman -S vagrant rsync
$ pacman -Q |grep vagrant
vagrant 2.0.0-1
vagrant-substrate 726.d082972-1
{{< /highlight >}}

{{< figure src="/img/blog/mgnt-virenv-7-vagrant/env.png" title="output of screenfetch" >}}

截至 2017 年 9 月，[arch wiki](https://wiki.archlinux.org/index.php/Vagrant#vagrant-libvirt) 中仍保留了插件 vagrant-libvirt 安装所需的特定 workaround。而我安装时觉得 vagrant 已非 wiki 中提示的版本，尝试直接运行插件安装指令，结果在后面 vagrant 启动 VM 时各种失败，都是各种找不到 API 的错误，如```undefined method `persistent?' for #<Libvirt::StoragePool:xxx```。解决办法是阅读 [gist](https://gist.github.com/j883376/d90933620c7ed14daa4e0963e005377f) 并按相应步骤卸载，反复重装插件才最终搞定。

> 另外我后来发现 AUR 有这个插件的[软件包](https://aur.archlinux.org/packages/vagrant-libvirt/)，有点纳闷到底是该自行安装，还是通过 AUR 安装。

启动 VM 时，基本流程是先联网下载 image ，然后根据 Vagrantfile 中的配置启动 VM 。

{{< highlight console >}}

$ vagrant plugin list
vagrant-libvirt (0.0.40)
vagrant-proxyconf (1.5.2)

$ export VAGRANT_DEFAULT_PROVIDER=libvirt

$ mkdir testbed
$ cd testbed
$ vagrant init centos/7
Bringing machine 'default' up with 'libvirt' provider...
=> default: Box 'centos/7' could not be found. Attempting to find and install...
   default: Box Provider: libvirt
   default: Box Version: >= 0
=> default: Loading metadata for box 'centos/7'
   default: URL: https://vagrantcloud.com/centos/7
=> default: Adding box 'centos/7' (v1708.01) for provider: libvirt
   default: Downloading: https://vagrantcloud.com/centos/boxes/7/versions/1708.01/providers/libvirt.box
=> default: Successfully added box 'centos/7' (v1708.01) for 'libvirt'!
=> default: Uploading base box image as volume into libvirt storage...
=> default: Creating image (snapshot of base box volume).
=> default: Creating domain with the following settings...
=> default:  -- Name:              testbed_default
=> default:  -- Domain type:       kvm
=> default:  -- Cpus:              1
=> default:  -- Feature:           acpi
=> default:  -- Feature:           apic
=> default:  -- Feature:           pae
=> default:  -- Memory:            512M
=> default:  -- Management MAC:    
=> default:  -- Loader:            
=> default:  -- Base box:          centos/7
=> default:  -- Storage pool:      default
=> default:  -- Image:             /var/lib/libvirt/images/testbed_default.img (41G)
=> default:  -- Volume Cache:      default
=> default:  -- Kernel:            
=> default:  -- Initrd:            
=> default:  -- Graphics Type:     vnc
=> default:  -- Graphics Port:     5900
=> default:  -- Graphics IP:       0.0.0.0
=> default:  -- Graphics Password: Not defined
=> default:  -- Video Type:        cirrus
=> default:  -- Video VRAM:        9216
=> default:  -- Sound Type:
=> default:  -- Keymap:            en-us
=> default:  -- TPM Path:          
=> default:  -- INPUT:             type=mouse, bus=ps2
=> default: Creating shared folders metadata...
=> default: Starting domain.
=> default: Waiting for domain to get an IP address...
=> default: Waiting for SSH to become available...
   default:
   default: Vagrant insecure key detected. Vagrant will automatically replace
   default: this with a newly generated keypair for better security.
   default:
   default: Inserting generated public key within guest...
   default: Removing insecure key from the guest if it's present...
   default: Key inserted! Disconnecting and reconnecting using new SSH key...
=> default: Configuring and enabling network interfaces...
   default: SSH address: 192.168.121.51:22
   default: SSH username: vagrant
   default: SSH auth method: private key
=> default: Rsyncing folder: ~/testbed/ => /vagrant

{{< /highlight >}}

我在这里遇到了一个很诡异的坑，启动 VM 时进程挂死在``` Waiting for domain to get an IP address...``` 处。之前 dnsmasq 已经正确安装，并不需要任何手动配置——因为 vagrant 会创建一个名为 vagrant-libvirt 的专用网络。在 vagrant up 运行时可以执行下面的命令验证:

{{< highlight console >}}
$ virsh net-list --all
Name                 State      Autostart     Persistent
----------------------------------------------------------
default              inactive   no            yes
vagrant-libvirt      active     no            yes

$ virsh net-info vagrant-libvirt
Name:           vagrant-libvirt
UUID:           bc5e692b-9850-4117-b140-30e0f2935685
Active:         yes
Persistent:     yes
Autostart:      no
Bridge:         virbr1

$ virsh net-dumpxml vagrant-libvirt
<network connections='2' ipv6='yes'>
  <name>vagrant-libvirt</name>
  <uuid>bc5e692b-9850-4117-b140-30e0f2935685</uuid>
  <forward mode='nat'>
    <nat>
      <port start='1024' end='65535'/>
    </nat>
  </forward>
  <bridge name='virbr1' stp='on' delay='0'/>
  <mac address='52:54:00:3d:22:16'/>
  <ip address='192.168.121.1' netmask='255.255.255.0'>
    <dhcp>
      <range start='192.168.121.1' end='192.168.121.254'/>
    </dhcp>
  </ip>
</network>

$ ip a
# ==== virbr1 should be available in output ===
{{< /highlight >}}

> 关于网络，我之前使用的是 NetworkManager ，但对于容器、虚拟机来说 [systemd-networkd](https://wiki.archlinux.org/index.php/Systemd-networkd) 也许是更好的选择，这里先挖个坑，今后找机会再填。

vagrant up 可以加 debug 参数查看详情。通过报错的关键字在 github vagrant-libvirt issues 中搜索，相关问题上报还挺多的，

- [Public networking is waiting endlessly for IP information #97](https://github.com/vagrant-libvirt/vagrant-libvirt/issues/97)
- [How to get more detail & solutions to resolve it about "libvirt: Waiting for domain to get an IP address..." ? #560](https://github.com/vagrant-libvirt/vagrant-libvirt/issues/560)
- [Waiting for IP fails silently after a long time #673](https://github.com/vagrant-libvirt/vagrant-libvirt/issues/673)
- [New box created using create_box.sh - gets stuck at Waiting for domain to get an IP address... #784](https://github.com/vagrant-libvirt/vagrant-libvirt/issues/784)

查问题的方法也很多，如 #560 中建议使用 wireshark 尝试网络抓包。而 #784 中建议的通过 vnc 连接到 VM 对我这个问题起到了关键性作用。

启动 vinagre 连接 VM 可以看到其内核载入启动部分。结果我发现这个 VM 居然报的错误是 boot from disk failure。然后就不用再和 DHCP 较劲了。转而一查 ```/var/lib/libvirt/images/```，大概是下载 image 时的网络问题，下载的介质并不完整。于是重新删除后下载果然就好了。

{{< highlight console >}}
$ vagrant destroy testbed
$ vagrant box list
$ vagrant box remote centos/7
$ virsh vol-list default
$ virsh vol-delete some.img
{{< /highlight >}}

vagrant ssh enjoy

{{< highlight console >}}
vagrant status
vagrant up
vagrant ssh
vagrant halt

# to save a snapshot with name "clean"
vagrant snapshot save clean
# to restore the snapshot
vagrant snapshot restore clean --no-provision

vagrant destroy
vagrant provision
vagrant reload
{{< /highlight >}}

参考文档

> - https://wiki.archlinux.org/index.php/QEMU
> - https://wiki.archlinux.org/index.php/Libvirt
> - https://wiki.archlinux.org/index.php/KVM
> - https://wiki.archlinux.org/index.php/Vagrant
> - https://wiki.archlinux.org/index.php/VirtualBox

封面图片来自 [Space Frolicking](https://dribbble.com/shots/2618501-Space-Frolicking) <a href="https://dribbble.com/TomasBrunsdon"><i class="fa fa-dribbble" aria-hidden="true"></i> Tomas Brunsdon</a>
