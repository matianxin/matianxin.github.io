---
title: Openstack Queens 环境搭建（六）Neutron服务
tags: [openstack]
categories: Openstack
description: OpenStack项目是一个开源云计算平台，支持所有类型的云环境。该项目旨在实现简单，大规模的可扩展性和丰富的功能。OpenStack通过各种补充服务提供基础架构即服务（IaaS）解决方案。每项服务都提供了一个应用程序编程接口（API），以促进这种集成。本文涵盖了使用适用于具有足够Linux经验的OpenStack新用户的功能性示例体系结构，逐步部署主要OpenStack服务。
date: 2019/3/19 16:45:00
---
### Controller节点：
Neutron服务安装网络类型：vxlan
Layer2 二层插件采用：openvswitch

#### 创建neutron数据库，授予权限：
```php
$ mysql -u root -p

MariaDB [(none)] CREATE DATABASE neutron;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY '123456';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY '123456';
MariaDB [(none)]> exit;
```

#### 创建neutron用户：
```php
$ . admin-openrc

$ openstack user create --domain default --password-prompt neutron
User Password: 123456
Repeat User Password: 123456
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 463fd14bf95b4cc49c0378623de56ffa |
| name                | neutron                          |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+

$ openstack role add --project service --user neutron admin
```

#### 创建neutron服务实体：
```php
$ openstack service create --name neutron --description "OpenStack Networking" network
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Networking             |
| enabled     | True                             |
| id          | e10e48790ede425ea81e1a62250f124a |
| name        | neutron                          |
| type        | network                          |
+-------------+----------------------------------+
```

#### 创建网络服务API端点：
```php
$ openstack endpoint create --region RegionOne network public http://controller:9696
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | f688ed8f1bf340d78794b600fa512145 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | e10e48790ede425ea81e1a62250f124a |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+
```

```php
$ openstack endpoint create --region RegionOne network internal http://controller:9696
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 571a008230c54cf8bcb1e38a75787c3f |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | e10e48790ede425ea81e1a62250f124a |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+
```

```php
$ openstack endpoint create --region RegionOne network admin http://controller:9696
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | a8d654c1c878423789aab3fa7cf634cb |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | e10e48790ede425ea81e1a62250f124a |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+
```

#### 安装及配置：
```php
# yum install openstack-neutron openstack-neutron-ml2 openstack-neutron-openvswitch ebtables

# vi /etc/neutron/neutron.conf
[DEFAULT]
auth_strategy = keystone
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = True
transport_url = rabbit://openstack:123456@controller
notify_nova_on_port_status_changes = true
notify_nova_on_port_data_changes = true
[database]
connection = mysql+pymysql://neutron:123456@controller/neutron
[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = 123456
[nova]
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = 123456
[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
```

```php
# vi /etc/neutron/plugins/ml2/ml2_conf.ini
[ml2]
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
mechanism_drivers = openvswitch,l2population
extension_drivers = port_security
[ml2_type_flat]
flat_networks = provider
[ml2_type_vlan]
#network_vlan_ranges = provider:1:1000
[ml2_type_vxlan]
vni_ranges = 1:1000
[securitygroup]
enable_ipset = true
```

```php
# vi /etc/sysctl.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
```

```php
# vi /etc/neutron/l3_agent.ini
[DEFAULT]
interface_driver = openvswitch
external_network_bridge =
```

```php
# vi /etc/neutron/dhcp_agent.ini
[DEFAULT]
interface_driver = openvswitch
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = true
```

```php
# vi /etc/neutron/metadata_agent.ini
[DEFAULT]
nova_metadata_host = controller
metadata_proxy_shared_secret = 123456
```

```php
# vi /etc/neutron/plugins/ml2/openvswitch_agent.ini
[agent]
tunnel_types = vxlan
l2_population = True
[ovs]
bridge_mappings = provider:br-provider
local_ip = 10.0.0.11
[securitygroup]
firewall_driver = iptables_hybrid
```


