---
title: Centos Cmd
date: 2017-09-08 23:24:00
tags:
---
可能人老了吧，记忆衰退了，一些常用的命令也会忘记，而且本人书签很多，找起来不容易，这里就整理下一些常用的Centos Cmd，方面后面查看。这些都是经常用到的 Cmd。大部分Cmd可以直接用参数 --help 来查看相应的注释。

##### 查看外网地址：
```
$ curl icanhazip.com
```
or
```
$ curl ifconfig.me
```

##### 查看文件夹大小:
```
$ du -sh dirname
```

##### 查找文件:
```
$ locate -i /etc/sh  #查找/etc目录下的sh开头的文件
```
or
```
$ find / -name mod.erl #查找/目录下的名为mod.erl的文件
```

##### 在多文件中查找指定字符：
```
$ grep "被查找字符" dirname
```

##### 服务器TIME_WAIT查看:
```
$ netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'    
```

##### 服务器TIME_WAIT处理:

修改 /etc/sysctl.conf
```
#对于一个新建连接，内核要发送多少个 SYN 连接请求才决定放弃,不应该大于255，默认值是5，对应于180秒左右时间  
net.ipv4.tcp_syn_retries=2  
#net.ipv4.tcp_synack_retries=2  
#表示当keepalive起用的时候，TCP发送keepalive消息的频度。缺省是2小时，改为300秒  
net.ipv4.tcp_keepalive_time=1200  
net.ipv4.tcp_orphan_retries=3  
#表示如果套接字由本端要求关闭，这个参数决定了它保持在FIN-WAIT-2状态的时间  
net.ipv4.tcp_fin_timeout=30  
#表示SYN队列的长度，默认为1024，加大队列长度为8192，可以容纳更多等待连接的网络连接数。  
net.ipv4.tcp_max_syn_backlog = 4096  
  
#表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭  
net.ipv4.tcp_syncookies = 1  
#表示开启重用。允许将TIME-WAIT sockets重新用于新的TCP连接，默认为0，表示关闭  
net.ipv4.tcp_tw_reuse = 1  
#表示开启TCP连接中TIME-WAIT sockets的快速回收，默认为0，表示关闭  
net.ipv4.tcp_tw_recycle = 1  
#可用端口范围  
net.ipv4.ip_local_port_range = 10000    61000  
  
##减少超时前的探测次数  
net.ipv4.tcp_keepalive_probes=5  
##优化网络设备接收队列  
net.core.netdev_max_backlog=3000  
```
/sbin/sysctl -p 让参数生效

##### 查找僵尸进程
```
$ ps -A -ostat,ppid,pid,cmd | grep -e '^[Zz]'
```

##### 服务器禁止ping
一般服务器不需要禁止ping,因为禁止之后我们无法通过ping命令来检测服务器是否丢包等。
```
#禁止
$ echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all (禁止)
#开启
$ echo 0 > /proc/sys/net/ipv4/icmp_echo_ignore_all
#永久保存(/etc/rc.d/rc.local 里面内容开机后会自动执行)
$ vi /etc/rc.d/rc.local
$ echo 1 >/proc/sys/net/ipv4/icmp_echo_ignore_all
```

##### 查看内存

```
$ free -m #MB为单位
$ free -g #GB为单位
```

##### 查看内核版本
```
$ uname -a
```
or
```
$ cat /proc/version
```

##### 查看发行版信息
```
$ cat /etc/issue
```

##### 查看cpu相关信息
```
$ cat /proc/cpuinfo
```

##### 查看磁盘空间
```
$ df
$ df -h #我们更加可读的方式显示
```

##### 统计目录下文件行数
```
$ find . -name "*.erl" | xargs cat | wc -l
```
or
```
$ find . -name "*.erl" -exec wc -l {} \; | awk '{sum+=$1}END{print sum}'
```

##### 统计目录下文件行数，排除空行
```
$ find . -name "*.erl" | xargs cat | grep -v ^$ | wc -l
```

##### 统计目录下文件数
```
$ find . -name "*.erl" | wc -l
```

##### 打包文件，排除指定目录,文件
```
$ tar -zcvf dnv_android.tar.gz ./server/ebin --exclude=./Dir
```

