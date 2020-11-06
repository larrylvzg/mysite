+++
title = "在VirtualBox虚拟机中安装Gentoo Linux系统"
date= 2020-11-01T17:00:06+08:00
author = "larrylvzg"
tags = ["gentoo"]
description = ""
draft = false
+++

## 前言
曾经使用过Redhat系（ CentOS, Fedora ）、Debian系（ Ubuntu, Kali ）的Linux发行版。不可否认，这些系统非常成熟，而且有强大的商业公司在其背后支持。但我个人比较喜欢的 Linux 系统是偏向于深度可定制的，比如直接从源代码编译安装，这样我就可以深入了解系统内部是如何构建的。另外最重要的是，这非常酷，有木有？

因为暂时没有物理机可供安装，所以先考虑在 VirtualBox 虚拟机里面安装 Gentoo Linux，顺便记录一下安装过程。

## 准备工作
1. 虚拟机基本配置（仅供参考）
CPU: 2 core
Mem: 4096M
System: Enable EFI
Storage: 100 GB VDI
Network: 1 NAT, 1 HostOnly vboxnet0

2. 下载Gentoo Minimal Installation CD (amd64)
[下载地址](https://bouncer.gentoo.org/fetch/root/all/releases/amd64/autobuilds/20201028T214503Z/install-amd64-minimal-20201028T214503Z.iso)

3. 下载 Stage 3 - systemd 存档包
[下载地址](https://bouncer.gentoo.org/fetch/root/all/releases/amd64/autobuilds/20201028T214503Z/stage3-amd64-systemd-20201028T214503Z.tar.xz)

## 磁盘分区
之前创建的 .vdi 磁盘还是一个空白磁盘，就和刚买来的新硬盘没什么两样。我们的目标就是将Gentoo安装到这个磁盘上，使它能自行启动。大概的步骤：
1. 使磁盘（.vdi）可启动
2. 分区并格式化文件系统

我们正式开始吧！首先，设置好虚拟机从 Minimal Installation CD 启动进入gentoo live环境。
{{< pagebundle/img "images/start-gentoo-livecd.png" "启动 Gentoo liveCD" >}}

为顺应未来的趋势，我将机器主板固件设置为UEFI, 磁盘分区表类型为GPT, 分区规划如下：
分区 | 文件系统 | 容量 | 备注
---- | -------- | ----- | ----
/boot | fat32  | 500M | EFI系统分区，也就是ESP
swap |         | 1G   |
/     | btrfs  | 剩余所有空间 |

在liveCD环境下，列出机器上安装的所有块设备，然后执行 parted 分区程序：
```console
livecd ~ # lsblk 
NAME  MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0   7:0    0  388M  1 loop /mnt/livecd
sda     8:0    0  100G  0 disk 
sr0    11:0    1  425M  0 rom  /mnt/cdrom
livecd ~ # parted -a opt /dev/sda
GNU Parted 3.2
Using /dev/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted)  
```

这样，我们就进入了 parted 分区程序，尝试输入 `print` 列出所有分区：
```console
(parted) print                                                            
Error: /dev/sda: unrecognised disk label
Model: ATA VBOX HARDDISK (scsi)                                           
Disk /dev/sda: 107GB
Sector size (logical/physical): 512B/512B
Partition Table: unknown
Disk Flags: 
(parted)
```

出现了预期的错误，因为这个磁盘是完全空白的，我们下一步是给磁盘添加 `label`:
```console
(parted) mklabel gpt
```

接着，按照我们的规划来分区：
```console
(parted) unit MB 
# 单位设置为 MB                                                         
(parted) mkpart primary 1 500                                             
# /boot分区 500MB                                                     
(parted) mkpart primary 501 1524    
# Swap分区 1024MB
(parted) mkpart primary 1525 -1
# 根分区， -1 代表剩余的所有可用空间
(parted) print                                                            
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sda: 107374MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags: 

Number  Start   End       Size      File system  Name     Flags
 1      1.05MB  500MB     499MB                  primary
 2      501MB   1524MB    1022MB                 primary
 3      1525MB  107373MB  105849MB               primary

(parted)
```

你可以从上看到每个分区的编号和名称Primary。但是还没有格式化和分区的命名，接下来就做这些：
```console
(parted) name 1 boot
(parted) set 1 boot on # UEFI启动模式下，parted自动将分区标记为ESP                          
(parted) name 2 swap                                                      
(parted) name 3 root                                                      
(parted) quit       
```

再次列出所有磁盘：
```console
livecd ~ # lsblk                                                          
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0    7:0    0  388M  1 loop /mnt/livecd
sda      8:0    0  100G  0 disk 
|-sda1   8:1    0  476M  0 part 
|-sda2   8:2    0  975M  0 part 
`-sda3   8:3    0 98.6G  0 part 
sr0     11:0    1  425M  0 rom  /mnt/cdrom
```

依次格式化：
```console
livecd ~ # mkfs.vfat /dev/sda1
mkfs.fat 4.1 (2017-01-24)
livecd ~ # mkswap /dev/sda2
Setting up swapspace version 1, size = 975 MiB (1022357504 bytes)
no label, UUID=909d2848-4101-45d5-a557-6b6ef1e07b83
livecd ~ # swapon /dev/sda2
livecd ~ # mkfs.btrfs /dev/sda3
btrfs-progs v5.4.1 
See http://btrfs.wiki.kernel.org for more information.

Label:              (null)
UUID:               be7dac62-b390-4c7d-a784-cc6f9494a79c
Node size:          16384
Sector size:        4096
Filesystem size:    98.58GiB
Block group profiles:
  Data:             single            8.00MiB
  Metadata:         DUP               1.00GiB
  System:           DUP               8.00MiB
SSD detected:       no
Incompat features:  extref, skinny-metadata
Checksum:           crc32c
Number of devices:  1
Devices:
   ID        SIZE  PATH
    1    98.58GiB  /dev/sda3
```

现在，我们将这些分区 mount 到live环境，
```console
mount /dev/sda3 /mnt/gentoo
mkdir /mnt/gentoo/boot
mount /dev/sda1 /mnt/gentoo/boot
```

`根`分区被mount到了liveCD环境的 `/mnt/gentoo`目录， `boot`分区被mount到了liveCD环境的`/mnt/gentoo/boot`目录。

## 安装Stage 3 存档文件和Chroot
将我们在准备工作中下载的Stage 3 tarball文件复制到`/mnt/gentoo`目录下，然后解开。
```console
cd /mnt/gentoo
tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
```

成功解开后，清理Stage3 tarball文件。
rm stage3-*.tar.xz

### 配置编译器选项
打开配置文件：
```console
nano -w /mnt/gentoo/etc/portage/make.conf
```

基本配置如下：
```
# Compiler flags to set for all languages
COMMON_FLAGS="-march=native -O2 -pipe"
# Use the same settings for both variables
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"

# 根据CPU的核心数来定
MAKEOPTS="-j2"
```

### Chroot
#### 配置镜像源地址
可以使用工具来选择地理位置最近的源，
```console
mirrorselect -i -o >> /mnt/gentoo/etc/portage/make.conf
```

#### Gentoo ebuild 仓库
创建仓库目录并复制配置文件：
```console
mkdir --parents /mnt/gentoo/etc/portage/repos.conf
cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
```

#### 复制DNS配置
可以确保进入新环境后，网络还是正常的。
```console
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
```

#### 挂载必要的文件系统
```console
mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
mount --make-rslave /mnt/gentoo/dev
```

#### 进入新的root环境
```console
chroot /mnt/gentoo /bin/bash
source /etc/profile
export PS1="(chroot) ${PS1}"
```

#### 挂载boot分区
```console
mount /dev/sda1 /boot
```

### 配置Portage
#### 安装Gentoo ebuild库
``` console
emerge-webrsync 
!!! Section 'gentoo' in repos.conf has location attribute set to nonexistent directory: '/var/db/repos/gentoo'
!!! Invalid Repository Location (not a dir): '/var/db/repos/gentoo'
Fetching most recent snapshot ...
Trying to retrieve 20201031 snapshot from http://gentoo.aditsu.net:8000 ...
Fetching file gentoo-20201031.tar.xz.md5sum ...
Fetching file gentoo-20201031.tar.xz.gpgsig ...
Fetching file gentoo-20201031.tar.xz ...

Checking digest ...
Getting snapshot timestamp ...
Syncing local tree ...

Number of files: 148,970 (reg: 122,534, dir: 26,436)
Number of created files: 148,969 (reg: 122,534, dir: 26,435)
Number of deleted files: 0
Number of regular files transferred: 122,534
Total file size: 208.69M bytes
Total transferred file size: 208.69M bytes
Literal data: 208.69M bytes
Matched data: 0 bytes
File list size: 3.32M
File list generation time: 0.001 seconds
File list transfer time: 0.000 seconds
Total bytes sent: 107.17M
Total bytes received: 2.44M

sent 107.17M bytes  received 2.44M bytes  6.64M bytes/sec
total size is 208.69M  speedup is 1.90
Cleaning up ...

 * IMPORTANT: 7 news items need reading for repository 'gentoo'.
 * Use eselect news read to view new items.

```

#### 选择Profile
查看Profile列表：
```console
eselect profile list
Available profile symlink targets:
  [1]   default/linux/amd64/17.1 (stable)
  [2]   default/linux/amd64/17.1/selinux (stable)
  [3]   default/linux/amd64/17.1/hardened (stable)
  [4]   default/linux/amd64/17.1/hardened/selinux (stable)
  [5]   default/linux/amd64/17.1/desktop (stable)
  [6]   default/linux/amd64/17.1/desktop/gnome (stable)
  [7]   default/linux/amd64/17.1/desktop/gnome/systemd (stable)
  [8]   default/linux/amd64/17.1/desktop/plasma (stable)
  [9]   default/linux/amd64/17.1/desktop/plasma/systemd (stable)
  [10]  default/linux/amd64/17.1/developer (stable)
  [11]  default/linux/amd64/17.1/no-multilib (stable)
  [12]  default/linux/amd64/17.1/no-multilib/hardened (stable)
  [13]  default/linux/amd64/17.1/no-multilib/hardened/selinux (stable)
  [14]  default/linux/amd64/17.1/systemd (stable) *
  [15]  default/linux/amd64/17.0 (stable)
  [16]  default/linux/amd64/17.0/selinux (stable)
  [17]  default/linux/amd64/17.0/hardened (stable)
  ```

这里我选择[14]  default/linux/amd64/17.1/systemd (stable):
```console
eselect profile set 14
```

#### 更新@world
执行命令：
```console
emerge --ask --verbose --update --deep --newuse @world
```

#### (可选)ACCEPT_LICENSE
修改`/etc/portage/make.conf`文件，`*`代表接受所有类型的license.
```
ACCEPT_LICENSE="*"
```

### 时区
查看系统可用的时区：
```console
ls /usr/share/zoneinfo
```

设置时区为"Asia/Shanghai":
```console
echo "Asia/Shanghai" > /etc/timezone
```

下一步重新配置`sys-libs/timezone-data`, 它将根据`/etc/timezone`的内容去更新`/etc/localtime`文件。
```console
emerge --config sys-libs/timezone-data
```

### 语言
支持的语言必须定义在`/etc/locale.gen`文件中：
```console
nano -w /etc/locale.gen
```

文件内容：
```
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
```

然后执行`locale-gen`:
```console
# locale-gen 
 * Generating 3 locales (this might take a while) with 2 jobs
 *  (1/3) Generating en_US.UTF-8 ...                                                                                                            [ ok ]
 *  (2/3) Generating zh_CN.UTF-8 ...                                                                                                            [ ok ]
 *  (3/3) Generating C.UTF-8 ...                                                                                                                [ ok ]
 * Generation complete
 * Adding locales to archive ...       
```

系统语言设置，查看语言列表：
```console
# eselect locale list
/usr/bin/locale: Cannot set LC_CTYPE to default locale: No such file or directory
Available targets for the LANG variable:
  [1]   C
  [2]   C.utf8
  [3]   POSIX
  [4]   en_US.utf8
  [5]   zh_CN.utf8
  [6]   C.UTF8 *
  [ ]   (free form)

eselect locale set 4
```

现在重新加载环境：
```console
env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
```

## 编译内核
针对基于amd64-系统的Gentoo，建议使用包 `sys-kernel/gentoo-sources`.
```console
emerge --ask sys-kernel/gentoo-sources
```

这将在/usr/src/中安装Linux内核源码，并有一个符号连接叫作linux将指向安装的内核源码：
```console
(chroot) livecd / # ls -l /usr/src/linux
lrwxrwxrwx 1 root root 19 Nov  2 09:23 /usr/src/linux -> linux-5.4.72-gentoo
```

安装`genkernel`:
```console
emerge genkernel
```

接下来，编辑/etc/fstab文件来使包含有第二个值为/boot/的那条的第一个值指向到正确的设备。
```console
nano -w /etc/fstab
```

```
/dev/sda1   /boot   vfat    defaults  0 0
```

{{< notice info >}}
在Gentoo将来的安装中，还要再配置一次/etc/fstab。现在只需要正确设置/boot来让genkernel应用程序读到相应的配置。
{{< /notice >}}

开始编译：
```console
genkernel all
```

{{< notice info >}}
使用genkernel。它将自动配置并编译内核。

genkernel配置内核的工作原理几乎和安装CD配置的内核完全一致。也就是说当使用genkernel建立内核，系统通常将在引导时检测全部硬件，就像安装CD所做的。因为genkernel不需要任何手动内核配置，它对于那些不能轻松的编译他们自动内核的用户来说是一个理想的解决方案。

运行`genkernel all`来编译内核源码。值得注意的是，使用genkernel编译一个内核将支持几乎全部的硬件，这将使编译过程需要一阵子来完成！
{{< /notice >}}

## 配置系统
### 创建/etc/fstab文件
/etc/fstab文件使用一种特殊语法格式。每行都包含六个字段。这些字段之间由空白键（空格键，tab键，或者两者混合使用）分隔。每个字段都有自己的含意：
1. 第一个字段显示要挂载的特殊 block 设备或远程文件系统。 有几种设备标识符可用于特殊块设备节点，包括设备文件路径，文件系统标签，UUID，分区标签以及UUID。
2. 第二个字段是分区挂载点，也就是分区应该挂载到的地方
3. 第三个字段给出分区所用的文件系统
4. 第四个字段给出的是挂载分区时mount命令所用的挂载选项。由于每个文件系统都有自己的挂载选项，我们建议你阅读mount手册（man mount）以获得所有挂载选项的列表。多个挂载选项之间是用逗号分隔的。
5. 第五个字段是给dump使用的，用以决定这个分区是否需要dump。一般情况下，你可以把该字段设为0（零）。
6. 第六个字段是给fsck使用的，用以决定系统非正常关机之后文件系统的检查顺序。根文件系统应该为1，而其它的应该为2（如果不需要文件系统自检的话可以设为0）。

```console
nano -w /etc/fstab
```

```
/dev/sda1   /boot   vfat    defaults                  0 0
/dev/sda2   none         swap    sw                   0 0
/dev/sda3   /            btrfs    noatime             0 1
/dev/cdrom  /mnt/cdrom   auto    noauto,user          0 0
```

### 主机名、域名信息
这两个配置后面可以重新修改的，先随便设置一下即可。
```console
nano -w /etc/conf.d/hostname
# 设置主机名变量，选择主机名
hostname="tux"

nano -w /etc/conf.d/net
# 设定dns_domain的变量值为你的域名
dns_domain_lo="homenetwork"
```

### 安装DHCP客户端
```console
emerge dhcpcd
```

### Grub 2
```console
echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf
emerge --ask sys-boot/grub:2
grub-install --target=x86_64-efi --efi-directory=/boot
grub-mkconfig -o /boot/grub/grub.cfg
```

### 重启
现在可以退出chroot环境，卸载相关的文件系统：
```
exit
umount -l /mnt/gentoo/dev{/shm,/pts,}
umount -l /mnt/gentoo{/boot,/proc,}
shutdown now
```

再次启动虚拟机后，就会直接从虚拟硬盘启动系统了。到此，gentoo linux基本已经安装到虚拟机了。