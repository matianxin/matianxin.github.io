---
title: Openstack Queens 环境搭建（二）环境相关服务
tags: [openstack]
categories: Openstack
description: OpenStack项目是一个开源云计算平台，支持所有类型的云环境。该项目旨在实现简单，大规模的可扩展性和丰富的功能。OpenStack通过各种补充服务提供基础架构即服务（IaaS）解决方案。每项服务都提供了一个应用程序编程接口（API），以促进这种集成。本文涵盖了使用适用于具有足够Linux经验的OpenStack新用户的功能性示例体系结构，逐步部署主要OpenStack服务。
date: 2019/3/19 16:25:00
---
### Controller节点：
#### 安装NTP服务
```php
# yum install chrony

# vi /etc/chrony.conf

server 0.centos.pool.ntp.org iburst
server 1.centos.pool.ntp.org iburst
server 2.centos.pool.ntp.org iburst
server 3.centos.pool.ntp.org iburst
...
allow 192.100.10.0/24
...

# systemctl enable chronyd.service                            开机启用NTP
# systemctl start chronyd.service                             开启NTP服务
```

验证NTP服务：
```php
# chronyc sources

  210 Number of sources = 2
  MS Name/IP address         Stratum Poll Reach LastRx Last sample
  ===============================================================================
  ^- 192.0.2.11                    2   7    12   137  -2814us[-3000us] +/-   43ms
  ^* 192.0.2.12                    2   6   177    46    +17us[  -23us] +/-   68ms
```

#### 安装Openstack相关库
```php
# yum install centos-release-openstack-queens                 安装Openstack库
# yum upgrade                                                 更新包
# yum install python-openstackclient                          安装Openstack客户端
# yum install openstack-selinux                               安装openstack-selinux用来管理Openstack服务的安全策略
```

#### 关闭防火墙
```php
# systemctl stop firewalld                                    关闭防火墙服务
# systemctl disable firewalld                                 永久防火墙开机自启动
```

#### 关闭SELINUX服务
```php
# setenforce 0                                               关闭selinux服务
# vi /etc/selinux/config                                     永久关闭selinux服务
    SELINUX=disabled
```

#### 安装数据库服务
```php
# yum install mariadb mariadb-server python2-PyMySQL

# vi /etc/my.cnf.d/openstack.cnf

[mysqld]
bind-address = 192.100.10.160
default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8


# systemctl enable mariadb.service                     开机启用Mysql服务
# systemctl start mariadb.service                      开启Mysql服务
# mysql_secure_installation                            设置Mysql密码->123456
```

#### 安装消息队列
```php
# yum install rabbitmq-server

# systemctl enable rabbitmq-server.service
# systemctl start rabbitmq-server.service

# rabbitmqctl add_user openstack 123456
# rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```

#### 安装Memcached缓存
```php
# yum install memcached python-memcached

# vi /etc/sysconfig/memcached
OPTIONS="-l 127.0.0.1,::1,controller"

# systemctl enable memcached.service
# systemctl start memcached.service
```

#### 安装Etcd
```php
# yum install etcd

# vi /etc/etcd/etcd.conf
#[Member]
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="http://192.100.10.160:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.100.10.160:2379"
ETCD_NAME="controller"
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://192.100.10.160:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.100.10.160:2379"
ETCD_INITIAL_CLUSTER="controller=http://192.100.10.160:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-01"
ETCD_INITIAL_CLUSTER_STATE="new"

# systemctl enable etcd
# systemctl start etcd
```

### Compute节点：

#### 安装NTP服务
```php
# yum install chrony

# vi /etc/chrony.conf

server controller iburst
...
allow 192.100.10.0/24
...

# systemctl enable chronyd.service                            开机启用NTP
# systemctl start chronyd.service                             开启NTP服务
```

#### 安装Openstack相关库
```php
# yum install centos-release-openstack-queens                 安装Openstack库
# yum upgrade                                                 更新包
# yum install python-openstackclient                          安装Openstack客户端
# yum install openstack-selinux                               安装openstack-selinux用来管理Openstack服务的安全策略
```

#### 关闭防火墙
```php
# systemctl stop firewalld                                    关闭防火墙服务
# systemctl disable firewalld                                 永久防火墙开机自启动
```

#### 关闭SELINUX服务
```php
# setenforce 0                                               关闭selinux服务
# vi /etc/selinux/config                                     永久关闭selinux服务
    SELINUX=disabled
```
