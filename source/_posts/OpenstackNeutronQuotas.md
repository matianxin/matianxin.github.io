---
title: Openstack修改系统网络配额
tags: [openstack]
categories: Openstack
description: Openstack修改系统网络配额
date: 2019/7/11 15:28:00
---

### 前言
neutron在安装配置完成之后，openstack为了实现对所有tenant对网络资源的使用，针对neutron设置有专门的配额，以防止租户使用过多的资源，而对其他的tenant造成影响。和nova的quota相类似，neutron也使用单独的一个驱动来实现网络neutron的配额控制。

### neutron默认的配额
neutron默认的配额针对network，port，router，subnet，floatingip做了配额方面的限定，参考neutron的配置文件，获取quota的配额内容为:
```php
[root@controller ~]# vim /etc/neutron/neutron.conf
[quotas]
quota_driver = neutron.db.quota_db.DbQuotaDriver        配额驱动
quota_items = network,subnet,port                       quota限定的范畴
default_quota = -1                                      默认的quota，-1表示没有限制(未启用)
quota_network = 10                                      建立的network个数
quota_subnet = 10                                       建立的subnet个数
quota_port = 50                                         允许的port个数
quota_security_group = 10                               安全组的个数
quota_security_group_rule = 100                         安全组规规则条数
quota_vip = 10                                          vip个数，以下的quota_member和quota_health_monitors 都用于LBaaS场景
quota_pool = 10                                         pool个数
quota_member = -1                                       member个数
quota_health_monitors = -1                              monitor个数
quota_router = 10                                       router的个数
quota_floatingip = 50                                   floating-ip个数
```

### 修改neutron的配额
查看neutron默认的配额
```php
[root@controller ~]# keystone tenant-list
+----------------------------------+----------+---------+
|                id                |   name   | enabled |
+----------------------------------+----------+---------+
| 842ab3268a2c47e6a4b0d8774de805ae |  admin   |   True  |
| 7ff1dfb5a6f349958c3a949248e56236 | companyA |   True  |        #得到tenant的uuid号
| 10d1465c00d049fab88dec1af0f56b1b |   demo   |   True  |
| 3b57a14f7c354a979c9f62b60f31a331 | service  |   True  |
+----------------------------------+----------+---------+
```

```php
[root@controller ~]# neutron quota-show --tenant-id 7ff1dfb5a6f349958c3a949248e56236
+---------------------+-------+
| Field               | Value |
+---------------------+-------+
| floatingip          | 50    |
| health_monitor      | -1    |
| member              | -1    |
| network             | 10    |
| pool                | 10    |
| port                | 50    |          #port，每台虚拟机都需要一个ip，即一个port，很容易就超过配额
| router              | 10    |
| security_group      | 10    |
| security_group_rule | 100   |
| subnet              | 10    |
| vip                 | 10    |
+---------------------+-------+
```

修改neutron配额
```php
[root@controller ~]# neutron quota-update --network 20 --subnet 20 --port 100  --router 5 --floatingip 100  --security-group 10 --security-group-rule 100 --tenant-id 7ff1dfb5a6f349958c3a949248e56236
+---------------------+-------+
| Field               | Value |
+---------------------+-------+
| floatingip          | 100   |
| health_monitor      | -1    |
| member              | -1    |
| network             | 20    |
| pool                | 10    |
| port                | 100   |
| router              | 5     |
| security_group      | 10    |
| security_group_rule | 100   |
| subnet              | 20    |
| vip                 | 10    |
+---------------------+-------+
```

校验neutron的quota配置
```php
[root@controller ~]# neutron quota-show --tenant-id 7ff1dfb5a6f349958c3a949248e56236
+---------------------+-------+
| Field               | Value |
+---------------------+-------+
| floatingip          | 100   |
| health_monitor      | -1    |
| member              | -1    |
| network             | 20    |
| pool                | 10    |
| port                | 100   |
| router              | 5     |
| security_group      | 10    |
| security_group_rule | 100   |
| subnet              | 20    |
| vip                 | 10    |
+---------------------+-------+
```

