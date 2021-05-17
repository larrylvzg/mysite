---
title: "Install Xrdp in Centos7"
date: 2020-11-18T11:16:03+08:00
draft: false
---

Xrdp 是一款非常好用的开源远程桌面服务端。它可以让IT管理员使用windows下面的`远程桌面`客户端远程登陆linux服务器的图形界面。
这篇文章将简单介绍如何在CentOS 7 下安装Xrdp服务端。

## 前置条件
首先需要在CentOS下面安装Gnome GUI

其次再安装 EPEL 库，
```bash
rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
```

## 开始安装 Xrdp
使用 yum 命令安装 xrdp 包
```bash
yum -y install xrdp tigervnc-server
```

待 xrdp 安装完成后，让它每次开机自动启动
```bash
systemctl enable xrdp
systemctl start xrdp
```

xrdp 启动后，应该监听在本机的 3389 端口下，你可以执行如下命令来确认：
```bash
netstat -antup | grep xrdp
```

## 开启防火墙的端口
配置防火墙以允许 3389 对外开放，
```bash
firewall-cmd --permanent --add-port=3389/tcp
firewall-cmd --reload
```

## 配置SELinux
如果不想禁用SElinux，那么就：
```bash
chcon --type=bin_t /usr/sbin/xrdp
chcon --type=bin_t /usr/sbin/xrdp-sesman
```

最后测试从客户端远程桌面到xrdp是否成功。
{{< pagebundle/img "xrdp_remote.png" "远程桌面连接到Xrdp" >}}