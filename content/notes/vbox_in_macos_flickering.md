+++
title = "在MacOS下的VirtualBox CentOS虚拟机图形界面闪屏问题"
date= 2020-11-17T16:00:06+08:00
author = "larrylvzg"
tags = ["linux"]
description = ""
draft = false
+++

闪屏问题的解决办法：

1. Open `/etc/gdm/custom.conf` in a text editor
2. Find `WaylandEnable` property, make sure that it is set to `false` and that the line is not commented out.
Or you can set the default display server by adding this line under 
```
[daemon]
DefaultSession=gnome=xorg.desktop
```

3. Save the changes and restart the computer.
Congratulation! You have fixed the flickering.

