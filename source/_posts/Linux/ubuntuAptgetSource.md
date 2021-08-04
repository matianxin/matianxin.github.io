---
title: Ubuntu16.04下的apt-get内网本地源的搭建
tags: [linux]
categories: Linux
description: APT本地源的搭建（可用于局域网apt-get源搭建或者本地源）介绍使用apt-mirror软件搭建apt本地源。需求：内网开发环境由于其特定原因不能上外网，所以需要本地环境下的内网源来方便开发人员下载安装软件。
date: 2021/8/4 11:00:00
---

#### 服务端配置

### 1.安装apt-mirror
```bash
apt-get install apt-mirror
```

### 2.修改apt-mirror配置文件

```bash
vim /etc/apt/mirror.list
```

```bash
参考以下配置文件：
清空原有的配置文件，直接使用以下配置文件即可

############# config ##################
# 以下注释的内容都是默认配置，如果需要自定义，取消注释修改即可
set base_path /var/spool/apt-mirror
#
# 镜像文件下载地址
# set mirror_path $base_path/mirror
# 临时索引下载文件目录，也就是存放软件仓库的dists目录下的文件（默认即可）
# set skel_path $base_path/skel
# 配置日志（默认即可）
# set var_path $base_path/var
# clean脚本位置
# set cleanscript $var_path/clean.sh
# 架构配置，i386/amd64，默认的话会下载跟本机相同的架构的源
set defaultarch amd64
# set postmirror_script $var_path/postmirror.sh
# set run_postmirror 0
# 下载线程数
set nthreads 20
set _tilde 0
#
############# end config ##############
# Ali yun（这里没有添加deb-src的源）
deb http://mirrors.aliyun.com/ubuntu/ trusty main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ trusty-backports main restricted universe multiverse

clean http://mirrors.aliyun.com/ubuntu
```

### 3.开始同步
```bash
apt-miiror
```
然后等待很长时间（该镜像差不多100G左右，具体时间看网络环境），同步的镜像文件目录为/var/spool/apt-mirror/mirror/mirrors.aliyun.com/ubuntu/，当然如果增加了其他的源，在/var/spool/apt-mirror/mirror目录下还有其他的地址为名的目录。


注意：当apt-mirror 被意外中断时，只需要重新运行即可，apt-mirror支持断点续存；另外，意外关闭，需要在/var/spool/apt-mirror/var目录下面删除 apt-mirror.lock文件【 sudo rm apt-mirror.lock 】，之后执行apt-mirror重新启动


### 4.安装apache2
```bash
apt-get install apache2
```
由于Apache2的默认网页文件目录位于/var/www/html，因此，可以做个软链接（这样我们就可以直接访问了，无需将其直接导入该目录）

```bash
ln -s /var/spool/apt-mirror/mirror/mirrors.aliyun.com/ubuntu /var/www/html/ubuntu
```
然后就可以通过如下地址访问了

```bash
http://[host]:[port]/ubuntu   #ip和port是自己本机的，其中端口默认为80
```

#### 客户端配置

### 1.编辑/etc/apt/source.list，加入以下内容
```bash

# Local Source 　　　　 #ip和port是自己本机的，其中端口默认为80
deb [arch=amd64] http://[host]:[port]/ubuntu/ trusty main restricted universe multiverse
deb [arch=amd64] http://[host]:[port]/ubuntu/ trusty-security main restricted universe multiverse
deb [arch=amd64] http://[host]:[port]/ubuntu/ trusty-updates main restricted universe multiverse
deb [arch=amd64] http://[host]:[port]/ubuntu/ trusty-proposed main restricted universe multiverse
deb [arch=amd64] http://[host]:[port]/ubuntu/ trusty-backports main restricted universe multiverse为80
```

### 2.更新apt-get源
```bash
apt-update　　　　#这步很重要
```
