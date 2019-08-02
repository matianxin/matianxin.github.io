---
title: Openstack Queens 环境搭建（八）计算节点实例首次孵化优化
tags: [openstack]
categories: Openstack
description: OpenStack项目是一个开源云计算平台，支持所有类型的云环境。该项目旨在实现简单，大规模的可扩展性和丰富的功能。OpenStack通过各种补充服务提供基础架构即服务（IaaS）解决方案。每项服务都提供了一个应用程序编程接口（API），以促进这种集成。本文涵盖了使用适用于具有足够Linux经验的OpenStack新用户的功能性示例体系结构，逐步部署主要OpenStack服务。
date: 2019/7/29 9:32:00
---
### Controller节点：

#### 创建镜像缓存目录，并调整权限

```php
# mkdir -p /var/lib/glance/imagecache
# chmod -R 777 /var/lib/glance/images
# chmod -R 777 /var/lib/glance/imagecache
```

#### 配置共享文件夹，将与控制节点管理网卡同网段的CIDR配置入文件，并赋读写权限
```php
# vim /etc/exports

/var/lib/glance/imagecache 192.100.10.0/24(rw)
```

#### 加载刚才的配置文件
```php
# exportfs -e
```

#### 注意控制节点的iptables，要放开111（rpc）、2049(nfs)和892(nfs挂载)端口，一般在OpenStack中这几个端口都是默认放开的。

#### 设置nfs服务开机启动，并启动nfs
```php
# systemctl enable rpcbind
# systemctl enable nfs
# systemctl restart rpcbind
# systemctl restart nfs
```

#### 确认本地共享目录是否正确（showmount -e）


### 计算节点

#### 计算节点配置nfs挂载以及系统开机自动挂载有脚本实现

#### 在计算节点创建文件夹，并将项目源码中的脚本拷贝上去
```php
# mkdir -p /var/www/kdpa/bin
```

#### 将项目源码中/kdpa/bin/目录下的sed_rclocal.sh、mount_nfs.sh脚本拷贝到计算节点新创建的目录下
```bash
sed_rclocal.sh

#!/usr/bin/env bash

rc_local_file="/etc/rc.d/rc.local"
chmod 755 ${rc_local_file}

# add mount nfs shell in rc.local
mount_nfs_sh="/var/www/kdpa/bin/mount_nfs.sh"
chmod 755 ${mount_nfs_sh}
echo "/usr/bin/sh ${mount_nfs_sh}">>${rc_local_file}


# add ovs set-manager shell in rc.local
ovs_set_manager_sh="/var/www/kdpa/bin/ovs_set_manager.sh"
chmod 755 ${ovs_set_manager_sh}
echo "/usr/bin/sh ${ovs_set_manager_sh}">>${rc_local_file}
```

```bash
mount_nfs.sh

#!/usr/bin/env bash

nfs_service_image_cache_path="/var/lib/glance/imagecache"
local_image_cache_path="/var/lib/nova/instances/_base"
controller_host=$(cat /etc/hosts | grep controller | awk '{print $1}')

mkdir -p ${local_image_cache_path}
chmod -R 777 ${local_image_cache_path}

mount -t nfs4 ${controller_host}:${nfs_service_image_cache_path}  ${local_image_cache_path}

```


#### 执行脚本mount_nfs.sh挂载控制节点nfs共享目录
```php
# sh /var/www/kdpa/bin/mount_nfs.sh
```

#### 验证挂载是否成功
```php
# mount -l | grep 服务端IP
```

#### 执行脚本sed_rclocal.sh调整系统开机启动脚本
```php
# sh /var/www/kdpa/bin/sed_rclocal.sh
```
最终查看/etc/rc.d/rc.local文件是否添加成功

