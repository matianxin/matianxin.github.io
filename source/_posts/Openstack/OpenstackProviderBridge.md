---
title: CentOS网卡配置文件设置为网桥模式
tags: [linux]
categories: Linux
description: CentOS网卡配置文件设置为网桥模式
date: 2019/11/8 8:32:00
---

#### 原eth0配置文件

**/etc/sysconfig/network-scripts/ifcfg-eth0**
```php
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=no
IPV4_FAILURE_FATAL=no
NAME=eth0
UUID=63754cd8-9152-44a4-b051-ea88c0182bd4
DEVICE=eth0
ONBOOT=yes
IPADDR=191.100.200.30
NETMASK=255.255.255.0
GATEWAY=192.100.10.161
```

#### 改成网桥配置后,eth0配置文件及br-provider网桥配置文件

**/etc/sysconfig/network-scripts/ifcfg-eth0**
```php
TYPE=OVSPort
DEVICETYPE=ovs
NAME=eth0
UUID=63754cd8-9152-44a4-b051-ea88c0182bd4
DEVICE=eth0
ONBOOT=yes
IPADDR=0.0.0.0
NETMASK=255.255.255.0
OVS_BRIDGE=br-provider
```

**/etc/sysconfig/network-scripts/ifcfg-br-provider**
```php
DEVICE=br-provider
DEVICETYPE=ovs
TYPE=OVSBridge
BOOTPROTO=static

IPADDR=192.100.200.30
NETMASK=255.255.255.0
GATEWAY=192.100.10.161
ONBOOT=yes
```
