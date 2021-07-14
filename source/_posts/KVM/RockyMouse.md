---
title: KVM安装凝思8解决鼠标漂移的问题
tags: [KVM]
categories: KVM
description: KVM安装凝思8解决鼠标漂移的问题。
date: 2020/7/12 19:30:00
---

KVM环境安装凝思8(V6.0.8)版本: Linx-6.0.80-20180821-amd64-DVD.iso
安装完成后，鼠标漂移严重，环境无法使用，解决方法如下:
需要在virt-manager修改虚拟机显卡配置项:

#### 1.显示协议

**修改显示协议 由SPICE服务器 -> VNC服务器**

![](/images/kvm/display_protocol.png)

#### 2.显卡

**修改显卡 由 QXL -> Cirrus**

![](/images/kvm/display_card.png)

#### 3.磁盘
**磁盘 - 高级选项 - 磁盘总线 IDE -> VirtIO**
