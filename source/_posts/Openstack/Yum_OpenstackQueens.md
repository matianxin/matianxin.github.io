---
title: 构建openstack Queens版本本地yum源
tags: [openstack]
categories: Openstack
description: 构建openstack Queens版本本地yum源
date: 2019/10/24 16:00:00
---

#### 选择一台CentOS服务器，安装以下软件：
```php
yum install yum-utils createrepo yum-plugin-priorities httpd
```

#### 开启httpd服务
```php
systemctl start httpd
```

#### 获取repo文件并使用reposync同步源
```php
yum install -y https://repos.fedorapeople.org/repos/openstack/openstack-queens/rdo-release-queens-2.noarch.rpm
```

查看源ID列表
```php
yum repolist
```

#### 同步openstack-queens这个repo
```php
cd /var/www/html/
reposync --repoid=openstack-queens
```

#### 第一次同步时间较长，同步结束后，执行
```php
createrepo –update /var/www/html/openstack-queens
```

#### 创建完成后，就可以使用web测试：
```php
http://[ip]/openstack-queens/
```

#### 选择另外一台服务器作为客户机
```php
cd /etc/yum.repos.d
mv CentOS-Base.repo CentOS-Base.repo_bak
cp CentOS-Media.repo CentOS-Media.repo_bak
vim CentOS-Media.repo

[openstack-queens]
name=OpenStack Queens Repository
baseurl=http://47.98.122.105/openstack-queens/
enabled=1
gpgcheck=0
```

#### 配置完成，清除缓存并查看软件包
```php
yum clean all
yum list
```

#### 问题
客户端yum安装报错：
```php
Error downloading packages:
  glusterfs-libs-7.5-1.el7.x86_64: failed to retrieve Packages/g/glusterfs-libs-7.5-1.el7.x86_64.rpm from centos7-gluster
error was [Errno 2] Local file does not exist: /etc/yum.repos.d/pdate/Packages/g/glusterfs-libs-7.5-1.el7.x86_64.rpm
  glusterfs-7.5-1.el7.x86_64: failed to retrieve Packages/g/glusterfs-7.5-1.el7.x86_64.rpm from centos7-gluster
error was [Errno 2] Local file does not exist: /etc/yum.repos.d/pdate/Packages/g/glusterfs-7.5-1.el7.x86_64.rpm
```
问题解决：
YUM服务器删除对应的repodata文件夹，使用如下命令
```php
createrepo -pdo /var/www/html/kdpa/ /var/www/html/kdpa/
```
重新生成repo链接，问题解决
