---
title: Openstack命令行
tags: [openstack]
categories: Openstack
description: Openstack可供参考的常用命令列表
date: 2019/7/11 15:42:00
---

### Keystone

#### 列出所有的用户
```php
$ openstack user list

+----------------------------------+-----------+
| ID                               | Name      |
+----------------------------------+-----------+
| 2ac5fe07105a43518ae444a37640222b | demo      |
| 5bc73e84451b4e71906232bd3849422d | neutron   |
| 84b0b75df69f4413a1380c9efc249b2d | placement |
| 87bf7fb4671f4524a6fa64ad75856594 | admin     |
| 9aaa5aa26b414d35b37b851ccf749a55 | nova      |
| eee2818e703e47c5a434e5926c90e7fb | glance    |
+----------------------------------+-----------+
```

#### 列出认证服务目录
```php
$ openstack catalog list

+-----------+-----------+-----------------------------------------+
| Name      | Type      | Endpoints                               |
+-----------+-----------+-----------------------------------------+
| placement | placement | RegionOne                               |
|           |           |   internal: http://controller:8778      |
|           |           | RegionOne                               |
|           |           |   public: http://controller:8778        |
|           |           | RegionOne                               |
|           |           |   admin: http://controller:8778         |
|           |           |                                         |
| keystone  | identity  | RegionOne                               |
|           |           |   internal: http://controller:5000/v3/  |
|           |           | RegionOne                               |
|           |           |   public: http://controller:5000/v3/    |
|           |           | RegionOne                               |
|           |           |   admin: http://controller:5000/v3/     |
|           |           |                                         |
| neutron   | network   | RegionOne                               |
|           |           |   public: http://controller:9696        |
|           |           | RegionOne                               |
|           |           |   admin: http://controller:9696         |
|           |           | RegionOne                               |
|           |           |   internal: http://controller:9696      |
|           |           |                                         |
| glance    | image     | RegionOne                               |
|           |           |   admin: http://controller:9292         |
|           |           | RegionOne                               |
|           |           |   internal: http://controller:9292      |
|           |           | RegionOne                               |
|           |           |   public: http://controller:9292        |
|           |           |                                         |
| nova      | compute   | RegionOne                               |
|           |           |   internal: http://controller:8774/v2.1 |
|           |           | RegionOne                               |
|           |           |   admin: http://controller:8774/v2.1    |
|           |           | RegionOne                               |
|           |           |   public: http://controller:8774/v2.1   |
|           |           |                                         |
+-----------+-----------+-----------------------------------------+
```

### Glance

#### 列出您可以访问的镜像
```php
$ openstack image list

+--------------------------------------+---------------------+--------+
| ID                                   | Name                | Status |
+--------------------------------------+---------------------+--------+
| 70d455f6-1a00-48f5-9b21-5f8fca019014 | centos7-mini        | active |
| 1a7e4d60-3c26-41bb-be0c-34c467b2d3c3 | pstunnel            | active |
| b2464312-8acb-4ff9-ba20-75560e5577f4 | pstunnelA           | active |
| 705c7007-8a28-49e8-8df7-61bbfbf41bbd | pstunnelB           | active |
| ad02946e-9c1f-42f0-9a9e-235ae7012bc3 | router              | active |
| ae8c618d-3c6e-449c-9d2a-3f08bf84cc5e | router22            | active |
| b54361d4-bc24-46cb-94b2-b1f672b01c2a | sw-f-in-centos7_ext | active |
| 3a1adf75-c82e-4540-91e0-2167cb921612 | sw-f-in-centos7_int | active |
| ebfc7c40-4ac3-42ab-96ae-65e3c17af194 | sw-z-in-centos7_ext | active |
| 7355a66c-3f05-4ed5-9088-540faf71ebf8 | sw-z-in-centos7_int | active |
| 3b19c8ad-6012-406a-9e7f-e66416deeb1e | testforimage        | active |
| 436c5ab5-14e7-4eb8-95b9-e47d85e60e83 | win7                | active |
+--------------------------------------+---------------------+--------+
```

#### 查看一个指定的镜像
```php
$ openstack image show [IMAGE-ID/IMAGE-Name]

$ openstack image show 70d455f6-1a00-48f5-9b21-5f8fca019014
+------------------+------------------------------------------------------+
| Field            | Value                                                |
+------------------+------------------------------------------------------+
| checksum         | f3ab346b3ca2b88d1347c24adf0b234b                     |
| container_format | bare                                                 |
| created_at       | 2019-06-26T09:08:51Z                                 |
| disk_format      | qcow2                                                |
| file             | /v2/images/70d455f6-1a00-48f5-9b21-5f8fca019014/file |
| id               | 70d455f6-1a00-48f5-9b21-5f8fca019014                 |
| min_disk         | 0                                                    |
| min_ram          | 0                                                    |
| name             | centos7-mini                                         |
| owner            | 2e69bc10ab5f427bbbd6d40148d96309                     |
| protected        | False                                                |
| schema           | /v2/schemas/image                                    |
| size             | 3758882816                                           |
| status           | active                                               |
| tags             |                                                      |
| updated_at       | 2019-06-26T09:09:32Z                                 |
| virtual_size     | None                                                 |
| visibility       | public                                               |
+------------------+------------------------------------------------------+

```

