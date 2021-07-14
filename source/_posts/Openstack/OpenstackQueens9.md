---
title: Openstack Queens 环境搭建（九）Cinder服务
tags: [openstack]
categories: Openstack
description: OpenStack项目是一个开源云计算平台，支持所有类型的云环境。该项目旨在实现简单，大规模的可扩展性和丰富的功能。OpenStack通过各种补充服务提供基础架构即服务（IaaS）解决方案。每项服务都提供了一个应用程序编程接口（API），以促进这种集成。本文涵盖了使用适用于具有足够Linux经验的OpenStack新用户的功能性示例体系结构，逐步部署主要OpenStack服务。
date: 2019/8/1 16:20:00
---

#### 存储服务概念
OpenStack块存储服务(Cinder)将持久性存储添加到一个虚拟机。块存储提供了管理卷的基础设施，并与OpenStack计算交互，为实例提供卷。该服务还支持卷快照和卷类型的管理。
块存储服务由以下组件组成:
**cinder-api**
接收API请求，并将其路由到cinders -volume以进行操作。
**cinder-volume**
直接与块存储服务和进程(如cinders -scheduler)交互。它还可以通过消息队列与这些进程进行交互。cinders -volume服务响应发送到块存储服务的读写请求，以维护状态。它可以通过驱动程序体系结构与各种存储提供者交互。
**cinder-scheduler daemon**
选择要在其上创建卷的最佳存储提供程序节点。与nova-scheduler类似的组件。
**cinder-backup daemon**
Cinder-backup服务向备份存储提供程序提供任何类型的备份卷。与cinders -volume服务一样，它可以通过驱动程序体系结构与各种存储提供者交互。
**Messaging queue**
在块存储进程之间路由信息。

### Controller节点：

#### 基础配置
在安装和配置块存储服务之前，必须创建数据库、服务凭据和API端点。

#### 创建数据库，为cinder数据库授权
```php
# mysql -uroot -p123456
MariaDB [(none)]> create database cinder;
MariaDB [(none)]> grant all privileges on cinder.* to 'cinder'@'localhost' identified by '123456';
MariaDB [(none)]> grant all privileges on cinder.* to 'cinder'@'%' identified by '123456';
```

#### 创建服务凭据
生成管理凭证，以获得访问只有管理CLI命令:
```php
# . admin-openrc
```

创建cinder用户：
```php
# openstack user create --domain default --password-prompt cinder
```

添加admin角色到cinder用户中： （无返回值）
```php
# openstack role add --project service --user cinder admin
```

创建cinder2和cinderv3服务实体:
注：块存储服务需要两个服务实体。
```php
# openstack service create --name cinderv2 \
   --description "OpenStack Block Storage" volumev2
# openstack service create --name cinderv3 \
   --description "OpenStack Block Storage" volumev3
```

```php
# openstack endpoint create --region RegionOne \
   volumev3 public http://controller:8776/v3/%\(project_id\)s
# openstack endpoint create --region RegionOne \
   volumev3 internal http://controller:8776/v3/%\(project_id\)s
# openstack endpoint create --region RegionOne \
  volumev3 admin http://controller:8776/v3/%\(project_id\)s
```

#### 安装配置组件
```php
# yum install -y openstack-cinder
```

#### 编辑 /etc/cinder/cinder.conf文件
```php
# vi  /etc/cinder/cinder.conf
```
 - 在[database]选项，配置数据库访问:
```php
[database]
connection = mysql+pymysql://cinder:123456@controller/cinder
```

- 在[DEFAULT]部分，配置RabbitMQ消息队列访问:
```php
[DEFAULT]
transport_url = rabbit://openstack:123456@controller
```

- 在[DEFAULT]和[keystone_authtoken]选项，配置身份服务访问:
```php
[DEFAULT]
auth_strategy = keystone
[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_id = default
user_domain_id = default
project_name = service
username = cinder
password = 123456
```

- 在[DEFAULT]选项，配置my_ip选项，使用Controller节点的管理接口IP地址:
```php
[DEFAULT]
my_ip = 192.100.200.68
```

- 在[oslo_concurrency]选项，配置lock路径:
```php
[oslo_concurrency]
lock_path = /var/lib/cinder/tmp
```

