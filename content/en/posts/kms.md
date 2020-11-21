+++
title = "Using KMS to activate your Windows"
date= 2020-10-24T17:00:06+08:00
author = "larrylvzg"
tags = ["kms"]
description = "Using KMS to activate your Windows"
draft = false
+++

Are you looking for KMS server addresses everywhere to activate your Windows system? It is not ruled out that there are some free KMS servers on the Internet, but the long-term stable use is not guaranteed. In this article, I will briefly describe how to set up your own KMS server and Windows activation steps.

## Deployment of KMS Server
The good news is that there is now an open source and free KMS server program [vlmcsd](https://github.com/Wind4/vlmcsd), whose author claims that it can be used to replace Microsoft's KMS server. In addition, some enthusiastic netizens packaged it into a Docker container [image](https://hub.docker.com/r/mikolatero/vlmcsd/) to facilitate quick installation and deployment. Only need to execute a command, your KMS server can run successfully,

```bash
$ docker run -d -p 1688:1688 --restart=always --name vlmcsd mikolatero/vlmcsd
```

## The Steps to activate.
KMS client refers to the Windows system that needs to be activated. First, open a command prompt in administrator mode:
### Installation of KMS Client Setup Key
Command：slmgr /ipk kms_client_setup_key
The kms_client_setup_key of each operating system version is different, please refer to the official Microsoft article for details. [KMS Client Setup Key](https://docs.microsoft.com/en-us/windows-server/get-started/kmsclientkeys)
Take window 10 Enterprise as an example：
```
slmgr /ipk NPPR9-FWDCX-D2C8J-H872K-2YT43
```

### Specify the KMS server address
Command： slmgr /skms `KMS Server address`
```
slmgr /skms KMS_Server_address
```

> KMS_Server_address == Docker Container Address

{{< expand "The free KMS Server Address" >}}
* `kms8.msguides.com`
{{< /expand >}}

### The last step
```
slmgr /ato
```

## References
* https://github.com/Wind4/vlmcsd
* https://docs.microsoft.com/en-us/windows-server/get-started/kmsclientkeys
