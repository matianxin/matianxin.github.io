---
title: Openstack Queens 环境搭建（七）Horizon服务
tags: [openstack]
categories: Openstack
description: OpenStack项目是一个开源云计算平台，支持所有类型的云环境。该项目旨在实现简单，大规模的可扩展性和丰富的功能。OpenStack通过各种补充服务提供基础架构即服务（IaaS）解决方案。每项服务都提供了一个应用程序编程接口（API），以促进这种集成。本文涵盖了使用适用于具有足够Linux经验的OpenStack新用户的功能性示例体系结构，逐步部署主要OpenStack服务。
date: 2019/3/19 16:50:00
---
### Controller节点：

#### 安装及配置：
```php
# yum install openstack-dashboard

# vi /etc/openstack-dashboard/local_settings
OPENSTACK_HOST = "controller"
ALLOWED_HOSTS = ['*']
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
CACHES = {
    'default': {
         'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
         'LOCATION': 'controller:11211',
    }
}
OPENSTACK_KEYSTONE_URL = "http://%s:5000/v3" % OPENSTACK_HOST
OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
OPENSTACK_API_VERSIONS = {
    "identity": 3,
    "image": 2,
    "volume": 2,
}
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "Default"
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "user"
TIME_ZONE = "Asia/Shanghai"
```

```php
# vi /etc/httpd/conf.d/openstack-dashboard.conf          在文件开头添加
WSGIApplicationGroup %{GLOBAL}
...
```

#### 完成安装：
```php
# systemctl restart httpd.service memcached.service
```

使用 http://controller/dashboard 上的Web浏览器访问Dashboard。
