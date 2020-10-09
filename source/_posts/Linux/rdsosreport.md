---
title: Centos7断电导致generating /run/initramfs/rdsosreport.txt 问题
tags: [linux]
categories: Linux
description: Centos7断电导致generating /run/initramfs/rdsosreport.txt 问题
date: 2020/4/14 9:06:08
---

#### 开机就进入命令窗口，窗口提示信息如下：

```php
generating "/run/initramfs/rdsosreport.txt"
entering emergencymode. exit the shell to continue
type "journalctl" to view system logs.
you might want to save "/run/initramfs/rdsosreport.txt" to a usb stick or /boot after mounting them and attach it to a bug report.

```

#### 解决办法：
```php
xfs_repair /dev/mapper/centos-root -L

```

```php
reboot

```
