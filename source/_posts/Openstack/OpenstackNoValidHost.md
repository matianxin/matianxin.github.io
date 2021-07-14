---
title: Openstack创建实例报错：No valid host was found.
tags: [openstack]
categories: Openstack
description: Openstack计算节点被删除后，重新挂载，原主机名称没有改变，在创建实例时，报错：No valid host was found 。
date: 2020/4/14 11:40:00
---

#### 错误原因：
UUID冲突，nova_api数据库中resource_providers表与nova.compute_nodes表中host的UUID不一致引起的，导致原因可能为删除计算节点时没有清空resource_providers表中的数据。


#### 解决办法：
```php
[root@controller]#  nova-manage cell_v2 list_hosts
+-----------+--------------------------------------+----------+
| Cell Name |              Cell UUID               | Hostname |
+-----------+--------------------------------------+----------+
|   cell1   | 9abb6c5c-25e2-440c-8295-4826d055298c | compute4 |
|   cell1   | 9abb6c5c-25e2-440c-8295-4826d055298c | compute5 |
|   cell1   | 9abb6c5c-25e2-440c-8295-4826d055298c | compute6 |
+-----------+--------------------------------------+----------+
```

```php
MariaDB [(none)]> use nova_api;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [nova_api]> select uuid,name from resource_providers where name='compute5';
+--------------------------------------+----------+
| uuid                                 | name     |
+--------------------------------------+----------+
| 305cc2a4-75e8-496f-b843-152400257c35 | compute5 |
+--------------------------------------+----------+
1 row in set (0.00 sec)

MariaDB [nova_api]> select uuid,host from nova.compute_nodes where host='compute5';
+--------------------------------------+----------+
| uuid                                 | host     |
+--------------------------------------+----------+
| 924c0d27-1952-4663-8f01-e8cecc67f964 | compute5 |
+--------------------------------------+----------+
1 row in set (0.00 sec)
```

可以看到，UUID为compute5的计算节点在resource_providers表中与nova.compute_nodes表中数据不一致，将resource_providers表中数据修改：

```php
MariaDB [nova_api]>  update resource_providers set uuid='924c0d27-1952-4663-8f01-e8cecc67f964' where name='compute5' and uuid='305cc2a4-75e8-496f-b843-152400257c35';
Query OK, 1 row affected (0.01 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

修改后，compute5上实例可以成功创建。