- 同步块存储数据（忽略此输出中的任何弃用消息。）
```php
# su -s /bin//sh -c "cinder-manage db sync" cinder
```

- 配置计算服务以使用块存储
编辑 /etc/nova/nova.conf 文件
```php
# vi /etc/nova/nova.conf
[cinder]
os_regioon_name = RegionOne
```

#### 启动服务
```php
# systemctl restart openstack-nova-api.service
# systemctl enable openstack-cinder-api.service openstack-cinder-scheduler.service
# systemctl start openstack-cinder-api.service openstack-cinder-scheduler.service
```

### 存储节点：

#### 安装LVM包:
```php
# yum install -y lvm2
```

#### 启动LVM元数据服务，并将其配置为在系统启动时启动:
注：一些发行版默认安装LVM。
```php
# systemctl enable lvm2-lvmetad.service
# systemctl start lvm2-lvmetad.service
```

#### 创建LVM物理卷/dev/sdb:
```php
# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
```

#### 创建LVM卷组cinder-volmes:
注：块存储服务在此卷组中创建逻辑卷。

```php
# vgcreate cinder-volumes /dev/sdb
  Volume group "cinder-volumes" successfully created
```

只有实例可以访问块存储卷。然而，底层操作系统管理与卷相关联的设备。默认情况下，LVM卷扫描工具扫描/dev目录中包含卷的块存储设备。如果项目在其卷上使用LVM，那么扫描工具将检测这些卷并试图缓存它们，这会导致底层操作系统和项目卷出现各种问题。必须重新配置LVM，以便只扫描包含cinder-volume卷组的设备。

#### 编辑 /etc/lvm/lvm.conf文件

在devices选项，添加一个接受/dev/sdb设备并拒绝所有其他设备的过滤器:
注：filter数组中的每个项都以for accept或r for reject开头，并包含一个用于设备名称的正则表达式。数组必须以r/结束。
*/ 拒绝任何剩余设备。您可以使用vgs -vvvv命令来测试过滤器。


```php
devices {
...
filter = [ "a/sdb/", "r/.*/"]
...}
```

#### 安装配置组件
```php
# yum install -y openstack-cinder targetcli python-keystone
```

#### 编辑/etc/cinder/cinder.conf 文件
```php
# vi /etc/cinder/cinder.conf
```

- 在[database]选项，配置数据库访问
```php
[database]
connection = mysql+pymysql://cinder:123456@controller/cinder
```

- 在[DEFAULT]选项，配置RabbitMQ消息队列访问:
```php
[DEFAULT]
transport_url = rabbit://openstack:123456@controller
```

- 在[DEFAULT]和[keystone_authtoken]选项，配置身份服务访问:
```php
[DEFAULT]
auth_strategy = keystone
[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_id = default
user_domain_id = default
project_name = service
username = cinder
password = 123456
```

- 在[DEFAULT]部分，配置my_ip选项:
```php
my_ip = 192.168.100.69
```

- 在[DEFAULT]选项，配置映像服务API的位置:
```php
glance_api_servers = http://controller:9292
```

- 在[lvm]选项，使用lvm驱动程序、Cinder卷组、iSCSI协议和iSCSI服务配置lvm后端。如果[lvm]部分不存在，则添加它:
```php
[lvm]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_group = cinder-volumes
iscsi_protocol = iscsi
iscsi_helper = lioadm
```

- 在[DEFAULT]部分，启用LVM后端:
```php
[DEFAULT]
enabled_backends = lvm
```

- 在[oslo_concurrency]节中，配置lock路径
```php
[oslo_concurrency]
lock_path = /var/lib/cinder/tmp
```

#### 开启服务
启动块存储卷服务，包括它的依赖项，并配置它们在系统启动时启动:
```php
# systemctl enable openstack-cinder-volume.service target.service
# systemctl restart openstack-cinder-volume.service target.service
```

#### 校验操作
生成临时环境变量
```php
# . admin-openrc
```

列出服务组件以验证每个流程的成功启动:
```php
# openstack volume service list
```

创建一个1G的卷，并查看其状态
```php
# cinder create --display-name myVolume 1
# cinder list
```