#### 完成安装：
```php
# ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
# su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron

# systemctl restart openstack-nova-api.service
# systemctl start neutron-server.service
# systemctl start neutron-openvswitch-agent.service

# ovs-vsctl add-br br-provider
# ifconfig eth0 0.0.0.0
# ifconfig br-provider 192.100.10.160/24
# route add default gw 192.100.10.1

# systemctl restart neutron-server.service
# systemctl restart neutron-openvswitch-agent.service
# systemctl enable neutron-server.service neutron-openvswitch-agent.service neutron-dhcp-agent.service neutron-metadata-agent.service
# systemctl start neutron-dhcp-agent.service neutron-metadata-agent.service
# systemctl enable neutron-l3-agent.service
# systemctl start neutron-l3-agent.service
```

### Compute节点：
#### 安装及配置：
```php# yum install openstack-neutron-openvswitch ebtables ipset

# vi /etc/neutron/neutron.conf
[DEFAULT]
transport_url = rabbit://openstack:123456@controller
auth_strategy = keystone
[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = 123456
[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
```

```php
# vi /etc/neutron/plugins/ml2/openvswitch_agent.ini
[ovs]
local_ip = 10.0.0.21
[agent]
tunnel_types = vxlan
l2_population = True

# systemctl restart neutron-openvswitch-agent.service
```

```php
# vi /etc/nova/nova.conf
...
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
```

#### 完成安装：
```php
# systemctl restart openstack-nova-compute.service
# systemctl enable neutron-openvswitch-agent.service
# systemctl start neutron-openvswitch-agent.service
```

