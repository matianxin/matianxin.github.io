---
title: Openstack控制节点删除计算节点的方法
tags: [openstack]
categories: Openstack
description: Openstack控制节点删除计算节点的方法
date: 2020/4/14 13:40:00
---

在控制节点Controller的操作：删除计算节点名称为compute5：
1.查看计算主机及服务相关：
```php
[root@controller ~]# openstack host list
+------------+-------------+----------+
| Host Name  | Service     | Zone     |
+------------+-------------+----------+
| controller | consoleauth | internal |
| controller | scheduler   | internal |
| controller | conductor   | internal |
| compute4   | compute     | nova     |
| compute6   | compute     | nova     |
| compute5   | compute     | nova     |
+------------+-------------+----------+
```

```php
[root@controller ~]# nova service-list
+--------------------------------------+------------------+------------+----------+---------+-------+----------------------------+-----------------+-------------+
| Id                                   | Binary           | Host       | Zone     | Status  | State | Updated_at                 | Disabled Reason | Forced down |
+--------------------------------------+------------------+------------+----------+---------+-------+----------------------------+-----------------+-------------+
| 936a4177-d08d-4f62-bb8c-7da7047ab6f8 | nova-consoleauth | controller | internal | enabled | up    | 2020-04-14T05:59:07.000000 | -               | False       |
| c1395bf8-47dd-47b0-bc8c-e10fc083ad40 | nova-scheduler   | controller | internal | enabled | up    | 2020-04-14T05:59:11.000000 | -               | False       |
| 28958175-5745-4b9f-b71d-e431168f6119 | nova-conductor   | controller | internal | enabled | up    | 2020-04-14T05:59:10.000000 | -               | False       |
| c586b1d2-f326-4e7e-8a3a-fead78a151c0 | nova-compute     | compute4   | nova     | enabled | up    | 2020-04-14T05:59:11.000000 | -               | False       |
| b6f3a4e3-a0ec-4f41-b47d-976bd45581b6 | nova-compute     | compute6   | nova     | enabled | up    | 2020-04-14T05:59:03.000000 | -               | False       |
| bb93240f-58f4-4d3c-a040-a66a9b2f0ea9 | nova-compute     | compute5   | nova     | enabled | down  | 2020-04-14T05:59:12.000000 | -               | False       |
+--------------------------------------+------------------+------------+----------+---------+-------+----------------------------+-----------------+-------------+
```

计算节点compute的State状态是down，但Status状态还是enabled可用。
2.修改compute5为不可用状态。
```php
[root@controller ~]# nova service-disable bb93240f-58f4-4d3c-a040-a66a9b2f0ea9
```

查看是否修改成功
```php
[root@controller ~]# nova service-list
+--------------------------------------+------------------+------------+----------+---------+-------+----------------------------+-----------------+-------------+
| Id                                   | Binary           | Host       | Zone     | Status  | State | Updated_at                 | Disabled Reason | Forced down |
+--------------------------------------+------------------+------------+----------+---------+-------+----------------------------+-----------------+-------------+
| 936a4177-d08d-4f62-bb8c-7da7047ab6f8 | nova-consoleauth | controller | internal | enabled | up    | 2020-04-14T05:59:07.000000 | -               | False       |
| c1395bf8-47dd-47b0-bc8c-e10fc083ad40 | nova-scheduler   | controller | internal | enabled | up    | 2020-04-14T05:59:11.000000 | -               | False       |
| 28958175-5745-4b9f-b71d-e431168f6119 | nova-conductor   | controller | internal | enabled | up    | 2020-04-14T05:59:10.000000 | -               | False       |
| c586b1d2-f326-4e7e-8a3a-fead78a151c0 | nova-compute     | compute4   | nova     | enabled | up    | 2020-04-14T05:59:11.000000 | -               | False       |
| b6f3a4e3-a0ec-4f41-b47d-976bd45581b6 | nova-compute     | compute6   | nova     | enabled | up    | 2020-04-14T05:59:03.000000 | -               | False       |
| bb93240f-58f4-4d3c-a040-a66a9b2f0ea9 | nova-compute     | compute5   | nova     | disabled| down  | 2020-04-14T05:59:12.000000 | -               | False       |
+--------------------------------------+------------------+------------+----------+---------+-------+----------------------------+-----------------+-------------+
```

3.在数据库（nova）中清理
```php
[root@controller ~]# mysql -p
Enter password:
Welcome to the MariaDB monitor. Commands end with ; or g.
Your MariaDB connection id is 980
Server version: 10.1.20-MariaDB MariaDB Server
Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.
Type 'help;' or 'h' for help. Type 'c' to clear the current input statement.
MariaDB [(none)]> use nova
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A
Database changed
MariaDB [nova]> delete from nova.services where host="compute5";
Query OK, 1 row affected (0.01 sec)
MariaDB [nova]> delete from compute_nodes where hypervisor_hostname="compute5";
Query OK, 0 rows affected (0.00 sec)

MariaDB [nova]> select host from nova.services;

MariaDB [nova]> select hypervisor_hostname from compute_nodes;
```
4.校验
```php
[root@controller ~]# openstack host list
[root@controller ~]# nova service-list
```
再次查看计算节点，就发现compute已经被删除了。
