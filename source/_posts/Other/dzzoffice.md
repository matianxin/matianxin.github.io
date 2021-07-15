---
title: centos7部署dzzoffice私有网盘
tags: [tool]
categories: 工具
description: centos7部署dzzoffice最新版详细教程
date: 2020/7/6 9:30:00
---

#### 搭建Lamp框架(Linux+apache+mariadb+php)
1.首先安装wget
```php
yum install -y wget
```
2.下载epel额外包及YUM源配置
```php
wget https://mirrors.aliyun.com/epel/RPM-GPG-KEY-EPEL-7       #epel额外包的密钥
wget https://mirrors.aliyun.com/epel/epel-release-latest-7.noarch.rpm    #epel额外包的yum的配置
```
3.导入密钥，为了防止安装错误，安装yum的配置文件
```php
rpm --import   RPM-GPG-KEY-EPEL-7    #导入epel额外包的密钥
rpm -ivh epel-release-latest-7.noarch.rpm    #安装epel额外包的yum配置
```
4.安装升级php，作为依赖httpd都安装好了，apache(httpd)和php就安装好了
```php
yum install -y php php-mysql           #升级php的软件包

 php                x86_64        7.3.4-1.el7.remi              php           3.2 M
为依赖而安装:
 libargon2          x86_64        20161029-3.el7                epel           23 k
 apr                x86_64        1.4.8-3.el7_4.1               centos        103 k
 apr-util           x86_64        1.5.2-6.el7                   centos         92 k
 httpd              x86_64        2.4.6-88.el7.centos           centos        2.7 M
 httpd-tools        x86_64        2.4.6-88.el7.centos           centos         90 k
 mailcap            noarch        2.1.41-2.el7                  centos         31 k
 php-cli            x86_64        7.3.4-1.el7.remi              php           4.9 M
 php-common         x86_64        7.3.4-1.el7.remi              php           1.1 M
 php-json           x86_64        7.3.4-1.el7.remi              php            63 k
———————————
```
5.安装mariadb数据库
```php
yum -y install mariadb mariadb-server    #安装mariadb数据库
```
6.关闭防火墙服务及关机selinux
```php
systemctl stop firewalld
systemctl disable firewalld
setenforce 0
vim /etc/selinux/config -> disabled
```
7.开启mariadb和httpd,设置开机自动启动；执行脚本，设置mariadb数据库参数
```php

[root@localhost ~]# systemctl start mariadb      #开启mariadb
[root@localhost ~]# systemctl start  httpd       #开启httpd
[root@localhost ~]# systemctl enable mariadb     #设置开机自动开启mariadb
[root@localhost ~]# systemctl enable httpd       #设置开机自动开启httpd

mysql_secure_installation                #执行脚本
...#省略部分内容
Set root password? [Y/n] y
New password: 123456
Re-enter new password: 123456
Password updated successfully!
Reload privilege tables now? [Y/n] y
 ... Success!
Disallow root login remotely? [Y/n] n
 ... skipping.
Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!
Reload privilege tables now? [Y/n] y
 ... Success!

```

#### 安装dzzoffice

1.下载最新稳定版本，我现在是2.02为最新版
```php
wget https://github.com/zyx0814/dzzoffice/archive/2.02.tar.gz
```

2.解压文件
```php
tar -zxvf 2.02.tar.gz
```

3.将解压后的文件移动到apache的目录下，并改名为dzzoffice
```php
mv dzzoffice-2.02 /var/www/html/dzzoffice
```

4.然后将目录权限授权给apache启动用户，默认为apache用户，如果自己修改了，则以你修改的为准
```php
cd /var/www/html/
chown -R apache. dzzoffice
```

5.启动apache
```php
systemctl start httpd
systemctl enable httpd    # 设置开机启动apache
```

#### 访问页面进行安装
上一步已启动apache，现在可以直接访问你服务器的ip或域名，后跟dzzoffice的路径来来访问dzzoffice，
访问如：http://ip/dzzoffice 会自动跳转到安装界面：

安装完成后，手动删除安装文件
rm -rf /var/www/html/dzzoffice/install/index.php

#### 安装onlyoffice插件
dzzoffice本身不支持excel或者文档的在线浏览和编辑，需要额外的第三方工具进行支持，
在官方文档中也有说明：http://dzzoffice.com/corpus/list?cid=3#

这里我现在安装onlyoffice作为在线文档服务器，部署方式，由于直接在服务器上部署比较繁琐，这里我直接使用docker部署docker版本。首先安装docker，然后用docker启动onlyoffice

1.删除旧版本，确保机器没有docker
```php
yum remove docker docker-client docker-client-latest docker-common \
    docker-latest docker-latest-logrotate docker-logrotate docker-engine
```

2.安装依赖
```php
yum install -y yum-utils device-mapper-persistent-data lvm2
```

3.安装yum仓库
```php
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

4.安装docker
```php
yum install docker-ce docker-ce-cli containerd.io
```

5.启动docker
```php
systemctl start docker
systemctl enable docker
```

6.启动onlyoffice，使用本地的8000端口
```php
docker run -i -t -d -p8000:80 --restart=always onlyoffice/documentserver
```

启动onlyoffice服务后，在浏览器中访问http://ip:8000查看是否可以正常使用，如果出现如下界面，则为正常
    √ Document Server is running

```php
请输入OnlyOffice Document Server API地址:
http://DZZOFFICE_IP:8000

填写您的onlyoffice Document Server API地址，如:http://192.168.0.2:90/ 根据你的文档服务器填写
请输入文件服务器(dzzoffice服务器)地址:

填写您的文件服务器地址，如:http://dzzoffice.com:90/,根据你的dzzoffice服务器地址填写,留空使用当前地址。tips:设置内网地址可以提高文件传输速度

缩略图后缀列表:
支持生成缩略图的后缀列表，用小写字母。多个用英文逗号隔开，如：pdf,doc,docx,ppt,pptx,xls,xlsx
```
点击保存，然后启动应用

然后在文档，excel应用中，就可以直接点击在线浏览和编辑啦。
