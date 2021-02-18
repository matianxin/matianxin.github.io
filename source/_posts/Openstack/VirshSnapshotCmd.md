---
title: QEMU / KVM 快照相关命令介绍
tags: [kvm]
categories: KVM
description: KVM 介绍：使用 libvirt 做 QEMU/KVM 内部快照
date: 2021/01/19 11:30:00
---

• INSTANCE_NAME： 实例instance名称
• SNAPSHOT_NAME： 快照名称
• SNAPSHOT_DESCRIPTION： 快照描述

1.创建快照命令：
**virsh snapshot-create-as --domain INSTANCE_NAME --name SNAPSHOT_NAME --description SNAPSHOT_DESCRIPTION**
```php
[root@compute144 ~]# virsh snapshot-create-as --domain instance-00001286 --name scene-init --description scene-init
Domain snapshot scene-init created
```

2.删除快照命令：
**virsh snapshot-delete INSTANCE_NAME SNAPSHOT_NAME**
```php
[root@compute144 ~]# virsh snapshot-delete instance-00001286 scene-init
Domain snapshot scene-init deleted
```

3.恢复快照命令：
**virsh snapshot-revert INSTANCE_NAME SNAPSHOT_NAME**
```php
[root@compute144 ~]# virsh snapshot-revert instance-00001286 scene-init
```

4.查看快照disk信息
**qemu-img info disk**
```php
根据instance_name得到实例的uuid
[root@compute144 ~]# cd /var/lib/nova/instances/27a7b4d6-9036-4625-99f2-dbe6141d7eb7
[root@compute144 27a7b4d6-9036-4625-99f2-dbe6141d7eb7]# qemu-img info disk
image: disk
file format: qcow2
virtual size: 10G (10737418240 bytes)
disk size: 445M
cluster_size: 65536
backing file: /var/lib/nova/instances/_base
ID        TAG                 VM SIZE                DATE       VM CLOCK
1         scene-init             253M 2021-01-18 11:48:18   02:25:45.194
Format specific information:
    compat: 1.1
    lazy refcounts: false
    refcount bits: 16
    corrupt: fals

* 此功能必须为虚拟机关机状态下生效，否则：
qemu-img: Could not open 'disk': Failed to get shared "write" lock
Is another process using the image [disk]?
```

5.查看快照列表
**virsh snapshot-list INSTANCE_NAME**
```php
[root@compute144 ~]# virsh snapshot-list instance-00001286
 Name                 Creation Time             State
------------------------------------------------------------
 scene-init           2021-01-19 08:51:22 +0800 shutoff
```

6.查看快照信息
**virsh snapshot-info INSTANCE_NAME --current**
```php
[root@compute144 ~]# virsh snapshot-info instance-00001286 --current
Name:           scene-init
Domain:         instance-00001286
Current:        yes
State:          shutoff
Location:       internal
Parent:         -
Children:       0
Descendants:    0
Metadata:       yes

```

