---
title: WebIssues
date: 2017-09-16 17:32:53
tags:
---
工作需要搭建一个项目管理软件，我选择了WebIssues, 这里记录下搭建的过程，比较简单。

系统环境：Centos6.5
WebIssues版本：WebIssues1.1.5

服务端：

 - 下载服务端：[webissues-server-1.1.5.tar.bz2](http://downloads.sourceforge.net/webissues/webissues-server-1.1.5.tar.bz2)
 - 创建数据库(支持MySQL,PostgreSQL,SQL Server)，我用的是MySQL,仅需要在MySQL里面创建一个库就可以啦。
 - 配置webissues-server的nginx(官方主要支持IIS,Apache)，nginx config 里面的root目录指向webissues-server-1.1.5.tar.bz2解压后的目录。
 - 访问配置好的网站，根据上面的提示安装就可以(中间会有个data目录下面的sites没有权限访问，需要分配777)。

客户端：

 - 下载客户端：[webissues-1.1.5-win_x64.exe](http://downloads.sourceforge.net/webissues/webissues-1.1.5-win_x64.exe)
 - 直接用客户端就可以啦。

官方文档：

 - [wiki](http://wiki.mimec.org/wiki/Main_Page)
 - [Manual](http://doc.mimec.org/webissues/server/)