### 统计port的个数
```php
[root@controller ~]# neutron port-list
+--------------------------------------+------+-------------------+---------------------------------------------------------------------------------------+
| id                                   | name | mac_address       | fixed_ips                                                                             |
+--------------------------------------+------+-------------------+---------------------------------------------------------------------------------------+
| 0060ec4a-957d-4571-b730-6b4a9bb3baf8 |      | fa:16:3e:48:42:3d | {"subnet_id": "9654a807-d4fa-49f1-abb6-2e45d776c69f", "ip_address": "10.16.4.19"}     |
| 00942be0-a3a9-471d-a4ba-336db0ee1539 |      | fa:16:3e:73:75:03 | {"subnet_id": "ad4a5ffc-3ccc-42c4-89a1-61e7b18632a3", "ip_address": "10.16.6.96"}     |
| 0119045c-8219-4744-bd58-a7e77294832c |      | fa:16:3e:10:ed:7f | {"subnet_id": "9654a807-d4fa-49f1-abb6-2e45d776c69f", "ip_address": "10.16.4.71"}     |
| 04f7d8ea-1849-4938-9ef7-e8114893132f |      | fa:16:3e:50:86:1b | {"subnet_id": "ad4a5ffc-3ccc-42c4-89a1-61e7b18632a3", "ip_address": "10.16.6.27"}     |

[root@controller ~]# neutron port-list |wc -l            #超过配额时，需要修改
194
```

### 总结
随着时间的推移，当越来越多的instance加入到openstack中，port也会相应增加，一个ip对应一个port，所以当port达到配额时，openstack会组织用户继续分配虚拟机，此时，就需要修改neutron的配额了，关于neutron配额的报错，可以参考neutron的日志/var/log/neutron/neutron-server.log，可以根据日志的信息，定位到报错的原因，具体不赘述。

### 附录
neutron实现quota的代码解读