7.查看当前快照所有信息
**virsh snapshot-current INSTANCE_NAME**
```php
[root@compute144 ~]# virsh snapshot-current instance-00001286
<domainsnapshot>
  <name>scene-init</name>
  <description>scene-init</description>
  <state>shutoff</state>
  <creationTime>1611017482</creationTime>
  <memory snapshot='no'/>
  <disks>
    <disk name='vda' snapshot='internal'/>
    <disk name='hda' snapshot='no'/>
  </disks>
  <domain type='kvm'>
    <name>instance-00001286</name>
    <uuid>27a7b4d6-9036-4625-99f2-dbe6141d7eb7</uuid>
    <metadata>
      <nova:instance xmlns:nova="http://openstack.org/xmlns/libvirt/nova/1.0">
        <nova:package version="17.0.10-1.el7"/>
        <nova:name>ps_T15Z2PS-SCENE220@-1</nova:name>
        <nova:creationTime>2021-01-18 06:41:30</nova:creationTime>
        <nova:flavor name="&#x4E3B;&#x7AD9;&#x7EB5;&#x5411;">
          <nova:memory>1024</nova:memory>
          <nova:disk>10</nova:disk>
          <nova:swap>0</nova:swap>
          <nova:ephemeral>0</nova:ephemeral>
          <nova:vcpus>1</nova:vcpus>
        </nova:flavor>
        <nova:owner>
          <nova:user uuid="d5853486b6904865abaf98e1eedfd926">admin</nova:user>
          <nova:project uuid="a385d3107887463abebad53b48dda754">admin</nova:project>
        </nova:owner>
        <nova:root type="image" uuid="ebe94aab-979a-4311-bf59-9be20a3f44e7"/>
      </nova:instance>
    </metadata>
    <memory unit='KiB'>1048576</memory>
    <currentMemory unit='KiB'>1048576</currentMemory>
    <vcpu placement='static'>1</vcpu>
    <cputune>
      <shares>1024</shares>
    </cputune>
    <sysinfo type='smbios'>
      <system>
        <entry name='manufacturer'>RDO</entry>
        <entry name='product'>OpenStack Compute</entry>
        <entry name='version'>17.0.10-1.el7</entry>
        <entry name='serial'>12c77e03-1355-43af-a482-61ecf2aad317</entry>
        <entry name='uuid'>27a7b4d6-9036-4625-99f2-dbe6141d7eb7</entry>
        <entry name='family'>Virtual Machine</entry>
      </system>
    </sysinfo>
    <os>
      <type arch='x86_64' machine='pc-i440fx-rhel7.6.0'>hvm</type>
      <boot dev='hd'/>
      <smbios mode='sysinfo'/>
    </os>
    <features>
      <acpi/>
      <apic/>
    </features>
    <cpu mode='host-model' check='partial'>
      <model fallback='allow'/>
      <topology sockets='1' cores='1' threads='1'/>
    </cpu>
    <clock offset='utc'>
      <timer name='pit' tickpolicy='delay'/>
      <timer name='rtc' tickpolicy='catchup'/>
      <timer name='hpet' present='no'/>
    </clock>
    <on_poweroff>destroy</on_poweroff>
    <on_reboot>restart</on_reboot>
    <on_crash>destroy</on_crash>
    <devices>
      <emulator>/usr/libexec/qemu-kvm</emulator>
      <disk type='file' device='disk'>
        <driver name='qemu' type='qcow2' cache='none'/>
        <source file='/var/lib/nova/instances/27a7b4d6-9036-4625-99f2-dbe6141d7eb7/disk'/>
        <target dev='vda' bus='virtio'/>
        <address type='pci' domain='0x0000' bus='0x00' slot='0x07' function='0x0'/>
      </disk>
      <disk type='file' device='cdrom'>
        <driver name='qemu' type='raw' cache='none'/>
        <source file='/var/lib/nova/instances/27a7b4d6-9036-4625-99f2-dbe6141d7eb7/disk.config'/>
        <target dev='hda' bus='ide'/>
        <readonly/>
        <address type='drive' controller='0' bus='0' target='0' unit='0'/>
      </disk>
      <controller type='ide' index='0'>
        <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x1'/>
      </controller>
      <controller type='usb' index='0' model='piix3-uhci'>
        <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x2'/>
      </controller>
      <controller type='pci' index='0' model='pci-root'/>
      <interface type='bridge'>
        <mac address='fa:16:3e:70:8f:e9'/>
        <source bridge='qbr9a416b99-f4'/>
        <target dev='tap9a416b99-f4'/>
        <model type='virtio'/>
        <mtu size='1450'/>
        <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
      </interface>
      <interface type='bridge'>
        <mac address='fa:16:3e:cb:8b:c6'/>
        <source bridge='qbr4e232a0d-3e'/>
        <target dev='tap4e232a0d-3e'/>
        <model type='virtio'/>
        <mtu size='1500'/>
        <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x0'/>
      </interface>
      <interface type='bridge'>
        <mac address='fa:16:3e:dd:4e:8d'/>
        <source bridge='qbr3a023c95-a5'/>
        <target dev='tap3a023c95-a5'/>
        <model type='virtio'/>
        <mtu size='1500'/>
        <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x0'/>
      </interface>
      <interface type='bridge'>
        <mac address='fa:16:3e:66:a0:b7'/>
        <source bridge='qbr0c33b092-d0'/>
        <target dev='tap0c33b092-d0'/>
        <model type='virtio'/>
        <mtu size='1450'/>
        <address type='pci' domain='0x0000' bus='0x00' slot='0x06' function='0x0'/>
      </interface>
      <serial type='pty'>
        <log file='/var/lib/nova/instances/27a7b4d6-9036-4625-99f2-dbe6141d7eb7/console.log' append='off'/>
        <target type='isa-serial' port='0'>
          <model name='isa-serial'/>
        </target>
      </serial>
      <console type='pty'>
        <log file='/var/lib/nova/instances/27a7b4d6-9036-4625-99f2-dbe6141d7eb7/console.log' append='off'/>
        <target type='serial' port='0'/>
      </console>
      <input type='tablet' bus='usb'>
        <address type='usb' bus='0' port='1'/>
      </input>
      <input type='mouse' bus='ps2'/>
      <input type='keyboard' bus='ps2'/>
      <graphics type='vnc' port='-1' autoport='yes' listen='0.0.0.0' keymap='en-us'>
        <listen type='address' address='0.0.0.0'/>
      </graphics>
      <video>
        <model type='cirrus' vram='16384' heads='1' primary='yes'/>
        <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x0'/>
      </video>
      <memballoon model='virtio'>
        <stats period='10'/>
        <address type='pci' domain='0x0000' bus='0x00' slot='0x08' function='0x0'/>
      </memballoon>
    </devices>
  </domain>
</domainsnapshot>
```