#### 上传QCOW2镜像
```php
$ openstack image create "centos7-mini2" --file centos7-mini.qcow2 --disk-format qcow2 --container-format bare --public

+------------------+------------------------------------------------------+
| Field            | Value                                                |
+------------------+------------------------------------------------------+
| checksum         | f3ab346b3ca2b88d1347c24adf0b234b                     |
| container_format | bare                                                 |
| created_at       | 2019-07-11T07:56:51Z                                 |
| disk_format      | qcow2                                                |
| file             | /v2/images/4072bdfb-f727-4e3f-a1f5-c3467b12a15e/file |
| id               | 4072bdfb-f727-4e3f-a1f5-c3467b12a15e                 |
| min_disk         | 0                                                    |
| min_ram          | 0                                                    |
| name             | centos7-mini2                                        |
| owner            | 2e69bc10ab5f427bbbd6d40148d96309                     |
| protected        | False                                                |
| schema           | /v2/schemas/image                                    |
| size             | 3758882816                                           |
| status           | active                                               |
| tags             |                                                      |
| updated_at       | 2019-07-11T07:57:54Z                                 |
| virtual_size     | None                                                 |
| visibility       | public                                               |
+------------------+------------------------------------------------------+
```

#### 删除指定的镜像
```php
$ openstack image delete [IMAGE-ID/IMAGE-Name]

$ openstack image delete 4072bdfb-f727-4e3f-a1f5-c3467b12a15e
```

### Nova

#### 列出实例
```php
$ openstack server list

+--------------------------------------+-------------------------+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------+---------------------+
| ID                                   | Name                    | Status  | Networks                                                                                                                                                                         | Image               | Flavor              |
+--------------------------------------+-------------------------+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------+---------------------+
| 2ec1ace1-ef83-4d30-837e-f7f2df344a6c | n-UDYIQCS6              | SHUTOFF | network_5261=192.168.1.3                                                                                                                                                         | win7                | win7                |
| 88a7d91d-3c29-461e-8d07-a0d272697a61 | n-L9QGOHRE              | SHUTOFF | network_12177=192.168.1.19                                                                                                                                                       | win7                | win7                |
| 274543b3-7890-4446-a5d0-1c6acfb9e1f4 | vm_n-4ejn7bk2c9o@28     | ACTIVE  | network_10888=192.168.1.18                                                                                                                                                       | win7                | win7                |
| c2cb7d6c-4f3e-4f32-bbd5-19639d415481 | sw_n-pf6p66vh4f_ext@33  | SHUTOFF | network_10360=192.168.1.3; n-XH29F3CG-net=8.8.8.9; network_21065=192.168.1.6; network_16174=192.168.1.5                                                                          | sw-z-in-centos7_ext | sw-z-in-centos7_ext |
| 5e009b71-4a8b-478a-9f17-349153a26ad4 | n-UE0N48YS              | SHUTOFF | network_11645=192.168.1.4; network_14835=192.168.1.13; network_20101=192.168.1.9; internal=20.0.0.23, 192.100.200.225; network_18377=192.168.1.3; network_8075=192.168.1.10      | pstunnel            | pstunnel            |
| 374aedbe-d7c3-4f89-a66d-020401a257dc | n-MF9XYLTQ_int          | SHUTOFF | network_23143=192.168.1.7; network_14693=192.168.1.7; network_13651=192.168.1.11                                                                                                 | sw-z-in-centos7_int | sw-z-in-centos7_int |
| acd8efcb-9350-4e19-a105-a1bcfb529697 | rt_n-une1ph6cts@20      | SHUTOFF | Rnet_n-une1ph6cts_0=192.168.3.1; Rnet_n-une1ph6cts_1=192.168.2.1                                                                                                                 | router              | router              |
| 1efe6a9a-d005-4b32-926f-bb94fb702a05 | vm_n-vp0eo8lm0c@20      | SHUTOFF | network_14083=192.168.1.10                                                                                                                                                       |                     |                     |
+--------------------------------------+-------------------------+---------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+---------------------+---------------------+

```

