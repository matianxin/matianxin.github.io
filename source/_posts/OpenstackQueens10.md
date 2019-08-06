---
title: Openstack Queens 环境搭建（十）Ceilometer + Gnocchi
tags: [openstack]
categories: Openstack
description: OpenStack项目是一个开源云计算平台，支持所有类型的云环境。该项目旨在实现简单，大规模的可扩展性和丰富的功能。OpenStack通过各种补充服务提供基础架构即服务（IaaS）解决方案。每项服务都提供了一个应用程序编程接口（API），以促进这种集成。本文涵盖了使用适用于具有足够Linux经验的OpenStack新用户的功能性示例体系结构，逐步部署主要OpenStack服务。
date: 2019/8/2 14:20:00
---

### ceilometer概述

#### 什么是ceilometer
Ceilometer是OpenStack中的一个子项目，它像一个漏斗一样，能把OpenStack内部发生的几乎所有的事件都收集起来，然后为计费和监控以及其它服务提供数据支撑。

Ceilometer服务主要功能：
- 有效地轮询与OpenStack服务相关的计量数据。
- 通过监视从服务发送的通知来收集事件和计量数据。
- 将收集的数据发布到各种目标，包括数据存储和消息队列。

#### ceilometer如何进行监控
Ceilometer监控通过在计算节点部署Compute服务，轮询其计算节点上的instance，获取各自的CPU、网络、磁盘等监控信息，发送到RabbitMQ，Collector服务负责接收信息进行持久化存储，也可通过libvirt和openstack服务的API采集数据。

#### 监控信息存储
Ceilometer以前提供了存储和API解决方案。截至N版本，此功能已被正式弃用并且不鼓励。
为了有效存储和统计分析ceilometer数据，建议使用Gnocchi。
采集信息的存储可以有多种，如文件，数据库等，openstack queens版本默认采用gnocchi+文件形式存储。

### gnocchi概述

#### gnocchi是什么
Gnocchi是一个多租户时间序列，计量和资源数据库。
提供了HTTP REST接口来创建和操作数据。
Gnocchi设计用于超大规模计量数据的存储，同时向操作者和用户提供对度量和资源信息的访问。
Gnocchi是OpenStack项目的一部分。因此它支持OpenStack，但也能完全独立的工作。

#### gnocchi项目起源
早期的openstack各类资源的计量数据(measurement) 存储在SQL数据库中的sample表中。随着云环境中需要被监控的资源增多和时间的推移，计量数据的增长变得难以预测；计量数据的使用方面，查询操作首先要从巨大的sample单表中过滤所需条目，然后还会涉及到相关的聚合计算；可想而知，由此带来的性能开销绝对是无法忍受的，并且随着时间的推移这个瓶颈会愈加明显直至奔溃。要解决上述问题方法有很多，比如分表：每个监控指标（Metirc）一张表，那么一个资源可能会有多张表（比如一个instance至少会有cpu，cpu.util，memory，memory.usage，disk.* 等监控指标metrics）；这似乎有点夸张，即使这样都可以接受的话，那么查询时对计量数据的聚合操作也还是个问题。
为解决以上问题，红帽的Julien Danjou，发起了Gnocchi项目来解决这类问题。其总体思路是：把各个计量指标Metric的计量数据measurement直接写入后端存储中；并在measurement写入之前根据预先设定的归档策略进行聚合操作；查询时直接读取对应的文件即可获得聚合后的监控信息点；gnocchi提供资源索引，这样能更快的找到每个资源的基础信息metadata和其相关的metrics信息。
注：数据存储分级：(资源项)resource->(资源指标条目)metric->(实际数据)measure

#### 为什么使用gnocchi
Gnocchi已能够具备在云计算环境中提供可用的时间序列数据库的需要，提供存储大量度量数据并且易于扩展的能力。
Gnocchi项目于2014年开始，作为OpenStack Ceilometer项目的分支，以解决Ceilometer在将标准数据库用作计量数据的存储后端时遇到的性能问题。
Gnocchi使用各种技术压缩存储数据，降低了数据存储的空间。

