---
title: Chrony - 一步搞定NTP时间同步问题
tags: [linux]
categories: Linux
description: chrony是一个ntp协议的实现程序，既可以当做服务端，也可以充当客户端；它专为间歇性互联网连接的系统而设计，当然也能良好应用于持久互联网连接的环境；chrony有三个时间参考：硬件时钟、实时时钟以及手动同步。
date: 2019/9/24 16:27:08
---

### 什么是chrony？
**A client/server for the Network Time Protocol, this program keeps your computer's clock accurate. It was specially designed to support systems with intermittent internet connections, but it also works well in permanently connected environments. It can use also hardware reference clocks, system real-time clock or manual input as time references.**
chrony是一个ntp协议的实现程序，既可以当做服务端，也可以充当客户端；它专为间歇性互联网连接的系统而设计，当然也能良好应用于持久互联网连接的环境；chrony有三个时间参考：硬件时钟、实时时钟以及手动同步。

### chrony的程序环境
主配置文件：/etc/chrony.conf
客户端程序：/usr/bin/chronyc
服务端程序：/usr/sbin/chronyd

### 为本地服务器配置一个时间服务器
#### 环境
时间服务器：192.168.1.10
允许本网段192.168.1.0/24同步时间。

#### 配置服务端
在编辑配置文件之前，我一般习惯于备份一下，以备重新调整。
网络时间服务器可以自行查找，网上很多资料，下边配置两个个国内常用的时间服务器：

]# cd /etc/
etc]# cp chrony.conf{,.bak}
etc]# vim chrony.conf
    ...
    #  iburst为固定格式，记住就可以，没有深究。
    server cn.pool.ntp.org iburst
    server tw.pool.ntp.org iburst
    ...
    # 允许指定网络的主机同步时间，不指定就是允许所有，默认不开启。
    allow 192.168.1.0/24
    ...
    # 还有一个默认不开启的选项，意思是，即使服务端没有同步到精确的网络时间，也允许向客户端同步不精确的时间。可以视情况而定。
    # Serve time even if not synchronized to any NTP server.
    #local stratum 10
~]# systemctl start chronyd.service
~]# systemctl enable chronyd.service

#### 客户端同步
配置192.168.1.10作为时间服务器

etc]# vim chrony.conf
    ...
    server 192.168.1.10 iburst
    ...
~]# systemctl start chronyd.service
~]# systemctl enable chronyd.service

进入chronyc客户端交互式模式：
~]# chronyc

查看现有的时间服务器
    chronyc> sources
        210 Number of sources = 1
        MS Name/IP address         Stratum Poll Reach LastRx Last sample
        ===============================================================================
        ^* 192.168.1.10               10   6   377    53  -4546ns[-5000ns] +/-  234us

查看时间服务器状态
    chronyc> sourcestats
        210 Number of sources = 1
        Name/IP Address            NP  NR  Span  Frequency  Freq Skew  Offset  Std Dev
        ==============================================================================
        192.168.1.10               10   6   584     +0.001      0.828    +35ns    74us

    chronyc>exit

也可以直接在命令行使用：
~]# chronyc sources
~]# chronyc sourcestats

chrony兼容ntpdate，客户端可以使用ntpdate手动同步时间
```php
~]# ntpdate 192.168.1.10
 9 Nov 02:54:30 ntpdate[39551]: adjust time server 192.168.43.101 offset -0.000003 sec
```