#### 显示实例详细信息
```php
$ openstack server show [SERVER-ID/SERVER-Name]

$ openstack server show 2ec1ace1-ef83-4d30-837e-f7f2df344a6c
+-------------------------------------+----------------------------------------------------------+
| Field                               | Value                                                    |
+-------------------------------------+----------------------------------------------------------+
| OS-DCF:diskConfig                   | MANUAL                                                   |
| OS-EXT-AZ:availability_zone         | nova                                                     |
| OS-EXT-SRV-ATTR:host                | compute                                                  |
| OS-EXT-SRV-ATTR:hypervisor_hostname | compute                                                  |
| OS-EXT-SRV-ATTR:instance_name       | instance-00000294                                        |
| OS-EXT-STS:power_state              | Shutdown                                                 |
| OS-EXT-STS:task_state               | None                                                     |
| OS-EXT-STS:vm_state                 | stopped                                                  |
| OS-SRV-USG:launched_at              | 2019-07-11T06:30:19.000000                               |
| OS-SRV-USG:terminated_at            | None                                                     |
| accessIPv4                          |                                                          |
| accessIPv6                          |                                                          |
| addresses                           | network_5261=192.168.1.3                                 |
| config_drive                        |                                                          |
| created                             | 2019-07-11T06:30:12Z                                     |
| flavor                              | win7 (fd8297ef-d7ac-4b4f-8c1e-1c90fcce488c)              |
| hostId                              | 37f71a5ebb11ac3ed0f0496a313bb9246829f1da1839c8fd3dc4f98b |
| id                                  | 2ec1ace1-ef83-4d30-837e-f7f2df344a6c                     |
| image                               | win7 (436c5ab5-14e7-4eb8-95b9-e47d85e60e83)              |
| key_name                            | None                                                     |
| name                                | n-UDYIQCS6                                               |
| project_id                          | 2e69bc10ab5f427bbbd6d40148d96309                         |
| properties                          |                                                          |
| security_groups                     | name='default'                                           |
| status                              | SHUTOFF                                                  |
| updated                             | 2019-07-11T06:31:30Z                                     |
| user_id                             | 87bf7fb4671f4524a6fa64ad75856594                         |
| volumes_attached                    |                                                          |
+-------------------------------------+----------------------------------------------------------+

```

### Neutron

#### 列出所有网络
```php
$ openstack network list

+--------------------------------------+----------------------+--------------------------------------+
| ID                                   | Name                 | Subnets                              |
+--------------------------------------+----------------------+--------------------------------------+
| 01e6c390-1d5e-4140-9ea3-98754bd0c58f | network_29287        | 8ed09dee-0e16-4514-8821-9a8e1f77c43a |
| 030f303e-e3d3-46e1-a85a-5ea6cea4d3ba | n-69PSABRW-net       | e2ea53d0-1b29-43d3-910b-3296d96d004f |
+--------------------------------------+----------------------+--------------------------------------+
```

#### 查看网络
```php
$ openstack network show NETWORK-ID

$ openstack network show 01e6c390-1d5e-4140-9ea3-98754bd0c58f
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        | nova                                 |
| created_at                | 2019-06-19T02:16:06Z                 |
| description               |                                      |
| dns_domain                | None                                 |
| id                        | 01e6c390-1d5e-4140-9ea3-98754bd0c58f |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| is_default                | None                                 |
| is_vlan_transparent       | None                                 |
| mtu                       | 1500                                 |
| name                      | network_29287                        |
| port_security_enabled     | True                                 |
| project_id                | 2e69bc10ab5f427bbbd6d40148d96309     |
| provider:network_type     | vxlan                                |
| provider:physical_network | None                                 |
| provider:segmentation_id  | 29287                                |
| qos_policy_id             | None                                 |
| revision_number           | 3                                    |
| router:external           | Internal                             |
| segments                  | None                                 |
| shared                    | True                                 |
| status                    | ACTIVE                               |
| subnets                   | 8ed09dee-0e16-4514-8821-9a8e1f77c43a |
| tags                      |                                      |
| updated_at                | 2019-06-19T02:16:07Z                 |
+---------------------------+--------------------------------------+
```

#### 列出所有子网
```php
$ openstack subnet list

+--------------------------------------+-----------------------+--------------------------------------+------------------+
| ID                                   | Name                  | Network                              | Subnet           |
+--------------------------------------+-----------------------+--------------------------------------+------------------+
| 00414731-ab21-4ee8-8bf2-86d960cc4339 | network_sub_18721     | 84d6ee39-77b3-4a0b-80f8-67922b63079a | 192.168.1.0/24   |
| 014cb2e2-bdbe-4849-aec7-5fe0581e3b67 | network_sub_20494     | 6eb235c5-798c-49de-8fd3-08aa64f9abaa | 192.168.1.0/24   |
+--------------------------------------+-----------------------+--------------------------------------+------------------+
```

