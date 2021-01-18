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
内部快照 - 使用单个的 qcow2 的文件来保存快照和快照之后的改动。这种快照是 libvirt 的默认行为，现在的支持很完善（创建、回滚和删除），但是只能针对 qcow2 格式的磁盘镜像文件，而且其过程较慢等。
外部快照 - 快照是一个只读文件，快照之后的修改是另一个 qcow2 文件中。外置快照可以针对各种格式的磁盘镜像文件。外置快照的结果是形成一个 qcow2 文件链：original <- snap1 <- snap2 <- snap3。这里有文章详细讨论外置快照。
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

snapshot | 做快照的 libvirt API | 从快照恢复的 libvirt API | virsh 命令
磁盘快照 | virDomainSnapshotCreateXML（flags = VIR_DOMAIN_SNAPSHOT_CREATE_DISK_ONLY ）| virDomainRevertToSnapshot | virsh snapshot-create/snapshot-revert

内存（状态）快照 | virDomainSave
virDomainSaveFlags
virDomainManagedSave |
virDomainRestore
virDomainRestoreFlags
virDomainCreate
virDomainCreateWithFlags |
virsh save/restore

系统检查点 | virDomainSnapshotCreateXML | virDomainRevertToSnapshot | virsh snapshot-create/snapshot-revert


