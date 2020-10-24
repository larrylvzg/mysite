+++
title = "使用KMS激活你的Windows系统"
author = "larrylvzg"
tags = ["kms"]
description = ""
draft = false
+++

如果你安装的Windows系统是属于微软VOL许可证的，比如Windows 10企业版，那么你可以使用KMS激活方式。在正规合法的情况下使用KMS激活方式，要求你安装部署微软的KMS服务器程序（一般是安装在企业内网环境），并让所有需要激活的Windows系统指向这个内部的KMS服务器，从而实现批量的激活。

当然，随着黑客技术的不断进步，网上出现了很多非官方版本的KMS服务程序，经过验证测试，发现完全可以替换微软官方的KMS服务器。这些非官方的KMS服务器端程序，我在这里就不详细列出了，大家可以自行搜索。

因为KMS方式激活，不需要在被激活的系统上安装任何破解程序，（一切都是用系统自带的脚本slmgr.vbs来激活），所以这种方式更正式、更安全。

在这篇文章中，我将详细说明如何利用KMS服务器激活Windows系统，同时，也会列出一些网上免费的KMS服务器地址。

## KMS Client Setup 密钥

{{< expand "Windows 2019" >}}
Operating system edition | KMS Client Setup Key
------------------------|--------------------
Windows Server 2019 Datacenter |	WMDGN-G9PQG-XVVXX-R3X43-63DFG
Windows Server 2019 Standard |	N69G4-B89J2-4G8F4-WWYCC-J464C
Windows Server 2019 Essentials |	WVDHN-86M7X-466P6-VHXV7-YY726
{{< /expand >}}

{{< expand "Windows Server 2016" >}}
Operating system edition | KMS Client Setup Key
------------------------|--------------------
Windows Server 2016 Datacenter |	CB7KF-BWN84-R7R2Y-793K2-8XDDG
Windows Server 2016 Standard |	WC2BQ-8NRM3-FDDYY-2BFGV-KHKQY
Windows Server 2016 Essentials |	JCKRF-N37P4-C2D82-9YXRT-4M63B
{{< /expand >}}

{{< expand "Windows Server 2012 R2" >}}
 Operating system edition | KMS Client Setup Key
------------------------|--------------------
 Windows Server 2012 R2 Server Standard | D2N9P-3P6X9-2R39C-7RTCD-MDVJX
 Windows Server 2012 R2 Datacenter | W3GGN-FT8W3-Y4M27-J84CP-Q3VJ9
 Windows Server 2012 R2 Essentials | KNC87-3J2TX-XB4WP-VCPJV-M4FWM
{{< /expand >}}

{{< expand "Windows 10" >}}
Operating system edition | KMS Client Setup Key
------------------------|--------------------
Windows 10 Pro |	W269N-WFGWX-YVC9B-4J6C9-T83GX
Windows 10 Pro N |	MH37W-N47XK-V7XM9-C7227-GCQG9
Windows 10 Pro for Workstations |	NRG8B-VKK3Q-CXVCJ-9G2XF-6Q84J
Windows 10 Pro for Workstations N |	9FNHH-K3HBT-3W4TD-6383H-6XYWF
Windows 10 Pro Education |	6TP4R-GNPTD-KYYHQ-7B7DP-J447Y
Windows 10 Pro Education N |	YVWGF-BXNMC-HTQYQ-CPQ99-66QFC
Windows 10 Education |	NW6C2-QMPVW-D7KKK-3GKT6-VCFB2
Windows 10 Education N |	2WH4N-8QGBV-H22JP-CT43Q-MDWWJ
Windows 10 Enterprise |	NPPR9-FWDCX-D2C8J-H872K-2YT43
Windows 10 Enterprise N |	DPH2V-TTNVB-4X9Q3-TJR4H-KHJW4
Windows 10 Enterprise G |	YYVX9-NTFWV-6MDM3-9PT4T-4M68B
Windows 10 Enterprise G N |	44RPN-FTY23-9VTTB-MP9BX-T84FV
{{< /expand >}}

## 配置KMS客户端
KMS客户端指的需要被激活的Windows系统。首先，以管理员模式打开命令提示符：
### 安装密钥
命令：slmgr /ipk [yourlicensekey]
以window 10 Enterprise为例：
```
slmgr /ipk NPPR9-FWDCX-D2C8J-H872K-2YT43
```

### 指定KMS服务器地址
命令： slmgr /skms [activation server address]
```
slmgr /skms `kms8.msguides.com`
```

{{< expand "免费的KMS服务器地址" >}}
* `kms8.msguides.com`
{{< /expand >}}

### 激活
```
slmgr /ato
```
