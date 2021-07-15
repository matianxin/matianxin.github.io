---
title: Openstack 虚拟机修改ERROR状态为ACTIVE
tags: [openstack]
categories: Openstack
description: 此方法针对于计算节点正常重启后，纵向的状态被Openstack置为错误状态的处理方法。
date: 2021/7/05 15:00:00
---

#### 1.SSH登录Controller节点，执行授权脚本
``` bash
 . /opt/kdpastack/admin-openrc
 ```

#### 2.浏览器登录Openstack的WebUI：http://controller:10080/dashboard
 进入项目 -> 计算 -> 实例，
 筛选条件选择：状态=， 筛选框中输入ERROR，筛选状态为错误的实例列表，
 点击 实例名称 如：ps_T16Z2PS-SCENE32@-1，进入实例概况页面，
 复制【ID】后面32位字符串。

#### 3.Controller节点，执行重置状态操作：
``` bash
 nova reset-state 2步复制的ID --active
```

#### 4.Controller节点，执行将实例关机：
``` bash
 nova stop 2步复制的ID
```

#### 5.以上步骤结束后，登录实训平台，场景管理对应的场景页面，
 在场景中对此错误状态的实例进行开机操作，
 *如果实例当前状态为开机，需先关机再开机。


