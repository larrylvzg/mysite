+++
title = "使用KMS激活你的Windows系统"
author = "larrylvzg"
tags = ["kms"]
description = ""
draft = false
+++

你是否为了激活你的Windows系统而到处寻找KMS服务器地址呢？不排除网上有一些免费用的KMS服务器，但能否长期稳定的使用就不敢保证了。在这篇文章中，我将简单描述如何搭建属于自己的KMS服务器和Windows的激活步骤。

## KMS服务器搭建

好消息是，现在有开源免费的KMS服务器程序[vlmcsd](https://github.com/Wind4/vlmcsd)，它的作者声称可以用来替代微软的KMS服务器。另外，有热心网友将它打包成Docker容器[镜像](https://hub.docker.com/r/mikolatero/vlmcsd/)，以方便大家快速的安装部署。只需要执行一条命令，你的KMS服务器即可成功运行，
```bash
$ docker run -d -p 1688:1688 --restart=always --name vlmcsd mikolatero/vlmcsd
```

## KMS客户端激活步骤
KMS客户端指的需要被激活的Windows系统。首先，以管理员模式打开命令提示符：
### 安装密钥
命令：slmgr /ipk kms_client_setup_key
每个操作系统版本的kms_client_setup_key都不一样，具体可以参考微软官方文章[KMS Client Setup Key](https://docs.microsoft.com/en-us/windows-server/get-started/kmsclientkeys)
以window 10 Enterprise为例：
```
slmgr /ipk NPPR9-FWDCX-D2C8J-H872K-2YT43
```

### 指定KMS服务器地址
命令： slmgr /skms KMS服务器地址
```
slmgr /skms KMS服务器地址
```

> KMS服务器地址==上述Docker容器的地址

{{< expand "免费的KMS服务器地址" >}}
* `kms8.msguides.com`
{{< /expand >}}

### 激活
```
slmgr /ato
```

## 参考
* https://github.com/Wind4/vlmcsd
* https://docs.microsoft.com/en-us/windows-server/get-started/kmsclientkeys
