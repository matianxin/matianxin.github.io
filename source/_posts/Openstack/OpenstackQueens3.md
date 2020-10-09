---
title: Openstack Queens 环境搭建（三）Keystone服务
tags: [openstack]
categories: Openstack
description: OpenStack项目是一个开源云计算平台，支持所有类型的云环境。该项目旨在实现简单，大规模的可扩展性和丰富的功能。OpenStack通过各种补充服务提供基础架构即服务（IaaS）解决方案。每项服务都提供了一个应用程序编程接口（API），以促进这种集成。本文涵盖了使用适用于具有足够Linux经验的OpenStack新用户的功能性示例体系结构，逐步部署主要OpenStack服务。
date: 2019/3/19 16:30:00
---
### Controller节点：
#### 创建keystone数据库，授予权限：
```php
$ mysql -u root -p
密码：123456
MariaDB [(none)]> CREATE DATABASE keystone;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
IDENTIFIED BY '123456';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
IDENTIFIED BY '123456';
MariaDB [(none)]> exit;
```

#### 安装及配置组件
```php
# yum install openstack-keystone httpd mod_wsgi

# vi /etc/keystone/keystone.conf
[database]
connection = mysql+pymysql://keystone:123456@controller/keystone
[token]
provider = fernet

# su -s /bin/sh -c "keystone-manage db_sync" keystone
# keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
# keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
# keystone-manage bootstrap --bootstrap-password 123456 \
  --bootstrap-admin-url http://controller:5000/v3/ \
  --bootstrap-internal-url http://controller:5000/v3/ \
  --bootstrap-public-url http://controller:5000/v3/ \
  --bootstrap-region-id RegionOne
```

#### 配置Apache HTTP Server
```php
# vi /etc/httpd/conf/httpd.conf
ServerName controller

# ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/
```

#### 完成安装：
```php
# systemctl enable httpd.service
# systemctl start httpd.service
```

#### 配置管理帐户
```php
$ export OS_USERNAME=admin
$ export OS_PASSWORD=123456
$ export OS_PROJECT_NAME=admin
$ export OS_USER_DOMAIN_NAME=Default
$ export OS_PROJECT_DOMAIN_NAME=Default
$ export OS_AUTH_URL=http://controller:35357/v3
$ export OS_IDENTITY_API_VERSION=3
```

#### 创建域、项目、用户和角色：
```php
$ openstack domain create --description "An Example Domain" example
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | An Example Domain                |
| enabled     | True                             |
| id          | 2f338489f6c64472a0b2b6db54ecc2df |
| name        | example                          |
| tags        | []                               |
+-------------+----------------------------------+
```

```php
$ openstack project create --domain default --description "Service Project" service
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Service Project                  |
| domain_id   | default                          |
| enabled     | True                             |
| id          | 84218999229845e2ad7f4e88208b3bee |
| is_domain   | False                            |
| name        | service                          |
| parent_id   | default                          |
| tags        | []                               |
+-------------+----------------------------------+
```

```php
$ openstack project create --domain default --description "Demo Project" demo
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Demo Project                     |
| domain_id   | default                          |
| enabled     | True                             |
| id          | 5c4692ce6659454eb830e7e9633a09f1 |
| is_domain   | False                            |
| name        | demo                             |
| parent_id   | default                          |
| tags        | []                               |
+-------------+----------------------------------+
```

```php
$ openstack user create --domain default --password-prompt demo
User Password:123456
Repeat User Password:123456
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 803e7ad2e94b4af39f9be9e0742b45fd |
| name                | demo                             |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```

```php
$ openstack role create user
+-----------+----------------------------------+
| Field     | Value                            |
+-----------+----------------------------------+
| domain_id | None                             |
| id        | cbe4799bac204eacbf0012a77dc349c4 |
| name      | user                             |
+-----------+----------------------------------+

$ openstack role add --project demo --user demo user
```

#### 验证操作：
```php
$ unset OS_AUTH_URL OS_PASSWORD

$ openstack --os-auth-url http://controller:35357/v3 \
  --os-project-domain-name Default --os-user-domain-name Default \
  --os-project-name admin --os-username admin token issue
Password: 123456
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                                                                                   |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2018-09-12T09:43:34+0000                                                                                                                                                                |
| id         | gAAAAABbmNG25wIya-0xFYb3zCW3ljtDTWnr8ZCpB4iAZPMfQnP-62EGiIr6aKEjO847h6jH5nNONRqeLXO2BC_bJ0O-b5Fwj2GZpYGWRSSucAU4Mh6MqLQzetbOsRCv9-ZGO6VQYkmr0cPTEm7kzuzUL2bwTcUCbAVCpuFvCnRUZ7Hu4FE5bAI |
| project_id | 4a5e42dd8cbf410f85a5f145039d69a6                                                                                                                                                        |
| user_id    | 2ffffa1e6cbe4d239bdacc9760a54dd5                                                                                                                                                        |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

$ openstack --os-auth-url http://controller:5000/v3 \
  --os-project-domain-name Default --os-user-domain-name Default \
  --os-project-name demo --os-username demo token issue
Password: 123456
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                                                                                   |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2018-09-12T09:45:20+0000                                                                                                                                                                |
| id         | gAAAAABbmNIgtMBObdQXwOlGu-HMLvKNTBZuYvVizTCn3aDJLMvqzQRTyjhfm5RjEkAgIWcYfal9TrjZan2VWL_AZ8cASpkBwoa0TQn_rWlZw1wh8xcDeb5XNES3jMNxhtZA87peDCnMkGJoMaJVhvkR4gsDQiIUmCImzjYv6ZvJjLgGEotBszY |
| project_id | 5c4692ce6659454eb830e7e9633a09f1                                                                                                                                                        |
| user_id    | 803e7ad2e94b4af39f9be9e0742b45fd                                                                                                                                                        |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

#### 创建OpenStack客户端环境脚本：
```php
# vi /root/admin-openrc
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=123456
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2

# vi /root/demo-openrc
export OS_PROJECT_DOMAIN_NAME=Default
export OS_USER_DOMAIN_NAME=Default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=123456
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
```

#### 使用脚本验证：
```php
$ . admin-openrc

$ openstack token issue
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                                                                                   |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| expires    | 2018-09-12T09:55:59+0000                                                                                                                                                                |
| id         | gAAAAABbmNSfM00gw3qvJi-U8ytTcBxfuVhgNkETRa-gh3PqLp6Md9cW_5FfbkUL1nyQGW4Bg_XvvdIhSBv7fXRnbfyqGxTxOUloe7BmnWgM9LqLn8Fm2FLQp8qcuFamyW-9_FZA5SPqxbYS1Ozk6fO7TRDWAIWdzy5i0-qqB4Ypt6vQOyW-pqk |
| project_id | 4a5e42dd8cbf410f85a5f145039d69a6                                                                                                                                                        |
| user_id    | 2ffffa1e6cbe4d239bdacc9760a54dd5                                                                                                                                                        |
+------------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```
