+++
title = "Linux系统启动流程"
date= 2020-11-05T16:00:06+08:00
author = "larrylvzg"
tags = ["linux"]
description = ""
draft = true
+++

基本上很少有人能够详细的理清楚Linux系统的启动流程。这篇笔记将归纳总结，从我们按下开机按钮后到Linux系统的登陆界面出现之前，发生的一系列故事。

1. 按下开机按钮后，电脑系统开始执行主板固件里面的BIOS（ 不管是Legacy BIOS 或 UEFI，我们这里统称为BIOS ）程序。
2. BIOS程序里面会根据设置的启动顺序列表，依次尝试启动。这些可启动的设备就包括了常用的U盘或移动硬盘、光驱、网卡、本地磁盘等等。
3. BIOS找到可启动的设备后，会将控制权移交给设备上的Boot Loader, 目前常用的Boot Loader就是Grub2, 比较旧的有LILO。
4. Boot Loader会告诉操作系统内核，让其先加载一个initramfs，这个文件是一个cpio的归档文件并用gzip压缩过。操作系统内核会创建一个tmpfs的文件系统，并将initramfs解压缩到里面，然后执行tmpfs根目录下的init脚本。
5. Initramfs里面的init脚本会挂载硬盘上的 `/` 文件系统。如果文件系统是加密的或LVM，那么在挂载前init需要先加载一些必要的驱动模块，否则就识别不了。当然，这些工作都是由这个init自动完成的。
6. 切换root到真实的 `/` 分区。最后，通过执行`/sbin/init`来进行后续的**服务**的启动。
