---
title: Openstack Too many open files 错误与解决办法
tags: [openstack]
categories: Openstack
description: Linux系统可以使用的文件描述符不够导致
date: 2019/6/20 15:20:00
---

Openstack WebUI页面无法打开，页面报500错误，查看httpd->error_log日志报如下错误:

```
[Tue Apr 02 14:01:05.658276 2019] [:error] [pid 9245]   File "/usr/lib/python2.7/site-packages/requests/sessions.py", line 518, in request
[Tue Apr 02 14:01:05.658280 2019] [:error] [pid 9245]   File "/usr/lib/python2.7/site-packages/requests/sessions.py", line 639, in send
[Tue Apr 02 14:01:05.658284 2019] [:error] [pid 9245]   File "/usr/lib/python2.7/site-packages/requests/adapters.py", line 438, in send
[Tue Apr 02 14:01:05.658287 2019] [:error] [pid 9245]   File "/usr/lib/python2.7/site-packages/requests/packages/urllib3/connectionpool.py", line 588, in urlopen
[Tue Apr 02 14:01:05.658291 2019] [:error] [pid 9245]   File "/usr/lib/python2.7/site-packages/requests/packages/urllib3/connectionpool.py", line 241, in _get_conn
[Tue Apr 02 14:01:05.658296 2019] [:error] [pid 9245]   File "/usr/lib/python2.7/site-packages/requests/packages/urllib3/util/connection.py", line 27, in is_connection_dropped
[Tue Apr 02 14:01:05.658300 2019] [:error] [pid 9245]   File "/usr/lib/python2.7/site-packages/requests/packages/urllib3/util/wait.py", line 33, in wait_for_read
[Tue Apr 02 14:01:05.658304 2019] [:error] [pid 9245]   File "/usr/lib/python2.7/site-packages/requests/packages/urllib3/util/wait.py", line 22, in _wait_for_io_events
[Tue Apr 02 14:01:05.658308 2019] [:error] [pid 9245]   File "/usr/lib/python2.7/site-packages/requests/packages/urllib3/util/selectors.py", line 581, in DefaultSelector
[Tue Apr 02 14:01:05.658312 2019] [:error] [pid 9245]   File "/usr/lib/python2.7/site-packages/requests/packages/urllib3/util/selectors.py", line 394, in __init__
[Tue Apr 02 14:01:05.658316 2019] [:error] [pid 9245] IOError: [Errno 24] Too many open files
```

解决方式：
修改操作系统打开的文件数；登录到Controller节点，执行:


``` bash
[root@controller ~]# ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 60587
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 60587
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

系统默认设置为1024。

使用命令查看当前打开文件数:

``` bash
[root@controller ~]# lsof | wc -l
174911
```

修改vim /etc/security/limits.conf，在文件最后加入如下信息：

```
*               soft    nofile            1024000
*               hard    nofile            1024000
```

*表示所有用户，
修改后重启服务器，配置生效。
