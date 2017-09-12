---
title: SVN Cmd
date: 2017-09-12 21:01:13
tags:
---
因为工作中经常需要用到SVN,虽然我知道git很好，但我们要考虑我们项目的大众啊，策划啊，美术啊，运营啊等等等等，所以我们选择了SVN，但因为一些命令，常会忘记，一到要用的时候，方恨未做记录。这里就记录些常用的，和一些遇到问题后用的。

问题:
```
svn: E155017: Checksum mismatch while updating 'xxx': 
   expected:  xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx 
     actual:  xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

解决：
```
svn update --set-depth empty 
svn update --set-depth infinity
```


问题:
```
svn: Previous operation has not finished; run 'cleanup' if it was interrupted
```

解决:
[sqlite3.exe](https://www.sqlite.org/2017/sqlite-tools-win32-x86-3200100.zip)
```
先下载sqlite3.exe(链接在上方)
将sqlite3复制到报错svn项目的.svn目录的同级目录
sqlite3 .svn/wc.db "select * from work_queue" (查看队列未完成的任务)
sqlite3 .svn/wc.db "delete from work_queue" (清空队列)
svn cleanup
```

问题:
[语义化版本](http://semver.org/lang/zh-CN/)
```
打版本(这里涉及到语义化版本的问题,参考上方链接内容)
```

解决：
```
svn copy svn://xxxx/server/trunk svn://xxxx/server/branch/0.1.0_beta -m "辅助脚本签出新版本：0.1.0_beta"
```
