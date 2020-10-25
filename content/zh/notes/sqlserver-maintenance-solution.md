+++
title = "一个好用的SQL Server维护脚本"
author = "larrylvzg"
tags = ["sql server", "tsql"]
description = ""
draft = false
+++

分享一个非常好用的SQL Server数据库备份脚本。这个脚本的功能介绍如下：
1. 可以根据数据库的大小来将备份文件分割成多个小文件。
2. 备份期间进行必要的索引优化(根据索引碎片率来判断是否要重建或重组索引)和 Statistics 更新。
3. 利用从SQL Server 2017开始的新特性 [Online Resumable Index](https://www.mssqltips.com/sqlservertip/4987/sql-server-2017-resumable-online-index-rebuilds/)。
4. 根据自上一次完整备份或事务日志备份时间点后修改了的数据量，进行智能的差异、事务日志备份。
5. 支持Linux平台
6. 支持直接备份到Azure Blob Storage云端，可以分割多个小文件进行传输。
7. 支持Availability Groups.
8. 针对大容量数据库进行完整性检查。

脚本的官网地址：https://ola.hallengren.com/

[下载地址](https://ola.hallengren.com/scripts/MaintenanceSolution.sql)