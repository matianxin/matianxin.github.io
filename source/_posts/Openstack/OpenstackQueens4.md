---
title: Openstack Queens 环境搭建（四）Glance服务
tags: [openstack]
categories: Openstack
description: OpenStack项目是一个开源云计算平台，支持所有类型的云环境。该项目旨在实现简单，大规模的可扩展性和丰富的功能。OpenStack通过各种补充服务提供基础架构即服务（IaaS）解决方案。每项服务都提供了一个应用程序编程接口（API），以促进这种集成。本文涵盖了使用适用于具有足够Linux经验的OpenStack新用户的功能性示例体系结构，逐步部署主要OpenStack服务。
date: 2019/3/19 16:35:00
---
### Controller节点：
#### 创建glance数据库，授予权限：
```php
$ mysql -u root -p
MariaDB [(none)]> CREATE DATABASE glance;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY '123456';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY '123456';
MariaDB [(none)]> exit;
```

#### 创建glance用户：
```php
$ . admin-openrc

$ openstack user create --domain default --password-prompt glance
User Password: 123456
Repeat User Password: 123456
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 5b7e76213b4b4945b7c702be5b595c0e |
| name                | glance                           |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

$ openstack role add --project service --user glance admin
```

#### 创建glance服务实体：
```php
$ openstack service create --name glance --description "OpenStack Image" image
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Image                  |
| enabled     | True                             |
| id          | b9cfd97d134e4ec2bf19d78306e85a5a |
| name        | glance                           |
| type        | image                            |
+-------------+----------------------------------+
```

#### 创建API端点：
```php
$ openstack endpoint create --region RegionOne image public http://controller:9292
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | b9c90172de704ea4a867190ba44fc931 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | b9cfd97d134e4ec2bf19d78306e85a5a |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+
```

```php
$ openstack endpoint create --region RegionOne image internal http://controller:9292
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 074bde7662044e93830f4eca15d9c887 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | b9cfd97d134e4ec2bf19d78306e85a5a |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+
```

```php
$ openstack endpoint create --region RegionOne image admin http://controller:9292
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 17030061f9b84301ac515765706933b2 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | b9cfd97d134e4ec2bf19d78306e85a5a |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+
```

#### 安装和配置：
```php
# yum install openstack-glance

# vi /etc/glance/glance-api.conf
[database]
connection = mysql+pymysql://glance:123456@controller/glance
[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = 123456
[paste_deploy]
flavor = keystone


# vi /etc/glance/glance-registry.conf
[database]
connection = mysql+pymysql://glance:123456@controller/glance
[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:5000
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = 123456
[paste_deploy]
flavor = keystone

# su -s /bin/sh -c "glance-manage db_sync" glance
```

#### 完成安装
```php
# systemctl enable openstack-glance-api.service openstack-glance-registry.service
# systemctl start openstack-glance-api.service openstack-glance-registry.service
```

#### 验证操作
```php
$ . admin-openrc

$ wget http://download.cirros-cloud.net/0.3.5/cirros-0.3.5-x86_64-disk.img

$ openstack image create "cirros" \
  --file cirros-0.3.5-x86_64-disk.img \
  --disk-format qcow2 --container-format bare \
  --public
+------------------+------------------------------------------------------+
| Field            | Value                                                |
+------------------+------------------------------------------------------+
| checksum         | f8ab98ff5e73ebab884d80c9dc9c7290                     |
| container_format | bare                                                 |
| created_at       | 2018-09-13T00:55:04Z                                 |
| disk_format      | qcow2                                                |
| file             | /v2/images/ad7da2d4-cb83-4a41-836f-e58e47e899f5/file |
| id               | ad7da2d4-cb83-4a41-836f-e58e47e899f5                 |
| min_disk         | 0                                                    |
| min_ram          | 0                                                    |
| name             | cirros                                               |
| owner            | 4a5e42dd8cbf410f85a5f145039d69a6                     |
| protected        | False                                                |
| schema           | /v2/schemas/image                                    |
| size             | 13267968                                             |
| status           | active                                               |
| tags             |                                                      |
| updated_at       | 2018-09-13T00:55:04Z                                 |
| virtual_size     | None                                                 |
| visibility       | public                                               |
+------------------+------------------------------------------------------+
```

```php
$ openstack image list
+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| ad7da2d4-cb83-4a41-836f-e58e47e899f5 | cirros | active |
+--------------------------------------+--------+--------+
```
