---
title: Windows CloudBase-Init 安装使用及配置说明
tags: [openstack]
categories: Openstack
description: Windows CloudBase-Init 安装使用及配置说明
date: 2020/10/20 10:10:00
---

1.下载Windows CloudBase-Init安装包，地址：
https://cloudbase.it/cloudbase-init/#download

当前版本为：CloudbaseInitSetup_1_1_2_x64.msi

2.安装：
![](/images/cloudbaseinit/install1.png)
![](/images/cloudbaseinit/install2.png)
安装目录默认为 C:\Program Files\Cloudbase Solutions\Cloudbase-Init\
![](/images/cloudbaseinit/install3.png)
用户名修改为当前Windows超级管理员Administrator
这里不勾选Use metadata password，勾选此项这里会在孵化虚拟机时生成新的随机密码
Serial口选择Windows当前使用的串口COM1
![](/images/cloudbaseinit/install4.png)
![](/images/cloudbaseinit/install5.png)
![](/images/cloudbaseinit/install6.png)
安装结束，不勾选Run Sysprep to create a generalized image.及 Shutdown when Sysprep terminates.
![](/images/cloudbaseinit/install7.png)

3.安装结束之后，修改Cloudbaase-init配置文件：
C:\Program Files\Cloudbase Solutions\Cloudbase-Init\conf\下的
cloudbase-init.conf及cloudbase-init-unattend.conf
配置修改为：
安装CloudBase-init后，cloudbase-init会自动重启，将其关闭重启：
allow_reboot=false

关闭CloudBase-init自动更新：
check_latest_version=false

4.* Windows在安装cloudbase-init后需要重新启动，再制作镜像。
5.使用Openstack在孵化过程中上传的文件目录为(E:)config\openstack\content\下。