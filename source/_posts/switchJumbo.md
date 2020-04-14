---
title: 交换机端口开启巨帧功能
tags: [tool]
categories: 工具
description: set jumbo port
date: 2020/4/14 10:30:00
---

#### 命令功能：
开启/关闭端口巨帧功能。

#### 命令模式：
全局配置模式

#### 命令格式：

```php
set jumbo port <portlist>{enable | disable}

```

#### 命令参数解释：
参数                 描述
----                ----
<portlist>          端口列表
enable              开启端口巨帧功能
disable             关闭端口巨帧功能

#### 使用说明：
ZXR10 2950设备可以转发10k大小的巨帧。

#### 缺省：
巨帧功能默认为disable状态。
