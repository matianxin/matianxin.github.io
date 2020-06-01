---
title: Openstack 安全组规则不生效
tags: [openstack]
categories: Openstack
description: Openstack安全组规则不生效，使用OpenVswitch的Iptables调整配置如下。
date: 2020/2/14 16:33:05
---

#### 编辑 /etc/sysctl.conf 文件

这里强调，安全组主要是依靠计算节点的iptables的forward链来生效的，每加一条规则就会根据网卡作为匹配条件，来生成一条iptables的规则。 如果没有任何规则，默认是丢弃所有的包。 猜测到的原因是因为，没有开启包转发功能，所以修改

```php
net.ipv4.ip_forward=1
net.ipv4.conf.default.rp_filter=1
net.bridge.bridge-nf-call-ip6tables=1
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-arptables=1
```
重启网络
```php
/etc/init.d/network restart
```

#### 编辑 /etc/neutron/plugins/ml2/openvswitch_agent.ini 文件

增加或修改项，包括控制节点和计算节点

```php
[securitygroup]
enable_security_group = true
firewall_driver = neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver
```

控制节点重启所有网络服务，计算节点重启Openvswitch服务。

注：
neutron 也提供两种安全组的实现：IptablesFirewallDriver 和 OVSHybridIptablesFirewallDriver
neutron.agent.linux.iptables_firewall.IptablesFirewallDriver  iptables-based FirewallDriver implementation
neutron.agent.linux.iptables_firewall.OVSHybridIptablesFirewallDriver: subclass of IptablesFirewallDriver with additional bridge
默认值是 neutron.agent.firewall.NoopFirewallDriver，表示不使用 neutron security group。
