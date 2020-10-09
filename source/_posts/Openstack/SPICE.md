---
title: Openstack虚拟桌面协议SPICE代替vnc
tags: [openstack]
categories: Openstack
description: VNC是OpenStack的Nova默认的连接协议，面对一些简单的管理工作表现也不错，但是如果用户经常使用Windows桌面，VNC就显得能力不足。一般情况下，使用Spice协议来代替VNC。
date: 2020/5/6 14:02:08
---

### 题记
VNC是OpenStack的Nova默认的连接协议，面对一些简单的管理工作表现也不错，但是如果用户经常使用Windows桌面，VNC就显得能力不足。一般情况下，使用Spice协议来代替VNC。

### VNC
VNC (Virtual Network Computing)是虚拟网络计算机的缩写。VNC 是一款优秀的远程控制工具软件，由著名的 AT&T 的欧洲研究实验室开发的。VNC 是在基于 UNIX 和 Linux 操作系统的免费的开源软件，远程控制能力强大，高效实用，其性能可以和 Windows 和 MAC 中的任何远程控制软件媲美。 在 Linux 中，VNC 包括以下四个命令：vncserver，vncviewer，vncpasswd，和 vncconnect。大多数情况下我只需要其中的两个命令：vncserver 和 vncviewer。

### SPICE 已经支持和即将支持的功能

当前支持功能:
• 图形界面 - processes and transmits 2D graphic commands
• 视频流 - heuristically identifies video streams and transmits M-JPEG video streams
• 图片压缩 - offers verios compression algorithm that were built specifically for Spice, including QUIC (based on SFALIC), LZ, GLZ (history-based global dictionary), and auto (heuristic compression choice per image)
• 硬件鼠标- processes and transmits cursor-specific commands
• 图像,颜色,鼠标缓存 - manages client caches to reduce bandwidth requirements
• 在线切换 - supports clients while migrating Spice servers to new hosts, thus avoiding interruptions
• Windows 驱动 - Windows drivers for QXL display device and VDI-port
• 多监视器
• 客户端支持linux和windows - can be easily ported to additional platforms.
• 立体声音频 - supports audio playback and captures; audio data stream is optionally compressed using CELT
• 加密 - using OpenSSL
• 两种鼠标模式- provides client (more user-friendly) and server (increased accuracy and fully synchronized) modes
• 音频视频同步 - synchronizes video streams with audio clocks
• Spice 代理 - running on the guest and performs tasks for the client
• 剪切板共享 - allows copy paste between clients and the virtual machine

未来将支持的新功能:
• 网络隧道 (in progress) - using virtual network interface to enable sharing of network resources. Currently the focus is on printer sharing but is not limited to that.
• Off-screen surfaces (in progress) - supports off-screen surfaces as infrastructure for future DirectDraw, video acceleration and 3D acceleration. GDI and X11 will also benefit from this feature. It will also lay foundation for multi-head support
• 共享usb (in progress) - allows clients to share their USB devices with Spice servers
• Direct Draw
• 客户端GUI - Enables user-friendly configuration
• 屏幕管理 - add support for enabling selection of the screen used by the client
• 配置文件 - enables persistent user and administrative settings
• 共享光驱 - share your CD with Spice server
• 视频加速
• 3D加速
• 支持Aero
• Linux features parity
• OSX client
• Simultaneous clients connection

### Openstack启用SPICE协议

#### 安装软件包
控制节点:
```php
yum install spice-server spice-protocol openstack-nova-spicehtml5proxy spice-html5
```

计算节点:
```php
yum install spice-server spice-protocol spice-html5
```

```php
spice-html5来自epel源，spice-server，spice-protocol来自CentOS官方源
如果找不到spice-html5，添加epel源
wget http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
rpm -ivh epel-release-latest-7.noarch.rpm
yum repolist      ##检查是否已添加至源列表)

如果离线安装:
需安装epel-release-latest-7.noarch,openstack-nova-spicehtml5proxy-17.0.10-1.el7.noarch,spice-html5-0.1.7-1.el7.noarch,spice-protocol-0.12.14-1.el7.noarch,spice-server-0.14.0-9.el7.x86_64的RPM包
```

#### 修改配置文件
控制节点:
```php
vi  /etc/nova/nova.conf

这里明确两点：
指定vnc_enabled=false，否则即使配置了spice，系统也仍然使用vnc
一定要注释掉原vnc配置

[default]
vnc_enabled=false

[spice]
html5proxy_host=192.100.200.140
html5proxy_port=6082
keymap=en-us
```

计算节点:
```php
vi /etc/nova/nova.conf

[default]
vnc_enabled=false

[spice]
html5proxy_base_url=http://192.100.200.140:6082/spice_auto.html
server_listen=0.0.0.0
#server_proxyclient_address=192.100.200.150
enabled=true
keymap=en-us
```

#### 重启服务
控制节点:
```php
停止novncproxy并取消自启动

systemctl stop openstack-nova-novncproxy.service
systemctl disable openstack-nova-novncproxy.service

启用spicehtml5proxy开机自启动并启动它

systemctl enable openstack-nova-spicehtml5proxy.service
systemctl start openstack-nova-spicehtml5proxy.service
```

计算节点:
```php
systemctl restart openstack-nova-compute.service
```
