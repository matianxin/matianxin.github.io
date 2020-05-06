---
title: Openstack凝思系统虚拟桌面鼠标漂移的问题
tags: [openstack]
categories: Openstack
description: Openstack凝思系统虚拟桌面鼠标漂移的问题。
date: 2020/5/6 15:37:08
---

### 问题描述
在OpenstackQueens环境下分别安装Windows，Kali及凝思操作系统，发现凝思操作系统出现鼠标漂移的问题。
经调查，默认情况下有图型界面的实例使用的输入设备类型为usbtablet，
Windows，Kali在usbtablet下没有问题，凝思操作系统鼠标漂移。
凝思在使用ps2mouse下没有问题，而ps2mouse下Windows，Kali鼠标出现漂移。

### 解决办法
镜像定制化:
在制作除凝思操作系统的镜像时，使用定制属性:
```php
hw_pointer_model='usbtablet'
```
即使用glanceClient创建镜像时，
```php
glanceClient.images.create(
    name=imageName, container_format='bare', disk_format='qcow2', hw_pointer_model='usbtablet')
```

或者镜像创建后，修改镜像信息:
```php
 glance image-update --property hw_pointer_model=usbtablet [IMAGE-ID]
```

然后在nova配置文件中[DEFAULT]中增加:
```php
[DEFAULT]
...
pointer_model=ps2mouse
```
更新后重启计算节点的Nova-compute服务:
```php
systemctl restart openstack-nova-compute.service
```

这时，使用hw_pointer_model属性的镜像默认仍使用镜像定制的usbtablet，
而凝思系统则使用ps2mouse的输入方式。