#### ceilometer+gnocchi环境搭建
安装部署ceilometer服务前，主机必须已经正确安装keystone、nova、neutron、image服务。部署过程须用到uwsgi和redis。

### Compute节点安装

#### 安装ceilometer相关包：
```php
# yum install openstack-ceilometer-compute
# yum install openstack-ceilometer-ipmi (optional)
```

#### 编辑/etc/ceilometer/ceilometer.conf文件并完成以下操作
- 在该[DEFAULT]部分中，配置RabbitMQ 消息队列访问
```php
[DEFAULT]
...
transport_url  =  rabbit://openstack:123456@controller
```

- 在该[service_credentials]部分中，配置服务凭据：
```php
[service_credentials]
auth_url = http://controller:5000
project_domain_id = default
user_domain_id = default
auth_type = password
username = ceilometer
project_name = service
password = 123456
interface = internalURL
region_name = RegionOne
```

- 配置计算服务，编辑/etc/nova/nova.conf文件并在以下[DEFAULT]部分配置通知：
```php
[DEFAULT]
...
instance_usage_audit  =  True
instance_usage_audit_period  =  hour
notify_on_state_change  =  vm_and_task_state
[oslo_messaging_notifications]
...
driver  =  messagingv2
```

- 配置计算以轮询IPMI，编辑/etc/sudoers文件并添加包含：
```php
ceilometer ALL = (root) NOPASSWD: /usr/bin/ceilometer-rootwrap /etc/ceilometer/rootwrap.conf *
```

- 编辑/etc/ceilometer/polling.yaml以包含所需：
```bash
- name: ipmi
      interval: 300
      meters:
        - hardware.ipmi.temperature
```

#### 完成安装
- 启动代理并将其配置为在系统引导时启动：
```php
# systemctl enable openstack-ceilometer-compute.service
# systemctl start openstack-ceilometer-compute.service
# systemctl enable openstack-ceilometer-ipmi.service (optional)
# systemctl start openstack-ceilometer-ipmi.service (optional)

```

- 重新启动Compute服务：
```php
# systemctl restart openstack-nova-compute.service
```

### Controller节点安装

#### 获取admin凭据来访问仅管理员CLI命令
```php
$ . admin-openrc
```

#### 要创建服务凭据，请完成以下步骤：

- 创建ceilometer用户：
```php
openstack user create --domain default --password-prompt ceilometer
User Password:123456
Repeat User Password:123456
+-----------+----------------------------------+
| Field     | Value                            |
+-----------+----------------------------------+
| domain_id | e0353a670a9e496da891347c589539e9 |
| enabled   | True                             |
| id        | c859c96f57bd4989a8ea1a0b1d8ff7cd |
| name      | ceilometer                       |
+-----------+----------------------------------+
```

- 将admin角色添加到ceilometer用户。
```php
$ openstack role add --project service --user ceilometer admin
```

#### 在Keystone注册Gnocchi服务

- 创建gnocchi用户：
```php
$ openstack user create --domain default --password-prompt gnocchi
User Password:123456
Repeat User Password:123456
+-----------+----------------------------------+
| Field     | Value                            |
+-----------+----------------------------------+
| domain_id | e0353a670a9e496da891347c589539e9 |
| enabled   | True                             |
| id        | 8bacd064f6434ef2b6bbfbedb79b0318 |
| name      | gnocchi                          |
+-----------+----------------------------------+
```

- 创建gnocchi服务实体
```php
$ openstack service create --name gnocchi \
  --description "Metric Service" metric
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Metric Service                   |
| enabled     | True                             |
| id          | 205978b411674e5a9990428f81d69384 |
| name        | gnocchi                          |
| type        | metric                           |
+-------------+----------------------------------+
```

- 将admin角色添加到gnocchi用户
```php
$ openstack role add --project service --user gnocchi admin
```

