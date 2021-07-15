---
title: QEMU / KVM 快照介绍
tags: [kvm]
categories: KVM
description: KVM 介绍：使用 libvirt 做 QEMU/KVM 快照
date: 2021/01/18 15:30:00
---

1.QEMU/KVM快照

1.1 概念
QEMU/KVM 快照的定义：快照就是将虚机在某一个时间点上的磁盘、内存和设备状态保存一下，以备将来之用。它包括以下几类：

磁盘快照：磁盘的内容（可能是虚机的全部磁盘或者部分磁盘）在某个时间点上被保存，然后可以被恢复。
磁盘数据的保存状态：
在一个运行着的系统上，一个磁盘快照很可能只是崩溃一致的（crash-consistent） 而不是完整一致（clean）的，也是说它所保存的磁盘状态可能相当于机器突然掉电时硬盘数据的状态，机器重启后需要通过 fsck 或者别的工具来恢复到完整一致的状态（类似于 Windows 机器在断电后会执行文件检查）。(注：命令 qemu-img check -f qcow2 --output=qcow2 -r all filename-img.qcow2 可以对 qcow2 和 vid 格式的镜像做一致性检查。)
对一个非运行中的虚机来说，如果上次虚机关闭的时候磁盘是完整一致的，那么其被快照的磁盘快照也将是完整一致的。
磁盘快照有两种：
• 内部快照 - 使用单个的 qcow2 的文件来保存快照和快照之后的改动。这种快照是 libvirt 的默认行为，现在的支持很完善（创建、回滚和删除），但是只能针对 qcow2 格式的磁盘镜像文件，而且其过程较慢等。
• 外部快照 - 快照是一个只读文件，快照之后的修改是另一个 qcow2 文件中。外置快照可以针对各种格式的磁盘镜像文件。外置快照的结果是形成一个 qcow2 文件链：original <- snap1 <- snap2 <- snap3。
内存状态（或者虚机状态）：只是保持内存和虚机使用的其它资源的状态。如果虚机状态快照在做和恢复之间磁盘没有被修改，那么虚机将保持一个持续的状态；如果被修改了，那么很可能导致数据corruption。
系统还原点（system checkpoint）：虚机的所有磁盘的快照和内存状态快照的集合，可用于恢复完整的系统状态（类似于系统休眠）。
关于 崩溃一致（crash-consistent）的附加说明：

应该尽量避免在虚机I/O繁忙的时候做快照。这种时候做快照不是可取的办法。
vmware 的做法是装一个 tools，它是个 PV driver，可以在做快照的时候挂起系统
似乎 KVM 也有类似的实现 QEMU Guest Agent，但是还不是很成熟，可参考 http://wiki.libvirt.org/page/Qemu_guest_agent
快照还可以分为 live snapshot（热快照）和 Clod snapshot：
Live snapshot：系统运行状态下做的快照
Cold snapshot：系统停止状态下的快照
libvit 做 snapshot 的各个 API：

![](/images/snapshot1.png)

分别来看看这些 API 是如何工作的：
1. virDomainSnapshotCreateXML (virDomainPtr domain, const char * xmlDesc, unsigned int flags)
作用：根据 xmlDesc 指定的 snapshot xml 和 flags 来创建虚机的快照。

![](/images/snapshot2.png)

其内部实现根据虚机的运行状态有两种情形：
• 对运行着的虚机，API 使用 QEMU Monitor 去做快照，磁盘镜像文件必须是 qcow2 格式，虚机的 CPU 被停止，快照结束后会重新启动。
• 对停止着的虚机，API 调用 qemu-img 方法来操作所有磁盘镜像文件。
这里有其实现代码，可见其基本的实现步骤：
```php
static virDomainSnapshotPtr qemuDomainSnapshotCreateXML
{
    ....
    call qemuDomainSnapshotCreateDiskActive
    {
        call qemuProcessStopCPUs # 停止 vCPUs
        for each disk call qemuDomainSnapshotCreateSingleDiskActive
        {
            call qemuMonitorDiskSnapshot # 调用 QEMU Monitor 去为每个磁盘做snapshot
        }
        call qemuProcessStartCPUs # 启动 vCPUs
     }
     ....
}
```

