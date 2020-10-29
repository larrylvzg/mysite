+++
title = "创建自定义的Windows PE镜像"
date= 2020-10-28T16:00:06+08:00
author = "larrylvzg"
tags = ["deployment"]
description = ""
draft = false
+++

## 准备工作
1. 下载和安装Windows ADK
2. 下载和安装Windows PE

## 创建
1. 启动“部署和镜像工具环境”命令提示符。
2. 创建一个WinPE文件夹
```
copype amd64 C:\WinPE_amd64
```

## 定制化
1. 挂载PE镜像
```
dism /Mount-Image /ImageFile:C:\WinPE_amd64\media\sources\boot.wim /index:1 /MountDir:C:\WinPE_amd64\mount
```

2. 添加驱动程序
```
dism /Image:C:\WinPE_amd64\mount /Add-Driver /Driver:c:\drivers /Recurse
```

3. 添加APP
你可以添加一些日常维护用的软件，比如硬盘分区、Windows用户账号密码重置等等。
```
md C:\WinPE_amd64\mount\windows\<MyApp>
xcopy C:\<MyApp> C:\WinPE_amd64\mount\windows\<MyApp>
```

4. 添加文件或文件夹
直接将文件或文件夹复制到 C:\WinPE_amd64\mount\，这个地址是WinPE的根目录。

5. 更改背景图
```
takeown /F C:\WinPE_amd64\mount\Windows\System32\winpe.jpg

icacls C:\WinPE_amd64\mount\Windows\System32\winpe.jpg /grant Administrators:F

copy new-background.jpg C:\WinPE_amd64\mount\Windows\System32\winpe.jpg
```

6. 卸载并提交
```
dism /Unmount-Image /MountDir:C:\WinPE_amd64\mount /Commit
```

7. 创建ISO镜像或可启动U盘
* ISO
```
MakeWinPEMedia /ISO C:\WinPE_amd64 C:\WinPE_amd64\WinPE_amd64.iso
```

* 可启动盘（这里假定F为U盘）
```
MakeWinPEMedia /UFD C:\WinPE_amd64 F:
```

