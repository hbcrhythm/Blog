---
title: Centos Cmd
date: 2017-09-08 23:24:00
tags:
---
可能人老了吧，记忆衰退了，一些常用的命令也会忘记，而且本人书签很多，找起来不容易，这里就整理下一些常用的Centos Cmd，方面后面查看。

查看外网地址：
```
$ curl icanhazip.com
```
or
```
$ curl ifconfig.me
```

查看文件夹大小:
```
$ du -sh dirname
```

查找文件:
```
$ locate -i /etc/sh  #查找/etc目录下的sh开头的文件
```
or
```
$ find / -name mod.erl #查找/目录下的名为mod.erl的文件
```