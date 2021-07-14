---
title: Openstack Queens 环境搭建（五）Nova服务
tags: [openstack]
categories: Openstack
description: OpenStack项目是一个开源云计算平台，支持所有类型的云环境。该项目旨在实现简单，大规模的可扩展性和丰富的功能。OpenStack通过各种补充服务提供基础架构即服务（IaaS）解决方案。每项服务都提供了一个应用程序编程接口（API），以促进这种集成。本文涵盖了使用适用于具有足够Linux经验的OpenStack新用户的功能性示例体系结构，逐步部署主要OpenStack服务。
date: 2019/3/19 16:40:00
---
### Controller节点：
#### 创建 nova_api, nova,和 nova_cell0 的数据库，授予权限：
```php
$ mysql -u root -p

MariaDB [(none)]>CREATE DATABASE nova_api;
MariaDB [(none)]> CREATE DATABASE nova;
MariaDB [(none)]> CREATE DATABASE nova_cell0;

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY '123456';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY '123456';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY '123456';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY '123456';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' IDENTIFIED BY '123456';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' IDENTIFIED BY '123456';

MariaDB [(none)]> exit;
```

#### 创建nova用户：
```php
$ . admin-openrc

$ openstack user create --domain default --password-prompt nova
User Password: 123456
Repeat User Password: 123456
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 81f1d5dfad5a42bb806d197ceb9881ce |
| name                | nova                             |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

$ openstack role add --project service --user nova admin
```

#### 创建nova服务实体：
```php
$ openstack service create --name nova --description "OpenStack Compute" compute
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Compute                |
| enabled     | True                             |
| id          | 3e011d345e4442fe8a232ab5ab1f8323 |
| name        | nova                             |
| type        | compute                          |
+-------------+----------------------------------+
```

#### 创建Compute API服务端点：
```php
$ openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 343b6a8fc9564623aca0097b2383650d |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 3e011d345e4442fe8a232ab5ab1f8323 |
| service_name | nova                             |
| service_type | compute                          |
| url          | http://controller:8774/v2.1      |
+--------------+----------------------------------+
```

```php
$ openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 3458cf55ac8b44d58c949fe88bf9afe3 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 3e011d345e4442fe8a232ab5ab1f8323 |
| service_name | nova                             |
| service_type | compute                          |
| url          | http://controller:8774/v2.1      |
+--------------+----------------------------------+
```

```php
$ openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 9f9115389c2a49a2874761b92c849bb0 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 3e011d345e4442fe8a232ab5ab1f8323 |
| service_name | nova                             |
| service_type | compute                          |
| url          | http://controller:8774/v2.1      |
+--------------+----------------------------------+
```

#### 创建Placement服务相关：
```php
$ openstack user create --domain default --password-prompt placement
User Password: 123456
Repeat User Password: 123456
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 74870bc86a7c4108869c620099bffc30 |
| name                | placement                        |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

$ openstack role add --project service --user placement admin
```

```php
$ openstack service create --name placement --description "Placement API" placement
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Placement API                    |
| enabled     | True                             |
| id          | bbd270a97c3a499fb73765120094e9da |
| name        | placement                        |
| type        | placement                        |
+-------------+----------------------------------+
```

```php
$ openstack endpoint create --region RegionOne placement public http://controller:8778
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | d79b3b62302a4055924762ac676fc9b4 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | bbd270a97c3a499fb73765120094e9da |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+
```

```php
$ openstack endpoint create --region RegionOne placement internal http://controller:8778
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 5424919fbee34a7a92946c607706b38a |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | bbd270a97c3a499fb73765120094e9da |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+
```

```php
$ openstack endpoint create --region RegionOne placement admin http://controller:8778
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | d9d5626cdb5442ac91dff8c1588f4726 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | bbd270a97c3a499fb73765120094e9da |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+
```