- 创建Ceilometer服务API端点
```php
$ openstack endpoint create --region RegionOne \
  metric public http://controller:8041
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | b808b67b848d443e9eaaa5e5d796970c |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 205978b411674e5a9990428f81d69384 |
| service_name | gnocchi                          |
| service_type | metric                           |
| url          | http://controller:8041           |
+--------------+----------------------------------+
```

```php
$ openstack endpoint create --region RegionOne \
  metric internal http://controller:8041
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | c7009b1c2ee54b71b771fa3d0ae4f948 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 205978b411674e5a9990428f81d69384 |
| service_name | gnocchi                          |
| service_type | metric                           |
| url          | http://controller:8041           |
+--------------+----------------------------------+
```

```php
$ openstack endpoint create --region RegionOne \
  metric admin http://controller:8041
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | b2c00566d0604551b5fe1540c699db3d |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 205978b411674e5a9990428f81d69384 |
| service_name | gnocchi                          |
| service_type | metric                           |
| url          | http://controller:8041           |
+--------------+----------------------------------+
```

#### 安装Gnocchi

- 安装服务组件
```php
# yum install openstack-gnocchi-api openstack-gnocchi-metricd python-gnocchiclient
```

- 新建文件 /etc/httpd/conf.d/10-gnocchi_wsgi.conf
```php
Listen 8041
<VirtualHost *:8041>
  ServerName controller
  ## Vhost docroot
  DocumentRoot "/var/www/cgi-bin/gnocchi"
  ## Directories, there should at least be a declaration for /var/www/cgi-bin/gnocchi
  <Directory "/var/www/cgi-bin/gnocchi">
    Options Indexes FollowSymLinks MultiViews
    AllowOverride None
    Require all granted
  </Directory>
  ## Logging
  ErrorLog "/var/log/httpd/gnocchi_wsgi_error.log"
  ServerSignature Off
  CustomLog "/var/log/httpd/gnocchi_wsgi_access.log" combined
  SetEnvIf X-Forwarded-Proto https HTTPS=1
  WSGIApplicationGroup %{GLOBAL}
  WSGIDaemonProcess gnocchi display-name=gnocchi_wsgi group=gnocchi processes=8 threads=8 user=gnocchi
  WSGIProcessGroup gnocchi
  WSGIScriptAlias / "/var/www/cgi-bin/gnocchi/app"
</VirtualHost>
```

- 创建文件夹路径下app文件
```php
mkdir /var/www/cgi-bin/gnocchi/
vim /var/www/cgi-bin/gnocchi/app
```

- app文件如下：
```python
#!/usr/bin/python
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or
# implied.
# See the License for the specific language governing permissions and
# limitations under the License.
if __name__ == '__main__':
    import sys
    from gnocchi.cli import api
    sys.exit(api.api())
else:
    from gnocchi.cli import api
    from gnocchi.rest import app
    application = app.load_app(api.prepare_service())
```

- 文件夹修改权限
```php
# chown -R gnocchi.gnocchi /var/www/cgi-bin/gnocchi
# chmod +777 /var/www/cgi-bin/gnocchi
```

- 重启httpd服务
```php
# systemctl restart httpd
```

#### 为Gnocchi的索引器创建数据库

- 使用数据库访问客户端以root用户身份连接到数据库服务器
```mysql
$ mysql -u root -p
CREATE DATABASE gnocchi;
GRANT ALL PRIVILEGES ON gnocchi.* TO 'gnocchi'@'localhost' IDENTIFIED BY '123456';
GRANT ALL PRIVILEGES ON gnocchi.* TO 'gnocchi'@'%' IDENTIFIED BY '123456';
```

- 编辑/etc/gnocchi/gnocchi.conf文件并添加Keystone选项
 - 在[api]中，配置gnocchi以使用keystone：
```php
[api]
auth_mode = keystone
```

 - 在[keystone_authtoken]部分中，配置keystone身份验证：
```php
[keystone_authtoken]
...
auth_type = password
auth_url = http://controller:5000/v3
project_domain_name = Default
user_domain_name = Default
project_name = service
username = gnocchi
password = 123456
interface = internalURL
region_name = RegionOne
[indexer]
url = mysql+pymysql://gnocchi:GNOCCHI_DBPASS@controller/gnocchi
```

 - 在[storage]部分中，配置位置以存储度量标准数据。在这种情况下，我们将它存储到本地文件系统。有关更耐用和高性能驱动程序的列表，
 请参阅Gnocchi文档：
