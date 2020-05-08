---
title: NFS 详解
tags: [linux]
categories: Linux
description: nfs常见问题解决办法
date: 2020/5/8 15:00:00
---

### nfs客户端卡住问题
碰到nfs客户端卡住的情况，umount -f /mnt提示device is busy，并且尝试访问挂载目录、df -h等操作都会使终端卡住，ctrl+c也不能强行退出。

造成这种现象的原因是nfs服务器/网络挂了，nfs客户端默认采用hard-mount选项，而不是soft-mount。他们的区别是
soft-mount: 当客户端加载NFS不成功时，重试retrans设定的次数.如果retrans次都不成功，则放弃此操作，返回错误信息 "Connect time out"
hard-mount: 当客户端加载NFS不成功时,一直重试，直到NFS服务器有响应。hard-mount 是系统的缺省值。在选定hard-mount 时，最好同时选 intr , 允许中断系统的调用请求，避免引起系统的挂起。当NFS服务器不能响应NFS客户端的 hard-mount请求时， NFS客户端会显示
"NFS server hostname not responding, still trying"


### nfs参数介绍
下面列出mount关于nfs相关的参数
（1）-a：把/etc/fstab中列出的路径全部挂载。
（2）-t：需要mount的类型，如nfs等。
（3）-r：将mount的路径定为read only。
（4）-v mount：过程的每一个操作都有message传回到屏幕上。
（5）rsize=n：在NFS服务器读取文件时NFS使用的字节数，默认值是4096个字节。
（6）wsize=n：向NFS服务器写文件时NFS使用的字节数，默认值是4096个字节。
（7）timeo=n：从超时后到第1次重新传送占用的1/7秒的数目，默认值是7/7秒。
（8）retry=n：在放弃后台mount操作之前可以尝试的次数，默认值是7 000次。
（9）soft：使用软挂载的方式挂载系统，若Client的请求得不到回应，则重新请求并传回错误信息。
（10）hard：使用硬挂载的方式挂载系统，该值是默认值，重复请求直到NFS服务器回应。
（11）intr：允许NFS中断文件操作和向调用它的程序返回值，默认不允许文件操作被中断。
（12）fg：一直在提示符下执行重复挂载。
（13）bg：如果第1次挂载文件系统失败，继续在后台尝试执行挂载，默认值是失败后不在后台处理。
（14）tcp：对文件系统的挂载使用TCP，而不是默认的UDP。

如#mount -t nfs -o soft 192.168.1.2:/home/nfs /mnt

### nfs传输尺寸
至于传输尺寸的选择，可以进行实际测试：
time dd if=/dev/zero of=/mnt/nfs.dat bs=16k count=16384
即向nfs服务器上的nfs.dat文件里写入16384个16KB的块（也有经验说文件大小可以设定为nfs服务器内存的2倍）。
得到输出如：
输出了 16384+0 个块
user    0m0.200s
输出了 66535+0 个块
user    0m0.420s
192.168.1.4:/mnt  /home/nfs  nfs   rsize=8192,wsize=8192,timeo=10,intr
重新挂载nfs服务器，调整读写块大小后重复上述过程，可以找到最佳传输尺寸。

### NFS服务器的故障排除
故障排除思路:
NFS出现了故障，可以从以下几个方面着手检查。
（1）NFS客户机和服务器的负荷是否太高，服务器和客户端之间的网络是否正常。
（2）/etc/exports文件的正确性。
（3）必要时重新启动NFS或portmap服务。
运行下列命令重新启动portmap和NFS：
service portmap restart
service nfs start
（4）检查客户端中的mount命令或/etc/fstab的语法是否正确。
（5）查看内核是否支持NFS和RPC服务。
普通的内核应有的选项为CONFIG_NFS_FS=m、CONFIG_NFS_V3=y、CONFIG_ NFSD=m、CONFIG_NFSD_V3=y和CONFIG_SUNRPC=m。
我们可以使用常见的网络连接和测试工具ping及tracerroute来测试网络连接及速度是否正常，网络连接正常是NFS作用的基础。rpcinfo命令用于显示系统的RPC信息
，一般使用-p参数列出某台主机的RPC服务。用rpcinfo-p命令检查服务器时，应该能看到portmapper、status、mountd nfs和nlockmgr。用该命令检查客户端时，应
该至少能看到portmapper服务。

### 使用nfsstat命令查看NFS服务器状态
nfsstat命令显示关于NFS和到内核的远程过程调用（RPC）接口的统计信息，也可以使用该命令重新初始化该信息。如果未给定标志，默认是nfsstat -csnr命令。使用该命令显示每条信息，但不能重新初始化任何信息。

nfsstat命令的主要参数如下。
（1）-b：显示NFS V4服务器的其他统计信息。
（2）c：只显示客户机端的NFS和RPC信息，允许用户仅查看客户机数据的报告。nfsstat命令提供关于被客户机发送和拒绝的RPC和NFS调用数目的信息。
要只显示客户机NFS或者RPC信息，将该参数与-n或者-r参数结合。
（3）-d：显示与NFS V4授权相关的信息。
（4）-g：显示RPCSEC_GSS信息。
（5）-m：显示每个NFS文件系统的统计信息，该文件系统和服务器名称、地址、安装标志、当前读和写大小，以及重新传输计数
（6）-n：为客户机和服务器显示NFS信息。要只显示NFS客户机或服务器信息，将该参数与-c和-s参数结合。
（7）-r：显示RPC信息。
（8）-s：显示服务器信息。
（9）-t：显示与NFS标识映射子系统的转换请求相关的统计信息，要只显示NFS客户机或服务器信息，将-c和-s选项结合。
（10）-4：当与-c、-n、-s或-z参数组合使用时，将包含NFS V4客户机或服务器的信息，以及现有的NFS V2和V3数据。
（11）-z：重新初始化统计信息。该参数仅供root用户使用，并且在显示上面的标志后可以和那些标志中的任何一个组合到统计信息的零特殊集合。

要显示关于客户机发送和拒绝的RPC和NFS调用数目的信息，输入：
nfsstat -c
要显示和打印与客户机NFS调用相关的信息，输入如下命令：
nfsstat -cn
要显示和打印客户机和服务器的与RPC调用相关的信息，输入如下命令：
nfsstat -r
要显示关于服务器接收和拒绝的RPC和NFS调用数目的信息，输入如下命令：
nfsstat –s