2. virDomainSave 相关的几个 API
这几个API 功能都比较类似：

![](/images/snapshot3.png)

1.2 使用 virsh 实验
**virsh snapshot-create/snapshort-create-as**
先看看它的用法：
```php
virsh # help snapshot-create-as
  NAME
    snapshot-create-as - Create a snapshot from a set of args

  SYNOPSIS
    snapshot-create-as <domain> [<name>] [<description>] [--print-xml] [--no-metadata] [--halt] [--disk-only] [--reuse-external] [--quiesce] [--atomic] [--live] [--memspec <string>] [[--diskspec] <string>]...

  DESCRIPTION
    Create a snapshot (disk and RAM) from arguments

  OPTIONS
    [--domain] <string>  domain name, id or uuid
    [--name] <string>  name of snapshot
    [--description] <string>  description of snapshot
    --print-xml      print XML document rather than create
    --no-metadata    take snapshot but create no metadata  #创建的快照不带任何元数据
    --halt           halt domain after snapshot is created #快照创建后虚机会关闭
    --disk-only      capture disk state but not vm state   #只对磁盘做快照，忽略其它参数
    --reuse-external  reuse any existing external files
    --quiesce        quiesce guest's file systems #libvirt 会通过 QEMU GA 尝试去freeze和unfreeze客户机已经mounted的文件系统；如果客户机没有安装QEMU GA，则操作会失败。
    --atomic         require atomic operation #快照要么完全成功要么完全失败，不允许部分成果。不是所有的VMM都支持。
    --live           take a live snapshot     #当客户机处于运行状态下做快照
    --memspec <string>  memory attributes: [file=]name[,snapshot=type]
    [--diskspec] <string>  disk attributes: disk[,snapshot=type][,driver=type][,file=name]
```

其中一些参数，比如 --atomic，在一些老的 QEMU libary 上不支持，需要更新它到新的版本。根据 这篇文章，atomic 应该是 QEMU 1.0 中加入的。
（1）默认的话，该命令创建虚机的所有磁盘和内存做内部快照，创建快照时虚机处于 paused 状态，快照完成后变为 running 状态。持续时间较长。

```php
<memory snapshot='internal'/>
  <disks>
    <disk name='vda' snapshot='internal'/>
    <disk name='vdb' snapshot='internal'/>
    <disk name='vdc' snapshot='internal'/>
  </disks>
```

每个磁盘的镜像文件都包含了 snapshot 的信息：
```php
[root@compute144 8984f8b3-8429-4d16-a6f6-bff239605f46]# qemu-img info disk
image: disk
file format: qcow2
virtual size: 10G (10737418240 bytes)
disk size: 745M
cluster_size: 65536
backing file: /var/lib/nova/instances/_base/af01f8737b75e5ccf8a5f51ce051bde57b71fdfb
Snapshot list:
ID        TAG                 VM SIZE                DATE       VM CLOCK
1         123                       0 2021-01-15 16:30:00   00:00:00.000
2         12345                  230M 2021-01-15 16:40:42   00:07:27.665
3         345                    229M 2021-01-15 16:57:44   00:07:16.928
Format specific information:
    compat: 1.1
    lazy refcounts: false
    refcount bits: 16
    corrupt: false
```
你可以运行 snapshot-revert 命令回滚到指定的snapshot。

virsh # snapshot-revert instance-0000002e 1433950148