```php
[storage]
# coordination_url is not required but specifying one will improve
# performance with better workload division across workers.
coordination_url = redis://controller:6379
file_basepath = /var/lib/gnocchi
driver = file
```

- 初始化Gnocchi：
```php
gnocchi-upgrade
```

#### 完成Gnocchi安装
启动Gnocchi服务并将其配置为在系统引导时启动

```php
# systemctl enable openstack-gnocchi-api.service openstack-gnocchi-metricd.service
# systemctl start openstack-gnocchi-api.service  openstack-gnocchi-metricd.service
```

#### 安装Ceilometer包
```php
# yum install openstack-ceilometer-notification openstack-ceilometer-central
```

- 编辑/etc/ceilometer/pipeline.yaml文件并完成以下部分，配置Gnocchi连接：
```php
publishers:
    # set address of Gnocchi
    # + filter out Gnocchi-related activity meters (Swift driver)
    # + set default archive policy
    - gnocchi://?filter_project=service&archive_policy=low
```

- 编辑/etc/ceilometer/ceilometer.conf文件并完成以下操作
 - 在该[DEFAULT]部分中，配置RabbitMQ消息队列访问：
```php
[DEFAULT]
...
transport_url  =  rabbit://openstack:123456@controller
```

 - 在该[service_credentials]部分中，配置服务凭据：
```php
[service_credentials]
...
auth_type = password
auth_url = http://controller:5000/v3
project_domain_id = default
user_domain_id = default
project_name = service
username = ceilometer
password = 123456
interface = internalURL
region_name = RegionOne
```

- 在Gnocchi创建Ceilometer资源。Gnocchi应该在这个阶段运行：
```php
# ceilometer-upgrade
```

#### 完成安装
启动Ceilometer服务并将其配置为在系统引导时启动：
```php
# systemctl enable openstack-ceilometer-notification.service \
  openstack-ceilometer-central.service
# systemctl start openstack-ceilometer-notification.service \
  openstack-ceilometer-central.service
```

#### glance neutron服务配置

**Image**

- 编辑/etc/glance/glance-api.conf文件并完成以下操作：
在[DEFAULT]，[oslo_messaging_notifications]部分中，配置通知和RabbitMQ消息代理访问：
```php
[DEFAULT]
transport_url = rabbit://openstack:123456@controller
[oslo_messaging_notifications]
driver = messagingv2
```

- 编辑/etc/glance/glance-registry.conf文件并完成以下操作：
在[DEFAULT]，[oslo_messaging_notifications]部分中，配置通知和RabbitMQ消息代理访问：
```php
[DEFAULT]
transport_url = rabbit://openstack:123456@controller
[oslo_messaging_notifications]
driver = messagingv2
```

- 完成安装
重新启动Image服务：
```php
# systemctl restart openstack-glance-api.service openstack-glance-registry.service
```

**Neutron**

配置网络服务以使用Ceilometer
- 编辑/etc/neutron/neutron.conf并完成以下操作：
在这些[oslo_messaging_notifications]部分中，启用通知：
```php
[oslo_messaging_notifications]
...
driver  =  messagingv2
```

完成安装
- 重启网络服务：
```php
# systemctl restart neutron-server.service
```

#### 配置环境变量及gnocchi文件夹权限
更改目录权限，否则gnocchi存储文件数据时报错，没有权限
```php
# chmod +777 /var/lib/gnocchi
```

/root/admin-openrc添加gnocchi相关环境变量
```php
# export OS_AUTH_TYPE=password
```

### 监控项列表