```python
[root@controller ~]# vim /usr/lib/python2.6/site-packages/neutron/db/quota_db.py

import sqlalchemy as sa

from neutron.common import exceptions
from neutron.db import model_base
from neutron.db import models_v2

'''
quota数据库表的表结构，tenant默认集成的配额从这里获取
mysql> desc quotas;
+-----------+--------------+------+-----+---------+-------+
| Field     | Type         | Null | Key | Default | Extra |
+-----------+--------------+------+-----+---------+-------+
| id        | varchar(36)  | NO   | PRI | NULL    |       |
| tenant_id | varchar(255) | YES  | MUL | NULL    |       |
| resource  | varchar(255) | YES  |     | NULL    |       |
| limit     | int(11)      | YES  |     | NULL    |       |
+-----------+--------------+------+-----+---------+-------+
'''
class Quota(model_base.BASEV2, models_v2.HasId):
    """Represent a single quota override for a tenant.

    If there is no row for a given tenant id and resource, then the
    default for the quota class is used.
    """
    tenant_id = sa.Column(sa.String(255), index=True)
    resource = sa.Column(sa.String(255))
    limit = sa.Column(sa.Integer)

'''
quota配额的具体实现，根据数据库的配置内容，实现quota的控制，即quota的增删改查方法
'''
class DbQuotaDriver(object):
    """Driver to perform necessary checks to enforce quotas and obtain quota
    information.

    The default driver utilizes the local database.
    """

    '''
    得到租户tenant的quota，执行neutron quota-show --tenant-id uuid时调用的方法
    '''
    @staticmethod
    def get_tenant_quotas(context, resources, tenant_id):
        """Given a list of resources, retrieve the quotas for the given
        tenant.

        :param context: The request context, for access checks.
        :param resources: A dictionary of the registered resource keys.
        :param tenant_id: The ID of the tenant to return quotas for.
        :return dict: from resource name to dict of name and limit
        """

        # init with defaults    得到quota默认的配额项item，即所谓的network，subnet，port和router等，以及对应的值
        tenant_quota = dict((key, resource.default)
                            for key, resource in resources.items())

        # update with tenant specific limits    从数据库中获取最新的quota配置信息，并更新
        q_qry = context.session.query(Quota).filter_by(tenant_id=tenant_id)
        tenant_quota.update((q['resource'], q['limit']) for q in q_qry)

        return tenant_quota

    '''
    quota的删除，即执行neutron quota-delete 的方法，删除之后，tenant将会集成默认的的quota配置
    '''
    @staticmethod
    def delete_tenant_quota(context, tenant_id):
        """Delete the quota entries for a given tenant_id.

        Atfer deletion, this tenant will use default quota values in conf.
        """
        #从neutron。quotas数据库中查询到所有的quota配置之后，过略某个具体的tenant的quota，之后执行delete()方法将其删除
        with context.session.begin():
            tenant_quotas = context.session.query(Quota)
            tenant_quotas = tenant_quotas.filter_by(tenant_id=tenant_id)
            tenant_quotas.delete()

    '''
    得到所有租户tenant的配额资源，即执行neutron quota-list所查看的内容
    '''
    @staticmethod
    def get_all_quotas(context, resources):
        """Given a list of resources, retrieve the quotas for the all tenants.

        :param context: The request context, for access checks.
        :param resources: A dictionary of the registered resource keys.
        :return quotas: list of dict of tenant_id:, resourcekey1:
        resourcekey2: ...
        """
        tenant_default = dict((key, resource.default)
                              for key, resource in resources.items())

        all_tenant_quotas = {}

        for quota in context.session.query(Quota):
            tenant_id = quota['tenant_id']

            # avoid setdefault() because only want to copy when actually req'd
            #如果quotas表中，没有找到配置选项，说明使用默认的quota配置，直接用默认的copy过来即可，有配置则继承quotas表中的配置
            tenant_quota = all_tenant_quotas.get(tenant_id)
            if tenant_quota is None:
                tenant_quota = tenant_default.copy()
                tenant_quota['tenant_id'] = tenant_id
                all_tenant_quotas[tenant_id] = tenant_quota

            tenant_quota[quota['resource']] = quota['limit']

        return all_tenant_quotas.values()

    '''
                更新quota的配置，即执行neutron quota-update命令的具体实现
    '''
    @staticmethod
    def update_quota_limit(context, tenant_id, resource, limit):
        with context.session.begin():
            tenant_quota = context.session.query(Quota).filter_by(
                tenant_id=tenant_id, resource=resource).first()

            #有配置内容，则更新，没有则根据资源的配置内容，在数据库中添加对应的条目
            if tenant_quota:
                tenant_quota.update({'limit': limit})
            else:
                tenant_quota = Quota(tenant_id=tenant_id,
                                     resource=resource,
                                     limit=limit)
                context.session.add(tenant_quota)

    def _get_quotas(self, context, tenant_id, resources, keys):
        """Retrieves the quotas for specific resources.

        A helper method which retrieves the quotas for the specific
        resources identified by keys, and which apply to the current
        context.

        :param context: The request context, for access checks.
        :param tenant_id: the tenant_id to check quota.
        :param resources: A dictionary of the registered resources.
        :param keys: A list of the desired quotas to retrieve.

        """
        desired = set(keys)
        sub_resources = dict((k, v) for k, v in resources.items()
                             if k in desired)

        # Make sure we accounted for all of them...
        if len(keys) != len(sub_resources):
            unknown = desired - set(sub_resources.keys())
            raise exceptions.QuotaResourceUnknown(unknown=sorted(unknown))

        # Grab and return the quotas (without usages)
        quotas = DbQuotaDriver.get_tenant_quotas(
            context, sub_resources, tenant_id)

        return dict((k, v) for k, v in quotas.items())

    '''
    neutron quota的校验，即在执行过程中，调用该方法，确认tenant的quota是否在合理的范围内
    '''
    def limit_check(self, context, tenant_id, resources, values):
        """Check simple quota limits.

        For limits--those quotas for which there is no usage
        synchronization function--this method checks that a set of
        proposed values are permitted by the limit restriction.

        This method will raise a QuotaResourceUnknown exception if a
        given resource is unknown or if it is not a simple limit
        resource.

        If any of the proposed values is over the defined quota, an
        OverQuota exception will be raised with the sorted list of the
        resources which are too high.  Otherwise, the method returns
        nothing.

        :param context: The request context, for access checks.
        :param tenant_id: The tenant_id to check the quota.
        :param resources: A dictionary of the registered resources.
        :param values: A dictionary of the values to check against the
                       quota.
        """

        # Ensure no value is less than zero    quota的配置值不能为负数
        unders = [key for key, val in values.items() if val < 0]
        if unders:
            raise exceptions.InvalidQuotaValue(unders=sorted(unders))

        # Get the applicable quotas
        quotas = self._get_quotas(context, tenant_id, resources, values.keys())

        # Check the quotas and construct a list of the resources that
        # would be put over limit by the desired values
        overs = [key for key, val in values.items()
                 if quotas[key] >= 0 and quotas[key] < val]
        if overs:
            raise exceptions.OverQuota(overs=sorted(overs))
```


摘自：https://segmentfault.com/a/1190000018750816