根据 这篇文章，libvirt 将内存状态保存到某一个磁盘镜像文件内 （”state is saved inside one of the disks (as in qemu's 'savevm'system checkpoint implementation). If needed in the future,we can also add an attribute pointing out _which_ disk saved the internal state; maybe disk='vda'.）

（2）可以使用 “--memspec” 和 “--diskspec” 参数来给内存和磁盘外部快照。这时候，在获取内存状态之前需要 Pause 虚机，就会产生服务的 downtime。

```php
virsh # snapshot-create-as 0000002e livesnap2  --memspec /home/s1/livesnap2mem,snapshot=external --diskspec vda,snapshot=external
Domain snapshot livesnap2 created
virsh # snapshot-dumpxml 0000002e livesnap2
  <memory snapshot='external' file='/home/s1/livesnap2mem'/>
  <disks>
    <disk name='vda' snapshot='external' type='file'>
      <driver type='qcow2'/>
      <source file='/home/s1/testvm/testvm1.livesnap2'/>
    </disk>
  </disks>
```

（3）可以使用 “--disk-only” 参数，这时会做所有磁盘的外部快照，但是不包含内存的快照。不指定快照文件名字的话，会放在原来的磁盘文件所在的目录中。多次快照后，会形成一个外部快照链，新的快照使用前一个快照的镜像文件作为 backing file。


```php
virsh # snapshot-list instance-0000002e --tree
1433950148 #内部快照
1433950810 #内部快照
1433950946 #内部快照
snap1 #第一个外部快照
  |
  +- snap2 #第二个外部快照
      |
      +- 1433954941 #第三个外部快照
          |
          +- 1433954977 #第四个外部快照

```

而第一个外部快照的镜像文件是以虚机的原始镜像文件作为 backing file 的：

```php
root@compute1:/var/lib/nova/instances/eddc46a8-e026-4b2c-af51-dfaa436fcc7b# qemu-img info disk.snap1
image: disk.snap1
file format: qcow2
virtual size: 30M (31457280 bytes)
disk size: 196K
cluster_size: 65536
backing file: /var/lib/nova/instances/eddc46a8-e026-4b2c-af51-dfaa436fcc7b/disk.swap #虚机的 swap disk 原始镜像文件
backing file format: qcow2
Format specific information:
    compat: 1.1
    lazy refcounts: false
```
目前还不支持回滚到某一个extrenal disk snapshot。这篇文章 谈到了一个workaround。

```php
[root@rh65 osdomains]# virsh snapshot-revert d-2 1434467974
error: unsupported configuration: revert to external disk snapshot not supported yet
```

（4）还可以使用 “--live” 参数创建系统还原点，包括磁盘、内存和设备状态等。使用这个参数时，虚机不会被 Paused（那怎么实现的？）。其后果是增加了内存 dump 文件的大小，但是减少了系统的 downtime。该参数只能用于做外部的系统还原点（external checkpoint）。

```php
virsh # snapshot-create-as 0000002e livesnap3  --memspec /home/s1/livesnap3mem,snapshot=external --diskspec vda,snapshot=external --live
Domain snapshot livesnap3 created
virsh # snapshot-dumpxml 0000002e livesnap3
  <memory snapshot='external' file='/home/s1/livesnap3mem'/>
  <disks>
    <disk name='vda' snapshot='external' type='file'>
      <driver type='qcow2'/>
      <source file='/home/s1/testvm/testvm1.livesnap3'/>
    </disk>
  </disks>
```

注意到加 "--live" 生成的快照和不加这个参数生成的快照不会被链在一起：
```php
virsh # snapshot-list 0000002e --tree
livesnap1 #没加 --live
  |
  +- livesnap2 #没加 --live

livesnap3 #加了 --live
  |
  +- livesnap4 #加了 --live
```

不过，奇怪的是，使用 QEMU 2.3 的情况下，即使加了 --live 参数，虚机还是会被短暂的 Paused 住：
```php
[root@rh65 ~]# virsh snapshot-create-as d-2 --memspec /home/work/d-2/mem3,snapshot=external --diskspec hda,snapshot=external --live
Domain snapshot 1434478667 created

[root@rh65 ~]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 40    osvm1                          running
 42    osvm2                          running
 43    d-2                            running

[root@rh65 ~]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 40    osvm1                          running
 42    osvm2                          running
 43    d-2                            paused

[root@rh65 ~]# virsh list --all
 Id    Name                           State
----------------------------------------------------
 40    osvm1                          running
 42    osvm2                          running
 43    d-2                            running
```

![](/images/snapshot4.png)

可以使用 snapshot-revert 命令来回滚到指定的系统还原点，不过得使用 "-force" 参数 。