注意：使用ceilometer监控虚拟机的memory.usage，要求libvirt版本1.1.1+，qemu版本1.5+。并且需要镜像支持balloon，即镜像中安装有balloon的driver。（一般linux镜像都默认包含，但是windows镜像需要另行安装）
虚拟机的Cpu：使用时间（cpu）,使用率（cpu_util），分配vcpu个数（vcpus）
虚拟机的硬盘：分配总大小（disk.root.size），占用宿主机大小（disk.usage）
虚拟机的内存：总大小（memory），使用大小（memory.usage）
虚拟机的网络：网络流量（bandwidth），网络下ip个数（ip.floating）
虚拟机的镜像：镜像大小（image.download）
节点的cpu、硬盘、内存等使用信息可通过linux常用命令监视

#### nova实例监控项

Name | Type | Unit | Resource | Origin | Support | Note | 翻译
- | - | - | - | - | - | - | -
memory | Gauge | MB | instance ID | Notification | Libvirt, Hyper-V | Volume of RAM allocated to the instance | 分配给实例的RAM量
memory.usage | Gauge | MB | instance ID | Pollster | Libvirt, Hyper-V, vSphere, XenAPI | Volume of RAM used by the instance from the amount of its allocated memory | 实例从其分配的内存量中使用的RAM量
...

参考：https://docs.openstack.org/ceilometer/latest/admin/telemetry-measurements.html

### 详细监控方法介绍
#### 采集信息查询方法
此方案中采集信息以gnocchi+文件方式存储。获取采集信息有两种方式，分别为 gnocchi-api和gnocchi 命令。以下采用gnocchi命令做详细介绍：
注：数据存储分级：(资源项)resource->(资源指标条目)metric->(实际数据)measure

- 获取权限
```php
# . admin-openrc
```

- 获取监控资源列表
```php
# gnocchi resource list
```

- 获取实例监控资源对象信息列表
获取某一instance下所有监控条目
```php
[root@controller ~]# gnocchi resource show 8bb00e90-7e3f-5ef6-bee5-00e7c6bbc5fc
+-----------------------+-------------------------------------------------------------------+
| Field                 | Value                                                             |
+-----------------------+-------------------------------------------------------------------+
| created_by_project_id | e06fb4ce67a24aa687dffb9d8dcc6b1e                                  |
| created_by_user_id    | 4895e340408e42c383ea8d82e459b182                                  |
| creator               | 4895e340408e42c383ea8d82e459b182:e06fb4ce67a24aa687dffb9d8dcc6b1e |
| ended_at              | None                                                              |
| id                    | 8bb00e90-7e3f-5ef6-bee5-00e7c6bbc5fc                              |
| metrics               | disk.device.allocation: 427a316e-1ef9-4054-8253-604b80e8f003      |
|                       | disk.device.capacity: 6915cd9b-1346-46db-8e86-438fd1884b49        |
|                       | disk.device.latency: fcfbc2ae-eddf-4ff9-bda9-33b9a2ba9c3d         |
|                       | disk.device.read.bytes: 6df3416c-ad02-401e-84e4-5ac0b75febf2      |
|                       | disk.device.read.latency: 4014d3c6-51ea-4e24-b8e6-649ed6ee896d    |
|                       | disk.device.usage: 7cd08e21-a0e1-4fce-8dc8-0e8b96d711d4           |
|                       | disk.device.write.bytes: b29abf7c-a9c4-4afc-89a5-ef766739028b     |
|                       | disk.device.write.latency: f82d69ec-98d6-45e8-bc13-cd040f8cc2ef   |
| original_resource_id  | 86d618b5-aadc-4346-b620-7f44fc35c7ad-vda                          |
| project_id            | 40026f6973464ee9a19ad04f6221e213                                  |
| revision_end          | None                                                              |
| revision_start        | 2019-05-07T05:08:10.450142+00:00                                  |
| started_at            | 2019-05-07T05:08:10.450123+00:00                                  |
| type                  | instance_disk                                                     |
| user_id               | 5a4da861d8e84cbdabd98b9804bc1547                                  |
+-----------------------+-------------------------------------------------------------------+
```

- 获取实例监控条目详细信息
```php
# gnocchi measures show 897356e6-d0bb-43cd-9cbd-43ab2808384d
```
