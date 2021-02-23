---
title: libguestfs 简介
tags: [libguestfs]
categories: tool
description: KVM libguestfs 工具介绍
date: 2021/2/23 14:00:00
---


### 简介

使用 libguestfs 实现宿主机与虚拟机的文件传输。

ibguestfs 是一组 Linux 下的 C 语言的 API ，用来访问虚拟机的磁盘映像文件。

其项目主页是http://libguestfs.org/ ，该工具包内包含的工具有virt-cat、virt-df、virt-ls、virt-copy-in、virt-copy-out、virt- edit、guestfs、guestmount、virt-list-filesystems、virt-list-partitions等工具，具体 用法也可以参看官网。该工具可以在不启动KVM guest主机的情况下，直接查看guest主机内的文内容，也可以直接向img镜像中写入文件和复制文件到外面的物理机，当然其也可以像mount一 样，支持挂载操作。

### 安装

计算节点安装libguestfs
```php
[root@compute ~]# yum -y install libguestfs-tools
# 对于windows虚拟机的支持
[root@compute ~]# yum -y install libguestfs-winsupport
```

### 命令列表

#### libguestfs命令

```php

root@compute144:~ # virt-
virt-alignment-scan      virt-customize           virt-host-validate       virt-pki-validate        virt-tar-out
virt-builder             virt-df                  virt-index-validate      virt-rescue              virt-what
virt-builder-repository  virt-diff                virt-inspector           virt-resize              virt-win-reg
virt-cat                 virt-edit                virt-install             virt-sparsify            virt-xml
virt-clone               virt-filesystems         virt-log                 virt-sysprep             virt-xml-validate
virt-copy-in             virt-format              virt-ls                  virt-tail
virt-copy-out            virt-get-kernel          virt-make-fs             virt-tar-in
```

#### virt-ls： 可以列出虚拟机中目录下的文件或目录
```php
root@compute144:~ # virt-ls -d instance-00001445 /home/
agent_linux
ftpuser
kedong
```

#### virt-cat： 可以查看虚拟机中文件的内容
```php
root@compute144:~ # virt-cat -d instance-00001445 /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
```

#### virt-df： 将在虚拟机中执行df命令的结果输出
```php
root@compute144:~ # virt-df -d instance-00001445
Filesystem                           1K-blocks       Used  Available  Use%
instance-00001445:/dev/sdb                 490        490          0  100%
instance-00001445:/dev/sda1            1038336     100948     937388   10%
instance-00001445:/dev/centos/root     2238464    1897392     341072   85%
```

#### virt-copy-in：将文件复制到虚拟机里面
```php
virt-list-partitions - List partitions in a virtual machine or disk image

SYNOPSIS
 virt-list-partitions [--options] domname

 virt-list-partitions [--options] disk.img [disk.img ...]
OBSOLETE
This tool is obsolete. Use virt-filesystems(1) as a more flexible replacement.

DESCRIPTION
virt-list-partitions is a command line tool to list the partitions that are contained in a virtual machine or disk image. It is mainly useful as a first step to using virt-resize(1).

virt-list-partitions is just a simple wrapper around libguestfs(3) functionality. For more complex cases you should look at the guestfish(1) tool.

OPTIONS
--help
Display brief help.

--version
Display version number and exit.

-c URI
--connect URI
If using libvirt, connect to the given URI. If omitted, then we connect to the default libvirt hypervisor.

If you specify guest block devices directly, then libvirt is not used at all.

--format raw
Specify the format of disk images given on the command line. If this is omitted then the format is autodetected from the content of the disk image.

If disk images are requested from libvirt, then this program asks libvirt for this information. In this case, the value of the format parameter is ignored.

If working with untrusted raw-format guest disk images, you should ensure the format is always specified.

-h
--human-readable
Show sizes in human-readable form (eg. "1G").

-l
--long
With this option, virt-list-partitions displays the type and size of each partition too (where "type" means ext3, pv etc.)

-t
--total
Display the total size of each block device (as a separate row or rows).

SEE ALSO
guestfs(3), guestfish(1), virt-filesystems(1), virt-list-filesystems(1), virt-resize(1), Sys::Guestfs(3), Sys::Virt(3), http://libguestfs.org/.

AUTHOR
Richard W.M. Jones http://people.redhat.com/~rjones/

COPYRIGHT
Copyright (C) 2009-2020 Red Hat Inc.

LICENSE
This program is free software; you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation; either version 2 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program; if not, write to the Free Software Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.

BUGS
To get a list of bugs against libguestfs, use this link: https://bugzilla.redhat.com/buglist.cgi?component=libguestfs&product=Virtualization+Tools

To report a new bug against libguestfs, use this link: https://bugzilla.redhat.com/enter_bug.cgi?component=libguestfs&product=Virtualization+Tools

When reporting a bug, please supply:

The version of libguestfs.

Where you got libguestfs (eg. which Linux distro, compiled from source, etc)

Describe the bug accurately and give a way to reproduce it.

Run libguestfs-test-tool(1) and paste the complete, unedited output into the bug report.
```

