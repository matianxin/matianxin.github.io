---
title: Openstack Queens 环境搭建（一）环境准备
tags: [openstack]
categories: Openstack
description: OpenStack项目是一个开源云计算平台，支持所有类型的云环境。该项目旨在实现简单，大规模的可扩展性和丰富的功能。OpenStack通过各种补充服务提供基础架构即服务（IaaS）解决方案。每项服务都提供了一个应用程序编程接口（API），以促进这种集成。本文涵盖了使用适用于具有足够Linux经验的OpenStack新用户的功能性示例体系结构，逐步部署主要OpenStack服务。
date: 2019/6/20 17:20:00
---

环境准备：
基于CentOS Linux release 7.6.1810 (Core)

控制节点（Controller）：
eth0：192.100.10.160/24
eth1：10.0.0.11/24
eth2：预留

计算节点（Compute):
eth0：192.100.10.161/24
eth1：10.0.0.12/24
eth2：预留

网卡0接口为管理网络 -> 交换机 + 路由器
网卡1接口为Overlay网络 -> 目前直连 / 交换机连接
网卡2接口为外部网络 -> 路由器
 -（可以先使用eth1作为外部网络下载Openstack安装所需资源，后修改）

通用密码： 123456

### Controller节点：
配置网卡信息：

```php
# vi /etc/sysconfig/network-scripts/ifcfg-eth0
BOOTPROTO=static
IPADDR=192.100.10.160
NETMASK=255.255.255.0
GATEWAY=192.100.10.1
```

```php
# vi /etc/sysconfig/network-scripts/ifcfg-eth1
BOOTPROTO=static
IPADDR=10.0.0.11
NETMASK=255.255.255.0
```

配置主机信息：

```php
# vi /etc/hosts
# controller
192.100.10.160     controller
# compute
192.100.10.161     compute
```
配置主机名：
控制节点的主机名为controller，设置如下：
```php
~# hostnamectl set-hostname controller
```

对主机名进行验证：
```php
~# hostname
```
看到输出为controller即可

配置DNS：

```php
# vi /etc/resolv.conf
nameserver 114.114.114.114
```

### Compute节点：

配置管理网络：
```php
# vi /etc/sysconfig/network-scripts/ifcfg-eth0
BOOTPROTO=static
IPADDR=192.100.10.161
NETMASK=255.255.255.0
GATEWAY=192.100.10.1
```
```php
# vi /etc/sysconfig/network-scripts/ifcfg-eth1
BOOTPROTO=static
IPADDR=10.0.0.21
NETMASK=255.255.255.0
```
配置主机信息：

```php
# vi /etc/hosts
# controller
192.100.10.160     controller
# compute
192.100.10.161     compute
```
配置主机名：
计算节点的主机名为compute，设置如下：
```php
~# hostnamectl set-hostname compute
```
对主机名进行验证：
```php
~# hostname
```
看到输出为compute即可

配置DNS：
```php
# vi /etc/resolv.conf
nameserver 114.114.114.114
```