#### 查看子网
```php
$ openstack subnet show SUBNET-ID

$ openstack subnet show 00414731-ab21-4ee8-8bf2-86d960cc4339
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| allocation_pools  | 192.168.1.2-192.168.1.254            |
| cidr              | 192.168.1.0/24                       |
| created_at        | 2019-07-11T02:09:49Z                 |
| description       |                                      |
| dns_nameservers   |                                      |
| enable_dhcp       | True                                 |
| gateway_ip        | 192.168.1.1                          |
| host_routes       |                                      |
| id                | 00414731-ab21-4ee8-8bf2-86d960cc4339 |
| ip_version        | 4                                    |
| ipv6_address_mode | None                                 |
| ipv6_ra_mode      | None                                 |
| name              | network_sub_18721                    |
| network_id        | 84d6ee39-77b3-4a0b-80f8-67922b63079a |
| project_id        | 2e69bc10ab5f427bbbd6d40148d96309     |
| revision_number   | 0                                    |
| segment_id        | None                                 |
| service_types     |                                      |
| subnetpool_id     | None                                 |
| tags              |                                      |
| updated_at        | 2019-07-11T02:09:49Z                 |
+-------------------+--------------------------------------+
```

#### 列出所有port
```php
$ openstack port list

+--------------------------------------+------+-------------------+--------------------------------------------------------------------------------+--------+
| ID                                   | Name | MAC Address       | Fixed IP Addresses                                                             | Status |
+--------------------------------------+------+-------------------+--------------------------------------------------------------------------------+--------+
| 01f86921-016b-4df8-b526-ec81418b5df5 |      | fa:16:3e:b9:c8:a8 | ip_address='20.0.0.2', subnet_id='c00a64e4-8921-427c-aa71-74f7a2c55954'        | ACTIVE |
| 0280ba23-7a51-483c-a7a4-2351f277f739 |      | fa:16:3e:ab:63:69 | ip_address='192.168.2.1', subnet_id='c4b84a4b-ce15-49e8-ae6b-72566224682c'     | DOWN   |
+--------------------------------------+------+-------------------+--------------------------------------------------------------------------------+--------+
```

#### 查看port
```php
$ openstack port show PORT-ID

$ openstack port show 01f86921-016b-4df8-b526-ec81418b5df5
+-----------------------+-------------------------------------------------------------------------------+
| Field                 | Value                                                                         |
+-----------------------+-------------------------------------------------------------------------------+
| admin_state_up        | UP                                                                            |
| allowed_address_pairs |                                                                               |
| binding_host_id       | controller                                                                    |
| binding_profile       |                                                                               |
| binding_vif_details   | datapath_type='system', ovs_hybrid_plug='True', port_filter='True'            |
| binding_vif_type      | ovs                                                                           |
| binding_vnic_type     | normal                                                                        |
| created_at            | 2019-06-19T01:41:33Z                                                          |
| data_plane_status     | None                                                                          |
| description           |                                                                               |
| device_id             | dhcpd3377d3c-a0d1-5d71-9947-f17125c357bb-361f196b-154a-4e51-bc34-7162d043be1b |
| device_owner          | network:dhcp                                                                  |
| dns_assignment        | None                                                                          |
| dns_name              | None                                                                          |
| extra_dhcp_opts       |                                                                               |
| fixed_ips             | ip_address='20.0.0.2', subnet_id='c00a64e4-8921-427c-aa71-74f7a2c55954'       |
| id                    | 01f86921-016b-4df8-b526-ec81418b5df5                                          |
| ip_address            | None                                                                          |
| mac_address           | fa:16:3e:b9:c8:a8                                                             |
| name                  |                                                                               |
| network_id            | 361f196b-154a-4e51-bc34-7162d043be1b                                          |
| option_name           | None                                                                          |
| option_value          | None                                                                          |
| port_security_enabled | False                                                                         |
| project_id            | 2e69bc10ab5f427bbbd6d40148d96309                                              |
| qos_policy_id         | None                                                                          |
| revision_number       | 6                                                                             |
| security_group_ids    |                                                                               |
| status                | ACTIVE                                                                        |
| subnet_id             | None                                                                          |
| tags                  |                                                                               |
| trunk_details         | None                                                                          |
| updated_at            | 2019-06-19T01:41:35Z                                                          |
+-----------------------+-------------------------------------------------------------------------------+

```


摘自：Openstack官方文档 https://docs.openstack.org/zh_CN/user-guide/cli-cheat-sheet.html