#### 安装和配置：
```php
# yum install openstack-nova-api openstack-nova-conductor openstack-nova-console openstack-nova-novncproxy openstack-nova-scheduler openstack-nova-placement-api

# vi /etc/nova/nova.conf
[DEFAULT]
my_ip=192.100.10.160
use_neutron=true
firewall_driver=nova.virt.firewall.NoopFirewallDriver
enabled_apis=osapi_compute,metadata
transport_url=rabbit://openstack:123456@controller
[api]
auth_strategy=keystone
[api_database]
connection = mysql+pymysql://nova:123456@controller/nova_api
[database]
connection = mysql+pymysql://nova:123456@controller/nova
[glance]
api_servers = http://controller:9292
[keystone_authtoken]
auth_url = http://controller:5000/v3
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = 123456
[libvirt]
#virt_type=kvm
[neutron]
url = http://controller:9696
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = 123456
service_metadata_proxy = true
metadata_proxy_shared_secret = 123456
[oslo_concurrency]
lock_path=/var/lib/nova/tmp
[placement]
os_region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller:5000/v3
username = placement
password = 123456
[vnc]
enabled=true
server_listen=$my_ip
server_proxyclient_address=$my_ip
#novncproxy_base_url=http://127.0.0.1:6080/vnc_auto.html
```

```php
# vi /etc/httpd/conf.d/00-nova-placement-api.conf            在最下方加入
<Directory /usr/bin>
   <IfVersion >= 2.4>
      Require all granted
   </IfVersion>
   <IfVersion < 2.4>
      Order allow,deny
      Allow from all
   </IfVersion>
</Directory>
```

#### 完成安装：
```php
# systemctl restart httpd

# su -s /bin/sh -c "nova-manage api_db sync" nova
# su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
# su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
# su -s /bin/sh -c "nova-manage db sync" nova

# nova-manage cell_v2 list_cells
+-------+--------------------------------------+------------------------------------+-------------------------------------------------+
|  名称 |                 UUID                 |           Transport URL            |                    数据库连接                   |
+-------+--------------------------------------+------------------------------------+-------------------------------------------------+
| cell0 | 00000000-0000-0000-0000-000000000000 |               none:/               | mysql+pymysql://nova:****@controller/nova_cell0 |
| cell1 | c795b2eb-4814-4fe7-b9ff-090a1b1b2be5 | rabbit://openstack:****@controller |    mysql+pymysql://nova:****@controller/nova    |
+-------+--------------------------------------+------------------------------------+-------------------------------------------------+
```

```php
# systemctl enable openstack-nova-api.service openstack-nova-consoleauth.service openstack-nova-scheduler.service openstack-nova-conductor.service openstack-nova-novncproxy.service
# systemctl start openstack-nova-api.service openstack-nova-consoleauth.service openstack-nova-scheduler.service openstack-nova-conductor.service openstack-nova-novncproxy.service
```

### Compute节点：
#### 安装和配置：
```php
# yum install openstack-nova-compute

# vi /etc/nova/nova.conf
[DEFAULT]
my_ip = 192.100.10.161
enabled_apis = osapi_compute,metadata
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
transport_url = rabbit://openstack:123456@controller
[api]
auth_strategy = keystone
[vnc]
enabled = True
server_listen = 0.0.0.0
server_proxyclient_address = $my_ip
novncproxy_base_url = http://controller:6080/vnc_auto.html
[glance]
api_servers = http://controller:9292
[oslo_concurrency]
lock_path = /var/lib/nova/tmp
[placement]
os_region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://controller:5000/v3
username = placement
password = 123456
[keystone_authtoken]
auth_url = http://controller:5000/v3
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = 123456
```

#### 完成安装
```php
# systemctl enable libvirtd.service openstack-nova-compute.service
# systemctl start libvirtd.service openstack-nova-compute.service
```

### Controller节点：
#### 将计算节点添加到cell数据库：
```php
$ . admin-openrc

$ openstack compute service list --service nova-compute
+----+--------------+-----------------------+------+---------+-------+----------------------------+
| ID | Binary       | Host                  | Zone | Status  | State | Updated At                 |
+----+--------------+-----------------------+------+---------+-------+----------------------------+
|  9 | nova-compute | localhost.localdomain | nova | enabled | up    | 2018-09-13T02:59:06.000000 |
+----+--------------+-----------------------+------+---------+-------+----------------------------+
```

