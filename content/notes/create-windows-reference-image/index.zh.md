+++
title = "创建干净的windows 10母盘"
date= 2020-10-27T16:00:06+08:00
author = "larrylvzg"
tags = ["deployment"]
description = ""
draft = false
+++

制作一个干净的windows 10母盘，需要大概如下步骤：
1. 从官方的ISO镜像安装一个纯净的windows 10系统，建议在虚拟机里面进行。
2. 在第一步安装完成后，在OOBE界面时，按 ctrl+shift+f3 进入审核模式(Audit Mode),安装必要的APP，驱动程序和配置优化。
3. 使用Sysprep.exe通用化后，关机。
4. 启动到WinPE系统，使用DISM工具捕捉镜像。