### Controller节点：
#### 验证操作：
```php
$ . admin-openrc

$ openstack extension list --network
+----------------------------------------------------------------------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------+
| Name                                                                                         | Alias                     | Description                                                                                                                                              |
+----------------------------------------------------------------------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------+
| Default Subnetpools                                                                          | default-subnetpools       | Provides ability to mark and use a subnetpool as the default.                                                                                            |
| Availability Zone                                                                            | availability_zone         | The availability zone extension.                                                                                                                         |
| Network Availability Zone                                                                    | network_availability_zone | Availability zone support for network.                                                                                                                   |
| Auto Allocated Topology Services                                                             | auto-allocated-topology   | Auto Allocated Topology Services.                                                                                                                        |
| Neutron L3 Configurable external gateway mode                                                | ext-gw-mode               | Extension of the router abstraction for specifying whether SNAT should occur on the external gateway                                                     |
| Port Binding                                                                                 | binding                   | Expose port bindings of a virtual port to external application                                                                                           |
| agent                                                                                        | agent                     | The agent management extension.                                                                                                                          |
| Subnet Allocation                                                                            | subnet_allocation         | Enables allocation of subnets from a subnet pool                                                                                                         |
| L3 Agent Scheduler                                                                           | l3_agent_scheduler        | Schedule routers among l3 agents                                                                                                                         |
| Tag support                                                                                  | tag                       | Enables to set tag on resources.                                                                                                                         |
| Neutron external network                                                                     | external-net              | Adds external network attribute to network resource.                                                                                                     |
| Tag support for resources with standard attribute: trunk, policy, security_group, floatingip | standard-attr-tag         | Enables to set tag on resources with standard attribute.                                                                                                 |
| Neutron Service Flavors                                                                      | flavors                   | Flavor specification for Neutron advanced services.                                                                                                      |
| Network MTU                                                                                  | net-mtu                   | Provides MTU attribute for a network resource.                                                                                                           |
| Network IP Availability                                                                      | network-ip-availability   | Provides IP availability data for each network and subnet.                                                                                               |
| Quota management support                                                                     | quotas                    | Expose functions for quotas management per tenant                                                                                                        |
| If-Match constraints based on revision_number                                                | revision-if-match         | Extension indicating that If-Match based on revision_number is supported.                                                                                |
| HA Router extension                                                                          | l3-ha                     | Adds HA capability to routers.                                                                                                                           |
| Provider Network                                                                             | provider                  | Expose mapping of virtual networks to physical networks                                                                                                  |
| Multi Provider Network                                                                       | multi-provider            | Expose mapping of virtual networks to multiple physical networks                                                                                         |
| Quota details management support                                                             | quota_details             | Expose functions for quotas usage statistics per project                                                                                                 |
| Address scope                                                                                | address-scope             | Address scopes extension.                                                                                                                                |
| Neutron Extra Route                                                                          | extraroute                | Extra routes configuration for L3 router                                                                                                                 |
| Network MTU (writable)                                                                       | net-mtu-writable          | Provides a writable MTU attribute for a network resource.                                                                                                |
| Subnet service types                                                                         | subnet-service-types      | Provides ability to set the subnet service_types field                                                                                                   |
| Resource timestamps                                                                          | standard-attr-timestamp   | Adds created_at and updated_at fields to all Neutron resources that have Neutron standard attributes.                                                    |
| Neutron 服务类型管理                                                                         | service-type              | 用于为 Neutron 高级服务检索服务提供程序的 API                                                                                                            |
| Router Flavor Extension                                                                      | l3-flavors                | Flavor support for routers.                                                                                                                              |
| Port Security                                                                                | port-security             | Provides port security                                                                                                                                   |
| Neutron Extra DHCP options                                                                   | extra_dhcp_opt            | Extra options configuration for DHCP. For example PXE boot options to DHCP clients can be specified (e.g. tftp-server, server-ip-address, bootfile-name) |
| Resource revision numbers                                                                    | standard-attr-revisions   | This extension will display the revision number of neutron resources.                                                                                    |
| Pagination support                                                                           | pagination                | Extension that indicates that pagination is enabled.                                                                                                     |
| Sorting support                                                                              | sorting                   | Extension that indicates that sorting is enabled.                                                                                                        |
| security-group                                                                               | security-group            | The security groups extension.                                                                                                                           |
| DHCP Agent Scheduler                                                                         | dhcp_agent_scheduler      | Schedule networks among dhcp agents                                                                                                                      |
| Router Availability Zone                                                                     | router_availability_zone  | Availability zone support for router.                                                                                                                    |
| RBAC Policies                                                                                | rbac-policies             | Allows creation and modification of policies that control tenant access to resources.                                                                    |
| Tag support for resources: subnet, subnetpool, port, router                                  | tag-ext                   | Extends tag support to more L2 and L3 resources.                                                                                                         |
| standard-attr-description                                                                    | standard-attr-description | Extension to add descriptions to standard attributes                                                                                                     |
| IP address substring filtering                                                               | ip-substring-filtering    | Provides IP address substring filtering when listing ports                                                                                               |
| Neutron L3 Router                                                                            | router                    | Router abstraction for basic L3 forwarding between L2 Neutron networks and access to external networks via a NAT gateway.                                |
| Allowed Address Pairs                                                                        | allowed-address-pairs     | Provides allowed address pairs                                                                                                                           |
| project_id field enabled                                                                     | project-id                | Extension that indicates that project_id field is enabled.                                                                                               |
| Distributed Virtual Router                                                                   | dvr                       | Enables configuration of Distributed Virtual Routers.                                                                                                    |
+----------------------------------------------------------------------------------------------+---------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------+
```

```php
$ openstack network agent list
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host       | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
| 0fcf4aa9-3592-4552-9b4c-f2b55e23ef6b | DHCP agent         | controller | nova              | :-)   | UP    | neutron-dhcp-agent        |
| 1a08e5eb-d867-4697-850d-bd2400134162 | Metadata agent     | controller | None              | :-)   | UP    | neutron-metadata-agent    |
| 9a33be1e-61bd-4d6b-9ee1-bda6dc7b44cd | Linux bridge agent | controller | None              | :-)   | UP    | neutron-linuxbridge-agent |
| bfdb443d-feee-4006-8618-558b73c3c4a2 | L3 agent           | controller | nova              | :-)   | UP    | neutron-l3-agent          |
| ce5abc8d-504a-4164-ae0f-801e56a06653 | Linux bridge agent | compute    | None              | :-)   | UP    | neutron-linuxbridge-agent |
+--------------------------------------+--------------------+------------+-------------------+-------+-------+---------------------------+
```
