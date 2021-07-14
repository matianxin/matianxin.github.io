---
title: pfSense防火墙扩展磁盘大小
tags: [linux]
categories: Firewall
description: pfSense防火墙扩展磁盘大小
date: 2020/10/10 10:45:00
---

```php
df -h
gpart show
su
gpart resize -i 1 -s 38G -a 4k vtbd0
gpart show
gpart delete -i 2 vtbd0s1
gpart resize -i 1 -s 47G -a 4k vtbd0s1
gpart show
growfs /dev/ufsid/5cde6f38370e9a3f
df -h

```

参考freebsd官方文档：https://www.freebsd.org/doc/handbook/disks-growing.html