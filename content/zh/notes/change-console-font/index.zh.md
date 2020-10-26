+++
title = "由研究如何更改Powershell控制台字体引发的故事"
date= 2020-10-26T16:00:06+08:00
author = "larrylvzg"
tags = ["powershell"]
description = ""
draft = false
+++

## 引言
这几天闲的没事干，打算重新学习一下已经丢了很久的Powershell脚本编程。突然发现一个只有"深度强迫症患者"才能关注到的问题---Powershell控制台的字体太他么难看了。
{{< pagebundle/img "default_powershell_console.png" "简体中文版系统默认的powershell控制台" >}}
不行，一定要先把控制台的字体改的好看一些才能继续我的Powershell旅程。目标至少是将字体从"新宋体"改成"consolas", 说干就干！接下来，就有了下面的故事。

## 遇到的一系列坑
### 修改控制台属性的字体设置
首先想到的是，能否直接在控制台程序的属性里面更改字体呢？ 结果打开控制台属性对话框后，发现只有字体栏中能改到的就那么几个中文字体。
{{< pagebundle/img "console_font_dialog.png" "powershell控制台属性对话框" >}}
通过研究发现，简体中文版的windows系统的控制台代码页（codePage）是936，也就是简体中文GBK。如果在控制台中输入如下命令可以修改当前控制台代码页为65001，也就是UTF-8。
```
chcp 65001
```
执行后，发现在控制台属性对话框里面，竟然出现了我最喜欢的consolas字体。
{{< pagebundle/img "utf8_console_font_dialog.png" "修改了代码页为UTF8后的字体对话框" >}}
本来以为一切都大功告成了，可是关掉当前控制台后，再次打开发现一切又回到了从前。这个更改操作只对当前窗口有效。

{{< alert theme="info" >}}
**补充说明：** 为了每次打开powershell控制台都可以自动执行chcp 65001命令，我们可以采用`Powershell Profile`的方式。`Powershell Profile`就是每次打开控制台时系统自动执行的powershell脚本文件。以下代码是自动在`所有用户所有宿主`的Profile文件里面附加上chcp 65001：
```powershell
Add-Content -Path $Profile.AllusersAllhosts -Value "chcp 65001"
```
{{< /alert >}}

将控制台的代码页改为65001有一些风险，参考链接：
* https://stackoverflow.com/questions/388490/how-to-use-unicode-characters-in-windows-command-line

所以，这种修改控制台字体的方式不算完美，不过没关系，我们继续研究！

### Beta 版：使用Unicode UTF-8提供全球语言支持
我又想到了去系统的“区域和语言设置”中一探究竟。在“开始”菜单---“运行”里面输入intl.cpl, 在打开的窗口中选择“管理”选项卡，在“非unicode程序的语言”下面，点击“更改系统区域设置”按钮。
{{< pagebundle/img "change_locale.png" "更改系统区域设置" >}}
在上图中，有没有发现一个新的东西？对的，就是红框标注的那个“Beta 版：使用Unicode UTF-8提供全球语言支持”，默认是没有选中的。如果选中后，系统会提示立即重启。待重启完成后，发现powershell控制台的代码页已经是65001了。

以为一切就此结束，但是不放心啊，因为设置里面有Beta字样，那就代表着还在测试阶段。于是，我在网上搜索了一下这个设置的”副作用“，别说还真被我猜到了，这个设置被形容为"dangerous"---危险啊。
参考链接：
* https://techcommunity.microsoft.com/t5/windows-10/non-unicode-utf-8-program-in-windows-10/m-p/612813


## 替代方案---Windows Terminal登场
既然问题没法完美解决，为了稳妥起见，就干脆寻找替代方案了。

幸运的是，还真有完美的替代方案，那就是微软新出的“Windows Terminal”，它专门用来替代旧的命令提示符和Powershell控制台的。但是要想使用这个软件，需要Windows 10 1903 (build 18362)及以后的版本。安装方式很简单，建议直接通过Microsoft Store来安装，便于后期版本升级。

下图是它的界面，有木有发现确实好看多了，没错，这就是我想要的效果。终于我可以继续研究Powershell了。
{{< pagebundle/img "windows_terminal.png" "windows terminal" >}}

Github地址：https://github.com/microsoft/terminal

