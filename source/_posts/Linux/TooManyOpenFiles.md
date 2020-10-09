---
title: Too many open files的四种解决办法
tags: [linux]
categories: Linux
description: Too many open files的四种解决办法
date: 2020/6/1 09:20:00
---

【摘要】Too many open files有四种可能:
- 一 单个进程打开文件句柄数过多
- 二 操作系统打开的文件句柄数过多
- 三 systemd对该进程进行了限制
- 四 inotify达到上限

#### 单个进程打开文件句柄数过多
ulimit中的nofile表示单进程可以打开的最大文件句柄数，可以通过ulimit -a查看，
子进程默认继承父进程的限制
（注意，是继承，不是共享，子进程和父进程打开的文件句柄数是单独算的）。

网上还有一种解读是nofile表示单用户可以打开的文件句柄数，因为他们在limit.conf中看到类似于"openstack soft nofile 65536"，便认为是openstack用户最多可以打开的文件句柄数。该解读是错误的，"openstack soft nofile 65536"表示的含义是当你执行"su - openstack"切换到openstack用户后，你创建的所有进程最大可以打开的文件句柄数是65536。
要查看一个进程可以打开的文件句柄数，可以通过"cat /proc/<pid>/limits"查看。

要修改ulimit中的nofile，可以通过修改/etc/security/limits.conf文件，
在其中加入类似"openstack soft nofile 65536"的语句来进行修改。
修改完成后，可以通过"su - openstack"切换用户，或者重新登录，来使该配置生效。

要动态修改一个进程的限制，可以使用prlimit命令，具体用法为："prlimit --pid ${pid} --nofile=102400:102400"。

#### 操作系统打开的文件句柄数过多
整个操作系统可以打开的文件句柄数是有限的，受内核参数"fs.file-max"影响。

可以通过执行"echo 100000000 > /proc/sys/fs/file-max"命令来动态修改该值，
也可以通过修改"/etc/sysctl.conf"文件来永久修改该值。

#### systemd对该进程进行了限制
该场景仅针对被systemd管理的进程（也就是可以通过systemctl来控制的进程）生效，
可以通过修改该进程的service文件（通常在/etc/systemd/system/目录下），
在"[Service]"下面添加"LimitNOFILE=20480000"来实现，
修改完成之后需要执行"systemctl daemon-reload"来使该配置生效。

#### inotify达到上限
inotify是linux提供的一种监控机制，可以监控文件系统的变化。
该机制受到2个内核参数的影响："fs.inotify.max_user_instances"和 "fs.inotify.max_user_watches"，
其中"fs.inotify.max_user_instances"表示每个用户最多可以创建的inotify instances数量上限，
"fs.inotify.max_user_watches"表示么个用户同时可以添加的watch数目，
当出现too many open files问题而上面三种方法都无法解决时，可以尝试通过修改这2个内核参数来生效。
修改方法是修改"/etc/sysctl.conf"文件，并执行"sysctl -p"。