#### 发现计算主机：
```php
# su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova

/usr/lib/python2.7/site-packages/oslo_db/sqlalchemy/enginefacade.py:332: NotSupportedWarning: Configuration option(s) ['use_tpool'] not supported
  exception.NotSupportedWarning
Found 2 cell mappings.
Skipping cell0 since it does not contain hosts.
Getting computes from cell 'cell1': c795b2eb-4814-4fe7-b9ff-090a1b1b2be5
Checking host mapping for compute host 'localhost.localdomain': 58be78ad-5220-4869-ab31-33c9674ecfd1
Creating host mapping for compute host 'localhost.localdomain': 58be78ad-5220-4869-ab31-33c9674ecfd1
Found 1 unmapped computes in cell: c795b2eb-4814-4fe7-b9ff-090a1b1b2be5

注意：添加新计算节点时，必须在控制器节点上运行nova-manage cell_v2 discover_hosts以注册这些新计算节点。
或者，您可以在 /etc/nova/nova.conf 中设置适当的间隔：
[scheduler]
discover_hosts_in_cells_interval = 300
```

#### 验证：
```php
$ . admin-openrc

$ openstack compute service list
+----+------------------+-----------------------+----------+---------+-------+----------------------------+
| ID | Binary           | Host                  | Zone     | Status  | State | Updated At                 |
+----+------------------+-----------------------+----------+---------+-------+----------------------------+
|  1 | nova-conductor   | controller            | internal | enabled | up    | 2018-09-13T03:00:28.000000 |
|  3 | nova-consoleauth | controller            | internal | enabled | up    | 2018-09-13T03:00:29.000000 |
|  4 | nova-scheduler   | controller            | internal | enabled | up    | 2018-09-13T03:00:29.000000 |
|  9 | nova-compute     | localhost.localdomain | nova     | enabled | up    | 2018-09-13T03:00:26.000000 |
+----+------------------+-----------------------+----------+---------+-------+----------------------------+
```

```php
$ openstack catalog list
+-----------+-----------+-----------------------------------------+
| Name      | Type      | Endpoints                               |
+-----------+-----------+-----------------------------------------+
| keystone  | identity  | RegionOne                               |
|           |           |   public: http://controller:5000/v3/    |
|           |           | RegionOne                               |
|           |           |   internal: http://controller:5000/v3/  |
|           |           | RegionOne                               |
|           |           |   admin: http://controller:5000/v3/     |
|           |           |                                         |
| nova      | compute   | RegionOne                               |
|           |           |   public: http://controller:8774/v2.1   |
|           |           | RegionOne                               |
|           |           |   internal: http://controller:8774/v2.1 |
|           |           | RegionOne                               |
|           |           |   admin: http://controller:8774/v2.1    |
|           |           |                                         |
| glance    | image     | RegionOne                               |
|           |           |   internal: http://controller:9292      |
|           |           | RegionOne                               |
|           |           |   admin: http://controller:9292         |
|           |           | RegionOne                               |
|           |           |   public: http://controller:9292        |
|           |           |                                         |
| placement | placement | RegionOne                               |
|           |           |   internal: http://controller:8778      |
|           |           | RegionOne                               |
|           |           |   public: http://controller:8778        |
|           |           | RegionOne                               |
|           |           |   admin: http://controller:8778         |
|           |           |                                         |
+-----------+-----------+-----------------------------------------+
```

```php
$ openstack image list
+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| ad7da2d4-cb83-4a41-836f-e58e47e899f5 | cirros | active |
+--------------------------------------+--------+--------+
```

```php
# nova-status upgrade check
/usr/lib/python2.7/site-packages/oslo_db/sqlalchemy/enginefacade.py:332: NotSupportedWarning: Configuration option(s) ['use_tpool'] not supported
  exception.NotSupportedWarning
Option "os_region_name" from group "placement" is deprecated. Use option "region-name" from group "placement".
+-------------------------------+
| 升级检查结果                  |
+-------------------------------+
| 检查: Cells v2                |
| 结果: 成功                    |
| 详情: None                    |
+-------------------------------+
| 检查: Placement API           |
| 结果: 成功                    |
| 详情: None                    |
+-------------------------------+
| 检查: Resource Providers      |
| 结果: 成功                    |
| 详情: None                    |
+-------------------------------+
| 检查: Ironic Flavor Migration |
| 结果: 成功                    |
| 详情: None                    |
+-------------------------------+
| 检查: API Service Version     |
| 结果: 成功                    |
| 详情: None                    |
+-------------------------------+
```