#### virt-copy-out： 可以把虚拟机里的文件复制出来到本地主机
```php
virt-copy-out - Copy files and directories out of a virtual machine disk image.

SYNOPSIS
 virt-copy-out -a disk.img /file|dir [/file|dir ...] localdir

 virt-copy-out -d domain /file|dir [/file|dir ...] localdir
DESCRIPTION
virt-copy-out copies files and directories out of a virtual machine disk image or named libvirt domain.

You can give one or more filenames and directories on the command line. Directories are copied out recursively.

EXAMPLES
Download the home directories from a virtual machine:

 mkdir homes
 virt-copy-out -d MyGuest /home homes
JUST A SHELL SCRIPT WRAPPER AROUND GUESTFISH
This command is just a simple shell script wrapper around the guestfish(1) copy-out command. For anything more complex than a trivial copy, you are probably better off using guestfish directly.

OPTIONS
Since the shell script just passes options straight to guestfish, read guestfish(1) to see the full list of options.

SEE ALSO
guestfish(1), virt-cat(1), virt-copy-in(1), virt-edit(1), virt-tar-in(1), virt-tar-out(1), http://libguestfs.org/.

AUTHORS
Richard W.M. Jones (rjones at redhat dot com)

COPYRIGHT
Copyright (C) 2011-2012 Red Hat Inc.

LICENSE
This program is free software; you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation; either version 2 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program; if not, write to the Free Software Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.

BUGS
To get a list of bugs against libguestfs, use this link: https://bugzilla.redhat.com/buglist.cgi?component=libguestfs&product=Virtualization+Tools

To report a new bug against libguestfs, use this link: https://bugzilla.redhat.com/enter_bug.cgi?component=libguestfs&product=Virtualization+Tools

When reporting a bug, please supply:

The version of libguestfs.

Where you got libguestfs (eg. which Linux distro, compiled from source, etc)

Describe the bug accurately and give a way to reproduce it.

Run libguestfs-test-tool(1) and paste the complete, unedited output into the bug report.
```

#### 系统组件适配
```mysql

系统组件适配 virt-copy-in virt-copy-out 备注
---------- ------------ ------------- ----------------------------------------
Win7            √             √         目前查到只支持传输到根目录 / 下 ，也就是C盘
---------- ------------ ------------- ----------------------------------------
Centos7         √             √
---------- ------------ ------------- ----------------------------------------
Ubuntu16        √             √
---------- ------------ ------------- ----------------------------------------
凝思8            √             √
---------- ------------ ------------- ----------------------------------------
pfSense         ×             ×         暂时不支持FreeBSD系统
---------- ------------ ------------- ----------------------------------------
Kali            √             √
---------- ------------ ------------- ----------------------------------------

```
