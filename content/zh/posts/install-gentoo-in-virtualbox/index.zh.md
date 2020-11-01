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
/swap |        | 1G |
/     | btrfs  | 剩余所有空间 |

在liveCD环境下，列出机器上安装的所有块设备，然后执行 parted 分区程序：
```bash
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
```bash
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
```bash
(parted) mklabel gpt
```

接着，按照我们的规划来分区：
```bash
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
```bash
(parted) name 1 boot
(parted) set 1 boot on # UEFI启动模式下，parted自动将分区标记为ESP                          
(parted) name 2 swap                                                      
(parted) name 3 root                                                      
(parted) quit       
```

再次列出所有磁盘：
```bash
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
```bash
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
```
# mount /dev/sda3 /mnt/gentoo
# mkdir /mnt/gentoo/boot
# mount /dev/sda1 /mnt/gentoo/boot
```

`根`分区被mount到了liveCD环境的 `/mnt/gentoo`目录， `boot`分区被mount到了liveCD环境的`/mnt/gentoo/boot`目录。

## 安装Stage 3 存档文件和Chroot
将我们在准备工作中下载的Stage 3 tarball文件复制到`/mnt/gentoo`目录下，然后解开。
```bash
cd /mnt/gentoo
tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
```

成功解开后，清理Stage3 tarball文件。
rm stage3-*.tar.xz

### 配置编译器选项
打开配置文件：
```bash
# nano -w /mnt/gentoo/etc/portage/make.conf
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
```
# mirrorselect -i -o >> /mnt/gentoo/etc/portage/make.conf
```

#### Gentoo ebuild 仓库
创建仓库目录并复制配置文件：
```
# mkdir --parents /mnt/gentoo/etc/portage/repos.conf
# cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
```

#### 复制DNS配置
可以确保进入新环境后，网络还是正常的。
```
# cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
```

#### 挂载必要的文件系统
```
# mount --types proc /proc /mnt/gentoo/proc
# mount --rbind /sys /mnt/gentoo/sys
# mount --make-rslave /mnt/gentoo/sys
# mount --rbind /dev /mnt/gentoo/dev
# mount --make-rslave /mnt/gentoo/dev
```

#### 进入新的root环境
```
# chroot /mnt/gentoo /bin/bash
# source /etc/profile
# export PS1="(chroot) ${PS1}"
(chroot) livecd / # 
```

#### 挂载boot分区
```
# mount /dev/sda1 /boot
mount: /boot: /dev/sda1 already mounted on /boot.
```

### 配置Portage
#### 安装Gentoo ebuild库
```bash
(chroot) livecd / # emerge-webrsync 
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

(chroot) livecd / # 
(chroot) livecd / # 
```

#### 选择Profile
查看Profile列表：
```bash
(chroot) livecd / # eselect profile list
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
  ```bash
  (chroot) livecd / # eselect profile set 14
  ```

#### 更新@world
  执行命令：
  ```bash
  emerge --ask --verbose --update --deep --newuse @world
  ```

#### (可选)ACCEPT_LICENSE
修改`/etc/portage/make.conf`文件，`*`代表接受所有类型的license.
```
ACCEPT_LICENSE="*"
```

### 时区
查看系统可用的时区：
```bash
ls /usr/share/zoneinfo
```

设置时区为"Asia/Shanghai":
```bash
echo "Asia/Shanghai" > /etc/timezone
```

下一步重新配置`sys-libs/timezone-data`, 它将根据`/etc/timezone`的内容去更新`/etc/localtime`文件。
```bash
emerge --config sys-libs/timezone-data
```

### 语言
支持的语言必须定义在`/etc/locale.gen`文件中：
```bash
nano -w /etc/locale.gen
```

文件内容：
```
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
```

然后执行`locale-gen`:
```bash
locale-gen 
 * Generating 3 locales (this might take a while) with 2 jobs
 *  (1/3) Generating en_US.UTF-8 ...                                                                                                            [ ok ]
 *  (2/3) Generating zh_CN.UTF-8 ...                                                                                                            [ ok ]
 *  (3/3) Generating C.UTF-8 ...                                                                                                                [ ok ]
 * Generation complete
 * Adding locales to archive ...       
```

系统语言设置，查看语言列表：
```bash
eselect locale list
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
```bash
env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
```






